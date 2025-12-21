# RESCUE REQUEST FLOW - UI DESIGN SCREENS (RESCUER ROLE)

## Thông tin tài liệu
- **Tên dự án:** SnakeAid - AI-Powered Platform for Snakebite First Aid and Rescue Support
- **Module:** Rescuer Mobile Application
- **Role:** 🚑 **SNAKE RESCUER** (Đội cứu hộ rắn chuyên nghiệp)
- **Flow:** Rescue Request Flow (Luồng yêu cầu cứu hộ rắn)
- **Công cụ thiết kế:** Stitch with Google (prompt-based design)
- **Số lượng màn hình:** 10 screens
- **Ngày tạo:** December 8, 2025
- **Location:** `/02-UI-Design/Rescuer/Rescuer-Rescue-Request-Flow-Screens.md`

> **⚠️ LƯU Ý:** Document này chỉ cover màn hình cho **RESCUER role** trong **Rescue Request Flow** (Swimlane 2).
> Đây là flow khi Patient phát hiện rắn (chưa bị cắn) và yêu cầu đội cứu hộ đến bắt rắn.

---

## 🎯 Flow Context (From Swimlane Diagram)

### Rescuer's Journey trong Rescue Request Flow:

```
1. Rescuer đang ONLINE sẵn sàng nhận yêu cầu cứu hộ
   ↓
2. Backend tìm top 3 Rescuer phù hợp (gần nhất, rating cao)
   ↓
3. Rescuer nhận PUSH NOTIFICATION yêu cầu cứu hộ rắn
   - Thông báo: Ảnh rắn, vị trí, kết quả AI, phí đề xuất
   - Thời gian chấp nhận: 2 phút
   ↓
4. Rescuer xem chi tiết yêu cầu
   - Ảnh rắn (nhiều góc độ)
   - Kết quả AI nhận diện loài rắn (FE-21)
   - Mức độ nguy hiểm
   - Vị trí GPS chính xác
   - Thông tin Patient
   - Phí cứu hộ đề xuất
   ↓
5. Rescuer CHẤP NHẬN hoặc TỪ CHỐI yêu cầu (FE-06)
   ↓
6. Nếu CHẤP NHẬN:
   - Xem lại thông tin rắn và hướng dẫn an toàn (FE-09, FE-10, FE-21)
   - Chuẩn bị thiết bị cần thiết (FE-23)
   ↓
7. [Optional] Nếu không chắc về loài rắn → Liên hệ Expert (FE-12)
   ↓
8. Bắt đầu di chuyển
   - Cập nhật trạng thái: "Đang trên đường" (FE-07)
   - Bật GPS tracking real-time (FE-18)
   - Navigation đến vị trí (FE-19)
   ↓
9. Patient theo dõi vị trí Rescuer trên bản đồ (FE-24, FE-25, FE-26)
   ↓
10. Rescuer đến nơi
    - Cập nhật trạng thái: "Đã đến" (FE-20)
    - Gặp Patient, khảo sát vị trí rắn
    ↓
11. Thực hiện bắt rắn
    - Cập nhật trạng thái: "Đang xử lý" (FE-07)
    - Áp dụng quy trình an toàn
    ↓
12. Sau khi bắt xong
    - Chụp ảnh rắn đã bắt (FE-16)
    - Xác nhận lại loài rắn
    - Cập nhật trạng thái: "Hoàn thành" (FE-07)
    - Ghi nhận chi tiết vào database (FE-15)
    ↓
13. Thanh toán và đánh giá (Giai đoạn 2.4)
    - Patient thanh toán (FE-28)
    - Rescuer nhận thanh toán (FE-26)
    - Patient đánh giá Rescuer
    - Rescuer xem đánh giá (FE-27)
```

### Key Features for Rescuer in Rescue Request Flow:
- **FE-01:** Nhận thông báo yêu cầu cứu hộ rắn với ảnh và vị trí
- **FE-02:** Xem chi tiết yêu cầu: loài rắn dự đoán, mức độ nguy hiểm
- **FE-03:** Xác nhận loại rắn (có độc/không độc) từ hình ảnh
- **FE-04:** Cập nhật kết quả xác minh lên hệ thống
- **FE-06:** Chấp nhận hoặc từ chối yêu cầu cứu hộ
- **FE-07:** Cập nhật tiến độ (Đang đến / Đã đến / Đang xử lý / Hoàn thành)
- **FE-09, FE-10:** Quy trình chuẩn bắt rắn an toàn, thiết bị cần thiết
- **FE-12:** Liên lạc với chuyên gia để nhận diện chính xác
- **FE-15:** Ghi nhận chi tiết cứu hộ (vị trí, thời gian, loài rắn, kết quả)
- **FE-16:** Chụp ảnh rắn sau khi bắt
- **FE-18:** Cập nhật vị trí real-time
- **FE-19:** Điều hướng đến vị trí Patient
- **FE-20:** Gửi thông báo trạng thái cho Patient
- **FE-21:** Sử dụng AI nhận diện rắn, nhận cảnh báo mức độ nguy hiểm
- **FE-23:** Chuẩn bị thiết bị và biện pháp an toàn
- **FE-24, FE-25, FE-26:** Revenue management
- **FE-27:** Xem đánh giá từ khách hàng

---

## 🎨 Design System Overview

### Color Palette:
- **Primary Color:** Orange-Red (Emergency) `#FF6B35`
- **Secondary Color:** Deep Orange `#F7931E`
- **Background:** White `#FFFFFF`
- **Text Primary:** Dark Gray `#333333`
- **Text Secondary:** Medium Gray `#666666`
- **Accent - Success:** Green `#28A745`
- **Accent - Danger (Venomous):** Red `#DC3545`
- **Accent - Warning:** Amber `#FFC107`
- **Accent - Info:** Blue `#007BFF`
- **Status Active:** Orange `#FF6B35`
- **Status Completed:** Green `#28A745`

