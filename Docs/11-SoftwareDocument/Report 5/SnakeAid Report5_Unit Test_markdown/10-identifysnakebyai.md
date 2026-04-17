# 10 - IdentifySnakeByAI

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `IdentifySnakeByAI`
- Used range: `A2:O38`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 5 |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 7 | 5 |  | 0 |  |  | 0 |  |  |  |  |  | 4 | 1 | 0 | 5 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Logged in as Member |  | O | O | O | O | O |  |  |  |  |  |
| 13 |  |  |  | SnakebiteIncident exist |  | O |  | O | O | O |  |  |  |  |  |
| 14 |  |  |  | RecognitionResult exist |  | O | O |  | O | O |  |  |  |  |  |
| 15 |  |  |  | RecognitionResult valid |  | O | O | O |  | O |  |  |  |  |  |
| 16 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 17 |  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 18 |  |  |  | Valid Guid (exist) |  | O |  | O | O | O |  |  |  |  |  |
| 19 |  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
| 20 |  | RecognitionResultId |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 21 |  |  |  | Valid (not null) |  | O | O | O | O |  |  |  |  |  |  |
| 22 |  |  |  | null |  |  |  |  |  | O |  |  |  |  |  |
| 23 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  | HTTP 404 |  |  | O | O |  |  |  |  |  |  |  |
| 26 |  |  |  | HTTP 400 |  |  |  |  | O |  |  |  |  |  |  |
| 27 |  |  |  | HTTP 422 |  |  |  |  |  | O |  |  |  |  |  |
| 28 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  | "Snake identified successfully" |  | O |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  | "Incident or recognition result not found" |  |  | O | O |  |  |  |  |  |  |  |
| 33 |  |  |  | "Invalid recognition result" |  |  |  |  | O |  |  |  |  |  |  |
| 34 |  |  |  | Validation failed |  |  |  |  |  | O |  |  |  |  |  |
| 35 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | N | N | A |  |  |  |  |  |
| 36 |  | Passed/Failed |  |  |  | P | P | P | P | P |  |  |  |  |  |
| 37 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |  |
| 38 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
