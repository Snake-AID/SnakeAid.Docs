---
doc_role: baseline
module: live-kit-cloud
kind: flow
status: active
last_updated: 2026-02-24
owners: [backend-team]
---

# LiveKit Cloud — Source Code Map

> [!NOTE]
> This document will be populated after implementation. The sections below define the planned structure.
> Per protocol: this file MUST only describe what exists in code — no TODOs, no future plans.

## Planned API Surface

Once implemented, this document will contain:

1. **Service Interface** — `ILiveKitService` public methods and signatures
2. **DTOs** — `LiveKitTokenRequest`, `LiveKitTokenResponse`, `WebhookEvent`, `RoomInfo`
3. **Controller Endpoints** — Token generation endpoint, webhook receiver
4. **Configuration** — `LiveKitOptions` settings class
5. **Entity Relationships** — Consultation ↔ Room mapping
6. **Cross-cutting** — Error handling, logging, auth requirements

---

_This file is intentionally minimal. Update after implementation per the Operation Lifecycle._
