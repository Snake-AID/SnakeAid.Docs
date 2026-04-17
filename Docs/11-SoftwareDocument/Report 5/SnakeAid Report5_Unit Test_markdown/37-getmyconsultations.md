# 37 - GetMyConsultations

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetMyConsultations`
- Used range: `A2:Q45`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||GetMyConsultations||||||
|Created By||KhiemNVD|||Executed By||||||||||||
|Lines  of code||96|||Lack of test cases||||||0||||||
|Test requirement||Member views combined scheduled and emergency consultation history with filter, sort, pagination, and price mapping|||||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|||
|0||0|||8||||||0|5|3|8|||

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|||||
|Condition|Precondition||||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O|||||
||||Consultation history data prepared||O|O|O|O|O|O|O|O|||||
||auth context||||User|User|User|User|User|public|Expert|User|||||
||status||||null|null|"Completed"|null|"invalid"|null|null|null|||||
||type||||null|"Scheduled"|"Emergency"|null|null|null|null|null|||||
||pageNumber||||1|1|1|2|1|1|1|0|||||
||pageSize||||10|10|10|2|10|10|10|10|||||
|Confirm|Return||||||||||||||||
||||HTTP 200||O|O|O|O|||||||||
||||HTTP 400||||||O||||||||
||||HTTP 401|||||||O|||||||
||||HTTP 403||||||||O||||||
||||HTTP 422|||||||||O|||||
||||contains Scheduled + Emergency items||O||||||||||||
||||type = Scheduled only|||O|||||||||||
||||type = Emergency only||||O||||||||||
||||emergency price from transaction or null if missing||O||O||||||||||
||||sorted by StartTime desc||O|O|O|O|||||||||
||||pagination meta correct|||||O|||||||||
||||"Invalid status value: invalid"||||||O||||||||
||||"The field PageNumber must be between 1 and 2147483647."|||||||||O|||||
||Log message||||||||||||||||
||||"Operation successful"||O|O|O|O|||||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|N|B|A|A|A|B|||||
||Passed/Failed||||||||||||||||
||Executed Date||||||||||||||||
||Defect ID||||||||||||||||

