# EXPERT CONSULTATION FLOW SCREENS - SNAKEAID PLATFORM

## Document Information
- **Module:** Snake Expert
- **Feature Category:** Remote Consultation (Dual Mode)
- **Total Screens:** 8 screens
- **Related Flows:** Flow 3.1 (Patient Scheduled), Flow 3.2 (Rescuer Urgent), Flow 3.3 (Consultation Session)
- **Color Scheme:** Purple Primary `#6B46C1`, Emergency Red `#DC3545` for urgent requests

---

## Flow Context

### Dual Consultation Modes

**Mode 1: Scheduled Patient Consultation (Flow 3.1)**
- Patient books appointment in advance
- Expert prepares by reviewing case materials
- Scheduled video/chat session
- Post-consultation summary and payment

**Mode 2: Urgent Rescuer Support (Flow 3.2)**
- Rescuer encounters unknown snake at scene
- Immediate expert help needed
- Real-time video/photo sharing
- Quick identification and safety guidance

**Key Features (Reference: Major-Features-Summary.md):**
- FE-10: Remote consultation for Patient via chat/video call
- FE-11: Consultation for Rescuer on complex snake handling
- FE-12: Assess patient condition and recommend emergency measures

---

## Design System

### Color Palette
```
Primary Purple:     #6B46C1  (Scheduled consultations)
Emergency Red:      #DC3545  (Urgent Rescuer requests)
Success Green:      #28A745  (Completed sessions)
Warning Amber:      #FFC107  (In-progress sessions)
Neutral Gray:       #6C757D  (Secondary elements)
White:              #FFFFFF
```

### Typography
```
Screen Titles:      24pt, Semi-bold, Purple Dark
Section Headers:    20pt, Semi-bold
Body Text:          16-18pt, Regular
Timer/Duration:     28pt, Bold (for countdown)
Status Badges:      14pt, Bold
```

---

## PART 1: PRE-CONSULTATION & REQUEST HANDLING (4 Screens)

---

### Screen 1: Urgent Rescuer Request Detail

**Screen Purpose:**  
Expert views urgent request from Rescuer who needs immediate help identifying snake at rescue scene.

**Navigation:**
- Entry: Tap urgent banner from Dashboard or notification
- Exit: Accept → Screen 5 (Live Consultation), Decline → Back to Dashboard

**Key Components:**

1. **Urgent Header Banner:**
   - Red background with pulse animation
   - "🚨 YÊU CẦU KHẨN CẤP" (24pt, Bold, White)
   - Timer: "Còn 2:45 phút để phản hồi" (countdown)

2. **Rescuer Information Card:**
   - Avatar + Name: "Đội Cứu Hộ Sài Gòn"
   - Rating: "4.9 ⭐ (234 đánh giá)"
   - Location: "Đang ở hiện trường: Quận 1, TP.HCM"
   - Distance: "2.3 km từ vị trí rắn"

3. **Snake Information Section:**
   - Large snake photo (full width, 300px height)
   - Zoom controls on image
   - AI Prediction: "Rắn độc không xác định (Độ tin cậy: 45%)" - amber badge
   - Environment: "Môi trường: Vườn nhà, ban đêm"

4. **Rescuer's Message:**
   - "Rắn có vằn đen-vàng, đầu to hình tam giác. Tôi không chắc đây là loài gì. Cần xác nhận ngay!"
   - Audio message button (if available)

5. **Quick Assessment Tools:**
   - Button: "Xem Ảnh Bổ Sung" (3 more photos)
   - Button: "Xem Vị Trí Trên Bản Đồ"

6. **Action Buttons:**
   - Primary: "Chấp Nhận & Tư Vấn Ngay" (large, red, 60px)
   - Secondary: "Từ Chối" (outlined gray)
   - Note: "Bạn sẽ nhận 500K VNĐ cho tư vấn này"

**Stitch Prompt (English):**

```
Urgent expert request screen for snake rescue platform mobile app.

URGENT BANNER (red #DC3545 background, pulse animation):
- "🚨 YÊU CẦU KHẨN CẤP" (24pt bold white)
- Countdown "Còn 2:45 phút để phản hồi" (18pt white)

RESCUER CARD (white background):
- Avatar (60px) + "Đội Cứu Hộ Sài Gòn" (20pt semi-bold)
- "4.9 ⭐ (234 đánh giá)" (14pt gray)
- Location pin + "Đang ở hiện trường: Quận 1, TP.HCM" (16pt)
- "2.3 km từ vị trí rắn" (14pt gray)

SNAKE IMAGE (full-width, 300px height):
- Large snake photo with zoom icons
- AI badge "Rắn độc không xác định (45%)" (amber)
- "Môi trường: Vườn nhà, ban đêm" below

MESSAGE CARD:
- Quote text: "Rắn có vằn đen-vàng, đầu to hình tam giác. Tôi không chắc đây là loài gì. Cần xác nhận ngay!"
- Audio icon "Tin nhắn thoại" (if available)

TWO BUTTONS:
- "Xem Ảnh Bổ Sung (3)" (outlined purple)
- "Xem Vị Trí Trên Bản Đồ" (outlined blue)

BOTTOM ACTIONS:
- Large red button "Chấp Nhận & Tư Vấn Ngay" (60px height)
- Small gray outlined "Từ Chối"
- Note "Bạn sẽ nhận 500K VNĐ cho tư vấn này" (green text)

DESIGN: Emergency interface, red urgency indicators, countdown timer, large clear photos, quick decision actions.
```

---

### Screen 2: Scheduled Consultation Preparation

**Screen Purpose:**  
Expert reviews Patient's case materials before scheduled consultation session.

**Navigation:**
- Entry: Tap consultation card from Dashboard/Calendar
- Exit: Start → Screen 5 (Live Consultation), Back → Dashboard

**Key Components:**

1. **Header:**
   - Back arrow + Title: "Chuẩn Bị Tư Vấn"
   - Time until session: "Còn 25 phút" (countdown badge, purple)

2. **Session Info Card:**
   - Patient name: "Nguyễn Văn A"
   - Appointment: "11/12/2025 - 14:00"
   - Duration: "45 phút"
   - Method: "Video Call" badge (purple)
   - Fee: "750,000 VNĐ"

3. **Patient's Case Summary:**
   - Section: "Lý Do Tư Vấn"
   - "Bị rắn cắn 3 ngày trước, vết thương sưng đỏ. Muốn xác nhận loài rắn và cách xử lý."

4. **Snake Identification Section:**
   - "Thông Tin Rắn"
   - Snake photo (200px square)
   - AI Result: "Rắn Hổ Mang Chúa (Confidence: 92%)" - green badge
   - Danger level: "CỰC ĐỘC" - red badge
   - Scientific name: "Ophiophagus hannah"

5. **Bite Wound Photos:**
   - Section: "Ảnh Vết Cắn"
   - 3 photos in horizontal scroll (150px each)
   - Dates: "08/12", "09/12", "10/12" (progression)

6. **Symptoms Timeline:**
   - "Triệu Chứng Ghi Nhận"
   - Timeline view:
     - Day 1: "Đau, sưng vùng cắn"
     - Day 2: "Tê nhẹ, khó thở"
     - Day 3: "Giảm đau, vẫn sưng"

7. **Expert Notes Field:**
   - "Ghi Chú Của Bạn" (text area)
   - Placeholder: "Thêm ghi chú chuẩn bị..."
   - Voice input button

8. **Quick Reference Links:**
   - "Xem Hướng Dẫn Xử Lý Rắn Hổ Mang"
   - "Liều Lượng Huyết Thanh Khuyến Nghị"

9. **Action Buttons:**
   - Primary: "Bắt Đầu Tư Vấn Ngay" (only if within 15min)
   - Secondary: "Lưu Ghi Chú"
   - Link: "Hủy Lịch Hẹn"

**Stitch Prompt (English):**

```
Consultation preparation screen for snake expert mobile app.

HEADER:
- Back arrow, "Chuẩn Bị Tư Vấn" (24pt), countdown badge "Còn 25 phút" (purple)

SESSION CARD (purple tint background):
- Patient "Nguyễn Văn A" (20pt bold)
- "11/12/2025 - 14:00" + "45 phút"
- "Video Call" badge (purple) + "750,000 VNĐ" (green)

CASE SUMMARY:
- "Lý Do Tư Vấn" header
- Quote: "Bị rắn cắn 3 ngày trước, vết thương sưng đỏ..."

SNAKE IDENTIFICATION:
- "Thông Tin Rắn" header
- Large snake photo (200px square)
- "Rắn Hổ Mang Chúa (92%)" green badge
- "CỰC ĐỘC" red badge
- "Ophiophagus hannah" (italic gray)

WOUND PHOTOS:
- "Ảnh Vết Cắn" header
- 3 photos horizontal scroll (150px each)
- Dates "08/12", "09/12", "10/12" below

SYMPTOMS TIMELINE:
- "Triệu Chứng Ghi Nhận" header
- Vertical timeline:
  * Day 1: "Đau, sưng vùng cắn"
  * Day 2: "Tê nhẹ, khó thở"
  * Day 3: "Giảm đau, vẫn sưng"

NOTES SECTION:
- "Ghi Chú Của Bạn" header
- Text area with mic icon
- Placeholder "Thêm ghi chú chuẩn bị..."

QUICK LINKS:
- "Xem Hướng Dẫn Xử Lý Rắn Hổ Mang" (blue link)
- "Liều Lượng Huyết Thanh Khuyến Nghị" (blue link)

BOTTOM BUTTONS:
- Large purple "Bắt Đầu Tư Vấn Ngay" (60px)
- Medium outlined "Lưu Ghi Chú"
- Small red link "Hủy Lịch Hẹn"

DESIGN: Professional medical preparation interface, comprehensive case review, AI-assisted identification display, timeline visualization.
```

---

### Screen 3: Accept Consultation Confirmation

**Screen Purpose:**  
Quick confirmation screen before starting consultation session (both scheduled and urgent).

**Navigation:**
- Entry: Tap "Bắt Đầu" from Screen 2 or "Chấp Nhận" from Screen 1
- Exit: Confirm → Screen 5 (Live Consultation), Cancel → Back

**Key Components:**

1. **Modal Overlay:**
   - Semi-transparent dark background
   - White card in center (80% width)

2. **Confirmation Header:**
   - Icon: Purple checkmark circle (if scheduled) or Red alert (if urgent)
   - Title: "Xác Nhận Bắt Đầu Tư Vấn"

3. **Session Details:**
   - For Scheduled:
     - "Tư vấn với: Nguyễn Văn A"
     - "Thời gian: 14:00 - 14:45 (45 phút)"
     - "Phương thức: Video Call"
   - For Urgent:
     - "Hỗ trợ khẩn cấp: Đội Cứu Hộ Sài Gòn"
     - "Loại: Tư vấn trực tiếp qua video"
     - "Thời gian dự kiến: 15-20 phút"

4. **Payment Info:**
   - "Phí tư vấn: 750,000 VNĐ"
   - "Trạng thái: Đã thanh toán" (green) or "Sẽ thanh toán tự động" (amber)
   - "Bạn sẽ nhận: 675,000 VNĐ (sau phí nền tảng 10%)"

5. **Checklist:**
   - ✓ Camera và micro đã kiểm tra
   - ✓ Môi trường yên tĩnh
   - ✓ Đã xem thông tin ca bệnh (if scheduled)

6. **Action Buttons:**
   - Primary: "Bắt Đầu Ngay" (purple or red, 56px height)
   - Secondary: "Hủy" (outlined gray)

**Stitch Prompt (English):**

```
Consultation confirmation modal for snake expert mobile app.

MODAL (white card on dark overlay, 80% width, rounded 16px):

HEADER:
- Purple checkmark icon (64px)
- "Xác Nhận Bắt Đầu Tư Vấn" (22pt bold purple)

SESSION DETAILS CARD:
- "Tư vấn với: Nguyễn Văn A" (18pt semi-bold)
- "Thời gian: 14:00 - 14:45 (45 phút)" (16pt gray)
- "Phương thức: Video Call" (16pt gray)

PAYMENT CARD (light green background):
- "Phí tư vấn: 750,000 VNĐ" (18pt bold)
- "Trạng thái: Đã thanh toán" green badge
- "Bạn sẽ nhận: 675,000 VNĐ" (16pt green)
- "(sau phí nền tảng 10%)" (14pt gray)

CHECKLIST:
- ✓ "Camera và micro đã kiểm tra" (green check)
- ✓ "Môi trường yên tĩnh" (green check)
- ✓ "Đã xem thông tin ca bệnh" (green check)

BUTTONS:
- Large purple button "Bắt Đầu Ngay" (56px height, full width)
- Medium gray outlined "Hủy" (44px height, full width)

DESIGN: Clean confirmation modal, clear payment breakdown, readiness checklist, prominent start button.
```

---

### Screen 4: Waiting Room (Before Session Starts)

**Screen Purpose:**  
Holding screen while connecting to Patient/Rescuer, testing audio/video.

**Navigation:**
- Entry: After confirming from Screen 3
- Exit: Auto-navigate to Screen 5 when connected

**Key Components:**

1. **Status Header:**
   - "Đang Kết Nối..." (24pt, Purple)
   - Animated connection indicator (pulsing dots)

2. **Video Preview:**
   - Large preview of Expert's own camera (60% screen height)
   - Mirror mode
   - Overlay controls:
     - Camera toggle (switch front/back)
     - Mute/unmute mic
     - Video on/off toggle

3. **Connection Status Card:**
   - "Đang chờ bên kia vào phòng..."
   - Or: "Kiểm tra kết nối mạng..."
   - Signal strength indicator: "Mạng: Tốt" (green) / "Trung bình" (amber) / "Yếu" (red)

4. **Session Info:**
   - Participant: "Nguyễn Văn A" (with avatar)
   - Expected start: "14:00"
   - Current time: "13:59"

5. **Technical Check:**
   - Camera status: "✓ Camera hoạt động"
   - Microphone status: "✓ Micro hoạt động"
   - Network status: "✓ Kết nối ổn định"

6. **Cancel Button:**
   - Small text link: "Hủy cuộc gọi"

**Stitch Prompt (English):**

```
Waiting room screen for snake expert video consultation.

STATUS HEADER:
- "Đang Kết Nối..." (24pt purple) with animated dots

VIDEO PREVIEW (large, 60% screen, rounded corners):
- Expert's camera preview (mirror mode)
- Overlay controls at bottom:
  * Camera flip icon
  * Mic on/off icon (purple when active)
  * Video on/off icon

CONNECTION CARD (white, below video):
- "Đang chờ bên kia vào phòng..." (18pt semi-bold)
- Network indicator "Mạng: Tốt" green badge

SESSION INFO:
- Avatar + "Nguyễn Văn A" (18pt)
- "Giờ bắt đầu: 14:00" (16pt gray)
- "Hiện tại: 13:59" (16pt gray)

TECHNICAL CHECK (3 rows with green checks):
- ✓ "Camera hoạt động" (green text)
- ✓ "Micro hoạt động" (green text)
- ✓ "Kết nối ổn định" (green text)

CANCEL LINK:
- Small gray text "Hủy cuộc gọi" (centered)

DESIGN: Professional video call waiting room, clear connection status, technical readiness indicators, preview with controls.
```

---

## PART 2: LIVE CONSULTATION & POST-SESSION (4 Screens)

---

### Screen 5: Live Video/Chat Consultation

**Screen Purpose:**  
Active consultation session with Patient or Rescuer via video call or chat.

**Navigation:**
- Entry: Auto from Screen 4 when connected
- Exit: End session → Screen 7 (Post-Consultation Summary)

**Key Components:**

**VIDEO CALL MODE:**

1. **Participant Video** (Full screen):
   - Large video of Patient/Rescuer (75% screen)
   - Picture-in-picture: Expert's video (small corner, draggable, 120x160px)

2. **Top Overlay Bar:**
   - Session timer: "12:34" (elapsed time)
   - Connection quality: Green/amber/red dot
   - Participant name: "Nguyễn Văn A"

3. **Bottom Control Bar** (Semi-transparent overlay):
   - Mute/Unmute mic button (purple when active)
   - End call button (red, large, center)
   - Camera on/off button
   - Switch camera button
   - More options (⋯) → Switch to chat, screen share

4. **Quick Notes Panel** (Slide-in from right):
   - Button to open: "Ghi Chú" (sticky button on right edge)
   - Text area for live notes
   - Voice-to-text button
   - Auto-save indicator

5. **Image Sharing Section:**
   - Button: "Yêu Cầu Ảnh Bổ Sung"
   - Received images appear as thumbnails (tap to fullscreen)

**CHAT MODE:**

1. **Chat Interface:**
   - Messages list (scrollable)
   - Expert messages: Right-aligned, purple bubbles
   - Patient/Rescuer messages: Left-aligned, gray bubbles
   - Timestamps for each message

