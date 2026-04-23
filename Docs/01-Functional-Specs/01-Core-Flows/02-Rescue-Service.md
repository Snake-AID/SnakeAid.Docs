# 2. Main Flow: Snake Catching Service

## 2.1 Objective

- Receive non-emergency snake catching requests.
- Allow the Operator to verify, triage, and assign a Rescuer.
- Apply two-phase payment control for the catching service.

---

## 2.2 Request Creation Stage (Member)

**Flow 2.1 - Create Catching Request**

1. The Member selects the **Snake Catching Service** function.
2. The Member submits:
   - A snake image.
   - The incident location.
   - A situation description.
3. The system runs preliminary AI identification.
4. The system generates an automatic price estimation.
5. The system displays the round 1 travel fee payment.
6. The Member pays round 1 through PayOS or a supported wallet mechanism.
7. After successful payment, the request is created and placed in the Operator queue.

---

## 2.3 Dispatch Stage (Operator)

**Flow 2.2 - Verify and Assign**

1. The Operator views the real-time snake catching request queue.
2. The Operator verifies the request details and triages the priority.
3. The Operator selects a suitable Rescuer.
4. The system sends a dispatch alert to the Rescuer.
5. The Rescuer acknowledges assignment reception.
6. If the Rescuer declines or times out, the Operator reassigns the request.

---

## 2.4 Mission Execution Stage (Rescuer)

**Flow 2.3 - Mission Execution**

1. The Rescuer reviews the request details, images, location, and safety alerts.
2. The Rescuer travels to the site and updates mission status:
   - Assigned
   - En Route
   - Arrived
   - Processing
   - Completed
   - Cancelled
3. During execution, the Member tracks real-time location and dynamic ETA.
4. If needed, the Rescuer can request remote support from an Expert.
5. After completion, the Rescuer uploads handling results and evidence images.

---

## 2.5 Round 2 Payment Stage

**Flow 2.4 - Final Service Settlement**

1. When the mission reaches the final payment point according to policy, the system creates the round 2 final service payment.
2. The round 1 deposit is applied based on deposit usage rules.
3. The Member pays round 2.
4. The system updates the invoice, payment status, and transaction history.
5. If the mission is cancelled, the system applies the cancellation window and penalty rules to determine refund or deduction.

---

## 2.6 Feedback Stage

**Flow 2.5 - Service Feedback**

1. After service completion, the Member rates the Rescuer.
2. The Member may leave a review.
3. The system updates history and performance metrics.

---

## 2.7 Business Rules

- Snake catching service uses two payment phases: travel fee and final service fee.
- A request enters the dispatch queue only after valid round 1 payment.
- The `Operator` performs manual assignment according to the SRS.
- Payment states must be synchronized through callback and reconciliation flows.
