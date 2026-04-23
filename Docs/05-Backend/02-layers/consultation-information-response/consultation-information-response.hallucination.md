# Consultation Information Response Hallucination

## Status

Current status:

- open

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

### Decision Record

- pending

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

### Decision Record

- pending

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

### Decision Record

- pending

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

### Decision Record

- pending

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

### Decision Record

- pending