2. **Media Messages:**
   - Images inline (tap to expand)
   - Voice messages with play button
   - "Đang gõ..." indicator

3. **Input Section:**
   - Text input field
   - Attach photo button
   - Voice message button (hold to record)
   - Send button (purple)

4. **Quick Reply Templates:**
   - Swipe up for templates:
     - "Vui lòng chụp ảnh rõ hơn"
     - "Bạn có thấy triệu chứng nào khác?"
     - "Tôi khuyên bạn nên đến bệnh viện"

5. **Side Panel Access:**
   - Button: "Xem Hồ Sơ Bệnh Nhân"
   - Button: "Tra Cứu Rắn"

**Stitch Prompt Part 2-A (Video Mode):**

```
Live video consultation screen for snake expert.

VIDEO LAYOUT:
- Large participant video (75% screen, full width)
- Small expert PIP video (120x160px, top-right corner, draggable, purple border)

TOP OVERLAY (semi-transparent dark bar):
- Timer "12:34" (white, 20pt)
- Green connection dot
- Name "Nguyễn Văn A" (white, 16pt)

BOTTOM CONTROLS (semi-transparent bar):
- Row of 5 circular buttons (56px each):
  * Mic icon (purple when active, gray when muted)
  * Large red phone icon "Kết Thúc" (center, 64px)
  * Camera icon
  * Flip camera icon
  * More menu (⋯)

STICKY NOTES BUTTON (right edge):
- Vertical purple tab "Ghi Chú" (rotated text)
- When tapped, slides in panel from right (50% width):
  * "Ghi Chú Tư Vấn" header
  * Text area
  * Voice-to-text mic button
  * "Auto-save" indicator

FLOATING ACTION (bottom-left):
- Purple button "Yêu Cầu Ảnh" (outlined)

DESIGN: Professional video call interface, clear controls, non-intrusive notes panel, focus on conversation.
```

**Stitch Prompt Part 2-B (Chat Mode):**

```
Live chat consultation screen for snake expert.

HEADER:
- Back arrow, "Tư Vấn Trực Tuyến" (20pt)
- Avatar + "Nguyễn Văn A" (16pt)
- "Đang hoạt động" green dot

CHAT MESSAGES (scrollable):
- Patient messages (left, gray bubbles):
  * "Tôi bị rắn cắn, không biết loài gì"
  * Snake image (150px wide, rounded)
  * "Vết cắn sưng đỏ, hơi đau"
  * Timestamps "14:05"

- Expert messages (right, purple bubbles):
  * "Cho tôi xem ảnh rõ hơn được không?"
  * "Đây là Rắn Hổ Mang, rất nguy hiểm"
  * "Bạn cần đến bệnh viện ngay"

TYPING INDICATOR:
- "Đang gõ..." (gray, animated dots)

INPUT SECTION (bottom):
- Photo attach button (left)
- Text field "Nhắn tin..." (rounded)
- Voice record button (mic icon, hold to record)
- Send button (purple arrow, right)

QUICK REPLY BAR (swipe up):
- 3 chips:
  * "Vui lòng chụp ảnh rõ hơn"
  * "Bạn có thấy triệu chứng nào khác?"
  * "Tôi khuyên bạn nên đến bệnh viện"

SIDE BUTTONS (floating, right edge):
- "Xem Hồ Sơ" (outlined purple)
- "Tra Cứu Rắn" (outlined purple)

DESIGN: Professional medical chat, clear message distinction, inline media, quick reply templates, easy media sharing.
```

---

### Screen 6: In-Call Quick Reference

**Screen Purpose:**  
Side panel or overlay showing snake database and treatment guidelines during consultation.

**Navigation:**
- Entry: Tap "Tra Cứu" during Screen 5 consultation
- Exit: Close panel → Back to Screen 5

**Key Components:**

1. **Slide-in Panel** (Right side, 70% width):
   - Semi-transparent overlay on left (tap to close)
   - White panel slides from right

2. **Search Bar:**
   - "Tìm loài rắn hoặc triệu chứng..."
   - Quick filters: "Rắn Độc", "Miền Nam", "Thường Gặp"

3. **Tabs:**
   - "Loài Rắn" (Snake species)
   - "Triệu Chứng" (Symptoms)
   - "Điều Trị" (Treatment)

