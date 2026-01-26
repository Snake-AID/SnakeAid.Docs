# EMERGENCY RESCUE WORKFLOW - HỆ THỐNG SNAKEAID

## 📋 Thông tin tài liệu
- **Business Flow:** Emergency Rescue (Cứu hộ khẩn cấp khi bị rắn cắn)
- **Mục đích:** Mô tả quy trình xử lý sự cố rắn cắn từ lúc phát hiện đến hoàn tất cứu hộ
- **Version:** 1.0
- **Last Updated:** December 30, 2025

---

## 🎯 Mục tiêu Business Flow

Emergency Rescue Flow được thiết kế để:
1. **Phản ứng nhanh** khi có sự cố rắn cắn khẩn cấp
2. **Hỗ trợ sơ cứu** ngay lập tức bằng AI
3. **Kết nối Rescuer/Supporter** (đội cứu hộ SnakeAid) đến hiện trường
4. **Theo dõi real-time** vị trí cứu hộ
5. **Hướng dẫn đến bệnh viện** có huyết thanh kháng nọc
6. **Đảm bảo thanh toán** công bằng cho các bên

---

## 👥 Các Actor tham gia

| Actor | Vai trò | Trách nhiệm chính |
|-------|---------|-------------------|
| **Patient** | Nạn nhân bị rắn cắn | - Báo cáo sự cố<br>- Thực hiện sơ cứu theo hướng dẫn<br>- Chụp ảnh rắn và vết cắn<br>- Nhấn SOS khi cần thiết |
| **Mobile App** | Giao diện người dùng | - Hướng dẫn sơ cứu<br>- Hiển thị thông tin AI<br>- Tracking real-time<br>- Quản lý thanh toán |
| **AI System** | Hệ thống trí tuệ nhân tạo | - Nhận diện loài rắn<br>- Đánh giá mức độ nghiêm trọng<br>- Đề xuất biện pháp xử lý |
| **Backend System** | Hệ thống xử lý nghiệp vụ | - Lưu trữ dữ liệu<br>- Matching Rescuer<br>- GPS tracking<br>- Xử lý thanh toán |
| **Rescuer/Supporter (SnakeAid)** | Đội cứu hộ của SnakeAid | - Nhận yêu cầu SOS<br>- Di chuyển đến hiện trường<br>- Hỗ trợ sơ cứu nâng cao<br>- Bắt rắn (nếu còn)<br>- Đưa Patient đến bệnh viện |
| **Snake Expert** | Chuyên gia rắn | - Xác minh loài rắn (nếu cần)<br>- Tư vấn từ xa cho Rescuer<br>- Cập nhật hướng dẫn điều trị |
| **Hospital/Treatment Facility** | Cơ sở y tế | - Cung cấp thông tin huyết thanh<br>- Tiếp nhận bệnh nhân |

---

## 🔄 Tổng quan Business Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                    EMERGENCY RESCUE FLOW - OVERVIEW                 │
└─────────────────────────────────────────────────────────────────────┘

[GIAI ĐOẠN 1: SƠ CỨU BAN ĐẦU]
Patient bị rắn cắn
    ↓
Mở app → Chọn "Tôi bị rắn cắn - Cần trợ giúp"
    ↓
Hiển thị hướng dẫn sơ cứu ngay lập tức (KHÔNG CẦN SOS)
  • Băng ép vết cắn
  • Giữ yên, hạn chế vận động
  • KHÔNG rạch vết thương, KHÔNG hút nọc
    ↓
Patient thực hiện sơ cứu theo hướng dẫn
    ↓
──────────────────────────────────────────────────────────────────────

[GIAI ĐOẠN 2: KÍCH HOẠT SOS]
Patient nhấn nút SOS "Gọi cứu hộ khẩn cấp"
    ↓
Tạo SOSRequest + SnakebiteIncident
    ↓
──────────────────────────────────────────────────────────────────────

[GIAI ĐOẠN 3: NHẬN DIỆN RẮN & ĐÁNH GIÁ]
Patient chụp ảnh rắn (nếu có thể)
    ↓
AI nhận diện loài rắn
    ↓
    ├─ Nếu nhận diện được → Hiển thị thông tin rắn + độc tính
    └─ Nếu UNKNOWN → Yêu cầu trả lời câu hỏi (SnakeQuestion)
    ↓
Patient chụp ảnh vết cắn + nhập triệu chứng
    ↓
AI đánh giá mức độ nghiêm trọng (train từ ảnh vết cắn)
    ↓
Hiển thị hướng dẫn sơ cứu theo loài rắn (Admin/Expert cấu hình)
    ↓
[Optional] Expert xác minh ảnh vết cắn (BiteVerification)
    ↓
──────────────────────────────────────────────────────────────────────

[GIAI ĐOẠN 4: MATCHING RESCUER]
Hệ thống tìm Rescuer phù hợp (10km)
    ↓
Gửi notification đến top 3 Rescuer (timeout 2 phút)
    ↓
    ├─ Nếu có Rescuer accept → Tạo RescueRequest
    └─ Nếu không accept → Tạo RescueRequest mới, match Rescuer khác
    ↓
