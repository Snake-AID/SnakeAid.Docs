# TÓM TẮT 4 LUỒNG TIỀN - SNAKEAID PLATFORM

**Phiên bản:** 1.0
**Ngày tạo:** 15/12/2025
**Mục đích:** Tóm tắt nhanh các luồng thanh toán để review

---

> [!IMPORTANT]
> Payment Correction - 2026-04-08
>
> Những phần bên dưới mô tả `Member -> Rescuer`, 85% cho Rescuer, 10% platform, 5% bảo hiểm, hoặc Rescuer chia tiền cho Expert là legacy/wrong for current target-state.
>
> Target hiện tại: Incident và Catching là payment một chiều vào system/platform. Escrow + net payout + platform fee split chỉ áp dụng cho Expert Consultation.

## 📌 TỔNG QUAN 4 LUỒNG TIỀN

1.  **Member → Rescuer** (Cứu hộ rắn)
2.  **Member → Expert** (Tư vấn trực tiếp)
3.  **Rescuer → Expert** (Hỗ trợ khẩn cấp)
4.  **Member → Expert** (Tư vấn SOS - Optional)

---

## 🎬 BỐI CẢNH & TÌNH HUỐNG SỬ DỤNG

### 🤔 Tại sao cần 4 luồng tiền khác nhau?

Mỗi luồng tiền phục vụ cho một **tình huống cụ thể** của người dùng. SnakeAid không chỉ là app gọi thợ bắt rắn, mà là **hệ sinh thái hoàn chỉnh** giải quyết mọi vấn đề liên quan đến rắn.

---

### 🔴 LUỒNG 1: Khi nào dùng "Cứu hộ rắn"?

> [!NOTE]
> **Tình huống:** "Tôi phát hiện rắn trong nhà/sân vườn, cần người đến bắt giúp"

*   **Kịch bản:** Phát hiện rắn → Đặt cọc 30% → Rescuer đến bắt → Trả 70% còn lại
*   **Đặc điểm:**
    *   ✅ Có rắn **hiện hữu** tại chỗ
    *   ✅ Cần người **đến tận nơi** xử lý
    *   ✅ Thanh toán theo **kết quả** (bắt được rắn)
    *   💰 **Giá:** 500K - 8M (tùy loại rắn & địa điểm)

---

### 🟠 LUỒNG 2: Khi nào dùng "Tư vấn chuyên gia"?

> [!NOTE]
> **Tình huống:** "Tôi không chắc đây là rắn gì, có độc không? Cần tư vấn trước khi quyết định"

*   **Kịch bản:** Thấy rắn lạ → Đặt lịch tư vấn → Trả trước 100% → Video call với Expert → Nhận diện & tư vấn
*   **Đặc điểm:**
    *   ✅ Muốn **tư vấn từ xa**, không cần người đến
    *   ✅ Nhận diện loài rắn qua hình ảnh/video
    *   ✅ Hỏi về cách phòng ngừa, xử lý
    *   💰 **Giá:** 150K - 900K (tùy thời gian)

---

### 🟡 LUỒNG 3: Khi nào dùng "Hỗ trợ khẩn cấp"?

> [!IMPORTANT]
> **Tình huống:** "Tôi (Rescuer) đang ở hiện trường, gặp rắn lạ không dám bắt, cần Expert tư vấn ngay"

*   **Kịch bản:** Rescuer nhận đơn → Đến nơi gặp rắn khó → Gọi Expert hỗ trợ → Video call hướng dẫn → Bắt thành công → Chia phí
*   **Đặc điểm:**
    *   ✅ Rescuer **đang làm việc**, gặp khó khăn
    *   ✅ Cần **tư vấn ngay lập tức** (1-2 phút)
    *   ✅ Expert hướng dẫn qua video call
    *   ✅ Rescuer **chia sẻ phí** với Expert (mất 50K nhưng tiết kiệm 1.5h)
    *   💰 **Member không trả thêm tiền**

