# Kế hoạch Triển khai Chức năng Rút Tiền (Withdrawal) cho SnakeAid

## 0. Phạm vi & Mục tiêu

### Phạm vi
- Triển khai dòng tiền ra (outbound flow) cho hệ thống ví của SnakeAid
- Hỗ trợ expert và member rút tiền từ ví về tài khoản ngân hàng
- Quy trình manual approval bởi admin (không sử dụng PayOS)
- Tích hợp với cơ chế ví và transaction hiện có
- Không phạm vi: Tự động payout qua API ngân hàng, crypto payments

### Không phạm vi
- Multi-currency support (chỉ VND)
- Third-party payment gateways cho outbound (chỉ manual admin transfer)
- Bulk payouts hoặc scheduled withdrawals
- International wire transfers
- Integration với ngân hàng API

### Mục tiêu thành công
1. Users có thể tạo withdrawal request với thông tin tài khoản ngân hàng
2. Admin có thể review và approve/reject requests
3. Admin thực hiện chuyển khoản manual qua ứng dụng ngân hàng
4. System track trạng thái và log audit trails
5. Notifications gửi tới users và admins
6. Compliance với local regulations (Việt Nam)

### Giả định
- Admin luôn có quyền truy cập vào tài khoản ngân hàng của platform
- Transfer manual qua mobile banking app của admin
- No automated bank integrations (manual process only)
- Transfer fees do platform chịu (không trừ user)
- Minimum withdrawal amount: 50,000 VND
- Maximum withdrawal: 5,000,000 VND per request
- Daily withdrawal limit: 10,000,000 VND per user

## 1. Kiến trúc hệ thống ở mức cao

### Các phần tử chính

#### 1.1 User Layer (Mobile/Web App)
- Withdrawal request form (amount, bank account, bank name)
- Withdrawal history list
- Real-time status updates
- Push notifications

#### 1.2 API Layer (SnakeAid.Api)
- `WithdrawalsController` - User CRUD operations cho withdrawals
- `AdminWithdrawalsController` - Admin operations (approve/reject/list/complete)
- Integration với existing `WalletController` (add bank directory endpoint)

#### 1.3 Service Layer (SnakeAid.Service)
- `IWalletWithdrawService` - Business logic cho withdrawals
- `IWalletWithdrawNotificationService` - Notifications
- `IVietQrAdapter` - QR code generation for VietQR payments (inspired by EzyFix template)
- `IBankDirectoryService` - Bank directory lookup for UI dropdowns (inspired by EzyFix template)
- Extend existing `IWalletService` và `ITransactionService`

#### 1.4 Repository Layer (SnakeAid.Repository)
- Extend existing repositories với withdrawal queries
- Add audit logging

#### 1.5 Data Layer (PostgreSQL)
- Existing `WalletWithdraw` table (cần extend)
- Existing `Transaction` table (có `WalletWithdraw` type)
- Existing `Wallet` table (balance management)

#### 1.6 External Systems
- Email service (notifications)
- Push notification service
- Admin dashboard (for approvals)

### Cách giao tiếp

```
User App → API Gateway → WithdrawalsController → IWalletWithdrawService
                                              ↓
                                   IWalletWithdrawNotificationService
                                              ↓
                                   Email/Push Services

Admin Dashboard → AdminWithdrawalsController → IWalletWithdrawService
                                                ↓
                                     Manual Bank Transfer (via mobile app)
```

## 2. Mô hình dữ liệu

### Thực thể hiện có cần extend

