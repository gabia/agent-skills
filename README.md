# Agent Skills

A collection of skills for AI coding agents to improve code quality, consistency, and best practices.

## Overview

This repository provides structured coding guidelines, conventions, and best practices that can be loaded by AI agents to ensure consistent, high-quality code generation. Each skill is designed to be modular and composable.

See [Agent Skills](https://agentskills.io/home) for more information.

## Installation

```shell
npx skills add gabia/agent-skills
```

## Available Skills

### Java Development

#### java-coding-guideline

Fundamental Java coding standards and conventions that apply to all Java code.

**When to use**: Load this skill at the start of any Java development task - writing, reviewing, or refactoring Java code.

**Key Topics**:

- Import statements (no wildcards)
- Constructor patterns (single primary constructor with delegation)
- Nullability handling (`@Nullable` annotations over `Optional`)
- Wrapper vs primitive types
- Immutability (`final` classes and fields)
- Exception handling
- Resource management (try-with-resources)
- Composition over inheritance
- Date/Time API (java.time, no legacy Date/Calendar)
- Static factory method naming (`of` convention)
- Package naming conventions
- Variable and method naming

#### java-xmlrpc-guideline

Best practices for Java XML-RPC client and server implementation using Apache XML-RPC library.

**When to use**: Load this skill when implementing XML-RPC clients or servers in Java, integrating with legacy XML-RPC APIs, or reviewing XML-RPC code for security vulnerabilities.

**Key Topics**:

- Apache XML-RPC client configuration
- Server implementation patterns (standalone and servlet)
- Transport and connection settings
- Security hardening (CVE-2019-17570 mitigation)
- Type mapping between XML-RPC and Java
- Anti-patterns to avoid

### Ansible

#### ansible-style-guideline

Authoring and review style rules for Ansible roles, tasks, and variables.

**When to use**: Load this skill when writing or reviewing Ansible roles, tasks, or variables, or fixing ansible-lint violations.

**Key Topics**:

- Required values with `undef()` and input validation with `meta/argument_specs.yml`
- FQCN modules and standard task keywords (no `with_*`, `action:`, `key=value`)
- Role-prefixed variable names (`__<prefix>_` for internal values)
- Booleans and conditions
- Minimal `become` (individual tasks only)
- Error handling and tags
- Files and templates (`template` over `copy`, `ansible_managed`, avoiding `lineinfile`)
- Review checklist
