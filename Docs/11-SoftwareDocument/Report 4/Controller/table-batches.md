# Report 4 Table Authoring Batches

## Purpose

This file groups all ERD tables into writing batches so the final `Table Descriptions` section can be drafted in controlled domain-sized passes.

## Authoring Order

Recommended writing order:

1. Identity, profiles, notifications, and system tables
2. Consultation, bookings, chat, and expert domain tables
3. Snakebite incident and rescue dispatch tables
4. Snake catching request and mission tables
5. Snake knowledge, antivenom, venom, and region mapping tables
6. AI, media, transaction, wallet, and operational tables

This order starts with the simplest reference and profile tables, then moves into business flows, and leaves the more polymorphic or operational tables for the end.

## Batch 1. Identity, Profiles, Notifications, and System Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| Accounts | Core identity root for multiple profile extensions | Yes |
| AppNotifications | User-facing notification records | No |
| ExpertProfiles | One-to-one expert profile extension | Yes |
| MemberProfiles | One-to-one member profile extension | Yes |
| RescuerProfiles | One-to-one rescuer profile extension | Yes |
| ExpertCertificates | Expert credential records | No |
| ExpertSpecializations | Expert-to-specialization mapping | No |
| Specializations | Specialization master data | No |
| Blogs | Expert-authored article content | No |
| SystemSettings | System-wide key-value settings | No |
| WorkShift | Shift-definition master data | No |
| ShiftAssignment | Rescuer shift assignment records | No |
| CommunityReports | Community-generated reporting records | Yes |
| Lessons | Standalone educational content | No |

## Batch 2. Consultation, Bookings, Chat, and Expert Domain Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| Consultations | Core consultation session entity | Yes |
| ConsultationBookings | Scheduled booking/payment bridge | Yes |
| ChatMessages | Message history within consultations | No |
| ExpertTimeSlots | Expert availability slots | No |
| UserFeedbacks | Review/feedback entity with consultation-related use | Yes |

## Batch 3. Snakebite Incident and Rescue Dispatch Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| SnakebiteIncidents | Core snakebite emergency incident entity | Yes |
| RescueRequestSessions | Dispatch-session entity for incident matching | No |
| RescuerRequests | Individual rescuer offers/requests in a dispatch session | No |
| RescueMissions | Fulfillment mission for emergency rescue | Yes |
| TrackingSessions | Live tracking state between member and rescuer | Yes |
| LocationEvents | Event-level location tracking records | Yes |

## Batch 4. Snake Catching Request and Mission Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| SnakeCatchingRequests | Core snake catching demand entity | Yes |
| CatchingRequestDetails | Species detail lines for a catching request | No |
| SnakeCatchingMissions | Fulfillment mission for a catching request | Yes |
| CatchingMissionDetails | Species detail lines for a catching mission | No |
| CatchingEnvironment | Catching environment reference data | No |
| SnakeTariffs | Price reference data used by catching domain | Yes |

## Batch 5. Snake Knowledge, Antivenom, Venom, and Region Mapping Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| SnakeSpecies | Core snake species master table | Yes |
| SnakeSpeciesNames | Alternate naming for species lookup/display | No |
| VenomTypes | Venom type master data | No |
| SpeciesVenoms | Species-to-venom mapping | No |
| FirstAidGuidelines | First-aid guideline reference content | Yes |
| SymptomConfigs | Symptom configuration by venom type | No |
| Antivenoms | Antivenom master data | No |
| SpeciesAntivenoms | Species-to-antivenom mapping | No |
| TreatementFacilities | Treatment facility master data | No |
| GeographicRegion | Geographic region master data | No |
| RegionSnakeMapping | Species distribution by region | No |
| LibraryMedias | Media assets for snake library content | Yes |

## Batch 6. AI, Media, Transaction, Wallet, and Operational Tables

| Table | Reason for grouping | Cross-domain |
| --- | --- | --- |
| AIModels | AI model registry | No |
| AISnakeClassMappings | Mapping between AI classes and species | No |
| SnakeAIRecognitionResults | AI/expert recognition output records | Yes |
| ReportMedias | Polymorphic media attachment records | Yes |
| PaymentCards | Payment card storage records | Yes |
| Transactions | User transaction ledger | Yes |
| Wallets | Per-user wallet balances | No |
| WalletWithdraws | Wallet withdrawal request records | No |

## Batch Coverage Check

- Total tables assigned: `51`
- Tables assigned to exactly one batch: `51`
- Unassigned tables: `0`

## Tables That Need Extra Care During Drafting

These are either cross-domain or semantically ambiguous enough that the final wording should be checked carefully before insertion into the report:

- `Accounts`
- `Consultations`
- `ConsultationBookings`
- `UserFeedbacks`
- `SnakebiteIncidents`
- `RescueMissions`
- `TrackingSessions`
- `LocationEvents`
- `SnakeCatchingRequests`
- `SnakeCatchingMissions`
- `SnakeTariffs`
- `SnakeSpecies`
- `FirstAidGuidelines`
- `LibraryMedias`
- `SnakeAIRecognitionResults`
- `ReportMedias`
- `PaymentCards`
- `Transactions`
- `CommunityReports`
