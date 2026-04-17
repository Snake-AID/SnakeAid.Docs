# 27 - GetExpertConsultations

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertConsultations`
- Used range: `A2:Q45`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetExpertConsultations |  |  |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 153 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| 5 | Test requirement |  | View expert consultation history with filter and pagination |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 7 | 0 |  | 0 |  |  | 8 |  |  |  |  |  | 0 | 6 | 2 | 8 |  |  |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O |  |  |  |  |
| 12 |  |  |  | Logged in as Expert |  | O | O | O | O | O |  |  | O |  |  |  |  |
| 13 |  |  |  | Consultation data exists |  | O | O | O | O | O | O | O | O |  |  |  |  |
| 14 |  | status |  |  |  | null | null | "Completed" | null | null | null | null | null |  |  |  |  |
| 15 |  | type |  |  |  | null | null | null | "Scheduled" | null | null | null | null |  |  |  |  |
| 16 |  | pageNumber |  |  |  | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |  |  |  |  |
| 17 |  | pageSize |  |  |  | 100 | 1 | 100 | 100 | 100 | 100 | 100 | 0 |  |  |  |  |
| 18 |  | role |  |  |  | Expert | Expert | Expert | Expert | Expert | public | User | Expert |  |  |  |  |
| 19 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 200 |  | O | O | O | O | O |  |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |  |  |
| 22 |  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |  |  |
| 23 |  |  |  | HTTP 422 |  |  |  |  |  |  |  |  | O |  |  |  |  |
| 24 |  |  |  | totalItems > 0 |  | O | O | O | O | O |  |  |  |  |  |  |  |
| 25 |  |  |  | item count <= pageSize |  |  | O |  |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | status = Completed |  |  |  | O |  |  |  |  |  |  |  |  |  |
| 27 |  |  |  | type = Scheduled |  |  |  |  | O |  |  |  |  |  |  |  |  |
| 28 |  |  |  | sorted by StartTime desc |  |  |  |  |  | O |  |  |  |  |  |  |  |
| 29 |  |  |  | "The field PageSize must be between 1 and 100." |  |  |  |  |  |  |  |  | O |  |  |  |  |
| 30 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  |  |  | "Operation successful" |  | O | O | O | O | O |  |  |  |  |  |  |  |
| 32 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | B | N | N | N | A | A | B |  |  |  |  |
| 33 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 44 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 45 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
