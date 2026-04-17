# 11 - IdentifySnakeManual

- Sheet: `IdentifySnakeManual`
- Used range: `A2:O37`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 6 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 4 |  | 0 |  |  | 0 |  |  |  |  |  | 3 | 1 | 0 | 4 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O | O |  |  |  |  |  |  |
| 13 |  |  |  | SnakebiteIncident exist |  | O |  | O | O |  |  |  |  |  |  |
| 14 |  |  |  | SnakeSpeciesId exist |  | O | O |  | O |  |  |  |  |  |  |
| 15 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 16 |  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 17 |  |  |  | Valid Guid (exist) |  | O |  | O | O |  |  |  |  |  |  |
| 18 |  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
| 19 |  | SnakeSpecies |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | Valid (not null) |  | O | O | O |  |  |  |  |  |  |  |
| 21 |  |  |  | null |  |  |  |  | O |  |  |  |  |  |  |
| 22 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 23 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | HTTP 404 |  |  | O | O |  |  |  |  |  |  |  |
| 25 |  |  |  | HTTP 400 |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | HTTP 422 |  |  |  |  | O |  |  |  |  |  |  |
| 27 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  |  |  | "Snake identified successfully" |  | O |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  | "Incident not found" |  |  | O |  |  |  |  |  |  |  |  |
| 32 |  |  |  | "Snake Species Id not found" |  |  |  | O |  |  |  |  |  |  |  |
| 33 |  |  |  | Validation failed |  |  |  |  | O |  |  |  |  |  |  |
| 34 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | N | A |  |  |  |  |  |  |
| 35 |  | Passed/Failed |  |  |  | P | P | P | P |  |  |  |  |  |  |
| 36 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |  |  |
| 37 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
