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

## Implementation Direction Under Consideration

The preferred direction under analysis is to include expert-rejected instant/emergency requests in the existing member/expert history endpoints as request-level rows.

This requires an explicit response contract because rejected rows do not have a real consultation id or room id.

## Delivered Artifacts

- `consultation-instant-booking-cancel.introduction.md`
- `consultation-instant-booking-cancel.roadmap.md`
- `consultation-instant-booking-cancel.hallucination.md`
- `consultation-instant-booking-cancel.sourcecode.md`
- `consultation-instant-booking-cancel.useguide.md`
