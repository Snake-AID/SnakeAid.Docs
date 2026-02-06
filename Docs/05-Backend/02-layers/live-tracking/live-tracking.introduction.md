# Live Tracking Introduction

## Context
Live tracking is a cross-layer capability for emergency rescue:
- Patient should see rescuer movement in near real time.
- Rescuer should receive dispatch offers quickly and react in-app.
- Admin should observe active incidents and spatial trends.

The architecture direction is documented in:
- `live-tracking.architecture.md`

This file describes what this layer means in SnakeAid and what is currently implemented in codebase.

## Business Goal

1. Dispatch the right rescuer quickly using location.
2. Keep rescue session visible in real time.
3. Provide fallback delivery for critical dispatch events.
4. Keep enough history for auditing and map analytics.

## Current Maturity (as of 2026-02-06)

Implemented:
- Session-based dispatch for rescue requests.
- Radius progression (`10 -> 20 -> 30`) with timeout-driven auto expansion.
- SignalR-based rescuer request delivery and acceptance race handling.

Partially implemented:
- Rescuer location message exists in hub (`UpdateLocation`) but is not persisted or fan-out to patient/admin.

Not implemented yet:
- Tracking snapshot API for patient/admin map.
- Tracking history API.
- Redis NOW-store/presence/geo-index.
- FCM fallback for critical events.
- Session-based map group broadcast for patient/admin clients.

## Scope Boundary for This Layer

In-scope:
- Dispatch and live location pipeline contracts.
- Service boundaries: incident, session, mission, notification, timeout orchestration.
- Realtime transport patterns and data ownership guidance.

Out-of-scope:
- UI rendering specifics in Flutter.
- Payment, review, and non-tracking mission business logic.

## Why This Matters for Current Backend

Existing rescue-trigger flow already provides a strong dispatch base.
The live-tracking layer now needs to:
- convert dispatch-only realtime into full session tracking,
- separate transport concerns from state ownership,
- add durable/recoverable mechanisms (snapshot, history, fallback).

## Related Documents

- Architecture direction: `live-tracking.architecture.md`
- Current code state: `live-tracking.sourcecode.md`
- Completion roadmap: `live-tracking.plan.md`
- Implementation prompt template: `live-tracking.prompt.md`
- Integration contract and current API/hub behavior: `live-tracking.usageguide.md`

