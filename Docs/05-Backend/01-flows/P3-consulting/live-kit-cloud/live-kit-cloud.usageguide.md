---
doc_role: baseline
module: live-kit-cloud
kind: flow
status: active
last_updated: 2026-02-24
owners: [backend-team]
---

# LiveKit Cloud — Usage Guide

> [!NOTE]
> This document will be populated after implementation. The sections below define the planned structure.
> Per protocol: this file describes the external contract for consumers.

## Planned Content

Once implemented, this document will contain:

1. **Endpoints**
   - `POST /api/consultation/{id}/video-token` — Generate LiveKit access token
   - `POST /api/webhooks/livekit` — Receive LiveKit webhook events

2. **Request/Response Examples** — Full JSON for each endpoint

3. **Status Codes** — Success, validation errors, auth failures

4. **Error Catalog** — Structured error responses

5. **Auth Requirements** — Bearer token, consultation ownership validation

6. **Webhook Payload Format** — Event types and payload structure

---

_This file is intentionally minimal. Update after implementation per the Operation Lifecycle._