#### WalletWithdraw (extend)
```csharp
public class WalletWithdraw : BaseEntity
{
    [Key] public Guid Id { get; set; }
    [Required] public Guid UserId { get; set; }
    [Required] public Guid WalletId { get; set; }
    [Required] [Range(10000, 50000000)] public decimal Amount { get; set; }

    // Bank account info
    [Required] [MaxLength(20)] public string BankAccount { get; set; }
    [Required] [MaxLength(100)] public string BankName { get; set; }
    [Required] [MaxLength(6)] public string BankBin { get; set; }

    [Required] public WalletWithdrawStatus Status { get; set; } = Pending;
    public DateTime? ProcessedAt { get; set; }
    public Guid? ProcessedByAdminId { get; set; } // Track who approved
    [MaxLength(500)] public string? RejectionReason { get; set; }
    [MaxLength(500)] public string? AdminNotes { get; set; }

    // VietQR support (inspired by EzyFix template)
    [MaxLength(1000)] public string? VietQrPayload { get; set; }
    public string? VietQrImageBase64 { get; set; } // Base64 encoded QR image for mobile banking apps

    // Navigation
    public Account User { get; set; }
    public Wallet Wallet { get; set; }
    public Account? ProcessedByAdmin { get; set; }
}
```

#### Transaction (extend TransactionType)
```csharp
public enum TransactionType
{
    // ... existing types
    
    // Wallet operations
    WalletTopup = 31,
    WalletWithdraw = 32,     // ReferenceId = WalletWithdrawId
    WalletWithdrawFee = 33,  // Platform fees if any
    AdminAdjustment = 34
}
```

### Thực thể mới (optional)

#### WithdrawalAuditLog
```csharp
public class WithdrawalAuditLog
{
    [Key] public Guid Id { get; set; }
    [Required] public Guid WithdrawalId { get; set; }
    [Required] public string Action { get; set; } // Created, Approved, Rejected, Completed, Failed
    [Required] public Guid PerformedBy { get; set; }
    [Required] public DateTime Timestamp { get; set; }
    public string? OldStatus { get; set; }
    public string? NewStatus { get; set; }
    [MaxLength(1000)] public string? Notes { get; set; }
    
    // Navigation
    public WalletWithdraw Withdrawal { get; set; }
    public Account PerformedByUser { get; set; }
}
```

### Mối quan hệ
- `WalletWithdraw` belongs to `Wallet` (1:1) và `Account` (User)
- `WalletWithdraw` has many `WithdrawalAuditLog`
- `Transaction` references `WalletWithdraw` via `ReferenceId` when `TransactionType = WalletWithdraw`

## 2.5 Naming Conventions

### Classes và Interfaces

#### Core Entities
- `WalletWithdraw` - Entity cho withdrawal requests
- `WithdrawalAuditLog` - Entity cho audit logging
- `WalletWithdrawStatus` - Enum: Pending, Approved, Rejected, Completed, Failed

#### Services
- `IWalletWithdrawService` - Interface cho withdrawal business logic
- `IWalletWithdrawNotificationService` - Interface cho notifications
- `IVietQrAdapter` - Interface cho QR code generation
- `IBankDirectoryService` - Interface cho bank directory lookup
- `VietQrAdapter` - Implementation của IVietQrAdapter
- `BankDirectoryService` - Implementation của IBankDirectoryService

#### Controllers
- `WithdrawalsController` - User endpoints cho withdrawals
- `AdminWithdrawalsController` - Admin endpoints cho withdrawal management

#### DTOs
- `CreateWithdrawalRequest` - Request DTO cho tạo withdrawal
- `WithdrawalResponse` - Response DTO cho withdrawal details
- `BankDirectoryResponse` - Response DTO cho bank directory
- `VietQrStatus` - Enum: TransferSupported, ReceiveOnly

### Endpoints

#### User Endpoints (prefix: `/api/withdrawals`)
- `POST /api/withdrawals/create` - CreateWithdrawalRequest
- `GET /api/withdrawals/me` - Get user withdrawal history
- `GET /api/withdrawals/{id}` - Get withdrawal details
- `POST /api/withdrawals/{id}/cancel` - Cancel pending withdrawal

#### Bank Directory Endpoints (prefix: `/api/wallet`)
- `GET /api/wallet/banks` - Get supported banks list

