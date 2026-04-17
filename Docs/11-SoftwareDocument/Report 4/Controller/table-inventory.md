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
| 11 | CommunityReports | `id` | `UserId` | No | Verified in current code as community-submitted reports with location, notes, optional species, and media. |
| 12 | ConsultationBookings | `id` | `UserId`, `ExpertId`, `ConsultationId`, `TimeSlotId` | No | Scheduled consultation booking and payment-state bridge. |
| 13 | Consultations | `id` | `CallerId`, `CalleeId`, `ExpertTimeSlotId` | No | Core consultation session table for scheduled and emergency calls. |
| 14 | ExpertCertificates | `id` | None | No | Verified in current code as expert qualification certificates; `ExpertId` exists even though the ERD does not mark it as an FK. |
| 15 | ExpertProfiles | `AccountId` | `AccountId` | No | One-to-one expert profile extension of account data. |
| 16 | ExpertSpecializations | `id` | `ExpertId`, `SpecializationId` | No | Junction table between experts and specialization categories. |
| 17 | ExpertTimeSlots | `id` | `ExpertId` | No | Expert availability slots for scheduled consultation. |
| 18 | FirstAidGuidelines | `Id` | None | No | Reference table for first-aid guidance content. |
| 19 | GeographicRegion | `Id` | None | No | Geographic region master data. |
| 20 | Lessons | `id` | None | No | Verified in current code as educational lesson content with category and publication state. |
| 21 | LibraryMedias | `Id` | `SnakeSpeciesId`, `UploadedById` | No | Snake library media assets and metadata. |
| 22 | LocationEvents | `Id` | None | No | Verified in current code as event-level location history for a session and account role. |
| 23 | MemberProfiles | `AccountId` | `AccountId` | No | One-to-one member profile extension of account data. |
| 24 | PaymentCards | `id` | None | No | Verified in current code as payment card storage; ownership is still not modeled by FK, so wording should remain conservative. |
| 25 | RegionSnakeMapping | `Id` | `SnakeSpeciesId`, `GeographicRegionId` | No | Distribution mapping between species and geographic regions. |
| 26 | ReportMedias | `Id` | None | No | Verified in current code as a polymorphic media attachment table using `ReferenceId` and `ReferenceType`. |
| 27 | RescueMissions | `Id` | `RescuerId`, `IncidentId` | No | Rescue mission execution records for snakebite incidents. |
| 28 | RescueRequestSessions | `Id` | `IncidentId` | No | Dispatch-session table for repeated rescuer search attempts. |
| 29 | RescuerProfiles | `AccountId` | `AccountId` | No | One-to-one rescuer profile extension of account data. |
| 30 | RescuerRequests | `Id` | `SessionId`, `IncidentId`, `RescuerId` | No | Individual rescuer offer/request records inside a rescue session. |
| 31 | ShiftAssignment | `id` | `RescuerId`, `ShiftId` | No | Rescuer-to-shift assignment records. |
| 32 | SnakeAIRecognitionResults | `Id` | `ReportMediaId`, `AIModelId`, `DetectedSpeciesId`, `ExpertCorrectedSpeciesId`, `ExpertId`, `SnakeCatchingMissionId` | No | Verified in current code as AI recognition output linked to report media with expert review/correction data. |
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
| 44 | TrackingSessions | `Id` | None | No | Verified in current code as current live-tracking state keyed by polymorphic `SessionId` and `SessionType`. |
| 45 | Transactions | `id` | `UserId` | No | Verified in current code as the monetary transaction ledger with optional `UserId` and logical `ReferenceId`. |
| 46 | TreatementFacilities | `Id` | None | No | Treatment facility master data. |
| 47 | UserFeedbacks | `id` | `RaterId`, `TargetUserId` | No | Verified in current code as ratings/comments between users tied to a business context by `ReferenceId` and `Type`. |
| 48 | VenomTypes | `Id` | `FirstAidGuidelineId` | No | Venom type master data linked to first-aid guidance. |
| 49 | Wallets | `id` | `UserId` | No | Per-user wallet balance table. |
| 50 | WalletWithdraws | `id` | `UserId`, `WalletId` | No | Withdrawal request records for user wallets. |
| 51 | WorkShift | `id` | None | No | Shift-definition master data for rescuer scheduling. |

## Verification Status

The originally flagged tables have now been verified against current backend code. See `Controller/verification-notes.md` for the supporting notes used to lock the final wording.
