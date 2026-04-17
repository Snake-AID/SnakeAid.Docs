# 12 - GetUserIncidents

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetUserIncidents`
- Used range: `A2:O37`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 4 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 6 |  | 0 |  |  | 0 |  |  |  |  |  | 1 | 3 | 2 | 6 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O |  | O | O |  |  |  |  |
| 13 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  | status |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 15 |  |  |  | Valid status |  | O |  | O | O | O |  |  |  |  |  |
| 16 |  |  |  | Invalid status |  |  | O |  |  |  |  |  |  |  |  |
| 17 |  |  |  | null |  |  |  |  |  |  | O |  |  |  |  |
| 18 |  | page |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 19 |  |  |  | Valid (>= 1) |  | O | O |  | O | O | O |  |  |  |  |
| 20 |  |  |  | Invalid (0 / negative) |  |  |  | O |  |  |  |  |  |  |  |
| 21 |  | pageSize |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 22 |  |  |  | Valid(eg. 5, 10) |  | O | O | O | O |  | O |  |  |  |  |
| 23 |  |  |  | Invalid (0/negative) |  |  |  |  |  | O |  |  |  |  |  |
| 24 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  | HTTP 200 |  | O |  |  |  |  | O |  |  |  |  |
| 26 |  |  |  | HTTP 401 |  |  |  |  | O |  |  |  |  |  |  |
| 27 |  |  |  | HTTP 422 |  |  | O | O |  | O |  |  |  |  |  |
| 28 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  | "User incidents retrieved" |  | O |  |  |  |  | O |  |  |  |  |
| 32 |  |  |  | "Validate failed" |  |  | O | O |  | O |  |  |  |  |  |
| 33 |  |  |  | "Unauthorized" |  |  |  |  | O |  |  |  |  |  |  |
| 34 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | B | A | B | A |  |  |  |  |
| 35 |  | Passed/Failed |  |  |  | P | P | P | P | P | P |  |  |  |  |
| 36 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |
| 37 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
