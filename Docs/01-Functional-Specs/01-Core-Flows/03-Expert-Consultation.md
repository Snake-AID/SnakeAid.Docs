# 3. Main Flow: Expert Consultation

## 3.1 Objective

- Connect Members with Experts through chat or video consultation.
- Support booking lifecycle, waiting room, escrow hold, release, and refund.
- Support Expert revenue tracking and payout control.

---

## 3.2 Onboarding and Availability Stage (Expert)

**Flow 3.1 - Professional Onboarding**

1. The Expert registers an account.
2. The Expert uploads certifications and identity documents.
3. The Expert completes the professional profile.
4. The Expert submits a verification request.
5. The Admin reviews and approves or declines the request.
6. Once approved, the Expert can toggle real-time availability.

---

## 3.3 Booking Creation Stage (Member)

**Flow 3.2 - Create Consultation Booking**

1. The Member opens the Expert list.
2. The Member selects an Expert and chooses:
   - Instant consultation.
   - Scheduled consultation.
3. The Member uploads media and a consultation description.
4. The system creates the booking state.
5. The system locks the Expert time slot when the booking is confirmed.
6. The Member pays through PayOS.
7. The system holds the payment in escrow.
8. The system creates a waiting room before the consultation starts.

---

## 3.4 Live Consultation Stage

**Flow 3.3 - Consultation Session**

1. At the scheduled time or after Expert acceptance, both parties enter the waiting room.
2. The system opens:
   - A real-time chat session.
   - A peer-to-peer video call for triage.
3. The Member can share additional images or media.
4. The Expert evaluates the case and provides recommendations.
5. At the end of the session, the Expert submits a consultation summary.
6. The system stores session status and related history.

---

## 3.5 Escrow Release and Revenue Tracking Stage

**Flow 3.4 - Settlement**

1. After valid session completion, the system releases escrow according to policy.
2. The system deducts platform fees based on configuration.
3. The remaining amount is recorded for Expert payout.
4. The Expert views monthly revenue reports and transaction history.
5. If the booking is cancelled, the system applies refund rules based on cancellation policy.

---

## 3.6 Business Rules

- Consultation bookings must use booking lifecycle and escrow control.
- Expert time slots must be automatically locked or released based on booking state.
- Expert revenue must be calculated after platform fee deduction.
- The system must keep the consultation summary and relevant audit trail.
