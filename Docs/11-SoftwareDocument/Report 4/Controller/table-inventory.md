# Report 4 ERD Table Inventory

## Purpose

This file is the master inventory extracted from the Report 4 physical ERD before drafting the final `Table Descriptions` section.

## Summary

- Source: `Report 4/Database Design/SnakeAid Physical ERD.mermaid.md`
- Total tables identified: `51`
- This inventory captures table name, primary key, explicit foreign keys from the ERD, and whether business meaning likely needs deeper verification.

## Inventory

| No | Table | Primary Key | Foreign Keys | Needs verification | Notes |
| --- | --- | --- | --- | --- | --- |
| 01 | Accounts | `id` | None | No | Core identity and account table. |
| 02 | AIModels | `Id` | None | No | AI model registry appears clear from ERD naming. |
| 03 | AISnakeClassMappings | `Id` | `SnakeSpeciesId`, `AIModelId` | No | Mapping table between deployed AI models and snake species classes. |
| 04 | Antivenoms | `Id` | `TreatmentFacilityId` | No | Antivenom catalog linked to treatment facilities. |
| 05 | AppNotifications | `id` | `UserId` | No | User-targeted application notifications. |
| 06 | Blogs | `id` | `AuthorId` | No | Blog or article content authored by experts. |
| 07 | CatchingEnvironment | `Id` | None | No | Reference table for snake catching environment categories. |
| 08 | CatchingMissionDetails | `Id` | `SnakeCatchingMissionId`, `SnakeSpeciesId` | No | Child detail table for species and quantity in a catching mission. |
| 09 | CatchingRequestDetails | `Id` | `SnakeCatchingRequestId`, `SnakeSpeciesId` | No | Child detail table for species and quantity in a catching request. |
| 10 | ChatMessages | `id` | `ConsultationId`, `SenderId` | No | Consultation chat message records. |
| 11 | CommunityReports | `id` | `UserId` | Yes | Table name is broad; exact business usage should be verified. |
| 12 | ConsultationBookings | `id` | `UserId`, `ExpertId`, `ConsultationId`, `TimeSlotId` | No | Scheduled consultation booking and payment-state bridge. |
| 13 | Consultations | `id` | `CallerId`, `CalleeId`, `ExpertTimeSlotId` | No | Core consultation session table for scheduled and emergency calls. |
| 14 | ExpertCertificates | `id` | None | Yes | ERD body does not mark `ExpertId` as FK even though the role is obvious. Verify relationship in code/docs. |
| 15 | ExpertProfiles | `AccountId` | `AccountId` | No | One-to-one expert profile extension of account data. |
| 16 | ExpertSpecializations | `id` | `ExpertId`, `SpecializationId` | No | Junction table between experts and specialization categories. |
| 17 | ExpertTimeSlots | `id` | `ExpertId` | No | Expert availability slots for scheduled consultation. |
| 18 | FirstAidGuidelines | `Id` | None | No | Reference table for first-aid guidance content. |
| 19 | GeographicRegion | `Id` | None | No | Geographic region master data. |
| 20 | Lessons | `id` | None | Yes | Standalone content table; exact feature surface should be verified. |
| 21 | LibraryMedias | `Id` | `SnakeSpeciesId`, `UploadedById` | No | Snake library media assets and metadata. |
| 22 | LocationEvents | `Id` | None | Yes | ERD does not declare explicit FK links; tracking purpose is likely but should be confirmed. |
| 23 | MemberProfiles | `AccountId` | `AccountId` | No | One-to-one member profile extension of account data. |
| 24 | PaymentCards | `id` | None | Yes | No explicit FK to account or wallet in the ERD; ownership model needs verification. |
| 25 | RegionSnakeMapping | `Id` | `SnakeSpeciesId`, `GeographicRegionId` | No | Distribution mapping between species and geographic regions. |
| 26 | ReportMedias | `Id` | None | Yes | `ReferenceId` is polymorphic by naming but not modeled as explicit FK in the ERD. |
| 27 | RescueMissions | `Id` | `RescuerId`, `IncidentId` | No | Rescue mission execution records for snakebite incidents. |
| 28 | RescueRequestSessions | `Id` | `IncidentId` | No | Dispatch-session table for repeated rescuer search attempts. |
| 29 | RescuerProfiles | `AccountId` | `AccountId` | No | One-to-one rescuer profile extension of account data. |
| 30 | RescuerRequests | `Id` | `SessionId`, `IncidentId`, `RescuerId` | No | Individual rescuer offer/request records inside a rescue session. |
| 31 | ShiftAssignment | `id` | `RescuerId`, `ShiftId` | No | Rescuer-to-shift assignment records. |
| 32 | SnakeAIRecognitionResults | `Id` | `ReportMediaId`, `AIModelId`, `DetectedSpeciesId`, `ExpertCorrectedSpeciesId`, `ExpertId`, `SnakeCatchingMissionId` | Yes | High-link table is clear overall, but some relationship semantics should be verified during description drafting. |
| 33 | SnakebiteIncidents | `Id` | `UserId`, `AssignedRescuerId`, `AIRecognitionResultId`, `IdentifiedSnakeSpeciesId` | No | Core emergency snakebite incident table. |
| 34 | SnakeCatchingMissions | `Id` | `RescuerId`, `SnakeCatchingRequestId`, `CatchingEnvironmentId` | No | Fulfillment mission table for snake catching requests. |
| 35 | SnakeCatchingRequests | `Id` | `UserId`, `AssignedRescuerId` | No | Core snake catching demand/request table. |
| 36 | SnakeSpecies | `Id` | None | No | Master snake species table. |
| 37 | SnakeSpeciesNames | `Id` | `SnakeSpeciesId` | No | Alternate names and slugs for snake species. |
| 38 | SnakeTariffs | `Id` | `SnakeSpeciesId` | No | Pricing reference table per species and size category. |
| 39 | Specializations | `id` | None | No | Expert specialization master data. |
| 40 | SpeciesAntivenoms | `Id` | `SnakeSpeciesId`, `AntivenomId` | No | Junction table between species and antivenoms. |
| 41 | SpeciesVenoms | `SnakeSpeciesId`, `VenomTypeId` | `SnakeSpeciesId`, `VenomTypeId` | No | Junction table between species and venom types. |
| 42 | SymptomConfigs | `Id` | `VenomTypeId` | No | Symptom and scoring configuration by venom type. |
| 43 | SystemSettings | `SettingKey` | None | No | Key-value style system configuration table. |
| 44 | TrackingSessions | `Id` | None | Yes | Session tracking table looks related to live tracking, but ERD does not expose explicit FK links. |
| 45 | Transactions | `id` | `UserId` | Yes | User transaction ledger is clear at a high level, but polymorphic `ReferenceId` use should be verified. |
| 46 | TreatementFacilities | `Id` | None | No | Treatment facility master data. |
| 47 | UserFeedbacks | `id` | `RaterId`, `TargetUserId` | Yes | `ReferenceId` and `Type` imply polymorphic feedback targets and should be described carefully. |
| 48 | VenomTypes | `Id` | `FirstAidGuidelineId` | No | Venom type master data linked to first-aid guidance. |
| 49 | Wallets | `id` | `UserId` | No | Per-user wallet balance table. |
| 50 | WalletWithdraws | `id` | `UserId`, `WalletId` | No | Withdrawal request records for user wallets. |
| 51 | WorkShift | `id` | None | No | Shift-definition master data for rescuer scheduling. |

## Tables Flagged For Deeper Review

- `CommunityReports`
- `ExpertCertificates`
- `Lessons`
- `LocationEvents`
- `PaymentCards`
- `ReportMedias`
- `SnakeAIRecognitionResults`
- `TrackingSessions`
- `Transactions`
- `UserFeedbacks`

These tables are not blocked. They are simply the ones most likely to need code or supporting-doc verification before the final report wording is locked.
