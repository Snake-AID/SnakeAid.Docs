# Report 4 Table Verification Notes

## Purpose

This file records the code-backed verification work for tables that were not safe to describe from the ERD alone.

## Verification Summary

The following tables were checked directly against current backend entities, configurations, migrations, or service/controller usage:

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

## Verified Findings

### CommunityReports

- Verified from `SnakeAid.Core/Domains/CommunityReport.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/CommunityReportConfiguration.cs`
- Confirmed meaning:
  - stores community-submitted reports
  - includes `UserId`, `LocationCoordinates`, `Notes`, optional `SnakeSpeciesId`
  - supports polymorphic media attachments through `ReportMedia`

### ExpertCertificates

- Verified from `SnakeAid.Core/Domains/ExpertCertificate.cs`
- Confirmed meaning:
  - stores expert qualification certificates
  - includes `ExpertId`, certificate metadata, verification status, and rejection reason
- Important note:
  - `ExpertId` exists in code even though the Report 4 ERD does not explicitly mark it as an FK

### Lessons

- Verified from `SnakeAid.Core/Domains/Lesson.cs`
- Verified from `LessonController` and `LessonService` presence in the backend graph
- Confirmed meaning:
  - stores educational lesson content
  - includes `Title`, `Content`, `Category`, and `IsPublished`

### LocationEvents

- Verified from `SnakeAid.Core/Domains/LocationEvent.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/LocationEventConfiguration.cs`
- Confirmed meaning:
  - stores event-level location history entries
  - records `SessionId`, `SessionType`, `AccountId`, `Role`, `Location`, `RecordedAt`, `Speed`, and `Heading`
- Important note:
  - the table uses logical references rather than explicit FK relationships in the ERD

### PaymentCards

- Verified from `SnakeAid.Core/Domains/PaymentCard.cs`
- Confirmed meaning:
  - stores payment card details and default-card status
- Important note:
  - current entity has no owner FK, so the table should be described conservatively

### ReportMedias

- Verified from `SnakeAid.Core/Domains/ReportMedia.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/ReportMediaConfiguration.cs`
- Confirmed meaning:
  - stores uploaded media attached to multiple parent entity types
  - uses `ReferenceId` + `ReferenceType` as a polymorphic parent-link pattern
  - includes `Purpose`, upload batching data, and AI processing flags

### SnakeAIRecognitionResults

- Verified from `SnakeAid.Core/Domains/SnakeAIRecognitionResult.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/SnakeAIRecognitionResultConfiguration.cs`
- Confirmed meaning:
  - stores AI recognition output for a `ReportMedia`
  - includes model, class name, confidence, mapped species, review status, expert correction, and notes

### TrackingSessions

- Verified from `SnakeAid.Core/Domains/TrackingSession.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/TrackingSessionConfiguration.cs`
- Confirmed meaning:
  - stores the current live-tracking state for a session
  - includes polymorphic `SessionId` + `SessionType`, current participant locations, freshness timestamps, distance, and ETA

### Transactions

- Verified from `SnakeAid.Core/Domains/Transaction.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/TransactionConfiguration.cs`
- Confirmed meaning:
  - stores the monetary transaction ledger
  - includes optional `UserId`, mandatory `ReferenceId`, amount, currency, transaction type, payment method, and external gateway id
- Important note:
  - `ReferenceId` is a logical business reference, not an explicit FK

### UserFeedbacks

- Verified from `SnakeAid.Core/Domains/UserFeedback.cs`
- Verified from `SnakeAid.Repository/Data/Configurations/UserFeedbackConfiguration.cs`
- Confirmed meaning:
  - stores ratings and comments from one account to another
  - uses `ReferenceId` + `Type` to tie the feedback to a business context

## Outcome

After this verification pass, the draft table descriptions can be treated as code-grounded for insertion into the main Report 4 file.
