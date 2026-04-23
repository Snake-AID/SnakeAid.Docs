---
doc_role: implementation
module: consultation-information-response
kind: flow
doc_type: introduction
status: active
last_updated: 2026-04-23
owners: [backend-team]
verification_status: code-verified
---

# Consultation Information Response Introduction

## Goal

This module documents the implemented response-contract cleanup for expert consultation history where `price` previously meant different things depending on consultation type.

Implemented target:

- stop forcing Flutter to calculate platform fee locally
- expose both gross and net price from backend
- remove the current ambiguity where one response returns pre-fee price and another returns post-fee price

Implemented contract direction:

- remove the single ambiguous `price` field
- replace it with:
  - `grossPrice`
  - `netPrice`

## Resume Summary

If this work must resume later with no prior chat memory, the current code-verified situation is:

1. expert consultation history is exposed by `GET /api/experts/me/consultations`
2. `ExpertConsultationResponse` currently has only one price field: `Price`
3. scheduled expert-history items currently map `Price = ConsultationBooking.Price`
4. emergency expert-history items currently map `Price = TransactionType.ExpertPayout.Amount` by `Consultation.Id`
5. the payment module already uses a configurable consultation platform fee, with default `20%`
6. the current contract can cause Flutter to subtract the platform fee twice for emergency items if the client still hardcodes the fee calculation

## Code-Verified Current Backend State

### Active Expert History Surface

Current verified endpoint:

- `GET /api/experts/me/consultations`

Current verified code locations:

- `SnakeAid.Api/Controllers/ExpertController.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`

### Current Scheduled Price Meaning

For expert history scheduled items:

- source = `ConsultationBooking.Price`
- current response field = `price`
- semantic meaning = gross consultation amount before platform fee

This is consistent with the current mapping in `ConsultationService.GetExpertConsultationsAsync(...)`.

### Current Emergency Price Meaning

For expert history emergency items:

- source = latest `Transaction` with `TransactionType = ExpertPayout`
- lookup key = `Consultation.Id`
- current response field = `price`
- semantic meaning = expert net payout after platform fee

This means emergency and scheduled items currently use different money semantics while sharing the same response field name.

### Why Flutter Is Getting Conflicting Values

Current client issue described by the business request is aligned with backend behavior:

- scheduled item `price` is currently gross
- emergency item `price` is currently net
- Flutter still hardcodes platform-fee deduction
- emergency UI therefore risks deducting the fee twice

## Problem Statement

The problem is not only naming.

The deeper issue is that one field currently represents two different business meanings:

- scheduled `price` = amount before fee
- emergency `price` = amount after fee

That creates integration ambiguity and makes client rendering depend on hidden server-side branching.

## Implemented Direction

Implemented direction for this module:

- change only `GET /api/experts/me/consultations`
- remove legacy `price`
- expose `grossPrice` and `netPrice`
- keep `grossPrice` and `netPrice` explicit instead of client-calculated
- leave member/admin consultation-history contracts unchanged in this release

## Likely Impacted Areas

- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/ConsultationPricePreservationTests.cs`
- possibly mobile integration docs and admin-facing consultation history docs if scope is expanded

## Locked Money Semantics

Locked money semantics for the current implementation:

- scheduled `grossPrice` = `ConsultationBooking.Price`
- scheduled `netPrice` = persisted `ExpertPayout.Amount`, otherwise `null`
- emergency `grossPrice` = persisted `ConsultationPayment.Amount` by `ConsultationPingRequest.Id`
- emergency `netPrice` = persisted `ExpertPayout.Amount` by `Consultation.Id`
- if expert payout does not exist yet, `netPrice = null`

Scope boundary already locked:

- only expert history changed in this release
- adjacent member/admin history APIs still keep their previous contracts
