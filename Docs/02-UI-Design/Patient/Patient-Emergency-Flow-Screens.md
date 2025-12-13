# EMERGENCY FLOW - UI DESIGN SCREENS

## Thông tin tài liệu
- **Tên dự án:** SnakeAid - AI-Powered Platform for Snakebite First Aid and Rescue Support
- **Module:** Patient Mobile Application
- **Flow:** Emergency Flow (Xử lý sự cố rắn cắn khẩn cấp)
- **Công cụ thiết kế:** Stitch with Google (prompt-based design)
- **Số lượng màn hình:** 9 screens
- **Ngày tạo:** November 30, 2025
- **Location:** `/02-UI-Design/Patient-Emergency-Flow-Screens.md`

---

## 🎨 Design System Overview

### Color Palette:
- **Primary Color:** Forest Green `#228B22`
- **Background:** White `#FFFFFF`
- **Text Primary:** Dark Gray `#333333`
- **Text Secondary:** Medium Gray `#666666`
- **Accent - Emergency:** Red `#DC3545`
- **Accent - Warning:** Amber `#FFC107`
- **Accent - Success:** Green `#28A745`
- **Accent - Info:** Blue `#007BFF`

### Typography:
- **Logo:** Bold, Large (32-36pt)
- **Headings:** Semi-bold (20-24pt)
- **Body Text:** Regular (16-18pt)
- **Button Text:** Medium (16pt)
- **Caption:** Regular (14pt)

### Component Style:
- **Cards:** Rounded corners (12px), subtle shadow
- **Buttons:** Rounded (8px), clear hierarchy (Primary/Secondary)
- **Input Fields:** Outlined style, rounded (8px)
- **Minimal icons:** Only essential ones (star rating, arrow navigation)

---

## 📱 SCREEN DESIGNS & PROMPTS

---

### Screen 1: Homepage (Patient App)

#### Thông tin màn hình:
- **Tên:** Màn hình chủ - Patient Dashboard
- **Mục đích:** Entry point của ứng dụng, cung cấp truy cập nhanh đến tính năng khẩn cấp và các chức năng chính
- **Flow position:** Điểm bắt đầu của Emergency Flow
- **Priority:** ⭐⭐⭐ (Cao nhất - cần chi tiết nhất)

#### Key Components:
1. **Header Section:**
   - Logo text "SnakeAid" (bold, forest green, centered)
   - User avatar icon (top-right corner, small circle)
   - Notification bell icon (top-right, minimal)

2. **Hero Emergency Card:**
   - Large card with red accent background (light red #FFEBEE)
   - Icon: Emergency symbol (or text "⚠️")
   - Main text: "Khẩn cấp - Tôi bị rắn cắn!"
   - Subtitle: "Nhận hướng dẫn sơ cứu ngay lập tức"
   - Right arrow indicator
   - Prominent position (top of content area)

3. **Quick Access Section:**
   - Title: "Truy cập nhanh"
   - 3 equal-width cards in a row:
     - Card 1: "Tìm bệnh viện" (with location pin icon or text)
     - Card 2: "Thông tin rắn" (with book/info icon or text)
     - Card 3: "Báo cáo rắn" (with camera icon or text)
   - Cards have forest green border

4. **Information Section:**
   - Title: "Phòng ngừa & Giáo dục"
   - Horizontal scrollable cards:
     - "Hướng dẫn sơ cứu"
     - "Rắn phổ biến"
     - "Mẹo an toàn"
   - Each card has thumbnail image placeholder + title

5. **Bottom Navigation Bar:**
   - 4 tabs with text labels:
     - "Trang chủ" (active - forest green)
     - "Cứu hộ"
     - "Chuyên gia"
     - "Cá nhân"
   - Active tab highlighted with forest green color

#### Stitch Prompt (English):

```
Mobile app home screen for emergency snakebite assistance app named "SnakeAid". Modern clean medical app design with forest green (#228B22) as primary brand color on white background.

Top header: Centered bold text logo "SnakeAid" in forest green. Small circular user avatar icon in top-right corner. Notification bell icon next to avatar.

Main content area begins with large prominent emergency card with light red background (#FFEBEE) and red accent border. Card contains warning symbol emoji, large bold text "Khẩn cấp - Tôi bị rắn cắn!", subtitle "Nhận hướng dẫn sơ cứu ngay lập tức", and right arrow. This card takes up full width with significant padding.

Below emergency card, section title "Truy cập nhanh" in dark gray. Three equal-width cards in horizontal row with forest green borders: "Tìm bệnh viện" with location pin, "Thông tin rắn" with info icon, "Báo cáo rắn" with camera icon. Cards have white background.

Next section titled "Phòng ngừa & Giáo dục" shows horizontally scrollable row of 3 cards. Each card has light gray rectangular placeholder for thumbnail image on top, and text label below: "Hướng dẫn sơ cứu", "Rắn phổ biến", "Mẹo an toàn".

Bottom of screen has fixed navigation bar with 4 evenly spaced text tabs: "Trang chủ" (active, forest green color), "Cứu hộ", "Chuyên gia", "Cá nhân" in gray. Clean separator line above nav bar.

Overall style: Clean, minimal, professional medical/emergency app, iOS and Android compatible, focus on typography and card-based layouts, subtle shadows, no complex illustrations.
```

#### Notes for Stitch:
- Nếu icons render không đẹp → Re-prompt: "Replace all icons with simple text labels only, no pictogram icons"
- Nếu màu emergency card quá sáng → "Use deeper red tint for emergency card background #FFCDD2"
- Nếu layout bị lệch → "Ensure all cards have equal padding and are vertically aligned"

---

### Screen 2: Emergency Alert Screen with Rescuer Finder

#### Thông tin màn hình:
- **Tên:** Màn hình cảnh báo khẩn cấp với tìm kiếm đội cứu hộ
- **Mục đích:** Xác nhận tình huống khẩn cấp, hiển thị map tìm Rescuer gần nhất real-time, và đưa ra hướng dẫn an toàn
- **Flow position:** Ngay sau khi user tap "Emergency - I'm Bitten" từ homepage
- **Priority:** ⭐⭐⭐
- **Design inspiration:** Grab-style modern map interface với radar scanning effect

#### Key Components:
1. **Header:**
   - Back button (top-left)
   - Title: "Cảnh báo khẩn cấp" (centered)
   - Close button (top-right, X icon)

2. **Alert Status Banner:**
   - Red background with white text
   - Large text: "Đang tìm đội cứu hộ gần bạn..."
   - Icon: Pulsing heartbeat animation
   - Status: "GPS đã kích hoạt"

3. **Interactive Map Section (占 40% screen height):**
   - Full-width map view (similar to Grab)
   - User's location: Blue pulsing dot in center
   - Radar scanning animation: Rotating green arc emanating from user location
   - Rescuer markers: Orange pins with rescuer icons appearing as they're found
   - Distance circles: 1km, 3km, 5km radius indicators (faint green lines)
   - Map controls:
     - Re-center button (bottom-right of map)
     - Current location indicator showing address
   - Overlay status card on map:
     - "Đang quét: 3 đội cứu hộ gần bạn"
     - "Khoảng cách gần nhất: 2.1 km"

4. **Bottom Sheet Panel (Slide-up drawer):**
   
   **Collapsed State (Shows top section):**
   - Drag handle bar at top
   - Quick stats: "3 đội cứu hộ | Gần nhất: 2.1km | ETA: 8 phút"
   - Swipe up indicator: "Vuốt lên để xem hướng dẫn sơ cứu"
   
   **Expanded State (Full panel):**
   
   a) **Critical Warning Section:**
      - Yellow warning box with amber background
      - Bold text: "⚠️ TUYỆT ĐỐI KHÔNG:"
      - Horizontal scrollable chips (save space):
        - "Cắt vết thương"
        - "Hút nọc độc"
        - "Đắp băng garo"
        - "Uống rượu"
   
   b) **Immediate Action Card:**
      - Green background card (compact)
      - Title: "✓ LÀM NGAY (Trong lúc chờ):"
      - 3 numbered items with icons:
        - "1️Giữ bình tĩnh và đứng yên"
        - "2️Cởi đồ/trang sức chật"
        - "3️Giữ vết cắn thấp hơn tim"
   
   c) **Rescuer Status (Dynamic):**
      - **Đang quét:** "Đang tìm đội cứu hộ gần bạn..."
        - Animated radar scanning
        - "Đã tìm thấy 3 đội cứu hộ trong phạm vi 5km"
      
      - **Đã gửi yêu cầu:** "Đã gửi yêu cầu khẩn cấp đến đội cứu hộ gần nhất"
        - Rescuer preview card (read-only):
          - Avatar + Name + Rating
          - "2.1 km - ETA 8 phút"
          - Status badge: "Đang chờ phản hồi..." (amber pulsing)
        - Text: "Đội cứu hộ có 60 giây để phản hồi"
        - Timer: "00:45"
      
      - **Đã chấp nhận:** "✅ Đội cứu hộ đã chấp nhận!"
        - Rescuer card with details
        - Status: "Đang trên đường đến"
        - Button: "Liên hệ đội cứu hộ"