> [!TIP]
> **Tại sao Rescuer phải trả chứ không phải Platform?**
> *   ✅ Ai được hỗ trợ thì chia sẻ chi phí (công bằng)
> *   ✅ Tránh lạm dụng (Rescuer cân nhắc trước khi gọi)
> *   ✅ Platform bền vững (không lỗ tiền)
> *   ✅ Expert nhận phí xứng đáng

---

### 🔴 LUỒNG 4: Khi nào dùng "Tư vấn SOS"?

> [!CAUTION]
> **Tình huống:** "Tôi BỊ RẮN CẮN rồi! Cần chuyên gia tư vấn SƠ CỨU ngay lập tức!"

*   **Kịch bản:** Bị rắn cắn → Bấm SOS → Chọn gọi Expert (optional) → Trả 500K → Expert kết nối trong 1-2 phút → Tư vấn sơ cứu & đánh giá
*   **Đặc điểm:**
    *   ✅ Tình huống **KHẨN CẤP** - bị rắn cắn
    *   ✅ **Optional** - Member tự chọn (có thể không dùng)
    *   ✅ Phản hồi **1-2 phút** (ưu tiên cao nhất)
    *   ✅ Expert tư vấn sơ cứu + đánh giá + đưa phương án
    *   ✅ Giúp Member **yên tâm** trong lúc chờ cấp cứu
    *   💰 **Giá:** Cao hơn (500K vs 300K) do tính khẩn cấp

> [!NOTE]
> **Tại sao phải trả trước trong tình huống khẩn cấp?**
> *   ✅ Đảm bảo Expert được trả công (không bị "bùng")
> *   ✅ Tiền vào ESCROW (giữ an toàn)
> *   ✅ Thanh toán online rất nhanh (30 giây)
> *   ✅ Expert chỉ nhận tiền sau khi tư vấn xong

---

## 📊 SO SÁNH 4 TÌNH HUỐNG

| Tiêu chí | Luồng 1: Cứu hộ | Luồng 2: Tư vấn | Luồng 3: Hỗ trợ Rescuer | Luồng 4: SOS |
| :--- | :--- | :--- | :--- | :--- |
| **Ai gọi?** | Member | Member | Rescuer | Member |
| **Ai phục vụ?** | Rescuer | Expert | Expert | Expert |
| **Tình huống** | Có rắn cần bắt | Tư vấn từ xa | Rescuer gặp khó | Bị rắn cắn |
| **Độ khẩn cấp** | Bình thường | Không gấp | Khẩn cấp | **CỰC KHẨN** |
| **Phản hồi** | 10-30 phút | 5-15 phút | 1-2 phút | 1-2 phút |
| **Giá** | 500K-8M | 150K-900K | 500K (Member) | 500K |
| **Ai trả?** | Member | Member | Member (qua Rescuer) | Member |
| **Thanh toán** | 2 lần (30%+70%) | 100% trước | Tự động chia | 100% trước |

---

## 🎯 LUỒNG TIỀN PHỤC VỤ CHO AI?

```mermaid
graph TD
    P[MEMBER<br>(Người dùng)]
    R[RESCUER<br>(Thợ bắt rắn)]
    E[EXPERT<br>(Chuyên gia)]

    P -- Luồng 1<br>(Cần bắt rắn) --> R
    P -- Luồng 2 & 4<br>(Cần tư vấn) --> E
    R -- Luồng 3<br>(Cần hỗ trợ) --> E

    style P fill:#d4f1f4,stroke:#0077b6,stroke-width:2px
    style R fill:#e2f0cb,stroke:#4caf50,stroke-width:2px
    style E fill:#ffdfba,stroke:#ff9800,stroke-width:2px
```

**Giải thích:**
*   **Member → Rescuer:** Bắt rắn tại chỗ
*   **Member → Expert:** Tư vấn từ xa (thường) hoặc SOS (khẩn cấp)
*   **Rescuer → Expert:** Hỗ trợ khi đang làm việc

---

## 💰 LUỒNG 1: DỊCH VỤ CỨU HỘ RẮN

**Giá:** 500,000 - 8,700,000 đ (tùy loại rắn & địa điểm)

**Phân chia:**
*   🔧 Rescuer: **85%** (425K với đơn 500K)
*   🏢 Platform: **10%** (50K)
*   🛡️ Bảo hiểm: **5%** (25K)