### Typography:
- **Logo:** Bold, Large (32-36pt)
- **Headings:** Semi-bold (20-24pt)
- **Body Text:** Regular (16-18pt)
- **Button Text:** Medium (16pt)
- **Caption:** Regular (14pt)
- **Alert Text:** Bold (18-20pt)

### Component Style:
- **Cards:** Rounded corners (12px), prominent shadows for requests
- **Buttons:** Rounded (8px), large touch targets (min 50px height)
- **Status Badges:** Rounded pills with color-coded backgrounds
- **Image Gallery:** Swipeable horizontal carousel
- **Timer:** Countdown with pulsing animation
- **Map View:** Full-screen with overlay controls

---

## 📱 SCREEN DESIGNS & PROMPTS

> **🚑 Tất cả screens dưới đây là cho RESCUER role** - đội cứu hộ nhận và xử lý yêu cầu bắt rắn

---

### Screen 1: Incoming Rescue Request Notification

#### Thông tin màn hình:
- **Tên:** Màn hình thông báo yêu cầu cứu hộ mới
- **Mục đích:** Alert Rescuer về yêu cầu cứu hộ rắn mới với thông tin quan trọng
- **Flow position:** Giai đoạn 2.2 - Rescuer nhận thông báo từ Backend
- **Priority:** ⭐⭐⭐ (Cao nhất - Critical notification)

#### Key Components:
1. **Full-Screen Overlay Alert:**
   - Semi-transparent dark background
   - Central alert card with shadow
   - Pulsing border animation (orange-red)

2. **Header Section:**
   - Icon: Warning bell (large, orange-red)
   - Title: "YÊU CẦU CỨU HỘ MỚI"
   - Badge: "KHẨN CẤP" (red) or "BÌNH THƯỜNG" (amber)

3. **Snake Preview:**
   - Small thumbnail of snake photo
   - AI result badge: "Rắn Hổ Mang" (venomous = red, non-venomous = green)
   - Confidence: "Độ chính xác: 92%"

4. **Location Info:**
   - Distance: "1.2 km từ bạn" (bold, large)
   - Address: "123 Nguyễn Văn Linh, Quận 1"
   - Time estimate: "~5 phút lái xe"

5. **Fee Info:**
   - Proposed fee: "200,000 VNĐ" (bold, green)
   - Platform fee note: "(Bạn nhận 85%)"

6. **Countdown Timer:**
   - Large circular countdown: "1:45" (minutes:seconds remaining)
   - Text: "Thời gian còn lại để chấp nhận"

7. **Action Buttons:**
   - Primary button: "XEM CHI TIẾT" (large, orange-red, full width)
   - Secondary button: "TỪ CHỐI" (outlined, gray)

#### Stitch Prompt (English):

```
Mobile app full-screen notification overlay for snake rescue request alert. Emergency notification interface with orange-red (#FF6B35) theme.

Full screen with semi-transparent dark overlay (black 60% opacity). Centered white card with shadow and pulsing orange-red border (2px, animation).

Card top section: Large orange-red bell icon (48px) centered. Below icon, bold dark gray title "YÊU CẦU CỨU HỘ MỚI" (24pt). Top-right corner, small red badge "KHẨN CẤP" rounded.

Snake preview section: Horizontal layout with small snake photo thumbnail (80px square, rounded 8px) left. Right side vertical: Bold snake name "Rắn Hổ Mang" dark gray (18pt), small red badge "Có độc" below name, small gray text "Độ chính xác AI: 92%" below badge.

Location section white background, rounded, padding: Bold large text "1.2 km" orange-red (20pt) with location pin icon. Below, gray text "123 Nguyễn Văn Linh, Quận 1". Below, small gray text with car icon "~5 phút lái xe".

Fee section: Bold large green text "200,000 VNĐ" (24pt) centered. Below, small gray text "(Bạn nhận 85% = 170,000 VNĐ)".

Countdown timer: Large circular progress ring orange-red. Center shows "1:45" bold dark gray (32pt). Below circle, small gray text "Thời gian còn lại để chấp nhận".

Bottom section: Two buttons vertically stacked. Top button large solid orange-red "XEM CHI TIẾT" full width (50px height). Below, large outlined gray button "TỪ CHỐI" full width.

Design: Urgent alert interface, clear information hierarchy, countdown pressure, strong CTAs.
```

#### Notes for Stitch:
- Countdown timer phải có animation và auto-dismiss khi hết giờ
- Pulsing border để tạo urgency
- Fee calculation transparent để build trust
- Distance và time estimate phải prominent

---

### Screen 2: Rescue Request Detail Screen

#### Thông tin màn hình:
- **Tên:** Màn hình chi tiết yêu cầu cứu hộ
- **Mục đích:** Hiển thị đầy đủ thông tin để Rescuer quyết định chấp nhận hay từ chối
- **Flow position:** Sau khi tap "Xem Chi Tiết" từ notification
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button (left)
   - Title: "Chi Tiết Yêu Cầu"
   - Timer countdown (top-right, red): "1:23"

2. **Snake Photos Gallery (top section):**
   - Horizontal swipeable carousel
   - Multiple snake photos (3-5 photos)
   - Dots indicator showing current photo
   - Zoom capability on tap

3. **AI Identification Result Card:**
   - **Snake Name:** "Rắn Hổ Mang" (bold, large)
   - **Toxicity Badge:** "Có Độc" (red) / "Không Độc" (green)
   - **Confidence:** "92% chính xác" (with progress bar)
   - **Danger Level:** "Nguy hiểm cao" (red badge)
   - **Scientific Name:** "Naja kaouthia" (gray, italic)

