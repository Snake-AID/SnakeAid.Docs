---
doc_role: baseline
module: consultation
kind: flow
status: active
last_updated: 2026-03-06
owners: [backend-team]
---

# Consultation Module Introduction

## Domain Context

The Consultation module facilitates expert-user communication by allowing users to schedule and conduct live video and text consultations with verified snake experts. This flow is critical for users needing professional advice or urgent help regarding snake identification, bites, or general inquiries.

## Business Rules / Invariants

1. **Expert Availability**: Experts define their availability dynamically week-by-week. These configurations are ingested and converted into bookable `ExpertTimeSlot` records.
2. **Scheduled Consultations**: Users can book an available slot, reserving it while payment completes, and proceed to the actual consultation session.
3. **Emergency Consultations**: Users can ping a selected online expert for an immediate consultation. If an expert accepts, overlapping regular `ExpertTimeSlot`s are automatically marked as `Reserved` to prevent double-booking (the Slot Paradox domain logic).
4. **Communication**: Real-time communication is strictly separated with presence and emergency routing handled by `ExpertHub`, and active session chat / UI signaling handled by `ConsultationHub`. Video streams use the existing LiveKit Cloud infrastructure.
5. **Emergency Request Room**: After creating an emergency request, the requester joins a request-specific SignalR group (`consultation:emergency:request:{requestId}`) to receive status updates (`Accepted` / `Rejected`) in real time.

## Scope

- Expert profile settings (bio, fees).
- Expert directory search and profile viewing.
- Reserving available time slots.
- Emergency ping flows for user-selected expert and request-room status updates.
- Real-time chat within an active session.

## Out of Scope

- Direct payment gateway integrations (handled by Payments module).
- WebRTC streaming internals (delegated to LiveKit Cloud).
