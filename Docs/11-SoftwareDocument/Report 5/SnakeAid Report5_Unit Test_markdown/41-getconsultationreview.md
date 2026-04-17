# 41 - GetConsultationReview

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetConsultationReview`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetConsultationReview |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 46 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Authenticated consultation participant retrieves review detail or receives `data = null` when no review exists |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 6 |  |  |  |  |  | 0 | 2 | 4 | 6 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  |  |  | Consultation/review data prepared |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  | auth context |  |  |  | participant | participant | public | other authenticated user | participant | participant |  |  |  |  |  |  |
|  | consultationId |  |  |  | consultation with review | consultation without review | consultation with review | consultation with review | missing consultation | consultation with review |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | data != null |  | O |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | data = null |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | rating/comments returned |  | O |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | message = "No review found for this consultation." |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "You are not a participant of this consultation." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "Consultation not found." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O |  |  |  |  | O |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | A | B |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