Kích hoạt GPS tracking → Hiển thị map 2 bên (Patient + Rescuer)
    ↓
[Trong lúc chờ] Patient/Rescuer có thể chat/video call Expert tư vấn
    ↓
Rescuer di chuyển → Patient theo dõi real-time
    ↓
Rescuer đến → Thực hiện các bước chuẩn bị (PreparationStep/IncidentPreparation):
  • Kiểm tra vết cắn
  • Hỗ trợ sơ cứu nâng cao
  • Băng ép đúng cách
  • Bắt rắn (nếu còn)
    ↓
Chọn bệnh viện ưu tiên: Có huyết thanh + Gần > Gần > Có huyết thanh
    ↓
Đưa Patient đến bệnh viện
    ↓
Rescuer chụp ảnh bằng chứng khi đến bệnh viện (xác nhận hoàn tất)
    ↓
──────────────────────────────────────────────────────────────────────

[GIAI ĐOẠN 5: THANH TOÁN & FEEDBACK]
Rescuer xác nhận hoàn tất (có ảnh bằng chứng)
    ↓
Patient thanh toán dịch vụ Rescuer (575,000 VNĐ)
    ↓
Hệ thống phân chia:
  • 85% (489K) → Wallet Rescuer (có thể rút)
  • 10% (57.5K) → Platform
  • 5% (28.5K) → Quỹ bảo hiểm
    ↓
Patient đánh giá Rescuer (rating + review)
    ↓
[Nếu có tư vấn Expert] Patient thanh toán phí tư vấn riêng
    ↓
Phân chia phí tư vấn:
  • 85% → Wallet Expert (có thể rút)
  • 10% → Platform
  • 5% → Quỹ
    ↓
Patient đánh giá Expert (nếu có tư vấn)
    ↓
Lưu lịch sử vào hồ sơ sức khỏe
    ↓
[KẾT THÚC]
```

---

## 📊 Entities & Database Tables

### Core Entities từ DBML

#### 1. User Management
```
User (user_id) ← người dùng chung
├─ Role (role_id) → vai trò (Patient, Rescuer, Expert, Admin)
├─ UserRole → mapping nhiều-nhiều
└─ Wallet (wallet_id) → ví điện tử cho thanh toán
```

#### 2. Domain Roles
```
Patient (patient_id) → người bị rắn cắn
SnakeRescuer (rescuer_id) → đội cứu hộ
├─ RescuerSkill → kỹ năng đặc biệt
└─ RescuerSkillMap → mapping rescuer-skill

SnakeExpert (expert_id) → chuyên gia rắn
```

#### 3. Incident Core
```
SnakebiteIncident (incident_id) → sự cố rắn cắn chính
├─ patient_id → ai bị cắn
├─ snake_species_id → loài rắn (nullable - có thể chưa xác định)
└─ Các thông tin: location, timestamp, severity_level
```

#### 4. SOS & Rescue
```
SOSRequest (sos_id) → yêu cầu cứu hộ khẩn cấp
├─ incident_id → liên kết với SnakebiteIncident
└─ status: PENDING | ACCEPTED | IN_PROGRESS | COMPLETED | RETRY

RescueRequest (rescue_request_id) → chi tiết cứu hộ (có thể nhiều nếu retry)
├─ sos_id → từ SOSRequest
├─ rescuer_id → Rescuer được assigned (NULL nếu chưa match)
├─ attempt_number → số lần thử (1, 2, 3...)
├─ RescueTracking → theo dõi vị trí real-time
└─ RescueReport → báo cáo kết quả + ảnh bằng chứng
```

#### 5. AI & Verification
```
SnakeSpecies (snake_species_id) → database loài rắn
└─ SnakeImage → hình ảnh training cho AI

IncidentMedia (media_id) → ảnh/video từ Patient
├─ incident_id
├─ media_type_id (IMAGE | VIDEO)
└─ BiteVerification → Expert xác minh
    ├─ media_id
    └─ expert_id
```

#### 6. Symptoms & Assessment
```
Symptom (symptom_id) → danh mục triệu chứng
└─ SymptomReport (symptom_report_id) → triệu chứng thực tế
    ├─ incident_id
    └─ symptom_id
```

#### 7. Medical Support
```
PreparationStep (step_id) → các bước sơ cứu/chuẩn bị
└─ SnakePreparation → mapping snake_species → preparation_steps
    (Admin/Expert cấu hình hướng dẫn cho từng loài rắn)

IncidentPreparation (incident_preparation_id) → ghi nhận bước đã thực hiện
├─ incident_id
├─ step_id → bước nào
├─ rescuer_id → ai thực hiện
└─ completed_at → khi nào

TreatmentFacility (facility_id) → bệnh viện/trạm y tế
└─ Antivenom (antivenom_id) → loại huyết thanh (theo snake_species_id)
    └─ FacilityAntivenom → mapping facility-antivenom
    