4. **Safety Guidelines Card:**
   - Icon: Shield with exclamation
   - Title: "Hướng Dẫn An Toàn"
   - Bullet points:
     - "Mang găng tay dày"
     - "Sử dụng móc bắt rắn chuyên dụng"
     - "Giữ khoảng cách an toàn 2m"
     - "Chuẩn bị túi vải dày"
   - Link: "Xem hướng dẫn đầy đủ"

5. **Required Equipment Checklist:**
   - Title: "Thiết Bị Cần Thiết"
   - Checkboxes (Rescuer can tick):
     - ☐ Móc bắt rắn (snake hook)
     - ☐ Găng tay bảo hộ
     - ☐ Túi vải/hộp đựng
     - ☐ Đèn pin
     - ☐ Phun nước (nếu cần)

6. **Location & Patient Info:**
   - **Address:** "123 Nguyễn Văn Linh, Quận 1, TP.HCM"
   - **Distance:** "1.2 km" (bold)
   - **ETA:** "5 phút lái xe"
   - **Mini map** showing location
   - Button: "Chỉ Đường" (opens navigation)
   - **Patient Name:** "Nguyễn Văn A"
   - **Phone:** "0912 345 678" (with call button)
   - **Additional Info:** "Rắn trong vườn, gần hồ nước"

7. **Fee Breakdown Card:**
   - **Total Fee:** "200,000 VNĐ" (bold, green, large)
   - Breakdown:
     - "Phí cứu hộ: 200,000 VNĐ"
     - "Bạn nhận: 170,000 VNĐ (85%)"
     - "Phí nền tảng: 20,000 VNĐ (10%)"
     - "Quỹ bảo hiểm: 10,000 VNĐ (5%)"

8. **Expert Consultation Option:**
   - Text: "Không chắc về loài rắn này?"
   - Button: "Hỏi Chuyên Gia" (outlined, blue)

9. **Action Buttons (sticky bottom):**
   - Primary button: "CHẤP NHẬN YÊU CẦU" (large, orange-red, full width)
   - Secondary button: "TỪ CHỐI" (outlined, gray, full width)

#### Stitch Prompt (English):

```
Mobile app rescue request detail screen for snake rescue mission. Comprehensive information interface with orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Chi Tiết Yêu Cầu" centered bold dark gray, countdown timer "1:23" right in red with clock icon.

Top section: Horizontal swipeable photo gallery showing 3 snake images. Each photo rectangular (full width, 250px height), rounded corners (12px). Below gallery, 3 gray dots indicating photo position, center dot orange-red (active).

AI result card white background, shadow, rounded: Bold large "Rắn Hổ Mang" dark gray (22pt) top. Next to title, red rounded badge "Có Độc". Below name, horizontal progress bar showing "92% chính xác" with orange fill. Below bar, red badge "Nguy hiểm cao". Bottom, small italic gray text "Naja kaouthia".

Safety guidelines card: Orange shield icon left. Bold title "Hướng Dẫn An Toàn" dark gray right. Four bullet points with orange checkmarks:
• Mang găng tay dày
• Sử dụng móc bắt rắn chuyên dụng
• Giữ khoảng cách an toàn 2m
• Chuẩn bị túi vải dày
Small blue text link "Xem hướng dẫn đầy đủ" bottom-right.

Equipment checklist card: Bold title "Thiết Bị Cần Thiết". Five rows with checkboxes left, equipment name gray text right:
□ Móc bắt rắn (snake hook)
□ Găng tay bảo hộ
□ Túi vải/hộp đựng
□ Đèn pin
□ Phun nước (nếu cần)

Location card: Small map thumbnail (full width, 120px height) top. Below map, bold "123 Nguyễn Văn Linh, Quận 1" dark gray. Row showing "1.2 km" bold orange left, "5 phút lái xe" gray right with car icon. Outlined orange button "Chỉ Đường" full width. Divider line. Patient info: "Nguyễn Văn A" bold with phone "0912 345 678" and green call icon button right. Small gray text "Rắn trong vườn, gần hồ nước".

Fee breakdown card: Large bold green "200,000 VNĐ" (28pt) centered. Below, four lines gray text with breakdown:
- Phí cứu hộ: 200,000 VNĐ
- Bạn nhận: 170,000 VNĐ (85%)
- Phí nền tảng: 20,000 VNĐ (10%)
- Quỹ bảo hiểm: 10,000 VNĐ (5%)

Expert consultation box light blue background: Gray text "Không chắc về loài rắn này?" left. Outlined blue button "Hỏi Chuyên Gia" right.

Bottom sticky section white background, top shadow: Large solid orange-red button "CHẤP NHẬN YÊU CẦU" full width (55px height). Below, large outlined gray button "TỪ CHỐI" full width.

Design: Comprehensive mission briefing, safety-first approach, clear fee transparency, easy decision making.
```

#### Notes for Stitch:
- Photos phải swipeable và zoomable
- Equipment checklist phải interactive
- Call button phải direct call
- Map thumbnail tap để open full map
- Timer phải countdown real-time

---

### Screen 3: Expert Consultation Request Screen

#### Thông tin màn hình:
- **Tên:** Màn hình yêu cầu tư vấn chuyên gia
- **Mục đích:** Rescuer gửi ảnh rắn để Expert xác nhận loài nếu không chắc chắn
- **Flow position:** Optional - Khi Rescuer tap "Hỏi Chuyên Gia" (FE-12)
- **Priority:** ⭐⭐ (Related to Flow 3.2)

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Tư Vấn Chuyên Gia Khẩn Cấp"
   - Status: "Đang tìm chuyên gia online..."

2. **Snake Photos Review:**
   - Shows same photos from request
   - AI result displayed
   - Text: "Bạn cần xác nhận về:"

