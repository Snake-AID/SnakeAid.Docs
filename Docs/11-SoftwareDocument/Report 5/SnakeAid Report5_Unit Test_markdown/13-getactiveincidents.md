# 13 - GetActiveIncidents

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetActiveIncidents`
- Used range: `A2:O40`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 1 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 9 |  | 0 |  |  | 0 |  |  |  |  |  | 2 | 7 | 0 | 9 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 | UTCID09 |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O | O |  |
| 12 |  |  |  | Logged in as Operator/Admin |  | O | O | O | O |  | O | O | O | O |  |
| 13 |  | status |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  |  |  | null (default statuses) |  | O | O |  | O | O | O | O |  | O |  |
| 15 |  |  |  | Valid status list |  |  |  | O |  |  |  |  |  |  |  |
| 16 |  |  |  | "abc" |  |  |  |  |  |  |  |  | O |  |  |
| 17 |  | page |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 18 |  |  |  | Valid (>= 1) |  | O | O | O | O | O | O |  | O | O |  |
| 19 |  |  |  | Invalid (0 / negative) |  |  |  |  |  |  |  | O |  |  |  |
| 20 |  | pageSize |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 21 |  |  |  | Valid(eg. 5, 10) |  | O | O | O | O | O | O | O | O |  |  |
| 22 |  |  |  | Invalid (0/negative) |  |  |  |  |  |  |  |  |  | O |  |
| 23 |  | since / until |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | Null |  | O |  | O |  | O |  | O | O | O |  |
| 25 |  |  |  | Valid range (eg. 15/4/2026 - 16/4/2026) |  |  | O |  | O |  |  |  |  |  |  |
| 26 |  |  |  | since -> until (eg. 16/4/2026 - 15/4/2026) |  |  |  |  |  |  | O |  |  |  |  |
| 27 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  | HTTP 200 |  | O | O | O |  |  | O | O | O | O |  |
| 29 |  |  |  | HTTP 401 |  |  |  |  | O | O |  |  |  |  |  |
| 30 |  |  |  | HTTP 422 |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  | "User incidents retrieved" |  | O | O | O |  |  | O | O | O | O |  |
| 35 |  |  |  | "Validate failed |  |  |  |  |  | O |  |  |  |  |  |
| 36 |  |  |  | "Unauthorized |  |  |  |  | O |  |  |  |  |  |  |
| 37 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | A | A | A | A | A |  |
| 38 |  | Passed/Failed |  |  |  | P | P | P | P | P | P | P | P | P |  |
| 39 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |
| 40 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