#### Admin Endpoints (prefix: `/api/admin/withdrawals`)
- `GET /api/admin/withdrawals` - List all withdrawals with filters
- `GET /api/admin/withdrawals/{id}` - Get withdrawal details with QR code
- `POST /api/admin/withdrawals/{id}/approve` - Approve withdrawal
- `POST /api/admin/withdrawals/{id}/reject` - Reject withdrawal
- `POST /api/admin/withdrawals/{id}/complete` - Mark as completed
- `POST /api/admin/withdrawals/{id}/fail` - Mark as failed

### Methods

#### IWalletWithdrawService
- `Task<WithdrawalResponse> CreateWithdrawalAsync(CreateWithdrawalRequest request, Guid userId)`
- `Task<WithdrawalResponse> GetWithdrawalAsync(Guid withdrawalId, Guid userId)`
- `Task<List<WithdrawalResponse>> GetUserWithdrawalsAsync(Guid userId)`
- `Task CancelWithdrawalAsync(Guid withdrawalId, Guid userId)`
- `Task<WithdrawalResponse> ApproveWithdrawalAsync(Guid withdrawalId, Guid adminId)`
- `Task<WithdrawalResponse> RejectWithdrawalAsync(Guid withdrawalId, Guid adminId, string reason)`
- `Task<WithdrawalResponse> CompleteWithdrawalAsync(Guid withdrawalId, Guid adminId)`
- `Task<WithdrawalResponse> FailWithdrawalAsync(Guid withdrawalId, Guid adminId, string reason)`

#### IVietQrAdapter
- `Task<VietQrResult> GenerateAsync(VietQrRequest request, CancellationToken cancellationToken)`

#### IBankDirectoryService
- `Task<IReadOnlyList<BankDirectoryResponse>> GetBanksAsync(CancellationToken cancellationToken)`

#### WithdrawalsController
- `Task<IActionResult> CreateWithdrawal(CreateWithdrawalRequest request)`
- `Task<IActionResult> GetMyWithdrawals()`
- `Task<IActionResult> GetWithdrawal(Guid id)`
- `Task<IActionResult> CancelWithdrawal(Guid id)`

#### AdminWithdrawalsController
- `Task<IActionResult> GetWithdrawals([FromQuery] AdminWithdrawalFilters filters)`
- `Task<IActionResult> GetWithdrawal(Guid id)`
- `Task<IActionResult> ApproveWithdrawal(Guid id, AdminApproveRequest request)`
- `Task<IActionResult> RejectWithdrawal(Guid id, AdminRejectRequest request)`
- `Task<IActionResult> CompleteWithdrawal(Guid id)`
- `Task<IActionResult> FailWithdrawal(Guid id, AdminFailRequest request)`

## 3. Thiết kế API và giao diện người dùng

### Endpoints chính

#### User Endpoints (`/api/withdrawals`)
```
POST   /api/withdrawals/create           - Tạo withdrawal request
GET    /api/withdrawals/me               - List user's withdrawal history
GET    /api/withdrawals/{id}             - Chi tiết withdrawal
POST   /api/withdrawals/{id}/cancel      - Hủy withdrawal (chỉ khi Pending)
```

#### Bank Directory Endpoints (`/api/wallet`)
```
GET    /api/wallet/banks                  - List supported banks for withdrawals (inspired by EzyFix template)
```

#### Admin Endpoints (`/api/admin/withdrawals`)
```
GET    /api/admin/withdrawals             - List tất cả withdrawals (có filter)
GET    /api/admin/withdrawals/{id}       - Chi tiết withdrawal với VietQR code
POST   /api/admin/withdrawals/{id}/approve - Approve withdrawal và generate VietQR code (inspired by EzyFix template)
POST   /api/admin/withdrawals/{id}/reject  - Reject withdrawal
POST   /api/admin/withdrawals/{id}/complete - Mark as completed sau khi transfer
POST   /api/admin/withdrawals/{id}/fail    - Mark as failed nếu transfer lỗi
```

### Request/Response DTOs

