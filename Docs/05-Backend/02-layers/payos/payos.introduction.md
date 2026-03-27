---
doc_role: baseline
module: payos
kind: layer
status: active
last_updated: 2026-03-28
owners: [backend-team]
---

# PayOS Layer Introduction

## Domain Context

PayOS layer la external payment-gateway integration cua SnakeAid.Backend. Sau refactor, layer nay da duoc tach thanh:

- **Gateway adapter** (`IPaymentGateway` / `PayOsGateway`): reusable, domain-neutral
- **Domain payment services**: moi domain tu own business rules cua minh

## Current Architecture

```
Domain Services          Gateway
+--------------------------+     +------------------+
| SnakeCatchingPaymentSvc  | --> | IPaymentGateway  |
| ConsultationPaymentSvc   | --> |   PayOsGateway   |
| WalletTopupService       | --> |                  |
+--------------------------+     +------------------+
```

Nguyen tac:
- Business rules nam trong domain service
- PayOS SDK integration nam trong gateway
- Khong co them Client/Provider/Orchestrator abstraction

## Current Consumers

| Domain | Service | Status |
|--------|---------|--------|
| Snake Catching | `SnakeCatchingPaymentService` | Production |
| Consultation | `ConsultationPaymentService` | Production (WalletBalance + PayOS) |
| Wallet Top-up | `WalletTopupService` | Production |

## Core Invariants

1. Gateway provider: PayOS
2. Payment records: shared `Transaction` table
3. Domain services own escrow/refund/settlement logic
4. Gateway chi lam: create link, cancel link, verify webhook, fetch status
5. Moi domain service tu quyet dinh khi nao move money, refund, settle

## Reusable vs Domain-Specific

### Reusable (shared infrastructure)

- `IPaymentGateway` / `PayOsGateway`
- PayOS gateway models (`PayOsCreatePaymentRequest`, `PayOsPaymentLinkResult`, `PayOsLinkInformation`, `PayOsWebhookData`)

### Domain-specific (khong reuse cross-domain)

- `SnakeCatchingPaymentService` + snake-catching DTOs
- `ConsultationPaymentService` + consultation DTOs
- `WalletTopupService` + wallet top-up DTOs

## Out of Scope

- Multi-provider support (VNPay, etc.) — chua implement
- Generic payment orchestrator — intentionally removed
- Payment status query endpoint cho consultation — chua implement

## Architectural Risks

1. `PayOsController` van la gateway-named nhung expose snake-catching business endpoints
2. Wallet top-up settlement logic van partially coupled voi snake-catching payment handling qua shared transaction interpretation
3. Shared payment models van PayOS-shaped, future VNPay work co the can contract cleanup
