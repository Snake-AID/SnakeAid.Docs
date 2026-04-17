# 24 - GetExpertTimeSlots

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertTimeSlots`
- Used range: `A2:N42`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  | GetExpertTimeSlots |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 15 |  |  | Lack of test cases |  |  |  |  | 0 |  |  |  |
| 5 | Test requirement |  | View available future time slots of one expert |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  | N/A/B |  | Total Test Cases |  |
| 7 | 0 |  | 0 |  |  | 4 |  |  |  |  | 0 | 3 | 1 | 4 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |
| 12 |  | slot set |  |  |  | future available | past + reserved only | none | mixed |  |  |  |  |  |
| 13 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  |  |  | HTTP 200 |  | O | O | O | O |  |  |  |  |  |
| 15 |  |  |  | size > 0 |  | O |  |  | O |  |  |  |  |  |
| 16 |  |  |  | size = 0 |  |  | O | O |  |  |  |  |  |  |
| 17 |  |  |  | future available only |  | O | O |  | O |  |  |  |  |  |
| 18 |  |  |  | reserved excluded |  |  | O |  | O |  |  |  |  |  |
| 19 |  |  |  | past excluded |  |  | O |  | O |  |  |  |  |  |
| 20 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |
| 21 |  |  |  | "Operation successful" |  | O | O | O | O |  |  |  |  |  |
| 22 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | B | N |  |  |  |  |  |
| 23 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |
| 24 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |
| 25 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
