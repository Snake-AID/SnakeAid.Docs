# 32 - CreateEmergencyConsultationRequest

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateEmergencyConsultationRequest`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateEmergencyConsultationRequest |  |  |  |  |  |
| Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 34 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Member creates an emergency consultation request with expert existence and duplicate-active-request validation |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 6 |  |  |  |  |  | 0 | 2 | 4 | 6 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  |  |  | Expert account/profile data prepared |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  | auth context |  |  |  | User | public | Expert | User | User | User |  |  |  |  |  |  |
|  | expertId |  |  |  | active expert | active expert | active expert | missing expert | active expert | active expert |  |  |  |  |  |  |
|  | active request state |  |  |  | none | none | none | none | PendingPayment not expired | PendingExpertResponse not expired |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 409 |  |  |  |  |  | O | O |  |  |  |  |  |  |
|  |  |  | status = PendingPayment |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | expiresAt > requestedAt |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | consultationId = null |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Selected expert was not found." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "An active emergency request already exists for this expert." |  |  |  |  |  | O | O |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O |  |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | B |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

