# MAJOR FEATURES SUMMARY TABLE - SNAKEAID PLATFORM

## Project Information

- **Project Name:** AI-Powered Platform for Snakebite First Aid and Rescue Support (SnakeAid)
- **Software Type:** Web Application, Mobile App
- **Purpose:** AI-powered platform and coordination center for first aid, snake identification, rescue dispatching, expert consultation, treatment facility intelligence, payments, and incident monitoring

---

# Major Features

## Member Application

### FE-01 Authentication & Authorization

- Sign Up for Member role.
- Sign In with email and password.
- Sign Out and session cleansing.
- Verify email identity via OTP.
- Reset password via secure token.

### FE-02 User Profile Management

- Get personal profile information.
- Update name, bio, and emergency contacts.
- Upload and crop profile avatar.
- View activity logs such as login history.

### FE-04 AI Image Recognition Engine

- Capture snake photo from mobile camera.
- Pre-process image for crop, blur detection, and brightness handling.
- Return top matches with probability scores.

### FE-05 Snake Species Knowledge Base

- View scientific name, local name, and snake description.
- View toxicity level and danger assessment.
- Display habitat and distribution references.

### FE-06 Severity Assessment

- Input symptoms such as pain, nausea, and breathing difficulty.
- Categorize severity into mild, moderate, severe, or critical.
- Generate immediate recommendation for next action.

### FE-08 Emergency SOS Trigger

- Hold-to-activate SOS.
- Capture instant GPS coordinates.
- Create emergency case and push it to Operator queue.

### FE-09 Medical Facility Intelligence

- Identify nearest hospitals or treatment facilities.
- Filter by distance and availability.
- Calculate ETA.
- Open integrated navigation.

### FE-10 Real-time Rescue Tracking

- Connect to real-time location stream.
- Track rescuer movement on map.
- Display route polyline and dynamic ETA.
- Receive arrival and task completion events.

### FE-11 Snake Catching Service

- Create non-emergency catching request with photo, location, and description.
- Receive automatic price estimation.
- Track mission sub-statuses.

### FE-12 Expert Consultation

- Create real-time chat session with Snake Expert.
- Start peer-to-peer video call for triage.
- Share high-resolution media for diagnosis.
- Receive consultation summary after session.

### FE-14 Safety Guidelines & Knowledge Hub

- Access step-by-step first aid instructions.
- View compression bandaging tutorials.
- Read prevention articles and FAQ.
- Search knowledge by topic or snake group.

### FE-16 Payment Integration & Processing

- Checkout via PayOS gateway.
- Process payment by service type.
- Receive refunds for valid cancellations.

### FE-24 Feedback & Rating System

- Rate rescuer performance.
- Rate expert consultation quality.
- Leave review or comment on services.

### FE-26 Wallet Management

- Top up wallet balance via PayOS.
- Withdraw to linked bank account.
- View wallet balance and available amount.
- View wallet transaction history.

### FE-27 PayOS Callback & Payment Reconciliation

- Handle success, cancel, or failed callback.
- Validate callback integrity.
- Sync final payment status to booking, wallet, and invoice.

### FE-28 Consultation Booking Lifecycle & Escrow Control

- Manage booking states.
- Create waiting room.
- Lock and release expert time slot.
- Hold escrow and release or refund by policy.

### FE-29 Catching Service Two-Phase Payment Control

- Collect round 1 travel fee.
- Collect round 2 final service payment.
- Apply deposit usage rules by status.
- Apply cancellation window and penalty rules.

### FE-30 Notification Center & Real-time Reliability

- In-app notification inbox.
- Real-time mission events.
- Reconnect on app resume or network change.
- Notification preference settings.

## Rescuer Application

### FE-13 Rescuer Mission Management

- Receive SignalR dispatch alerts from Operator.
- Acknowledge assignment reception.
- Update mission status.
- View mission history and performance.

### FE-14 Safety Guidelines & Knowledge Hub

- Access rescue guidance and learning content.
- Review first-aid and handling lessons for field work.

### FE-24 Feedback & Rating System

- Receive ratings and reviews from Members.

### FE-26 Wallet Management

- Access role-based wallet functions when permitted by policy.
- View rescue-related wallet transactions if applicable.

## Expert Application

### FE-01 Authentication & Authorization

- Sign Up for Expert role.
- Sign In, Sign Out, verify email, and reset password.

### FE-03 Professional Onboarding

- Upload certification and identity documents.
- Complete expert registration profile.
- Submit verification request.
- Track verification status.
- Toggle real-time availability.

### FE-12 Expert Consultation

- Accept immediate or scheduled consultation request.
- Conduct text chat and video consultation.
- Share professional advice and summary.

### FE-17 Revenue Tracking

- Get monthly revenue report.
- Calculate final payout after platform fees.
- View transaction history with filtering.

### FE-24 Feedback & Rating System

- Receive consultation ratings and reviews.

### FE-26 Wallet Management

- View wallet balance, available amount, and withdrawal history.
- Withdraw to linked bank account when eligible.

## Operator Console

### FE-18 Operational Dispatching

- View real-time emergency and snake catching request queues.
- Verify and triage incoming reports.
- Manually assign rescuers to specific cases.

### FE-10 Real-time Rescue Tracking

- Monitor live rescuer movement, ETA, arrival, and task completion.

### FE-23 Smart Notification Orchestration

- Trigger operational notifications by role and area.

## Admin Application

### FE-15 Service Pricing & Fee Configuration

- Set expert consultation rates.
- Configure service fee.
- Define platform commission percentages.

### FE-19 Identity & Professional Verification

- Review professional documents.
- Approve or decline expert registration requests.
- Manage professional badges and verification status.
- Suspend or reactivate accounts for compliance.

### FE-20 Hospital & Antivenom Management

- Create, update, and delete hospital records.
- Update antivenom stock and availability.
- Tag facilities with specialty such as 24/7 treatment.
- Import hospital data from external sources.

### FE-21 Snake Database Administration

- Manage master snake species list.
- Upload training data for AI model after admin review.
- Manage toxicity classifications and symptoms metadata.
- Link species to specific antivenom types.

### FE-22 System Content Management

- Update first aid guidelines and snake knowledge content.
- Manage blogs for Member.
- Manage lessons for Rescuer.

### FE-23 Smart Notification Orchestration

- Push service notifications.
- Send in-app alerts for system updates.
- Apply role-based and area-based campaign rules.

### FE-25 Platform Revenue & Commission Management

- Track total revenue and distribute income.
- Manage commission and payout schedules.
- Handle disputes and refund requests.

### FE-31 Audit, Compliance & AI Governance

- Log expert override for AI identification results.
- Track decision history for severity recommendations.
- Apply fallback risk rules for low-confidence AI outputs.

---

# Notes

- Hospital lookup and antivenom management remain in scope as informational and administrative capabilities.
- PayOS is the target payment gateway.
- Wallet, escrow, reconciliation, and two-phase payment control are part of the target product scope.
