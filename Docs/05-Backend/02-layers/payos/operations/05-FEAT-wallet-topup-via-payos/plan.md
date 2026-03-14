---
doc_role: operation
operation_id: 05-FEAT-wallet-topup-via-payos
type: FEAT
status: draft
created_at: 2026-03-09
affects:
  - SnakeAid.Api/Controllers/WalletController.cs
  - SnakeAid.Service/Interfaces/*
  - SnakeAid.Service/Services/PayOs/*
  - SnakeAid.Core/Requests/*
  - SnakeAid.Core/Responses/*
---

# Plan - 05-FEAT-wallet-topup-via-payos

## 1. As-Is

The system has wallets, but there is currently no clear public top-up flow that lets a user add funds into `SApay` through PayOS.

## 2. Gap Analysis

Without wallet top-up:
- new users cannot fund `SApay`
- consultation payment cannot realistically rely on `WalletBalance` only

## 3. To-Be Design

Implement wallet top-up as a domain flow that uses the shared `PayOsProvider`, not a copied snake-catching PayOS stack.

## 4. Impacted Components

- wallet controller/service
- `PayOsProvider`
- webhook confirmation path
- transaction/wallet update orchestration

## 5. Risks & Constraints

- top-up must credit the member wallet, not the system escrow path used by snake catching or consultation
- webhook idempotency is mandatory

## 6. Validation Plan

- users can create top-up checkout links
- successful PayOS callback credits member wallet balance
- no snake-catching DTO is reused in wallet top-up API