5. **Action Buttons (Sticky bottom):**
   - **State 1 (Scanning):**
     - Primary button (large, red): "Gửi Yêu Cầu SOS →"
     - Secondary button (outlined, green): "Bắt đầu sơ cứu ngay"
   
   - **State 2 (Waiting for response):**
     - Primary button (large, red, disabled): "Đang chờ phản hồi..."
     - Secondary button (outlined, green): "Xem hướng dẫn sơ cứu"
     - Tertiary text link: "Hủy yêu cầu"
   
   - **State 3 (Accepted):**
     - Primary button (large, green): "Liên hệ đội cứu hộ"
     - Secondary button (outlined): "Xem vị trí đội cứu hộ"
     - Tertiary text link: "Gọi 115 trực tiếp"

#### Stitch Prompt (English):

```
Modern mobile app emergency screen with interactive map and rescuer finder. Grab-style interface with forest green (#228B22) and red (#DC3545) emergency theme.

Top navigation bar: Back arrow left, centered title "Cảnh báo khẩn cấp", X close button right. White background.

Full-width red status banner (#DC3545) below nav with white text "Đang tìm đội cứu hộ gần bạn..." and small pulsing heartbeat icon. Subtext "GPS đã kích hoạt" with green checkmark.

Large interactive map section (40% screen height): Light style map similar to Grab interface. Center shows blue pulsing dot (user location) with animated rotating green radar arc sweeping outward. Faint green concentric circles at 1km, 3km, 5km radius. Three orange pins with rescuer icons scattered on map at various distances. Small white floating card overlay on map bottom shows "Đang quét: 3 đội cứu hộ gần bạn | Khoảng cách: 2.1 km". Small circular re-center button bottom-right of map.

Below map, slide-up bottom sheet panel with rounded top corners and drag handle bar at top (gray horizontal line).

**Collapsed state** shows quick stats bar: "3 đội cứu hộ | Gần nhất: 2.1km | ETA: 8 phút" with small up arrow and text "Vuốt lên để xem hướng dẫn".

**Expanded state** shows full panel content:

Yellow-amber warning box (#FFF3CD) with bold text "⚠️ TUYỆT ĐỐI KHÔNG:" followed by horizontal scrollable row of 4 compact chips with red X icons: "Cắt vết thương", "Hút nọc độc", "Đắp băng garo", "Uống rượu".

Green success card (#D4EDDA) titled "✓ LÀM NGAY (Trong lúc chờ):" with 3 numbered compact items:
Giữ bình tĩnh và đứng yên
Cởi đồ/trang sức chật  
Giữ vết cắn thấp hơn tim

**Rescuer Status Section (showing "waiting for response" state):**
Gray text "Đã gửi yêu cầu khẩn cấp đến đội cứu hộ gần nhất"

White card with subtle shadow showing rescuer preview:
- Left: Small circular avatar placeholder
- Center: "Nguyễn Văn A" bold text, "4.9 ⭐ (156 đánh giá)" below
- Right: "2.1 km" bold orange text, "ETA 8 phút" gray text below
- Amber pulsing badge "Đang chờ phản hồi..." below avatar
- Small gray text "Đội cứu hộ có 60 giây để phản hồi"
- Timer display "00:45" in amber color

Sticky bottom section with white background and top shadow:
- Large disabled gray button "Đang chờ phản hồi..." (60px height)
- Medium outlined green button "Xem hướng dẫn sơ cứu" (50px height)
- Small gray text link "Hủy yêu cầu" centered

Design: Modern ride-hailing app style (Grab/Uber pattern), live map interface with radar animation, clear status updates, bottom sheet UX pattern, emergency medical context, professional and calming despite urgency.
```

