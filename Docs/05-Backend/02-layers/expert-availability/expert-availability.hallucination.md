# Expert Availability Hallucination

## Status

Current status:

- closed

## Purpose

This file records ambiguity that should not be silently invented and keeps decision records when a risk is closed.

Main docs affected by these decisions:

- `expert-availability.introduction.md`
- `expert-availability.roadmap.md`
- `expert-availability.sourcecode.md`
- `expert-availability.useguide.md`

## Current Direction Summary

Current direction is already implemented:

- reuse the existing expert SignalR route `/hubs/expert`
- keep `ExpertProfile.IsOnline` as the source of persisted truth
- align the implementation structure with `RescuerOnlineStatusService`
- add an explicit app-driven offline trigger instead of relying only on disconnect

There is no open user-decision risk for the current baseline.

## Risk 1. Trigger Surface For Expert App

### Context

The current code already marks experts online on `JoinAsExpert()` and offline on disconnect, but the requested app behavior needs a deliberate On or Off availability action.

Potential options considered:

- option A: rely only on socket connect or disconnect as the app button behavior
- option B: add explicit trigger methods on `ExpertHub` while keeping the same hub route
- option C: add dedicated HTTP toggle endpoints

### Decision

- chose option B

### Reason

- it stays closest to the current expert implementation
- it stays structurally similar to the rescuer realtime pattern
- it avoids introducing a second transport style just for one realtime availability action
- it gives the app a deliberate offline trigger without depending on network teardown timing

### Decision Record

- original options:
  - option A: rely only on socket connect or disconnect
  - option B: add explicit trigger methods on `ExpertHub`
  - option C: add dedicated HTTP toggle endpoints
- chosen option:
  - option B

## Closure Summary

This file stays closed because the chosen trigger surface is now implemented and reflected in:

- `expert-availability.introduction.md`
- `expert-availability.roadmap.md`
- `expert-availability.sourcecode.md`
- `expert-availability.useguide.md`

## Promotion Rule

Only promote a risk decision into the main docs after:

- one option is chosen
- the choice is reflected in roadmap and useguide
- the resulting contract is specific enough to implement and test
