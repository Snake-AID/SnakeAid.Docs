# 07 - GetIncidentDetail

- Sheet: `GetIncidentDetail`
- Used range: `A2:O35`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 5 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 5 |  | 0 |  |  | 0 |  |  |  |  |  | 2 | 3 | 0 | 5 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O | O |  |  |  |  |  |  |
| 13 |  |  |  | SnakebiteIncident exist |  | O |  |  |  |  |  |  |  |  |  |
| 14 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 15 |  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 16 |  |  |  | Valid Guid (exist) |  | O |  |  |  | O |  |  |  |  |  |
| 17 |  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
| 18 |  |  |  | "abc" |  |  |  | O |  |  |  |  |  |  |  |
| 19 |  |  |  | null |  |  |  |  | O |  |  |  |  |  |  |
| 20 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 22 |  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
| 23 |  |  |  | HTTP 422 |  |  |  | O | O |  |  |  |  |  |  |
| 24 |  |  |  | HTTP 401 |  |  |  |  |  | O |  |  |  |  |  |
| 25 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  | "Incident details retrieved successfully" |  | O |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "Incident not found" |  |  | O |  |  |  |  |  |  |  |  |
| 30 |  |  |  | Validation failed |  |  |  | O | O |  |  |  |  |  |  |
| 31 |  |  |  | "Unauthorized" |  |  |  |  |  | O |  |  |  |  |  |
| 32 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | A |  |  |  |  |  |
| 33 |  | Passed/Failed |  |  |  | P | P | P | P | P |  |  |  |  |  |
| 34 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |  |
| 35 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
