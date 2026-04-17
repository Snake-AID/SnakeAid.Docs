# 06 - CreateSnakebiteIncident

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `CreateSnakebiteIncident`
- Used range: `A2:Q46`

| Row | A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 | Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | CreateSnakeCatchingRequest |  |  |  |  |  |
| 3 | Created By |  | NhanNP |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| 4 | Lines  of code |  | 100.0 |  |  | Lack of test cases |  |  |  |  |  | -2 |  |  |  |  |  |
| 5 | Test requirement |  | <Brief description about requirements which are tested in this function> |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 6 | Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 7 | 10 |  | 2 |  |  | 0 |  |  |  |  |  | 2 | 6 | 4 | 12 |  |  |
| 8 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 | UTCID09 | UTCID11 | UTCID12 | UTCID13 |
| 10 | Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 11 |  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O | O | O | O | O |
| 12 |  |  |  | Logged in as Member |  | O | O | O | O | O | O | O | O | O | O | O | O |
| 13 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 14 |  | lng |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 15 |  |  |  | 106.0 |  | O |  |  | O | O |  |  | O | O | O |  | O |
| 16 |  |  |  | -181.0 |  |  | O |  |  |  |  |  |  |  |  |  |  |
| 17 |  |  |  | 181.0 |  |  |  | O |  |  |  |  |  |  |  |  |  |
| 18 |  |  |  | 180.0 |  |  |  |  |  |  | O |  |  |  |  |  |  |
| 19 |  |  |  | -180.0 |  |  |  |  |  |  |  | O |  |  |  |  |  |
| 20 |  |  |  | null |  |  |  |  |  |  |  |  |  |  |  | O |  |
| 21 |  | lat |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 22 |  |  |  | 10.0 |  | O | O | O |  |  | O | O |  |  | O | O |  |
| 23 |  |  |  | -91.0 |  |  |  |  | O |  |  |  |  |  |  |  |  |
| 24 |  |  |  | 91.0 |  |  |  |  |  | O |  |  |  |  |  |  |  |
| 25 |  |  |  | -90.0 |  |  |  |  |  |  |  |  | O |  |  |  |  |
| 26 |  |  |  | 90.0 |  |  |  |  |  |  |  |  |  | O |  |  |  |
| 27 |  |  |  | null |  |  |  |  |  |  |  |  |  |  |  |  | O |
| 28 |  | Address |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 29 |  |  |  | "HCM" |  | O | O | O | O | O | O | O | O | O |  | O | O |
| 30 |  |  |  | "" |  |  |  |  |  |  |  |  |  |  | O |  |  |
| 31 | Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 32 |  |  |  | HTTP 200 |  | O |  |  |  |  | O | O | O | O | O |  |  |
| 33 |  |  |  | HTTP 422 |  |  | O | O | O | O |  |  |  |  |  | O | O |
| 34 |  |  |  | HTTP 400 |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 35 |  | Exception |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 36 |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 37 |  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| 38 |  |  |  | "Snakebite incident created" |  | O |  |  |  |  | O | O | O | O | O |  |  |
| 39 |  |  |  | "The field Lng must be a valid number. (-180 to 180)" |  |  | O | O |  |  |  |  |  |  |  |  |  |
| 40 |  |  |  | "The field Lat must be a valid number. (-90 to 90)" |  |  |  |  | O | O |  |  |  |  |  |  |  |
| 41 |  |  |  | The field Lng is Required |  |  |  |  |  |  |  |  |  |  |  | O |  |
| 42 |  |  |  | The field Lat is Required |  |  |  |  |  |  |  |  |  |  |  |  | O |
| 43 | Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | A | A | A | A | B | B | B | B | N | A | A |
| 44 |  | Passed/Failed |  |  |  | P | P | P | P | P | F | F | P | P | P | P | P |
| 45 |  | Executed Date |  |  |  | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 | 2026-04-16 00:00:00 |
| 46 |  | Defect ID |  |  |  |  |  |  |  |  | DFID002 | DFID004 | DFID005 | DFID006 | DFID007 | DFID007 | DFID007 |
