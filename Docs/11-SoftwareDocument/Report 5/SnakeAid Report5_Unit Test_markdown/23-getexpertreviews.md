# 23 - GetExpertReviews

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertReviews`
- Used range: `A2:N43`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  | GetExpertReviews |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 28 |  |  | Lack of test cases |  |  |  |  | 0 |  |  |  |
| 5 | Test requirement |  | View consultation reviews of one expert |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  | N/A/B |  | Total Test Cases |  |
| 7 | 0 |  | 0 |  |  | 4 |  |  |  |  | 0 | 2 | 2 | 4 |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Feedback data exists |  | O |  | O | O |  |  |  |  |  |
| 13 |  | pageNumber |  |  |  | 1 | 1 | 0 | 1 |  |  |  |  |  |
| 14 |  | pageSize |  |  |  | 10 | 10 | 10 | 0 |  |  |  |  |  |
| 15 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |
| 16 |  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |
| 17 |  |  |  | HTTP 422 |  |  |  | O | O |  |  |  |  |  |
| 18 |  |  |  | size > 0 |  | O |  |  |  |  |  |  |  |  |
| 19 |  |  |  | size = 0 |  |  | O |  |  |  |  |  |  |  |
| 20 |  |  |  | type = Consultation |  | O |  |  |  |  |  |  |  |  |
| 21 |  |  |  | "The field PageNumber must be between 1 and 2147483647." |  |  |  | O |  |  |  |  |  |  |
| 22 |  |  |  | "The field PageSize must be between 1 and 100." |  |  |  |  | O |  |  |  |  |  |
| 23 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |  |
| 25 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | B | B |  |  |  |  |  |
| 26 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |
| 27 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |
| 28 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |
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
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
