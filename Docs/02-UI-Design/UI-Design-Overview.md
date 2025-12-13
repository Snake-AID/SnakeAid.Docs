# UI DESIGN OVERVIEW - SNAKEAID PLATFORM

## Thông tin tổng quan
- **Tên dự án:** SnakeAid - AI-Powered Platform for Snakebite First Aid and Rescue Support
- **Mục đích:** Quản lý và tracking toàn bộ UI designs cho các roles trong hệ thống
- **Công cụ thiết kế:** Stitch with Google (prompt-based design)
- **Ngày tạo:** December 4, 2025
- **Location:** `/02-UI-Design/UI-Design-Overview.md`

---

## 🎯 ROLES & APPLICATIONS

SnakeAid platform có **4 roles chính**, mỗi role có application riêng:

### 1. 🧑 **PATIENT** (Bệnh nhân / Người dùng)
- **Platform:** Mobile Application (iOS + Android)
- **Main Features:**
  - Emergency first aid guidance
  - AI snake identification
  - Hospital location finder
  - Request snake rescue service
  - Expert consultation booking
  - Payment & rating system

### 2. 🚑 **SNAKE RESCUER** (Đội cứu hộ rắn)
- **Platform:** Mobile Application (iOS + Android)
- **Main Features:**
  - Receive rescue alerts
  - Accept/decline rescue requests
  - GPS navigation & live tracking
  - Safety guidelines for snake handling
  - Revenue management
  - Communication with experts

### 3. 🧑‍🔬 **SNAKE EXPERT** (Chuyên gia về rắn)
- **Platform:** Mobile Application (iOS + Android)
- **Main Features:**
  - Verify snake identification
  - Remote consultation (chat/video)
  - Update first aid guidelines
  - Revenue management from consultations
  - Support rescuers on-field

### 4. 👨‍💼 **ADMIN** (Quản trị viên)
- **Platform:** Web Application (Desktop)
- **Main Features:**
  - User & role management
  - Snake species database management
  - Hospital/treatment facility management
  - Content management
  - Analytics & reporting
  - Community alerts
  - Financial management

---

## 📂 DOCUMENT STRUCTURE

```
02-UI-Design/
│
├── UI-Design-Overview.md                          (THIS FILE)
│
├── 🧑 PATIENT SCREENS/
│   ├── Patient-Emergency-Flow-Screens.md          ✅ COMPLETE (9 screens)
│   ├── Patient-Rescue-Request-Flow-Screens.md     ✅ COMPLETE (11 screens)
│   ├── Patient-Expert-Consultation-Flow-Screens.md ✅ COMPLETE (8 screens)
│   ├── Patient-Homepage-Enhanced.md               ✅ COMPLETE (1 screen)
│   └── Patient-Profile-Settings-Screens.md        ✅ COMPLETE (7 screens)
│
├── 🚑 RESCUER SCREENS/
│   ├── Rescuer-Emergency-Response-Flow-Screens.md ✅ COMPLETE (8 screens)
│   ├── Rescuer-Rescue-Request-Flow-Screens.md     ✅ COMPLETE (10 screens)
│   ├── Rescuer-Expert-Consultation-Flow-Screens.md ✅ COMPLETE (5 screens)
│   ├── Rescuer-Profile-Settings-Screens.md        ✅ COMPLETE (7 screens - Part1+Part2)
│   ├── Rescuer-Safety-Guidelines-Screens.md       ✅ COMPLETE (4 screens)
│   └── Rescuer-Revenue-Management-Screens.md      ✅ COMPLETE (6 screens)
│
├── 🧑‍🔬 EXPERT SCREENS/
│   ├── Expert-Dashboard-Screens.md                ✅ COMPLETE (4 screens)
│   ├── Expert-Consultation-Flow-Screens.md        ✅ COMPLETE (8 screens)
│   ├── Expert-Snake-Verification-Screens.md       ✅ COMPLETE (5 screens)
│   └── Expert-Revenue-Management-Screens.md       ✅ COMPLETE (5 screens)
│
└── 👨‍💼 ADMIN SCREENS/
    ├── Admin-Dashboard-Screens.md                 ✅ COMPLETE (2 screens)
    ├── Admin-User-Management-Screens.md           ✅ COMPLETE (5 screens)
    ├── Admin-Snake-Database-Screens.md            ✅ COMPLETE (4 screens)
    ├── Admin-Hospital-Management-Screens.md       ✅ COMPLETE (3 screens)
    ├── Admin-Analytics-Reporting-Screens.md       ✅ COMPLETE (4 screens)
    └── Admin-Financial-Management-Screens.md      ⏳ TODO
```

---

## 📊 PROGRESS TRACKING

**Overall Platform Progress:** 116/120 screens completed (97%) ✅

### 🧑 PATIENT Screens (Mobile)

