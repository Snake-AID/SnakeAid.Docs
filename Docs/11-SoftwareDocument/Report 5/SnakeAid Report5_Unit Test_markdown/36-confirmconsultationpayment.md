# 36 - ConfirmConsultationPayment

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `ConfirmConsultationPayment`
- Used range: `A2:Q45`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||ConfirmConsultationPayment||||||
|Created By||KhiemNVD|||Executed By||||||||||||
|Lines  of code||63|||Lack of test cases||||||0||||||
|Test requirement||Member confirms a pending PayOS consultation payment and handles already-confirmed or unconfirmable transactions|||||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|||
|0||0|||8||||||0|2|6|8|||

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|||||
|Condition|Precondition||||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O|||||
||||Consultation payment transaction data prepared||O|O|O|O|O|O|O|O|||||
||auth context||||User|User|public|Expert|User|User|User|User|||||
||transactionId||||pending PayOs tx|already confirmed PayOs tx|pending PayOs tx|pending PayOs tx|missing tx|tx without order code|tx with missing PayOS link|tx with non-paid PayOS status|||||
|Confirm|Return||||||||||||||||
||||HTTP 200||O|O|||||||||||
||||HTTP 401||||O||||||||||
||||HTTP 403|||||O|||||||||
||||HTTP 404||||||O||||||||
||||HTTP 409|||||||O|O|O|||||
||||status = "Escrowed"||O|O|||||||||||
||||booking.Status = Confirmed||O|O|||||||||||
||||externalTransactionId != null||O|O|||||||||||
||||"Consultation payment transaction was not found."||||||O||||||||
||||"Consultation payment order code is missing."|||||||O|||||||
||||"Unable to retrieve PayOS payment information for order code ..."||||||||O||||||
||||"PayOS reports status 'PENDING'. Payment cannot be confirmed."|||||||||O|||||
||Log message||||||||||||||||
||||"Operation successful"||O|O|||||||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|B|A|A|A|A|A|A|||||
||Passed/Failed||||||||||||||||
||Executed Date||||||||||||||||
||Defect ID||||||||||||||||

