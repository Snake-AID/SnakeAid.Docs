# PayOS Loose Coupling Refactor — Report

## Summary

The `PayOsController` was tightly coupled to `SnakeCatchingPaymentService`. Webhook routing used exception-based fallback, the return/confirm endpoints were hardcoded to SnakeCatching, and Consultation webhooks were entirely excluded from the central `/webhook` endpoint. This refactor introduced deterministic prefix-based routing via a plugin-style handler pattern.

## Coupling Issues Found

| # | Issue | Severity |
|---|-------|----------|
| 1 | `PayOsController` constructor injected only `ISnakeCatchingPaymentService` and `ISnakebiteIncidentPaymentService` — every action method called SnakeCatching directly | High |
| 2 | `/webhook` used `try { SnakebiteIncident } catch { SnakeCatching }` fallback — Consultation never reached | High |
| 3 | `/return` only called `ConfirmSnakeCatchingPaymentByOrderCodeAsync` — SnakebiteIncident and Consultation ignored | High |
| 4 | `/confirm-payment` always delegated to `ConfirmSnakeCatchingPaymentAsync` regardless of flow | High |
| 5 | SnakebiteIncident and SnakeCatching shared the `SNAKEAID-` prefix — ambiguous routing | Medium |
| 6 | `/transfer-to-rescuer` hardcoded to SnakeCatching with no abstraction | Low |

## Impacted Files

| File | Change |
|------|--------|
| `SnakeAid.Core/Enums/PaymentFlowType.cs` | New — enum with `SnakeCatching`, `SnakebiteIncident`, `Consultation` |
| `SnakeAid.Service/Services/PayOs/OrderCodePrefixResolver.cs` | New — static prefix-to-flow resolver |
| `SnakeAid.Service/Services/PayOs/PaymentFlowRouter.cs` | New — resolves handler by description, order code, or transaction ID |
| `SnakeAid.Service/Interfaces/IPaymentFlowHandler.cs` | New — plugin interface (`FlowType`, `OrderCodePrefix`, `ProcessWebhookAsync`, `ConfirmByOrderCodeAsync`, `ConfirmByTransactionIdAsync`, `OwnsOrderCode`) |
| `SnakeAid.Service/Implements/SnakeCatchingPaymentService.cs` | Implements `IPaymentFlowHandler` (delegation only) |
| `SnakeAid.Service/Implements/SnakebiteIncidentPaymentService.cs` | Implements `IPaymentFlowHandler`; prefix changed from `SNAKEAID-` to `INCIDENT-` |
| `SnakeAid.Service/Implements/ConsultationPaymentService.cs` | Implements `IPaymentFlowHandler` (delegation only) |
| `SnakeAid.Api/Controllers/PayOsController.cs` | Replaced hardcoded service injections with `PaymentFlowRouter`; all endpoints use router |
| `SnakeAid.Api/Program.cs` | Added DI registrations for `IPaymentFlowHandler` (3 implementations) and `PaymentFlowRouter` |

## Order Code Prefix Registry

| Flow | Prefix | Example |
|------|--------|---------|
| SnakeCatching | `SNAKEAID-` | `SNAKEAID-123456` |
| SnakebiteIncident | `INCIDENT-` | `INCIDENT-789012` |
| Consultation | `CONSULTPAY-` | `CONSULTPAY-345678` |

Prefixes are unique and non-overlapping. `OrderCodePrefixResolver.ResolveFlowType()` performs ordinal `StartsWith` matching in the order: `CONSULTPAY-` → `INCIDENT-` → `SNAKEAID-`.

## Handler Pattern

```
PayOsController
  └─ PaymentFlowRouter
       ├─ IPaymentFlowHandler (SnakeCatchingPaymentService)
       ├─ IPaymentFlowHandler (SnakebiteIncidentPaymentService)
       └─ IPaymentFlowHandler (ConsultationPaymentService)
```

The controller no longer references any flow-specific service. Each endpoint calls the router to resolve the correct `IPaymentFlowHandler` by:

- **Webhook**: `ResolveByDescription(webhookData.Description)` — prefix match
- **Return**: `ResolveByOrderCodeAsync(orderCode)` — transaction lookup then prefix match
- **Confirm**: `ResolveByTransactionIdAsync(transactionId)` — transaction lookup then prefix match
- **Cancel**: `ResolveByOrderCodeAsync(orderCode)` — transaction lookup then prefix match

Adding a new payment flow requires only: implement `IPaymentFlowHandler`, register in DI. No controller changes needed.
