# SnakeAid Physical ERD to Mermaid

- Source: `SnakeAid Physical ERD.drawio`
- Source page: `Page-4`
- Notes: Converted from the latest page in the draw.io file and normalized to `PK -> FK` relationships for Mermaid rendering.

```mermaid
erDiagram
    Accounts {
        uuid id PK
        varchar FullName
        varchar AvatarUrl
        int4 Role
        timestamptz CreatedAt
        timestamptz UpdatedAt
        bool IsActive
        int4 ReputationPoints
        int4 ReputationStatus
        timestamptz SuspendedUntil
        text SuspensionReason
        varchar UserName
        varchar Email
        text PasswordHash
        text SuspensionReason
    }
    AIModels {
        int4 Id PK
        varchar Version
        varchar Description
        varchar ContactNumber
        bool IsDefault
        bool IsActive
        timestampz DeployedAt
        timestampz RetiredAt
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    AISnakeClassMappings {
        uuid Id PK
        int4 SnakeSpeciesId FK
        int4 AIModelId FK
        text YoloClassname
        int4 YoloClassId
        bool IsActive
    }
    Antivenoms {
        int4 Id PK
        int4 TreatmentFacilityId FK
        text Name
        varchar Manufacturer
        varchar Manufacturer
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    AppNotifications {
        uuid id PK
        uuid UserId FK
        text Title
        text Message
        bool IsRead
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    Blogs {
        uuid id PK
        uuid AuthorId FK
        varchar Title
        text Content
        int4 Status
        text RejectionReason
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    CatchingEnvironment {
        int4 Id PK
        text Name
        text Description
        numeric Price
        varchar Currency
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    CatchingMissionDetails {
        uuid Id PK
        uuid SnakeCatchingMissionId FK
        int4 SnakeSpeciesId FK
        int4 Quantity
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    CatchingRequestDetails {
        uuid Id PK
        uuid SnakeCatchingRequestId FK
        int4 SnakeSpeciesId FK
        int4 Quantity
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    ChatMessages {
        uuid id PK
        uuid ConsultationId FK
        uuid SenderId FK
        varchar Content
        timestamptz SentAt
        text AttachmentUrl
    }
    CommunityReports {
        uuid id PK
        uuid UserId FK
        geometry CommunityReport
        text AdditionalDetails
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    ConsultationBookings {
        uuid id PK
        uuid UserId FK
        uuid ExpertId FK
        numeric Price
        timestamptz BookedAt
        timestamptz PaymentDeadline
        int4 Status
        timestamptz CancelledAt
        text CancellationReason
        uuid ConsultationId FK
        uuid TimeSlotId FK
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    Consultations {
        uuid id PK
        uuid CallerId FK
        uuid CalleeId FK
        text RoomId
        timestamptz StartTime
        timestamptz EndTime
        int4 Status
        int4 Type
        uuid ExpertTimeSlotId FK
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    ExpertCertificates {
        uuid id PK
        uuid ExpertId
        varchar CertificateName
        varchar IssuingOrganization
        timestamptz IssueDate
        timestamptz ExpiryDate
        text CertificateUrl
        int4 VerificationStatus
        text RejectionReason
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    ExpertProfiles {
        uuid AccountId PK,FK
        varchar Biography
        bool IsOnline
        numeric ConsultationFee
        numeric Rating
        int4 RatingCount
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    ExpertSpecializations {
        uuid id PK
        uuid ExpertId FK
        int4 SpecializationId FK
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    ExpertTimeSlots {
        uuid id PK
        uuid ExpertId FK
        timestamptz StartTime
        timestamptz EndTime
        int4 Status
    }
    FirstAidGuidelines {
        int4 Id PK
        varchar Name
        jsonb Content
        int4 Type
        varchar Summary
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    GeographicRegion {
        int4 Id PK
        varchar Name
        varchar Code
        varchar Description
        geography Boundary
        int4 DisplayOrder
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    Lessons {
        uuid id PK
        text Title
        text Content
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    LibraryMedias {
        int4 Id PK
        int4 SnakeSpeciesId FK
        int4 UploadedById FK
        varchar MediaUrl
        int4 MediaType
        varchar FileName
        int8 FileSizeBytes
        varchar ContentType
        bool IsPublic
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    LocationEvents {
        int8 Id PK
        uuid SessionId
        int4 SessionType
        uuid AccountId
        int4 Role
        timestamptz RecordedAt
        float4 Speed
        float4 Heading
        geometry Location
    }
    MemberProfiles {
        uuid AccountId PK,FK
        float4 Rating
        int4 RatingCount
        jsonb EmergencyContacts
        bool HasUnderlyingDisease
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    PaymentCards {
        uuid id PK
        text CardNumber
        text CardHolderName
        timestamptz ExpiryDate
        text Cvv
        bool IsDefault
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    RegionSnakeMapping {
        int4 Id PK
        int4 SnakeSpeciesId FK
        int4 GeographicRegionId FK
        int4 CommonLevel
        int4 Priority
        varchar DistributionNotes
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    ReportMedias {
        uuid Id PK
        uuid ReferenceId
        int4 ReferenceType
        text FileName
        text MediaUrl
        text ContentType
        int8 FileSize
        int4 Purpose
        uuid UploadBatchId
        int4 SequenceOrder
        bool RequiresAIProcessing
        bool IsProcessed
        timestampz ProcessedAt
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    RescueMissions {
        uuid Id PK
        uuid RescuerId FK
        uuid IncidentId FK
        numeric Price
        numeric EstimatedCost
        numeric ActualCost
        timestampz StartedAt
        timestampz ArrivedAt
        timestampz CompletedAt
        int4 Status
        varchar CancellationReason
        varchar Notes
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    RescueRequestSessions {
        uuid Id PK
        uuid IncidentId FK
        int4 SessionNumber
        int4 RadiusKm
        int4 Status
        timestampz CompletedAt
        int4 TriggerType
        int4 RescuersPinged
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    RescuerProfiles {
        uuid AccountId PK,FK
        bool IsOnline
        numeric Rating
        int4 RatingCount
        int4 Type
        geometry LastLocation
        timestamptz LastLocationUpdate
        int4 TotalMissions
        int4 CompletedMissions
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    RescuerRequests {
        uuid Id PK
        uuid SessionId FK
        uuid IncidentId FK
        uuid RescuerId FK
        int4 Status
        timestampz RequestSentAt
        timestampz ResponseAt
        timestampz ExpiredAt
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    ShiftAssignment {
        uuid id PK
        uuid RescuerId FK
        uuid ShiftId FK
        timestamptz ShiftStartLocal
        timestamptz ShiftEndLocal
        int4 Status
        timestamptz CheckInAtUtc
        timestamptz CheckOutAtUtc
        varchar Notes
        bool IsActive
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    SnakeAIRecognitionResults {
        uuid Id PK
        uuid ReportMediaId FK
        int4 AIModelId FK
        int4 DetectedSpeciesId FK
        int4 ExpertCorrectedSpeciesId FK
        uuid ExpertId FK
        uuid SnakeCatchingMissionId FK
        varchar YoloClassName
        numeric Confidence
        bool IsMapped
        jsonb AllDetections
        int4 Status
        timestampz ExpertVerifiedAt
        varchar ExpertNotes
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakebiteIncidents {
        uuid Id PK
        uuid UserId FK
        uuid AssignedRescuerId FK
        uuid AIRecognitionResultId FK
        int4 IdentifiedSnakeSpeciesId FK
        geometry LocationCoordinates
        jsonb SymptomsReport
        int4 Status
        int4 CurrentSessionNumber
        int4 CurrentRadiusKm
        timestampz LastSessionAt
        timestampz AssignedAt
        varchar CancellationReason
        int4 SeverityLevel
        timestampz IncidentOccurredAt
        timestampz IdentifiedAt
        jsonb FilterAnswers
        int4 IdentificationMethod
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakeCatchingMissions {
        uuid Id PK
        uuid RescuerId FK
        uuid SnakeCatchingRequestId FK
        int4 CatchingEnvironmentId FK
        numeric Price
        numeric EstimatedCost
        numeric ActualCost
        timestampz StartedAt
        timestampz ArrivedAt
        timestampz CompletedAt
        int4 Status
        varchar CancellationReason
        varchar Notes
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakeCatchingRequests {
        uuid Id PK
        uuid UserId FK
        uuid AssignedRescuerId FK
        varchar Address
        geometry LocationCoordinates
        varchar AdditionalDetails
        int4 Status
        int4 Priority
        timestampz RequestDate
        timestampz PreferredTime
        timestampz AssignedAt
        numeric EstimatedPrice
        varchar CancellationReason
        varchar Notes
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakeSpecies {
        int4 Id PK
        varchar SciencetificName
        varchar Slug
        varchar CommonName
        varchar ImageUrl
        varchar Description
        varchar IdentificationSummary
        int4 PrimaryVenomType
        jsonb Identification
        jsonb SymptomsByTime
        jsonb FirstAidGuidelineOverride
        float4 RiskLevel
        bool IsVenomous
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakeSpeciesNames {
        uuid Id PK
        int4 SnakeSpeciesId FK
        varchar Name
        varchar Slug
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SnakeTariffs {
        uuid Id PK
        int4 SnakeSpeciesId FK
        int4 SizeCategory
        numeric BasePrice
        text Currency
        varchar Note
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    Specializations {
        int4 id PK
        varchar Name
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    SpeciesAntivenoms {
        int4 Id PK
        int4 SnakeSpeciesId FK
        int4 AntivenomId FK
        varchar Slug
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SpeciesVenoms {
        int4 SnakeSpeciesId PK,FK
        int4 VenomTypeId PK,FK
    }
    SymptomConfigs {
        int4 Id PK
        int4 VenomTypeId FK
        varchar GroupName
        varchar AttributeKey
        varchar AttributeLabel
        int4 DisplayOrder
        varchar Name
        varchar Description
        bool IsCritical
        varchar AlertMessage
        int4 Category
        jsonb TimeScoreJson
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    SystemSettings {
        varchar SettingKey PK
        varchar Value
        varchar Description
        int4 ValueType
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    TrackingSessions {
        uuid Id PK
        uuid SessionId
        int4 SessionType
        bool IsActive
        geometry MemberLocation
        geometry RescuerLocation
        timestamptz MemberLastUpdate
        timestamptz RescuerLastUpdate
        int8 DistanceMeters
        int4 EtaMinutes
        timestamptz UpdatedAt
        timestamptz UpdatedAt
    }
    Transactions {
        uuid id PK
        uuid UserId FK
        uuid ReferenceId
        numeric Amount
        varchar Currency
        int4 TransactionType
        varchar Description
        varchar PaymentMethod
        varchar ExternalTransactionId
        timestamptz CreatedAt
    }
    TreatementFacilities {
        int4 Id PK
        varchar Name
        varchar Address
        varchar ContactNumber
        geometry Location
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    UserFeedbacks {
        uuid id PK
        uuid RaterId FK
        uuid TargetUserId FK
        uuid ReferenceId
        int4 Type
        int4 Rating
        varchar Comments
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    VenomTypes {
        int4 Id PK
        int4 FirstAidGuidelineId FK
        varchar Name
        varchar Description
        varchar SciencetificName
        varchar Description
        int4 ServerityIndex
        bool IsActive
        timestampz CreatedAt
        timestampz UpdatedAt
    }
    Wallets {
        uuid id PK
        uuid UserId FK
        numeric Balance
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    WalletWithdraws {
        uuid id PK
        uuid UserId FK
        uuid WalletId FK
        numeric Amount
        varchar BankAccount
        varchar BankName
        int4 Status
        timestamptz ProcessedAt
        varchar RejectionReason
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }
    WorkShift {
        uuid id PK
        varchar Name
        interval StartTime
        interval EndTime
        int4 RequiredRescuers
        bool IsActive
        timestamptz CreatedAt
        timestamptz UpdatedAt
    }

    AIModels ||--o{ AISnakeClassMappings : "AIModelId"
    AIModels ||--o{ SnakeAIRecognitionResults : "AIModelId"
    Accounts ||--o{ AppNotifications : "UserId"
    Accounts ||--o{ ChatMessages : "SenderId"
    Accounts ||--o{ CommunityReports : "UserId"
    Accounts ||--o{ LibraryMedias : "UploadedById"
    Accounts ||--o{ Transactions : "UserId"
    Accounts ||--o{ Wallets : "UserId"
    Accounts ||--|| ExpertProfiles : "AccountId"
    Antivenoms ||--o{ SpeciesAntivenoms : "AntivenomId"
    CatchingEnvironment ||--o{ SnakeCatchingMissions : "CatchingEnvironmentId"
    Consultations ||--o{ ChatMessages : "ConsultationId"
    Consultations ||--o{ ConsultationBookings : "ConsultationId"
    Consultations ||--o{ UserFeedbacks : "ReferenceId"
    ExpertCertificates ||--|| ExpertProfiles : "AccountId"
    ExpertProfiles ||--o{ Blogs : "AuthorId"
    ExpertProfiles ||--o{ ConsultationBookings : "ExpertId"
    ExpertProfiles ||--o{ Consultations : "CalleeId"
    ExpertProfiles ||--o{ ExpertSpecializations : "ExpertId"
    ExpertProfiles ||--o{ ExpertTimeSlots : "ExpertId"
    ExpertProfiles ||--o{ UserFeedbacks : "RaterId"
    ExpertProfiles ||--o{ UserFeedbacks : "TargetUserId"
    ExpertTimeSlots ||--o{ ConsultationBookings : "TimeSlotId"
    ExpertTimeSlots ||--o{ Consultations : "ExpertTimeSlotId"
    FirstAidGuidelines ||--o{ VenomTypes : "FirstAidGuidelineId"
    GeographicRegion ||--o{ RegionSnakeMapping : "GeographicRegionId"
    MemberProfiles ||--o{ Consultations : "CallerId"
    MemberProfiles ||--o{ SnakeCatchingRequests : "UserId"
    MemberProfiles ||--o{ SnakebiteIncidents : "UserId"
    MemberProfiles ||--o{ UserFeedbacks : "RaterId"
    MemberProfiles ||--o{ UserFeedbacks : "TargetUserId"
    ReportMedias ||--o{ SnakeAIRecognitionResults : "ReportMediaId"
    RescueMissions ||--o{ ReportMedias : "ReferenceId"
    RescueRequestSessions ||--o{ RescuerRequests : "SessionId"
    RescuerProfiles ||--o{ Consultations : "CallerId"
    RescuerProfiles ||--o{ RescueMissions : "RescuerId"
    RescuerProfiles ||--o{ RescuerRequests : "RescuerId"
    RescuerProfiles ||--o{ ShiftAssignment : "RescuerId"
    RescuerProfiles ||--o{ SnakeCatchingMissions : "RescuerId"
    RescuerProfiles ||--o{ SnakeCatchingRequests : "AssignedRescuerId"
    RescuerProfiles ||--o{ SnakebiteIncidents : "AssignedRescuerId"
    RescuerProfiles ||--o{ UserFeedbacks : "RaterId"
    RescuerProfiles ||--o{ UserFeedbacks : "TargetUserId"
    SnakeAIRecognitionResults ||--o{ SnakebiteIncidents : "AIRecognitionResultId"
    SnakeCatchingMissions ||--o{ CatchingMissionDetails : "SnakeCatchingMissionId"
    SnakeCatchingMissions ||--o{ ReportMedias : "ReferenceId"
    SnakeCatchingMissions ||--o{ SnakeAIRecognitionResults : "SnakeCatchingMissionId"
    SnakeCatchingRequests ||--o{ CatchingRequestDetails : "SnakeCatchingRequestId"
    SnakeCatchingRequests ||--o{ ReportMedias : "ReferenceId"
    SnakeCatchingRequests ||--o{ SnakeCatchingMissions : "SnakeCatchingRequestId"
    SnakeCatchingRequests ||--o{ UserFeedbacks : "ReferenceId"
    SnakeSpecies ||--o{ AISnakeClassMappings : "SnakeSpeciesId"
    SnakeSpecies ||--o{ CatchingMissionDetails : "SnakeSpeciesId"
    SnakeSpecies ||--o{ CatchingRequestDetails : "SnakeSpeciesId"
    SnakeSpecies ||--o{ LibraryMedias : "SnakeSpeciesId"
    SnakeSpecies ||--o{ RegionSnakeMapping : "SnakeSpeciesId"
    SnakeSpecies ||--o{ SnakeAIRecognitionResults : "DetectedSpeciesId"
    SnakeSpecies ||--o{ SnakeAIRecognitionResults : "ExpertCorrectedSpeciesId"
    SnakeSpecies ||--o{ SnakeSpeciesNames : "SnakeSpeciesId"
    SnakeSpecies ||--o{ SnakeTariffs : "SnakeSpeciesId"
    SnakeSpecies ||--o{ SnakebiteIncidents : "IdentifiedSnakeSpeciesId"
    SnakeSpecies ||--o{ SpeciesAntivenoms : "SnakeSpeciesId"
    SnakeSpecies ||--o{ SpeciesVenoms : "SnakeSpeciesId"
    SnakebiteIncidents ||--o{ ReportMedias : "ReferenceId"
    SnakebiteIncidents ||--o{ RescueMissions : "IncidentId"
    SnakebiteIncidents ||--o{ RescueRequestSessions : "IncidentId"
    SnakebiteIncidents ||--o{ RescuerRequests : "IncidentId"
    SnakebiteIncidents ||--o{ UserFeedbacks : "ReferenceId"
    Specializations ||--o{ ExpertSpecializations : "SpecializationId"
    TreatementFacilities ||--o{ Antivenoms : "TreatmentFacilityId"
    VenomTypes ||--o{ SpeciesVenoms : "VenomTypeId"
    VenomTypes ||--o{ SymptomConfigs : "VenomTypeId"
    Wallets ||--o{ WalletWithdraws : "WalletId"
    WorkShift ||--o{ ShiftAssignment : "ShiftId"
```
