---
doc_role: implementation
module: expert-availability
kind: flow
doc_type: introduction
status: planning
last_updated: 2026-04-21
owners: [backend-team]
verification_status: current-state-code-verified-plan-not-yet-implemented
---

# Expert Availability Introduction

## Goal

This module plans the expert online or offline availability trigger for the Expert App.

Business goal:

- the Expert App needs a clear online or offline button similar to a ride-hailing driver app
- when an expert turns online, the backend should mark the expert as available for realtime emergency consultation discovery
- when an expert turns offline, the backend should stop treating that expert as online
- the implementation should reuse the existing availability pattern already used in the `Rescuer` flow as much as possible

## Resume Summary

If this work is resumed later without prior chat memory, the current code-verified state is:

1. `ExpertProfile` already persists `IsOnline`.
2. `ExpertProfileResponse` and `ExpertMyProfileResponse` already expose `IsOnline`.
3. `ExpertHub.JoinAsExpert()` already marks `ExpertProfile.IsOnline = true`.
4. `ExpertHub.OnDisconnectedAsync(...)` already marks `ExpertProfile.IsOnline = false`.
5. there is currently no dedicated `ExpertOnlineStatusService`.
6. there is currently no explicit expert offline trigger equivalent to a deliberate app-side toggle action.
7. there is currently no dedicated HTTP endpoint for expert availability toggling.

## Code-Verified Current State

### Expert persistence

Current verified facts:

- `SnakeAid.Core/Domains/ExpertProfile.cs` already has:
  - `AccountId`
  - `Biography`
  - `IsOnline`
  - fee and rating fields
- `IsOnline` defaults to `false`

### Current public read surface

Current expert-facing and member-facing reads already expose expert online state:

- `GET /api/experts`
- `GET /api/experts/{id}`
- `GET /api/experts/me/profile`

Current response contracts already include `IsOnline` in:

- `ExpertProfileResponse`
- `ExpertMyProfileResponse`

This means mobile can already read expert online state from both self-profile and expert directory surfaces.

### Current realtime availability behavior

Current verified realtime route:

- SignalR hub: `/hubs/expert`

Current verified expert presence behavior:

- `ExpertHub.JoinAsExpert()`:
  - validates `Expert` role
  - adds the current connection into `SignalRExpertEmergencyNotificationService.ConnectedExperts`
  - updates `ExpertProfile.IsOnline = true`
  - broadcasts `ExpertPresenceChanged` to `ConsultationMembers`
- `ExpertHub.OnDisconnectedAsync(...)`:
  - removes the expert connection from `ConnectedExperts`
  - updates `ExpertProfile.IsOnline = false`
  - broadcasts `ExpertPresenceChanged` to `ConsultationMembers`

### Current implementation gap

The backend already has a connection-driven expert online flag, but the code is still incomplete for the requested app behavior because:

- online-state write logic is embedded inside `ExpertHub`
- there is no dedicated expert availability service like `RescuerOnlineStatusService`
- there is no explicit `LeaveAsExpert()` style trigger for an intentional offline action from the app while the socket is still connected
- docs for this availability contract do not yet exist as a resumable module baseline

## Planned Implementation Direction

The planned direction for this module is:

- create `IExpertOnlineStatusService`
- create `ExpertOnlineStatusService`
- move expert online-state writes out of `ExpertHub` private helper logic and into the service
- keep the existing `JoinAsExpert()` online flow
- add an explicit expert offline trigger for the app button
- keep member-facing presence broadcasts aligned with the final expert status change
- sync the docs set so frontend and backend can resume work without relying on chat memory

## Scope Boundary

In scope:

- expert online or offline persistence
- expert realtime presence trigger contract
- app-facing availability toggle flow
- diagrams, roadmap, useguide, and risk tracking for this module

Out of scope:

- redesigning expert emergency consultation matching rules
- adding availability scheduling by time blocks
- changing member expert-directory filters beyond existing `IsOnline` usage
- redesigning the full expert consultation hub

## Expected Impacted Areas

- `SnakeAid.Api/Hubs/ExpertHub.cs`
- `SnakeAid.Api/Services/SignalRExpertEmergencyNotificationService.cs`
- `SnakeAid.Service/Interfaces/*`
- `SnakeAid.Service/Implements/*`
- integration or unit tests for expert availability
- docs under `SnakeAid.Docs/Docs/05-Backend/02-layers/expert-availability`
