---
doc_role: implementation
module: symptom-config
kind: layer
doc_type: roadmap
status: active
last_updated: 2026-04-29
owners: [backend-team]
verification_status: code-verified
---

# Symptom Config Roadmap

## Current Status Snapshot

- module status: `Implemented CRUD, docs baseline created`
- domain entity: `Available`
- CRUD controller: `Available`
- grouped UI endpoint: `Available`
- prod mirror CSV: `Available`
- backend `GroupName` to `AttributeKey` matrix enforcement: `Intentionally not planned for this scope`
- frontend/admin `GroupName` to `AttributeKey` matrix validation: `Required`
- backend soft delete for `DELETE`: `Implemented`
- backend authorization enforcement: `Implemented for admin read/write endpoints`
- docs baseline: `Created from code-verified state`

## Current Truth To Resume From

This roadmap is written so work can resume from zero memory.

Current verified state:

- `SymptomConfig` is the configuration table used for symptom question options.
- `GET /api/symptom-configs/grouped-for-ui` is the best current frontend endpoint for dynamic dropdown/question rendering.
- `POST`, `PUT`, and `DELETE` exist for CRUD.
- `GET /api/symptom-configs/filter` supports paginated admin listing with filters.
- `TimeScoreList` is accepted in create/update requests and serialized into `TimeScoresJson`.
- `VenomTypeId` is nullable and should not be required by frontend/admin UI.
- The target group/key matrix is a frontend/admin UI validation rule for this scope, not a backend validation rule.
- `GET /api/symptom-configs` currently returns active and inactive rows because the service has no `IsActive` predicate.
- `GET /api/symptom-configs` remains flat and is ordered by `DisplayOrder`, then `Id`.
- `GET /api/symptom-configs/filter` can return active and inactive rows when `isActive` is omitted; it can fetch only inactive rows with `isActive=false`.
- `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` currently return active rows only.
- `DELETE /api/symptom-configs/{id}` soft-deletes by setting `IsActive = false`.
- `POST`, `PUT`, `DELETE`, `GET /api/symptom-configs`, `GET /api/symptom-configs/filter`, and `GET /api/symptom-configs/{id}` require role `Admin`.
- `GET /api/symptom-configs/grouped` and `GET /api/symptom-configs/grouped-for-ui` remain public active-only read endpoints.

## Target Outcome

After this module is fully hardened:

1. frontend/mobile can render symptom questions from a single grouped endpoint
2. admin UI can CRUD options using controlled group/key selections
3. invalid `GroupName` and `AttributeKey` combinations cannot be submitted accidentally from the admin UI
4. request/response examples stay synchronized with backend DTOs
5. this folder contains enough context to resume implementation without chat memory
6. delete uses soft-delete semantics by setting `IsActive = false`
7. backend enforces authorization for admin CRUD before docs mark CRUD as admin-enforced

## Locked Functional Direction

- [x] keep `BACKGROUND`, `LOCAL`, and `CRITICAL` as the documented group names
- [x] keep `AGE_GROUP`, `MEDICAL_HISTORY`, `BITE_LOCATION`, `SYMPTOM_LOCAL`, and `CORE_SIGNS` as the documented attribute keys
- [x] document `VenomTypeId` as optional
- [x] document the current grouped UI endpoint as the primary mobile/frontend read endpoint
- [x] recommend controlled admin selects instead of free-text group/key entry
- [x] record user decision that group/key matrix validation stays in frontend/admin UI
- [x] record user decision that `AttributeLabel` remains backend-editable while admin UI auto-fills it
- [x] change backend delete to soft-delete by setting `IsActive = false`
- [x] add backend authorization enforcement for admin CRUD/read endpoints

## Implementation Checklist

### Phase 1. Baseline Docs

- [x] create `symptom-config.introduction.md`
- [x] create `symptom-config.roadmap.md`
- [x] create `symptom-config.hallucination.md`
- [x] create `symptom-config.sourcecode.md`
- [x] create `symptom-config.useguide.md`
- [x] capture current code-verified API state before backend changes
- [x] capture current prod mirror data shape from `symtom.csv`

### Phase 2. Frontend Contract Alignment

- [x] document the recommended mobile read endpoint
- [x] document the admin CRUD endpoints
- [x] document request and response examples for important endpoints
- [x] document the group/key matrix frontend should enforce immediately
- [x] document `TimeScoreList` and `Category` meaning
- [x] document `VenomTypeId` as optional and non-blocking

### Phase 3. Frontend Matrix Validation Contract

- [x] decide that matrix validation belongs to frontend/admin UI for this scope
- [x] document allowed group/key combinations
- [x] document admin dependent-select behavior
- [x] document `AttributeLabel` auto-fill behavior
- [ ] frontend/admin implementation: when admin selects a known label/key, auto-detect related `GroupName`, `AttributeKey`, and canonical `AttributeLabel`
- [ ] frontend/admin implementation: block save when selected group/key does not match the matrix

