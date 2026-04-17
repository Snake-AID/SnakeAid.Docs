# 26 - CreateBulkTimeSlots

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateBulkTimeSlots`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateBulkTimeSlots |  |  |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 183 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Expert creates weekly bulk time slots |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 7 |  |  |  |  |  | 0 | 5 | 2 | 7 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O |  |  |  |  |  |
|  |  |  | Logged in as Expert |  | O | O | O | O | O |  |  |  |  |  |  |  |
|  |  |  | Expert profile exists |  | O | O | O | O | O | O | O |  |  |  |  |  |
|  | weekStartDate |  |  |  | UTC | UTC | Unspecified | UTC | UTC | UTC | UTC |  |  |  |  |  |
|  | days |  |  |  | 1 block | 1 block | 1 block | 0 | 1 block | 1 block | 1 block |  |  |  |  |  |
|  | timeBlock |  |  |  | 08:00-09:00 | 08:00-09:00 | 08:00-09:00 | null | 09:00-08:00 | 08:00-09:00 | 08:00-09:00 |  |  |  |  |  |
|  | role |  |  |  | Expert | Expert | Expert | Expert | Expert | public | User |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  | O | O | O |  |  |  |  |  |  |  |
|  |  |  | slot count = 2 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | duplicate skipped |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "WeekStartDate must be UTC." |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | "At least one day block is required." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "EndTime must be later than StartTime." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Time slots generated successfully" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | B | A | A |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
