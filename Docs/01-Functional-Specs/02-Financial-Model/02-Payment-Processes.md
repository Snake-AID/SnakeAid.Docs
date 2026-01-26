# LUỒNG TIỀN CHI TIẾT - HỆ THỐNG SNAKEAID

**Phiên bản:** 1.0
**Ngày tạo:** 14/12/2025
**Mục đích:** Làm rõ luồng tiền giữa các bên: Patient, Rescuer, Expert, và Platform

---

## 📌 TỔNG QUAN LUỒNG TIỀN

Hệ thống SnakeAid có **4 luồng thanh toán chính**:

1.  **Patient → Platform → Rescuer** (Dịch vụ cứu hộ rắn)
2.  **Patient → Platform → Expert** (Tư vấn chuyên gia trực tiếp)
3.  **Platform → Expert** hoặc **Rescuer → Expert** (Hỗ trợ khẩn cấp cho Rescuer)
4.  **Patient → Platform → Expert** (Tư vấn khẩn cấp qua SOS - Optional)

---

## 💰 CHI TIẾT CÁC LUỒNG THANH TOÁN

### 1. LUỒNG TIỀN: DỊCH VỤ CỨU HỘ RẮN

**Kịch bản:** Patient phát hiện rắn trong nhà → Yêu cầu đội cứu hộ đến bắt rắn → Rescuer thực hiện → Thanh toán

#### 1.1. Quy trình thanh toán

```mermaid
sequenceDiagram
    participant Patient
    participant Platform
    participant Rescuer
    participant Insurance as Quỹ Bảo Hiểm

    Patient->>Platform: 1. Yêu cầu cứu hộ
    Patient->>Platform: 2. CỌC TRƯỚC 150,000 VNĐ (Vào Escrow)
    Platform->>Rescuer: 3. Gửi yêu cầu (Hiển thị "Đã cọc 150K")
    Rescuer->>Platform: 4. Chấp nhận yêu cầu
    Rescuer->>Platform: 5. Hoàn thành nhiệm vụ
    Patient->>Platform: 6. TRẢ SỐ DƯ (Tổng - 150K) (Ví dụ: 425K)
    
    Note over Platform: 7. Tính tổng: 150K + 425K = 575K
    
    Platform->>Rescuer: 85% (489K)
    Platform->>Platform: 10% (57.5K)
    Platform->>Insurance: 5% (28.5K)
    
    Platform->>Patient: 8. Gửi hóa đơn
    Platform->>Rescuer: 9. Thông báo nhận tiền
```

#### 1.2. Phân chia doanh thu chi tiết - RESCUE REQUEST

> [!EXAMPLE]
> **Ví dụ đơn hàng 575,000 VNĐ:**
> *   **Cọc trước (FIXED):** 150,000 VNĐ
> *   **Số dư sau:** 425,000 VNĐ

| Bên nhận | Tỷ lệ | Số tiền | Mục đích |
| :--- | :--- | :--- | :--- |
| **Rescuer** | 85% | 489,000 VNĐ | Thu nhập dịch vụ |
| **Platform** | 10% | 57,500 VNĐ | Phí vận hành |
| **Quỹ bảo hiểm** | 5% | 28,500 VNĐ | Bảo hiểm tai nạn Rescuer |
| **TỔNG** | **100%** | **575,000 VNĐ** | |

#### 1.3. Cơ chế Tiền Cọc (Anti-Ghosting)

<details>
<summary><strong>🔍 Tại sao cần cọc 150,000 VNĐ? (Nhấn để xem chi tiết)</strong></summary>

**Rủi ro nếu không cọc:**
1.  **Địa chỉ ảo:** Rescuer đến nơi không có ai.
2.  **Khách bùng:** "Rắn chạy rồi, không trả tiền".
3.  **Hủy ngang:** Hủy khi Rescuer đang đi.

**Giải pháp:** Cọc Fixed 150K bảo vệ Rescuer (bù chi phí xăng xe/thời gian) nhưng đủ nhỏ để Patient không ngại đặt.
</details>

**Xử lý các tình huống cọc:**

| Tình huống | Xử lý tiền cọc (150K) | Số dư (425K) |
| :--- | :--- | :--- |
| **Hoàn thành** | Tính vào tổng | Patient trả thêm |
| **Patient hủy SAU khi nhận** | Rescuer nhận 100% | Không trả |
| **Patient hủy TRƯỚC khi nhận** | Hoàn 100% Patient | Không trả |
| **Rescuer không đến** | Hoàn 100% Patient + Phạt Rescuer | Không trả |
| **Rắn tự chạy mất** | Chia đôi (75K mỗi bên) | Không trả |

---

### 2. LUỒNG TIỀN: TƯ VẤN CHUYÊN GIA

**Kịch bản:** Patient đặt lịch tư vấn → Thanh toán trước → Expert tư vấn