3. **Quick Question Form:**
   - Text area: "Mô tả chi tiết (tùy chọn)"
   - Placeholder: "VD: Rắn có vằn vàng đen, đầu to..."
   - Character count: "0/200"

4. **Urgency Indicator:**
   - Badge: "ƯU TIÊN CAO" (red)
   - Text: "Chuyên gia sẽ phản hồi trong 2-3 phút"

5. **Action Button:**
   - Primary: "GỬI YÊU CẦU TƯ VẤN" (orange-red, full width)
   - Text below: "Miễn phí cho đội cứu hộ"

#### Stitch Prompt (English):

```
Mobile app expert consultation request screen for rescuer snake identification help. Urgent consultation interface with orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Tư Vấn Chuyên Gia Khẩn Cấp" bold dark gray centered. Below header, animated text "Đang tìm chuyên gia online..." with loading dots.

Photos review section: Horizontal scrollable showing 3 snake thumbnails (100px square each). Below photos, AI result badge "Rắn Hổ Mang - 92%" with question mark icon. Text "Bạn cần xác nhận về:" gray above photos.

Form section white card: Label "Mô tả chi tiết (tùy chọn)" bold dark gray. Large text area (4 lines height) with gray border, rounded corners, placeholder "VD: Rắn có vằn vàng đen, đầu to...". Bottom-right of textarea, small gray text "0/200".

Urgency card light red background (#FFEBEE): Red badge "ƯU TIÊN CAO" with alert icon left. Text "Chuyên gia sẽ phản hồi trong 2-3 phút" dark gray right.

Bottom section: Large solid orange-red button "GỬI YÊU CẦU TƯ VẤN" full width (55px height). Below button, centered small green text "Miễn phí cho đội cứu hộ" with checkmark icon.

Design: Quick consultation request, urgency emphasized, free service highlighted.
```

#### Notes for Stitch:
- Loading animation cho "finding expert"
- Text area auto-focus
- Free service badge để encourage usage

---

### Screen 4: Waiting for Expert Response Screen

#### Thông tin màn hình:
- **Tên:** Màn hình chờ phản hồi từ chuyên gia
- **Mục đích:** Show progress trong khi đợi Expert accept và respond
- **Flow position:** Sau khi gửi yêu cầu tư vấn
- **Priority:** ⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Đang Chờ Chuyên Gia"

2. **Progress Animation:**
   - Large animated icon (searching/connecting)
   - Text: "Đang kết nối với chuyên gia..."
   - Timer: "Đã chờ: 0:35"

3. **Status Updates:**
   - "✓ Yêu cầu đã gửi"
   - "↻ Đang tìm chuyên gia phù hợp..."
   - "⏳ Dự kiến: 2-3 phút"

4. **Meanwhile Section:**
   - Title: "Trong lúc chờ, bạn có thể:"
   - Quick actions:
     - "Xem lại ảnh rắn"
     - "Đọc hướng dẫn an toàn"
     - "Kiểm tra thiết bị"

5. **Cancel Option:**
   - Text link: "Hủy yêu cầu và tiếp tục"

#### Stitch Prompt (English):

```
Mobile app waiting screen for expert consultation response. Loading interface with orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Đang Chờ Chuyên Gia" centered bold dark gray.

Center section: Large animated circular loading icon orange-red (80px) with rotating effect. Below icon, bold text "Đang kết nối với chuyên gia..." dark gray (18pt). Below text, timer "Đã chờ: 0:35" gray with clock icon.

Status timeline vertical layout: Three rows with icons and text:
- Row 1: Green checkmark icon, "Yêu cầu đã gửi" gray strikethrough
- Row 2: Orange rotating arrow icon, "Đang tìm chuyên gia phù hợp..." bold dark gray
- Row 3: Gray clock icon, "Dự kiến: 2-3 phút" gray

Meanwhile section white card: Bold title "Trong lúc chờ, bạn có thể:" dark gray. Three action items with chevron right icons:
• Xem lại ảnh rắn
• Đọc hướng dẫn an toàn
• Kiểm tra thiết bị

Bottom: Centered blue text link "Hủy yêu cầu và tiếp tục".

Design: Progress feedback, reduce waiting anxiety, provide useful actions during wait time.
```

#### Notes for Stitch:
- Timer phải real-time counting
- Animation cho connecting status
- Quick actions phải functional

---

### Screen 5: Expert Consultation Chat Screen

#### Thông tin màn hình:
- **Tên:** Màn hình chat với chuyên gia
- **Mục đích:** Chat/Video call real-time với Expert để xác nhận loài rắn
- **Flow position:** Khi Expert accept và bắt đầu tư vấn (FE-11, FE-14)
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button (left)
   - Expert info: Avatar + Name "TS. Nguyễn Văn An"
   - Status: "Online" (green dot)
   - Video call button (top-right)

2. **Chat Messages:**
   - Expert photo/message bubbles (left, purple background)
   - Rescuer messages (right, orange background)
   - Timestamps
   - Snake photos shared (inline preview)

3. **Quick Photo Share:**
   - Camera button to take new photos
   - Gallery to share existing photos

4. **Expert Conclusion Card (after consultation):**
   - "KẾT LUẬN TƯ VẤN"
   - Confirmed snake: "Rắn Hổ Mang Chúa"
   - Toxicity: "Cực kỳ nguy hiểm"
   - Safety notes from expert
   - Button: "Xác Nhận & Tiếp Tục"

5. **Input Section:**
   - Text input field
   - Send button
   - Voice message button

#### Stitch Prompt (English):