4. **Snake Species Cards** (Scrollable list):
   - Each card:
     - Snake thumbnail (80x80px)
     - Vietnamese name: "Rắn Hổ Mang Chúa"
     - Scientific name: "Ophiophagus hannah" (italic)
     - Danger badge: "CỰC ĐỘC" (red)
     - Quick facts: "Miền Nam, môi trường rừng"
     - Tap to expand → Full details

5. **Expanded Details:**
   - Full image gallery
   - Identification features
   - Venom type and effects
   - First aid guidelines
   - Antivenom recommendations
   - Distribution map

6. **Quick Actions:**
   - Button: "Gửi Cho Bệnh Nhân" (share info)
   - Button: "Thêm Vào Ghi Chú"

**Stitch Prompt (English):**

```
In-call quick reference panel for snake expert consultation.

SLIDE-IN PANEL (white, 70% width, from right):

SEARCH BAR:
- "Tìm loài rắn hoặc triệu chứng..." (rounded input)
- 3 filter chips: "Rắn Độc", "Miền Nam", "Thường Gặp"

TABS (3 tabs):
- "Loài Rắn" (active, purple underline)
- "Triệu Chứng" (gray)
- "Điều Trị" (gray)

SNAKE CARDS (scrollable list):

Card 1:
- Left: Snake thumbnail (80x80px)
- Right: 
  * "Rắn Hổ Mang Chúa" (18pt semi-bold)
  * "Ophiophagus hannah" (14pt italic gray)
  * "CỰC ĐỘC" red badge
  * "Miền Nam, môi trường rừng" (12pt gray)
- Tap expands to full details

Card 2 (EXPANDED):
- Full width
- Image gallery (3 photos, scrollable)
- "Đặc Điểm Nhận Dạng" section:
  * "Đầu to hình tam giác"
  * "Vảy vàng-đen xen kẽ"
  * "Dài 3-5m"
- "Nọc Độc" section:
  * "Neurotoxin mạnh"
  * "Tỷ lệ tử vong cao"
- "Sơ Cứu" section:
  * "Băng ép ngay"
  * "Gọi cấp cứu 115"
- "Huyết Thanh" section:
  * "Monospecific cobra antivenom"
  * "Liều: 10 vials IV"

QUICK ACTIONS (bottom of expanded card):
- Purple button "Gửi Cho Bệnh Nhân"
- Outlined "Thêm Vào Ghi Chú"

DESIGN: Efficient quick reference, searchable database, expandable details, share functionality, medical accuracy.
```

---

### Screen 7: Post-Consultation Summary & Notes

**Screen Purpose:**  
Expert completes consultation summary, adds diagnosis, and recommendations after session ends.

**Navigation:**
- Entry: End call from Screen 5
- Exit: Submit → Screen 8 (Completion Confirmation), then Dashboard

**Key Components:**

1. **Header:**
   - "Hoàn Tất Tư Vấn"
   - Session info: "Nguyễn Văn A - 14:00 (45 phút)"

2. **Session Duration Card:**
   - Actual duration: "Thời gian tư vấn: 47 phút"
   - Status: "Hoàn thành" (green badge)

3. **Snake Identification Section:**
   - "Kết Quả Nhận Diện"
   - Dropdown: Select confirmed species
   - Pre-filled from AI: "Rắn Hổ Mang Chúa (Ophiophagus hannah)"
   - Confidence: Slider or buttons: "Chắc chắn 100%" / "90%" / "70%" / "Không chắc"
   - Option: "Cập nhật vào cơ sở dữ liệu" (checkbox)

4. **Assessment Section:**
   - "Đánh Giá Tình Trạng"
   - Severity: Radio buttons
     - "Nhẹ" (green)
     - "Trung Bình" (amber)
     - "Nặng" (orange)
     - "Nguy Kịch" (red)

5. **Diagnosis & Recommendations:**
   - "Chẩn Đoán & Khuyến Nghị" (text area, large)
   - Pre-filled notes from live session
   - Voice-to-text button
   - Template button: Common recommendations

6. **Treatment Plan:**
   - "Phác Đồ Điều Trị"
   - Checkbox list:
     - "Đến bệnh viện ngay"
     - "Tiêm huyết thanh kháng nọc"
     - "Theo dõi tại nhà"
     - "Tái khám sau 3 ngày"
   - Custom text area for details