| Document | Status | Screens | Priority | Notes |
|----------|--------|---------|----------|-------|
| **Patient-Emergency-Flow-Screens.md** | ✅ Complete | 9 screens | ⭐⭐⭐ | Emergency snakebite handling |
| **Patient-Rescue-Request-Flow-Screens.md** | ✅ Complete | 11 screens | ⭐⭐⭐ | Snake rescue request flow |
| **Patient-Homepage-Enhanced.md** | ✅ Complete | 1 screen | ⭐⭐⭐ | Main dashboard |
| **Patient-Expert-Consultation-Flow-Screens.md** | ✅ Complete | 8 screens | ⭐⭐ | Book & consult with experts |
| **Patient-Profile-Settings-Screens.md** | ✅ Complete | 7 screens | ⭐⭐⭐ | Profile, settings, history |

**Total Patient Screens:** 36 screens completed (29 completed + 7 completed)

---

### 🚑 RESCUER Screens (Mobile)

| Document | Status | Screens | Priority | Notes |
|----------|--------|---------|----------|-------|
| **Rescuer-Emergency-Response-Flow-Screens.md** | ✅ Complete | 8 screens | ⭐⭐⭐ | SOS emergency response |
| **Rescuer-Rescue-Request-Flow-Screens.md** | ✅ Complete | 10 screens | ⭐⭐⭐ | Accept, navigate, complete rescue |
| **Rescuer-Expert-Consultation-Flow-Screens.md** | ✅ Complete | 5 screens | ⭐⭐⭐ | Request expert help on-field |
| **Rescuer-Profile-Settings-Screens.md** | ✅ Complete | 7 screens | ⭐⭐⭐ | Profile, settings, history (Part1+Part2) |
| **Rescuer-Safety-Guidelines-Screens.md** | ✅ Complete | 4 screens | ⭐⭐ | Safety procedures, equipment |
| **Rescuer-Revenue-Management-Screens.md** | ✅ Complete | 6 screens | ⭐⭐⭐ | Earnings, transactions, withdrawal, bank accounts |

**Total Rescuer Screens:** 40/40 screens completed (100%) ✅

---

### 🧑‍🔬 EXPERT Screens (Mobile)

| Document | Status | Screens | Priority | Notes |
|----------|--------|---------|----------|-------|
| **Expert-Dashboard-Screens.md** | ✅ Complete | 4 screens | ⭐⭐⭐ | Main dashboard, notifications, schedule, earnings |
| **Expert-Consultation-Flow-Screens.md** | ✅ Complete | 8 screens | ⭐⭐⭐ | Dual mode: Patient scheduled + Rescuer urgent, video/chat |
| **Expert-Snake-Verification-Screens.md** | ✅ Complete | 5 screens | ⭐⭐ | AI verification queue, review, correction, notes, complete |
| **Expert-Revenue-Management-Screens.md** | ✅ Complete | 5 screens | ⭐⭐ | Revenue dashboard, transaction history, withdrawal, bank accounts |

**Total Expert Screens:** 22/22 screens (100% complete) ✅

---

### 👨‍💼 ADMIN Screens (Web)

| Document | Status | Screens | Priority | Notes |
|----------|--------|---------|----------|-------|
| **Admin-Dashboard-Screens.md** | ✅ Complete | 2 screens | ⭐⭐⭐ | Overview, real-time map |
| **Admin-User-Management-Screens.md** | ✅ Complete | 5 screens | ⭐⭐⭐ | CRUD users, roles, permissions, activity log |
| **Admin-Snake-Database-Screens.md** | ✅ Complete | 4 screens | ⭐⭐ | Manage snake species, images, AI testing |
| **Admin-Hospital-Management-Screens.md** | ✅ Complete | 3 screens | ⭐⭐ | Hospital map, facilities, antivenom inventory |
| **Admin-Analytics-Reporting-Screens.md** | ✅ Complete | 4 screens | ⭐⭐ | Dashboard, reports, performance, custom exports |
| **Admin-Financial-Management-Screens.md** | ⏳ TODO | ~4 screens | ⭐⭐ | Revenue, transactions |

**Total Admin Screens:** 18/22 screens completed (82%)

---

## 🎨 SHARED DESIGN SYSTEM

Tất cả screens trong platform share **common design system**:

### Color Palette (Base):
- **Primary Color (Patient):** Forest Green `#228B22`
- **Primary Color (Rescuer):** Orange `#FF8A00` (Action, rescue, energy)
- **Background:** White `#FFFFFF`
- **Text Primary:** Dark Gray `#333333`
- **Text Secondary:** Medium Gray `#666666`
- **Accent - Emergency:** Red `#DC3545`
- **Accent - Warning:** Amber `#FFC107`
- **Accent - Success:** Green `#28A745`
- **Accent - Info:** Blue `#007BFF`

> **Note:** Patient role uses Forest Green, Rescuer role uses Orange for consistent branding.

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
- **Icons:** Minimal, only essential ones

---

## 🔗 CROSS-ROLE INTERACTIONS

Mapping các interactions giữa roles:

### 1. Patient → Rescuer
- Patient requests rescue → Rescuer receives alert
- Patient tracks rescuer location → Rescuer shares GPS
- Patient pays rescuer → Rescuer receives payment
- Patient rates rescuer → Rescuer sees rating

