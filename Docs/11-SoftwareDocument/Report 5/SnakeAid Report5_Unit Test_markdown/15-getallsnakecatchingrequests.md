# 15 - GetAllSnakeCatchingRequests

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetAllSnakeCatchingRequests`
- Used range: `A2:O45`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||CreateSnakeCatchingRequest||||
|Created By||NhanNP|||Executed By||||||||||
|Lines  of code||100.0|||Lack of test cases||||||0||||
|Test requirement||<Brief description about requirements which are tested in this function>|||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|
|10||0|||0||||||4|6|0|10|

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|UTCID09|UTCID10|
|Condition|Precondition||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O||O|O|
||||Logged in as Member/Operator/Rescuer/Admin||O|O|O|O|O|O|O|O|||
||UserId||||||||||||||
||||Valid Id||O|||O|O|O|O|O|O|O|
||||Invalid Id|||O|||||||||
||||null||||O||||||||
||AssignedRescuerId||||||||||||||
||||Valid Id||O|O|O|||O|O|O|O|O|
||||Invalid Id|||||O|||||||
||||null||||||O||||||
||Status||||||||||||||
||||Valid status||O|O|O|O|O|||O|O|O|
||||Invalid status|||||||O|||||
||||null||||||||O||||
|Confirm|Return||||||||||||||
||||HTTP 200||O||O||O||O||||
||||HTTP 400|||||||O|||||
||||HTTP 404|||O||O|||||||
||||HTTP 500|||||||||O|||
||||HTTP 401||||||||||O||
||||HTTP 403|||||||||||O|
||Exception||||||||||||||
||||||||||||||||
||Log message||||||||||||||
||||"Retrieved {count} snake catching request(s) successfully."||O||O||O||O||||
||||"User with ID {UserId} not found."|||O|||||||||
||||"Rescuer with ID {AssignedRescuerId} not found."|||||O|||||||
||||"Invalid status."|||||||O|||||
||||"Internal server error."|||||||||O|||
||||"Access denied. You don't have permission to access this resource."|||||||||||O|
||||"Authentication required."||||||||||O||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|A|A|A|A|A|A|N|N|N|
||Passed/Failed||||P|P|P|P|P|P|P|P|P|P|
||Executed Date||||2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|
||Defect ID||||||||||||||
