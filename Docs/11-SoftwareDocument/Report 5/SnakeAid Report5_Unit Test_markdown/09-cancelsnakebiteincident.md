# 09 - CancelSnakebiteIncident

- Sheet: `CancelSnakebiteIncident`
- Used range: `A2:O39`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 5 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 5 |  | 0 |  |  | 0 |  |  |  |  |  | 3 | 2 | 0 | 5 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O |  | O |  |  |  |  |  |
| 13 |  |  |  | SnakebiteIncident exist |  | O |  | O | O | O |  |  |  |  |  |
| 14 |  |  |  | Incident in valid status |  | O |  |  | O | O |  |  |  |  |  |
| 15 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 16 |  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 17 |  |  |  | Valid Guid (exist) |  | O |  | O | O | O |  |  |  |  |  |
| 18 |  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
| 19 |  | Reason |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | Valid (not null) |  | O | O | O | O |  |  |  |  |  |  |
| 21 |  |  |  | null |  |  |  |  |  | O |  |  |  |  |  |
| 22 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 23 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
| 25 |  |  |  | HTTP 400 |  |  |  | O |  |  |  |  |  |  |  |
| 26 |  |  |  | HTTP 422 |  |  |  |  |  | O |  |  |  |  |  |
| 27 |  |  |  | HTTP 401 |  |  |  |  | O |  |  |  |  |  |  |
| 28 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  | "Incident cancelled successfully" |  | O |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  | "Incident not found" |  |  | O |  |  |  |  |  |  |  |  |
| 33 |  |  |  | "Cannot cancel incident with status {invalid status}" |  |  |  | O |  |  |  |  |  |  |  |
| 34 |  |  |  | Validation failed |  |  |  |  |  | O |  |  |  |  |  |
| 35 |  |  |  | "Unauthorized" |  |  |  |  | O |  |  |  |  |  |  |
| 36 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | N | N | A |  |  |  |  |  |
| 37 |  | Passed/Failed |  |  |  | P | P | P | P | P |  |  |  |  |  |
| 38 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |  |
| 39 |  | Defect ID |  |  |  |  |  |  |  | DFID002 |  |  |  |  |  |
