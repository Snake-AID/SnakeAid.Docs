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
   - Main text: "Emergency - I'm Bitten!"
   - Subtitle: "Get immediate first aid guidance"
   - Right arrow indicator
   - Prominent position (top of content area)

3. **Quick Access Section:**
   - Title: "Quick Actions"
   - 3 equal-width cards in a row:
     - Card 1: "Find Hospital" (with location pin icon or text)
     - Card 2: "Snake Info" (with book/info icon or text)
     - Card 3: "Report Snake" (with camera icon or text)
   - Cards have forest green border

4. **Information Section:**
   - Title: "Prevention & Education"
   - Horizontal scrollable cards:
     - "First Aid Guide"
     - "Common Snakes"
     - "Safety Tips"
   - Each card has thumbnail image placeholder + title

5. **Bottom Navigation Bar:**
   - 4 tabs with text labels:
     - "Home" (active - forest green)
     - "Rescuer"
     - "Expert"
     - "Profile"
   - Active tab highlighted with forest green color

#### Stitch Prompt (English):

```
Mobile app home screen for emergency snakebite assistance app named "SnakeAid". Modern clean medical app design with forest green (#228B22) as primary brand color on white background.

Top header: Centered bold text logo "SnakeAid" in forest green. Small circular user avatar icon in top-right corner. Notification bell icon next to avatar.

Main content area begins with large prominent emergency card with light red background (#FFEBEE) and red accent border. Card contains warning symbol emoji, large bold text "Emergency - I'm Bitten!", subtitle "Get immediate first aid guidance", and right arrow. This card takes up full width with significant padding.

Below emergency card, section title "Quick Actions" in dark gray. Three equal-width cards in horizontal row with forest green borders: "Find Hospital" with location pin, "Snake Info" with info icon, "Report Snake" with camera icon. Cards have white background.

Next section titled "Prevention & Education" shows horizontally scrollable row of 3 cards. Each card has light gray rectangular placeholder for thumbnail image on top, and text label below: "First Aid Guide", "Common Snakes", "Safety Tips".

Bottom of screen has fixed navigation bar with 4 evenly spaced text tabs: "Home" (active, forest green color), "Rescuer", "Expert", "Profile" in gray. Clean separator line above nav bar.

Overall style: Clean, minimal, professional medical/emergency app, iOS and Android compatible, focus on typography and card-based layouts, subtle shadows, no complex illustrations.
```

#### Notes for Stitch:
- Nếu icons render không đẹp → Re-prompt: "Replace all icons with simple text labels only, no pictogram icons"
- Nếu màu emergency card quá sáng → "Use deeper red tint for emergency card background #FFCDD2"
- Nếu layout bị lệch → "Ensure all cards have equal padding and are vertically aligned"

---

### Screen 2: Emergency Alert Screen

#### Thông tin màn hình:
- **Tên:** Màn hình cảnh báo khẩn cấp
- **Mục đích:** Xác nhận người dùng đang trong tình huống khẩn cấp, đưa ra hướng dẫn nhanh trước khi chuyển sang first aid
- **Flow position:** Ngay sau khi user tap "Emergency - I'm Bitten" từ homepage
- **Priority:** ⭐⭐⭐

#### Key Components:
1. **Header:**
   - Back button (top-left)
   - Title: "Emergency Alert" (centered)
   - Close button (top-right, X icon)

2. **Alert Banner:**
   - Red background with white text
   - Large text: "Stay Calm - We're Here to Help"
   - Icon: Heartbeat or medical cross

3. **Critical Warning Section:**
   - Yellow warning box with amber background
   - Bold text: "⚠️ DO NOT:"
   - Bullet list:
     - "Cut the wound"
     - "Suck out venom"
     - "Apply ice or tourniquet"
     - "Drink alcohol"

4. **Immediate Action Card:**
   - Green background card
   - Title: "✓ DO THIS NOW:"
   - Step 1: "Stay calm and still"
   - Step 2: "Remove tight clothing/jewelry"
   - Step 3: "Keep bitten area below heart level"

5. **Action Buttons:**
   - Primary button (large, red): "Start First Aid Guide →"
   - Secondary button (outlined): "Call Emergency Hotline"
   - Tertiary text link: "I'm not bitten, I just saw a snake"

