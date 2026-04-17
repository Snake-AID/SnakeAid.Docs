# 10 - IdentifySnakeByAI

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `IdentifySnakeByAI`
- Used range: `A2:O38`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |
| Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | 5 |  |  |  |
| Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |
| 5 |  | 0 |  |  | 0 |  |  |  |  |  | 4 | 1 | 0 | 5 |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O |  |  |  |  |  |
|  |  |  | Logged in as Member |  | O | O | O | O | O |  |  |  |  |  |
|  |  |  | SnakebiteIncident exist |  | O |  | O | O | O |  |  |  |  |  |
|  |  |  | RecognitionResult exist |  | O | O |  | O | O |  |  |  |  |  |
|  |  |  | RecognitionResult valid |  | O | O | O |  | O |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | incidentId |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Valid Guid (exist) |  | O |  | O | O | O |  |  |  |  |  |
|  |  |  | Valid Guid (not exist) |  |  | O |  |  |  |  |  |  |  |  |
|  | RecognitionResultId |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Valid (not null) |  | O | O | O | O |  |  |  |  |  |  |
|  |  |  | null |  |  |  |  |  | O |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  | O | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 400 |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  | O |  |  |  |  |  |
|  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Snake identified successfully" |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | "Incident or recognition result not found" |  |  | O | O |  |  |  |  |  |  |  |
|  |  |  | "Invalid recognition result" |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | Validation failed |  |  |  |  |  | O |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | N | N | A |  |  |  |  |  |
|  | Passed/Failed |  |  |  | P | P | P | P | P |  |  |  |  |  |
|  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |
