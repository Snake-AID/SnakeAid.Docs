---
doc_role: planning
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

- module status: `Planning`
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
- [ ] Lock final field names
- [ ] Lock backward-compatibility strategy for legacy `price`
- [ ] Lock the source of truth for `priceAfterPlatformFee` when payout data does not yet exist
- [ ] Lock whether the change applies only to expert history or also to adjacent consultation-history APIs

## Implementation Checklist

### Phase 1. Contract Lock

- [ ] Decide final field names for gross/net price
- [ ] Decide whether legacy `price` stays temporarily
- [ ] Decide nullability rule for both new fields
- [ ] Decide exact source rules for scheduled and emergency items

### Phase 2. Response DTO

- [ ] Update `ExpertConsultationResponse`
- [ ] Add explicit gross/net fields
- [ ] Remove or deprecate ambiguous field depending on migration decision

### Phase 3. Service Mapping

- [ ] Update scheduled expert-history mapping
- [ ] Update emergency expert-history mapping
- [ ] Handle edge cases where transaction data is missing
- [ ] Keep pagination and filters unchanged

### Phase 4. Tests

- [ ] Replace single-price assertions with before/after assertions
- [ ] Add scheduled case coverage
- [ ] Add emergency case coverage
- [ ] Add missing-transaction edge case coverage
- [ ] Add compatibility test if legacy `price` is kept during migration

### Phase 5. Documentation Sync

- [ ] Update `introduction` after decisions are locked
- [ ] Update `sourcecode` diagrams to the final DTO and mapper flow
- [ ] Update `useguide` only when the contract is implemented
- [ ] Merge resolved risks into baseline docs and close them in `hallucination`

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
4. missing-transaction cases behave according to the locked nullability rule
5. docs and response examples match the code exactly

## Change Log

### 2026-04-23

- initialized planning docs for consultation information response cleanup
- documented the current mismatch between scheduled and emergency price semantics
- documented the current Flutter double-deduction risk
- narrowed the minimum safe scope to expert consultation history response
- opened unresolved design risks in `hallucination`
