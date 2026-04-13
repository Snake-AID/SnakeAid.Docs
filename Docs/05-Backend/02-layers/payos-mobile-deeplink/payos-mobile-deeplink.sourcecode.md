# PayOS Mobile Deeplink Sourcecode

## Class diagram

```mermaid
classDiagram
    class DeepLinkHandler {
      +initialize()
      +dispose()
    }

    class PaymentDeepLinkCoordinator {
      +tryPublish(Uri)
      +stream
      +latestEvent
    }

    class PaymentDeepLinkEvent {
      +eventId
      +uri
      +isSuccess
      +isCancelled
      +status
      +id
      +reason
      +orderCode
      +tryParse(Uri)
    }

    class PayOsPendingContext {
      +flowType
      +transactionId
      +orderCode
      +expectedPrefix
      +expectedTransactionType
      +referenceId
      +startedAt
    }

    class PayOsPaymentVerifier {
      +verify(PayOsPendingContext, PaymentDeepLinkEvent)
    }

    class PayOsVerificationResult {
      +status
      +transaction
      +message
      +shouldFallbackConfirm
    }

    class TransactionRepository {
      +getTransactionById(String)
    }

    class WalletRepository {
      +confirmPayment(String)
    }

    class IncidentScreen {
      +listenDeepLink()
      +showSuccessDialog()
    }

    class ConsultationScreen {
      +listenDeepLink()
      +navigateAfterConfirmed()
    }

    class DepositScreen {
      +persistPendingTopup()
      +refreshWallet()
    }

    class ActivityDetailScreen {
      +checkDepositRound()
      +checkFinalRound()
    }

    DeepLinkHandler --> PaymentDeepLinkCoordinator
    PaymentDeepLinkCoordinator --> PaymentDeepLinkEvent
    IncidentScreen --> PaymentDeepLinkCoordinator
    ConsultationScreen --> PaymentDeepLinkCoordinator
    DepositScreen --> PaymentDeepLinkCoordinator
    ActivityDetailScreen --> PaymentDeepLinkCoordinator
    IncidentScreen --> PayOsPaymentVerifier
    ConsultationScreen --> PayOsPaymentVerifier
    DepositScreen --> PayOsPaymentVerifier
    ActivityDetailScreen --> PayOsPaymentVerifier
    PayOsPaymentVerifier --> TransactionRepository
    DepositScreen --> WalletRepository
```

## Backend routing diagram

```mermaid
classDiagram
    class PayOsController {
      +Return(code,id,cancel,status,orderCode)
      +ConfirmPayment(transactionId)
      -ConfirmByOrderCodeAsync(orderCode)
    }

    class PayOsPaymentFlowPrefixes {
      +Topup = "TOPUP-"
      +SnakeCatching = "CATCHING-"
      +SnakebiteIncident = "INCIDENT-"
      +Consultation = "CONSULTPAY-"
      +TryResolve(description)
    }

    class WalletTopupService {
      +ConfirmWalletTopupAsync(transactionId)
      +ConfirmWalletTopupByOrderCodeAsync(orderCode)
    }

    class ConsultationPaymentService {
      +ConfirmConsultationPaymentAsync(transactionId)
      +ConfirmConsultationPaymentByOrderCodeAsync(orderCode)
    }

    class SnakebiteIncidentPaymentService {
      +ConfirmSnakebiteIncidentPaymentAsync(transactionId)
      +ConfirmSnakebiteIncidentPaymentByOrderCodeAsync(orderCode)
    }

    class SnakeCatchingPaymentService {
      +ConfirmSnakeCatchingPaymentAsync(transactionId)
      +ConfirmSnakeCatchingPaymentByOrderCodeAsync(orderCode)
    }

    PayOsController --> PayOsPaymentFlowPrefixes
    PayOsController --> WalletTopupService
    PayOsController --> ConsultationPaymentService
    PayOsController --> SnakebiteIncidentPaymentService
    PayOsController --> SnakeCatchingPaymentService
```

## Sequence diagram: target happy path

```mermaid
sequenceDiagram
    participant User
    participant Flutter
    participant PayOS
    participant Backend
    participant TxAPI as GET /api/transactions/{id}

    Flutter->>Backend: create payment link
    Backend-->>Flutter: transactionId + orderCode + checkoutUrl
    Flutter->>Flutter: save PayOsPendingContext
    Flutter->>PayOS: open checkoutUrl
    PayOS->>Backend: returnUrl?code=00&status=PAID&orderCode=...
    Backend->>Backend: ConfirmByOrderCodeAsync(orderCode)
    PayOS-->>Flutter: deeplink/app-link return event
    Flutter->>Flutter: match pending context by orderCode
    loop retry ngắn
      Flutter->>TxAPI: getTransactionById(transactionId)
      TxAPI-->>Flutter: transaction detail
    end
    Flutter->>Flutter: verify prefix + transactionType + externalTransactionId
    Flutter-->>User: show payment success UI
```

## Sequence diagram: fallback path

```mermaid
sequenceDiagram
    participant Flutter
    participant Backend
    participant PayOS
    participant TxAPI as GET /api/transactions/{id}

    PayOS-->>Flutter: deeplink success
    loop retry ngắn
      Flutter->>TxAPI: getTransactionById(transactionId)
      TxAPI-->>Flutter: externalTransactionId vẫn rỗng
    end
    Flutter->>Backend: POST /api/v1/PayOs/confirm-payment
    Backend->>PayOS: get payment link information by orderCode
    Backend->>Backend: process confirmed payment
    Flutter->>TxAPI: getTransactionById(transactionId)
    TxAPI-->>Flutter: externalTransactionId đã có
    Flutter->>Flutter: verify prefix + orderCode
```

## Flow-specific verification matrix

| Flow | Prefix backend | Expected transaction type | Current mobile status | Ghi chú |
|---|---|---|---|---|
| Wallet topup | `TOPUP-` | topup transaction type ở wallet module | Sai hướng | đang gọi `confirm-payment` quá sớm |
| Consultation PayOS | `CONSULTPAY-` | consultation payment | Gần đúng | còn phụ thuộc domain confirm API |
| Snakebite incident | `INCIDENT-` | `SnakebiteIncidentPayment` | Gần đúng nhất | nên dùng `startsWith` |
| Snake catching deposit/final | `CATCHING-` | `CatchingDeposit` hoặc `CatchingPayment` | Thiếu dữ liệu verify | model hiện chưa có `externalTransactionId` |

## Refactor extraction order

```mermaid
flowchart TD
    A[member_incident_finished_detail] --> B[extract shared verifier]
    B --> C[deposit_money_screen]
    B --> D[payment_confirmation_screen]
    B --> E[activity_detail_screen]
    C --> F[remove confirm-payment as happy path]
    D --> G[unify consultation and topup callback handling]
    E --> H[upgrade snake catching transaction model]
```