#### CreateWithdrawalRequest
```csharp
public class CreateWithdrawalRequest
{
    [Required] [Range(10000, 50000000)] public decimal Amount { get; set; }
    [Required] [MaxLength(20)] public string BankAccount { get; set; }
    [Required] [MaxLength(100)] public string BankName { get; set; }
    [Required] [MaxLength(6)] public string BankBin { get; set; }
}
```

#### WithdrawalResponse
```csharp
public class WithdrawalResponse
{
    public Guid Id { get; set; }
    public Guid UserId { get; set; }
    public decimal Amount { get; set; }
    public string BankAccount { get; set; }
    public string BankName { get; set; }
    public string? BankBin { get; set; }
    public WalletWithdrawStatus Status { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime? ProcessedAt { get; set; }
    public string? RejectionReason { get; set; }
    public string? VietQrPayload { get; set; }
    public string? VietQrImageBase64 { get; set; }
}
```

#### BankDirectoryResponse (inspired by EzyFix template)
```csharp
public class BankDirectoryResponse
{
    public string Key { get; set; } = string.Empty;
    public string Code { get; set; } = string.Empty;
    public string ShortName { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string Bin { get; set; } = string.Empty;
    public VietQrStatus VietQrStatus { get; set; } = VietQrStatus.TransferSupported;
    public bool LookupSupported { get; set; }
    public string? SwiftCode { get; set; }
}

public enum VietQrStatus
{
    TransferSupported = 0,
    ReceiveOnly = 1
}
```

### UI Wireframes (High-level)

#### User App - Withdrawal Request Form
- Amount input (min 50k, max 5M, step 10k)
- Bank name dropdown (loaded from /api/wallet/banks inspired by EzyFix template)
- Bank account number input (validation)
- Account holder name (optional)
- Submit button
- Balance display

#### User App - Withdrawal History
- List view với status badges
- Filter by status
- Detail view với timeline

#### Admin Dashboard - Withdrawal Management
- Table view với filters (status, date, amount)
- Bulk actions (approve selected)
- Detail modal với audit log và VietQR code display (inspired by EzyFix template)
- Manual completion buttons

## 4. Luồng nghiệp vụ và trạng thái

### State Machine

```
PENDING → APPROVED → COMPLETED
    ↓         ↓
REJECTED     FAILED
```

#### Transitions & Guards

1. **PENDING → APPROVED**
   - Admin action
   - Guards: User có đủ balance, account info valid, không exceed daily limit

2. **APPROVED → COMPLETED** 
   - Admin action sau khi transfer thành công
   - Actions: Debit wallet balance, create Transaction record, send notification

3. **APPROVED → FAILED**
   - Admin action nếu transfer thất bại
   - Actions: Không debit balance, send notification, có thể retry

4. **PENDING → REJECTED**
   - Admin action
   - Actions: Send notification với reason

### Quy trình chi tiết

#### 4.1 User tạo request
1. User nhập thông tin withdrawal
2. System validate: balance đủ, amount trong range, account format đúng
3. Tạo `WalletWithdraw` với status `Pending`
4. Hold balance (optional - không debit ngay)
5. Send notification tới user và admins

#### 4.2 Admin review
1. Admin view pending withdrawals
2. Check user balance và account info
3. Có thể reject nếu suspicious
4. Nếu approve: change status → `Approved`, generate VietQR code cho mobile banking app (inspired by EzyFix template)

#### 4.3 Admin thực hiện transfer
1. Admin mở app ngân hàng
2. Scan QR code VietQR được generate hoặc nhập thông tin thủ công
3. Transfer tiền từ platform account → user account
4. Upload proof nếu cần (screenshots)
5. Mark as `Completed` trong system

#### 4.4 System xử lý completion
1. Debit wallet balance
2. Tạo Transaction record
3. Send success notification tới user
4. Log audit trail

## 5. Quy trình phê duyệt với thông báo và audit

