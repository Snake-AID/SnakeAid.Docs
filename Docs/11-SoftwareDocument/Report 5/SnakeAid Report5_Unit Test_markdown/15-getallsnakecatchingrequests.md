# 15 - GetAllSnakeCatchingRequests

- Sheet: `GetAllSnakeCatchingRequests`
- Used range: `A2:O45`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 10 |  | 0 |  |  | 0 |  |  |  |  |  | 4 | 6 | 0 | 10 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 | UTCID09 | UTCID10 |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O |  | O | O |
| 12 |  |  |  | Logged in as Member/Operator/Rescuer/Admin |  | O | O | O | O | O | O | O | O |  |  |
| 13 |  | UserId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  |  |  | Valid Id |  | O |  |  | O | O | O | O | O | O | O |
| 15 |  |  |  | Invalid Id |  |  | O |  |  |  |  |  |  |  |  |
| 16 |  |  |  | null |  |  |  | O |  |  |  |  |  |  |  |
| 17 |  | AssignedRescuerId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 18 |  |  |  | Valid Id |  | O | O | O |  |  | O | O | O | O | O |
| 19 |  |  |  | Invalid Id |  |  |  |  | O |  |  |  |  |  |  |
| 20 |  |  |  | null |  |  |  |  |  | O |  |  |  |  |  |
| 21 |  | Status |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 22 |  |  |  | Valid status |  | O | O | O | O | O |  |  | O | O | O |
| 23 |  |  |  | Invalid status |  |  |  |  |  |  | O |  |  |  |  |
| 24 |  |  |  | null |  |  |  |  |  |  |  | O |  |  |  |
| 25 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | HTTP 200 |  | O |  | O |  | O |  | O |  |  |  |
| 27 |  |  |  | HTTP 400 |  |  |  |  |  |  | O |  |  |  |  |
| 28 |  |  |  | HTTP 404 |  |  | O |  | O |  |  |  |  |  |  |
| 29 |  |  |  | HTTP 500 |  |  |  |  |  |  |  |  | O |  |  |
| 30 |  |  |  | HTTP 401 |  |  |  |  |  |  |  |  |  | O |  |
| 31 |  |  |  | HTTP 403 |  |  |  |  |  |  |  |  |  |  | O |
| 32 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  |  |  | "Retrieved {count} snake catching request(s) successfully." |  | O |  | O |  | O |  | O |  |  |  |
| 36 |  |  |  | "User with ID {UserId} not found." |  |  | O |  |  |  |  |  |  |  |  |
| 37 |  |  |  | "Rescuer with ID {AssignedRescuerId} not found." |  |  |  |  | O |  |  |  |  |  |  |
| 38 |  |  |  | "Invalid status." |  |  |  |  |  |  | O |  |  |  |  |
| 39 |  |  |  | "Internal server error." |  |  |  |  |  |  |  |  | O |  |  |
| 40 |  |  |  | "Access denied. You don't have permission to access this resource." |  |  |  |  |  |  |  |  |  |  | O |
| 41 |  |  |  | "Authentication required." |  |  |  |  |  |  |  |  |  | O |  |
| 42 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | A | A | N | N | N |
| 43 |  | Passed/Failed |  |  |  | P | P | P | P | P | P | P | P | P | P |
| 44 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |
| 45 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
