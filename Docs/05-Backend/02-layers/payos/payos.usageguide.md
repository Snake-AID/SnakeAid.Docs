---
doc_role: baseline
module: payos
kind: layer
status: active
last_updated: 2026-03-09
owners: [backend-team]
---

# PayOS Layer Usage Guide

## Purpose

This document describes the **current external contract** of the PayOS layer.

Important current limitation:
- these contracts are currently snake-catching-specific
- they should not be mistaken for a reusable `PayOsProvider` contract

## Authentication

- `POST /api/v1/PayOs/webhook` and PayOS return/cancel URLs are anonymous
- all other PayOS endpoints require authenticated access

## Public Endpoints

### 1. Create Payment Link

`POST /api/v1/PayOs/create-payment-link`

Current request DTO:
- `CreateSnakeCatchingPaymentRequest`

Example request:
```json
{
  "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555",
  "amount": 150000,
  "description": "Payment for snake rescue",
  "transactionType": "CatchingPayment"
}
```

Example response shape:
```json
{
  "success": true,
  "message": "PayOS payment link created successfully",
  "data": {
    "transactionId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
    "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555",
    "amount": 150000,
    "status": "Pending",
    "checkoutUrl": "https://pay.payos.vn/web/...",
    "orderCode": 1741499999123,
    "paymentLinkId": "plink_xxx",
    "expiresAt": null,
    "provider": "PayOS",
    "gatewayRawResponse": {
      "orderCode": 1741499999123,
      "paymentLinkId": "plink_xxx",
      "checkoutUrl": "https://pay.payos.vn/web/...",
      "amount": 150000,
      "status": "PENDING",
      "currency": "VND"
    }
  }
}
```

### 2. Cancel Payment Link

`POST /api/v1/PayOs/cancel-payment-link/{orderCode}`

Example request:
```json
{
  "cancellationReason": "User cancelled checkout"
}
```

### 3. Confirm Payment

`POST /api/v1/PayOs/confirm-payment`

Example request:
```json
{
  "transactionId": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee"
}
```

### 4. Return URL

`GET /api/v1/PayOs/return`

Current purpose:
- browser return endpoint after PayOS portal
- auto-confirms payment when status is successful

### 5. Cancel URL

`GET /api/v1/PayOs/cancel`

Current purpose:
- browser cancel endpoint after user cancels on PayOS portal

### 6. Webhook

`POST /api/v1/PayOs/webhook`

Current purpose:
- receive asynchronous PayOS webhook
- update transaction external reference
- credit system wallet
- update snake-catching request state

### 7. Transfer To Rescuer

`POST /api/v1/PayOs/transfer-to-rescuer`

Current request DTO:
- `TransferToRescuerRequest`

Example request:
```json
{
  "snakeCatchingRequestId": "11111111-2222-3333-4444-555555555555"
}
```

## Consumer Warning

Current PayOS contracts should only be used by:
- snake-catching payment flow
- snake-catching rescuer settlement flow

Do not reuse current DTOs for:
- consultation payment
- wallet top-up
- future generic payment flows

Reason:
- current request/response contracts encode snake-catching identifiers and business semantics directly.

## Current Reuse Boundary

Safe to reuse:
- `IPayOsClient` behavior conceptually
- gateway transport logic

Unsafe to reuse without refactor:
- `IPayOsPaymentService`
- `CreateSnakeCatchingPaymentRequest`
- `SnakeCatchingPaymentResponse`
- `TransferToRescuer*`
- webhook orchestration as-is

Target future boundary:
- `IPayOsClient` = low-level PayOS API wrapper
- `IPayOsProvider` = reusable PayOS façade for multiple business domains
- domain services = snake catching / consultation / wallet top-up orchestration