```
Mobile app chat screen for expert consultation with snake specialist. Real-time messaging interface with orange-red (#FF6B35) theme.

Top header white background shadow: Back arrow left. Center: small circular avatar (40px) with name "TS. Nguyễn Văn An" bold dark gray right of avatar, small green dot and "Online" text below name. Top-right: video camera icon button.

Chat area white background: Scrollable message list. Expert messages: Left-aligned bubble purple background (#6F42C1), white text, rounded corners (12px sharp corner bottom-left). Small timestamp "14:32" gray below bubble. Rescuer messages: Right-aligned bubble orange background (#FF6B35), white text, rounded corners (12px sharp corner bottom-right). Timestamp below.

Shared photo message: Full-width image preview (200px height), rounded corners, with caption text below.

Expert conclusion card (when received): White card with shadow, purple left border (4px). Bold title "KẾT LUẬN TƯ VẤN" purple (18pt). Row: "Loài rắn:" gray left, "Rắn Hổ Mang Chúa" bold dark gray right. Row: "Độc tính:" gray left, red badge "Cực kỳ nguy hiểm" right. Gray text paragraph "Lưu ý an toàn từ chuyên gia...". Bottom: outlined orange button "Xác Nhận & Tiếp Tục" full width.

Bottom input section white background, top border: Camera icon button left. Text input field center (gray border, rounded, placeholder "Nhắn tin..."). Microphone icon button. Orange send arrow icon button right.

Design: Professional consultation chat, clear expert identity, visual conclusion summary.
```

#### Notes for Stitch:
- Real-time chat với WebSocket
- Video call button mở camera
- Expert conclusion phải clear và actionable

---

### Screen 6: En Route to Location Screen

#### Thông tin màn hình:
- **Tên:** Màn hình di chuyển đến địa điểm
- **Mục đích:** Navigation với GPS tracking real-time và status updates
- **Flow position:** Giai đoạn 2.3 - Sau khi chấp nhận yêu cầu, đang di chuyển (FE-07, FE-18, FE-19)
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Full-Screen Map:**
   - Rescuer's current location (orange marker with avatar)
   - Destination (Patient location, red pin)
   - Route highlighted (orange line)
   - Traffic overlay (if available)

2. **Top Status Bar (overlay on map):**
   - Time remaining: "5 phút nữa"
   - Distance: "1.2 km"
   - Status badge: "ĐANG TRÊN ĐƯỜNG" (orange)

3. **Bottom Info Card (slide-up drawer):**
   - Patient info mini card
   - Phone: "0912 345 678" with call button
   - Address: "123 Nguyễn Văn Linh, Quận 1"
   - Snake preview: Small thumbnail + "Rắn Hổ Mang"
   - Safety reminder: "Nhớ mang găng tay bảo hộ"

4. **Navigation Controls:**
   - "Bắt đầu chỉ đường" button (opens Google Maps/Waze)
   - "Gọi cho khách hàng" button
   - "Dừng nhiệm vụ" button (red, outlined)

5. **Status Update Button:**
   - Floating button: "Đã đến nơi"
   - Only visible when near destination (<100m)

6. **Auto-notifications to Patient:**
   - System auto-sends updates every 30 seconds
   - "Đội cứu hộ cách bạn 5 phút"
   - "Đội cứu hộ đang đến gần"

#### Stitch Prompt (English):

```
Mobile app navigation screen for rescuer en route to snake rescue location. Full-screen map navigation interface with orange-red (#FF6B35) theme.

Full-screen map view showing: Orange location marker with small avatar for rescuer position. Red destination pin marker. Orange route line connecting them. Blue current location dot with accuracy circle.

Top overlay white card shadow, rounded bottom corners: Row layout: "5 phút nữa" bold dark gray (18pt) left with clock icon, vertical divider, "1.2 km" bold right with road icon. Below row, orange badge "ĐANG TRÊN ĐƯỜNG" full width centered.

Bottom slide-up drawer white background, rounded top corners (16px), shadow: Handle bar gray centered top. Patient mini card: Row with small circular avatar, name "Nguyễn Văn A" bold, green phone icon button right. Below, gray text "123 Nguyễn Văn Linh, Quận 1" with location pin. Below, horizontal layout: small snake thumbnail (60px), "Rắn Hổ Mang" bold dark gray, small red "Có độc" badge. Yellow info box: "Nhớ mang găng tay bảo hộ" with alert icon.

Three buttons stacked: Solid orange "Bắt đầu chỉ đường" with navigation icon. Outlined orange "Gọi cho khách hàng" with phone icon. Outlined red "Dừng nhiệm vụ".

Floating action button bottom-right on map: Large circular green button "Đã đến nơi" (only shows when near destination).

Design: Map-first navigation, clear ETA, easy communication, safety reminders, prominent arrival confirmation.
```

#### Notes for Stitch:
- Map phải real-time tracking với GPS
- "Đã đến nơi" button chỉ show khi distance < 100m
- Auto-send notification to Patient mỗi 30 giây

---

### Screen 7: Arrived at Location Screen

#### Thông tin màn hình:
- **Tên:** Màn hình đã đến nơi
- **Mục đích:** Confirm arrival và chuẩn bị bắt đầu nhiệm vụ
- **Flow position:** Giai đoạn 2.3 - Khi Rescuer tap "Đã đến nơi" (FE-20)
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Đã Đến Địa Điểm"
   - Timer: "Thời gian di chuyển: 6 phút"

2. **Location Confirmation:**
   - Small map showing exact location
   - Text: "Bạn đã đến 123 Nguyễn Văn Linh"
   - Accuracy: "GPS chính xác ±5m"

3. **Patient Contact Card:**
   - Avatar + Name "Nguyễn Văn A"
   - Phone with direct call button
   - Text: "Khách hàng sẽ ra đón bạn"

4. **Pre-Rescue Checklist:**
   - Title: "Kiểm Tra Trước Khi Bắt"
   - Interactive checkboxes:
     - ☐ Đã gặp khách hàng
     - ☐ Đã xác định vị trí rắn
     - ☐ Thiết bị đầy đủ
     - ☐ Đánh giá môi trường an toàn