### 2. Patient → Expert
- Patient requests consultation → Expert receives booking
- Patient chats/video calls → Expert responds
- Patient pays expert → Expert receives payment
- Patient rates expert → Expert sees rating

### 3. Rescuer → Expert
- Rescuer requests help → Expert provides consultation
- Rescuer shares photo → Expert verifies snake
- Platform pays expert → Expert receives consultation fee

### 4. Admin → All Roles
- Admin manages users → Affects all roles
- Admin updates content → All users see updates
- Admin views analytics → Data from all roles
- Admin manages fees → Affects payments

---

## 📋 PRIORITY & ROADMAP

### Phase 1: Core MVP (High Priority ⭐⭐⭐)
**Target:** Q1 2026

✅ **COMPLETED:**
- Patient Emergency Flow (9 screens)
- Patient Rescue Request Flow (11 screens)
- Patient Expert Consultation Flow (8 screens)
- Patient Homepage (1 screen)
- Patient Profile & Settings (7 screens)
- Rescuer Emergency Response Flow (8 screens)
- Rescuer Rescue Request Flow (10 screens)
- Rescuer Expert Consultation Flow (5 screens)
- Rescuer Profile & Settings (7 screens)
- Rescuer Safety Guidelines (4 screens)
- Rescuer Revenue Management (6 screens)
- Expert Dashboard (4 screens)
- Expert Consultation Flow (8 screens)
- Expert Snake Verification (5 screens)
- Expert Revenue Management (5 screens)

**Total Completed:** 116 screens

⏳ **NEXT UP:**
- Admin Financial Management (~4 screens) - FINAL MODULE 🎉

### Phase 2: Enhanced Features (Medium Priority ⭐⭐)
**Target:** Q2 2026
- Patient Expert Consultation
- Rescuer Safety Guidelines
- Expert Snake Verification
- Revenue Management (all roles)
- Admin Snake Database Management
- Admin Hospital Management

### Phase 3: Advanced Features (Lower Priority ⭐)
**Target:** Q3 2026
- Patient Profile & Settings
- Advanced Analytics
- Community features
- Notification preferences
- Multi-language support

---

## 📝 DESIGN GUIDELINES

### Mobile Apps (Patient, Rescuer, Expert):
1. **Mobile-first approach** - optimize for thumb reach
2. **Touch targets** - minimum 44x44px
3. **Loading states** - always show progress
4. **Offline support** - graceful degradation
5. **Push notifications** - real-time updates

### Web App (Admin):
1. **Desktop-optimized** - utilize screen space
2. **Data tables** - sortable, filterable
3. **Bulk actions** - efficient management
4. **Keyboard shortcuts** - power user features
5. **Export functionality** - CSV, PDF reports

---

## ✅ QUALITY CHECKLIST

Before marking any screen design as "Complete":

- [ ] All screens have Stitch prompts
- [ ] Color codes match design system
- [ ] Typography is consistent
- [ ] Touch targets are minimum 44x44px
- [ ] Loading/error states are designed
- [ ] Accessibility considerations (contrast, labels)
- [ ] Cross-references to Main Flow & Swimlane diagrams
- [ ] API endpoints documented
- [ ] Role clearly indicated in document title

---

## 🔗 RELATED DOCUMENTATION

- **Requirements:** `/01-Requirements/Main-Flow/Main-Flow.md`
- **Swimlane Diagrams:** `/01-Requirements/Swimlane-Diagram/`
- **Features:** `/01-Requirements/Major-Features/Major-Features-Summary.md`
- **Architecture:** `/02-Architecture-Design/Context-Diagram.md`

---

**Last Updated:** December 13, 2025  
**Maintained by:** SnakeAid Design Team  
**Status:** Living Document - Updated as designs progress

---

## 📈 CURRENT PROGRESS SUMMARY

**Overall Platform Progress:**
- **Patient Module:** 36/36 screens (100% complete) ✅
  - ✅ Emergency Flow: 9 screens
  - ✅ Rescue Request Flow: 11 screens
  - ✅ Expert Consultation Flow: 8 screens
  - ✅ Homepage: 1 screen
  - ✅ Profile & Settings: 7 screens

- **Rescuer Module:** 40/40 screens (100% complete) ✅
  - ✅ Emergency Response Flow: 8 screens
  - ✅ Rescue Request Flow: 10 screens
  - ✅ Expert Consultation Flow: 5 screens
  - ✅ Profile & Settings: 7 screens (Part1+Part2)
  - ✅ Safety Guidelines: 4 screens
  - ✅ Revenue Management: 6 screens

- **Expert Module:** 22/22 screens (100% complete) ✅
  - ✅ Dashboard: 4 screens
  - ✅ Consultation Flow: 8 screens
  - ✅ Snake Verification: 5 screens
  - ✅ Revenue Management: 5 screens

- **Admin Module:** 18/22 screens (82% complete) 🔄
  - ✅ Dashboard: 2 screens
  - ✅ User Management: 5 screens
  - ✅ Snake Database: 4 screens
  - ✅ Hospital Management: 3 screens
  - ✅ Analytics & Reporting: 4 screens

**Total Platform:** 116/120 screens completed (97% overall) 🎉
