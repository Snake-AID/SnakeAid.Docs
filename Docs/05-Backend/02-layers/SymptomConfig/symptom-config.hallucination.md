# Symptom Config Hallucination

## Status

Current status:

- closed

## Purpose

This file records ambiguity that should not be silently invented and keeps decision records when a risk is closed.

Main docs affected by these decisions:

- `symptom-config.introduction.md`
- `symptom-config.roadmap.md`
- `symptom-config.sourcecode.md`
- `symptom-config.useguide.md`

When a risk is solved, keep the original option list, add the chosen decision, promote the decision into the baseline docs, and mark the risk closed.

## Current Direction Summary

Current recommended direction:

- mobile/frontend should use `GET /api/symptom-configs/grouped-for-ui` for patient-facing dropdown/question rendering
- admin CRUD should use controlled selects for `GroupName` and `AttributeKey`
- `VenomTypeId` should stay optional in UI and docs
- frontend/admin UI owns the requested group/key validation matrix to avoid backend technical debt
- backend should change delete behavior to soft-delete by setting `IsActive = false`
- backend authorization enforcement has been added before documenting CRUD as admin-enforced

## Risk 1. Backend Matrix Enforcement Location

### Context

The requested business rule is clear:

- `BACKGROUND` allows `AGE_GROUP`, `MEDICAL_HISTORY`, `BITE_LOCATION`
- `LOCAL` allows `SYMPTOM_LOCAL`
- `CRITICAL` allows `CORE_SIGNS`

Current code does not enforce this matrix. The frontend can enforce it immediately, but backend hardening still needs a design choice.

### Options

- option A: enforce in `SymptomConfigService` with a private helper method
- option B: create a dedicated `SymptomConfigRules` or `SymptomConfigPolicy` class and call it from the service
- option C: use DTO validation attributes only
- option D: enforce in database check constraints

### Recommended Decision

- choose option A

### Reason

- the user explicitly chose frontend-owned validation to avoid adding backend technical debt
- the current backend CRUD already exists and should not be changed only to enforce this dropdown matrix
- the admin UI can prevent invalid combinations with dependent selects

### Decision

- chose option A

### Decision Record

- original options:
  - option A: enforce in `SymptomConfigService` with a private helper method
  - option B: create a dedicated `SymptomConfigRules` or `SymptomConfigPolicy` class and call it from the service
  - option C: use DTO validation attributes only
  - option D: enforce in database check constraints
- chosen direction:
  - frontend/admin UI handles the group/key matrix validation
  - backend is not changed for matrix validation in this scope
- status:
  - closed

## Risk 2. AttributeLabel Authority

### Context

The current request DTOs allow clients to submit `AttributeLabel`. The prod mirror has canonical labels for each `AttributeKey`.

There is ambiguity about whether admin should be allowed to override the label for a known key.

### Options

- option A: backend auto-derives `AttributeLabel` from `AttributeKey` and ignores client label
- option B: backend validates that submitted `AttributeLabel` exactly matches the canonical label for the key
- option C: backend keeps `AttributeLabel` editable, while admin UI auto-fills it by default

### Recommended Decision

- choose option C for now

### Reason

- it matches current code with minimal behavior change
- it gives admin flexibility while frontend still gets display text from the API
- strict label enforcement can be added later if product wants labels to be controlled vocabulary

### Decision

- chose option C

### Decision Record

- original options:
  - option A: backend auto-derives `AttributeLabel` from `AttributeKey` and ignores client label
  - option B: backend validates that submitted `AttributeLabel` exactly matches the canonical label for the key
  - option C: backend keeps `AttributeLabel` editable, while admin UI auto-fills it by default
- chosen direction:
  - backend keeps `AttributeLabel` editable
  - admin UI should auto-detect and auto-fill the related `GroupName`, `AttributeKey`, and `AttributeLabel` when an admin chooses one of the known labels/keys
  - admin can still edit the label if product needs a wording adjustment
- status:
  - closed

## Risk 3. Delete Versus Deactivate

### Context

The current API has `DELETE /api/symptom-configs/{id}` and also supports `IsActive` in create/update. The grouped UI endpoint only reads active options.

For admin tools, hard deletion may remove options that historical consultation data still references by ID.

### Options

- option A: keep hard delete as the main admin delete action
- option B: make admin "delete" call `PUT` with `IsActive = false`, and reserve hard delete for backend maintenance
- option C: change backend delete to soft-delete by setting `IsActive = false`

### Recommended Decision

- choose option C

### Reason

- the user explicitly chose backend soft delete
- the entity already has `IsActive`, and grouped UI reads already exclude inactive rows
- soft delete prevents accidental removal of config IDs that may be referenced by historical user flows

### Decision

- chose option C

### Decision Record

- original options:
  - option A: keep hard delete as the main admin delete action
  - option B: make admin "delete" call `PUT` with `IsActive = false`, and reserve hard delete for backend maintenance
  - option C: change backend delete to soft-delete by setting `IsActive = false`
- chosen direction:
  - `DELETE /api/symptom-configs/{id}` should set `IsActive = false` instead of hard-deleting
  - `GET /api/symptom-configs` currently fetches all rows because `GetAllSymptomConfigAsync()` has no `IsActive` predicate
  - `GET /api/symptom-configs/filter` can also fetch all rows when `isActive` is omitted, or fetch inactive rows with `isActive=false`
  - `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` currently fetch active rows only
- user confirmation:
  - the current read-endpoint split is accepted
  - keep `GET /api/symptom-configs` and `GET /api/symptom-configs/filter` as admin/all-data surfaces that can include inactive records
  - keep `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` as active-only rendering surfaces
- status:
  - closed

## Risk 4. Auth And Role Contract

### Context

Swagger descriptions mention admin-only behavior for create/update/delete, but the current controller snippet does not show explicit authorization attributes.

The docs should not invent a verified role requirement if the code does not enforce it in the controller or a global policy.

### Options

- option A: document admin-only as product intent but mark enforcement as unverified
- option B: document CRUD as currently callable by any authenticated role
- option C: add backend authorization enforcement first, then update docs

### Recommended Decision

- choose option C

### Reason

- the user explicitly chose backend authorization enforcement first
- create, update, delete, and admin read surfaces should not rely on Swagger wording alone
- docs should be updated after code enforcement exists so the contract does not overstate current behavior

### Decision

- chose option C

### Decision Record

- original options:
  - option A: document admin-only as product intent but mark enforcement as unverified
  - option B: document CRUD as currently callable by any authenticated role
  - option C: add backend authorization enforcement first, then update docs
- chosen direction:
  - add backend authorization enforcement first
  - update useguide after enforcement is code-verified
- status:
  - closed

## Closure Summary

All current risks have user decisions.

Implementation decisions have been promoted into `symptom-config.roadmap.md`:

- keep group/key matrix validation on frontend/admin UI
- keep `AttributeLabel` editable in backend while admin UI auto-fills related metadata
- backend delete behavior is soft delete
- backend authorization enforcement is active before documenting CRUD as enforced admin-only

## Promotion Rule

Only promote a risk decision into the main docs after:

- one option is chosen
- the choice is reflected in roadmap and useguide
- the resulting contract is specific enough to implement and test
