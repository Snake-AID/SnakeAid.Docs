# 31 - PayScheduledBooking

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `PayScheduledBooking`
- Used range: `A2:Q45`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||PayScheduledBooking||||||
|Created By||KhiemNVD|||Executed By||||||||||||
|Lines  of code||115|||Lack of test cases||||||0||||||
|Test requirement||Member pays a scheduled booking by wallet or PayOS with ownership, status, and payment-method validation|||||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|||
|0||0|||9||||||0|3|6|9|||

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|UTCID09||||
|Condition|Precondition||||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O|O||||
||||Scheduled booking data prepared||O|O|O|O|O|O|O|O|O||||
||auth context||||User|User|public|Expert|User|User|User|User|User||||
||booking ownership||||current user|current user|current user|current user|missing booking|other user|current user|current user|current user||||
||booking status||||PendingPayment|PendingPayment|PendingPayment|PendingPayment|n/a|PendingPayment|Confirmed|PendingPayment|PendingPayment||||
||existing payment transaction||||none|none|none|none|n/a|none|none|exists|none||||
||paymentMethod||||WalletBalance|PayOs|WalletBalance|WalletBalance|WalletBalance|WalletBalance|WalletBalance|WalletBalance|99||||
|Confirm|Return||||||||||||||||
||||HTTP 200||O|O|||||||||||
||||HTTP 401||||O||||||||||
||||HTTP 403|||||O||O|||||||
||||HTTP 404||||||O||||||||
||||HTTP 409||||||||O|O|||||
||||HTTP 422||||||||||O||||
||||status = "Escrowed"||O||||||||||||
||||status = "Pending"|||O|||||||||||
||||booking.Status = Confirmed||O||||||||||||
||||booking.Status = PendingPayment|||O|||||||||||
||||provider = "Wallet"||O||||||||||||
||||checkoutUrl != null|||O|||||||||||
||||orderCode != null|||O|||||||||||
||||"Consultation booking was not found."||||||O||||||||
||||"You are not allowed to pay for this booking."|||||||O|||||||
||||"Consultation booking is no longer waiting for payment."||||||||O||||||
||||"Consultation booking has already been paid."|||||||||O|||||
||||"Unsupported payment method: 99."||||||||||O||||
||Log message||||||||||||||||
||||"Operation successful"||O|O|||||||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|A|A|A|A|A|A|B||||
||Passed/Failed||||||||||||||||
||Executed Date||||||||||||||||
||Defect ID||||||||||||||||

