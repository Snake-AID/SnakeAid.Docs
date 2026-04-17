# 40 - CreateConsultationReview

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateConsultationReview`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateConsultationReview |  |  |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 89 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Member creates a consultation review with completed-consultation, duplicate-review, and rating validation |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 8 |  |  |  |  |  | 0 | 2 | 6 | 8 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O |  |  |  |  |
|  |  |  | Consultation data prepared |  | O | O | O | O | O | O | O | O |  |  |  |  |
|  | auth context |  |  |  | User caller | User caller | public | Expert | other User | User caller | User caller | User caller |  |  |  |  |
|  | consultationId |  |  |  | completed consultation | missing consultation | completed consultation | completed consultation | completed consultation | ongoing consultation | completed consultation | completed consultation |  |  |  |  |
|  | existing review |  |  |  | none | none | none | none | none | none | exists | none |  |  |  |  |
|  | rating |  |  |  | 5 | 5 | 5 | 5 | 5 | 5 | 5 | 0 |  |  |  |  |
|  | comments |  |  |  | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." | "Very helpful consultation." |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 400 |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 409 |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  |  |  | type = Consultation |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | targetUserId = expertId |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | updatedAverageRating > 0 |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Consultation not found." |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Only the user who booked the consultation can submit a review." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "Consultation must be completed before review submission." |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | "You have already submitted a review for this consultation." |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | "The field Rating must be between 1 and 5." |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O |  |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | A | A | B |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