**Đặc điểm:**
*   ✅ Thanh toán **2 lần:** Cọc 30% + Trả 70% sau khi hoàn thành
*   ✅ Cọc 150K trước → Rescuer chấp nhận → Hoàn thành → Trả 350K
*   ✅ Giá: 500K-2.5M (nhà), 8M-14M (KCN), 7M-10M (Resort)

> [!EXAMPLE]
> **Ví dụ đơn 500K:**
> *   **Member trả:** 500,000 đ
>     *   150,000 đ (30% cọc trước)
>     *   350,000 đ (70% sau hoàn thành)
> *   **Phân chia:**
>     *   Rescuer: 425,000 đ (85%)
>     *   Platform: 50,000 đ (10%)
>     *   Bảo hiểm: 25,000 đ (5%)

---

## 💰 LUỒNG 2: TƯ VẤN CHUYÊN GIA TRỰC TIẾP

**Giá:** 150,000 - 900,000 đ (tùy thời gian)

**Phân chia:**
*   👨‍⚕️ Expert: **90%** (270K với đơn 300K)
*   🏢 Platform: **10%** (30K)

**Đặc điểm:**
*   ✅ Thanh toán **100% TRƯỚC** khi Expert chấp nhận
*   ✅ Tiền vào **ESCROW** → Expert nhận sau khi hoàn thành
*   ✅ Gói: 150K (15 phút), 300K (30 phút), 500K (60 phút)

> [!EXAMPLE]
> **Ví dụ gói 300K:**
> 1.  Member trả: **300,000 đ** (100% trước)
> 2.  Vào **ESCROW** (giữ tạm)
> 3.  Expert tư vấn 30 phút
> 4.  **Phân chia:**
>     - Expert: 270,000 đ (90%)
>     - Platform: 30,000 đ (10%)

---

## 💰 LUỒNG 3: HỖ TRỢ KHẨN CẤP (Rescuer ↔ Expert)

**Giá:** 500,000 đ (Member trả cho Rescuer)

**Phân chia:**
*   🔧 Rescuer: **75%** (375K) ← *giảm từ 85%*
*   👨‍⚕️ Expert: **10%** (50K) ← *từ phần của Rescuer*
*   🏢 Platform: **10%** (50K)
*   🛡️ Bảo hiểm: **5%** (25K)

**Đặc điểm:**
*   ✅ **Rescuer chia sẻ 10%** phí cứu hộ cho Expert
*   ✅ Member **KHÔNG trả thêm** tiền
*   ✅ Rescuer mất 50K nhưng tiết kiệm 1.5h → kiếm lại 350K từ ca khác

> [!TIP]
> **Lý do chọn phương án này:**
> *   ✅ **Bền vững:** Platform không lỗ
> *   ✅ **Công bằng:** Người được hỗ trợ chia sẻ chi phí
> *   ✅ **Tránh lạm dụng:** Rescuer cân nhắc kỹ trước khi gọi
> *   ✅ **Động lực:** Expert nhận phí xứng đáng

> [!EXAMPLE]
> **Ví dụ:**
> *   Member trả: **500,000 đ** (như bình thường)
> *   Rescuer gọi Expert hỗ trợ
> *   **Phân chia:**
>     - Rescuer: 375,000 đ (75%) ← *mất 50K*
>     - Expert: 50,000 đ (10%) ← *từ Rescuer*
>     - Platform: 50,000 đ (10%)
>     - Bảo hiểm: 25,000 đ (5%)

---

## 💰 LUỒNG 4: TƯ VẤN KHẨN CẤP QUA SOS (Optional)

**Giá:** 500,000 đ (cao hơn tư vấn thường 300K)

**Phân chia:**
*   👨‍⚕️ Expert: **90%** (450K)
*   🏢 Platform: **10%** (50K)

**Đặc điểm:**
*   ✅ **OPTIONAL** - Member tự quyết định khi đang SOS
*   ✅ Phản hồi **1-2 phút** (nhanh gấp 5 lần)
*   ✅ **Ưu tiên cao nhất** trong hệ thống
*   ✅ Thanh toán **100% TRƯỚC** → Vào ESCROW
*   ✅ Expert tư vấn sơ cứu + đánh giá + đưa phương án