5. **Snake Info Reminder:**
   - Photo + "Rắn Hổ Mang - Có độc"
   - "Cần găng tay dày và móc 2m"

6. **Status Update Button:**
   - Primary: "BẮT ĐẦU BẮT RẮN" (orange-red, large)
   - This updates status to "Đang xử lý"

#### Stitch Prompt (English):

```
Mobile app arrival confirmation screen for snake rescue mission. Pre-work checklist interface with orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Đã Đến Địa Điểm" bold dark gray centered. Below header, small gray text "Thời gian di chuyển: 6 phút" with timer icon.

Location card: Small map thumbnail (full width, 120px height) rounded corners. Below map, bold text "Bạn đã đến 123 Nguyễn Văn Linh" dark gray with green checkmark. Small gray text "GPS chính xác ±5m" with location icon.

Patient contact card white shadow: Row with circular avatar (50px), name "Nguyễn Văn A" bold dark gray center, large green phone icon button right. Below, gray text "Khách hàng sẽ ra đón bạn".

Checklist card: Bold title "Kiểm Tra Trước Khi Bắt" dark gray (18pt). Four checkbox rows with gray text:
□ Đã gặp khách hàng
□ Đã xác định vị trí rắn
□ Thiết bị đầy đủ
□ Đánh giá môi trường an toàn

Snake reminder card light yellow background: Row with small snake thumbnail (60px) left, bold "Rắn Hổ Mang" dark gray center, red badge "Có độc" right. Below, orange text with alert icon "Cần găng tay dày và móc 2m".

Bottom section: Large solid orange-red button "BẮT ĐẦU BẮT RẮN" full width (55px height).

Design: Arrival confirmation, safety checklist emphasis, patient communication ready, clear next action.
```

#### Notes for Stitch:
- Checklist phải interactive và recommend ticking all
- Call button direct dial
- Status auto-notify Patient when arrival confirmed

---

### Screen 8: Rescue in Progress Screen

#### Thông tin màn hình:
- **Tên:** Màn hình đang thực hiện bắt rắn
- **Mục đích:** Track progress và allow documentation during rescue
- **Flow position:** Giai đoạn 2.3 - Trong quá trình bắt rắn (FE-07)
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Đang Bắt Rắn"
   - Status badge: "ĐANG XỬ LÝ" (orange, pulsing)

2. **Timer:**
   - Large running timer: "12:34" (elapsed time)
   - Text: "Thời gian xử lý"

3. **Snake Info Banner:**
   - Photo + "Rắn Hổ Mang - Có độc"
   - Badge: "Cực kỳ nguy hiểm"

4. **Safety Reminders (expandable):**
   - "⚠️ An Toàn Trong Quá Trình Bắt"
   - Tap to expand safety guidelines
   - Quick reference for emergency procedures

5. **Photo Documentation:**
   - Section: "Chụp Ảnh Quá Trình"
   - Camera button: "Chụp ảnh rắn sau khi bắt"
   - Gallery showing photos taken (0-5 photos)
   - Required before completing

6. **Quick Notes:**
   - Text area: "Ghi chú về quá trình bắt"
   - Placeholder: "VD: Rắn trong bụi rậm, khó tiếp cận..."
   - Voice-to-text button

7. **Emergency Actions:**
   - Red button: "Gọi Hỗ Trợ Khẩn Cấp"
   - Link: "Liên hệ Expert"

8. **Status Update:**
   - Button: "HOÀN THÀNH BẮT RẮN" (disabled until photo taken)

#### Stitch Prompt (English):

```
Mobile app rescue in progress screen for active snake capture operation. Real-time work tracking interface with orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Đang Bắt Rắn" bold dark gray centered. Orange badge "ĐANG XỬ LÝ" with pulsing animation right.

Timer section centered: Very large bold "12:34" orange-red (36pt) with stopwatch icon. Below, small gray text "Thời gian xử lý".

Snake info banner yellow background, padding: Row with snake thumbnail (60px) left, bold "Rắn Hổ Mang" dark gray center, red badge "Có độc" and "Cực kỳ nguy hiểm" stacked right.

Safety reminders collapsed card: Orange alert icon left, bold "An Toàn Trong Quá Trình Bắt" dark gray, down chevron icon right. When expanded, shows bullet list of safety guidelines.

Photo documentation section: Bold title "Chụp Ảnh Quá Trình" dark gray (18pt). Large dashed border box with camera icon center, text "Chụp ảnh rắn sau khi bắt" gray. Below, horizontal scrollable gallery showing captured photos (thumbnail 100px square each) with X delete button on each. Red asterisk "*Bắt buộc" small text.

Notes section: Bold title "Ghi Chú" dark gray. Large text area (3 lines) gray border rounded, placeholder "VD: Rắn trong bụi rậm, khó tiếp cận...". Bottom-right: blue microphone icon button for voice input.

Emergency section: Outlined red button "Gọi Hỗ Trợ Khẩn Cấp" with phone icon. Below, blue text link "Liên hệ Expert".

Bottom: Large solid orange-red button "HOÀN THÀNH BẮT RẮN" full width (55px height). If no photos, button is disabled gray with lock icon.

Design: Focus on safety, documentation requirements, emergency access, clear completion criteria.
```

#### Notes for Stitch:
- Timer phải running real-time
- Photo REQUIRED trước khi complete
- Voice-to-text cho notes
- Emergency buttons prominent

---

### Screen 9: Rescue Completion Screen

#### Thông tin màn hình:
- **Tên:** Màn hình hoàn thành cứu hộ
- **Mục đích:** Confirm success, xác nhận loài rắn cuối cùng, và ghi nhận chi tiết (FE-15, FE-16)
- **Flow position:** Giai đoạn 2.3 - Sau khi bắt rắn xong
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Xác Nhận Hoàn Thành"

