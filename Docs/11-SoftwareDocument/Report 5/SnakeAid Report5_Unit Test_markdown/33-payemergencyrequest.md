# 33 - PayEmergencyRequest

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `PayEmergencyRequest`
- Used range: `A2:Q45`

**Summary**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Function Code |  | Function1 |  |  | Function Name |  |  |  |  |  | PayEmergencyRequest |  |  |  |  |  |
| Created By |  | KhiemNVD |  |  | Executed By |  |  |  |  |  |  |  |  |  |  |  |
| Lines  of code |  | 121 |  |  | Lack of test cases |  |  |  |  |  | 0 |  |  |  |  |  |
| Test requirement |  | Member pays an emergency request by wallet or PayOS with status, ownership, and online-expert validation |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
| Passed |  | Failed |  |  | Untested |  |  |  |  |  | N/A/B |  |  | Total Test Cases |  |  |
| 0 |  | 0 |  |  | 9 |  |  |  |  |  | 0 | 3 | 6 | 9 |  |  |

**Matrix**

| A | B | C | D | E | F | G | H | I | J | K | L | M | N | O | P | Q |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  |  | UTCID01 | UTCID02 | UTCID03 | UTCID04 | UTCID05 | UTCID06 | UTCID07 | UTCID08 | UTCID09 |  |  |  |
| Condition | Precondition |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | Can connect with server |  | O | O | O | O | O | O | O | O | O |  |  |  |
|  |  |  | Emergency request data prepared |  | O | O | O | O | O | O | O | O | O |  |  |  |
|  | auth context |  |  |  | User | User | public | Expert | User | User | User | User | User |  |  |  |
|  | request ownership |  |  |  | current user | current user | current user | current user | missing request | other user | current user | current user | current user |  |  |  |
|  | request status |  |  |  | PendingPayment | PendingPayment | PendingPayment | PendingPayment | n/a | PendingPayment | PendingExpertResponse | PendingPayment | PendingPayment |  |  |  |
|  | expert connected |  |  |  | true | true | true | true | n/a | true | true | false | true |  |  |  |
|  | paymentMethod |  |  |  | WalletBalance | PayOs | WalletBalance | WalletBalance | WalletBalance | WalletBalance | WalletBalance | WalletBalance | 99 |  |  |  |
| Confirm | Return |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 200 |  | O | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 401 |  |  |  | O |  |  |  |  |  |  |  |  |  |
|  |  |  | HTTP 403 |  |  |  |  | O |  | O |  |  |  |  |  |  |
|  |  |  | HTTP 404 |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | HTTP 409 |  |  |  |  |  |  |  | O | O |  |  |  |  |
|  |  |  | HTTP 422 |  |  |  |  |  |  |  |  |  | O |  |  |  |
|  |  |  | status = "Escrowed" |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | status = "Pending" |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | request.Status = PendingExpertResponse |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | expiresAt refreshed |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | provider = "Wallet" |  | O |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | checkoutUrl != null |  |  | O |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Emergency consultation request was not found." |  |  |  |  |  | O |  |  |  |  |  |  |  |
|  |  |  | "You are not allowed to pay for this emergency request." |  |  |  |  |  |  | O |  |  |  |  |  |  |
|  |  |  | "Emergency consultation request is no longer waiting for payment." |  |  |  |  |  |  |  | O |  |  |  |  |  |
|  |  |  | "Selected expert is currently offline for immediate consultation." |  |  |  |  |  |  |  |  | O |  |  |  |  |
|  |  |  | "Unsupported payment method: 99." |  |  |  |  |  |  |  |  |  | O |  |  |  |
|  | Log message |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  |  |  | "Operation successful" |  | O | O |  |  |  |  |  |  |  |  |  |  |
| Result | Type(N : Normal, A : Abnormal, B : Boundary) |  |  |  | N | N | A | A | A | A | A | A | B |  |  |  |
|  | Passed/Failed |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Executed Date |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|  | Defect ID |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |

