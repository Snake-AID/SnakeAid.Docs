# 16 - GetSnakeCatchingRequestDetail

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetSnakeCatchingRequestDetail`
- Used range: `A2:O36`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 3 |  |  |  |
| Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 6 |  | 1 |  |  | 0 |  |  |  |  |  | 4 | 3 | 0 | 7 |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O |  | O | O |  |  |  |
|  |  |  | Logged in as Member/Operator/Rescuer/Admin |  | O | O | O | O | O |  |  |  |  |  |
|  | RequestId |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Valid Id |  | O |  |  |  | O | O | O |  |  |  |
|  |  |  | Invalid Id |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | null |  |  |  | O | O |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 500 |  |  |  |  | O | O |  |  |  |  |  |
|  |  |  | HTTP 400 |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |
|  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Snake catching request details retrieved successfully." |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | "Snake catching request with ID {requestId} not found." |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "Snake catching request ID is required." |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "Internal server error." |  |  |  |  | O | O |  |  |  |  |  |
|  |  |  | "Access denied. You don't have permission to access this resource." |  |  |  |  |  |  |  | O |  |  |  |
|  |  |  | "Authentication required." |  |  |  |  |  |  | O |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | N | N | N |  |  |  |
|  | Passed/Failed |  |  |  | P | P | F | P | P | P | P |  |  |  |
|  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
