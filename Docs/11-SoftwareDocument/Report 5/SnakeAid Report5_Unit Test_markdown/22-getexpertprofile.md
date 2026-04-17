# 22 - GetExpertProfile

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertProfile`
- Used range: `A2:M43`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  | GetExpertProfile |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 49 |  |  | Lack of test cases |  |  |  | 0 |  |  |  |
| 5 | Test requirement |  | View one expert profile by id |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  | N/A/B |  | Total Test Cases |  |
| 7 | 0 |  | 0 |  |  | 4 |  |  |  | 0 | 3 | 1 | 4 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |
| 12 |  |  |  | Expert profile exists |  | O | O |  |  |  |  |  |  |
| 13 |  |  |  | Account is active |  | O | O |  |  |  |  |  |  |
| 14 |  | role |  |  |  | public | user | public | public |  |  |  |  |
| 15 |  | expertId |  |  |  | valid existing id | valid existing id | valid missing id | inactive expert id |  |  |  |  |
| 16 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |
| 17 |  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |
| 18 |  |  |  | HTTP 404 |  |  |  | O | O |  |  |  |  |
| 19 |  |  |  | data != null |  | O | O |  |  |  |  |  |  |
| 20 |  |  |  | data = null |  |  |  | O | O |  |  |  |  |
| 21 |  |  |  | "Expert profile not found." |  |  |  | O | O |  |  |  |  |
| 22 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |
| 23 |  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |
| 24 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | B |  |  |  |  |
| 25 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |
| 26 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |
