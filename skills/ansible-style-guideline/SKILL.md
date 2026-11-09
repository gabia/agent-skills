---
name: ansible-style-guideline
description: >
    Ansible role/task/variable authoring style rules. Required variables use undef(), parameter validation uses argument_specs,
    tasks use FQCN and standard keywords only, variables use the role prefix, booleans are true/false only, minimal become.
    Use for "write ansible", "create a role", "style review", "fix lint", "check variable names", and similar requests.
---

# ansible-style-guideline

Follow these rules when writing new roles/tasks/variables or reviewing existing code.
Format rules that ansible-lint or `.editorconfig` can check are not covered here.

## 1. Required Values and Validation

- **Declare required values with `undef()`.** For values the user must pass, give no default and use `undef()`.
  Include a hint message.

    ```yaml
    # defaults/main.yml
    opensearch_app_name: "{{ undef(hint='opensearch_app_name is required') }}"
    opensearch_port: 9200
    ```

- **Validate parameters with `argument_specs`.** Define role inputs in `meta/argument_specs.yml`
  and specify type, `required`, and `choices`. Use `assert` only for cross-parameter checks that argument_specs cannot express.

    ```yaml
    # meta/argument_specs.yml
    argument_specs:
        main:
            short_description: OpenSearch application library
            options:
                opensearch_app_name:
                    type: str
                    required: true
                opensearch_port:
                    type: int
    ```

## 2. Variable and Role Names

- **Every variable in a role uses the role name as its prefix.**
    - role `opensearch` → `opensearch_*`
    - role `gcldep_opensearch` → `gcldep_opensearch_*`
- Derived or temporary values used only inside the role start with `__<prefix>_` (e.g. `__opensearch_bind_ips`). Users do not change them, so put them in `vars`, not `defaults`.
- Referencing global variables (`gcldep_globals`, etc.) directly from a role follows the upper-layer rules.
- Role names use only lowercase letters and underscores, no dashes. A name with dashes cannot be used inside a collection or as a variable prefix.
- Use only underscores as special characters in variable names. Jinja interprets `-` and `.` as operators or attribute access.

## 3. Writing Tasks

- Write modules with **FQCN** (`ansible.builtin.copy`, `ansible.posix.sysctl`).
- Use only **standard task keywords**: `name`, the module, `when`, `loop`, `register`, `vars`, `become`,
  `changed_when`, `failed_when`, `notify`, `tags`, etc.
- Do not use legacy shorthand or alias forms: `action:`, `local_action:`, `with_*` (→ `loop`), short module names (`copy:`).
- Write module arguments in YAML style, not `key=value`. With `key=value`, every value is parsed as a string and lint cannot check the arguments.
- Give every task a meaningful `name`. Write it in the imperative (e.g. "Ensure service is running") and do not put variables or loop items in it. Names appear in run logs and are used by `--start-at-task`, so a fixed string is easier to find.
- Prefer descriptive task names over comments. Delete dead code instead of commenting it out; git keeps the history.
- Add `changed_when: false` to tasks that only gather information. Without it, the task reports changed on every run and real changes become hard to spot.
- Set `state` explicitly to `present` or `absent`. `latest` can install a different version on each run, so results are not reproducible.
- Pass multiple packages as a list to a single `ansible.builtin.package` call instead of looping. The package manager runs once, which is faster and resolves dependencies together.
- Use shell commands (`command`, `shell`) only when no dedicated module can replace them, and leave a comment explaining why. Shell commands do not guarantee idempotency or check mode support.
- Put one space inside Jinja delimiters (`{{ variable_name }}`). This matches the ansible-lint `jinja[spacing]` rule.
- Use American English spelling. Ansible modules and official docs use American spelling, which helps searchability and consistency.
- **Add `quiet: true` to `assert`.** Always add it unless there is a specific reason not to; it cuts unnecessary output when the check passes.

    ```yaml
    - name: Validate TLS settings
      ansible.builtin.assert:
          that:
              - opensearch_tls_cert_path is defined
          fail_msg: "Certificate path is required when TLS is enabled."
          quiet: true
    ```

