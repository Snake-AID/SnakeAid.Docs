---
doc_role: implementation
module: consultation-information-response
kind: flow
doc_type: roadmap
status: active
last_updated: 2026-04-23
owners: [backend-team]
verification_status: code-verified
---

# Consultation Information Response Roadmap

## Current Status Snapshot

- module status: `Implemented`
- backend issue confirmed: `Yes`
- endpoint currently affected: `GET /api/experts/me/consultations`
- response ambiguity confirmed in code: `Yes`
- doc baseline initialized: `Yes`

## Current Truth To Resume From

Current verified state:

- `ExpertConsultationResponse` exposes only `Price`
- scheduled expert-history mapping uses `ConsultationBooking.Price`
- emergency expert-history mapping uses `ExpertPayout.Amount`
- consultation settlement already supports platform fee, default `0.20`
- Flutter currently hardcodes platform-fee deduction for this area

## Target Outcome

After this work is complete:

1. expert consultation history returns explicit gross and net fields
2. Flutter can render price directly without local fee calculation
3. scheduled and emergency items no longer overload one field with different meanings
4. tests protect against future regression in money semantics
5. docs clearly separate active contract from planned migration behavior

## Recommended Scope

Recommended minimum scope:

- update expert history response contract only
- add two explicit money fields
- keep mobile migration straightforward
- update docs and tests in the same task

Recommended follow-up scope after that:

- review whether member/admin consultation-history responses should align to the same naming pattern

## Provisional Decisions

- [x] Treat this as a response-contract cleanup, not a Flutter-only workaround
- [x] Baseline docs must reflect current code before implementation starts
- [x] Use docs to record actual current money semantics for both scheduled and emergency
- [x] Lock final field names as `grossPrice` and `netPrice`
- [x] Lock backward-compatibility strategy: remove legacy `price` in the same release
- [x] Lock `netPrice` source of truth as persisted `ExpertPayout`, otherwise `null`
- [x] Lock scope to `GET /api/experts/me/consultations` only

## Implementation Checklist

### Phase 1. Contract Lock

- [x] Decide final field names for gross/net price
- [x] Decide whether legacy `price` stays temporarily
- [x] Decide nullability rule for both new fields
- [x] Decide exact source rules for scheduled and emergency items

### Phase 2. Response DTO

- [x] Update `ExpertConsultationResponse`
- [x] Add explicit `grossPrice` and `netPrice`
- [x] Remove ambiguous `price`

### Phase 3. Service Mapping

- [x] Update scheduled expert-history mapping
- [x] Update emergency expert-history mapping
- [x] Handle edge cases where transaction data is missing
- [x] Keep pagination and filters unchanged

### Phase 4. Tests

- [x] Replace single-price assertions with `grossPrice/netPrice` assertions
- [x] Add scheduled case coverage
- [x] Add emergency case coverage
- [x] Add missing-transaction edge case coverage

### Phase 5. Documentation Sync

- [x] Update `introduction` after decisions are locked
- [x] Update `sourcecode` diagrams to the final DTO and mapper flow
- [x] Update `useguide` when the contract is implemented
- [x] Merge resolved risks into baseline docs and close them in `hallucination`

## Suggested Execution Order

1. lock the unresolved business decisions in `hallucination`
2. change DTO + mapping in backend
3. update integration tests
4. update `useguide` to the active implemented contract
5. close resolved risks and record decision history

## Verification Strategy

Implementation should be considered done only when all of these are verified:

1. scheduled expert-history items return deterministic gross/net values
2. emergency expert-history items return deterministic gross/net values
3. emergency items no longer require Flutter-side percentage deduction
4. missing-payout cases return `netPrice = null` according to the locked rule
5. docs and response examples match the code exactly

## Change Log

### 2026-04-23

- initialized planning docs for consultation information response cleanup
- documented the current mismatch between scheduled and emergency price semantics
- documented the current Flutter double-deduction risk
- narrowed the minimum safe scope to expert consultation history response
- opened unresolved design risks in `hallucination`

### 2026-04-23 Decision Update

- locked new field names as `grossPrice` and `netPrice`
- locked same-release removal of legacy `price`
- locked `netPrice` to persisted payout truth only
- locked scheduled `netPrice = null` until actual payout exists
- locked scope to expert history only

### 2026-04-23 Implementation Update

- implemented `grossPrice` and `netPrice` for `GET /api/experts/me/consultations`
- removed legacy `price` from `ExpertConsultationResponse`
- mapped scheduled `grossPrice` from booking and `netPrice` from persisted payout when present
- mapped emergency `grossPrice` from consultation payment and `netPrice` from persisted payout
- left member/admin consultation-history contracts unchanged
- added expert-history integration tests for scheduled and emergency price semantics
