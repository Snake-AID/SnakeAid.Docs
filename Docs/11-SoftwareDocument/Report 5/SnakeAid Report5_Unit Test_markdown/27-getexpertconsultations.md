# 27 - GetExpertConsultations

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertConsultations`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetExpertConsultations |  |  |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 153 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | View expert consultation history with filter and pagination |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 8 |  |  |  |  |  | 0 | 6 | 2 | 8 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O |  |  |  |  |
|  |  |  | Logged in as Expert |  | O | O | O | O | O |  |  | O |  |  |  |  |
|  |  |  | Consultation data exists |  | O | O | O | O | O | O | O | O |  |  |  |  |
|  | status |  |  |  | null | null | "Completed" | null | null | null | null | null |  |  |  |  |
|  | type |  |  |  | null | null | null | "Scheduled" | null | null | null | null |  |  |  |  |
|  | pageNumber |  |  |  | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |  |  |  |  |
|  | pageSize |  |  |  | 100 | 1 | 100 | 100 | 100 | 100 | 100 | 0 |  |  |  |  |
|  | role |  |  |  | Expert | Expert | Expert | Expert | Expert | public | User | Expert |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O | O | O | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  |  |  | totalItems > 0 |  | O | O | O | O | O |  |  |  |  |  |  |  |
|  |  |  | item count <= pageSize |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | status = Completed |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | type = Scheduled |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | sorted by StartTime desc |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "The field PageSize must be between 1 and 100." |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O | O | O | O |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | B | N | N | N | A | A | B |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
