# 06 - CreateSnakebiteIncident

- Sheet: `CreateSnakebiteIncident`
- Used range: `A2:Q46`

**Summary**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
|Function Code||Function1|||Function Name||||||CreateSnakeCatchingRequest||||||
|Created By||NhanNP|||Executed By||||||||||||
|Lines  of code||100.0|||Lack of test cases||||||-2||||||
|Test requirement||<Brief description about requirements which are tested in this function>|||||||||||||||
|Passed||Failed|||Untested||||||N/A/B|||Total Test Cases|||
|10||2|||0||||||2|6|4|12|||

**Matrix**

|A|B|C|D|E|F|G|H|I|J|K|L|M|N|O|P|Q|
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
||||||UTCID01|UTCID02|UTCID03|UTCID04|UTCID05|UTCID06|UTCID07|UTCID08|UTCID09|UTCID11|UTCID12|UTCID13|
|Condition|Precondition||||||||||||||||
||||Can connect with server||O|O|O|O|O|O|O|O|O|O|O|O|
||||Logged in as Member||O|O|O|O|O|O|O|O|O|O|O|O|
||||||||||||||||||
||lng||||||||||||||||
||||106.0||O|||O|O|||O|O|O||O|
||||-181.0|||O|||||||||||
||||181.0||||O||||||||||
||||180.0|||||||O|||||||
||||-180.0||||||||O||||||
||||null||||||||||||O||
||lat||||||||||||||||
||||10.0||O|O|O|||O|O|||O|O||
||||-91.0|||||O|||||||||
||||91.0||||||O||||||||
||||-90.0|||||||||O|||||
||||90.0||||||||||O||||
||||null|||||||||||||O|
||Address||||||||||||||||
||||"HCM"||O|O|O|O|O|O|O|O|O||O|O|
||||""|||||||||||O|||
|Confirm|Return||||||||||||||||
||||HTTP 200||O|||||O|O|O|O|O|||
||||HTTP 422|||O|O|O|O||||||O|O|
||||HTTP 400||||||||||||||
||Exception||||||||||||||||
||||||||||||||||||
||Log message||||||||||||||||
||||"Snakebite incident created"||O|||||O|O|O|O|O|||
||||"The field Lng must be a valid number. (-180 to 180)"|||O|O||||||||||
||||"The field Lat must be a valid number. (-90 to 90)"|||||O|O||||||||
||||The field Lng is Required||||||||||||O||
||||The field Lat is Required|||||||||||||O|
|Result|Type(N : Normal, A : Abnormal, B : Boundary)||||N|A|A|A|A|B|B|B|B|N|A|A|
||Passed/Failed||||P|P|P|P|P|F|F|P|P|P|P|P|
||Executed Date||||2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|2026-04-16 00:00:00|
||Defect ID|||||||||DFID002|DFID004|DFID005|DFID006|DFID007|DFID007|DFID007|
