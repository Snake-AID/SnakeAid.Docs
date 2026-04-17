# 13 - GetActiveIncidents

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetActiveIncidents`
- Used range: `A2:O40`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||CreateSnakeCatchingRequest||||
|Created By||NhanNP|||Executed By||||||||||
|Lines  of code||100.0|||Lack of test cases||||||1||||
|Test requirement||<Brief description about requirements which are tested in this function>|||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|
|9||0|||0||||||2|7|0|9|

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|UTCID09||
|Condition|Precondition||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O|O||
||||Logged in as Operator/Admin||O|O|O|O||O|O|O|O||
||status||||||||||||||
||||null (default statuses)||O|O||O|O|O|O||O||
||||Valid status list||||O||||||||
||||"abc"|||||||||O|||
||page||||||||||||||
||||Valid (>= 1)||O|O|O|O|O|O||O|O||
||||Invalid (0 / negative)||||||||O||||
||pageSize||||||||||||||
||||Valid(eg. 5, 10)||O|O|O|O|O|O|O|O|||
||||Invalid (0/negative)||||||||||O||
||since / until||||||||||||||
||||Null||O||O||O||O|O|O||
||||Valid range (eg. 15/4/2026 - 16/4/2026)|||O||O|||||||
||||since -> until (eg. 16/4/2026 - 15/4/2026)|||||||O|||||
|Confirm|Return||||||||||||||
||||HTTP 200||O|O|O|||O|O|O|O||
||||HTTP 401|||||O|O||||||
||||HTTP 422||||||||||||
||Exception||||||||||||||
||||||||||||||||
||Log message||||||||||||||
||||"User incidents retrieved"||O|O|O|||O|O|O|O||
||||"Validate failed||||||O||||||
||||"Unauthorized|||||O|||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|A|A|A|A|A|A|A||
||Passed/Failed||||P|P|P|P|P|P|P|P|P||
||Executed Date||||2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00||
||Defect ID||||||||||||||
