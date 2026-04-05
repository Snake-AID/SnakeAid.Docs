# Money Aspect Sourcemap

## Purpose

File này mô tả structure mục tiêu của money aspect sau refactor.

Mục đích:

- giúp dev đọc nhanh ownership mới
- giúp reviewer kiểm tra đúng ranh giới flow
- giúp agent resume với structural memory rõ ràng

## Document mode

File này là `decision-only`.

Quy ước sử dụng:

- các diagram trong file này luôn là `target-state`
- không sửa diagram để phản ánh implementation tạm thời giữa chừng
- tiến độ implementation được ghi bằng note, mô tả hệ thống hiện đã đi tới đâu so với target diagram
- nếu target diagram cần đổi, đó là thay đổi decision và phải được chốt trước khi sửa file này

## Flow Map

| Flow | Semantic | Owner service | Prefix | Domain side-effect |
|---|---|---|---|---|
| Wallet Topup | nạp tiền vào ví user | `WalletTopupService` | `TOPUP-` | credit user wallet |
| Snake Catching | thanh toán cho snake catching | `SnakeCatchingPaymentService` | `CATCHING-` | update catching payment/domain state |
| Consultation | thanh toán cho consultation | `ConsultationPaymentService` | `CONSULTPAY-` | update booking/request payment state |
| Snakebite Incident | thanh toán cho incident | `SnakebiteIncidentPaymentService` | `INCIDENT-` | update incident payment/domain state |

## Module Diagram

```mermaid
flowchart LR
    PayOsController --> WalletTopupService
    PayOsController --> SnakeCatchingPaymentService
    PayOsController --> ConsultationPaymentService
    PayOsController --> SnakebiteIncidentPaymentService

    WalletController --> WalletTopupService
    SnakeCatchingPaymentsController --> SnakeCatchingPaymentService
    ConsultationPaymentsController --> ConsultationPaymentService
    SnakebiteIncidentController --> SnakebiteIncidentPaymentService

    WalletTopupService --> MoneyTransferService
    WalletTopupService --> MoneyLedgerService

    SnakeCatchingPaymentService --> MoneyEscrowService
    SnakeCatchingPaymentService --> MoneyTransferService
    SnakeCatchingPaymentService --> MoneyLedgerService

    ConsultationPaymentService --> MoneyEscrowService
    ConsultationPaymentService --> MoneyTransferService
    ConsultationPaymentService --> MoneyLedgerService

    SnakebiteIncidentPaymentService --> MoneyEscrowService
    SnakebiteIncidentPaymentService --> MoneyTransferService
    SnakebiteIncidentPaymentService --> MoneyLedgerService
```

## Ownership Rules

- mỗi flow sở hữu `CreatePaymentIntent`
- mỗi flow sở hữu `ConfirmPayment`
- mỗi flow sở hữu `ApplyDomainSideEffects`
- money primitive được share qua service riêng
- `PayOsController` là callback entrypoint chung, chỉ nhận request và dispatch theo prefix
- `PayOsController` không apply domain side-effect
- `wallet topup` dùng prefix `TOPUP-`
- `snake catching` dùng prefix `CATCHING-`
- chấp nhận migration từ `SNAKEAID-` sang `CATCHING-` để đổi lấy routing trật tự theo flow
- manual confirm, return, webhook có thể nhận input khác nhau nhưng owner flow luôn được resolve bằng `description prefix`
- `manual confirm` tiếp tục nhận `transactionId`
- `return` tiếp tục nhận `orderCode`
- `manual confirm` và `return` phải hội tụ về cùng processing path sau bước lookup ban đầu

## Class Diagram

