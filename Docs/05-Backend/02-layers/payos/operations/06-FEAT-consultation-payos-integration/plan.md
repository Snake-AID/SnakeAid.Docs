---
doc_role: operation
operation_id: 06-FEAT-consultation-payos-integration
type: FEAT
status: draft
created_at: 2026-03-09
affects:
  - SnakeAid.Api/Controllers/ConsultationPaymentsController.cs
  - SnakeAid.Service/Implements/ConsultationPaymentService.cs
  - SnakeAid.Service/Interfaces/*
  - SnakeAid.Core/Requests/Consultation/*
  - SnakeAid.Core/Responses/Consultation/*
---

# Plan - 06-FEAT-consultation-payos-integration

## 1. As-Is

Consultation payment currently supports only `WalletBalance`.

## 2. Gap Analysis

Without external provider support:
- new users cannot pay consultation unless they already have wallet balance
- product is forced to depend on wallet top-up readiness

## 3. To-Be Design

Integrate consultation payment with the decoupled `PayOsProvider` so:
- scheduled consultation can create external checkout
- emergency consultation can create external checkout
- payment success still lands in consultation escrow logic

## 4. Impacted Components

- consultation payment service
- consultation payment controller
- `PayOsProvider`
- webhook orchestration to consultation escrow

## 5. Risks & Constraints

- consultation escrow semantics must remain intact
- provider integration must not bypass consultation state machine
- no duplication of snake-catching PayOS stack

## 6. Validation Plan

- scheduled and emergency consultation can create provider-backed payment sessions
- webhook/confirmation maps into consultation escrow correctly
- consultation contracts remain consultation-specific while `PayOsProvider` stays domain-neutral at the business-domain boundary
