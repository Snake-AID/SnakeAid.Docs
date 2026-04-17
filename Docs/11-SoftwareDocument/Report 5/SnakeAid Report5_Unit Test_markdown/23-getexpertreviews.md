# 23 - GetExpertReviews

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `GetExpertReviews`
- Used range: `A2:N43`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name|||||GetExpertReviews||||
|Created By||KhiemNVD|||Executed By|||||||||
|Lines  of code||28|||Lack of test cases|||||0||||
|Test requirement||View consultation reviews of one expert||||||||||||
|Passed||Failed|||Untested|||||N/A/B||Total Test Cases||
|0||0|||4|||||0|2|2|4|

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04||||||
|Condition|Precondition|||||||||||||
||||Can connect with server||O|O|O|O||||||
||||Feedback data exists||O||O|O||||||
||pageNumber||||1|1|0|1||||||
||pageSize||||10|10|10|0||||||
|Confirm|Return|||||||||||||
||||HTTP 200||O|O||||||||
||||HTTP 422||||O|O||||||
||||size > 0||O|||||||||
||||size = 0|||O||||||||
||||type = Consultation||O|||||||||
||||"The field PageNumber must be between 1 and 2147483647."||||O|||||||
||||"The field PageSize must be between 1 and 100."|||||O||||||
||Log message|||||||||||||
||||"Operation successful"||O|O||||||||
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|N|B|B||||||
||Passed/Failed|||||||||||||
||Executed Date|||||||||||||
||Defect ID|||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
|||||||||||||||