#### Alternative States for Stitch:

**State 1 - Scanning (Before sending request):**
```
Map shows radar animation actively scanning. No rescuer card yet. Bottom shows animated text "Đang quét khu vực..." Primary red button "Gửi Yêu Cầu SOS →" enabled. Secondary green outlined button "Bắt đầu sơ cứu ngay".
```

**State 3 - Accepted (Rescuer confirmed):**
```
Rescuer card shows green checkmark badge "✅ Đã chấp nhận!". Status text "Đang trên đường đến". Map shows route line from rescuer pin to user location. Primary green button "Liên hệ đội cứu hộ" enabled. Secondary outlined "Xem vị trí đội cứu hộ". Tertiary link "Gọi 115 trực tiếp".
```

#### Notes for Stitch:
- Nếu text quá nhỏ → "Increase font size for DO NOT section to 18pt minimum"
- Nếu buttons không rõ hierarchy → "Make primary button 60px height, secondary 50px height"
- Alert banner phải nổi bật nhất trong screen

---

### Screen 3: First Aid Guide Screen

#### Thông tin màn hình:
- **Tên:** Màn hình hướng dẫn sơ cứu từng bước
- **Mục đích:** Cung cấp hướng dẫn sơ cứu chi tiết theo từng bước với hình ảnh minh họa
- **Flow position:** Sau Emergency Alert, khi user chọn "Start First Aid Guide"
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Progress indicator: "Bước 1 / 4" (text-based)
   - Timer: "02:15" (elapsed time)

2. **Step Indicator:**
   - Horizontal stepper: ●—○—○—○
   - Current step highlighted in forest green
   - Completed steps in green, upcoming in gray

3. **Instruction Card:**
   - Large card with white background
   - Step number badge (top-left): "BƯỚC 1"
   - Main heading: "Băng ép vết cắn"
   - Illustration placeholder: Rectangle area for image/diagram
   - Detailed text instructions (bullet points):
     - "Bắt đầu băng từ vị trí vết cắn"
     - "Băng chặt vừa phải, không quá chặt"
     - "Băng toàn bộ chi bị cắn"
     - "Kiểm tra tuần hoàn - ngón chân/tay vẫn hồng"

4. **Visual Aid Section:**
   - Image placeholder with caption: "Kỹ thuật băng ép đúng cách"
   - Border in forest green

5. **Navigation Buttons:**
   - Primary button (bottom): "Bước tiếp theo →" (forest green)
   - Secondary button: "⚠️ Tôi cần cấp cứu ngay" (red, outlined)
   - Skip option: "Bỏ qua đến tìm bệnh viện" (text link)

6. **Quick Access Bar (sticky footer above buttons):**
   - 3 small icon buttons:
     - "SOS"
     - "Bệnh viện"
     - "Chụp rắn"

#### Stitch Prompt (English):

```
Mobile app step-by-step first aid instruction screen. Clean educational interface with forest green (#228B22) theme.

Top navigation: Back arrow left, centered text "Bước 1 / 4", right side shows timer "02:15" in gray.

Below nav, horizontal progress stepper with 4 circles connected by lines. First circle filled green (active), others outlined gray. Circles contain step numbers 1-2-3-4.

Main content area: White card with subtle shadow containing step badge "BƯỚC 1" in forest green at top-left. Large heading "Băng ép vết cắn" in dark gray below badge.

Card contains rectangular placeholder area (16:9 ratio) with light gray background and centered text "Khu vực minh họa" for diagram image. Below illustration, 4 bullet points with clear instructions:
• "Bắt đầu băng từ vị trí vết cắn"
• "Băng chặt vừa phải, không quá chặt"  
• "Băng toàn bộ chi bị cắn"
• "Kiểm tra tuần hoàn - ngón chân/tay vẫn hồng"

Below main card, smaller image placeholder with forest green border and caption "Kỹ thuật băng ép đúng cách" underneath.

Above bottom buttons, sticky bar with 3 small equal-width outlined buttons labeled "SOS", "Bệnh viện", "Chụp rắn" with forest green borders.

Bottom has 2 full-width buttons stacked:
- Primary solid green button "Bước tiếp theo →"
- Secondary red outlined button "⚠️ Tôi cần cấp cứu ngay"
Small gray text link below "Bỏ qua đến tìm bệnh viện"

Style: Educational, calm, step-by-step tutorial interface, clear typography, adequate spacing, easy to read while stressed.
```

#### Notes for Stitch:
- Illustration area phải đủ lớn để user thấy rõ
- Text instructions phải có line height tốt (1.6-1.8) để dễ đọc
- Buttons phải đủ lớn cho emergency situation (min 50px height)

---

### Screen 4: Snake Photo Capture Screen

#### Thông tin màn hình:
- **Tên:** Màn hình chụp ảnh rắn để AI nhận diện
- **Mục đích:** Cho phép user chụp/upload ảnh rắn để hệ thống AI nhận diện loài
- **Flow position:** Có thể truy cập từ First Aid Guide hoặc từ Homepage
- **Priority:** ⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Nhận diện rắn"
   - Help icon (?)

2. **Camera Viewfinder Area:**
   - Large rectangle taking up most of screen
   - Dark overlay with center focus frame
   - Guide text overlay: "Đặt con rắn vào giữa khung hình"
   - Corner brackets to indicate focus area