```mermaid
classDiagram
    class PayOsController {
      +ConfirmPayment()
      +Return()
      +Webhook()
      +ResolveByDescriptionPrefix()
      +DispatchByPrefix()
    }

    class WalletTopupService {
      +CreatePaymentIntent()
      +ConfirmPayment()
      +ProcessWebhook()
      +ApplyWalletCredit()
    }

    class SnakeCatchingPaymentService {
      +CreatePaymentIntent()
      +ConfirmPayment()
      +ProcessWebhook()
      +ApplyDomainSideEffects()
    }

    class ConsultationPaymentService {
      +CreatePaymentIntent()
      +ConfirmPayment()
      +ProcessWebhook()
      +ApplyDomainSideEffects()
    }

    class SnakebiteIncidentPaymentService {
      +CreatePaymentIntent()
      +ConfirmPayment()
      +ProcessWebhook()
      +ApplyDomainSideEffects()
    }

    class MoneyEscrowService {
      +MoveMoneyToEscrow()
    }

    class MoneyTransferService {
      +CreditWallet()
      +DebitWallet()
      +TransferSystemMoney()
      +RefundMoney()
    }

    class MoneyLedgerService {
      +CreatePendingTransaction()
      +MarkConfirmed()
      +CreateLedgerPair()
    }

    PayOsController --> WalletTopupService
    PayOsController --> SnakeCatchingPaymentService
    PayOsController --> ConsultationPaymentService
    PayOsController --> SnakebiteIncidentPaymentService

    WalletTopupService --> MoneyTransferService
    WalletTopupService --> MoneyLedgerService

    SnakeCatchingPaymentService --> MoneyEscrowService
    SnakeCatchingPaymentService --> MoneyTransferService
    SnakeCatchingPaymentService --> MoneyLedgerService

    ConsultationPaymentService --> MoneyEscrowService
    ConsultationPaymentService --> MoneyTransferService
    ConsultationPaymentService --> MoneyLedgerService

    SnakebiteIncidentPaymentService --> MoneyEscrowService
    SnakebiteIncidentPaymentService --> MoneyTransferService
    SnakebiteIncidentPaymentService --> MoneyLedgerService
```

## Function Graph

```mermaid
flowchart TD
    A[CreatePaymentIntent] --> B[Validate domain state]
    B --> C[Create pending transaction]
    C --> D{Payment method}
    D -->|PayOS| E[Create checkout link]
    D -->|Wallet| F[Debit wallet or move to escrow]
    E --> G[ConfirmPayment or Webhook]
    F --> H[Apply domain side-effects]
    G --> I[Verify gateway result]
    I --> J[Apply money movement]
    J --> H
    H --> K[Post payment actions]
```

## Sequence Diagram: Wallet Topup

```mermaid
sequenceDiagram
    participant Flutter
    participant WalletController
    participant WalletTopupService
    participant PaymentGateway
    participant PayOsController
    participant MoneyTransferService
    participant MoneyLedgerService

    Flutter->>WalletController: POST /api/wallet/topup
    WalletController->>WalletTopupService: CreatePaymentIntent
    WalletTopupService->>MoneyLedgerService: CreatePendingTransaction
    WalletTopupService->>PaymentGateway: CreatePaymentLink
    WalletTopupService-->>Flutter: checkoutUrl + transactionId + orderCode

    PaymentGateway->>PayOsController: return/webhook
    PayOsController->>WalletTopupService: Dispatch by TOPUP-
    WalletTopupService->>MoneyTransferService: CreditWallet
    WalletTopupService->>MoneyLedgerService: MarkConfirmed
```

## Sequence Diagram: Domain Payment Flow

```mermaid
sequenceDiagram
    participant Client
    participant DomainController
    participant DomainPaymentService
    participant PaymentGateway
    participant PayOsController
    participant MoneyEscrowService
    participant MoneyLedgerService

    Client->>DomainController: Create payment
    DomainController->>DomainPaymentService: CreatePaymentIntent
    DomainPaymentService->>MoneyLedgerService: CreatePendingTransaction
    DomainPaymentService-->>Client: wallet success or checkoutUrl

    PaymentGateway->>PayOsController: return/webhook
    PayOsController->>DomainPaymentService: Dispatch by flow prefix
    DomainPaymentService->>MoneyEscrowService: MoveMoneyToEscrow
    DomainPaymentService->>MoneyLedgerService: MarkConfirmed
    DomainPaymentService->>DomainPaymentService: ApplyDomainSideEffects
```

## File Placement

Nếu phải tạo mới trong repo, vị trí mục tiêu là:

- `SnakeAid.Service/Interfaces/IWalletTopupService.cs`
- `SnakeAid.Service/Implements/WalletTopupService.cs`
- `SnakeAid.Service/Interfaces/IMoneyEscrowService.cs`
- `SnakeAid.Service/Implements/MoneyEscrowService.cs`
- `SnakeAid.Service/Interfaces/IMoneyTransferService.cs`
- `SnakeAid.Service/Implements/MoneyTransferService.cs`
- `SnakeAid.Service/Interfaces/IMoneyLedgerService.cs`
- `SnakeAid.Service/Implements/MoneyLedgerService.cs`

## Reading Order

Khi review hoặc resume:

1. đọc `money-aspect.refactoring.md` để biết phase và scope
2. đọc file này để biết structure mục tiêu
3. đối chiếu implementation thực tế với 3 câu hỏi:
   - flow owner đã đúng chưa
   - shared primitive đã tách khỏi domain side-effect chưa
   - callback/webhook đã không còn đi nhờ flow khác chưa