### Quy trình phê duyệt
1. **Auto-check**: Balance validation, amount limits, daily limits
2. **Admin review**: Manual approval với fraud checks
3. **Dual approval** (optional): 2 admins phải approve cho amounts > 1M VND
4. **Compliance check**: KYC verification nếu amount > 2M VND

### Thông báo
| Event | Recipients | Channel | Content |
|-------|------------|---------|---------|
| Request Created | User + Admins | Push + Email | New withdrawal request |
| Request Approved | User | Push + Email | Your withdrawal approved, processing |
| Request Rejected | User | Push + Email | Withdrawal rejected: {reason} |
| Transfer Completed | User | Push + Email | Money transferred to your account |
| Transfer Failed | User + Admins | Push + Email | Transfer failed, will retry |

### Audit Logging
- **Who**: User ID, Admin ID, System
- **What**: Action performed (Create, Approve, Reject, Complete, Fail)
- **When**: Timestamp
- **Where**: IP address, User agent
- **Why**: Notes, rejection reasons
- **What changed**: Old status → New status

## 6. Yêu cầu về bảo mật và tuân thủ

### Xác thực và ủy quyền
- JWT tokens cho API access
- Role-based access: User vs Admin
- Admin endpoints require `Admin` role
- User chỉ access own withdrawals

### Validation dữ liệu đầu vào
- Amount: Range 50k-5M VND, decimal(18,2)
- Bank account: Regex validation cho Vietnamese accounts
- Bank name: Whitelist of supported banks
- Rate limiting: 5 requests/day per user
- Anti-spam: Prevent duplicate requests

### Chống gian lận
- Velocity checks: Amount per day/hour limits
- Account validation: Basic checksum cho bank accounts
- Fraud detection: Suspicious patterns (round numbers, frequent requests)
- Manual review cho high-risk requests

### Logging và auditing
- All withdrawal operations logged
- Sensitive data encrypted (account numbers)
- Audit trail immutable
- Admin actions tracked with IP/User agent

### Bảo vệ dữ liệu nhạy cảm
- Bank account numbers: AES-256 encryption at rest
- PII data: GDPR compliant storage
- API responses: Mask sensitive fields
- Logs: No sensitive data in plain text

### Tuân thủ pháp lý
- Vietnam banking regulations compliance
- Anti-money laundering checks
- Transaction reporting requirements
- Data retention policies (7 years for financial data)

## 7. Các yêu cầu phi chức năng

### Hiệu suất
- Response time: <500ms cho read operations
- Throughput: 100 requests/minute
- Database queries optimized với indexes
- Caching cho frequently accessed data

### Khả năng mở rộng
- Stateless services
- Horizontal scaling support
- Database connection pooling
- CDN cho static assets

### Độ tin cậy
- 99.9% uptime target
- Graceful error handling
- Circuit breaker patterns
- Retry mechanisms cho external calls

### Sao lưu và phục hồi
- Daily database backups
- Point-in-time recovery
- Disaster recovery plan
- Data retention: 7 years

### Observability
- Application metrics (Prometheus)
- Distributed tracing (OpenTelemetry)
- Structured logging (Serilog)
- Health checks (/health endpoint)

## 8. Kế hoạch triển khai theo giai đoạn

### Giai đoạn 1: Foundation (Week 1-2)
#### Mục tiêu: Core infrastructure
#### Tasks:
1. Extend `WalletWithdraw` entity với new fields và enum
2. Add `WithdrawalAuditLog` entity (optional)
3. Update database migration
4. Create `IWalletWithdrawService` interface
5. Create `IBankDirectoryService` interface (inspired by EzyFix template)
6. Implement basic CRUD operations
7. Add validation rules

#### Tiêu chí đánh giá:
- Entities compile successfully
- Migration runs without errors
- Basic create/get operations work
- Unit tests pass (70% coverage)

