---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-05
owners: [backend-team]
---

# Consultation Module Introduction

## Domain Context

The Consultation module facilitates expert-user communication by allowing users to schedule and conduct live video and text consultations with verified snake experts. This flow is critical for users needing professional advice or urgent help regarding snake identification, bites, or general inquiries.

## Business Rules / Invariants

1. **Expert Availability**: Experts define their availability dynamically week-by-week. These configurations are ingested and converted into bookable `ExpertTimeSlot` records.
2. **Scheduled Consultations**: Users can book an available slot, reserving it while payment completes, and proceed to the actual consultation session.
3. **Emergency Consultations**: Users can ping online experts for an immediate consultation. If an expert accepts, overlapping regular `ExpertTimeSlot`s are automatically marked as `Reserved` or `Cancelled` to prevent double-booking (the Slot Paradox domain logic).
4. **Communication**: Real-time communication is strictly separated with presence and ping logic handled by `ExpertHub`, and active session chat / UI signaling handled by `ConsultationHub`. Video streams use the existing LiveKit Cloud infrastructure.

## Scope

- Expert profile settings (bio, fees).
- Expert directory search and profile viewing.
- Reserving available time slots.
- Emergency ping flows and matching.
- Real-time chat within an active session.

## Out of Scope

- Direct payment gateway integrations (handled by Payments module).
- WebRTC streaming internals (delegated to LiveKit Cloud).