7. **Antivenom Information:**
   - "Huyết Thanh Khuyến Nghị"
   - Dropdown: Select antivenom type
   - Dosage field: "10 vials"
   - Administration: "IV (tĩnh mạch)"

8. **Follow-up:**
   - "Lịch Tái Khám" (optional)
   - Date picker
   - Notes field

9. **Attachments:**
   - "Tài Liệu Đính Kèm"
   - Show images from session (thumbnails)
   - Option to add more documents

10. **Action Buttons:**
    - Primary: "Hoàn Tất & Gửi" (large, purple)
    - Secondary: "Lưu Nháp"
    - Link: "Xem Lại"

**Stitch Prompt (English):**

```
Post-consultation summary screen for snake expert.

HEADER:
- "Hoàn Tất Tư Vấn" (24pt semi-bold purple)
- "Nguyễn Văn A - 14:00 (45 phút)" (16pt gray)

DURATION CARD:
- "Thời gian tư vấn: 47 phút" (18pt semi-bold)
- "Hoàn thành" green badge

IDENTIFICATION SECTION:
- "Kết Quả Nhận Diện" header
- Dropdown "Rắn Hổ Mang Chúa (Ophiophagus hannah)"
- Confidence buttons: "100%" (selected, purple), "90%", "70%", "Không chắc"
- Checkbox "Cập nhật vào cơ sở dữ liệu"

ASSESSMENT:
- "Đánh Giá Tình Trạng" header
- 4 radio buttons:
  * "Nhẹ" (green outline)
  * "Trung Bình" (amber outline)
  * "Nặng" (orange outline, SELECTED)
  * "Nguy Kịch" (red outline)

DIAGNOSIS:
- "Chẩn Đoán & Khuyến Nghị" header
- Large text area (8 lines visible)
- Pre-filled: "Bệnh nhân bị rắn hổ mang cắn, nọc độc neurotoxin. Triệu chứng sưng đỏ, tê nhẹ..."
- Mic button (voice-to-text) + "Mẫu" button (templates)

TREATMENT PLAN:
- "Phác Đồ Điều Trị" header
- Checkboxes:
  * ✓ "Đến bệnh viện ngay"
  * ✓ "Tiêm huyết thanh kháng nọc"
  * ☐ "Theo dõi tại nhà"
  * ☐ "Tái khám sau 3 ngày"
- Text area for details

ANTIVENOM:
- "Huyết Thanh Khuyến Nghị" header
- Dropdown "Monospecific cobra antivenom"
- Input "Liều lượng: 10 vials"
- Input "Đường dùng: IV (tĩnh mạch)"

FOLLOW-UP:
- "Lịch Tái Khám" header (optional)
- Date picker "15/12/2025"
- Notes field

ATTACHMENTS:
- "Tài Liệu Đính Kèm" header
- 3 image thumbnails (80px each)
- "+ Thêm tài liệu" button

BOTTOM BUTTONS:
- Large purple "Hoàn Tất & Gửi" (60px)
- Medium outlined "Lưu Nháp"
- Small link "Xem Lại"

DESIGN: Comprehensive medical summary form, structured data entry, template support, evidence-based recommendations.
```

---

### Screen 8: Consultation Completion & Payment

**Screen Purpose:**  
Confirmation screen showing consultation completed successfully and payment processed.

**Navigation:**
- Entry: Submit from Screen 7
- Exit: Auto-redirect to Dashboard after 3 seconds, or tap "Xong"

**Key Components:**

1. **Success Animation:**
   - Large green checkmark with animation (120x120px)
   - Confetti or sparkle effect

2. **Completion Message:**
   - "✓ Hoàn Tất Tư Vấn Thành Công!"
   - Subtext: "Báo cáo đã được gửi đến Nguyễn Văn A"

3. **Session Summary Card:**
   - Patient/Rescuer name and avatar
   - Date and time: "11/12/2025 - 14:00"
   - Duration: "47 phút"
   - Type: "Video Call"

4. **Payment Confirmation:**
   - "Thanh Toán Hoàn Tất" (green badge)
   - Fee: "750,000 VNĐ"
   - Platform fee (10%): "-75,000 VNĐ"
   - Your earnings: "+675,000 VNĐ" (large, green, bold)
   - Payment method: "Chuyển vào ví SnakeAid"
   - Processing time: "Trong vòng 24h"

