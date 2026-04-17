# 29 - GetMyScheduledBookings

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetMyScheduledBookings`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetMyScheduledBookings |  |  |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 28 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Member retrieves their own scheduled bookings list including pending-payment rows and descending booking order |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 4 |  |  |  |  |  | 0 | 2 | 2 | 4 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |  |  |  |
|  |  |  | Booking data prepared |  | O | O | O | O |  |  |  |  |  |  |  |  |
|  | auth context |  |  |  | User | User | public | Expert |  |  |  |  |  |  |  |  |
|  | current user bookings |  |  |  | has Confirmed + PendingPayment rows | no rows | has rows | has rows |  |  |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | userId = current user only |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | includes PendingPayment rows |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | sorted by BookedAt desc |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | size = 0 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | response items include ExpertName and SlotStartTime |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | B | A | A |  |  |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

