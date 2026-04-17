# 03 - Functions

- Sheet: `Functions`
- Used range: `A2:H18`

| Row | A | B | C | D | E | F | G | H |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 2 |  |  |  |  | Function List |  |  |  |
| 3 |  |  |  |  |  |  |  |  |
| 4 | Project Name |  |  |  | <Project Name> |  |  |  |
| 5 | Project Code |  |  |  | <Project Code> |  |  |  |
| 6 | Normal number of Test cases/KLOC |  |  |  | 100.0 |  |  |  |
| 7 | Test Environment Setup Description |  |  |  | <List enviroment requires in this system<br>1. Server<br>2. Database<br>3. Web Browser<br>...<br>> |  |  |  |
| 8 |  |  |  |  |  |  |  |  |
| 9 |  |  |  |  |  |  |  |  |
| 10 | No | Requirement<br>Name | Class Name | Function Name | Function Code(Optional) | Sheet Name | Description | Pre-Condition |
| 11 | 1.0 |  | Class1 | Function A | Function1 | Function1 |  |  |
| 12 | 2.0 |  | Class2 | Function B | Function2 | Function2 |  |  |
| 13 | 3.0 |  | Class3 | Function C | Function3 | Function3 |  |  |
| 14 |  | REQ-CREATESNAKECATCHINGREQUEST-001 | SnakeCatchingRequest | CreateSnakeCatchingRequest |  | CreateSnakeCatchingRequest | Member create a snake catching request to the center | - Can connect with server<br>- Can connect with AI server<br>- Logged in as Member |
| 15 |  | REQ-GETALLSNAKECATCHINGREQUESTS-001 | SnakeCatchingRequest | GetAllSnakeCatchingRequests |  | GetAllSnakeCatchingRequests | User can view list of snake catching requests | - Can connect with server<br>- Logged in as Member/Operator/Rescuer/Admin |
| 16 |  | REQ-GETSNAKECATCHINGREQUESTDETAIL-001 | SnakeCatchingRequest | GetSnakeCatchingRequestDetail |  | GetSnakeCatchingRequestDetail | User can view snake catching request detail | - Can connect with server<br>- Logged in as Member/Operator/Rescuer/Admin |
| 17 |  | REQ-CONFIRMSNAKECATCHINGREQUEST-001 | SnakeCatchingRequest | ConfirmSnakeCatchingRequest |  | ConfirmSnakeCatchingRequest | Operator can confirm snake catching request | - Can connect with server<br>- Logged in as Operator<br>- Request is in Pending state |
| 18 |  |  | SnakeCatchingRequest | AssignSnakeCatchingRequest |  | AssignSnakeCatchingRequest | Operator can assign rescuer to the snake catching <br>request | - Can connect with server<br>- Logged in as Operator<br>- Request is in Confirmed state<br>- Request is paid up-front fee by Member |