5. **Rating Reminder:**
   - "Bệnh nhân sẽ được yêu cầu đánh giá bạn sau tư vấn"
   - Star rating placeholder: "⭐⭐⭐⭐⭐"

6. **Next Steps:**
   - "Báo cáo đã được lưu vào hồ sơ"
   - "Bạn có thể xem lại trong 'Lịch Sử Tư Vấn'"

7. **Action Buttons:**
   - Primary: "Xem Báo Cáo" (outlined purple)
   - Secondary: "Về Trang Chủ" (filled purple)

8. **Auto-redirect:**
   - Countdown: "Tự động chuyển về trang chủ sau 3s..."

**Stitch Prompt (English):**

```
Consultation completion screen for snake expert.

SUCCESS ANIMATION (center top):
- Large green checkmark icon (120px) with fade-in animation
- Sparkle effects around it

COMPLETION MESSAGE:
- "✓ Hoàn Tất Tư Vấn Thành Công!" (24pt bold green)
- "Báo cáo đã được gửi đến Nguyễn Văn A" (16pt gray)

SESSION CARD (white, rounded):
- Avatar (60px) + "Nguyễn Văn A" (20pt semi-bold)
- "11/12/2025 - 14:00" (16pt gray)
- "47 phút" + "Video Call" badge

PAYMENT CARD (light green background):
- "Thanh Toán Hoàn Tất" green badge
- Row: "Phí tư vấn" | "750,000 VNĐ"
- Row: "Phí nền tảng (10%)" | "-75,000 VNĐ" (red)
- Divider line
- Row: "Bạn nhận được" | "+675,000 VNĐ" (28pt bold green)
- Small text: "Chuyển vào ví SnakeAid trong vòng 24h"

RATING SECTION:
- "Bệnh nhân sẽ được yêu cầu đánh giá bạn" (14pt gray)
- 5 star icons (yellow outline, 32px each)

NEXT STEPS:
- ✓ "Báo cáo đã được lưu vào hồ sơ" (green check)
- ✓ "Bạn có thể xem lại trong 'Lịch Sử Tư Vấn'" (green check)

BUTTONS:
- Outlined purple "Xem Báo Cáo" (48px height)
- Filled purple "Về Trang Chủ" (56px height)

COUNTDOWN:
- "Tự động chuyển về trang chủ sau 3s..." (12pt gray, bottom)

DESIGN: Celebratory success screen, clear payment confirmation, transparent fee breakdown, smooth auto-redirect.
```

---

## Integration Points

### API Endpoints:
- `GET /expert/consultation/request/{id}` - Get consultation request details
- `POST /expert/consultation/accept` - Accept consultation
- `POST /expert/consultation/decline` - Decline consultation
- `GET /expert/consultation/session/{id}` - Get active session data
- `POST /expert/consultation/notes` - Save consultation notes during session
- `POST /expert/consultation/complete` - Submit post-consultation summary
- `GET /expert/snake-database/search?q={query}` - Search snake species
- `POST /expert/snake-database/verify` - Verify AI identification
- `GET /expert/consultation/history` - Get past consultations

### Real-time Features:
- WebRTC for video/audio calls
- WebSocket for chat messages
- Live typing indicators
- Connection quality monitoring
- Auto-save notes every 30 seconds

### Payment Flow:
- Scheduled: Patient pre-pays → Escrow → Expert completes → Auto-release
- Urgent: Platform pays Expert directly → Deduct from Rescuer earnings (if agreed) or Platform absorbs cost

---

## Version History
- **v1.0** - December 11, 2025: Initial consultation flow design (8 screens)

---

## Design Review Checklist
- [x] Dual mode support (Scheduled + Urgent)
- [x] Video and chat interfaces designed
- [x] Pre-consultation preparation for scheduled sessions
- [x] Quick reference database access during calls
- [x] Comprehensive post-consultation summary
- [x] Payment transparency and confirmation
- [x] Real-time connection quality indicators
- [x] Evidence-based medical documentation
- [x] All touch targets minimum 44x44px
- [x] Consistent purple color scheme

---

*This document is part of the SnakeAid Platform UI Design Documentation*  
*Related Documents: Expert-Dashboard-Screens.md, Expert-Revenue-Management-Screens.md*
