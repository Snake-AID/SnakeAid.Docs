# 28 - CreateConsultationBooking

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateConsultationBooking`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateConsultationBooking |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 89 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Member creates scheduled booking from an expert time slot with booking, consultation, and slot-state validation |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 9 |  |  |  |  |  | 0 | 7 | 2 | 9 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 | UTCID09 |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O | O |  |  |  |
|  |  |  | Expert time slot data prepared |  | O | O | O | O | O | O | O | O | O |  |  |  |
|  | auth context |  |  |  | User | public | Expert | User | User | User | User | User | User |  |  |  |
|  | timeSlotId |  |  |  | valid future slot | valid future slot | valid future slot | missing slot | started slot | reserved slot | valid future slot | valid future slot | valid future slot |  |  |  |
|  | problemDescription |  |  |  | "Need scheduled advice" | "Need scheduled advice" | "Need scheduled advice" | "Need scheduled advice" | "Need scheduled advice" | "Need scheduled advice" | > 2000 chars | "Need scheduled advice" | "Need scheduled advice" |  |  |  |
|  | expert profile |  |  |  | exists | exists | exists | exists | exists | exists | exists | missing | exists |  |  |  |
|  | database concurrency |  |  |  | normal | normal | normal | normal | normal | normal | normal | normal | DbUpdateConcurrencyException |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  | O |  |  |  | O |  |  |  |  |
|  |  |  | HTTP 409 |  |  |  |  |  | O | O |  |  | O |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | booking.Status = PendingPayment |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultation.Status = Scheduled |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultation.Type = Scheduled |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | slot.Status = Reserved |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | roomId starts with "consultation-" |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Requested time slot was not found." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "This time slot has already started and can no longer be booked." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "This time slot is no longer available." |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | "The field ProblemDescription must be a string or array type with a maximum length of '2000'." |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | "Expert profile not found for the selected slot." |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  |  |  | "This time slot was booked by another request. Please refresh and try again." |  |  |  |  |  |  |  |  |  | O |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O |  |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | A | B | A | B |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

