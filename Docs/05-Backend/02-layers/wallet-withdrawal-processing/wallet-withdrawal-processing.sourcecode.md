---
doc_role: implementation
module: wallet-withdrawal-processing
kind: flow
doc_type: sourcecode
status: active
last_updated: 2026-04-21
owners: [backend-team]
verification_status: code-verified
---

# Wallet Withdrawal Processing Sourcecode

## Relevant Classes

- `WithdrawalsController`
- `AdminWithdrawalsController`
- `WalletController`
- `IWalletWithdrawService`
- `WalletWithdrawService`
- `WalletWithdraw`
- `Wallet`
- `Transaction`
- `CreateWithdrawalRequest`
- `WithdrawalResponse`
- `AdminWithdrawalResponse`

## Code-Verified Current Backend Surface

### User Routes

- `GET /api/wallet/me`
- `GET /api/wallet/banks`
- `POST /api/withdrawals/create`
- `GET /api/withdrawals/me`
- `GET /api/withdrawals/{id}`
- `POST /api/withdrawals/{id}/cancel`

### Admin Routes

- `GET /api/admin/withdrawals`
- `GET /api/admin/withdrawals/pending`
- `GET /api/admin/withdrawals/{id}`
- `POST /api/admin/withdrawals/{id}/approve`
- `POST /api/admin/withdrawals/{id}/reject`
- `POST /api/admin/withdrawals/{id}/complete`
- `POST /api/admin/withdrawals/{id}/fail`

## Current Financial Lifecycle

Current verified lifecycle:

1. `CreateWithdrawalRequestAsync(...)`
2. insert `WalletWithdraw` with status `Pending`
3. do not mutate `Wallet.Balance`
4. validate available amount by subtracting current pending-withdrawal sum
5. `ApproveWithdrawalAsync(...)` deducts wallet balance and inserts `TransactionType.WalletWithdraw`
6. `CompleteWithdrawalAsync(...)` only marks `Completed`
7. `RejectWithdrawalAsync(...)` refunds only when the source status is `Approved`
8. `FailWithdrawalAsync(...)` refunds and inserts `TransactionType.AdminAdjustment`
9. `CancelWithdrawalAsync(...)` marks `Rejected` without wallet mutation

## Target Financial Lifecycle

Planned target lifecycle:

1. `CreateWithdrawalRequestAsync(...)` inserts `Pending` withdrawal
2. the same create transaction also holds funds by reducing `Wallet.Balance`
3. create `TransactionType.WalletWithdrawHold`
4. `ApproveWithdrawalAsync(...)` generates QR and marks `Approved`
5. `CompleteWithdrawalAsync(...)` marks `Completed`
6. `RejectWithdrawalAsync(...)` releases held funds through `TransactionType.WalletWithdrawRelease`
7. `CancelWithdrawalAsync(...)` releases held funds through `TransactionType.WalletWithdrawRelease`
8. `FailWithdrawalAsync(...)` releases held funds through `TransactionType.WalletWithdrawRelease`

## Class Diagram

