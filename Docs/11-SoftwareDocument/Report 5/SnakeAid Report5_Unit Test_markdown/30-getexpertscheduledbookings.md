# 30 - GetExpertScheduledBookings

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertScheduledBookings`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetExpertScheduledBookings |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 31 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Expert retrieves only their confirmed or completed scheduled bookings in slot-start ascending order |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 4 |  |  |  |  |  | 0 | 2 | 2 | 4 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |  |  |  |
|  |  |  | Booking data prepared |  | O | O | O | O |  |  |  |  |  |  |  |  |
|  | auth context |  |  |  | Expert | Expert | public | User |  |  |  |  |  |  |  |  |
|  | current expert rows |  |  |  | Confirmed + Completed + PendingPayment + foreign expert | no Confirmed/Completed rows | has rows | has rows |  |  |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | expertId = current expert only |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | statuses in {Confirmed, Completed} |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | excludes PendingPayment rows |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | sorted by SlotStartTime asc |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | roomId not blank |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | size = 0 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | B | A | A |  |  |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

