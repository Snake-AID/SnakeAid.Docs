# 5. Main Flow: System Administration and Operations

## 5.1 Objective

- Cover the Admin-side scope defined in the investor SRS.
- Support pricing, KYC, facilities, snake database, content, finance, and AI governance.
- Keep daily dispatch responsibilities separate from Admin governance responsibilities.

---

## 5.2 Service Pricing and Fee Configuration

**Flow 5.1 - Service Pricing**

1. The Admin opens the pricing configuration screen.
2. The Admin sets:
   - Expert consultation rates.
   - Service fees for snake catching.
   - Platform commission percentages.
3. The system saves the configuration for payment flows.

---

## 5.3 KYC and Professional Verification

**Flow 5.2 - Identity and Professional Verification**

1. The Admin views the list of Expert verification requests.
2. The Admin reviews certificates and identity documents.
3. The Admin approves or declines each request.
4. The Admin assigns badges and verification status.
5. The Admin suspends or reactivates accounts when necessary.

---

## 5.4 Hospital and Antivenom Management

**Flow 5.3 - Facility Administration**

1. The Admin creates, updates, or deletes hospital records.
2. The Admin updates antivenom stock and availability.
3. The Admin tags facilities with specialties such as `24/7 Treatment`.
4. The Admin imports hospital data from external sources when available.

---

## 5.5 Snake Database Administration

**Flow 5.4 - Snake Data Management**

1. The Admin manages the master snake species list.
2. The Admin updates toxicity classifications and symptoms metadata.
3. The Admin uploads training data for AI review workflows.
4. The Admin links species to related antivenom types.

---

## 5.6 Content and Notification Management

**Flow 5.5 - Content and Campaign Management**

1. The Admin updates first-aid guidelines and snake knowledge content.
2. The Admin manages blogs for Members.
3. The Admin manages lessons for Rescuers.
4. The Admin creates notification campaigns by role or area.

---

## 5.7 Finance, Wallet, and Reconciliation

**Flow 5.6 - Financial Operations**

1. The Admin tracks total platform revenue.
2. The Admin manages commission and payout schedules.
3. The Admin handles disputes and refund requests.
4. The Admin monitors wallet top-ups, withdrawals, and balance movements.
5. The Admin monitors payment callback processing, reconciliation, idempotency, and final payment synchronization.

---

## 5.8 Audit, Compliance, and AI Governance

**Flow 5.7 - Governance Control**

1. The system logs Expert overrides for AI identification results.
2. The system tracks decision history for severity recommendations.
3. The system applies risk rules when AI confidence is low.
4. The system stores audit trails for compliance and post-incident review.

---

## 5.9 Business Rules

- The `Admin` manages policy, data, finance, and governance, while the `Operator` handles day-to-day dispatch queues.
- Hospital records and antivenom data are active scope items.
- Payment, wallet, callback, reconciliation, and payout scheduling are all within Admin scope.
- AI outputs must support auditability and low-confidence fallback rules.
