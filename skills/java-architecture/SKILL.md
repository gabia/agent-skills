---
name: java-architecture
description: Team architecture conventions for designing and evolving Java-based systems (Arc42 + C4 docs, ADR/RFC process, modular-monolith boundaries, layering, and API cross-cutting rules).
license: MIT
metadata:
  author: agent-skills
  version: "1.0"
  language: java
---

# Java Architecture (Team Conventions)

This skill captures our team’s architecture conventions and “how we design systems” guidance.

**Primary source (SSOT):**
- `gcloud-docs/architecture/src/main` (Arc42-based docs, C4 diagrams, conventions)

> Keep the architecture docs up to date as the system evolves (SSOT principle).

## When to use this skill

Load this skill when you are:
- Designing a new module/service or changing module boundaries
- Introducing a new framework, methodology, or significant refactor
- Making decisions that affect cross-cutting concerns (API design, observability, auth, concurrency)
- Writing/reviewing architecture artifacts (Arc42 sections, C4 diagrams, ADRs, RFC discussions)

Also load:
- `java-coding-guideline` for universal Java code conventions

## Non-negotiable conventions

### 1) Architecture documentation: Arc42 + C4

- **Architecture document structure:** follow **Arc42** sections.
- **Diagrams:** use **C4** for context/container/component views.
- Treat documentation as a product artifact, not a one-time deliverable.

Minimum required views for meaningful changes:
- **System Context** (who uses it, what external systems exist)
- **Container/Component** (major runtime/process boundaries, dependencies)
- **Building Block View** (module decomposition and responsibilities)

### 2) ADR (Architecture Decision Record)

Use ADRs to record **why** we made a decision and what alternatives were considered.

Write an ADR at least for:
1. 3rd-party library **add/change/remove**
2. A change expected to be **large or disruptive**
3. Introducing a **new concept/methodology** not present in the existing system
4. When explicitly requested by a manager/lead

ADR should include:
- Context / problem statement
- Decision
- Alternatives considered
- Consequences (trade-offs, risks, follow-ups)

### 3) RFC (Requests for Comments) for team alignment

For large changes or new concepts, first drive discussion via an issue / RFC-style thread.
After consensus, capture the decision in an ADR.

## Architectural style and system decomposition

### Modular Monolith (default)

We prefer a **modular monolith** as a pragmatic middle-ground:
- Keep a single deployable unit where appropriate
- Enforce strong module boundaries internally
- Design modules so they can be extracted into microservices later, if needed

Rules of thumb:
- Modules should not hard-couple via database-style foreign keys across modules.
- **Inter-module communication should be via APIs/contracts**, not direct deep linking.

### Poly-repo (default)

We use **poly-repo** to avoid monorepo versioning and release-branch pitfalls.
To manage the repo sprawl:
- Use `git-repo` (multi-repo coordination)
- Use Gradle **Composite Builds** for local development across repositories

## Layering rules (API server)

The API server follows **layered architecture**. Each layer has clear responsibilities.

### Controller layer
Responsibilities:
- Request DTO validation (shape/range checks only)
- AuthN/AuthZ (usually via filters/middleware)
- Route/facade orchestration (minimal “traffic control”)
- Response mapping

Rules:
- If validation requires database lookups or domain invariants, it belongs in **Service**.
- After Controller validates a request, deeper layers **must not re-validate** the same DTO fields.
- If Controller requires complex branching, re-check the design (risk of bloated controllers).

### Service layer
Responsibilities:
- Business logic
- Domain invariant enforcement
- Transaction boundaries (as appropriate)

### Driver layer
Responsibilities:
- Host/agent interactions
- System calls / external tool execution
- May access DB **only when the data is driver-specific** and not meaningful to higher layers

### Domain layer

We borrow DDD concepts, but do not force “everything must be behavior-rich objects”.
Prefer clarity and enforce invariants where they naturally belong.

Key guidance:
- Keep aggregate roots (AR) from becoming too large
- Split ARs when the “many-side” could become large (avoid loading huge collections)

### Infrastructure layer
Infrastructure handles persistence and domain state IO.

Team convention:
- Multiple layers may access infrastructure directly when it avoids meaningless proxy methods.
- Still maintain **module boundaries**; avoid using infrastructure as a backdoor across modules.

## Domain modeling rules

### Strongly typed IDs

Avoid mixing IDs from different entities by using strongly typed IDs instead of raw primitives.

**Do (example):**
```java
public record NetworkId(long value) {}
public record SubnetworkId(long value) {}
```

**Don’t:** pass raw `long` IDs across unrelated entity types.

### One-to-many: avoid embedding large collections

Do not embed unbounded collections inside an aggregate root.
Instead model the “many-side” as its own AR with pagination-friendly repositories.

## Shared resource “Lease” concept

For cluster-unique/shared resources (e.g., IPs, NICs), use a **lease** concept:
- The managing domain owns the resource registry (“key checkout ledger”)
- Consumer domains request a lease, then materialize/realize usage in their own domain

Implementation guidance:
- Track ownership with a URN-like identifier:
  - `urn:fdc:gcloud-hawaii.gabia.com:2024:<domain>:<resource-alias>[#<fragment>]`
- Managing domain should not assume how consumers materialize the resource.

## Data deletion policy

Prefer **hard delete** at the database level to reduce application complexity and improve integrity.

If audit/restore is required:
- Record deleted resources into a separate “deleted_*” table (or dedicated audit storage), rather than mixing soft-delete flags into primary tables.

## API cross-cutting rules

### REST API design

- List endpoints should provide pagination unless there is a strong reason not to.
- Expose pagination links via **RFC 8288 Web Linking**:
  - `rel="first"`, `rel="next"`, `rel="prev"`, `rel="last"`

### Idempotency

Resource creation APIs should support `Idempotency-Key` to prevent accidental duplicate creation.

### Error format

For error responses, return **RFC 7807 Problem Details**.

## Observability and tracing

In distributed flows, ensure cross-service request tracking.

Rules:
- Prefer **W3C Trace Context** propagation.
- Ensure logs/metrics/traces can be correlated via TraceId/SpanId.

## gRPC (server-agent) design rules (when applicable)

Baseline:
- Follow Google’s API design guidance where it applies.

Key rules:
1. **RPCs should operate at node-level** (batch work per node), not per tiny sub-resource calls.
2. Acquire **exclusive locks** before agent operations that mutate system state.
3. Long-running work should be **asynchronous** and expose an **Operation**-style API.
4. **Success via return values; failures via exceptions** (don’t rely on sentinel values).

## Decision-making checklist (use in reviews)

- [ ] Does this change require an ADR? (3rd-party change / large change / new concept / requested)
- [ ] Are module boundaries respected (no hidden coupling)?
- [ ] Is the layering respected (controller thin, business logic in service, drivers isolated)?
- [ ] Are shared resources managed via lease (clear ownership and tracking)?
- [ ] Are APIs consistent (pagination, idempotency, RFC 7807 errors)?
- [ ] Is observability included (W3C trace context, log correlation)?

## References (SSOT)

Use these as the canonical sources for details and diagrams:
- `gcloud-docs/architecture/src/main/quarto/architecture-constraints.qmd`
- `gcloud-docs/architecture/src/main/quarto/solution-strategy.qmd`
- `gcloud-docs/architecture/src/main/quarto/solution-strategy-server.qmd`
- `gcloud-docs/architecture/src/main/quarto/cross-cutting-concepts.qmd`
- `gcloud-docs/architecture/src/main/quarto/build-block-view*.qmd`
