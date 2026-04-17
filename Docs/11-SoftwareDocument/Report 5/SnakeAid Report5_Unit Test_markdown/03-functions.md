# 03 - Functions

- Source workbook: [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx)
- Sheet: `Functions`
- Used range: `A2:H41`

|Row|A|B|C|D|E|F|G|H|
|---|---|---|---|---|---|---|---|---|
|2|||||Function List||||
|3|||||||||
|4|Project Name||||<Project Name>||||
|5|Project Code||||<Project Code>||||
|6|Normal number of Test cases/KLOC||||100.0||||
|7|Test Environment Setup Description||||<List enviroment requires in this system<br>1. Server<br>2. Database<br>3. Web Browser<br>...<br>>||||
|8|||||||||
|9|||||||||
|10|No|Requirement<br>Name|Class Name|Function Name|Function Code(Optional)|Sheet Name|Description|Pre-Condition|
|11|1.0||Class1|Function A|Function1|Function1|||
|12|2.0||Class2|Function B|Function2|Function2|||
|13|3.0||Class3|Function C|Function3|Function3|||
|14||REQ-CREATESNAKECATCHINGREQUEST-001|SnakeCatchingRequest|CreateSnakeCatchingRequest||CreateSnakeCatchingRequest|Member create a snake catching request to the center|- Can connect with server<br>- Can connect with AI server<br>- Logged in as Member|
|15||REQ-GETALLSNAKECATCHINGREQUESTS-001|SnakeCatchingRequest|GetAllSnakeCatchingRequests||GetAllSnakeCatchingRequests|User can view list of snake catching requests|- Can connect with server<br>- Logged in as Member/Operator/Rescuer/Admin|
|16||REQ-GETSNAKECATCHINGREQUESTDETAIL-001|SnakeCatchingRequest|GetSnakeCatchingRequestDetail||GetSnakeCatchingRequestDetail|User can view snake catching request detail|- Can connect with server<br>- Logged in as Member/Operator/Rescuer/Admin|
|17||REQ-CONFIRMSNAKECATCHINGREQUEST-001|SnakeCatchingRequest|ConfirmSnakeCatchingRequest||ConfirmSnakeCatchingRequest|Operator can confirm snake catching request|- Can connect with server<br>- Logged in as Operator<br>- Request is in Pending state|
|18|||SnakeCatchingRequest|AssignSnakeCatchingRequest||AssignSnakeCatchingRequest|Operator can assign rescuer to the snake catching <br>request|- Can connect with server<br>- Logged in as Operator<br>- Request is in Confirmed state<br>- Request is paid up-front fee by Member|
|19|4.0|REQ-GETEXPERTS-001|Expert|GetExperts||GetExperts|User can browse the expert directory|- Can connect with server|
|20|5.0|REQ-GETEXPERTPROFILE-001|Expert|GetExpertProfile||GetExpertProfile|User can view a specific expert profile|- Can connect with server|
|21|6.0|REQ-GETEXPERTREVIEWS-001|Expert|GetExpertReviews||GetExpertReviews|User can view reviews of a specific expert|- Can connect with server|
|22|7.0|REQ-GETEXPERTTIMESLOTS-001|Expert|GetExpertTimeSlots||GetExpertTimeSlots|User can view available time slots of an expert|- Can connect with server|
|23|8.0|REQ-UPDATEEXPERTSETTINGS-001|Expert|UpdateExpertSettings||UpdateExpertSettings|Expert can update consultation profile settings|- Can connect with server<br>- Logged in as Expert|
|24|9.0|REQ-CREATEBULKTIMESLOTS-001|Expert|CreateBulkTimeSlots||CreateBulkTimeSlots|Expert can generate weekly consultation time slots in bulk|- Can connect with server<br>- Logged in as Expert|
|25|10.0|REQ-GETEXPERTCONSULTATIONS-001|Expert|GetExpertConsultations||GetExpertConsultations|Expert can view their consultation history|- Can connect with server<br>- Logged in as Expert|
|26|11.0|REQ-CREATECONSULTATIONBOOKING-001|ConsultationScheduled|CreateConsultationBooking||CreateConsultationBooking|Member can create a scheduled consultation booking|- Can connect with server<br>- Logged in as User<br>- Target slot is available and in the future|
|27|12.0|REQ-GETMYSCHEDULEDBOOKINGS-001|ConsultationScheduled|GetMyScheduledBookings||GetMyScheduledBookings|Member can view their scheduled consultation bookings|- Can connect with server<br>- Logged in as User|
|28|13.0|REQ-GETEXPERTSCHEDULEDBOOKINGS-001|ConsultationScheduled|GetExpertScheduledBookings||GetExpertScheduledBookings|Expert can view their scheduled consultation bookings|- Can connect with server<br>- Logged in as Expert|
|29|14.0|REQ-PAYSCHEDULEDBOOKING-001|ConsultationPayments|PayScheduledBooking||PayScheduledBooking|Member can pay for a scheduled consultation booking|- Can connect with server<br>- Logged in as User<br>- Booking is in PendingPayment state|
|30|15.0|REQ-CREATEEMERGENCYCONSULTATIONREQUEST-001|ConsultationInstant|CreateEmergencyConsultationRequest||CreateEmergencyConsultationRequest|Member can create an emergency consultation request|- Can connect with server<br>- Logged in as User|
|31|16.0|REQ-PAYEMERGENCYREQUEST-001|ConsultationPayments|PayEmergencyRequest||PayEmergencyRequest|Member can pay for an emergency consultation request|- Can connect with server<br>- Logged in as User<br>- Request is in PendingPayment state|
|32|17.0|REQ-ACCEPTEMERGENCYCONSULTATIONREQUEST-001|ConsultationInstant|AcceptEmergencyConsultationRequest||AcceptEmergencyConsultationRequest|Expert can accept a paid emergency consultation request|- Can connect with server<br>- Logged in as Expert<br>- Request is assigned to current expert<br>- Request is in PendingExpertResponse state|
|33|18.0|REQ-REJECTEMERGENCYCONSULTATIONREQUEST-001|ConsultationInstant|RejectEmergencyConsultationRequest||RejectEmergencyConsultationRequest|Expert can reject a paid emergency consultation request|- Can connect with server<br>- Logged in as Expert<br>- Request is assigned to current expert<br>- Request is in PendingExpertResponse state|
|34|19.0|REQ-CONFIRMCONSULTATIONPAYMENT-001|ConsultationPayments|ConfirmConsultationPayment||ConfirmConsultationPayment|Member can confirm a pending PayOS consultation payment|- Can connect with server<br>- Logged in as User<br>- Transaction exists|
|35|20.0|REQ-GETMYCONSULTATIONS-001|Consultation|GetMyConsultations||GetMyConsultations|Member can view their consultation history|- Can connect with server<br>- Logged in as User|
|36|21.0|REQ-GENERATEVIDEOTOKEN-001|VideoCall|GenerateVideoToken||GenerateVideoToken|Participant or admin can generate a consultation video token|- Can connect with server<br>- Logged in as consultation participant or Admin<br>- Consultation exists|
|37|22.0|REQ-ENDCONSULTATION-001|Consultation|EndConsultation||EndConsultation|Consultation participant can end an active consultation|- Can connect with server<br>- Logged in as consultation participant|
|38|23.0|REQ-CREATECONSULTATIONREVIEW-001|Consultation|CreateConsultationReview||CreateConsultationReview|Member can create a review for a consultation|- Can connect with server<br>- Logged in as User<br>- Consultation exists|
|39|24.0|REQ-GETCONSULTATIONREVIEW-001|Consultation|GetConsultationReview||GetConsultationReview|Authenticated participant can view consultation review data|- Can connect with server<br>- Logged in as consultation participant|
|40|25.0|REQ-GETALLCONSULTATIONS-001|AdminConsultation|GetAllConsultations||GetAllConsultations|Admin can view the consultation list across scheduled and emergency flows|- Can connect with server<br>- Logged in as Admin|
|41|26.0|REQ-GETCONSULTATIONBYID-001|AdminConsultation|GetConsultationById||GetConsultationById|Admin can view consultation detail by ID|- Can connect with server<br>- Logged in as Admin|
