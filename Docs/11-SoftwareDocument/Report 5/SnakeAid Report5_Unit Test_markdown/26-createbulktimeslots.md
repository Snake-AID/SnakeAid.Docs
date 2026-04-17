# 26 - CreateBulkTimeSlots

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateBulkTimeSlots`
- Used range: `A2:Q45`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateBulkTimeSlots |  |  |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 183 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| 5 | Test requirement |  | Expert creates weekly bulk time slots |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 7 | 0 |  | 0 |  |  | 7 |  |  |  |  |  | 0 | 5 | 2 | 7 |  |  |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Logged in as Expert |  | O | O | O | O | O |  |  |  |  |  |  |  |
| 13 |  |  |  | Expert profile exists |  | O | O | O | O | O | O | O |  |  |  |  |  |
| 14 |  | weekStartDate |  |  |  | UTC | UTC | Unspecified | UTC | UTC | UTC | UTC |  |  |  |  |  |
| 15 |  | days |  |  |  | 1 block | 1 block | 1 block | 0 | 1 block | 1 block | 1 block |  |  |  |  |  |
| 16 |  | timeBlock |  |  |  | 08:00-09:00 | 08:00-09:00 | 08:00-09:00 | null | 09:00-08:00 | 08:00-09:00 | 08:00-09:00 |  |  |  |  |  |
| 17 |  | role |  |  |  | Expert | Expert | Expert | Expert | Expert | public | User |  |  |  |  |  |
| 18 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 19 |  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |  |  |
| 22 |  |  |  | HTTP 422 |  |  |  | O | O | O |  |  |  |  |  |  |  |
| 23 |  |  |  | slot count = 2 |  | O | O |  |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | duplicate skipped |  |  | O |  |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  | "WeekStartDate must be UTC." |  |  |  | O |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | "At least one day block is required." |  |  |  |  | O |  |  |  |  |  |  |  |  |
| 27 |  |  |  | "EndTime must be later than StartTime." |  |  |  |  |  | O |  |  |  |  |  |  |  |
| 28 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "Time slots generated successfully" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| 30 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | B | A | A |  |  |  |  |  |
| 31 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 44 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 45 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
