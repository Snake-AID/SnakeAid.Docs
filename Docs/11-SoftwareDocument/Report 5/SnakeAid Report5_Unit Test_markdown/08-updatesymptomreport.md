# 08 - UpdateSymptomReport

- Sheet: `UpdateSymptomReport`
- Used range: `A2:O40`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 4 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 5 |  | 1 |  |  | 0 |  |  |  |  |  | 3 | 3 | 0 | 6 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O | O |  | O |  |  |  |  |
| 13 |  |  |  | SnakebiteIncident exist |  | O |  | O | O |  | O |  |  |  |  |
| 14 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 15 |  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 16 |  |  |  | Valid Guid (exist) |  | O |  | O | O | O |  |  |  |  |  |
| 17 |  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
| 18 |  |  |  | Invalid format |  |  |  |  |  |  | O |  |  |  |  |
| 19 |  | SymptomIdList |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | Valid (>= 1 item) |  | O | O |  | O | O | O |  |  |  |  |
| 21 |  |  |  | null |  |  |  | O |  |  |  |  |  |  |  |
| 22 |  | TimeSinceBiteMinutes |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 23 |  |  |  | valid (eg 30) |  | O | O | O |  | O | O |  |  |  |  |
| 24 |  |  |  | null |  |  |  |  | O |  |  |  |  |  |  |
| 25 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | HTTP 200 |  | O |  |  | O |  |  |  |  |  |  |
| 27 |  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |
| 28 |  |  |  | HTTP 422 |  |  |  | O |  |  | O |  |  |  |  |
| 29 |  |  |  | HTTP 401 |  |  |  |  |  | O |  |  |  |  |  |
| 30 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  |  |  | "Symptom report updated successfully" |  | O |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  | "Incident not found" |  |  | O |  |  |  | O |  |  |  |  |
| 35 |  |  |  | Validation failed |  |  |  | O | O |  |  |  |  |  |  |
| 36 |  |  |  | "Unauthorized" |  |  |  |  |  | O |  |  |  |  |  |
| 37 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | N | A |  |  |  |  |
| 38 |  | Passed/Failed |  |  |  | P | P | P | P | P | F |  |  |  |  |
| 39 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |
| 40 |  | Defect ID |  |  |  |  |  |  |  |  | DFID002 |  |  |  |  |
