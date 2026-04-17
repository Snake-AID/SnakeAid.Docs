# 17 - ConfirmSnakeCatchingRequest

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `ConfirmSnakeCatchingRequest`
- Used range: `A2:O38`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 2 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 7 |  | 1 |  |  | 0 |  |  |  |  |  | 5 | 3 | 0 | 8 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  | O | O | O |  |  |
| 12 |  |  |  | Logged in as Operator |  | O | O | O | O | O |  |  | O |  |  |
| 13 |  |  |  | Request is in Pending state |  | O | O | O | O | O | O | O |  |  |  |
| 14 |  | RequestId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 15 |  |  |  | Valid Id |  | O |  |  |  | O | O | O | O |  |  |
| 16 |  |  |  | Invalid Id |  |  | O |  |  |  |  |  |  |  |  |
| 17 |  |  |  | null |  |  |  | O | O |  |  |  |  |  |  |
| 18 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 19 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 500 |  |  |  |  | O | O |  |  |  |  |  |
| 22 |  |  |  | HTTP 400 |  |  |  | O |  |  |  |  | O |  |  |
| 23 |  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |
| 24 |  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |
| 25 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  | "Snake catching request confirmed successfully!" |  | O |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "Snake catching request with ID {requestId} not found." |  |  | O |  |  |  |  |  |  |  |  |
| 30 |  |  |  | "Snake catching request ID is required." |  |  |  | O |  |  |  |  |  |  |  |
| 31 |  |  |  | "Request cannot be accepted. Current status: {status}" |  |  |  |  |  |  |  |  | O |  |  |
| 32 |  |  |  | "Internal server error." |  |  |  |  | O | O |  |  |  |  |  |
| 33 |  |  |  | "Access denied. You don't have permission to access this resource." |  |  |  |  |  |  |  | O |  |  |  |
| 34 |  |  |  | "Authentication required." |  |  |  |  |  |  | O |  |  |  |  |
| 35 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | N | N | N | N |  |  |
| 36 |  | Passed/Failed |  |  |  | P | P | F | P | P | P | P | P |  |  |
| 37 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |
| 38 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
