---
doc_role: implementation
module: expert-availability
kind: flow
doc_type: roadmap
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Expert Availability Roadmap

## Current Status Snapshot

- module status: `Implemented`
- `ExpertProfile.IsOnline` persistence: `Available`
- expert online trigger on hub connect: `Available`
- expert offline trigger on hub disconnect: `Available`
- dedicated expert online-status service: `Available`
- explicit app-driven offline trigger: `Available`
- module docs baseline: `Synced to implementation`

## Current Truth To Resume From

This roadmap is written so work can resume from zero memory.

Current verified state:

- `ExpertHub.JoinAsExpert()` already sets expert online
- `ExpertHub.OnDisconnectedAsync(...)` already sets expert offline
- `SignalRExpertEmergencyNotificationService.ConnectedExperts` tracks online expert connections
- `ExpertService` already exposes `IsOnline` in expert directory responses
- `MyProfileService` already exposes `IsOnline` in expert self-profile responses
- `ExpertOnlineStatusService` now owns persisted expert online-state writes
- `ExpertHub.LeaveAsExpert()` now exists as the explicit offline trigger

## Implemented Outcome

After this module is complete:

1. the Expert App can intentionally switch expert availability on or off
2. expert online-state writes are owned by a reusable service instead of a private hub helper
3. realtime expert presence broadcasts stay aligned with the persisted `IsOnline` state
4. the docs set cleanly reflects the final backend contract and can be resumed later

## Locked Functional Direction

- [x] reuse the existing expert SignalR route `/hubs/expert`
- [x] reuse the current `ExpertProfile.IsOnline` field instead of introducing a new availability table
- [x] align implementation direction with the rescuer online-status service pattern
- [x] keep expert presence broadcasts through `ExpertHub`
- [x] document both realtime trigger contract and HTTP read surfaces that expose `IsOnline`
- [x] prefer an explicit app-driven offline trigger instead of relying only on transport disconnect side effects

## Implementation Checklist

### Phase 1. Baseline Docs

- [x] create `expert-availability.introduction.md`
- [x] create `expert-availability.roadmap.md`
- [x] create `expert-availability.hallucination.md`
- [x] create `expert-availability.sourcecode.md`
- [x] create `expert-availability.useguide.md`
- [x] capture current code-verified state before feature edits

### Phase 2. Service Extraction

- [x] add `IExpertOnlineStatusService`
- [x] add `ExpertOnlineStatusService`
- [x] move expert online-state persistence out of `ExpertHub` helper logic
- [x] keep update logic idempotent when requested state is already current

### Phase 3. Expert App Trigger

- [x] keep `JoinAsExpert()` as the online trigger
- [x] add explicit expert offline trigger for deliberate app toggle
- [x] update `ConnectedExperts` tracking to stay consistent with explicit offline actions
- [x] keep `ExpertPresenceChanged` broadcast aligned with the new trigger flow

### Phase 4. Verification

- [x] cover online transition
- [x] cover offline transition
- [x] cover duplicate toggle behavior
- [ ] verify persisted `IsOnline` becomes visible through existing expert read APIs

### Phase 5. Docs Sync

- [x] update all baseline docs from planning state to implemented state
- [x] record final trigger contract for mobile integration
- [x] close or archive any remaining hallucination items

## Verification Strategy

Minimum verification for the final implementation:

1. expert starts offline in persistence
2. expert triggers online flow
3. `ExpertProfile.IsOnline` becomes `true`
4. member-facing presence event is emitted
5. expert triggers offline flow intentionally
6. `ExpertProfile.IsOnline` becomes `false`
7. expert directory and self-profile surfaces reflect the updated value

## Change Log

### 2026-04-21 Baseline

- created the baseline roadmap for expert availability
- documented that the current expert online flag is already driven by `ExpertHub`
- locked the implementation direction to extract a reusable expert online-status service
- locked the need for an explicit app-driven offline trigger

### 2026-04-21 Implementation Update

- added `IExpertOnlineStatusService`
- added `ExpertOnlineStatusService`
- updated `ExpertHub` to use the service instead of a private online-flag helper
- added `LeaveAsExpert()` as the explicit offline trigger for the Expert App button
- added service-level tests for online, offline, and idempotent offline behavior