SnakeImage → ảnh rắn dùng để train AI
```

#### 8. Expert Consultation (trong lúc chờ Rescuer)
```
ExpertConsultation (consultation_id) → phiên tư vấn
├─ incident_id
├─ expert_id
├─ consultation_type → CHAT | VIDEO_CALL
├─ initiated_by → PATIENT | RESCUER
└─ ConsultationHistory → lịch sử trao đổi
    ├─ patient_id
    ├─ expert_id
    └─ rescuer_id (nullable - nếu Rescuer tham gia)
```

#### 9. Payment & Wallet
```
Payment (payment_id) → thanh toán
├─ incident_id
├─ patient_id
├─ payment_type_id → phương thức thanh toán
├─ wallet_id
└─ Invoice → hóa đơn
    └─ PlatformFee → phí nền tảng

Wallet (wallet_id) → ví điện tử (cho Rescuer/Expert)
├─ user_id
├─ balance → số dư có thể rút
├─ total_earned → tổng thu nhập
└─ can_withdraw → có thể rút tiền
```

---

## 🔥 BUSINESS FLOW CHI TIẾT - PHẦN 1/3

### GIAI ĐOẠN 1: SƠ CỨU BAN ĐẦU

**⏱️ Thời gian:** 10-30 giây

#### 1.1. Patient bị rắn cắn và mở app

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 1.1 | Patient | Bị rắn cắn, mở app | - | - |
| 1.2 | Patient | Chọn "Tôi bị rắn cắn - Cần trợ giúp" | - | - |
| 1.3 | Mobile App | Hiển thị hướng dẫn sơ cứu NGAY LẬP TỨC | - | FE-01, FE-02, FE-03 |
| 1.4 | Patient | Đọc và thực hiện sơ cứu theo hướng dẫn | - | - |

**Hướng dẫn sơ cứu ban đầu (FE-01, FE-02, FE-03):**
- ✅ **LÀM:** Băng ép vết cắn (compression bandage)
- ✅ **LÀM:** Giữ yên, hạn chế vận động
- ✅ **LÀM:** Tháo bỏ đồ trang sức, giày dép chật
- ❌ **KHÔNG LÀM:** Rạch vết thương
- ❌ **KHÔNG LÀM:** Hút nọc độc bằng miệng
- ❌ **KHÔNG LÀM:** Đắp lá cây, bùn đất
- ❌ **KHÔNG LÀM:** Bó chặt cầm máu

**Lưu ý:** Chưa tạo SOSRequest ở bước này, chỉ hướng dẫn sơ cứu.

---

### GIAI ĐOẠN 2: KÍCH HOẠT SOS

**⏱️ Thời gian:** 5-10 giây

#### 2.1. Patient nhấn SOS sau khi sơ cứu

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 2.1 | Patient | Nhấn nút SOS "Gọi cứu hộ khẩn cấp" | - | FE-04 |
| 2.2 | Backend | Tạo SOSRequest (status=PENDING) | `INSERT SOSRequest` | FE-05 |
| 2.3 | Backend | Tạo SnakebiteIncident | `INSERT SnakebiteIncident` | - |
| 2.4 | Backend | Lưu GPS location | `INSERT UserLocationHistory` | - |

```sql
-- Khi nhấn SOS mới tạo SOSRequest
INSERT INTO SOSRequest (
    sos_id, incident_id, location_lat, location_lng,
    status, priority_level, created_at
) VALUES (?, ?, ?, ?, 'PENDING', 'HIGH', NOW());

INSERT INTO SnakebiteIncident (
    incident_id, patient_id, incident_time,
    location_lat, location_lng, created_at
) VALUES (?, ?, NOW(), ?, ?, NOW());
```

---

### GIAI ĐOẠN 3: NHẬN DIỆN RẮN & ĐÁNH GIÁ

**⏱️ Thời gian:** 1-2 phút

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 2.1 | Patient | Chụp ảnh vết cắn + nhập triệu chứng | `INSERT SymptomReport` | FE-09, FE-10 |
| 2.2 | AI System | Phân tích và tính điểm (0-100) | `INSERT SnakebiteIncident` | FE-15 |
| 2.3 | AI System | Phân loại: MILD \| MODERATE \| SEVERE \| CRITICAL | `UPDATE severity_level` | FE-17 |
| 2.4 | Mobile App | **IF** SEVERE/CRITICAL → Hiển thị nút SOS | - | FE-16, FE-04 |

**AI Scoring:**
```
score = 0.4×venom + 0.3×swelling + 0.2×symptoms + 0.1×time
```

---

### GIAI ĐOẠN 3A: KÍCH HOẠT SOS & MATCHING RESCUER

**⏱️ Matching:** 10-20 giây | **Di chuyển:** 5-15 phút

#### 3A.1. Patient nhấn SOS

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 3.1 | Patient | Nhấn nút SOS | - | FE-04 |
| 3.2 | Backend | Tạo SOSRequest | `INSERT SOSRequest` | FE-05 |
| 3.3 | Backend | Lưu GPS location | `INSERT UserLocationHistory` | - |

```sql
INSERT INTO SOSRequest (
    sos_id, incident_id, location_lat, location_lng,
    status, priority_level, created_at
) VALUES (...);
```

#### 3A.2. Matching Rescuer

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 4.1 | Backend | Query Rescuer trong 10km, rating ≥ 4.0 | `SELECT SnakeRescuer` | - |
| 4.2 | Backend | Sắp xếp: distance → rating → response_time | - | - |
| 4.3 | Backend | Gửi notification đến top 3 | - | - |
| 4.4 | Rescuer | Xem chi tiết + Nhấn "Chấp nhận" | - | FE-01, FE-02 (Rescuer) |
| 4.5 | Backend | Cập nhật SOS status = ACCEPTED | `UPDATE SOSRequest` | FE-06 (Rescuer) |
| 4.6 | Backend | Tạo RescueRequest | `INSERT RescueRequest` | - |

```sql
-- Matching query
SELECT rescuer_id, rating, distance_km
FROM SnakeRescuer
WHERE is_available = TRUE 
  AND distance_km <= 10 
  AND rating >= 4.0
