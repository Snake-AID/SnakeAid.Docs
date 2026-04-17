# 16 - GetSnakeCatchingRequestDetail

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetSnakeCatchingRequestDetail`
- Used range: `A2:O36`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 3 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 6 |  | 1 |  |  | 0 |  |  |  |  |  | 4 | 3 | 0 | 7 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  | O | O |  |  |  |
| 12 |  |  |  | Logged in as Member/Operator/Rescuer/Admin |  | O | O | O | O | O |  |  |  |  |  |
| 13 |  | RequestId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  |  |  | Valid Id |  | O |  |  |  | O | O | O |  |  |  |
| 15 |  |  |  | Invalid Id |  |  | O |  |  |  |  |  |  |  |  |
| 16 |  |  |  | null |  |  |  | O | O |  |  |  |  |  |  |
| 17 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 18 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 19 |  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 500 |  |  |  |  | O | O |  |  |  |  |  |
| 21 |  |  |  | HTTP 400 |  |  |  | O |  |  |  |  |  |  |  |
| 22 |  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |
| 23 |  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |
| 24 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  |  |  | "Snake catching request details retrieved successfully." |  | O |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  | "Snake catching request with ID {requestId} not found." |  |  | O |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "Snake catching request ID is required." |  |  |  | O |  |  |  |  |  |  |  |
| 30 |  |  |  | "Internal server error." |  |  |  |  | O | O |  |  |  |  |  |
| 31 |  |  |  | "Access denied. You don't have permission to access this resource." |  |  |  |  |  |  |  | O |  |  |  |
| 32 |  |  |  | "Authentication required." |  |  |  |  |  |  | O |  |  |  |  |
| 33 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | N | N | N |  |  |  |
| 34 |  | Passed/Failed |  |  |  | P | P | F | P | P | P | P |  |  |  |
| 35 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |
| 36 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