### Giai đoạn 2: API Development (Week 3-4)
#### Mục tiêu: User và admin APIs
#### Tasks:
1. Implement `WithdrawalsController`
2. Implement `AdminWithdrawalsController`
3. Implement `VietQrAdapter` service (inspired by EzyFix template) với QRCoder và VietQRHelper
4. Implement `BankDirectoryService` với VietQRHelper integration (inspired by EzyFix template)
5. Add QR code generation khi admin approve withdrawal
6. Add bank directory endpoint trong `WalletController`
7. Add DTOs và mappings
8. Implement business logic (balance checks, limits)
9. Add notifications (email/push)

#### Tiêu chí đánh giá:
- All endpoints return correct responses
- Validation works (amount limits, balance checks)
- Bank directory endpoint returns valid bank list (inspired by EzyFix template)
- Notifications sent successfully
- Integration tests pass
- Security tests pass

### Giai đoạn 3: Admin Workflow (Week 5-6)
#### Mục tiêu: Complete admin experience
#### Tasks:
1. Build admin dashboard UI components
2. Implement approve/reject/complete actions
3. Add audit logging
4. Implement fraud detection rules
5. Add reporting/analytics

#### Tiêu chí đánh giá:
- Admin can approve/reject withdrawals
- Audit logs capture all actions
- Dashboard shows pending withdrawals
- E2E flow works end-to-end

### Giai đoạn 4: Testing & Launch (Week 7-8)
#### Mục tiêu: Production ready
#### Tasks:
1. Comprehensive testing (unit, integration, e2e)
2. Performance testing
3. Security audit
4. Documentation update
5. User acceptance testing
6. Production deployment

#### Tiêu chí đánh giá:
- All tests pass
- Performance benchmarks met
- Security scan clean
- Users can create và track withdrawals
- Admins can process requests
- Bank directory loads correctly in UI (inspired by EzyFix template)
- 99% success rate in UAT

## 9. Chiến lược kiểm thử

### Unit Tests
- Service layer logic (balance validation, limits)
- Entity validation rules
- State transitions
- Notification logic
- QR code generation (VietQrAdapter inspired by EzyFix template)

### Integration Tests
- API endpoints với database
- Service interactions
- External service calls (email, push)
- Bank directory service với VietQRHelper

### End-to-End Tests
- Full withdrawal flow: Create → Approve → Complete
- Error scenarios: Insufficient balance, invalid account
- Admin workflow: Review → Process → Complete

### Data mẫu
- Test users với various balances
- Valid/invalid bank accounts
- Edge cases: Min/max amounts, daily limits
- Admin test accounts

## 10. Yêu cầu về tài liệu và deliverables

### Tài liệu thiết kế
- API specification (OpenAPI/Swagger)
- Database schema documentation
- State machine diagrams
- Sequence diagrams cho key flows

### Sơ đồ ERD
- Updated ERD với new entities
- Relationship diagrams
- Index recommendations

### API Contract
- Request/Response schemas
- Error response formats
- Authentication requirements
- Rate limiting specs
- Bank directory API specification (inspired by EzyFix template)

### Wireframes/Mock UI
- User withdrawal form
- Admin approval interface
- Status tracking screens
- Notification designs

### Tài liệu kiểm thử
- Test cases matrix
- Test data specifications
- Automation scripts
- Performance test scenarios

### Kế hoạch rollout
- Deployment checklist
- Rollback procedures
- Monitoring setup
- Communication plan

## 11. Giả định và ràng buộc

### Giả định về luồng thanh toán outbound
- Admin luôn available để process transfers trong 24-48h
- Bank transfers succeed 99% of time
- No automated reconciliation (manual admin confirmation)
- Platform account có đủ funds cho payouts
- Transfer fees absorbed by platform
- Admin mobile banking apps support VietQR scanning (inspired by EzyFix template)

### Khóa dữ liệu và quyền hạn
- Only admins can approve withdrawals
- Users can only view/cancel own withdrawals
- Audit logs immutable, only system can write
- Sensitive data encrypted, only admins can decrypt for processing

