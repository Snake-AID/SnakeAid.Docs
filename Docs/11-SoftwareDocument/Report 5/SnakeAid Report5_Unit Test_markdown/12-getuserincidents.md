# 12 - GetUserIncidents

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetUserIncidents`
- Used range: `A2:O37`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||CreateSnakeCatchingRequest||||
|Created By||NhanNP|||Executed By||||||||||
|Lines  of code||100.0|||Lack of test cases||||||4||||
|Test requirement||<Brief description about requirements which are tested in this function>|||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|
|6||0|||0||||||1|3|2|6|

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|||||
|Condition|Precondition||||||||||||||
||||Can connect with server||O|O|O|O|O|O|||||
||||Logged in as Member||O|O|O||O|O|||||
||||||||||||||||
||status||||||||||||||
||||Valid status||O||O|O|O||||||
||||Invalid status|||O|||||||||
||||null|||||||O|||||
||page||||||||||||||
||||Valid (>= 1)||O|O||O|O|O|||||
||||Invalid (0 / negative)||||O||||||||
||pageSize||||||||||||||
||||Valid(eg. 5, 10)||O|O|O|O||O|||||
||||Invalid (0/negative)||||||O||||||
|Confirm|Return||||||||||||||
||||HTTP 200||O|||||O|||||
||||HTTP 401|||||O|||||||
||||HTTP 422|||O|O||O||||||
||Exception||||||||||||||
||||||||||||||||
||Log message||||||||||||||
||||"User incidents retrieved"||O|||||O|||||
||||"Validate failed"|||O|O||O||||||
||||"Unauthorized"|||||O|||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|A|B|A|B|A|||||
||Passed/Failed||||P|P|P|P|P|P|||||
||Executed Date||||2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|||||
||Defect ID||||||||||||||
