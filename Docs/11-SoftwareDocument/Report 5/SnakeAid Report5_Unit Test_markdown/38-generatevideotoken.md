# 38 - GenerateVideoToken

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GenerateVideoToken`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GenerateVideoToken |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 56 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Participant or admin generates a LiveKit consultation token with consultation existence, room, and authorization checks |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 6 |  |  |  |  |  | 0 | 2 | 4 | 6 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 |  |  |  |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  |  |  | Consultation data prepared |  | O | O | O | O | O | O |  |  |  |  |  |  |
|  | auth context |  |  |  | participant User | Admin | public | other authenticated user | participant User | participant User |  |  |  |  |  |  |
|  | consultationId |  |  |  | existing consultation with roomId | existing consultation with roomId | existing consultation with roomId | existing consultation with roomId | missing consultation | existing consultation with blank roomId |  |  |  |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | token != null |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | roomName = consultation.RoomId |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | wsUrl != null |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "You are not allowed to access this consultation room." |  |  |  |  | O |  |  |  |  |  |  |  |  |
|  |  |  | "Consultation not found." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "Consultation room is not initialized." |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Video token generated successfully" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | A | A |  |  |  |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

