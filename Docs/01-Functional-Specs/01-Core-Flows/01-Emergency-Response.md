# 1. Main Flow: Emergency Snakebite Response

## 1.1 Objective

- Provide immediate first-aid support to the Member.
- Use AI to identify snake species and assess severity.
- Create an emergency case for the Operator and support routing to appropriate treatment facilities.

---

## 1.2 Initial Response Stage (Member)

**Flow 1.1 - First Aid and Data Collection**

1. The Member opens the **SnakeAid** application.
2. The Member selects the emergency snakebite function.
3. The system displays:
   - Step-by-step first-aid instructions.
   - Compression bandaging tutorial media.
   - Warnings about harmful actions to avoid.
4. The Member captures a snake photo if it is safe to do so.
5. The Member captures bite images and enters symptoms.
6. The AI engine performs:
   - Snake species identification.
   - Probability scoring and top matches.
   - Severity classification as mild, moderate, severe, or critical.
7. The system generates an immediate recommendation based on images and symptoms.

---

## 1.3 SOS Activation Stage

**Flow 1.2 - Emergency Case Creation**

1. The Member presses and holds the **SOS** button.
2. The system captures the current GPS coordinates.
3. The system creates an emergency case.
4. The emergency case is pushed to the **Operator** queue.
5. The system enables real-time location sharing for the Member.
6. The Member sees the case status as pending verification and dispatch.

---

## 1.4 Operator Verification and Dispatch Stage

**Flow 1.3 - Rescue Dispatching**

1. The Operator opens the queue and sees the new emergency case.
2. The Operator verifies the submitted information with the Member.
3. The Operator triages the urgency level.
4. The Operator manually assigns a suitable Rescuer.
5. The system sends a SignalR dispatch alert to the assigned Rescuer.
6. If the Rescuer acknowledges the assignment:
   - The case moves to active dispatch.
   - The Member can track the Rescuer in real time.
7. If no suitable Rescuer is available, the case remains in the queue for continued handling.

---

## 1.5 Treatment Support Stage

**Flow 1.4 - Medical Facility Intelligence**

1. The Member or Operator opens the list of nearby treatment facilities.
2. The system filters facilities by:
   - Distance.
   - Availability.
   - Relevant treatment or antivenom information.
3. The system calculates ETA to each facility.
4. The Member can open integrated navigation.

---

## 1.6 Business Rules

- The platform does not replace professional medical treatment.
- AI identification and severity assessment are decision-support tools only.
- Every emergency case must record status history, decisions, and related AI outputs.
- Low-confidence AI results must trigger a safe fallback protocol.