#### 2.1. Quy trình thanh toán (THANH TOÁN TRƯỚC)

```mermaid
sequenceDiagram
    participant Patient
    participant Platform
    participant Expert

    Patient->>Platform: 1. Đặt lịch tư vấn
    Patient->>Platform: 2. THANH TOÁN 100% (300K) -> Escrow
    Platform->>Expert: 3. Gửi yêu cầu (Hiển thị "Đã thanh toán")
    Expert->>Platform: 4. Chấp nhận
    Expert-->>Patient: 5. Video Call tư vấn
    Patient->>Platform: 6. Xác nhận hoàn thành
    
    Platform->>Expert: 90% (270K)
    Platform->>Platform: 10% (30K)
    
    Platform->>Patient: 7. Gửi hóa đơn
    Platform->>Expert: 8. Thông báo nhận tiền
```

#### 2.2. Phân chia doanh thu

> [!NOTE]
> **Quy tắc:** Thanh toán 100% trước, giữ trong Escrow. Hoàn tiền nếu Expert không đến.

| Bên nhận | Tỷ lệ | Số tiền (Ví dụ 300K) |
| :--- | :--- | :--- |
| **Expert** | 90% | 270,000 VNĐ |
| **Platform** | 10% | 30,000 VNĐ |

---

### 3. LUỒNG TIỀN: HỖ TRỢ KHẨN CẤP (RESCUER ↔ EXPERT)

**Kịch bản:** Rescuer gặp rắn khó → Gọi Expert hỗ trợ → Chia sẻ phí

#### 3.1. Quy trình thanh toán (Phương án 2: Chia sẻ)

```mermaid
sequenceDiagram
    participant Rescuer
    participant Platform
    participant Expert
    
    Note over Rescuer: Đang thực hiện ca 500K
    Rescuer->>Platform: 1. Yêu cầu hỗ trợ khẩn cấp
    Platform->>Expert: 2. Kết nối ngay lập tức
    Expert-->>Rescuer: 3. Tư vấn Video Call
    Rescuer->>Platform: 4. Hoàn thành nhiệm vụ
    
    Note over Platform: Phân chia 500K từ Patient:
    Platform->>Rescuer: 75% (375K)
    Platform->>Expert: 10% (50K) [Từ phần Rescuer]
    Platform->>Platform: 10% (50K)
    Platform->>Insurance: 5% (25K)
```

#### 3.2. Tại sao Rescuer phải trả?

> [!TIP]
> **Lợi ích cho Rescuer dù mất 50K:**
> *   ✅ **An toàn:** Tránh bị rắn lạ cắn.
> *   ✅ **Tiết kiệm thời gian:** Xử lý nhanh (15p thay vì 1h) → Nhận thêm ca khác.
> *   ✅ **Uy tín:** Không bắt nhầm/xử lý sai.

---

### 4. LUỒNG TIỀN: TƯ VẤN KHẨN CẤP SOS (OPTIONAL)

**Kịch bản:** Patient bị rắn cắn → Gọi SOS → Chọn thêm Expert hỗ trợ

#### 4.1. Quy trình

```mermaid
sequenceDiagram
    participant Patient
    participant Platform
    participant Expert

    Patient->>Platform: 1. Bấm SOS
    Patient->>Platform: 2. Chọn "Gọi Expert ngay" (Optional)
    Patient->>Platform: 3. Thanh toán 500K -> Escrow
    Platform->>Expert: 4. Kết nối khẩn cấp (1-2 phút)
    Expert-->>Patient: 5. Hướng dẫn sơ cứu
    
    Note over Platform: Sau khi hoàn thành:
    Platform->>Expert: 90% (450K)
    Platform->>Platform: 10% (50K)
```

**So sánh chi phí:**

| Tiêu chí | Tư vấn Thường | SOS Khẩn Cấp |
| :--- | :--- | :--- |
| **Giá** | 300,000 đ | **500,000 đ** |
| **Phản hồi** | 5-15 phút | **1-2 phút** |
| **Ưu tiên** | Bình thường | **Cao nhất** |
| **Mục đích** | Kiến thức | **Cứu người** |

---

## 🔒 TỔNG HỢP CÁC CHÍNH SÁCH BẢO VỆ

*   🛡️ **Escrow:** Tiền luôn được giữ trung gian cho đến khi dịch vụ hoàn tất.
*   🛡️ **Bảo hiểm:** 5% doanh thu cứu hộ luôn được trích vào quỹ bảo vệ Rescuer.
*   🛡️ **Anti-Fraud:**
    *   Cọc 150K để chống đơn ảo.
    *   GPS Tracking xác minh vị trí.
    *   Rating 2 chiều.

---

**📞 Hỗ trợ thanh toán:** payment@snakeaid.vn