```mermaid
classDiagram
    class WithdrawalsController {
        +CreateWithdrawal(CreateWithdrawalRequest request)
        +GetMyWithdrawals()
        +GetWithdrawalById(Guid id)
        +CancelWithdrawal(Guid id)
    }

    class AdminWithdrawalsController {
        +GetAllWithdrawals()
        +GetPendingWithdrawals()
        +GetWithdrawalById(Guid id)
        +ApproveWithdrawal(Guid id, ApproveWithdrawalRequest? request)
        +RejectWithdrawal(Guid id, RejectWithdrawalRequest request)
        +CompleteWithdrawal(Guid id, CompleteWithdrawalRequest? request)
        +FailWithdrawal(Guid id, FailWithdrawalRequest request)
    }

    class IWalletWithdrawService {
        +CreateWithdrawalRequestAsync(Guid userId, decimal amount, string bankAccount, string bankName, string accountHolderName, string bankBin)
        +CancelWithdrawalAsync(Guid withdrawalId, Guid userId)
        +ApproveWithdrawalAsync(Guid withdrawalId, Guid adminUserId, string? adminNotes)
        +RejectWithdrawalAsync(Guid withdrawalId, Guid adminUserId, string reason, string? adminNotes)
        +CompleteWithdrawalAsync(Guid withdrawalId, Guid adminUserId, string? adminNotes)
        +FailWithdrawalAsync(Guid withdrawalId, Guid adminUserId, string reason, string? adminNotes)
    }

    class WalletWithdrawService
    class WalletWithdraw {
        +Guid Id
        +Guid UserId
        +Guid WalletId
        +decimal Amount
        +WalletWithdrawStatus Status
        +string? VietQrPayload
        +string? VietQrImageBase64
        +string? RejectionReason
        +Guid? ProcessedByAdminId
        +string? AdminNotes
        +DateTime CreatedAt
        +DateTime? ProcessedAt
    }

    class Wallet {
        +Guid Id
        +Guid UserId
        +decimal Balance
    }

    class Transaction {
        +Guid Id
        +Guid UserId
        +Guid? ReferenceId
        +decimal Amount
        +TransactionType TransactionType
    }

    WithdrawalsController --> IWalletWithdrawService
    AdminWithdrawalsController --> IWalletWithdrawService
    WalletWithdrawService ..|> IWalletWithdrawService
    WalletWithdraw --> Wallet
    WalletWithdrawService --> WalletWithdraw
    WalletWithdrawService --> Wallet
    WalletWithdrawService --> Transaction
```

## Sequence Diagram

### Current Verified Flow

```mermaid
sequenceDiagram
    participant User as User App
    participant API as WithdrawalsController
    participant Service as WalletWithdrawService
    participant DB as Database
    participant Admin as AdminWithdrawalsController

    User->>API: POST /api/withdrawals/create
    API->>Service: CreateWithdrawalRequestAsync(...)
    Service->>DB: load Wallet
    Service->>DB: sum Pending withdrawals
    Service->>DB: insert WalletWithdraw(Pending)
    Service-->>API: WithdrawalResponse

    Admin->>Admin: POST /api/admin/withdrawals/{id}/approve
    Admin->>Service: ApproveWithdrawalAsync(...)
    Service->>DB: deduct Wallet.Balance
    Service->>DB: insert Transaction(WalletWithdraw)
    Service->>DB: store VietQR payload and status Approved

    Admin->>Admin: POST /api/admin/withdrawals/{id}/complete
    Admin->>Service: CompleteWithdrawalAsync(...)
    Service->>DB: update status Completed
```

### Planned Target Flow

```mermaid
sequenceDiagram
    participant User as User App
    participant API as WithdrawalsController
    participant Service as WalletWithdrawService
    participant DB as Database
    participant Admin as AdminWithdrawalsController

    User->>API: POST /api/withdrawals/create
    API->>Service: CreateWithdrawalRequestAsync(...)
    Service->>DB: load Wallet
    Service->>DB: validate amount and limits
    Service->>DB: deduct Wallet.Balance
    Service->>DB: insert Transaction(WalletWithdrawHold)
    Service->>DB: insert WalletWithdraw(Pending)
    Service-->>API: WithdrawalResponse

    Admin->>Admin: POST /api/admin/withdrawals/{id}/approve
    Admin->>Service: ApproveWithdrawalAsync(...)
    Service->>DB: store VietQR payload and status Approved

    Admin->>Admin: POST /api/admin/withdrawals/{id}/reject
    Admin->>Service: RejectWithdrawalAsync(...)
    Service->>DB: release held amount
    Service->>DB: insert Transaction(WalletWithdrawRelease)
    Service->>DB: update status Rejected
```

## Test Focus

- create now mutates wallet balance
- approve no longer mutates wallet balance
- complete no longer mutates wallet balance
- reject from pending releases funds
- cancel from pending releases funds
- fail from approved releases funds once
- no path can double-deduct or double-refund