3. **Safety Warning Banner (top of camera area):**
   - Yellow background strip
   - Text: "⚠️ Giữ khoảng cách an toàn - KHÔNG đến gần rắn"

4. **Instructions Panel (bottom overlay):**
   - Semi-transparent dark background
   - White text: "Mẹo để có kết quả tốt nhất:"
   - Bullet points:
     - "Chụp toàn thân nếu có thể"
     - "Tập trung vào đầu và hoa văn"
     - "Chụp từ khoảng cách an toàn"
     - "Sử dụng zoom nếu cần"

5. **Action Buttons (bottom):**
   - Large circular camera button (center, white)
   - Gallery icon button (left): "Tải ảnh lên"
   - Flash toggle (right): "Flash: Tắt"

6. **Skip Option:**
   - Text link: "Tôi không có ảnh rắn →"

#### Stitch Prompt (English):

```
Mobile app camera capture screen for snake identification. Camera viewfinder interface with safety warnings.

Top nav bar: Back arrow left, centered title "Nhận diện rắn", help icon (?) right. White background.

Main area shows camera viewfinder mockup: Large dark gray rectangle (#2C2C2C) representing camera view taking up 70% of vertical space. In center, white outlined frame/bracket corners indicating focus area. Inside frame, light gray text "Đặt con rắn vào giữa khung hình".

Top of camera area has yellow warning banner strip (#FFF3CD) with dark text "⚠️ Giữ khoảng cách an toàn - KHÔNG đến gần rắn" centered.

Bottom overlay on camera area: Semi-transparent dark panel (#000000 50% opacity) with white text. Title "Mẹo để có kết quả tốt nhất:" followed by 4 bullet points in smaller white text:
• "Chụp toàn thân nếu có thể"
• "Tập trung vào đầu và hoa văn"  
• "Chụp từ khoảng cách an toàn"
• "Sử dụng zoom nếu cần"

Below camera viewfinder, white bottom section with 3 buttons in horizontal row:
- Left: Small outlined button "Tải ảnh lên" with gallery icon
- Center: Large circular button (white fill, 80px diameter) for camera capture
- Right: Small outlined button "Flash: Tắt" with flash icon

At very bottom, centered gray text link "Tôi không có ảnh rắn →"

Style: Camera app interface, dark viewfinder, clear safety messaging, simple controls, iOS/Android standard camera UI patterns.
```

#### Notes for Stitch:
- Camera viewfinder area phải đủ lớn và nổi bật
- Warning banner phải prominent để user chú ý an toàn
- Nếu không render được camera effect → "Show placeholder camera area with dark background and center frame outline"

---

### Screen 5: AI Snake Identification Result

#### Thông tin màn hình:
- **Tên:** Màn hình kết quả nhận diện loài rắn bằng AI
- **Mục đích:** Hiển thị kết quả nhận diện rắn, mức độ độc tính, và hướng dẫn xử lý phù hợp
- **Flow position:** Sau khi AI xử lý ảnh từ Screen 4
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Kết quả nhận diện"
   - Share button (top-right)

2. **Result Status Badge:**
   - Top banner với màu theo mức độ nguy hiểm:
     - VENOMOUS (Red): "⚠️ PHÁT HIỆN RẮN ĐỘC"
     - NON-VENOMOUS (Green): "✓ Rắn không độc"
   - Large, prominent, full-width

