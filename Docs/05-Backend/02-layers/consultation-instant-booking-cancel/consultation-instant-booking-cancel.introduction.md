---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: introduction
status: decision-updated-implementation-paused
last_updated: 2026-05-05
owners: [backend-team]
verification_status: decision-recorded
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

Target history behavior:

- accepted instant/emergency requests appear in member/expert consultation history
- expert-rejected instant/emergency requests appear in member/expert consultation history as `kind = instant`
- expired instant/emergency requests appear in member/expert consultation history as `kind = instant`

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

Usecase boundary decision:

- History is its own usecase and may use a typed union response for the mixed timeline.
- Expert absent remains a separate usecase and continues to use `MyConsultationResponse`.
- `MyConsultationResponse` and `ExpertConsultationResponse` must not be converted into union DTOs.
- Do not use `JsonIgnore(WhenWritingNull)` to hide non-applicable history fields.
- Do not expose history contracts as `object` or `dynamic`.
- Runtime serialization of derived history DTOs is acceptable; the remaining tradeoff is Swagger/OpenAPI schema quality.

Target DTO location:

- `SnakeAid.Core/Responses/Consultation/History/`

Target DTO names:

- `MyConsultationHistoryUnionResponse`
- `MyConsultationHistoryResponse`
- `MyInstantConsultationRequestHistoryResponse`
- `ExpertConsultationHistoryUnionResponse`
- `ExpertConsultationHistoryResponse`
- `ExpertInstantConsultationRequestHistoryResponse`

Locked `kind = instant` behavior:

- uses `instantRequestId`
- uses `requestStatus`
- uses `requestedAt` and `respondedAt`
- keeps member/expert actor fields flat
- omits `consultationId`, `roomId`, `startTime`, `endTime`, `rescueMissionId`, `expiresAt`, and price fields
- currently covers `DeclinedByExpert` and `Expired`
- does not cover `RescuerCancelled` until a production flow sets that status

## Status Filter Decision

Status filter behavior is selected conservatively.

When `status` is not supplied:

- member/expert emergency history includes real consultation rows and terminal instant request rows

When `status` is supplied:

- `status` filters only `kind = consultation` rows by `ConsultationStatus`
- `kind = instant` rows are not returned by the `status` filter
- request-level filtering by `requestStatus` is not implemented in this change

This avoids mapping `ConsultationStatus` values to `ConsultationPingStatus` values before a broader admin/filter contract exists.

## Delivered Artifacts

- `consultation-instant-booking-cancel.introduction.md`
- `consultation-instant-booking-cancel.roadmap.md`
- `consultation-instant-booking-cancel.hallucination.md`
- `consultation-instant-booking-cancel.sourcecode.md`
- `consultation-instant-booking-cancel.useguide.md`

## Verification

- Pending after the DTO naming/usecase-boundary decision is applied in code.
