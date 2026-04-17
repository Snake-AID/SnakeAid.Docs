**CAPSTONE PROJECT REPORT**

**Report 4 – Software Design Document**

– Hanoi, August 2019 –

**Table of Contents**
[I. Record of Changes	3](#i.-record-of-changes)

[II. Software Design Document	4](#ii.-software-design-document)

[1\. System Design	4](#1.-system-design)

[1.1 System Architecture	4](#1.1-system-architecture)

[1.2 Package Diagram	4](#1.2-package-diagram)

[2\. Database Design	4](#2.-database-design)

[3\. Detailed Design	5](#3.-detailed-design)

[3.1 \<Feature/Function Name1\>	5](#3.1-\<feature/function-name1\>)

[3.2 \<Feature/Function Name2\>	6](#3.2-snake-capture)

[3.3 Consultation	7](#3.3-consultation)

# **I. Record of Changes**

| Date | A\*
M, D | In charge | Change Description |
| ---- | -------- | --------- | ------------------ |
|      |          |           |                    |

\*A \- Added M \- Modified D \- Deleted

# **II. Software Design Document**

## **1\. System Design**

### **1.1 System Architecture**

### **1.2 Package Diagram**

## **2\. Database Design**

*Rendered ERD reference: `Docs\11-SoftwareDocument\Report 4\Database Design\SnakeAid Physical ERD.mermaid.md`.*

*The following table descriptions summarize the purpose, primary keys, and main foreign-key relationships of the physical database design.*

***Table Descriptions***

| No | Table | Description |
| :-- | :-- | :-- |
| 01 | Accounts | Stores the main user account, authentication, and role information used across the platform. Primary key: `id`. Foreign keys: none. |
| 02 | AIModels | Stores metadata about deployed AI model versions used for snake recognition. Primary key: `Id`. Foreign keys: none. |
| 03 | AISnakeClassMappings | Maps AI model output classes to snake species so raw AI detections can be interpreted by the application. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`, `AIModelId`. |
| 04 | Antivenoms | Stores antivenom master data and links each antivenom to its treatment facility. Primary key: `Id`. Foreign keys: `TreatmentFacilityId`. |
| 05 | AppNotifications | Stores in-app notifications sent to users. Primary key: `id`. Foreign keys: `UserId`. |
| 06 | Blogs | Stores blog posts or educational articles authored by experts. Primary key: `id`. Foreign keys: `AuthorId`. |
| 07 | CatchingEnvironment | Stores reference categories describing the environment in which a snake catching mission takes place. Primary key: `Id`. Foreign keys: none. |
| 08 | CatchingMissionDetails | Stores species and quantity detail lines recorded under a snake catching mission. Primary key: `Id`. Foreign keys: `SnakeCatchingMissionId`, `SnakeSpeciesId`. |
| 09 | CatchingRequestDetails | Stores species and quantity detail lines recorded under a snake catching request. Primary key: `Id`. Foreign keys: `SnakeCatchingRequestId`, `SnakeSpeciesId`. |
| 10 | ChatMessages | Stores chat messages exchanged during a consultation session. Primary key: `id`. Foreign keys: `ConsultationId`, `SenderId`. |
| 11 | CommunityReports | Stores community-submitted snake sighting or incident reports with location, notes, optional identified species, and attached media. Primary key: `id`. Foreign keys: `UserId`. |
| 12 | ConsultationBookings | Stores scheduled consultation bookings, including user, expert, slot, consultation link, pricing, and payment-related status. Primary key: `id`. Foreign keys: `UserId`, `ExpertId`, `ConsultationId`, `TimeSlotId`. |
| 13 | Consultations | Stores core consultation session records for scheduled and emergency consultations, including participants, room, timing, type, and status. Primary key: `id`. Foreign keys: `CallerId`, `CalleeId`, `ExpertTimeSlotId`. |
| 14 | ExpertCertificates | Stores certificate and verification records submitted by experts to prove qualifications. Primary key: `id`. Foreign keys: `ExpertId` in current code, though it is not marked as an FK in the ERD. |
| 15 | ExpertProfiles | Stores expert-specific profile information such as biography, online status, fee, and rating metrics. Primary key: `AccountId`. Foreign keys: `AccountId`. |
| 16 | ExpertSpecializations | Stores the many-to-many relationship between experts and their specializations. Primary key: `id`. Foreign keys: `ExpertId`, `SpecializationId`. |
| 17 | ExpertTimeSlots | Stores expert availability slots used for scheduled consultation booking. Primary key: `id`. Foreign keys: `ExpertId`. |
| 18 | FirstAidGuidelines | Stores first-aid guideline content used by the venom and snake knowledge domain. Primary key: `Id`. Foreign keys: none. |
| 19 | GeographicRegion | Stores geographic region master data used for distribution and mapping features. Primary key: `Id`. Foreign keys: none. |
| 20 | Lessons | Stores educational lesson content with category and publication status for the learning feature. Primary key: `id`. Foreign keys: none. |
| 21 | LibraryMedias | Stores media assets associated with snake species in the snake library. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`, `UploadedById`. |
| 22 | LocationEvents | Stores time-stamped location tracking events for a session, including the account, role, coordinates, speed, and heading. Primary key: `Id`. Foreign keys: no explicit FK is modeled in the ERD, but current code uses `SessionId` and `AccountId` as logical references. |
| 23 | MemberProfiles | Stores member-specific profile information such as rating, emergency contacts, and health indicators. Primary key: `AccountId`. Foreign keys: `AccountId`. |
| 24 | PaymentCards | Stores payment card details and default-card status for payment-related features. Primary key: `id`. Foreign keys: none explicitly modeled in the ERD or current entity. |
| 25 | RegionSnakeMapping | Stores the relationship between snake species and geographic regions, including distribution priority and commonness. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`, `GeographicRegionId`. |
| 26 | ReportMedias | Stores uploaded media files attached to multiple business entities through a polymorphic `ReferenceId` and `ReferenceType` pattern, with processing and sequencing metadata. Primary key: `Id`. Foreign keys: no explicit FK is modeled in the final design because parent linkage is polymorphic. |
| 27 | RescueMissions | Stores rescue mission execution records created to handle snakebite incidents. Primary key: `Id`. Foreign keys: `RescuerId`, `IncidentId`. |
| 28 | RescueRequestSessions | Stores dispatch sessions that group rescuer matching attempts for a snakebite incident. Primary key: `Id`. Foreign keys: `IncidentId`. |
| 29 | RescuerProfiles | Stores rescuer-specific profile information such as online status, rating, location, and mission counters. Primary key: `AccountId`. Foreign keys: `AccountId`. |
| 30 | RescuerRequests | Stores the request and response record for each rescuer contacted during a rescue dispatch session. Primary key: `Id`. Foreign keys: `SessionId`, `IncidentId`, `RescuerId`. |
| 31 | ShiftAssignment | Stores the assignment of rescuers to work shifts together with check-in, check-out, and activity status. Primary key: `id`. Foreign keys: `RescuerId`, `ShiftId`. |
| 32 | SnakeAIRecognitionResults | Stores AI snake recognition outputs for uploaded report media, including mapped species, confidence, review status, and expert correction data. Primary key: `Id`. Foreign keys: `ReportMediaId`, `AIModelId`, `DetectedSpeciesId`, `ExpertCorrectedSpeciesId`, `ExpertId`, `SnakeCatchingMissionId`. |
| 33 | SnakebiteIncidents | Stores snakebite emergency incident records, including reporter, assigned rescuer, location, symptoms, AI/species identification, and dispatch status. Primary key: `Id`. Foreign keys: `UserId`, `AssignedRescuerId`, `AIRecognitionResultId`, `IdentifiedSnakeSpeciesId`. |
| 34 | SnakeCatchingMissions | Stores snake catching mission execution records that fulfill snake catching requests. Primary key: `Id`. Foreign keys: `RescuerId`, `SnakeCatchingRequestId`, `CatchingEnvironmentId`. |
| 35 | SnakeCatchingRequests | Stores snake catching requests created by users, including location, priority, assignment, timing, and pricing information. Primary key: `Id`. Foreign keys: `UserId`, `AssignedRescuerId`. |
| 36 | SnakeSpecies | Stores the main snake species master data used by the library, AI mapping, identification, and medical guidance features. Primary key: `Id`. Foreign keys: none. |
| 37 | SnakeSpeciesNames | Stores alternative names and slugs for a snake species to support lookup and multilingual naming. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`. |
| 38 | SnakeTariffs | Stores pricing rules for snake catching based on species and size category. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`. |
| 39 | Specializations | Stores the master list of expert specializations. Primary key: `id`. Foreign keys: none. |
| 40 | SpeciesAntivenoms | Stores the many-to-many relationship between snake species and antivenoms. Primary key: `Id`. Foreign keys: `SnakeSpeciesId`, `AntivenomId`. |
| 41 | SpeciesVenoms | Stores the many-to-many relationship between snake species and venom types. Primary key: `SnakeSpeciesId`, `VenomTypeId`. Foreign keys: `SnakeSpeciesId`, `VenomTypeId`. |
| 42 | SymptomConfigs | Stores symptom configuration, scoring, alerts, and display metadata for each venom type. Primary key: `Id`. Foreign keys: `VenomTypeId`. |
| 43 | SystemSettings | Stores configurable system settings as key-value pairs with type metadata. Primary key: `SettingKey`. Foreign keys: none. |
| 44 | TrackingSessions | Stores the current live-tracking state for a mission or request session, including participant locations, freshness timestamps, and calculated distance/ETA. Primary key: `Id`. Foreign keys: no explicit FK is modeled in the ERD because `SessionId` is used polymorphically with `SessionType`. |
| 45 | Transactions | Stores monetary transaction records for users, including amount, type, payment method, gateway reference, and business reference. Primary key: `id`. Foreign keys: `UserId` is optional in current code; `ReferenceId` is a logical business reference rather than an explicit FK. |
| 46 | TreatementFacilities | Stores treatment facility master data such as name, address, contact, and location. Primary key: `Id`. Foreign keys: none. |
| 47 | UserFeedbacks | Stores ratings and comments created by one user for another user in the context of a referenced business record such as a consultation or mission. Primary key: `id`. Foreign keys: `RaterId`, `TargetUserId`; `ReferenceId` is a logical reference used together with `Type`. |
| 48 | VenomTypes | Stores venom type master data and links each venom type to its first-aid guideline. Primary key: `Id`. Foreign keys: `FirstAidGuidelineId`. |
| 49 | Wallets | Stores each user wallet and its current balance. Primary key: `id`. Foreign keys: `UserId`. |
| 50 | WalletWithdraws | Stores withdrawal requests created against user wallets, including bank information, amount, status, and processing outcome. Primary key: `id`. Foreign keys: `UserId`, `WalletId`. |
| 51 | WorkShift | Stores shift-definition master data, including time range, required rescuers, and active status. Primary key: `id`. Foreign keys: none. |