2. **Success Animation:**
   - Large green checkmark icon
   - Text: "Đã bắt rắn thành công!"

3. **Rescue Summary:**
   - Time taken: "Thời gian: 15 phút"
   - Location: "123 Nguyễn Văn Linh"
   - Date/time: "8/12/2025, 14:30"

4. **Snake Confirmation:**
   - Title: "Xác Nhận Loài Rắn"
   - Photos taken during rescue (gallery)
   - Dropdown: "Chọn loài rắn"
   - Options: List of common snakes + "Khác"
   - If different from AI: Show alert "Khác với kết quả AI"
   - Scientific name field (optional)

5. **Final Details:**
   - Snake size: Slider or input "~120 cm"
   - Snake condition: "Khỏe mạnh" / "Bị thương" / "Đã chết"
   - Release location: Text input
   - Notes: Text area (from Screen 8, editable)

6. **Fee Reminder:**
   - "Bạn sẽ nhận: 170,000 VNĐ"
   - "Khách hàng thanh toán sau"

7. **Action Button:**
   - Primary: "XÁC NHẬN HOÀN THÀNH" (green, large)

#### Stitch Prompt (English):

```
Mobile app rescue completion confirmation screen. Success documentation interface with green (#28A745) and orange-red (#FF6B35) theme.

Top navigation: Back arrow left, title "Xác Nhận Hoàn Thành" bold dark gray centered.

Success section centered: Large animated green checkmark icon (80px) with scale-in effect. Below, bold green text "Đã bắt rắn thành công!" (22pt). Confetti animation background.

Summary card white rounded shadow: Three rows with icons:
- Clock icon, "Thời gian: 15 phút" gray
- Location pin icon, "123 Nguyễn Văn Linh" gray
- Calendar icon, "8/12/2025, 14:30" gray

Snake confirmation card: Bold title "Xác Nhận Loài Rắn" dark gray (18pt) with red asterisk. Horizontal scrollable photo gallery showing 3 captured snake images (120px square, rounded). Below gallery, dropdown selector with down arrow "Chọn loài rắn" - placeholder, border orange. If selected different from AI, show yellow alert box "⚠️ Khác với kết quả AI (Rắn Hổ Mang)". Optional text input "Tên khoa học (tùy chọn)" below.

Details section: Label "Kích thước ước tính" with slider 0-200cm, current value "~120 cm" displayed. Label "Tình trạng" with three radio buttons horizontal: "Khỏe mạnh" (selected green), "Bị thương", "Đã chết". Label "Địa điểm thả" with text input placeholder "VD: Rừng xa dân cư". Label "Ghi chú bổ sung" with text area (2 lines) showing previous notes.

Fee reminder card green background light (#E8F5E9): Bold green text "Bạn sẽ nhận: 170,000 VNĐ" (20pt) centered. Below, small gray text "Khách hàng thanh toán sau".

Bottom: Large solid green button "XÁC NHẬN HOÀN THÀNH" full width (55px height) with checkmark icon.

Design: Celebration of success, thorough documentation, species verification, clear financial expectation.
```

#### Notes for Stitch:
- Success animation để celebrate
- Snake confirmation critical - nếu khác AI, phải có alert
- All fields validate trước khi submit
- Fee reminder để set expectation

---

### Screen 10: Waiting for Payment Screen

#### Thông tin màn hình:
- **Tên:** Màn hình chờ thanh toán từ khách hàng
- **Mục đích:** Show status của payment process và rating (FE-26, FE-27)
- **Flow position:** Giai đoạn 2.4 - Sau khi confirm completion
- **Priority:** ⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Chờ Thanh Toán"

2. **Completion Status:**
   - Green checkmark: "Nhiệm vụ hoàn thành"
   - Info sent to: "Nguyễn Văn A"

3. **Payment Status:**
   - Status badge: "Đang chờ khách hàng thanh toán"
   - Timer: "Chờ từ: 2 phút trước"

4. **Expected Payment:**
   - Amount: "170,000 VNĐ" (bold, green, large)
   - Breakdown reminder:
     - "Phí cứu hộ: 200,000 VNĐ"
     - "Phí nền tảng: -20,000 VNĐ"
     - "Quỹ bảo hiểm: -10,000 VNĐ"

5. **Mission Summary:**
   - Snake: "Rắn Hổ Mang"
   - Time: "15 phút"
   - Location: "123 Nguyễn Văn Linh"
   - Photos count: "3 ảnh"

6. **Auto-notification:**
   - Text: "Chúng tôi đã gửi yêu cầu thanh toán đến khách hàng"
   - "Bạn sẽ nhận thông báo khi thanh toán thành công"

7. **Quick Actions:**
   - "Xem chi tiết nhiệm vụ"
   - "Nhắn tin với khách hàng"

8. **Navigation:**
   - Button: "Về Trang Chủ" (outlined)

#### Stitch Prompt (English):

```
Mobile app waiting for payment screen after rescue completion. Payment pending interface with green (#28A745) theme.

Top navigation: Back arrow left, title "Chờ Thanh Toán" bold dark gray centered.

Completion status card: Large green checkmark icon (60px) left, bold text "Nhiệm vụ hoàn thành" green (18pt) right. Below, small gray text "Đã gửi thông tin đến Nguyễn Văn A" with send icon.

Payment status card: Orange badge "Đang chờ khách hàng thanh toán" with clock icon. Below badge, gray text "Chờ từ: 2 phút trước" with timer icon.

Expected payment card white shadow prominent: Very large bold green "170,000 VNĐ" centered (32pt). Below, small gray text "Bạn sẽ nhận:" above amount. Thin divider line. Breakdown in small gray text:
- Phí cứu hộ: 200,000 VNĐ
- Phí nền tảng: -20,000 VNĐ
- Quỹ bảo hiểm: -10,000 VNĐ

Mission summary card: Four rows with icons and gray text:
- Snake icon, "Rắn Hổ Mang - Có độc"
- Clock icon, "Thời gian: 15 phút"
- Location pin icon, "123 Nguyễn Văn Linh"
- Camera icon, "3 ảnh đã ghi nhận"

Info card light blue background: Blue info icon left, text "Chúng tôi đã gửi yêu cầu thanh toán đến khách hàng. Bạn sẽ nhận thông báo khi thanh toán thành công" gray right.

Quick actions: Two text links blue:
- "Xem chi tiết nhiệm vụ"
- "Nhắn tin với khách hàng"

Bottom: Large outlined orange button "Về Trang Chủ" full width.

Design: Clear payment expectation, mission recap, patient reassurance during wait time.
```

