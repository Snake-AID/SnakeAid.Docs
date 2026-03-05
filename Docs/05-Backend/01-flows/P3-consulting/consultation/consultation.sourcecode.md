---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-05
owners: [backend-team]
---

# Consultation Module Source Code

## Entities and Schema

- `Consultation`: The core session record (`Id`, `CallerId`, `CalleeId`, `StartTime`, `EndTime`, `Status`).
- `ConsultationBooking`: Connects a scheduled booking to a user and time slot.
- `ExpertProfile`: Expert metadata (`IsOnline`, `ConsultationFee`, `Rating`).
- `ExpertTimeSlot`: Discrete time block for booking (`StartTime`, `EndTime`, `Status`, `Version` for optimistic concurrency).

## Public API Surface (Endpoints)

- _Currently uninitialized. To be implemented in subsequent operations._

## Hubs

- _Currently uninitialized. To be implemented in subsequent operations._

## Cross-Cutting Concerns

- **Concurrency**: Optimistic concurrency via `Version` field on `ExpertTimeSlot` to prevent double-booking.
- **Authentication**: Custom roles required for Expert-facing vs User-facing endpoints. User contexts resolved via Claims.