ORDER BY distance_km, rating DESC
LIMIT 3;

-- Tạo Rescue Request
INSERT INTO RescueRequest (
    rescue_request_id, sos_id, rescuer_id,
    status, estimated_arrival_time
) VALUES (...);
```

#### 3A.3. GPS Tracking Real-time

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 5.1 | Backend | Kích hoạt tracking session | `INSERT RescueTracking` | FE-05 |
| 5.2 | Rescuer | Chia sẻ vị trí (mỗi 5s) | `UPDATE current_lat/lng` | FE-18 (Rescuer) |
| 5.3 | Patient App | Hiển thị Rescuer trên map + ETA | - | FE-24, FE-25, FE-26 |

```sql
-- Tracking
INSERT INTO RescueTracking (
    tracking_id, rescue_request_id, start_time, status
) VALUES (...);

-- Update vị trí
UPDATE RescueTracking 
SET current_lat = ?, current_lng = ?, last_updated = NOW()
WHERE tracking_id = ?;
```

---

## 🔥 BUSINESS FLOW CHI TIẾT - PHẦN 2/3

### GIAI ĐOẠN 3A.4: RESCUER ĐẾN & XỬ LÝ

**⏱️ Thời gian:** 10-30 phút

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 6.1 | Rescuer | Đến địa điểm | `UPDATE status = ARRIVED` | FE-20 (Rescuer) |
| 6.2 | Patient | Nhận thông báo "Đội cứu hộ đã đến" | - | FE-25 |
| 6.3 | Rescuer | Hỗ trợ sơ cứu + Bắt rắn (nếu có) | `INSERT RescuerSnakeHandlingHistory` | FE-09, FE-10 (Rescuer) |
| 6.4 | Rescuer | [Optional] Liên hệ Expert nếu cần | - | FE-12 (Rescuer) |
| 6.5 | Rescuer | Đưa Patient đến BV (nếu cần) | `SELECT TreatmentFacility` | - |
| 6.6 | Rescuer | Đánh dấu "Hoàn thành" | `UPDATE status = COMPLETED` | FE-07 (Rescuer) |
| 6.7 | Rescuer | Viết báo cáo | `INSERT RescueReport` | FE-15, FE-16 (Rescuer) |

```sql
-- Hoàn tất rescue
UPDATE RescueRequest 
SET status = 'COMPLETED',
    completed_at = NOW(),
    actual_duration_minutes = TIMESTAMPDIFF(MINUTE, accepted_at, NOW())
WHERE rescue_request_id = ?;

-- Tạo báo cáo
INSERT INTO RescueReport (
    report_id, rescue_request_id,
    snake_caught, patient_transported, hospital_id,
    notes, created_at
) VALUES (...);
```

---

### GIAI ĐOẠN 3B: TÌM BỆNH VIỆN (NHẸ/TRUNG BÌNH)

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 7.1 | Patient | Chọn "Tìm bệnh viện gần nhất" | - | FE-06 |
| 7.2 | Backend | Query BV có huyết thanh trong 20km | `SELECT TreatmentFacility` | FE-07, FE-08 |
| 7.3 | Mobile App | Hiển thị map + danh sách | - | FE-06 |
| 7.4 | Patient | Chọn BV → Chỉ đường / Gọi điện | `UPDATE selected_hospital_id` | FE-11 |

---

## 💰 PAYMENT FLOW - EMERGENCY SOS

### Đặc điểm thanh toán Emergency

**❗ KHÔNG CỌC TRƯỚC - Ưu tiên y tế khẩn cấp**

| Tiêu chí | Emergency SOS |
|----------|---------------|
| **Tình huống** | Bị rắn cắn (y tế khẩn cấp) |
| **Cọc trước** | ❌ KHÔNG cọc |
| **Thanh toán** | 100% sau khi hoàn tất |
| **Lý do** | Ưu tiên cứu người trước |
| **Phí dịch vụ** | 575,000 VNĐ |

### Quy trình thanh toán

```
┌─────────────┐                ┌─────────────┐                ┌──────────────┐
│   PATIENT   │────────────────│  PLATFORM   │────────────────│   RESCUER    │
└──────┬──────┘                └──────┬──────┘                └──────┬───────┘
       │                              │                              │
       │ 1. SOS Alert (KHÔNG CỌC)    │                              │
       ├─────────────────────────────>│                              │
       │                              │ 2. Match Rescuer             │
       │                              ├─────────────────────────────>│
       │                              │                              │
       │                              │ 3. Rescuer chấp nhận         │
       │                              │<─────────────────────────────┤
       │                              │                              │
       │ 4. Rescuer đến & xử lý       │                              │
       │                              │<─────────────────────────────┤
       │                              │                              │
       │ 5. Hoàn tất cứu hộ           │                              │
       │                              │<─────────────────────────────┤
       │                              │                              │
       │ 6. THANH TOÁN SAU 100%       │                              │
       │    575,000 VNĐ                │                              │
       ├─────────────────────────────>│                              │
       │                              │                              │
       │                              │ 7. Phân chia:                │
       │                              │   • 85% (489K) → Rescuer     │
       │                              │   • 10% (57.5K) → Platform   │
       │                              │   • 5% (28.5K) → Bảo hiểm    │
       │                              ├─────────────────────────────>│
       │                              │                              │
       │ 8. Nhận hóa đơn              │                              │
       │<─────────────────────────────┤                              │