### Phase 4. Backend Soft Delete And Authorization

- [x] update `DeleteSymptomConfigAsync(id)` to load the config and set `IsActive = false`
- [x] keep `UpdatedAt` refreshed during soft delete
- [x] keep grouped UI reads filtering `IsActive = true`
- [x] confirm `GET /api/symptom-configs` remains the all-data admin endpoint including inactive rows
- [x] confirm `GET /api/symptom-configs` remains flat
- [x] order `GET /api/symptom-configs` by `DisplayOrder`, then `Id`
- [x] confirm `GET /api/symptom-configs/filter?isActive=false` fetches inactive rows
- [x] add backend authorization enforcement for create, update, delete, and admin read endpoints
- [x] decide that `grouped` and `grouped-for-ui` stay public/auth-light while CRUD/admin reads become admin-only

### Phase 5. Tests

- [x] add delete test verifying `IsActive` becomes `false`
- [x] add delete test verifying the row still exists after delete
- [x] add grouped UI read test verifying inactive options are excluded
- [x] add all-data read test verifying inactive options can still be fetched by admin endpoint
- [x] add authorization test for create
- [x] add authorization test for update
- [x] add authorization test for delete
- [x] add authorization test for admin read endpoint if protected

### Phase 6. Docs Sync After Code Changes

- [x] update `symptom-config.introduction.md` after soft delete is code-enforced
- [x] update `symptom-config.introduction.md` after authorization is code-enforced
- [x] update `symptom-config.roadmap.md` checklist and changelog
- [x] update `symptom-config.hallucination.md` with user decisions
- [ ] update `symptom-config.sourcecode.md` diagrams after new validation code is added
- [x] update `symptom-config.sourcecode.md` diagrams after soft delete and auth code are added
- [x] update `symptom-config.useguide.md` after soft delete behavior is implemented
- [x] update `symptom-config.useguide.md` after authorization behavior is implemented

## Decided Validation Shape

User decision:

- frontend/admin UI handles the group/key matrix validation
- backend is not changed for matrix validation in this scope

Reason:

- avoid backend technical debt for a UI dropdown matrix
- keep current backend CRUD behavior unless a stronger server-side invariant is required later

## Suggested Admin UI Shape

Recommended admin UI:

- left side: grouped list/filter table
- right side: edit drawer or form
- `GroupName`: select with `BACKGROUND`, `LOCAL`, `CRITICAL`
- `AttributeKey`: dependent select filtered by selected `GroupName`
- `AttributeLabel`: auto-filled from key/label selection, but still editable
- `Name`: option label shown to users
- `Description`: optional helper text
- `IsCritical`: toggle
- `AlertMessage`: enabled when `IsCritical = true`
- `Category`: segmented control for `Core` or `Modifier`
- `TimeScoreList`: repeatable rows for `minMinutes`, `maxMinutes`, `score`
- `VenomTypeId`: optional select or nullable field; do not make it required
- `IsActive`: toggle for hiding options without deleting them

## Verification Strategy

Minimum verification for future backend hardening:

1. valid create requests succeed for all allowed group/key pairs
2. frontend/admin blocks invalid group/key pairs before submit
3. backend soft delete sets `IsActive = false`
4. nullable `VenomTypeId` remains accepted
5. non-null unknown `VenomTypeId` still returns not found
6. grouped UI endpoint returns only active records
7. grouped UI endpoint returns question groups sorted by `DisplayOrder`
8. all-data admin read can fetch inactive records
9. admin CRUD authorization is enforced before useguide marks it code-verified

## Change Log

### 2026-04-29 Baseline

- created baseline docs for `SymptomConfig`
- verified current CRUD and grouped UI endpoint from backend code
- verified current prod mirror data from `symtom.csv`
- documented target `GroupName` to `AttributeKey` matrix
- recorded that backend matrix validation is intentionally not planned for this scope

### 2026-04-29 User Decision Update

- closed hallucination risks with user decisions
- locked matrix validation to frontend/admin UI for this scope
- locked `AttributeLabel` as backend-editable with admin UI auto-fill
- locked backend delete behavior as soft delete through `IsActive = false`
- locked backend authorization enforcement as required before docs mark CRUD as admin-enforced
- documented that `GET /api/symptom-configs` currently returns inactive rows too
- changed `GET /api/symptom-configs` ordering from `DisplayOrder`, then `Name` to `DisplayOrder`, then `Id`

### 2026-04-29 Backend Hardening Update

- changed `DeleteSymptomConfigAsync` from hard delete to soft delete by setting `IsActive = false`
- added admin role authorization to create, update, delete, all-data list, filter, and get-by-id endpoints
- kept `grouped` and `grouped-for-ui` as public active-only read endpoints for frontend/mobile rendering
- added unit tests for soft delete, active-only grouped reads, all-data inactive reads, and admin authorization metadata