3. **Snake Information Card:**
   - Snake photo (user's uploaded image)
   - Snake name:
     - English: "King Cobra"
     - Scientific: "Ophiophagus hannah"
     - Vietnamese: "Rắn hổ mang chúa"
   - Confidence score: "Độ tin cậy AI: 94%"

4. **Danger Level Section:**
   - Visual indicator: Red/Amber/Green bar
   - Text: "Mức độ nguy hiểm: CAO"
   - Description: "Có độc rất cao - Cần chăm sóc y tế ngay lập tức"

5. **Recommended Actions Card:**
   - Title: "Cần làm NGAY:"
   - Numbered action items:
     - "1. Gọi cấp cứu ngay lập tức"
     - "2. Băng ép vết cắn"
     - "3. Đến bệnh viện có huyết thanh gần nhất"
   - CTA button: "Tìm bệnh viện có huyết thanh →" (red primary button)

6. **Snake Details (Expandable Section):**
   - Collapsible panel: "Xem chi tiết rắn ▼"
   - When expanded shows:
     - Môi trường sống
     - Vị trí thường gặp
     - Hành vi thường thấy
     - Tác dụng của nha độc

7. **Bottom Actions:**
   - Secondary button: "Báo cáo lần nhìn thấy này"
   - Text link: "Không đúng? Chụp lại"

#### Stitch Prompt (English):

```
Mobile app screen showing AI snake identification results. Emergency medical information design with clear danger indicators.

Top nav: Back arrow left, title "Kết quả nhận diện", share icon right. White background.

Full-width top banner: Red background (#DC3545) with white bold text "⚠️ PHÁT HIỆN RẮN ĐỘC" centered. High visual prominence.

Below banner, white card with padding showing user's uploaded snake photo (square placeholder, rounded corners). Below photo, snake name displayed in hierarchical typography:
- Large bold text "King Cobra" (20pt)
- Italic gray text "Ophiophagus hannah" (16pt)  
- Regular text "Rắn hổ mang chúa" (16pt)
- Light gray text "Độ tin cậy AI: 94%" (14pt)

Next section shows danger indicator: Horizontal bar with gradient red-to-green, marker positioned at "CAO" level. Below bar, large text "Mức độ nguy hiểm: CAO" and description "Có độc rất cao - Cần chăm sóc y tế ngay lập tức" in dark gray.

White card titled "Cần làm NGAY:" containing 3 numbered items in bold:
1. Gọi cấp cứu ngay lập tức
2. Băng ép vết cắn  
3. Đến bệnh viện có huyết thanh gần nhất

Below list, large red primary button "Tìm bệnh viện có huyết thanh →" taking full card width.

Expandable section with forest green header bar "Xem chi tiết rắn ▼" (collapsed state shown).

Bottom of screen has 2 buttons:
- Secondary outlined button "Báo cáo lần nhìn thấy này"
- Small gray text link "Không đúng? Chụp lại"

Style: Emergency medical results interface, clear hierarchy, danger indicators prominent, actionable next steps emphasized, professional medical app design.
```

#### Notes for Stitch:
- Danger banner phải là element nổi bật nhất
- Phân biệt rõ giữa VENOMOUS (red) và NON-VENOMOUS (green) cases
- Confidence score giúp user đánh giá độ tin cậy
- Nếu expandable không render được → "Show as separate section with 'Details' heading"

---

### Screen 6: Symptom Input Screen

#### Thông tin màn hình:
- **Tên:** Màn hình nhập triệu chứng và chụp vết cắn
- **Mục đích:** Thu thập thông tin về triệu chứng và ảnh vết cắn để AI đánh giá mức độ nghiêm trọng
- **Flow position:** Sau AI Snake Identification hoặc từ Emergency Alert
- **Priority:** ⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Báo cáo triệu chứng"
   - Progress: "Bước 2 / 3"

2. **Photo Section:**
   - Title: "Ảnh vết cắn"
   - Large image upload area:
     - Dashed border rectangle
     - Camera icon
     - Text: "Nhấn để chụp hoặc tải ảnh"
   - If photo uploaded: Show thumbnail with edit/remove options
   - Helper text: "Điều này giúp đánh giá mức độ nghiêm trọng"

3. **Symptom Checklist:**
   - Title: "Chọn triệu chứng bạn đang gặp:"
   - Multi-select checkboxes (forest green when checked):
     - ☐ Đau tại vị trí vết cắn
     - ☐ Sưng tấy
     - ☐ Tê bỏi/Chốt mặt
     - ☐ Buồn nôn/Nôn mửa
     - ☐ Khó thở
     - ☐ Mờ mắt
     - ☐ Đổ mồ hôi nhiều
     - ☐ Chảy máu từ vết thương
     - ☐ Triệu chứng khác

4. **Severity Scale:**
   - Title: "Mức độ đau của bạn? (1-10)"
   - Visual slider from 1 (Nhẹ) to 10 (Nghiêm trọng)
   - Color gradient: Green → Yellow → Red
   - Current value displayed: "7"

5. **Time Since Bite:**
   - Title: "Thời gian kể từ khi bị cắn:"
   - Dropdown or picker: "15 phút trước"
   - Options: "Vừa xong", "5 phút", "15 phút", "30 phút", "1 giờ", "Trên 1 giờ"

6. **Additional Notes:**
   - Text area: "Thông tin bổ sung? (tùy chọn)"
   - Placeholder: "Mô tả các triệu chứng khác..."

7. **Action Buttons:**
   - Primary button: "Phân tích triệu chứng →" (forest green)
   - Secondary link: "Bỏ qua bước này"

#### Stitch Prompt (English):

```
Mobile app symptom input form screen for snakebite tracking. Clean medical form design.

Top nav: Back arrow left, centered title "Report Symptoms", right shows "Step 2 of 3" in gray.

First section titled "Photo of Bite Wound" in bold. Large rectangular upload area with dashed border (#CCCCCC), rounded corners, containing camera icon and centered text "Tap to capture or upload photo". Below upload area, small gray helper text "This helps assess severity".

Next section titled "Select symptoms you're experiencing:" with vertical list of checkboxes. 9 checkbox items with forest green checkmarks when selected:
□ Pain at bite site
□ Swelling  
□ Numbness/Tingling
□ Nausea/Vomiting
□ Difficulty breathing
□ Blurred vision
□ Excessive sweating
□ Bleeding from wound
□ Other symptoms

Below checkboxes, section titled "How would you rate the pain? (1-10)". Horizontal slider track with gradient from green (left) to yellow (center) to red (right). Labels "1 Mild" on left end, "10 Severe" on right end. Current value "7" displayed prominently above slider.

Next section titled "Time since bitten:" with dropdown/picker showing "15 minutes ago" with down arrow indicator.

Text area input labeled "Any other information? (optional)" with light gray placeholder text "Describe any other symptoms..." inside. Text area has light border, rounded corners.

Bottom has large primary button "Analyze Symptoms →" in forest green, full width. Small gray text link below button "Skip this step".

Style: Medical form interface, clear labels, adequate spacing between sections, touch-friendly inputs, professional healthcare app design.
```

#### Notes for Stitch:
- Checkboxes phải đủ lớn để easy to tap (min 44px touch target)
- Pain slider phải có visual feedback rõ ràng
- Photo upload area phải prominent
- Form validation cần rõ ràng nếu skip required fields

---

### Screen 7: Severity Assessment Result

#### Thông tin màn hình:
- **Tên:** Màn hình kết quả đánh giá mức độ nghiêm trọng
- **Mục đích:** Hiển thị kết quả phân tích AI về mức độ nguy hiểm và khuyến nghị hành động khẩn cấp
- **Flow position:** Sau khi AI phân tích symptoms từ Screen 6
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Đánh giá mức độ nghiêm trọng"
   - Time stamp: "Phân tích lúc 14:35"

2. **Severity Level Banner:**
   - Large top section với màu theo mức độ:
     - CRITICAL (Dark Red #C0392B): "🚨 NGHIÊM TRỌNG - CẦN CẤP CỨU NGAY"
     - SEVERE (Red #E74C3C): "⚠️ NẶNG - Đến bệnh viện NGAY"
     - MODERATE (Amber #F39C12): "⚠️ VỮA - Cần chăm sóc y tế"
     - MILD (Green #27AE60): "✓ NHẺ - Theo dõi triệu chứng"
   - Full-width, bold text, large font

3. **Assessment Score Card:**
   - Visual score: Circular progress indicator or bar (0-100)
   - Text: "Điểm mức độ: 85/100"
   - Color-coded based on severity
   - AI confidence: "Dựa trên triệu chứng và phân tích ảnh"

4. **Symptoms Summary:**
   - Title: "Các yếu tố nguy cơ:"
   - Icon list (red exclamation marks for critical symptoms):
     - ❗ Phát hiện khó thở
     - ❗ Mức độ đau cao (7/10)
     - ❗ Sưng tấy và tê bỏi
     - ⚠️ Xác nhận rắn độc
   - Time elapsed: "⏱️ 15 phút kể từ khi bị cắn"

5. **Immediate Actions Required:**
   - Large card with numbered urgent steps:
     - "1. GỌI CẤP CỨU NGAY"
     - "2. Đến bệnh viện gần nhất ngay lập tức"
     - "3. Thông báo người thân khẩn cấp"
     - "4. Tiếp tục sơ cứu trong khi chờ"

6. **Emergency Call Buttons:**
   - Large red primary button: "Gọi Đường dây nóng khẩn cấp" (with phone number)
   - Secondary button: "Tìm bệnh viện gần nhất →"
   - Tertiary button: "Gửi cảnh báo SOS"

7. **Progress Tracking:**
   - Text: "Triệu chứng của bạn đang được theo dõi"
   - Link: "Cập nhật triệu chứng" (if condition changes)

#### Stitch Prompt (English):

```
Mobile app emergency severity assessment results screen. High-urgency medical alert interface.

Top nav: Back arrow left, title "Đánh giá mức độ nghiêm trọng", timestamp "Phân tích lúc 14:35" in gray on right.

Large full-width banner at top with dark red background (#C0392B), white bold text "🚨 NGHIÊM TRỌNG - CẦN CẤP CỨU NGAY" centered. Very prominent, high contrast.

Below banner, white card showing circular severity indicator (85% filled in red) with large text "Điểm mức độ: 85/100" centered. Below score, small gray text "Dựa trên triệu chứng và phân tích ảnh".

Next white card titled "Các yếu tố nguy cơ:" with 4 items listed vertically, each with red exclamation icon:
❗ Phát hiện khó thở
❗ Mức độ đau cao (7/10)
❗ Sưng tấy và tê bỏi  
⚠️ Xác nhận rắn độc
Bottom of this card shows "⏱️ 15 phút kể từ khi bị cắn" in amber color.

Large white card titled "Cần làm NGAY:" containing 4 numbered items in bold text:
1. GỌI CẤP CỨU NGAY
2. Đến bệnh viện gần nhất ngay lập tức
3. Thông báo người thân khẩn cấp
4. Tiếp tục sơ cứu trong khi chờ

Bottom section has 3 vertically stacked buttons with spacing:
- Large red primary button "Gọi Đường dây nóng khẩn cấp" (60px height)
- Secondary outlined forest green button "Tìm bệnh viện gần nhất →"
- Tertiary outlined gray button "Gửi cảnh báo SOS"

At very bottom, small text "Triệu chứng của bạn đang được theo dõi" with link "Cập nhật triệu chứng" in forest green.

Style: Emergency medical alert interface, high urgency, clear hierarchy, critical information prominent, actionable buttons emphasized, professional medical emergency design.
```

#### Notes for Stitch:
- Severity banner màu phải thay đổi theo level: Critical (dark red), Severe (red), Moderate (amber), Mild (green)
- Score indicator phải rõ ràng và color-coded
- Call buttons phải largest và most prominent
- Layout phải work cho cả trường hợp Mild (ít urgent) và Critical

---

### Screen 8: SOS Emergency Call Screen

#### Thông tin màn hình:
- **Tên:** Màn hình gọi cấp cứu khẩn cấp - Kết nối với đội cứu hộ SnakeAid
- **Mục đích:** Kết nối với đội Rescuer/Supporter gần nhất, chia sẻ vị trí GPS, và hiển thị trạng thái chờ cứu hộ
- **Flow position:** Khi user nhấn nút SOS từ bất kỳ màn hình nào
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Title: "SOS Khẩn Cấp Đang Kích Hoạt"
   - Status indicator: Red pulsing dot
   - Cancel button (top-right): "Hủy SOS"

2. **Rescuer Matching Status:**
   - Large icon: Searching animation (radar/pulse effect)
   - Status text: "Đang tìm đội cứu hộ gần bạn..."
   - Then changes to: "✅ Đã tìm thấy đội cứu hộ!"
   - Rescuer info card:
     - Avatar/Name: "Nguyễn Văn A - Chuyên viên cứu hộ"
     - Rating: "4.9 (156 đánh giá)"
     - Distance: "2.1 km từ vị trí của bạn"
     - ETA: "Dự kiến đến trong 8 phút"

3. **GPS Location Card:**
   - Title: "Vị Trí Của Bạn"
   - Status: "✓ Đã chia sẻ vị trí GPS thành công"
   - Address text: "123 Nguyễn Huệ, Quận 1, TP.HCM"
   - Coordinates: "10.7769° N, 106.7009° E"
   - Small map preview showing user location + rescuer location
   - Button: "Cập Nhật Vị Trí"

4. **Information Sent:**
   - Title: "Thông Tin Đã Gửi Cho Đội Cứu Hộ:"
   - Checklist with green checkmarks:
     - ✓ Vị trí GPS của bạn (theo thời gian thực)
     - ✓ Loài rắn: Rắn hổ mang chúa (King Cobra)
     - ✓ Mức độ nguy hiểm: Nghiêm trọng
     - ✓ Triệu chứng: Khó thở, sưng tấy
     - ✓ Thời gian bị cắn: 15 phút trước

5. **Emergency Contact Notification:**
   - Text: "Đã thông báo người thân khẩn cấp:"
   - List: "• Nguyễn Văn B (Anh trai) - Đã gửi SMS"

6. **Timer:**
   - Large display: "Thời Gian Chờ: 02:35"
   - Subtitle: "Đội cứu hộ đang trên đường đến"

7. **Action Buttons:**
   - Large button: "Gọi Cho Đội Cứu Hộ" (forest green, primary)
   - Secondary button: "Xem Bệnh Viện Gần Nhất"
   - Tertiary button: "Gọi 115 (Cấp cứu y tế)" (outlined)
   - Text link: "Hủy Yêu Cầu SOS"

8. **Guidance Card (bottom):**
   - Title: "Trong Lúc Chờ Cứu Hộ:"
   - Bullet points:
     - "Giữ bình tĩnh và để điện thoại ở gần"
     - "Tiếp tục sơ cứu (băng ép)"
     - "Không ăn uống gì"
     - "Giữ vùng bị cắn thấp hơn tim"

#### Stitch Prompt (English):

```
Mobile app emergency SOS screen connecting to rescue team. Urgent rescue matching interface with GPS tracking and real-time rescuer location.

Top header with red background (#DC3545): White Vietnamese text "SOS Khẩn Cấp Đang Kích Hoạt" on left, small pulsing red dot indicator, white button text "Hủy SOS" on right.

Large central section showing rescuer matching status. Radar/pulse animation icon at top. Status text changes from "Đang tìm đội cứu hộ gần bạn..." to "✅ Đã tìm thấy đội cứu hộ!" in forest green.

White card showing matched rescuer profile:
- Small circular avatar placeholder
- Name: "Nguyễn Văn A - Chuyên viên cứu hộ" (bold, 18pt)
- Rating: "4.9 (156 đánh giá)" in gray
- Distance badge: "2.1 km từ vị trí của bạn" with location pin icon
- ETA: "Dự kiến đến trong 8 phút" in amber color (#FFC107)

White card section titled "Vị Trí Của Bạn" with green checkmark text "✓ Đã chia sẻ vị trí GPS thành công". Below shows Vietnamese address "123 Nguyễn Huệ, Quận 1, TP.HCM" and coordinates "10.7769° N, 106.7009° E" in smaller gray text. Small rectangular map preview showing two location pins (user in blue, rescuer in green). Small outlined button "Cập Nhật Vị Trí" at bottom of card.

Next card titled "Thông Tin Đã Gửi Cho Đội Cứu Hộ:" containing 5 lines with green checkmarks (Vietnamese text):
✓ Vị trí GPS của bạn (theo thời gian thực)
✓ Loài rắn: Rắn hổ mang chúa (King Cobra)
✓ Mức độ nguy hiểm: Nghiêm trọng
✓ Triệu chứng: Khó thở, sưng tấy
✓ Thời gian bị cắn: 15 phút trước

Small section showing "Đã thông báo người thân khẩn cấp:" with bullet point "• Nguyễn Văn B (Anh trai) - Đã gửi SMS".

Large timer display showing "Thời Gian Chờ: 02:35" in bold, large font. Subtitle below "Đội cứu hộ đang trên đường đến" in gray.

Four vertically stacked buttons with spacing:
- Large primary forest green button "Gọi Cho Đội Cứu Hộ" (60px height)
- Secondary outlined button "Xem Bệnh Viện Gần Nhất"
- Tertiary outlined gray button "Gọi 115 (Cấp cứu y tế)"
- Small gray text link "Hủy Yêu Cầu SOS"

Bottom card with light yellow background (#FFFACD) titled "Trong Lúc Chờ Cứu Hộ:" with 4 Vietnamese bullet points:
• Giữ bình tĩnh và để điện thoại ở gần
• Tiếp tục sơ cứu (băng ép)
• Không ăn uống gì
• Giữ vùng bị cắn thấp hơn tim

Style: Emergency rescue matching interface, rescuer profile prominent, real-time ETA tracking, GPS location sharing clear, calm but urgent design, Vietnamese text throughout, professional emergency rescue app design.
```

#### Notes for Stitch:
- Red header phải persistent để user biết SOS đang active
- Rescuer profile card phải nổi bật với avatar, rating, distance, ETA
- Map preview phải show 2 pins: user (blue) và rescuer (green) đang di chuyển
- Timer và ETA phải prominent và real-time update
- "Gọi 115" là option phụ (outlined button) - ưu tiên gọi Rescuer trước
- "While waiting" guidance critical để user không panic
- Toàn bộ text phải tiếng Việt

---

### Screen 9: Hospital Finder Map Screen

#### Thông tin màn hình:
- **Tên:** Màn hình bản đồ tìm kiếm bệnh viện có huyết thanh kháng nọc
- **Mục đích:** Hiển thị bản đồ các cơ sở y tế gần nhất có huyết thanh, khoảng cách, và chỉ đường
- **Flow position:** Từ Homepage, Emergency Alert, hoặc Severity Assessment
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button
   - Title: "Find Hospital"
   - Filter icon (top-right): "Filter by antivenom type"

2. **Search Bar:**
   - Search input: "Search by name or location..."
   - Current location button: "Dùng vị trí của tôi"

3. **Map View:**
   - Large map area (占 50-60% screen height)
   - Map placeholder with:
     - User's location pin (blue dot)
     - Hospital markers (red cross icons) with numbers
     - Distance circles overlay
   - Zoom controls (+/- buttons)

4. **Hospital List (Bottom Sheet / Scrollable List):**
   - List of 3-4 hospitals as cards, each containing:
     
     **Card 1 (Nearest):**
     - Hospital name: "Cho Ray Hospital"
     - Distance badge: "2.3 km" (forest green circle)
     - Estimated time: "8 phút lái xe"
     - Antivenom availability:
       - "✓ King Cobra antivenom available"
       - "✓ 24/7 Emergency service"
     - Rating: "4.8 (1,234 reviews)"
     - Primary button: "Get Directions →"
     - Secondary button: "Gọi bệnh viện"
     
     **Card 2:**
     - Similar structure with different distance: "5.1 km"
     - "✓ Multiple antivenom types"
     - "⚠️ Closes at 22:00"
     
     **Card 3:**
     - Distance: "8.7 km"

5. **Quick Filters (above list):**
   - Horizontal scrollable chips:
     - "Open Now" (selected - forest green)
     - "24/7"
     - "Has Antivenom"
     - "Closest"

6. **Bottom Info Banner:**
   - Light blue background
   - Text: "💡 Tip: Call ahead to confirm antivenom availability"

#### Stitch Prompt (English):

```
Mobile app hospital finder map screen for snakebite antivenom facilities. Map-based location finder with list view.

Top nav: Back arrow left, centered title "Tìm bệnh viện", filter icon right (funnel symbol).

Below nav, search bar with light gray background, rounded corners, placeholder text "Tìm theo tên hoặc vị trí..." with search icon. Small button on right "Dùng vị trí của tôi" in forest green text.

Large map area taking up 55% of screen height. Map placeholder shown as light gray rectangle with simple illustrated elements: blue dot for user location in center, 3-4 red cross markers around it representing hospitals numbered 1-3, faint distance circles. Small zoom buttons (+/-) in bottom-right corner of map.

Below map, horizontal row of filter chips (rounded pill buttons): "Đang mở cửa" (selected, forest green background), "24/7", "Có huyết thanh", "Gần nhất" (gray outlined).

Scrollable list of hospital cards below filters. First card most prominent:

Card 1 (white background, shadow, rounded corners):
- Bold text "Bệnh viện Chợ Rẫn" (18pt)
- Distance badge top-right: green circle with "2.3 km" in white
- Gray text "8 phút lái xe"
- Two lines with green checkmarks: "✓ Có huyết thanh King Cobra" and "✓ Cấp cứu 24/7"
- Rating line: "4.8 (1,234 đánh giá)" in gray
- Two buttons horizontally aligned: Primary green "Chỉ đường →" and secondary outlined "Gọi bệnh viện"

Card 2 visible below (partial):
- "Bệnh viện Quận 10"
- "5.1 km" badge
- "✓ Nhiều loại huyết thanh"
- "⚠️ Đóng cửa lúc 22:00"

At very bottom, light blue info banner (#E3F2FD) with text "💡 Mẹo: Gọi trước để xác nhận có huyết thanh".

Style: Map-based finder interface, clear geographic context, practical travel information, hospital cards with medical facility details, professional healthcare location finder design, iOS/Android map app patterns.
```

#### Notes for Stitch:
- Map area phải đủ lớn để user see context
- Hospital cards phải có clear hierarchy (nearest first)
- Distance và time estimates prominent
- "Get Directions" button phải clear CTA
- Nếu map không render tốt → "Show simplified map mockup with location pins and distance circles"
- Antivenom availability status critical - phải rõ ràng

---

## 📊 Screen Flow Diagram

```
┌─────────────────┐
│  1. Homepage    │
└────────┬────────┘
         │ User taps "Emergency - I'm Bitten"
         ▼
┌─────────────────┐
│ 2. Emergency    │
│    Alert        │
└────────┬────────┘
         │ Taps "Start First Aid Guide"
         ▼
┌─────────────────┐
│ 3. First Aid    │
│    Guide        │ ◄─── Can loop through Steps 1-4
└────────┬────────┘
         │ Parallel options
         ├──────────────────┐
         ▼                  ▼
┌─────────────────┐  ┌─────────────────┐
│ 4. Snake Photo  │  │ 6. Symptom      │
│    Capture      │  │    Input        │
└────────┬────────┘  └────────┬────────┘
         │                    │
         ▼                    ▼
┌─────────────────┐  ┌─────────────────┐
│ 5. AI Snake     │  │ 7. Severity     │
│    Identification│  │   Assessment    │
└────────┬────────┘  └────────┬────────┘
         │                    │
         └────────┬───────────┘
                  ▼
         ┌─────────────────┐
         │ 8. SOS          │ ◄─── Can trigger from any screen
         │    Emergency    │
         └────────┬────────┘
                  │
                  ▼
         ┌─────────────────┐
         │ 9. Hospital     │ ◄─── Can access from multiple screens
         │    Finder Map   │
         └─────────────────┘
```

---

## ✅ Checklist trước khi dùng Stitch

### Chuẩn bị:
- [ ] Đã đọc kỹ prompt tiếng Anh cho từng màn hình
- [ ] Đã hiểu rõ Key Components của mỗi screen
- [ ] Đã có brand colors: Forest Green #228B22
- [ ] Đã có logo text "SnakeAid" (bold font)

### Khi sử dụng Stitch:
- [ ] Copy-paste prompt NGUYÊN VĂN vào Stitch
- [ ] Nếu kết quả không đúng → đọc Notes section để refine
- [ ] Generate từng screen một, không generate hết 9 screens cùng lúc
- [ ] Save mỗi screen với tên rõ ràng: "SnakeAid_01_Homepage.png"

### Sau khi generate:
- [ ] Check màu sắc đúng brand (Forest Green)
- [ ] Check hierarchy rõ ràng (CTA buttons prominent)
- [ ] Check readability (font size đủ lớn)
- [ ] Check touch targets (buttons min 44-50px height)

### Nếu gặp vấn đề:
- **Icons xấu/không chuyên nghiệp:** Re-prompt: "Remove all icons, use text labels only"
- **Màu sai:** "Use exactly #228B22 for forest green"
- **Layout lộn xộn:** "Increase spacing between cards, use 16px padding"
- **Text quá nhỏ:** "Increase font size to 16pt minimum for body text"

---

## 🎨 Tips cho thiết kế tiếp theo:

1. **Test với 3 screens đầu tiên** (Homepage, Emergency Alert, First Aid Guide) trước
2. **Refine prompt** dựa trên kết quả thực tế từ Stitch
3. **Maintain consistency** về spacing, colors, typography giữa các screens
4. **Document changes** nếu cần adjust prompts
5. Sau khi có 9 screens → **import vào Figma** để tạo prototype với transitions

---

## 📝 Version History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | Nov 30, 2025 | Initial creation - 9 screens with Stitch prompts | AI Assistant |
| 1.1 | Nov 30, 2025 | Moved to `/02-UI-Design/` folder (proper location) | AI Assistant |

---

**Next Steps:**
1. Copy prompts vào Stitch with Google
2. Generate từng screen
3. Review và refine nếu cần
4. Import vào Figma để tạo interactive prototype
5. Tạo UI Design doc cho các flows khác (Rescue, Expert, Admin)

---

*Document này là phần của SnakeAid Project Documentation*
*Để cập nhật hoặc feedback, liên hệ team lead*
