# 25 - UpdateExpertSettings

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `UpdateExpertSettings`
- Used range: `A2:R45`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||UpdateExpertSettings|||||||
|Created By||KhiemNVD|||Executed By|||||||||||||
|Lines  of code||53|||Lack of test cases||||||0|||||||
|Test requirement||Expert updates biography and fees||||||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases||||
|0||0|||8||||||0|6|2|8||||

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|R|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08||||||
|Condition|Precondition|||||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O||||||
||||Logged in as Expert||O|O|O|O|O|||O||||||
||||Expert profile exists||O|O|O|O|O|O|O|||||||
||biography||||"bio"|"bio"|null|"bio"|"bio"|"bio"|"bio"|"bio"||||||
||scheduledConsultationFee||||150000|150000|150000|null|-1|150000|150000|150000||||||
||emergencyConsultationFee||||250000|null|250000|250000|250000|250000|250000|250000||||||
||role||||Expert|Expert|Expert|Expert|Expert|public|User|Expert||||||
|Confirm|Return|||||||||||||||||
||||HTTP 200||O|O||||||||||||
||||HTTP 401|||||||O||||||||
||||HTTP 403||||||||O|||||||
||||HTTP 404|||||||||O||||||
||||HTTP 422||||O|O|O|||||||||
||||emergencyFee = scheduledFee|||O||||||||||||
||||"ScheduledConsultationFee is required."|||||O||||||||||
||||"The field Biography is required."||||O|||||||||||
||||"The field ScheduledConsultationFee must be between 0 and 999999.99."||||||O|||||||||
||Log message|||||||||||||||||
||||"Settings updated successfully"||O|O||||||||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|A|A|B|A|A|A||||||
||Passed/Failed|||||||||||||||||
||Executed Date|||||||||||||||||
||Defect ID|||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
|||||||||||||||||||
