---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: introduction
status: current
last_updated: 2026-05-05
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Introduction

## Goal

This documentation pack tracks how instant/emergency consultation request terminal events should appear in member and expert consultation history.

The scope is:

- expert rejects an instant/emergency request
- instant/emergency request expires before expert accepts
- request-level rows appear in:
  - `GET /api/users/me/consultations`
  - `GET /api/experts/me/consultations`

Scheduled booking cancellation is out of scope except as a comparison point for real `Consultation` rows.

## Current Code-Verified State

Current instant/emergency flow:

1. member creates an instant/emergency request as `ConsultationPingRequest`
2. request starts before a `Consultation` row exists
3. payment can move the request into `PendingExpertResponse`
4. expert accept creates a `Consultation`
5. expert accept sets `ConsultationPingRequest.Status = AcceptedByExpert`
6. expert accept sets `ConsultationPingRequest.ConsultationId`
7. expert reject sets `ConsultationPingRequest.Status = DeclinedByExpert`
8. expert reject does not create a `Consultation`
9. expiration sets `ConsultationPingRequest.Status = Expired`
10. expiration does not create a `Consultation`

Current history behavior:

- accepted instant/emergency requests appear in member/expert consultation history
- expert-rejected instant/emergency requests do not appear in member/expert consultation history
- expired instant/emergency requests do not appear in member/expert consultation history

## Root Cause Summary

The current history endpoints are consultation-session history endpoints.

They include instant/emergency rows only when a `ConsultationPingRequest` has:

- `ConsultationId.HasValue`
- `Status == AcceptedByExpert`

Rejected or expired instant/emergency requests remain request records only:

- `ConsultationId = null`
- `Status = DeclinedByExpert` or `Expired`

## Locked Decision

The selected direction is a union history contract.

Each history item is one of:

- `kind = consultation`: a real `Consultation` row
- `kind = instant`: a request-level `ConsultationPingRequest` row that has no `Consultation`

`kind = instant` is a separate DTO, not a consultation DTO with nullable/fake fields.

Locked `kind = instant` behavior:

- uses `instantRequestId`
- uses `requestStatus`
- uses `requestedAt` and `respondedAt`
- keeps member/expert actor fields flat
- omits `consultationId`, `roomId`, `startTime`, `endTime`, `rescueMissionId`, `expiresAt`, and price fields
- currently covers `DeclinedByExpert` and `Expired`
- does not cover `RescuerCancelled` until a production flow sets that status

## Remaining Open Decision

Only status filter mapping remains open.

The unresolved question is how `status` filters should interact with `kind = instant`, especially when admin history/filter contracts are considered.

## Delivered Artifacts

- `consultation-instant-booking-cancel.introduction.md`
- `consultation-instant-booking-cancel.roadmap.md`
- `consultation-instant-booking-cancel.hallucination.md`
- `consultation-instant-booking-cancel.sourcecode.md`
- `consultation-instant-booking-cancel.useguide.md`
