# Consultation Information Response Hallucination

## Status

Current status:

- partially closed

## Purpose

This file records decisions that should not be silently invented from incomplete code context.

When a risk is resolved:

1. keep the original option list
2. append the chosen decision as a decision record
3. merge the decision into:
   - `consultation-information-response.introduction.md`
   - `consultation-information-response.roadmap.md`
   - `consultation-information-response.sourcecode.md`
   - `consultation-information-response.useguide.md`
4. then mark the risk closed

## Risk 1. Final Field Names

### Why this is open

The business request says `price before` and `price after`, but the public API should avoid vague names if the payload survives long-term.

### Options

- Option A: `priceBefore` and `priceAfter`
- Option B: `priceBeforePlatformFee` and `priceAfterPlatformFee`
- Option C: `grossPrice` and `netPrice`

### Recommendation

- recommend Option B

### Reason

- explicit meaning
- easier for mobile and backend to read later
- avoids confusion with other before/after concepts such as tax, discount, or settlement timing

### Decision

- closed

### Decision Record

- chosen option: Option C
- chosen fields:
  - `grossPrice`
  - `netPrice`
- decision date: `2026-04-23`
- rationale from requester:
  - prefers concise money naming over fee-specific wording

## Risk 2. Backward Compatibility For Legacy `price`

### Why this is open

Removing `price` immediately is cleaner, but keeping it temporarily may reduce rollout risk if mobile and backend cannot release simultaneously.

### Options

- Option A: remove `price` in the same release and treat this as a breaking change
- Option B: keep `price` temporarily, mark it deprecated, and define exactly which of the new fields it mirrors
- Option C: keep `price` only for one actor path and remove it elsewhere later

### Recommendation

- recommend Option B

### Reason

- safer rollout for mobile coordination
- simpler to deploy backend first without breaking older app builds
- still allows a later cleanup once Flutter is migrated

### Decision

- closed

### Decision Record

- chosen option: Option A
- chosen behavior:
  - remove legacy `price` in the same release
  - treat the response change as a breaking API-contract change
- decision date: `2026-04-23`
- rationale from requester:
  - wants the contract cleaned in one step instead of carrying an ambiguous legacy field

## Risk 3. Source Of Truth For `priceAfterPlatformFee`

### Why this is open

The codebase already has a configurable consultation platform fee. Historical consultations may not always have an `ExpertPayout` transaction yet, and the configured fee can change over time.

That creates a data-truth question:

- should `priceAfterPlatformFee` come from persisted payout truth only
- or should backend calculate it when payout truth is absent

### Options

- Option A: prefer persisted `ExpertPayout`; if absent, calculate from gross using current configured fee
- Option B: prefer persisted `ExpertPayout`; if absent, return `null`
- Option C: always calculate from gross using current configured fee and ignore payout as display source

### Recommendation

- recommend Option B

### Reason

- safest against historical drift when fee configuration changes later
- preserves the difference between settled truth and projected value
- avoids silently showing a net number that may never match the eventual payout

### Decision

- closed

### Decision Record

- chosen option: Option B
- chosen behavior:
  - prefer persisted `ExpertPayout`
  - if payout does not exist yet, `netPrice = null`
- decision date: `2026-04-23`
- rationale:
  - keep net value tied to persisted payout truth only

## Risk 4. Scheduled `priceAfterPlatformFee` Before Settlement

### Why this is open

Scheduled consultations currently expose gross booking price in expert history. If payout has not happened yet, the expected net display rule is not fully locked by code today.

### Options

- Option A: return `null` until actual payout exists
- Option B: calculate projected net from scheduled gross and current configured fee
- Option C: return the same value as `priceBeforePlatformFee` until settlement exists

### Recommendation

- recommend Option A

### Reason

- it keeps `after` as actual realized payout semantics
- it avoids projected numbers being mistaken as settled truth
- it is safer if platform fee percent becomes configurable per period later

### Decision

- closed

### Decision Record

- chosen option: Option A
- chosen behavior:
  - for scheduled consultations, `netPrice = null` until actual payout exists
- decision date: `2026-04-23`
- rationale:
  - do not expose projected net payout as if it were settled truth

## Risk 5. Scope Boundary

### Why this is open

The urgent bug is on expert history, but nearby APIs already expose consultation prices and could become semantically inconsistent if only one contract is cleaned up.

### Options

- Option A: change only `GET /api/experts/me/consultations` now
- Option B: change expert history plus user history in the same release
- Option C: change expert history, user history, and admin consultation history together

### Recommendation

- recommend Option A now, then schedule a consistency review

### Reason

- fastest path to remove the active Flutter bug
- smallest breaking surface
- easier to test and coordinate

### Clarification

Why this needs a clearer decision before coding:

- `GET /api/experts/me/consultations` is the confirmed broken integration path today
- `GET /api/users/me/consultations` already uses a different money source for emergency items:
  - it prefers `ConsultationPayment` by `ConsultationPingRequest.Id`
- `GET /api/admin/consultations` and `GET /api/admin/consultations/{consultationId}` already have their own documented money semantics:
  - scheduled price from `ConsultationBooking.Price`
  - emergency price prefers `ConsultationPayment`, then falls back to `ExpertPayout`

So Risk 5 is not just about how much work to do.

It changes the public meaning of money fields across actor-specific APIs:

- expert history is expert-facing money
- member history is customer-facing money
- admin history is operational/audit-facing money

If all three are changed together:

- naming consistency improves
- but rollout risk becomes larger
- and each actor path may still need different source rules even with the same field names

If only expert history is changed now:

- the urgent Flutter bug is fixed faster
- but docs must explicitly note that adjacent APIs may still keep older price semantics until a later alignment pass

### Current Hold Status

- open
- blocked on scope choice from requester

### Decision Record

- pending