> [!EXAMPLE]
> **Tình huống sử dụng:**
> 1.  Member bị rắn cắn → Bấm SOS
> 2.  Chọn "Gọi Expert ngay" (optional)
> 3.  Thanh toán 500K → **ESCROW**
> 4.  Expert kết nối trong 1-2 phút
> 5.  Tư vấn: Sơ cứu + Đánh giá + Phương án
> 6.  **Phân chia:**
>     - Expert: 450,000 đ (90%)
>     - Platform: 50,000 đ (10%)

**So sánh với tư vấn thường:**

| Tiêu chí | Tư vấn Thường | Tư vấn SOS |
| :--- | :--- | :--- |
| **Giá** | 300K | **500K** (+67%) |
| **Phản hồi** | 5-15 phút | **1-2 phút** |
| **Ưu tiên** | Bình thường | **Cao nhất** |
| **Tình huống** | Tư vấn thông thường | **Khẩn cấp (bị cắn)** |

---

## 📊 BẢNG SO SÁNH NHANH

| Luồng | Giá | Expert | Rescuer | Platform | Bảo hiểm |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1. Cứu hộ** | 500K | - | **85%** (425K) | 10% (50K) | 5% (25K) |
| **2. Tư vấn thường** | 300K | **90%** (270K) | - | 10% (30K) | - |
| **3. Hỗ trợ Rescuer** | 500K | **10%** (50K) | **75%** (375K) | 10% (50K) | 5% (25K) |
| **4. Tư vấn SOS** | 500K | **90%** (450K) | - | 10% (50K) | - |

---

## ⚡ ĐIỂM QUAN TRỌNG CẦN NHỚ

### ✅ Cơ chế thanh toán
1.  **Cứu hộ:** Cọc 30% → Hoàn thành → Trả 70%
2.  **Tư vấn thường:** Trả 100% trước → Vào ESCROW
3.  **Hỗ trợ Rescuer:** Rescuer chia 10% cho Expert
4.  **Tư vấn SOS:** Trả 100% trước → Vào ESCROW

### 🛡️ Bảo vệ người dùng
*   ✅ Tiền giữ trong **ESCROW** cho đến khi hoàn thành
*   ✅ Có **quỹ bảo hiểm 5%** cho Rescuer
*   ✅ Chính sách **hoàn tiền rõ ràng**
*   ✅ **GPS tracking** theo dõi Rescuer

### 💳 Phương thức thanh toán
*   Ví điện tử: Momo, VNPay, ZaloPay
*   Thẻ: Visa, Mastercard, JCB
*   Internet Banking: Vietcombank, BIDV, Techcombank
*   Tiền mặt: Trực tiếp cho Rescuer

### 📈 Giá cạnh tranh
*   Cứu hộ: **Rẻ hơn 10-15%** so với thị trường
*   Tư vấn: **90% cho Expert** (cao nhất thị trường)
*   Minh bạch: **Báo giá trước**, không phát sinh

---

## 🎯 KẾT LUẬN

**4 luồng tiền được thiết kế để:**
1.  ✅ **Công bằng** - Phân chia hợp lý cho tất cả các bên
2.  ✅ **Minh bạch** - Member biết rõ giá trước khi đặt
3.  ✅ **An toàn** - ESCROW + Bảo hiểm bảo vệ người dùng
4.  ✅ **Linh hoạt** - Nhiều lựa chọn cho Member

**Thu nhập ước tính:**
*   🔧 Rescuer: **12-20M/tháng** (1-2 đơn/ngày)
*   👨‍⚕️ Expert: **8-15M/tháng** (5-10 buổi/tuần)
*   🏢 Platform: **2-4 tỷ/tháng** (40-80 đơn/ngày toàn quốc)

---

**📞 Liên hệ:**
*   Email: [payment@snakeaid.vn](mailto:payment@snakeaid.vn)
*   Hotline: 1900-xxxx-xx

**🔄 Cập nhật:** 15/12/2025
