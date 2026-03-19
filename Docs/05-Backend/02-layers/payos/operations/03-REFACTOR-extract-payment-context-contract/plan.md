---
doc_role: operation
operation_id: 03-REFACTOR-extract-payment-context-contract
type: REFACTOR
status: done
created_at: 2026-03-09
affects:
  - SnakeAid.Core/Requests/PayOS/*
  - SnakeAid.Core/Responses/PayOS/*
  - SnakeAid.Service/Interfaces/*
  - SnakeAid.Service/Services/PayOs/*
---

# Plan - 03-REFACTOR-extract-payment-context-contract

## 1. As-Is

Current PayOS contracts are snake-catching-specific and cannot be safely reused by consultation or wallet top-up.

## 2. Gap Analysis

The system lacks a shared domain-neutral orchestration contract such as:
- payment intent
- payment reference type
- settlement/refund context

## 3. To-Be Design

Introduce a payment-context contract that lets any domain provide:
- reference id
- reference type
- amount
- actor ids
- item description
- success/failure callback semantics

This contract should be consumed by domain orchestrators while `PayOsProvider` remains PayOS-specific but domain-neutral.

## 4. Impacted Components

- PayOS DTO layer
- service contracts between domain layer and provider layer

## 5. Risks & Constraints

- must not leak domain-specific fields into generic contracts
- must keep provider API expressive enough for all three target domains:
  - snake catching
  - wallet top-up
  - consultation

## 6. Validation Plan

- one shared payment context can represent all three target domains
- no contract requires `SnakeCatchingRequestId`
- no contract assumes rescuer-specific payout semantics
