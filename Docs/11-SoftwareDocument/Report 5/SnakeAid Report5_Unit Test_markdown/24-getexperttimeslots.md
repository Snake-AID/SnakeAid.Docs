# 24 - GetExpertTimeSlots

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertTimeSlots`
- Used range: `A2:N42`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  | GetExpertTimeSlots |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |
| Lines  of code |  | 15 |  |  | Lack of test cases |  |  |  |  | 0 |  |  |  |
| Test requirement |  | View available future time slots of one expert |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  | N/A/B |  | Total Test Cases |  |
| 0 |  | 0 |  |  | 4 |  |  |  |  | 0 | 3 | 1 | 4 |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |
|  | slot set |  |  |  | future available | past + reserved only | none | mixed |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O | O | O |  |  |  |  |  |
|  |  |  | size > 0 |  | O |  |  | O |  |  |  |  |  |
|  |  |  | size = 0 |  |  | O | O |  |  |  |  |  |  |
|  |  |  | future available only |  | O | O |  | O |  |  |  |  |  |
|  |  |  | reserved excluded |  |  | O |  | O |  |  |  |  |  |
|  |  |  | past excluded |  |  | O |  | O |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O | O | O |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | B | N |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