```

### Phân chia doanh thu

| Bên nhận | Tỷ lệ | Số tiền | Mục đích |
|----------|-------|---------|----------|
| **Rescuer** | 85% | 489,000 VNĐ | Thu nhập từ dịch vụ khẩn cấp |
| **Platform** | 10% | 57,500 VNĐ | Phí vận hành hệ thống |
| **Quỹ bảo hiểm** | 5% | 28,500 VNĐ | Bảo hiểm tai nạn cho Rescuer |
| **TỔNG** | 100% | 575,000 VNĐ | |

### Database Operations

```sql
-- Tạo payment sau khi hoàn tất
INSERT INTO Payment (
    payment_id, incident_id, patient_id,
    payment_type_id, wallet_id,
    amount, currency,
    status, created_at
) VALUES (?, ?, ?, ?, ?, 575000, 'VND', 'PENDING', NOW());

-- Tạo invoice
INSERT INTO Invoice (
    invoice_id, payment_id,
    subtotal, platform_fee, insurance_fee,
    total_amount, issued_at
) VALUES (
    ?, ?, 
    500000, 57500, 28500,
    575000, NOW()
);

-- Platform fee
INSERT INTO PlatformFee (
    fee_id, payment_id,
    fee_percentage, fee_amount
) VALUES (?, ?, 10, 57500);

-- Cập nhật payment status
UPDATE Payment 
SET status = 'COMPLETED', completed_at = NOW()
WHERE payment_id = ?;

-- Chuyển tiền vào ví Rescuer
UPDATE Wallet 
SET balance = balance + 489000
WHERE wallet_id = (SELECT wallet_id FROM User WHERE user_id = ?);
```

### Timeline thanh toán

```
[T0] Rescuer hoàn thành cứu hộ
  ↓
[T1] Rescuer nhấn "Hoàn thành" → Patient nhận notification
  ↓
[T2] Patient xác nhận và thanh toán 575,000 VNĐ
  ↓
[T3] Hệ thống xác minh payment → Gửi hóa đơn cho Patient
  ↓
[T4] Phân chia tiền:
     ├─ 489,000 VNĐ → Ví Rescuer (trong 5-10 phút)
     ├─  57,500 VNĐ → Platform
     └─  28,500 VNĐ → Quỹ bảo hiểm
  ↓
[T5] Patient đánh giá Rescuer (rating + review)
  ↓
[T6] Lưu vào lịch sử giao dịch
```

### Xử lý trường hợp đặc biệt

| Trường hợp | Xử lý | Database Action |
|------------|-------|-----------------|
| **Patient không thanh toán trong 48h** | Khóa tài khoản + gửi reminder | `UPDATE User SET status = 'LOCKED'` |
| **Patient khiếu nại dịch vụ** | Admin review → Có thể refund một phần | `INSERT Dispute` |
| **Rescuer không hoàn thành** | Không tính phí, tìm Rescuer khác | `UPDATE RescueRequest SET status = 'CANCELLED'` |

---

## ⭐ FEEDBACK & RATING

### Giai đoạn cuối: Patient đánh giá

| Bước | Actor | Hành động | Database | Features |
|------|-------|-----------|----------|----------|
| 8.1 | Patient | Đánh giá Rescuer (1-5 sao) | `INSERT Feedback` | FE-27 (Rescuer) |
| 8.2 | Patient | Viết review (optional) | - | - |
| 8.3 | Backend | Cập nhật rating Rescuer | `UPDATE SnakeRescuer.rating` | - |
| 8.4 | Backend | Hiển thị rating cho Rescuer | - | FE-27 (Rescuer) |

```sql
-- Lưu feedback
INSERT INTO Feedback (
    feedback_id, incident_id, patient_id,
    rating, review_text,
    created_at
) VALUES (?, ?, ?, 5, 'Rescuer rất nhiệt tình...', NOW());

-- Cập nhật rating trung bình của Rescuer
UPDATE SnakeRescuer 
SET rating = (
    SELECT AVG(f.rating) 
    FROM Feedback f
    JOIN SnakebiteIncident si ON f.incident_id = si.incident_id
    JOIN RescueRequest rr ON si.incident_id = rr.sos_id
    WHERE rr.rescuer_id = ?
),
total_reviews = total_reviews + 1
WHERE rescuer_id = ?;
```

---

## 📊 WORKFLOW SUMMARY

### Các giai đoạn chính

| # | Giai đoạn | Thời gian | Actors | Database Tables |
|---|-----------|-----------|--------|-----------------|
| 1 | Phát hiện & Xử lý ban đầu | 30-60s | Patient, AI | SnakebiteIncident, IncidentMedia |
| 2 | Đánh giá mức độ nghiêm trọng | 30-45s | AI, Backend | SymptomReport, PatientSnakebiteHistory |
| 3A | SOS & Matching Rescuer | 10-20s | Backend, Rescuer | SOSRequest, RescueRequest |
| 3A+ | GPS Tracking | 5-15min | Rescuer | RescueTracking |
| 3A++ | Rescuer xử lý | 10-30min | Rescuer | RescueReport, RescuerSnakeHandlingHistory |
| 3B | Tìm bệnh viện | 20-30s | Patient | TreatmentFacility, FacilityAntivenom |
| 4 | Thanh toán | 1-2min | Patient, Platform | Payment, Invoice, PlatformFee |
| 5 | Feedback | 30s | Patient | Feedback |

### Key Metrics

**SLA (Service Level Agreement):**
- ⏱️ **SOS Response Time:** < 2 phút (từ lúc nhấn SOS đến có Rescuer chấp nhận)
- ⏱️ **Rescuer Arrival Time:** < 15 phút (trong bán kính 10km)
- ⏱️ **AI Recognition Time:** < 5 giây
- ⏱️ **Payment Processing:** < 10 phút

**Success Criteria:**
- ✅ 95% SOS requests được match với Rescuer trong 2 phút
- ✅ 90% Rescuer đến trong thời gian dự kiến (ETA ±20%)
- ✅ 85% AI recognition accuracy
- ✅ 98% payment success rate

---

## 🔗 Liên kết với Requirements

### Features được implement

| Feature ID | Tên | Actor | Giai đoạn |
|------------|-----|-------|-----------|
| FE-01 | Hướng dẫn sơ cứu khẩn cấp | Patient | 1 |
| FE-02 | Hướng dẫn băng ép có hình ảnh | Patient | 1 |
| FE-03 | Cảnh báo hành động cấm kỵ | Patient | 1 |
| FE-04 | Nút SOS | Patient | 3A |
| FE-05 | Chia sẻ vị trí real-time | Patient | 3A |
| FE-06 | Định vị bệnh viện | Patient | 3B |
| FE-07 | Tính khoảng cách và thời gian | Patient | 3B |
| FE-08 | Thông tin huyết thanh | Patient | 3B |
| FE-09 | Nhập triệu chứng | Patient | 2 |
| FE-10 | Chụp ảnh vết cắn | Patient | 2 |
| FE-11 | Lưu lịch sử | Patient | All |
| FE-12 | AI nhận diện rắn | AI | 1 |
| FE-13 | Hiển thị độc tính | AI | 1 |
| FE-14 | Đề xuất xử lý | AI | 1 |
| FE-15 | Đánh giá mức độ nghiêm trọng | AI | 2 |
| FE-16 | Cảnh báo khẩn cấp | Patient | 2 |
| FE-17 | Phân loại level | AI | 2 |
| FE-24 | Theo dõi Rescuer real-time | Patient | 3A |
| FE-25 | Nhận thông báo | Patient | 3A |
| FE-26 | Hiển thị ETA | Patient | 3A |
| FE-01 (R) | Nhận cảnh báo cứu hộ | Rescuer | 3A |
| FE-06 (R) | Chấp nhận yêu cầu | Rescuer | 3A |
| FE-18 (R) | Cập nhật vị trí | Rescuer | 3A |
| FE-20 (R) | Gửi trạng thái | Rescuer | 3A |

### Database Tables Coverage

**✅ Được sử dụng trong Emergency Flow:**
- User, Role, UserRole
- Patient, SnakeRescuer, SnakeExpert
- Wallet, PaymentType
- UserLocationHistory
- SnakeSpecies, SnakeImage
- SnakebiteIncident
- SOSRequest
- RescueRequest, RescueTracking, RescueReport
- RescuerSnakeHandlingHistory
- SnakeQuestion (optional)
- Symptom, SymptomReport
- IncidentMediaType, IncidentMedia
- BiteVerification (optional)
- PatientSnakebiteHistory
- ExpertConsultation (optional)
- ConsultationHistory (optional)
- Antivenom, TreatmentFacility, FacilityAntivenom
- Feedback
- Payment, Invoice, PlatformFee
- Content, CommunityAlert (optional)

---

## 🎓 Business Rules

### Rules cho Emergency SOS

1. **Matching Rules:**
   - Chỉ match Rescuer trong bán kính 10km
   - Rescuer phải có rating >= 4.0
   - Rescuer phải online và available
   - Ưu tiên: Khoảng cách → Rating → Response time

2. **Payment Rules:**
   - Emergency SOS: KHÔNG cọc trước
   - Thanh toán 100% sau khi hoàn tất
   - Phân chia: 85% Rescuer, 10% Platform, 5% Bảo hiểm
   - Timeout thanh toán: 48 giờ

3. **Timeout Rules:**
   - Rescuer response timeout: 2 phút
   - Nếu không có Rescuer: mở rộng bán kính lên 20km
   - Payment reminder: 24h, 48h
   - Account lock: sau 48h không thanh toán

4. **Safety Rules:**
   - Patient phải nhập tối thiểu 1 triệu chứng
   - Nếu CRITICAL: tự động gợi ý gọi 115
   - Rescuer có quyền từ chối nếu nguy hiểm quá mức

---

## 📝 Notes cho Development Team

### Priority Implementation

**Phase 1 (MVP):**
- ✅ Giai đoạn 1-2: AI recognition + Severity assessment
- ✅ Giai đoạn 3A.1-3A.2: SOS + Matching
- ✅ Giai đoạn 3A.3: GPS Tracking
- ✅ Giai đoạn 4: Payment flow

**Phase 2:**
- Giai đoạn 3A.4-3A.5: Rescuer actions
- Giai đoạn 3B: Hospital finding
- Feedback system

**Phase 3:**
- Expert consultation integration
- Advanced analytics
- Community alerts

### Technical Considerations

1. **Real-time tracking:** WebSocket hoặc Firebase Realtime Database
2. **AI Model:** CNN cho snake recognition, severity classification
3. **Payment Gateway:** VNPay, Momo, ZaloPay integration
4. **Maps:** Google Maps API hoặc Mapbox
5. **Notification:** Firebase Cloud Messaging (FCM)

---

## ✅ Kết luận

Emergency Rescue Workflow là luồng nghiệp vụ **cốt lõi** của hệ thống SnakeAid, đảm bảo:

1. ⚡ **Phản ứng nhanh** khi có sự cố rắn cắn
2. 🤖 **AI hỗ trợ** nhận diện và đánh giá
3. 🚑 **Kết nối Rescuer** trong thời gian ngắn
4. 📍 **Theo dõi real-time** an toàn cho Patient
5. 💰 **Thanh toán công bằng** cho tất cả các bên

Workflow được thiết kế dựa trên:
- ✅ Requirements từ Main-Flow.md
- ✅ Swimlane Diagram chi tiết
- ✅ Payment Flow specifications
- ✅ Database schema từ DBML
- ✅ Major Features Summary

**Lưu ý:** Đây là Emergency flow (bị rắn cắn), khác với Rescue Request flow (gọi bắt rắn có cọc trước 150K).

---

## 📝 BỔ SUNG: EXPERT CONSULTATION TRONG LÚC CHỜ

### Khi nào cần Expert?

| Tình huống | Ai liên hệ | Mục đích |
|------------|-----------|----------|
| AI không nhận diện được rắn | Patient | Xác định loài rắn qua mô tả |
| Vết cắn phức tạp | Patient | Đánh giá mức độ nguy hiểm |
| Rescuer gặp khó khăn hiện trường | Rescuer | Tư vấn cách xử lý |
| Loài rắn hiếm gặp | Rescuer | Hỗ trợ nhận diện chính xác |

### Quy trình Expert Consultation

```sql
-- Tạo consultation request
INSERT INTO ExpertConsultation (
    consultation_id, incident_id, expert_id,
    consultation_type, initiated_by,
    status, started_at
) VALUES (?, ?, ?, 'VIDEO_CALL', 'PATIENT', 'ACTIVE', NOW());

