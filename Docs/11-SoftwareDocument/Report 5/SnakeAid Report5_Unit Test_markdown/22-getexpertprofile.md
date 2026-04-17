# 22 - GetExpertProfile

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertProfile`
- Used range: `A2:M43`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  | GetExpertProfile |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |
| Lines  of code |  | 49 |  |  | Lack of test cases |  |  |  | 0 |  |  |  |
| Test requirement |  | View one expert profile by id |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  | N/A/B |  | Total Test Cases |  |
| 0 |  | 0 |  |  | 4 |  |  |  | 0 | 3 | 1 | 4 |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |
|  |  |  | Expert profile exists |  | O | O |  |  |  |  |  |  |
|  |  |  | Account is active |  | O | O |  |  |  |  |  |  |
|  | role |  |  |  | public | user | public | public |  |  |  |  |
|  | expertId |  |  |  | valid existing id | valid existing id | valid missing id | inactive expert id |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  | O | O |  |  |  |  |
|  |  |  | data != null |  | O | O |  |  |  |  |  |  |
|  |  |  | data = null |  |  |  | O | O |  |  |  |  |
|  |  |  | "Expert profile not found." |  |  |  | O | O |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | B |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |
