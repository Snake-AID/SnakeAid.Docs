# 🤖 AGENT MEMORY: Backend Documentation Protocol
# SnakeAid — Baseline + Operations Model

> SYSTEM INSTRUCTION (STRICT)
>
> This file defines the official documentation standard for SnakeAid Backend.
> You MUST follow this protocol when creating, reading, or updating documentation.
>
> This system is designed for an AI-generated-first workflow.
> It prevents document drift, context mixing, and historical corruption.

---

# 📂 Repo Folder Structure (READ FIRST)

SnakeAid documentation is organized by **Flows** and **Layers**:

## 1) Vertical Flows (`01-flows/`)

* **Concept**: User journeys spanning multiple layers.
* **Naming**: `P{Priority}-{FlowName} / S{Sequence}-{SubFlowName}`
* **Example**: `01-flows/P1-emergency/S1-identified/`

## 2) Horizontal Layers (`02-layers/`)

* **Concept**: Technical infrastructure shared across flows.
* **Naming**: lowercase, kebab-case
* **Example**: `02-layers/aspnet-identity/`

---

# 🧱 Canonical Module Structure (Inside any Flow/Layer)

Every module root (either a flow folder or a layer folder) MUST follow:

```
<module-root>/
  <module>.introduction.md
  <module>.sourcecode.md
  <module>.usageguide.md

  operations/
    FEAT-short-slug/
      plan.md
      prompt.md

    FIX-short-slug/
      plan.md
      prompt.md
```

Example:

```
02-layers/aspnet-identity/
  aspnet-identity.introduction.md
  aspnet-identity.sourcecode.md
  aspnet-identity.usageguide.md

  operations/
    FEAT-refresh-token/
      plan.md
      prompt.md
```

---

# 🧠 Core Philosophy

Docs are split into **Baseline** (current truth) and **Operations** (controlled changes).

## 1️⃣ Baseline (Current System State)

Represents the system **as it exists in code right now**.

Baseline files:

* `*.introduction.md`
* `*.sourcecode.md` (CRITICAL)
* `*.usageguide.md`

Baseline must NEVER contain future plans.

## 2️⃣ Operations (Controlled Mutations)

Represents **intentional changes** to the baseline.

* Each operation is isolated in its own folder.
* Operations are append-only artifacts.
* Do not merge unrelated changes into one operation.
* Do not overwrite historical operations.

---

# 🚦 Quick Rules (Non-Negotiable)

* Baseline describes current reality only.
* Operations describe change intent only.
* Never mix future plans into baseline.
* Always update baseline after implementation.
* Never store secrets in documentation.
* UTF-8 only (no BOM).

---

# 🧾 Baseline Documents (State Layer)

## 1) `<module>.introduction.md`

Purpose:

* Domain context
* Business rules / invariants
* Scope / out-of-scope
* Non-functional requirements

Explains WHY the module exists.

## 2) `<module>.sourcecode.md` (CRITICAL)

Compressed Source of Truth.

Must reflect:

* Public API surface (endpoints / public services)
* Key signatures
* Entities and schema
* Relationships (Mermaid allowed)
* Invariants and guarantees
* Cross-cutting concerns (auth, logging, caching)

Rules:

* No TODOs
* No future plans
* No “about to implement”
* Only what exists in code

## 3) `<module>.usageguide.md`

External contract for consumers.

Includes:

* Endpoints
* Request/Response examples
* Status codes
* Error catalog
* Auth requirements

Update whenever contract changes.

---

# ⚙️ Operations (Mutation Layer)

Every change MUST be implemented through an operation folder:

## Operation Folder Naming

```
{TYPE}-{short-slug}
```

Allowed TYPE:

* FEAT
* FIX
* REFACTOR
* PERF
* SECURITY
* HOTFIX

Examples:

* `FEAT-refresh-token`
* `FIX-null-claim`
* `SECURITY-rate-limit`

Do NOT include dates in folder names.
Dates belong in metadata.

---

# 📄 Operation Content

Each operation contains:

```
plan.md
prompt.md
```

## plan.md (Strategy Layer)

Written BEFORE implementation.

Purpose:

* Read baseline
* Assess current state (As-Is)
* Define gap
* Define To-Be direction/design
* List impacted components
* Define risks/constraints
* Define validation plan

REQUIRED STRUCTURE:

1. As-Is (from `<module>.sourcecode.md`)
2. Gap Analysis
3. To-Be Design
4. Impacted Components
5. Risks & Constraints
6. Validation Plan

## prompt.md (Execution Layer)

Generated FROM plan.md.

Purpose:

* Convert plan into precise machine instructions
* Define required outputs
* Define forbidden changes
* Define test requirements

Rules:

* No secrets
* No environment-specific values
* Do not modify unrelated modules

---

# 🔁 Operation Lifecycle

1. Create operation folder.
2. Write plan.md.
3. Generate prompt.md.
4. Implement code.
5. Update baseline:

   * `<module>.sourcecode.md`
   * `<module>.usageguide.md`
6. Mark operation status as `done`.

Operations are historical artifacts.
Never delete past operations.

---

# 🧾 Required Frontmatter

## Baseline files frontmatter

```yaml
---
doc_role: baseline
module: <module-name>
kind: <layer|flow>
status: active
last_updated: YYYY-MM-DD
owners: [backend-team]
---
```

## Operation plan.md frontmatter

```yaml
---
doc_role: operation
operation_id: FEAT-refresh-token
type: FEAT
status: draft   # draft | approved | in_progress | done
created_at: YYYY-MM-DD
affects:
  - Controllers/AuthController
  - Services/TokenService
---
```

## Operation prompt.md frontmatter

```yaml
---
doc_role: operation
operation_id: FEAT-refresh-token
generated_from: plan.md
status: draft
created_at: YYYY-MM-DD
---
```

---

# 🔒 Security & Privacy Rules

Never write secrets into docs:

* API keys, tokens, passwords, private certs, JWT secrets
* Internal IPs/hostnames, production URLs, personal data

Always use placeholders:

* `{{JWT_SECRET}}`, `{{DB_PASSWORD}}`, `https://api.example.com`

---

# 📅 Encoding Hygiene (UTF-8 only)

* All markdown files MUST be UTF-8 (no BOM).
* If mojibake occurs, repair once with `ftfy.fix_encoding`.
* Do not keep alternate encodings in the repo.

---

# 🧩 Context Loading Protocol (For AI Agents)

When working on a module:

1. Locate module under `01-flows/` or `02-layers/`.
2. Read `<module>.introduction.md`.
3. Read `<module>.sourcecode.md`.
4. If you need changes:

   * Create a new `operations/{TYPE}-{slug}/`
   * Write `plan.md` → generate `prompt.md`
   * Do NOT edit baseline until implementation completes.

This minimizes tokens and prevents drift.

---

# 🧠 Mental Model

Baseline = Current State
Operation = Controlled Mutation
Plan = Strategy
Prompt = Execution

System evolves through operations.
Baseline always reflects final truth.

---

# 📌 Scalability Rule

Only ONE operation should be `in_progress` per module at a time,
unless explicitly coordinated.

Parallel operations risk state conflict.