### Ràng buộc ảnh hưởng đến thời gian triển khai
- Compliance approval required before launch (1-2 weeks)
- Bank account validation API may not be available (manual checks only)
- Mobile app updates needed for UI changes (coordinate với mobile team)
- Testing với real bank data restricted (use test accounts only)
- VietQRHelper library dependency và bank directory data availability

## 12. Implementation Status & Checklist

### ✅ Completed Tasks
- [x] Review and approve naming conventions from the plan
- [x] Extend WalletWithdraw entity with QR fields (VietQrPayload, VietQrImageBase64) and add Completed/Failed to Status enum
- [x] Add VietQRHelper and QRCoder NuGet dependencies to Service layer
- [x] Implement VietQrAdapter service for QR code generation with fallback payload building
- [x] Implement BankDirectoryService with cached bank list from VietQRHelper.BankApp.BanksObject
- [x] Extend IWalletService interface and implement wallet withdrawal operations
- [x] Create IWalletWithdrawService interface with withdrawal-specific methods
- [x] Implement WithdrawalsController with user endpoints (/api/withdrawals/create, /me, /{id}, /{id}/cancel)
- [x] Add Bank Directory endpoint (/api/wallet/banks) to existing WalletController
- [x] Implement AdminWithdrawalsController with approve/reject/complete/fail endpoints
- [x] Create DTOs for requests and responses (CreateWithdrawalRequest, WithdrawalResponse, BankDirectoryResponse)
- [x] Add validation for Vietnamese bank account formats and daily withdrawal limits
- [x] Implement security measures: encrypt bank accounts, mask responses, audit admin actions
- [x] Create database migration for new fields and enum values
- [x] Integrate with existing IUnitOfWork and transaction patterns
- [x] Run lint and typecheck commands to ensure code quality

### 🔄 Pending Tasks
- [ ] Add unit tests and integration tests for withdrawal flow

### 📊 Progress Summary
- **Completed**: 16/17 tasks (94.1%)
- **Remaining**: 1/17 tasks (5.9%)
- **Status**: Implementation phase completed, ready for testing and deployment

---

## 12. Phác thảo lịch trình và ước lượng effort

### Giai đoạn 1: Foundation (2 weeks)
- **Effort**: 2 dev weeks
- **Dependencies**: Database schema approval
- **Risk**: Entity design changes may require remigration
- **Success criteria**: Core entities implemented, basic CRUD working

### Giai đoạn 2: API Development (2 weeks)  
- **Effort**: 2 dev weeks
- **Dependencies**: Stage 1 completion, notification services ready
- **Risk**: Integration với existing wallet system
- **Success criteria**: All APIs implemented, notifications working

### Giai đoạn 3: Admin Workflow (2 weeks)
- **Effort**: 1.5 dev weeks + 0.5 QA
- **Dependencies**: Admin dashboard framework ready
- **Risk**: UI/UX design approval delays
- **Success criteria**: Admin can fully process withdrawals

### Giai đoạn 4: Testing & Launch (2 weeks)
- **Effort**: 1 dev week + 1 QA week
- **Dependencies**: All previous stages, compliance approval
- **Risk**: Production data migration issues
- **Success criteria**: Feature live in production

### Tổng effort: 6.5 dev weeks + 1.5 QA weeks

### Phụ thuộc quan trọng
- **Critical**: Existing wallet system stability
- **High**: Admin dashboard availability  
- **Medium**: Notification services reliability
- **Low**: Mobile app coordination

### Đánh giá thành công sau triển khai
1. **User metrics**: 80% withdrawal requests processed within 48h
2. **Admin efficiency**: Average processing time < 5min per request  
3. **System reliability**: < 0.1% failure rate for approved withdrawals
4. **Security**: Zero security incidents in first 3 months
5. **User satisfaction**: > 4.5/5 rating for withdrawal experience</content>
<parameter name="filePath">D:\SourceCode\Snake_AID\SnakeAid.Backend\.kilo\plans\1775131669531-crisp-panda.md
