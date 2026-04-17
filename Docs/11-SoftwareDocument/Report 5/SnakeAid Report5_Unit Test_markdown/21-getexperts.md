# 21 - GetExperts

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExperts`
- Used range: `A2:Q45`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | GetExperts |  |  |  |  |  |
| 3 | Created By |  | Codex |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 89 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| 5 | Test requirement |  | Browse experts with filter, sort, and pagination validation |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 7 | 0 |  | 0 |  |  | 7 |  |  |  |  |  | 0 | 5 | 2 | 7 |  |  |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 |  |  |  |  |  |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O |  |  |  |  |  |
| 12 |  |  |  | Active expert data exists |  | O | O | O | O | O | O | O |  |  |  |  |  |
| 13 |  | isOnline |  |  |  | null | true | null | null | null | null | null |  |  |  |  |  |
| 14 |  | specialization |  |  |  | null | "Doc" | null | null | null | null | null |  |  |  |  |  |
| 15 |  | sortBy |  |  |  | null | null | "consultationFee" | "consultationFee" | "invalidField" | "rating" | "rating" |  |  |  |  |  |
| 16 |  | sortOrder |  |  |  | null | null | "asc" | "desc" | "asc" | "invalid" | "asc" |  |  |  |  |  |
| 17 |  | pageNumber |  |  |  | 1 | 1 | 1 | 1 | 1 | 1 | 0 |  |  |  |  |  |
| 18 |  | pageSize |  |  |  | 10 | 10 | 10 | 10 | 10 | 10 | 10 |  |  |  |  |  |
| 19 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 20 |  |  |  | HTTP 200 |  | O | O | O | O |  |  |  |  |  |  |  |  |
| 21 |  |  |  | HTTP 422 |  |  |  |  |  | O | O | O |  |  |  |  |  |
| 22 |  |  |  | size > 0 |  | O | O | O | O |  |  |  |  |  |  |  |  |
| 23 |  |  |  | filter = online + specialization |  |  | O |  |  |  |  |  |  |  |  |  |  |
| 24 |  |  |  | order = fee asc |  |  |  | O |  |  |  |  |  |  |  |  |  |
| 25 |  |  |  | order = fee desc |  |  |  |  | O |  |  |  |  |  |  |  |  |
| 26 |  |  |  | "SortBy must be one of: isOnline, rating, consultationFee." |  |  |  |  |  | O |  |  |  |  |  |  |  |
| 27 |  |  |  | "SortOrder must be either asc or desc." |  |  |  |  |  |  | O |  |  |  |  |  |  |
| 28 |  |  |  | "The field PageNumber must be between 1 and 2147483647." |  |  |  |  |  |  |  | O |  |  |  |  |  |
| 29 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 30 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 31 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  | "Operation successful" |  | O | O | O | O |  |  |  |  |  |  |  |  |
| 33 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | N | N | A | A | B |  |  |  |  |  |
| 34 |  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 39 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 41 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 42 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 43 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 44 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 45 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
