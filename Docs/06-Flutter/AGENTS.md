# AGENT MEMORY: Flutter Integration Protocol

# SnakeAid.Mobile — Integration-Driven Model

> SYSTEM INSTRUCTION (STRICT)
>
> This file defines the official documentation standard for SnakeAid.Mobile (Flutter).
> You MUST follow this protocol when creating, reading, or updating documentation.
>
> Flutter is a Backend Contract Consumer.
> Backend remains the authoritative source of truth.
>
> This system prevents contract drift, duplication, and unsafe AI-generated code.

---

# Documentation Repository Structure

All documentation lives in the separate Docs repository.
Flutter documentation is organized as follows:

```
Docs/
  06-Flutter/
    AGENTS.md

    01-Standards/
      networking-core.standards.md
      contract-governance.standards.md

    02-Features/
      <feature>/
        <feature>.integration.md
        operations/
          <TYPE>-<short-slug>/
            plan.md
            prompt.md

    03-Migrations/
      <topic>.migration.md
```

Operations are NOT a top-level category.
They are subordinate to a specific feature.

---

# Core Philosophy

Flutter is NOT a source of business truth.
Flutter is an Integration Boundary + Presentation Layer.

Backend owns:

- Business rules
- Schema
- API contract
- Invariants

Flutter owns:

- Contract consumption
- DTO mapping
- State orchestration
- UI rendering

Flutter documentation focuses on:

- Contract alignment
- Mapping clarity
- Drift prevention
- Duplication prevention

It does NOT redefine backend behavior.

---

# 01-Standards

Standards define long-term architectural rules.
All standard documents MUST use the suffix:

```
.standards.md
```

Examples:

- networking-core.standards.md
- contract-governance.standards.md

Standards are:

- Long-lived
- Architecture-defining
- Not time-bound
- Not tied to a single feature
  Standards MUST NOT describe temporary refactors.

---

## networking-core.standards.md

Defines:

- HttpService responsibilities
- Base URL configuration
- Interceptor chain
- Token refresh mechanism
- Error normalization contract
- Endpoint exclusion rules
  All features depend on this documented networking contract.

---

## contract-governance.standards.md

Defines:

- No hardcoded endpoints
- Mandatory backend_reference in operations
- DTO naming conventions
- Error normalization policy
- Rules for backend schema changes
  This document prevents contract drift.

---

# 02-Features

Each feature documentation lives under:

```
Docs/06-Flutter/02-Features/<feature>/
  <feature>.integration.md
  <feature>.function-graph.md (optional)
  operations/
```

Example:

```
Docs/06-Flutter/02-Features/auth/
  auth.integration.md
  auth.function-graph.md
  operations/
    FEAT-integrate-refresh-token/
      plan.md
      prompt.md
```

---

## <feature>.integration.md

This is the integration baseline truth for the feature.
It MUST document:

1. Backend Endpoints Consumed

- Endpoint path
- HTTP method
- Authentication requirement
- Backend usageguide reference

2. DTO Mapping Table

| Backend Field | Dart Field | Nullable | Notes |
| ------------- | ---------- | -------- | ----- |

3. Repository Surface

- Public repository methods
- Return types
- Error behavior

4. State Flow
   UI → Provider → Repository → HttpService → Backend

5. Error Handling Strategy

- Shared error normalization usage
- No duplicated \_handleError

6. Backend Dependencies

- Token refresh reliance
- Required headers
- Backend invariants assumed

RULES:

- Do NOT duplicate backend usageguide.
- Do NOT redefine backend schema.
- Only describe integration assumptions.

---

## <feature>.function-graph.md (Optional Supplement)

This document serves as a high-level code architecture reference for how a feature's components interact at the Flutter client level.
It MUST document:

1. Sequence Diagram

- Use Mermaid syntax (`sequenceDiagram`)
- Clearly map out interactions between UI (Widgets), State Management (Providers), Managers/Services, and Backend APIs

2. Core Components and Logic

- List the key files and classes involved
- Describe their responsibilities (e.g., Trigger, Service Manager, Network Service)
- Explain complex internal logic (e.g., Throttling, Permission Handling)

3. Architecture Overview

- Summarize the Separation of Concerns (SoC) for the feature.

RULES:

- Do NOT duplicate `.integration.md` (which maps the contract). Focus exclusively on the codebase execution flow.
- Use this primarily for complex features that span across multiple layers (UI, Provider, Core Service).

---

# Feature Operations (Subordinate Element)

Every backend integration MUST go through an operation.
Operations live under a feature, not as a peer category.

Folder naming:

```
{NN}-{TYPE}-{short-slug}
```

Where `{NN}` is a **sequential zero-padded number** (01, 02, 03...) based on creation order within the feature.
Always check existing operations to determine the next sequence number.

Allowed TYPE:

- FEAT
- FIX
- REFACTOR
- PERF
- SECURITY

Each operation contains:

```
plan.md
prompt.md
```

---

## plan.md (Integration Strategy)

Written BEFORE coding.
REQUIRED STRUCTURE:

1. Backend Contract Reference

- Backend repository path
- Backend module path
- Backend operation (if any)
- Backend usageguide section

2. As-Is (Flutter State)

- Existing models
- Existing repository methods
- Existing provider flow

3. Gap Analysis

- New models required?
- Repository changes?
- State changes?

4. Mapping Definition

- JSON key expectations
- Nullable rules
- Error structure expectations

5. Duplication Check

- Endpoint must use central constant
- No duplicate \_handleError

6. Validation Plan

- Happy path scenario
- Token expiry scenario
- Error scenario

---

## prompt.md (Execution Instructions)

Generated FROM plan.md.
Must enforce:

- Use centralized endpoint definitions
- No hardcoded URLs
- No duplicated error handling
- Respect backend JSON schema exactly

AI MUST NOT:

- Invent endpoint paths
- Modify backend assumptions
- Create new interceptors
- Copy error handling logic

---

# 03-Migrations

Migration documents describe transitional refactors.
All migration documents MUST use the suffix:

```
.migration.md
```

Examples:

- migrate-manual-json-to-json-serializable.migration.md
- unify-error-handling.migration.md

Migration documents:

- Are temporary
- Have clear start and end state
- Include completion criteria
- May be archived after completion
  They are NOT architectural standards.

---

# Backend Contract Dependency Rule (CRITICAL)

Flutter must NEVER define API contracts independently.
All endpoints, request/response models, and error assumptions MUST be traceable to Backend documentation.
Each operation MUST include a backend_reference block in plan.md.

---

# Operation Lifecycle

1. Backend operation completes.
2. Read backend usageguide.
3. Create feature operation.
4. Write plan.md with backend reference.
5. Generate prompt.md.
6. Implement code.
7. Update <feature>.integration.md.
8. Mark operation status as done.

---

# Traceability Model

Backend Operation → Backend Baseline → Flutter Operation → Flutter Integration Baseline

---

# Scalability Rule

Only ONE integration operation per feature should be `in_progress` at a time.

---

# Security & Hygiene

- Never store API keys or tokens in docs.
- Use placeholders.
- UTF-8 only (no BOM).

---

# Canonical Statement

This protocol guarantees:

- Explicit backend contract traceability
- Clear separation of standards, features, and migrations
- No integration drift
- No duplicated networking logic
- AI-safe Dart generation
