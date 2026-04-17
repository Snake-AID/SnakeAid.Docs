# 34 - AcceptEmergencyConsultationRequest

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `AcceptEmergencyConsultationRequest`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | AcceptEmergencyConsultationRequest |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 72 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Expert accepts a paid emergency request and creates an ongoing emergency consultation while reserving overlapping slots |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 7 |  |  |  |  |  | 0 | 2 | 5 | 7 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O |  |  |  |  |  |
|  |  |  | Emergency request data prepared |  | O | O | O | O | O | O | O |  |  |  |  |  |
|  | auth context |  |  |  | Expert | public | User | Expert | Expert | Expert | Expert |  |  |  |  |  |
|  | requestId |  |  |  | pending request | pending request | pending request | missing request | other expert request | expired request | AcceptedByExpert request |  |  |  |  |  |
|  | overlapping available slots |  |  |  | exists | exists | exists | n/a | n/a | n/a | n/a |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  | O |  | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 409 |  |  |  |  |  |  | O | O |  |  |  |  |  |
|  |  |  | status = AcceptedByExpert |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultation.Status = Ongoing |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultation.Type = Emergency |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | roomId starts with "consultation-" |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | overlapping slots = Reserved |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | non-overlapping slots unchanged |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Emergency consultation request was not found." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "You are not allowed to accept this emergency request." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "Emergency request has expired." |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | "Emergency request is no longer pending." |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O |  |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | B | A |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