## 4. Booleans and Conditions

- Name boolean variables in the positive form with a meaningful prefix such as `is_`, `enable_`, or `use_`. Example: `opensearch_enable_tls: true` (not `disable_tls`).
- Write boolean values only as `true` or `false`. Avoid `yes`/`no`, `True`, or quoted `"true"`. `yes`/`no` are booleans only in YAML 1.1, a quoted value is a string, and anything other than lowercase `true`/`false` fails the yamllint `truthy` rule that ansible-lint applies.
- Use booleans directly, as in `when: x` or `when: not x`. `x == true` gives different results when mixed with the string `"true"`.
- Split conditions joined with `and` in `when` into a list. Each condition reads on its own line and diffs stay small.
- Do not use `when: x is changed`; use a handler instead. A handler runs only once even when notified by several tasks, and can be handled with `--force-handlers` when rerunning after a failure.
- Compare numbers with ordinary operators such as `==`, `<`, and `>`.
- When comparing strings by pattern, use `match`, `search`, or `regex` anchored with `^...$`. Without anchors, `match('CentOS')` also matches `CentOS Stream`, an unintended partial match. When you only need to check that two strings are exactly equal, `==` is fine.

## 5. become

- Use **`become` only as much as needed**.
- Do not set `become: true` on a whole play or role. Add it only to the **individual tasks** that need privileges.
- Set `become_user` only when a different user is actually required.
- Do not add it to read-only check tasks (`stat`, query `command`s, etc.).

## 6. Error Handling

- Use `ignore_errors` and `failed_when` only where truly needed. Hiding a failure makes the next task run on a broken state. If only specific results should not count as failures, use a narrowly scoped `failed_when` instead of `ignore_errors`.
- Do not use `ignore_errors: true` on a block that contains `assert`. It disables the check.
- Do not use `meta: end_play`. Use `meta: end_host` if needed. `end_play` stops every host because of one host's condition.

## 7. Tags

- Add tags only for the role name or a meaningful purpose. With too many tags, nobody knows which tag combinations are safe.
- Do not create tags that produce dangerous results when run alone. For example, a tag that restarts the service without deploying its config can bring the service up with a bad config.

## 8. Files and Templates

- Deploy most files with `ansible.builtin.template`; use `ansible.builtin.copy` only for static files or binaries. If a variable is needed later, you do not have to switch modules.
- Put `{{ ansible_managed | comment }}` on the first line of a template, and do not put timestamps in templates. The header warns that manual edits will be overwritten, and a timestamp changes the content on every run so the task is always changed.
- Use `backup: true` when replacing files. A bad deployment can be rolled back right away.
- Start file include paths with `{{ role_path }}/`. Relative paths resolve differently depending on where they are called from.
- Avoid `ansible.builtin.lineinfile`; use `ini_file`, `blockinfile`, `template`, or a dedicated module. `lineinfile` does not guarantee the state of the rest of the file, and a wrong regex duplicates lines. When it cannot be avoided, such as when there is no source file to manage as a template, use it and leave a comment explaining why.

## 9. Review Checklist

After writing or reviewing, check the following and report any violations.

- [ ] Required variables are declared with `undef()`
- [ ] `meta/argument_specs.yml` exists and covers every external input variable
- [ ] Modules use FQCN, and there is no `action:`/`local_action:`/`with_*`
- [ ] Every `assert` has `quiet: true`, or there is a reason it does not
- [ ] All role variables have the role prefix
- [ ] `become` is only on individual tasks, and only where needed

## 10. References

Some rules in sections 2, 3, 4, 6, 7, and 8 come from the Red Hat COP documents below.

- [RHIS ansible practices (Red Hat COP)](https://redhat-cop.github.io/rhis-code/ansible_practices)
- [Red Hat COP automation good practices](https://redhat-cop.github.io/automation-good-practices)
- [automation good practices (GitHub mirror)](https://github.com/kubealex/automation-good-practices)
- [Ansible 4 style guide (for docs contributors, reference only)](https://docs.ansible.com/ansible/4/dev_guide/style_guide/index.html)
