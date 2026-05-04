---
doc_role: implementation
module: consultation-instant-booking-cancel
kind: flow
doc_type: introduction
status: current
last_updated: 2026-05-04
owners: [backend-team]
verification_status: code-investigated
---

# Consultation Instant Booking Cancel Introduction

## Goal

This documentation pack tracks the instant/emergency consultation cancellation and expert-rejection history behavior.

The current business question is scoped only to instant/emergency consultations:

- when an expert rejects or cancels an instant/emergency request
- should that request appear in `GET /api/users/me/consultations`
- should that request appear in `GET /api/experts/me/consultations`

Scheduled booking cancellation is out of scope for this pack except as a known working comparison point.

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

Current history behavior:

- accepted instant/emergency requests appear in member/expert consultation history
- expert-rejected instant/emergency requests do not appear in member/expert consultation history

## Root Cause Summary

The current history endpoints are consultation-session history endpoints.

They include instant/emergency rows only when a `ConsultationPingRequest` has:

- `ConsultationId.HasValue`
- `Status == AcceptedByExpert`

Expert-rejected instant/emergency requests remain request records only:

- `ConsultationId = null`
- `Status = DeclinedByExpert`

## Decision Space

There are three candidate directions for showing expert-rejected instant/emergency requests in history.

### Approach 1: Split The Contract And Force Mobile To Build Two History Screens

The history contract explicitly supports both real `Consultation` rows and request-level `ConsultationPingRequest` rows.

Expected behavior:

- accepted scheduled and emergency consultations remain real consultation rows
- expert-rejected instant/emergency requests appear as request-level rows
- request-level rows do not fabricate `Consultation` data
- mobile must build two history screens or sections:
  - consultation history
  - instant request history

### Approach 2: Keep The Old Contract And Force `ConsultationPingRequest` Into Consultation History

The backend fetches rejected `ConsultationPingRequest` records and maps them into the existing history response shape without creating `Consultation` rows.

Expected behavior:

- database stays cleaner than the Fake Consultation approach
- rejected request rows are mixed into the existing `/me/consultations` response
- the response contract becomes ambiguous unless special values or extra fields are introduced
- mobile must avoid treating request-only rows as real consultations

### Approach 3: Keep The Old Contract By Creating A Fake `Consultation`

The backend creates a Fake cancelled emergency `Consultation` only to satisfy the existing non-null `consultationId` contract.

Expected behavior:

- history rows keep a real `consultationId`
- mobile sees the rejected request as a cancelled consultation row
- the database contains a Fake `Consultation` that does not represent a real session
- consultation-scoped flows such as room, chat, review, payment, cleanup, and reporting need guards to avoid treating the Fake `Consultation` as a real session

## Current Recommendation Status

No implementation direction is locked in this pack.

The next decision is to choose one of the three approaches above before changing code or frontend/mobile contracts.

## Delivered Artifacts

- `consultation-instant-booking-cancel.introduction.md`
- `consultation-instant-booking-cancel.roadmap.md`
- `consultation-instant-booking-cancel.hallucination.md`
- `consultation-instant-booking-cancel.sourcecode.md`
- `consultation-instant-booking-cancel.useguide.md`
