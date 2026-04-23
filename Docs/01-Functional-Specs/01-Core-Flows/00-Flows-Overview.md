# MAIN FLOWS - SNAKEAID SYSTEM

## Project Information
- **Project Name:** AI-Powered Platform for Snakebite First Aid and Rescue Support (SnakeAid)
- **Purpose:** Standardize the main business flows based on the investor SRS

---

## Overview of Main Flows

### MEMBER
- **[1] Emergency Snakebite Incident**
  - Receive immediate first-aid guidance
  - Capture snake or bite images
  - Use AI identification and severity assessment
  - Trigger SOS
  - Share GPS and track rescue progress
  - Find the nearest treatment facility when needed

- **[2] Snake Sighting / Catching Service Request**
  - Submit image, location, and description
  - Receive automatic price estimation
  - Pay in two phases
  - Track rescuers and mission status

- **[3] Expert Consultation**
  - Select an Expert
  - Request instant consultation or schedule a booking
  - Join the waiting room
  - Pay through escrow flow
  - Submit a rating after the session

- **[4] Wallet, Payments, and Notifications**
  - Top up wallet
  - Withdraw funds
  - View transaction history
  - Receive real-time notifications

- **[5] Education and Community Alerts**
  - Read snake knowledge content
  - Search by topic or species
  - Receive area-based alerts

### OPERATOR
- **[6] Request Intake and Dispatching**
  - View SOS and snake catching queues
  - Verify and triage requests
  - Assign Rescuers
  - Monitor mission progress

### RESCUER
- **[7] Mission Execution**
  - Receive SignalR dispatch alerts
  - Acknowledge assignment reception
  - Travel to the incident location
  - Update mission status
  - Review mission history and performance

### EXPERT
- **[8] Consultation and Professional Validation**
  - Complete onboarding and verification
  - Provide chat or video consultation
  - Receive payment after escrow release
  - Track revenue

### ADMIN
- **[9] System Administration**
  - Manage pricing and commission
  - Manage hospitals and antivenom data
  - Manage snake database
  - Manage content
  - Manage KYC, finance, audit, and AI governance

---

## Interaction Matrix Across Modules

| Scenario | Member | Operator | Rescuer | Expert | Admin | AI System |
|----------|--------|----------|---------|--------|-------|-----------|
| Emergency snakebite incident | Send SOS, images, and symptoms | Verify and triage | Execute if assigned | Support if needed | Monitor | Identification + severity assessment |
| Non-emergency snake catching | Create request and pay by phase | Queue and assign | Accept and execute mission | Support if needed | Monitor pricing and performance | Price estimation + preliminary identification |
| Expert consultation | Book, join waiting room, pay escrow | - | - | Consult and close session | Monitor payout and compliance | Support initial triage |
| Facility lookup | Search hospitals | - | Use as reference if needed | - | Manage data | Calculate distance and ETA |
| Wallet and finance | Top up, pay, view history | - | Follow role policy | Receive payout | Manage commission | Reconcile status |
| Community alerts | Receive alerts | Trigger operationally when needed | Receive internal alerts if relevant | - | Configure and publish | Analyze trends |

---

## Core Business Principles

- All emergency cases and snake catching requests enter the `Operator` queue.
- Hospital and antivenom intelligence are in scope.
- `PayOS` is the primary payment gateway.
- Consultation uses booking lifecycle and escrow control.
- Snake catching service uses two-phase payment control.
- Wallet, callback reconciliation, audit logging, and AI governance are part of the target state.