-- Lưu lịch sử
INSERT INTO ConsultationHistory (
    consultation_history_id, consultation_id,
    patient_id, expert_id, rescuer_id,
    duration_minutes, created_at
) VALUES (?, ?, ?, ?, NULL, 15, NOW());

-- Payment cho Expert
INSERT INTO Payment (
    payment_id, incident_id, patient_id,
    payment_for, amount, status
) VALUES (?, ?, ?, 'EXPERT_CONSULTATION', 200000, 'PENDING');
```

### Phân chia phí tư vấn Expert

| Bên nhận | Tỷ lệ | Số tiền (200K) | Mục đích |
|----------|-------|----------------|----------|
| **Expert** | 85% | 170,000 VNĐ → Wallet | Thu nhập tư vấn |
| **Platform** | 10% | 20,000 VNĐ | Phí vận hành |
| **Quỹ** | 5% | 10,000 VNĐ | Bảo hiểm |

---

## 🏥 BỔ SUNG: ƯU TIÊN BỆNH VIỆN

### Logic chọn bệnh viện

```sql
-- Query bệnh viện với ưu tiên
SELECT 
    tf.facility_id,
    tf.name,
    tf.distance_km,
    COUNT(fa.antivenom_id) AS has_antivenom_count,
    CASE 
        WHEN COUNT(fa.antivenom_id) > 0 AND tf.distance_km < 10 THEN 1  -- Có huyết thanh + Gần
        WHEN tf.distance_km < 10 THEN 2                                  -- Gần
        WHEN COUNT(fa.antivenom_id) > 0 THEN 3                           -- Có huyết thanh
        ELSE 4                                                            -- Khác
    END AS priority
