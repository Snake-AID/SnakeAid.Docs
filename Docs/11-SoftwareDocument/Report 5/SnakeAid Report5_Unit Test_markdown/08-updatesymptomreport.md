# 08 - UpdateSymptomReport

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `UpdateSymptomReport`
- Used range: `A2:O40`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||CreateSnakeCatchingRequest||||
|Created By||NhanNP|||Executed By||||||||||
|Lines  of code||100.0|||Lack of test cases||||||4||||
|Test requirement||<Brief description about requirements which are tested in this function>|||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|
|5||1|||0||||||3|3|0|6|

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|||||
|Condition|Precondition||||||||||||||
||||Can connect with server||O|O|O|O|O|O|||||
||||Logged in as Member||O|O|O|O||O|||||
||||SnakebiteIncident exist||O||O|O||O|||||
||||||||||||||||
||incidentId||||||||||||||
||||Valid Guid (exist)||O||O|O|O||||||
||||Valid Guid (not exist)|||O|||||||||
||||Invalid format|||||||O|||||
||SymptomIdList||||||||||||||
||||Valid (>= 1 item)||O|O||O|O|O|||||
||||null||||O||||||||
||TimeSinceBiteMinutes||||||||||||||
||||valid (eg 30)||O|O|O||O|O|||||
||||null|||||O|||||||
|Confirm|Return||||||||||||||
||||HTTP 200||O|||O|||||||
||||HTTP 404|||O|||||||||
||||HTTP 422||||O|||O|||||
||||HTTP 401||||||O||||||
||Exception||||||||||||||
||||||||||||||||
||Log message||||||||||||||
||||"Symptom report updated successfully"||O||||||||||
||||"Incident not found"|||O||||O|||||
||||Validation failed||||O|O|||||||
||||"Unauthorized"||||||O||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|A|A|N|A|||||
||Passed/Failed||||P|P|P|P|P|F|||||
||Executed Date||||2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|||||
||Defect ID|||||||||DFID002|||||
