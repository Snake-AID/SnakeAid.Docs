---
doc_role: operation
operation_id: 03-FEAT-emergency-consultation
generated_from: plan.md
status: draft
created_at: 2026-03-05
---

# Prompt: Implement Emergency Consultation Flow

## Requirements

Implement the API endpoints and Hub logic defined in the gap analysis of `plan.md`.

Specific tasks:

1. Implement `Api/Hubs/ExpertHub.cs`. Derive from standard SignalR `Hub`. Track connections in a ConcurrentDictionary for available experts. On connect/disconnect, update the `ExpertProfile.IsOnline` DB flag.
2. Implement `POST /v1/consultations/emergency` to create a `ConsultationPingRequest` and broadcast via the `IHubContext<ExpertHub>` to online experts matching the criteria.
3. Implement `POST /api/v1/consultations/emergency-requests/{requestId}/accept`. Create the `Consultation` record as `Ongoing`.
4. **Slot Paradox Requirement**: Within the accept transaction, query for overlapping `ExpertTimeSlot` rows. Change them to `Reserved` to block parallel bookings.
5. Implement `POST .../reject`.

## Constraints

- Do not mix room chat logic into `ExpertHub`. This hub is strictly for global presence and routing emergency pings to available experts.
- Handle state drift on SignalR disconnects gracefully.
