# 39 - EndConsultation

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `EndConsultation`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | EndConsultation |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 58 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Authenticated consultation participant ends a consultation with lifecycle completion and scheduled-booking cleanup |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 5 |  |  |  |  |  | 0 | 2 | 3 | 5 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 |  |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O |  |  |  |  |  |  |  |
|  |  |  | Consultation data prepared |  | O | O | O | O | O |  |  |  |  |  |  |  |
|  | auth context |  |  |  | participant | participant | public | other authenticated user | participant |  |  |  |  |  |  |  |
|  | consultation state |  |  |  | Ongoing scheduled with booking + slot | Completed | Ongoing | Ongoing | missing consultation |  |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | consultation.Status = Completed |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultation.EndTime != null |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | booking.Status = Completed |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | slot.Status = Booked |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "You are not a participant of this consultation." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "Consultation not found." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Consultation ended successfully." |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | B | A | A | A |  |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

