# 25 - UpdateExpertSettings

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `UpdateExpertSettings`
- Used range: `A2:R45`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q | R |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | UpdateExpertSettings |  |  |  |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 53 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |  |
| 5 | Test requirement |  | Expert updates biography and fees |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |  |
| 7 | 0 |  | 0 |  |  | 8 |  |  |  |  |  | 0 | 6 | 2 | 8 |  |  |  |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Logged in as Expert |  | O | O | O | O | O |  |  | O |  |  |  |  |  |
| 13 |  |  |  | Expert profile exists |  | O | O | O | O | O | O | O |  |  |  |  |  |  |
| 14 |  | biography |  |  |  | "bio" | "bio" | null | "bio" | "bio" | "bio" | "bio" | "bio" |  |  |  |  |  |
| 15 |  | scheduledConsultationFee |  |  |  | 150000 | 150000 | 150000 | null | -1 | 150000 | 150000 | 150000 |  |  |  |  |  |
| 16 |  | emergencyConsultationFee |  |  |  | 250000 | null | 250000 | 250000 | 250000 | 250000 | 250000 | 250000 |  |  |  |  |  |
| 17 |  | role |  |  |  | Expert | Expert | Expert | Expert | Expert | public | User | Expert |  |  |  |  |  |
| 18 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 19 |  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 401 |  |  |  |  |  |  | O |  |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 403 |  |  |  |  |  |  |  | O |  |  |  |  |  |  |
| 22 |  |  |  | HTTP 404 |  |  |  |  |  |  |  |  | O |  |  |  |  |  |
| 23 |  |  |  | HTTP 422 |  |  |  | O | O | O |  |  |  |  |  |  |  |  |
| 24 |  |  |  | emergencyFee = scheduledFee |  |  | O |  |  |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  | "ScheduledConsultationFee is required." |  |  |  |  | O |  |  |  |  |  |  |  |  |  |
| 26 |  |  |  | "The field Biography is required." |  |  |  | O |  |  |  |  |  |  |  |  |  |  |
| 27 |  |  |  | "The field ScheduledConsultationFee must be between 0 and 999999.99." |  |  |  |  |  | O |  |  |  |  |  |  |  |  |
| 28 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "Settings updated successfully" |  | O | O |  |  |  |  |  |  |  |  |  |  |  |
| 30 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | B | A | A | A |  |  |  |  |  |
| 31 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 33 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 34 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 44 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 45 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