#### Stitch Prompt (English):

```
Mobile app emergency alert screen for snakebite app. Full-screen urgent design with clear visual hierarchy.

Top navigation bar: Back arrow on left, centered title "Emergency Alert", X close button on right. White background nav bar.

Full-width red banner (#DC3545) at top of content with white text "Stay Calm - We're Here to Help" in large bold font. Small medical cross icon on left side of text.

Below banner, prominent yellow-amber warning box (#FFF3CD with #FFC107 border) containing warning emoji and bold text "⚠️ DO NOT:" followed by 4 bullet points in dark text: "Cut the wound", "Suck out venom", "Apply ice or tourniquet", "Drink alcohol". Each point on separate line with bullet.

Below warning box, green success-style card (#D4EDDA background with #28A745 border) titled "✓ DO THIS NOW:" with 3 numbered steps in dark text: "1. Stay calm and still", "2. Remove tight clothing/jewelry", "3. Keep bitten area below heart level".

Bottom section has 3 vertically stacked buttons with spacing: 
- Large primary red button (#DC3545) "Start First Aid Guide →"
- Secondary outlined button "Call Emergency Hotline"
- Small text link in gray "I'm not bitten, I just saw a snake"

Design: Emergency medical interface, high contrast, clear readability, urgent but not panic-inducing, mobile-friendly touch targets.
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
   - Progress indicator: "Step 1 of 4" (text-based)
   - Timer: "02:15" (elapsed time)

2. **Step Indicator:**
   - Horizontal stepper: ●—○—○—○
   - Current step highlighted in forest green
   - Completed steps in green, upcoming in gray

3. **Instruction Card:**
   - Large card with white background
   - Step number badge (top-left): "STEP 1"
   - Main heading: "Apply Pressure Bandage"
   - Illustration placeholder: Rectangle area for image/diagram
   - Detailed text instructions (bullet points):
     - "Start bandaging at bite site"
     - "Wrap firmly but not too tight"
     - "Cover entire affected limb"
     - "Check circulation - toes/fingers should remain pink"

4. **Visual Aid Section:**
   - Image placeholder with caption: "Correct bandaging technique"
   - Border in forest green

5. **Navigation Buttons:**
   - Primary button (bottom): "Next Step →" (forest green)
   - Secondary button: "⚠️ I Need Emergency Help Now" (red, outlined)
   - Skip option: "Skip to hospital finder" (text link)

6. **Quick Access Bar (sticky footer above buttons):**
   - 3 small icon buttons:
     - "📞 SOS"
     - "🏥 Hospital"
     - "📷 Photo Snake"

#### Stitch Prompt (English):

```
Mobile app step-by-step first aid instruction screen. Clean educational interface with forest green (#228B22) theme.

Top navigation: Back arrow left, centered text "Step 1 of 4", right side shows timer "02:15" in gray.

Below nav, horizontal progress stepper with 4 circles connected by lines. First circle filled green (active), others outlined gray. Circles contain step numbers 1-2-3-4.

Main content area: White card with subtle shadow containing step badge "STEP 1" in forest green at top-left. Large heading "Apply Pressure Bandage" in dark gray below badge.

Card contains rectangular placeholder area (16:9 ratio) with light gray background and centered text "Illustration Area" for diagram image. Below illustration, 4 bullet points with clear instructions:
• "Start bandaging at bite site"
• "Wrap firmly but not too tight"  
• "Cover entire affected limb"
• "Check circulation - toes/fingers should remain pink"

Below main card, smaller image placeholder with forest green border and caption "Correct bandaging technique" underneath.

Above bottom buttons, sticky bar with 3 small equal-width outlined buttons labeled "📞 SOS", "🏥 Hospital", "📷 Photo Snake" with forest green borders.

Bottom has 2 full-width buttons stacked:
- Primary solid green button "Next Step →"
- Secondary red outlined button "⚠️ I Need Emergency Help Now"
Small gray text link below "Skip to hospital finder"

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
   - Title: "Identify Snake"
   - Help icon (?)

2. **Camera Viewfinder Area:**
   - Large rectangle taking up most of screen
   - Dark overlay with center focus frame
   - Guide text overlay: "Frame the snake in the center"
   - Corner brackets to indicate focus area

3. **Safety Warning Banner (top of camera area):**
   - Yellow background strip
   - Text: "⚠️ Keep safe distance - Do NOT approach snake"

4. **Instructions Panel (bottom overlay):**
   - Semi-transparent dark background
   - White text: "Tips for best results:"
   - Bullet points:
     - "Capture full body if possible"
     - "Focus on head and pattern"
     - "Take photo from safe distance"
     - "Use zoom if needed"

5. **Action Buttons (bottom):**
   - Large circular camera button (center, white)
   - Gallery icon button (left): "Upload Photo"
   - Flash toggle (right): "Flash: Off"

6. **Skip Option:**
   - Text link: "I don't have photo of snake →"

#### Stitch Prompt (English):

```
Mobile app camera capture screen for snake identification. Camera viewfinder interface with safety warnings.

Top nav bar: Back arrow left, centered title "Identify Snake", help icon (?) right. White background.

Main area shows camera viewfinder mockup: Large dark gray rectangle (#2C2C2C) representing camera view taking up 70% of vertical space. In center, white outlined frame/bracket corners indicating focus area. Inside frame, light gray text "Frame the snake in the center".

Top of camera area has yellow warning banner strip (#FFF3CD) with dark text "⚠️ Keep safe distance - Do NOT approach snake" centered.

Bottom overlay on camera area: Semi-transparent dark panel (#000000 50% opacity) with white text. Title "Tips for best results:" followed by 3 bullet points in smaller white text:
• "Capture full body if possible"
• "Focus on head and pattern"  
• "Take photo from safe distance"

Below camera viewfinder, white bottom section with 3 buttons in horizontal row:
- Left: Small outlined button "Upload Photo" with gallery icon
- Center: Large circular button (white fill, 80px diameter) for camera capture
- Right: Small outlined button "Flash: Off" with flash icon

At very bottom, centered gray text link "I don't have photo of snake →"

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
   - Title: "Identification Result"
   - Share button (top-right)

2. **Result Status Badge:**
   - Top banner với màu theo mức độ nguy hiểm:
     - VENOMOUS (Red): "⚠️ VENOMOUS SNAKE DETECTED"
     - NON-VENOMOUS (Green): "✓ Non-Venomous Snake"
   - Large, prominent, full-width

3. **Snake Information Card:**
   - Snake photo (user's uploaded image)
   - Snake name:
     - English: "King Cobra"
     - Scientific: "Ophiophagus hannah"
     - Vietnamese: "Rắn hổ mang chúa"
   - Confidence score: "AI Confidence: 94%"

4. **Danger Level Section:**
   - Visual indicator: Red/Amber/Green bar
   - Text: "Danger Level: HIGH"
   - Description: "Highly venomous - Requires immediate medical attention"

5. **Recommended Actions Card:**
   - Title: "What to do NOW:"
   - Numbered action items:
     - "1. Call emergency services immediately"
     - "2. Apply pressure bandage"
     - "3. Get to nearest hospital with antivenom"
   - CTA button: "Find Hospital with Antivenom →" (red primary button)

6. **Snake Details (Expandable Section):**
   - Collapsible panel: "View Snake Details ▼"
   - When expanded shows:
     - Habitat
     - Common locations
     - Typical behavior
     - Venom effects

7. **Bottom Actions:**
   - Secondary button: "Report This Sighting"
   - Text link: "Not correct? Retake photo"

#### Stitch Prompt (English):

```
Mobile app screen showing AI snake identification results. Emergency medical information design with clear danger indicators.

Top nav: Back arrow left, title "Identification Result", share icon right. White background.

Full-width top banner: Red background (#DC3545) with white bold text "⚠️ VENOMOUS SNAKE DETECTED" centered. High visual prominence.

Below banner, white card with padding showing user's uploaded snake photo (square placeholder, rounded corners). Below photo, snake name displayed in hierarchical typography:
- Large bold text "King Cobra" (20pt)
- Italic gray text "Ophiophagus hannah" (16pt)  
- Regular text "Rắn hổ mang chúa" (16pt)
- Light gray text "AI Confidence: 94%" (14pt)

Next section shows danger indicator: Horizontal bar with gradient red-to-green, marker positioned at "HIGH" level. Below bar, large text "Danger Level: HIGH" and description "Highly venomous - Requires immediate medical attention" in dark gray.

White card titled "What to do NOW:" containing 3 numbered items in bold:
1. Call emergency services immediately
2. Apply pressure bandage  
3. Get to nearest hospital with antivenom

Below list, large red primary button "Find Hospital with Antivenom →" taking full card width.

Expandable section with forest green header bar "View Snake Details ▼" (collapsed state shown).

Bottom of screen has 2 buttons:
- Secondary outlined button "Report This Sighting"
- Small gray text link "Not correct? Retake photo"

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
   - Title: "Report Symptoms"
   - Progress: "Step 2 of 3"

2. **Photo Section:**
   - Title: "Photo of Bite Wound"
   - Large image upload area:
     - Dashed border rectangle
     - Camera icon
     - Text: "Tap to capture or upload photo"
   - If photo uploaded: Show thumbnail with edit/remove options
   - Helper text: "This helps assess severity"

3. **Symptom Checklist:**
   - Title: "Select symptoms you're experiencing:"
   - Multi-select checkboxes (forest green when checked):
     - ☐ Pain at bite site
     - ☐ Swelling
     - ☐ Numbness/Tingling
     - ☐ Nausea/Vomiting
     - ☐ Difficulty breathing
     - ☐ Blurred vision
     - ☐ Excessive sweating
     - ☐ Bleeding from wound
     - ☐ Other symptoms

4. **Severity Scale:**
   - Title: "How would you rate the pain? (1-10)"
   - Visual slider from 1 (Mild) to 10 (Severe)
   - Color gradient: Green → Yellow → Red
   - Current value displayed: "7"

5. **Time Since Bite:**
   - Title: "Time since bitten:"
   - Dropdown or picker: "15 minutes ago"
   - Options: "Just now", "5 min", "15 min", "30 min", "1 hour", "More than 1 hour"

6. **Additional Notes:**
   - Text area: "Any other information? (optional)"
   - Placeholder: "Describe any other symptoms..."

7. **Action Buttons:**
   - Primary button: "Analyze Symptoms →" (forest green)
   - Secondary link: "Skip this step"

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
   - Title: "Severity Assessment"
   - Time stamp: "Analyzed at 14:35"

2. **Severity Level Banner:**
   - Large top section với màu theo mức độ:
     - CRITICAL (Dark Red #C0392B): "🚨 CRITICAL - SEEK IMMEDIATE HELP"
     - SEVERE (Red #E74C3C): "⚠️ SEVERE - Go to hospital NOW"
     - MODERATE (Amber #F39C12): "⚠️ MODERATE - Medical attention needed"
     - MILD (Green #27AE60): "✓ MILD - Monitor symptoms"
   - Full-width, bold text, large font

3. **Assessment Score Card:**
   - Visual score: Circular progress indicator or bar (0-100)
   - Text: "Severity Score: 85/100"
   - Color-coded based on severity
   - AI confidence: "Based on reported symptoms and photo analysis"

4. **Symptoms Summary:**
   - Title: "Identified Risk Factors:"
   - Icon list (red exclamation marks for critical symptoms):
     - ❗ Difficulty breathing detected
     - ❗ High pain level (7/10)
     - ❗ Swelling and numbness
     - ⚠️ Venomous snake confirmed
   - Time elapsed: "⏱️ 15 minutes since bite"

5. **Immediate Actions Required:**
   - Large card with numbered urgent steps:
     - "1. 🚨 CALL EMERGENCY SERVICES NOW"
     - "2. 🏥 Get to nearest hospital immediately"
     - "3. 📞 Notify emergency contact"
     - "4. 🩹 Continue first aid while waiting"

6. **Emergency Call Buttons:**
   - Large red primary button: "📞 Call Emergency Hotline" (with phone number)
   - Secondary button: "Find Nearest Hospital →"
   - Tertiary button: "Send SOS Alert"

7. **Progress Tracking:**
   - Text: "Your symptoms are being monitored"
   - Link: "Update symptoms" (if condition changes)

#### Stitch Prompt (English):

```
Mobile app emergency severity assessment results screen. High-urgency medical alert interface.

Top nav: Back arrow left, title "Severity Assessment", timestamp "Analyzed at 14:35" in gray on right.

Large full-width banner at top with dark red background (#C0392B), white bold text "🚨 CRITICAL - SEEK IMMEDIATE HELP" centered. Very prominent, high contrast.

Below banner, white card showing circular severity indicator (85% filled in red) with large text "Severity Score: 85/100" centered. Below score, small gray text "Based on reported symptoms and photo analysis".

Next white card titled "Identified Risk Factors:" with 4 items listed vertically, each with red exclamation icon:
❗ Difficulty breathing detected
❗ High pain level (7/10)
❗ Swelling and numbness  
⚠️ Venomous snake confirmed
Bottom of this card shows "⏱️ 15 minutes since bite" in amber color.

Large white card titled "Immediate Actions Required:" containing 4 numbered items with emojis in bold text:
1. 🚨 CALL EMERGENCY SERVICES NOW
2. 🏥 Get to nearest hospital immediately
3. 📞 Notify emergency contact
4. 🩹 Continue first aid while waiting

Bottom section has 3 vertically stacked buttons with spacing:
- Large red primary button "📞 Call Emergency Hotline" (60px height)
- Secondary outlined forest green button "Find Nearest Hospital →"
- Tertiary outlined gray button "Send SOS Alert"

At very bottom, small text "Your symptoms are being monitored" with link "Update symptoms" in forest green.

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
   - Title: "🚨 SOS Khẩn Cấp Đang Kích Hoạt"
   - Status indicator: Red pulsing dot
   - Cancel button (top-right): "Hủy SOS"

2. **Rescuer Matching Status:**
   - Large icon: Searching animation (radar/pulse effect)
   - Status text: "Đang tìm đội cứu hộ gần bạn..."
   - Then changes to: "✅ Đã tìm thấy đội cứu hộ!"
   - Rescuer info card:
     - Avatar/Name: "Nguyễn Văn A - Chuyên viên cứu hộ"
     - Rating: "⭐ 4.9 (156 đánh giá)"
     - Distance: "2.1 km từ vị trí của bạn"
     - ETA: "🚗 Dự kiến đến trong 8 phút"

3. **GPS Location Card:**
   - Title: "📍 Vị Trí Của Bạn"
   - Status: "✓ Đã chia sẻ vị trí GPS thành công"
   - Address text: "123 Nguyễn Huệ, Quận 1, TP.HCM"
   - Coordinates: "10.7769° N, 106.7009° E"
   - Small map preview showing user location + rescuer location
   - Button: "Cập Nhật Vị Trí"

4. **Information Sent:**
   - Title: "📤 Thông Tin Đã Gửi Cho Đội Cứu Hộ:"
   - Checklist with green checkmarks:
     - ✓ Vị trí GPS của bạn (theo thời gian thực)
     - ✓ Loài rắn: Rắn hổ mang chúa (King Cobra)
     - ✓ Mức độ nguy hiểm: Nghiêm trọng
     - ✓ Triệu chứng: Khó thở, sưng tấy
     - ✓ Thời gian bị cắn: 15 phút trước

5. **Emergency Contact Notification:**
   - Text: "📧 Đã thông báo người thân khẩn cấp:"
   - List: "• Nguyễn Văn B (Anh trai) - Đã gửi SMS"

6. **Timer:**
   - Large display: "Thời Gian Chờ: 02:35"
   - Subtitle: "Đội cứu hộ đang trên đường đến"

7. **Action Buttons:**
   - Large button: "📞 Gọi Cho Đội Cứu Hộ" (forest green, primary)
   - Secondary button: "🏥 Xem Bệnh Viện Gần Nhất"
   - Tertiary button: "☎️ Gọi 115 (Cấp cứu y tế)" (outlined)
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

Top header with red background (#DC3545): White Vietnamese text "🚨 SOS Khẩn Cấp Đang Kích Hoạt" on left, small pulsing red dot indicator, white button text "Hủy SOS" on right.

Large central section showing rescuer matching status. Radar/pulse animation icon at top. Status text changes from "Đang tìm đội cứu hộ gần bạn..." to "✅ Đã tìm thấy đội cứu hộ!" in forest green.

White card showing matched rescuer profile:
- Small circular avatar placeholder
- Name: "Nguyễn Văn A - Chuyên viên cứu hộ" (bold, 18pt)
- Rating: "⭐ 4.9 (156 đánh giá)" in gray
- Distance badge: "2.1 km từ vị trí của bạn" with location pin icon
- ETA: "🚗 Dự kiến đến trong 8 phút" in amber color (#FFC107)

White card section titled "📍 Vị Trí Của Bạn" with green checkmark text "✓ Đã chia sẻ vị trí GPS thành công". Below shows Vietnamese address "123 Nguyễn Huệ, Quận 1, TP.HCM" and coordinates "10.7769° N, 106.7009° E" in smaller gray text. Small rectangular map preview showing two location pins (user in blue, rescuer in green). Small outlined button "Cập Nhật Vị Trí" at bottom of card.

Next card titled "📤 Thông Tin Đã Gửi Cho Đội Cứu Hộ:" containing 5 lines with green checkmarks (Vietnamese text):
✓ Vị trí GPS của bạn (theo thời gian thực)
✓ Loài rắn: Rắn hổ mang chúa (King Cobra)
✓ Mức độ nguy hiểm: Nghiêm trọng
✓ Triệu chứng: Khó thở, sưng tấy
✓ Thời gian bị cắn: 15 phút trước

Small section showing "📧 Đã thông báo người thân khẩn cấp:" with bullet point "• Nguyễn Văn B (Anh trai) - Đã gửi SMS".

Large timer display showing "Thời Gian Chờ: 02:35" in bold, large font. Subtitle below "Đội cứu hộ đang trên đường đến" in gray.

Four vertically stacked buttons with spacing:
- Large primary forest green button "📞 Gọi Cho Đội Cứu Hộ" (60px height)
- Secondary outlined button "🏥 Xem Bệnh Viện Gần Nhất"
- Tertiary outlined gray button "☎️ Gọi 115 (Cấp cứu y tế)"
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
   - Current location button: "📍 Use my location"

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
     - Estimated time: "🚗 8 mins drive"
     - Antivenom availability:
       - "✓ King Cobra antivenom available"
       - "✓ 24/7 Emergency service"
     - Rating: "⭐ 4.8 (1,234 reviews)"
     - Primary button: "Get Directions →"
     - Secondary button: "📞 Call Hospital"
     
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

Top nav: Back arrow left, centered title "Find Hospital", filter icon right (funnel symbol).

Below nav, search bar with light gray background, rounded corners, placeholder text "Search by name or location..." with search icon. Small button on right "📍 Use my location" in forest green text.

Large map area taking up 55% of screen height. Map placeholder shown as light gray rectangle with simple illustrated elements: blue dot for user location in center, 3-4 red cross markers around it representing hospitals numbered 1-3, faint distance circles. Small zoom buttons (+/-) in bottom-right corner of map.

Below map, horizontal row of filter chips (rounded pill buttons): "Open Now" (selected, forest green background), "24/7", "Has Antivenom", "Closest" (gray outlined).

Scrollable list of hospital cards below filters. First card most prominent:

Card 1 (white background, shadow, rounded corners):
- Bold text "Cho Ray Hospital" (18pt)
- Distance badge top-right: green circle with "2.3 km" in white
- Gray text "🚗 8 mins drive"
- Two lines with green checkmarks: "✓ King Cobra antivenom available" and "✓ 24/7 Emergency service"
- Rating line: "⭐ 4.8 (1,234 reviews)" in gray
- Two buttons horizontally aligned: Primary green "Get Directions →" and secondary outlined "📞 Call Hospital"

Card 2 visible below (partial):
- "District 10 Hospital"
- "5.1 km" badge
- "✓ Multiple antivenom types"
- "⚠️ Closes at 22:00"

At very bottom, light blue info banner (#E3F2FD) with text "💡 Tip: Call ahead to confirm antivenom availability".

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