#### Notes for Stitch:
- Auto-refresh khi có payment update
- Push notification khi payment complete
- Có thể message Patient nếu cần

---

## 🔗 NAVIGATION FLOW

```
Notification Alert (Screen 1)
    │
    ├─→ Rescue Request Detail (Screen 2)
    │   │
    │   ├─→ [Optional] Expert Consultation Request (Screen 3)
    │   │   └─→ Waiting for Expert (Screen 4)
    │   │       └─→ Expert Chat (Screen 5)
    │   │           └─→ Back to Detail (Screen 2)
    │   │
    │   ├─→ ACCEPT → En Route (Screen 6)
    │   │   └─→ Arrived (Screen 7)
    │   │       └─→ Rescue in Progress (Screen 8)
    │   │           └─→ Completion (Screen 9)
    │   │               └─→ Waiting for Payment (Screen 10)
    │   │                   └─→ [Payment Success] → Dashboard
    │   │
    │   └─→ DECLINE → Dashboard
```

---

## 📋 FEATURE MAPPING

| Screen | Related Major Features | Priority |
|--------|------------------------|----------|
| Notification Alert | FE-01, FE-02 | ⭐⭐⭐ |
| Request Detail | FE-02, FE-06, FE-09, FE-10, FE-21, FE-23 | ⭐⭐⭐ |
| Expert Consultation Request | FE-12 | ⭐⭐ |
| Waiting for Expert | FE-12 | ⭐⭐ |
| Expert Chat | FE-12, FE-14 | ⭐⭐⭐ |
| En Route | FE-07, FE-18, FE-19, FE-20 | ⭐⭐⭐ |
| Arrived | FE-20 | ⭐⭐⭐ |
| Rescue in Progress | FE-07, FE-16 | ⭐⭐⭐ |
| Completion | FE-15, FE-16 | ⭐⭐⭐ |
| Waiting for Payment | FE-24, FE-25, FE-26, FE-27 | ⭐⭐ |

---

## ✅ DESIGN CHECKLIST

Before implementation:

- [ ] All screens follow design system (colors, typography, spacing)
- [ ] Countdown timers functional và visible
- [ ] Photo galleries swipeable và zoomable
- [ ] GPS tracking real-time updates
- [ ] Map integration với Google Maps/Apple Maps
- [ ] Call buttons direct dial functionality
- [ ] Expert consultation chat real-time
- [ ] Status updates auto-notify Patient
- [ ] Payment breakdown transparent và clear
- [ ] Safety guidelines easily accessible
- [ ] Equipment checklist interactive
- [ ] Loading states for all async operations
- [ ] Error states for failed operations
- [ ] Offline mode considerations
- [ ] Push notifications cho critical updates

---

## 🔗 RELATED DOCUMENTATION

- **Main Flow:** `/01-Requirements/Main-Flow/Main-Flow.md` (Section 2)
- **Swimlane Diagram:** `/01-Requirements/Swimlane-Diagram/02-Swimlane-Rescue-Request-Flow.md`
- **Major Features:** `/01-Requirements/Major-Features/Major-Features-Summary.md` (Rescuer section)
- **Emergency Flow:** `/02-UI-Design/Rescuer/Rescuer-Emergency-Response-Flow-Screens.md`

---

**Last Updated:** December 8, 2025  
**Status:** ✅ Complete  
**Total Screens:** 10 screens

---

## 📊 IMPLEMENTATION NOTES

### Technical Requirements:
1. **Real-time GPS Tracking:** WebSocket connection cho live location updates
2. **Push Notifications:** Firebase Cloud Messaging cho incoming requests
3. **Maps Integration:** Google Maps SDK / Apple Maps
4. **Photo Upload:** Camera access + image compression
5. **Chat System:** WebSocket cho real-time messaging với Expert
6. **Timer/Countdown:** Precise countdown với server sync
7. **Payment Integration:** Connect với payment gateway APIs

### Business Logic:
1. **Matching Algorithm:** Backend tìm top 3 Rescuers dựa trên distance, rating, availability
2. **Timeout Handling:** 2 phút để accept, nếu không auto-decline và gửi cho Rescuer khác
3. **Fee Calculation:** 85% Rescuer, 10% Platform, 5% Insurance fund
4. **Expert Consultation:** Free cho Rescuers, không tính vào fee
5. **Rating Impact:** Rating từ Patient ảnh hưởng priority trong matching

### Security:
1. **Location Privacy:** Chỉ chia sẻ location khi mission active
2. **Payment Security:** Escrow system - hold payment until completion
3. **Photo Privacy:** Auto-delete sau 30 ngày nếu không dispute
4. **Chat History:** Encrypted và lưu để dispute resolution

### Analytics Events:
- `rescue_request_received`
- `rescue_request_accepted`
- `rescue_request_declined`
- `expert_consultation_requested`
- `en_route_started`
- `arrived_at_location`
- `rescue_started`
- `rescue_completed`
- `payment_received`
- `rating_received`
