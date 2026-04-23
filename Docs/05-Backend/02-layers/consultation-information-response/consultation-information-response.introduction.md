---
doc_role: planning
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

This module plans the response-contract cleanup for consultation history APIs where `price` currently means different things depending on consultation type.

Business target:

- stop forcing Flutter to calculate platform fee locally
- expose both gross and net price from backend
- remove the current ambiguity where one response returns pre-fee price and another returns post-fee price

Recommended target contract direction:

- replace the single ambiguous `price` meaning with two explicit values:
  - `priceBeforePlatformFee`
  - `priceAfterPlatformFee`

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

## Planned Direction

Planned direction for this module:

- keep the endpoint surface stable unless a stronger refactor is approved
- make response semantics explicit with separate gross/net fields
- update tests so scheduled and emergency history items both expose deterministic price meaning
- sync docs so mobile developers can consume the response without reverse-engineering the service logic

## Likely Impacted Areas

- `SnakeAid.Core/Responses/Consultation/ExpertConsultationResponse.cs`
- `SnakeAid.Service/Interfaces/IConsultationService.cs`
- `SnakeAid.Service/Implements/ConsultationService.cs`
- `SnakeAid.Tests/Integration/ConsultationPricePreservationTests.cs`
- possibly mobile integration docs and admin-facing consultation history docs if scope is expanded

## Main Constraint

One important unresolved design constraint remains:

- for consultations where expert payout is not yet persisted, the source of truth for `priceAfterPlatformFee` is not fully locked

That decision is tracked in `consultation-information-response.hallucination.md`.