FROM TreatmentFacility tf
LEFT JOIN FacilityAntivenom fa ON tf.facility_id = fa.facility_id
LEFT JOIN Antivenom av ON fa.antivenom_id = av.antivenom_id 
    AND av.snake_species_id = :snake_species_id
WHERE tf.distance_km < 30
GROUP BY tf.facility_id
ORDER BY priority ASC, tf.distance_km ASC
LIMIT 5;
```

**Giải thích ưu tiên:**
1. **Priority 1:** Có huyết thanh phù hợp + Gần (< 10km) → **TỐT NHẤT**
2. **Priority 2:** Gần nhưng không có huyết thanh → Xử lý ban đầu, chuyển viện sau
3. **Priority 3:** Có huyết thanh nhưng xa → Đáng để đi xa
4. **Priority 4:** Xa và không có huyết thanh → Phương án cuối

---

## 📸 BỔ SUNG: XÁC NHẬN BẰNG ẢNH

### Rescuer phải chụp ảnh bằng chứng

| Thời điểm | Loại ảnh | Lưu vào | Mục đích |
|-----------|----------|---------|----------|
| Khi bắt được rắn | Ảnh rắn đã bắt | RescueReport | Xác nhận loài rắn |
| Khi đến bệnh viện | Ảnh Patient + Biển bệnh viện | RescueReport | Chứng minh đã hoàn tất |
| Sau sơ cứu | Ảnh vết cắn đã băng | IncidentMedia | Theo dõi tiến triển |

```sql
-- Lưu ảnh bằng chứng
INSERT INTO RescueReport (
    report_id, rescue_request_id,
    snake_caught, snake_image_url,
    patient_transported, hospital_arrival_image_url,
    notes, completed_at
) VALUES (
    ?, ?, 
    TRUE, 's3://bucket/snake_captured_123.jpg',
    TRUE, 's3://bucket/hospital_arrival_123.jpg',
    'Đã đưa patient đến BV Chợ Rẫy', NOW()
);
```

---

## 💳 BỔ SUNG: WALLET & RÚT TIỀN

### Rescuer/Expert Wallet

```sql
-- Sau khi thanh toán thành công
UPDATE Wallet 
SET balance = balance + 489000,           -- 85% của 575K
    total_earned = total_earned + 489000,
    last_updated = NOW()
WHERE user_id = :rescuer_id;

-- Rescuer rút tiền
INSERT INTO WithdrawalRequest (
    withdrawal_id, user_id, amount,
    bank_account, status, requested_at
) VALUES (?, ?, 489000, '1234567890', 'PENDING', NOW());

-- Sau khi admin duyệt
UPDATE Wallet 
SET balance = balance - 489000
WHERE user_id = :rescuer_id;

UPDATE WithdrawalRequest 
SET status = 'COMPLETED', 
    completed_at = NOW()
WHERE withdrawal_id = ?;
```

### Điều kiện rút tiền

- ✅ Balance ≥ 100,000 VNĐ
- ✅ Đã xác thực tài khoản ngân hàng
- ✅ Không có tranh chấp đang chờ xử lý
- ⏱️ Thời gian xử lý: 1-3 ngày làm việc

---

## ✅ CẬP NHẬT: KẾT LUẬN MỚI

Emergency Rescue Workflow đã được cập nhật với:

1. **SOS trước tiên** → Sau đó mới nhận diện rắn (ưu tiên cứu người)
2. **AI Unknown handling** → Dùng SnakeQuestion để xác định
3. **Multiple RescueRequest** → Retry nếu không có Rescuer accept
4. **Expert Consultation** → Trong lúc chờ, cả Patient/Rescuer có thể tư vấn Expert
5. **PreparationStep** → Rescuer phải làm các bước chuẩn bị trước khi đưa đi BV
6. **Hospital Priority** → Ưu tiên: Có huyết thanh + Gần > Gần > Có huyết thanh
7. **Ảnh bằng chứng** → Rescuer phải chụp ảnh xác nhận hoàn tất
8. **Wallet system** → Tiền vào wallet, Rescuer/Expert có thể rút
9. **Training data** → Ảnh vết cắn dùng để train AI

**Flow chính xác:**
```
Bị rắn cắn → Mở app → Hướng dẫn sơ cứu ban đầu → 
Patient thực hiện sơ cứu → Nhấn SOS (tạo SOSRequest) → 
Chụp rắn → AI (có unknown handling) → Chụp vết cắn → 
Triệu chứng → Match Rescuer (retry nếu cần) → 
[Optional: Chat Expert] → GPS Tracking → Rescuer đến → 
Preparation steps → Chọn BV (priority) → Đưa đến BV → 
Chụp ảnh bằng chứng → Payment (vào wallet) → Feedback → 
[Nếu có tư vấn: Payment Expert vào wallet]
```

**Điểm quan trọng:**
- ✅ Sơ cứu ban đầu TRƯỚC khi nhấn SOS
- ✅ SOSRequest chỉ được tạo KHI nhấn nút SOS
- ✅ AI recognition và đánh giá SAU khi tạo SOS

