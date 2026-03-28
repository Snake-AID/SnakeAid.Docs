# PayOS Loose Coupling Refactor — Roadmap

## Phase 1: Routing Fix (Completed)

Deterministic prefix-based routing replacing hardcoded SnakeCatching coupling in `PayOsController`.

- [x] **Step 1** — Unique order code prefix for SnakebiteIncident (`SNAKEAID-` → `INCIDENT-`)
- [x] **Step 2** — `PaymentFlowType` enum and `OrderCodePrefixResolver` utility
- [x] **Step 3** — `IPaymentFlowHandler` plugin interface
- [x] **Step 4** — Implement `IPaymentFlowHandler` on all 3 payment services (delegation only, no business logic changes)
- [x] **Step 5** — `PaymentFlowRouter` service (resolve by description, order code, transaction ID)
- [x] **Step 6** — Refactor `PayOsController` to use router instead of hardcoded services
- [x] **Step 9** — DI registrations for `IPaymentFlowHandler` (×3) and `PaymentFlowRouter`

## Phase 2: Shared Infrastructure (Not Started)

Extract duplicated utilities and DTOs from the three payment services (~2800+ lines combined).

- [ ] **Step 7** — Extract shared infrastructure into `PaymentInfrastructure` service
  - [ ] `GenerateOrderCode()` — identical across all 3 services
  - [ ] `BuildDescription(orderCode, prefix, info)` — parameterized by prefix
  - [ ] `ExtractOrderCodeFromDescription(description, prefix)` — parameterized by prefix
  - [ ] `GetRequiredWalletAsync(userId)` — identical in SnakebiteIncident and Consultation
  - [ ] `GetOrCreateWalletAsync(userId)` — identical in SnakebiteIncident and Consultation
  - [ ] `IsPaymentLinkPaid(linkInfo)` — identical in SnakeCatching and Consultation
  - [ ] `SystemWalletUserId` constant — hardcoded in all 3 services
- [ ] **Step 8** — Shared payment DTOs
  - [ ] Create `CreatePaymentRequest` base class with common fields (`ReferenceId`, `Amount`, `Description`, `TransactionType`)
  - [ ] Refactor `CreateSnakeCatchingPaymentRequest`, `CreateSnakebiteIncidentPaymentRequest`, `ProcessConsultationPaymentRequest` to extend base
  - [ ] Create shared `PaymentResponse` base where applicable
