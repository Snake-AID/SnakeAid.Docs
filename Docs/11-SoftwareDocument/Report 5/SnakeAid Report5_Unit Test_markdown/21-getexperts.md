# 21 - GetExperts

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExperts`
- Used range: `A2:Q45`

**Summary**

| A                | B | C                                                           | D | E | F                  | G | H | I | J | K | L          | M | N | O                | P | Q |
| ---------------- | - | ----------------------------------------------------------- | - | - | ------------------ | - | - | - | - | - | ---------- | - | - | ---------------- | - | - |
| Function Code    |   | Function1                                                   |   |   | Function Name      |   |   |   |   |   | GetExperts |   |   |                  |   |   |
| Created By       |   | KhiemNVD                                                       |   |   | Executed By        |   |   |   |   |   |            |   |   |                  |   |   |
| Lines  of code   |   | 89                                                          |   |   | Lack of test cases |   |   |   |   |   | 0          |   |   |                  |   |   |
| Test requirement |   | Browse experts with filter, sort, and pagination validation |   |   |                    |   |   |   |   |   |            |   |   |                  |   |   |
| Passed           |   | Failed                                                      |   |   | Untested           |   |   |   |   |   | N/A/B      |   |   | Total Test Cases |   |   |
| 0                |   | 0                                                           |   |   | 7                  |   |   |   |   |   | 0          | 5 | 2 | 7                |   |   |

**Matrix**

| A         | B                                            | C | D                                                           | E | F       | G       | H                 | I                 | J              | K         | L        | M | N | O | P | Q |
| --------- | -------------------------------------------- | - | ----------------------------------------------------------- | - | ------- | ------- | ----------------- | ----------------- | -------------- | --------- | -------- | - | - | - | - | - |
|           |                                              |   |                                                             |   | UTCID01 | UTCID02 | UTCID03           | UTCID04           | UTCID05        | UTCID06   | UTCID07  |   |   |   |   |   |
| Condition | Precondition                                 |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   | Can connect with server                                     |   | O       | O       | O                 | O                 | O              | O         | O        |   |   |   |   |   |
|           |                                              |   | Active expert data exists                                   |   | O       | O       | O                 | O                 | O              | O         | O        |   |   |   |   |   |
|           | isOnline                                     |   |                                                             |   | null    | true    | null              | null              | null           | null      | null     |   |   |   |   |   |
|           | specialization                               |   |                                                             |   | null    | "Doc"   | null              | null              | null           | null      | null     |   |   |   |   |   |
|           | sortBy                                       |   |                                                             |   | null    | null    | "consultationFee" | "consultationFee" | "invalidField" | "rating"  | "rating" |   |   |   |   |   |
|           | sortOrder                                    |   |                                                             |   | null    | null    | "asc"             | "desc"            | "asc"          | "invalid" | "asc"    |   |   |   |   |   |
|           | pageNumber                                   |   |                                                             |   | 1       | 1       | 1                 | 1                 | 1              | 1         | 0        |   |   |   |   |   |
|           | pageSize                                     |   |                                                             |   | 10      | 10      | 10                | 10                | 10             | 10        | 10       |   |   |   |   |   |
| Confirm   | Return                                       |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   | HTTP 200                                                    |   | O       | O       | O                 | O                 |                |           |          |   |   |   |   |   |
|           |                                              |   | HTTP 422                                                    |   |         |         |                   |                   | O              | O         | O        |   |   |   |   |   |
|           |                                              |   | size > 0                                                    |   | O       | O       | O                 | O                 |                |           |          |   |   |   |   |   |
|           |                                              |   | filter = online + specialization                            |   |         | O       |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   | order = fee asc                                             |   |         |         | O                 |                   |                |           |          |   |   |   |   |   |
|           |                                              |   | order = fee desc                                            |   |         |         |                   | O                 |                |           |          |   |   |   |   |   |
|           |                                              |   | "SortBy must be one of: isOnline, rating, consultationFee." |   |         |         |                   |                   | O              |           |          |   |   |   |   |   |
|           |                                              |   | "SortOrder must be either asc or desc."                     |   |         |         |                   |                   |                | O         |          |   |   |   |   |   |
|           |                                              |   | "The field PageNumber must be between 1 and 2147483647."    |   |         |         |                   |                   |                |           | O        |   |   |   |   |   |
|           | Exception                                    |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           | Log message                                  |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   | "Operation successful"                                      |   | O       | O       | O                 | O                 |                |           |          |   |   |   |   |   |
| Result    | Type(N : Normal, A : Abnormal, B : Boundary) |   |                                                             |   | N       | N       | N                 | N                 | A              | A         | B        |   |   |   |   |   |
|           | Passed/Failed                                |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           | Executed Date                                |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           | Defect ID                                    |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
|           |                                              |   |                                                             |   |         |         |                   |                   |                |           |          |   |   |   |   |   |
