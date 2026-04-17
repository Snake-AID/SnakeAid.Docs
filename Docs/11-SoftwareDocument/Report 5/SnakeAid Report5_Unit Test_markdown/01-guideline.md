# 01 - Guideline

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `Guideline`
- Used range: `A1:A47`

|Row|A|
|---|---|
|1|Guideline to make and understand Unit Test Case|
|2||
|3|1. Overview|
|4|- In the template, Unit test cases are based on functions. Each sheet presents test cases for one function.|
|5|- Cover: General information of the project and Unit Test cases|
|6|- FunctionList: The list of Classes and Functions in the document. <br>     + To control that the number of Unit TC meets customer's requirement or the norm, user should fill value for  <br>     'Normal number of Test cases/KLOC'.|
|7|+ Click on Function link to open the related Test cases of the function.  <br>     Note: You should create new Function sheet before creating the link|
|8|- Test Report: provive the overview results of Functions Unit test: Test coverage, Test successful coverage <br>    (Summary, for normal/abnormal/boundary cases)|
|9|Note:  Should check the formula of "Sub Total" if you add more functions|
|10||
|11|2. Content in Test function sheet|
|12|2.1 Combination of test cases.|
|13|- To verify that number of Unit TC meets customer's requirement or not. User has to fill number LOC of tested function and fill value of 'Normal number test cases/KLOC' item in FunctionList sheet, which is required by customer or normal value. The number of lacked TC is shown in 'Lack of test cases' item.|
|14|- If the number of Unit TC does not meet the requirement, creator should explain the reasons.|
|15|- If the number of  'Normal number test cases/KLOC' item in FunctionList sheet is not recorded, the number in 'Lack of test cases' is not calculated.|
|16||
|17|2.2 Condition and confirmation of Test cases.|
|18|Each test case is the combination of condition and confirmation.|
|19|a. Condition:|
|20|- Condition is combination of precondition and values of inputs.|
|21|- Precondition: it is setting condition that must exist before execution of the test case. <br>                    Example: file A is precondition for the test case that needs to access file A.|
|22|- Values of inputs: it includes 3 types of values: normal, boundary and abnormal.|
|23|. Normal values are values of inputs used mainly and usually to ensure the function works.|
|24|. Boundary values are limited values that contain upper and lower values.|
|25|. Abnormal values are non-expected values. And normally it processes exception cases.|
|26|- For examples:|
|27|Input value belongs to 5<= input <=10.|
|28|. 6,7,8,9 are normal values.|
|29|. 5, 10 are boundary values.|
|30|. -1, 11,... are abnormal values.|
|31|b. Confirmation:|
|32|- It is combination of expected result to check output of each function. <br>          If the results are the same with confirmation, the test case is passed, other case it is failed.|
|33|- Confirmation can include:|
|34|+ Output result of the function.|
|35|+ Output log messages in log file.|
|36|+ Output screen message...|
|37|c. Type of test cases and result:|
|38|- Type of test case: It includes normal, boundary and abnormal test cases. User selects the type based on the type of input data.|
|39|- Test case result: the actual output results comparing with the Confirmation.<br>                 P for Passed and F for Failed cases.<br>          It can 'OK' or 'NG' (it depends on habit of the teams or customers)|
|40||
|41|2.3. Other items:|
|42|- Function Code: it is ID of the function and updated automatically according to FunctionList sheet.|
|43|- Function Name: it is name  of the function and updated automatically according to FunctionList sheet.|
|44|- Created By: Name of creator.|
|45|- Executed By: Name of person who executes the unit test|
|46|- Lines of code: Number of Code line of the function.|
|47|- Test requirement: Brief description about requirements which are tested in this function, it is not mandatory.|
