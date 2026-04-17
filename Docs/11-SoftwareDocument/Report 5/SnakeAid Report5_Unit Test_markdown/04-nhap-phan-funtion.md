# 04 - nháp phần funtion

- Sheet: `nháp phần funtion`
- Used range: `A2:H35`

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
| 14 |  | REQ-INCIDENT-CREATION-001 | SnakebiteIncident | CreateSnakebiteIncident | FC01 | CreateSnakebiteIncident | Member create a snakebite incident emergency to the center | - Can connect with server<br>- Logged in as Member |
| 15 |  | REQ-INCIDENT-DETAIL-002 | SnakebiteIncident | GetIncidentDetail | FC02 | GetIncidentDetail | View detail snakebite incident | - Can connect with server<br>- Logged in as Member<br>- SnakebiteIncident exist |
| 16 |  | REQ-INCIDENT-SYMPTOM-UPDATE-003 | SnakebiteIncident | UpdateSymptomReport | FC03 | UpdateSymptomReport | Update report symptom and serverity assertment | - Can connect with server<br>- Logged in as Member<br>- SnakebiteIncident exist<br>- Valid UpdateSymptomRequest |
| 17 |  | REQ-INCIDENT-CANCEL-004 | SnakebiteIncident | CancelIncident | FC04 | CancelSnakebiteIncident | Member cancel a Snakebite Incident | - Can connect with server<br>- Logged in as Member<br>- SnakebiteIncident exist<br>- SnakebiteIncident in valid status |
| 18 |  | REQ-INCIDENT-IDENTIFY-AI-005 | SnakebiteIncident | IdentifySnakeByAI | FC05 | IdentifySnakeByAI | Detect snake species via AI Recognition | - Can connect with server<br>- Logged in as Member<br>- SnakebiteIncident exist<br>- RecongnitionResultId exist and valid |
| 19 |  | REQ-INCIDENT-IDENTIFY-MANUAL-006 | SnakebiteIncident | IdentifySnakeManual | FC06 | IdentifySnakeManual | Detect snake species manual | - Can connect with server<br>- Logged in as Member<br>- SnakebiteIncident exist<br>- ConfirmSnakeByFilter exist and valid |
| 20 |  | REQ-INCIDENT-USER-LIST-007 | SnakebiteIncident | GetUserIncidents | FC07 | GetUserIncidents | Get Member SnakeBite Incident (paging and filter status) | - Can connect with server<br>- Logged in as Member |
| 21 |  | REQ-INCIDENT-ACTIVE-LIST-008 | SnakebiteIncident | GetActiveIncidents | FC08 |  | Operator/Admin get list active Snakebite Incident for map/dashboard | - Can connect with server<br>- Logged in as Operator/Admin |
| 22 |  | REQ-INCIDENT-ADMIN-LIST-009 | SnakebiteIncident | GetAdminIncidentList | FC09 |  | Admin get list of Snakebite Incident with filter and paging | - Can connect with server<br>- Logged in as Admin<br>- Valid query param |
| 23 |  | REQ-INCIDENT-ADMIN-DETAIL-010 | SnakebiteIncident | GetAdminIncidentDetail | FC10 |  | Admin get detail information Sankebite Incident (Mission, Dispatch Request) | - Can connect with server<br>- Logged in as Admin<br>- SnakebiteIncident exist |
| 24 |  | REQ-INCIDENT-CONFIRM-011 | SnakebiteIncident | ConfirmIncident | FC11 |  | Operator confirm Snakebite Incident after contact with member | - Can connect with server<br>- Logged in as Operator<br>- SnakebiteIncident exist |
| 25 |  | REQ-INCIDENT-FALSE-ALARM-012 | SnakebiteIncident | MarkFalseAlarm | FC12 |  | Operator make the Snakebite Incident to False Alarn | - Can connect with server<br>- Logged in as Operator<br>- SnakebiteIncident exist |
| 26 |  | REQ-INCIDENT-DISPATCH-013 | SnakebiteIncident | DispatchIncidentRequest | FC13 |  | Operator assign Incident rescue request to Rescuer | - Can connect with server<br>- Logged in as Operator<br>- Valid RescuerId |
| 27 |  | REQ-INCIDENT-DISPATCH-CANCEL-014 | SnakebiteIncident | CancelDispatchRequest | FC14 |  | Operator cancel a pending dispatch request | - Can connect with server<br>- Logged in as Operator<br>- Request status is Pending |
| 28 |  | REQ-INCIDENT-DISPATCH-LIST-015 | SnakebiteIncident | GetIncidentDispatchRequest | FC15 |  | Operator/Admin get list dispatch request for IncidentId | - Can connect with server<br>- Logged in as Operator/Operator<br>- SnakebiteIncident exist |
| 29 |  | REQ-MISSION-DETAIL-001 | RescueMission | GetMissionDetails | FC16 |  | Rescuer get detail information about mission | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist |
| 30 |  | REQ-MISSION-ADMIN-LIST-002 | RescueMission | GetAdminMissionList | FC17 |  | Admin get mission list with filter and paging | - Can connect with server<br>- Logged in as Admin<br>- Valid query param |
| 31 |  | REQ-MISSION-START-003 | RescueMission | StartMission | FC18 |  | Rescuer start the mission (Preparing to Enroute) | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist<br>- Rescuer is handling Rescue Mission<br>- Mission status is Pending |
| 32 |  | REQ-MISSION-ARRIVE-004 | RescueMission | ArriveAtLocation | FC19 |  | Rescuer mark the mission status from Enroute to Arrived | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist<br>- Rescuer is handling Rescue Mission<br>- Mission status is Enroute |
| 33 |  | REQ-MISSION-COMPLETE-005 | RescueMission | CompleteMission | FC20 |  | Rescuer complete the mission with envidence media, Incident status is Finished | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist<br>- Rescuer is handling Rescue Mission<br>- Mission status is Arrived<br>- asleast 1 EvidenceReportMediaId<br>- Report Media is valid |
| 34 |  | REQ-MISSION-ABORT-006 | RescueMission | AbortMission | FC21 |  | Rescuer abort mission, Incident status reset to Verified for redispatch | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist<br>- Rescuer is handling Rescue Mission<br>- Mission status is Preparing/Enroute |
| 35 |  | REQ-MISSION-HOSPITAL-TRANSFER-007 | RescueMission | TransferToHospital | FC22 |  | Rescuer mark need tranfer victim to a choosen hosptial | - Can connect with server<br>- Logged in as Rescuer<br>- Rescue Mission exist<br>- Rescuer is handling Rescue Mission |
