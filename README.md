# SnakeAid

**AI-Powered Platform for Snakebite First Aid and Rescue Support**

SnakeAid is a web and mobile platform designed to improve how snakebite incidents and snake-catching requests are handled. It combines first-aid guidance, AI-assisted snake identification, severity assessment, operator-led rescue dispatching, expert consultation, treatment facility intelligence, and financial workflows into one coordinated system.

## Project Information

- **Project Name:** AI-Powered Platform for Snakebite First Aid and Rescue Support
- **Software Type:** Web Application, Mobile App
- **Product Name:** SnakeAid

## Problem Statement

Snakebite incidents are often mishandled because first aid is delayed, snake species are misidentified, nearby treatment facilities are not easy to find, and rescue coordination is slow. Victims may panic and perform harmful actions, rescuers may operate with incomplete field information, experts may not be available in time, and administrators may lack real-time visibility into incident hotspots and response efficiency.

SnakeAid addresses this by providing an AI-powered platform and coordination center that supports:

- Immediate first-aid guidance
- Snake image recognition
- Severity assessment
- SOS emergency handling
- Real-time rescue tracking
- Expert consultation
- Hospital and antivenom intelligence
- Payment, wallet, and reconciliation flows
- Operational monitoring and governance

## Product Vision

SnakeAid aims to become a reliable snakebite first-aid platform and rescue coordination center for high-risk regions. The product connects Members, Rescuers, Experts, Operators, and Admins in a unified real-time system.

The core vision is to deliver:

- **Faster response:** Immediate first-aid guidance and faster case intake
- **Better decision support:** AI-based snake identification and severity categorization
- **Coordinated rescue operations:** Operator-managed dispatching and live mission tracking
- **Accessible expertise:** Remote expert consultation through chat and video
- **Data-driven administration:** Facility management, financial reporting, audit trails, and AI governance

## User Roles

- **Member:** Requests help, receives guidance, tracks rescue, books consultations, pays for services, and manages wallet activity
- **Rescuer:** Receives operator-assigned missions, updates field status, navigates to incidents, and records outcomes
- **Expert:** Completes professional onboarding, provides consultation, and tracks revenue
- **Operator:** Verifies incoming reports, triages cases, and assigns Rescuers
- **Admin:** Manages pricing, KYC, facilities, snake data, content, finance, notifications, and AI governance

## Core Functional Scope

### Member

- Authentication and profile management
- AI snake image recognition
- Snake species knowledge access
- Severity assessment from symptoms and images
- SOS emergency trigger with GPS capture
- Medical facility lookup with ETA and navigation
- Real-time rescue tracking
- Snake catching service requests
- Expert consultation via chat and video
- First-aid and prevention knowledge hub
- PayOS-based payment flows
- Ratings and reviews
- Wallet management
- Notification center and real-time updates

### Rescuer

- Operator-assigned mission intake
- SignalR dispatch alerts
- Mission status updates
- Mission history and performance review
- Safety guidance and learning content

### Expert

- Expert registration and professional onboarding
- Verification workflow
- Consultation delivery
- Revenue tracking
- Wallet withdrawals where applicable

### Operator

- Real-time emergency and catching queues
- Verification and triage of incoming reports
- Manual assignment of Rescuers
- Monitoring of rescue progress and live location updates

### Admin

- Service pricing and fee configuration
- KYC and professional verification
- Hospital and antivenom management
- Snake database administration
- Content and lesson management
- Notification orchestration
- Platform revenue and commission management
- Payment dispute and refund handling
- Wallet and reconciliation oversight
- Audit, compliance, and AI governance

## Payment Model

SnakeAid includes multiple financial flows defined by the standardized requirement set:

- **PayOS** as the primary payment gateway
- **Consultation booking escrow** with release or refund by policy
- **Snake catching two-phase payment control**
  - Round 1 travel fee
  - Round 2 final service payment
- **Wallet management** for supported user roles
- **Callback and reconciliation** to synchronize final payment state

## Main Business Flows

### 1. Emergency Snakebite Response

- Member receives immediate first-aid guidance
- AI identifies snake species and estimates severity
- SOS creates an emergency case
- Operator verifies and dispatches a Rescuer
- Member tracks rescue progress in real time
- Member or Operator can reference nearby treatment facilities

### 2. Snake Catching Service

- Member submits a non-emergency snake catching request
- System generates automatic price estimation
- Member completes round 1 payment
- Operator verifies and assigns a Rescuer
- Rescuer executes the mission
- Member completes round 2 payment and leaves feedback

### 3. Expert Consultation

- Expert completes onboarding and verification
- Member creates an instant or scheduled booking
- Booking enters waiting room and escrow flow
- Consultation is delivered through chat or video
- Escrow is released based on completion policy

### 4. Education and Notifications

- Members and Rescuers access safety guidance and learning content
- Notification engine delivers service and area-based alerts
- Notification center stores in-app updates and mission events

### 5. Administration and Governance

- Admin manages pricing, facilities, content, finance, compliance, and AI governance
- Operator handles daily dispatch queues and rescue triage

## Limitations and Exclusions

SnakeAid follows the investor SRS constraints. Key points include:

- The system does not replace hospital care or licensed medical treatment
- AI outputs are decision-support only and may not be fully accurate
- Availability of Rescuers and Experts depends on actual operational coverage
- Stable internet is required for core workflows
- Integration with public ambulance or government emergency infrastructure is out of scope
- Offline-first behavior is excluded
- SMS, phone, and VoIP dispatch channels are excluded
- PayOS is the only supported payment gateway in scope
- Multi-tenant SaaS behavior is excluded
- Hardware and IoT integrations are excluded

## Documentation Source of Truth

Supporting functional documents include:

- [Feature-Matrix.md](Docs/01-Functional-Specs/00-Concepts/Feature-Matrix.md)
- [00-Flows-Overview.md](Docs/01-Functional-Specs/01-Core-Flows/00-Flows-Overview.md)
- [01-Emergency-Response.md](Docs/01-Functional-Specs/01-Core-Flows/01-Emergency-Response.md)
- [02-Rescue-Service.md](Docs/01-Functional-Specs/01-Core-Flows/02-Rescue-Service.md)
- [03-Expert-Consultation.md](Docs/01-Functional-Specs/01-Core-Flows/03-Expert-Consultation.md)
- [04-Community-Safety.md](Docs/01-Functional-Specs/01-Core-Flows/04-Community-Safety.md)
- [05-Admin-Operations.md](Docs/01-Functional-Specs/01-Core-Flows/05-Admin-Operations.md)
