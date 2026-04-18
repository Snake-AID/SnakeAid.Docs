

#### ***3.3.2 Sequence Diagram View List Experts and Presence***

```mermaid
sequenceDiagram
    actor MemberApp
    actor ExpertApp
    participant ExpertController
    participant ExpertService
    participant ExpertProfileRepository
    participant ExpertHub

    MemberApp->>ExpertController: GET /api/experts
    ExpertController->>ExpertService: GetExpertsAsync(request)
    ExpertService->>ExpertProfileRepository: GetPagingListAsync(predicate, include, orderBy, page, size)
    ExpertProfileRepository-->>ExpertService: PagingResponse<ExpertProfile>
    ExpertService->>ExpertService: MapExpertProfilesAsync(pagedData.Items)
    ExpertService-->>ExpertController: PagingResponse<ExpertProfileResponse>
    ExpertController-->>MemberApp: ApiResponse<PagingResponse<ExpertProfileResponse>>

    MemberApp->>ExpertHub: JoinAsMember()
    ExpertHub-->>MemberApp: OnlineExpertsSnapshot

    ExpertApp->>ExpertHub: JoinAsExpert()
    ExpertHub-->>MemberApp: ExpertPresenceChanged(ExpertId, IsOnline=true, ChangedAtUtc)
```

#### ***3.3.3 Sequence Diagram Create and Pay Scheduled Booking***

```mermaid
sequenceDiagram
    actor MemberApp
    participant ConsultationScheduledController
    participant BookingService
    participant ExpertTimeSlotRepository
    participant ExpertProfileRepository
    participant ConsultationRepository
    participant ConsultationPaymentsController
    participant ConsultationPaymentService
    participant ConsultationBookingRepository
    participant PaymentGateway
    activate ConsultationScheduledController
    activate BookingService
    activate ExpertTimeSlotRepository
    activate ExpertProfileRepository
    activate ConsultationRepository
    activate ConsultationPaymentsController
    activate ConsultationPaymentService
    activate ConsultationBookingRepository
    activate PaymentGateway

    MemberApp->>ConsultationScheduledController: POST /api/consultations/scheduled
    ConsultationScheduledController->>BookingService: CreateScheduledBookingAsync(userId, request)
    BookingService->>ExpertTimeSlotRepository: FirstOrDefaultAsync(slotId, asNoTracking:false)
    BookingService->>ExpertProfileRepository: FirstOrDefaultAsync(AccountId == slot.ExpertId)
    BookingService->>ConsultationRepository: InsertAsync(new Consultation { RoomId = "consultation-{consultationId}", Status = Scheduled, Type = Scheduled })
    BookingService->>ConsultationBookingRepository: InsertAsync(new ConsultationBooking { Status = PendingPayment, ConsultationId, TimeSlotId, PaymentDeadline })
    BookingService->>ExpertTimeSlotRepository: Update(slot.Status = Reserved)
    BookingService-->>ConsultationScheduledController: ConsultationBookingResponse
    ConsultationScheduledController-->>MemberApp: Booking created (PendingPayment, ConsultationId, RoomId)

    MemberApp->>ConsultationPaymentsController: POST /api/consultations/scheduled/{bookingId}/payments
    ConsultationPaymentsController->>ConsultationPaymentService: PayScheduledBookingAsync(userId, bookingId, request, cancellationToken)
    alt WalletBalance
        ConsultationPaymentService->>ConsultationPaymentService: PayScheduledBookingWithWalletAsync(userId, bookingId, request, cancellationToken)
        ConsultationPaymentService->>ConsultationBookingRepository: FirstOrDefaultAsync(Id == bookingId, include Consultation, asNoTracking:false)
        ConsultationPaymentService->>ConsultationPaymentService: MoveMoneyToEscrowAsync(userId, booking.Id, booking.Price, ConsultationPayment, Wallet, ...)
        ConsultationPaymentService->>ConsultationBookingRepository: Update(booking.Status = Confirmed)
        ConsultationPaymentService-->>ConsultationPaymentsController: ConsultationPaymentResponse(status=Escrowed)
        ConsultationPaymentsController-->>MemberApp: 200 OK Escrowed
    else PayOs
        ConsultationPaymentService->>ConsultationPaymentService: CreateScheduledBookingPayOsIntentAsync(userId, bookingId, request, cancellationToken)
        ConsultationPaymentService->>ConsultationBookingRepository: FirstOrDefaultAsync(Id == bookingId, include Consultation, asNoTracking:false)
        ConsultationPaymentService->>ConsultationPaymentService: PreparePendingPayOsTransactionAsync(userId, booking.Id, booking.Price, ...)
        ConsultationPaymentService->>PaymentGateway: CreatePayOsLinkAsync(orderCode, booking.Price, ...)
        PaymentGateway-->>ConsultationPaymentService: checkoutUrl + orderCode + paymentLinkId
        ConsultationPaymentService-->>ConsultationPaymentsController: ConsultationPaymentResponse(status=Pending)
        ConsultationPaymentsController-->>MemberApp: 200 OK Pending
        MemberApp->>PaymentGateway: Complete PayOS checkout
        alt Client fallback confirm
            MemberApp->>ConsultationPaymentsController: POST /api/consultations/payments/confirm
            ConsultationPaymentsController->>ConsultationPaymentService: ConfirmConsultationPaymentAsync(transactionId, cancellationToken)
            ConsultationPaymentService->>PaymentGateway: GetPaymentLinkInformationAsync(orderCode, cancellationToken)
            ConsultationPaymentService->>ConsultationPaymentService: ProcessConfirmedPayOsPaymentAsync(webhookData, cancellationToken)
        else PayOS webhook
            PaymentGateway->>ConsultationPaymentService: ProcessConsultationWebhookAsync(request, cancellationToken)
            ConsultationPaymentService->>ConsultationPaymentService: ProcessConfirmedPayOsPaymentAsync(webhookData, cancellationToken)
        end
        ConsultationPaymentService->>ConsultationPaymentService: MoveMoneyToEscrowAsync(payerUserId, transaction.ReferenceId, transaction.Amount, ConsultationPayment, PayOS, ..., skipExistingPaymentInsert:true)
        ConsultationPaymentService->>ConsultationBookingRepository: FirstOrDefaultAsync(Id == transaction.ReferenceId, include Consultation, asNoTracking:false)
        ConsultationPaymentService->>ConsultationBookingRepository: Update(booking.Status = Confirmed)
    end
```

#### ***3.3.4 Sequence Diagram Create, Pay, and Notify Emergency Consultation Request***

```mermaid
sequenceDiagram
    actor MemberApp
    participant ConsultationInstantController
    participant EmergencyConsultationService
    participant ExpertHub
    participant ConsultationPaymentsController
    participant ConsultationPaymentService
    participant ConsultationPingRequestRepository
    participant PaymentGateway
    participant ExpertNotificationService
    activate ConsultationInstantController
    activate EmergencyConsultationService
    activate ExpertHub
    activate ConsultationPaymentsController
    activate ConsultationPaymentService
    activate ConsultationPingRequestRepository
    activate PaymentGateway
    activate ExpertNotificationService

    MemberApp->>ConsultationInstantController: POST /api/consultations/instant
    ConsultationInstantController->>EmergencyConsultationService: CreateEmergencyRequestAsync(requesterId, request)
    EmergencyConsultationService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(active pending request check)
    EmergencyConsultationService->>ConsultationPingRequestRepository: InsertAsync(new ConsultationPingRequest { Status = PendingPayment })
    EmergencyConsultationService-->>ConsultationInstantController: EmergencyConsultationRequestResponse
    ConsultationInstantController-->>MemberApp: Request created (PendingPayment)

    MemberApp->>ExpertHub: JoinEmergencyRequestRoom(requestId)
    ExpertHub-->>MemberApp: JoinedEmergencyRequestRoom(RequestId, GroupName)

    MemberApp->>ConsultationPaymentsController: POST /api/consultations/instant/{requestId}/payments
    ConsultationPaymentsController->>ConsultationPaymentService: PayEmergencyRequestAsync(userId, requestId, request, cancellationToken)
    alt WalletBalance
        ConsultationPaymentService->>ConsultationPaymentService: PayEmergencyRequestWithWalletAsync(userId, requestId, request, cancellationToken)
        ConsultationPaymentService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(Id == requestId, asNoTracking:false)
        ConsultationPaymentService->>ExpertNotificationService: IsExpertConnected(ping.ExpertId.ToString())
        ConsultationPaymentService->>ConsultationPaymentService: GetEmergencyFeeAsync(ping.ExpertId, cancellationToken)
        ConsultationPaymentService->>ConsultationPaymentService: MoveMoneyToEscrowAsync(userId, ping.Id, emergencyFee, ConsultationPayment, Wallet, ...)
        ConsultationPaymentService->>ConsultationPingRequestRepository: Update(ping.Status = PendingExpertResponse, RequestedAt = UtcNow, ExpiresAt = RequestedAt + EmergencyRequestTtl)
        ConsultationPaymentService->>ExpertNotificationService: SendEmergencyRequestAsync(...)
        ExpertNotificationService-->>ExpertHub: EmergencyConsultationRequest
        ConsultationPaymentService-->>ConsultationPaymentsController: ConsultationPaymentResponse(status=Escrowed)
        ConsultationPaymentsController-->>MemberApp: 200 OK Escrowed
    else PayOs
        ConsultationPaymentService->>ConsultationPaymentService: CreateEmergencyRequestPayOsIntentAsync(userId, requestId, request, cancellationToken)
        ConsultationPaymentService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(Id == requestId, asNoTracking:false)
        ConsultationPaymentService->>ExpertNotificationService: IsExpertConnected(ping.ExpertId.ToString())
        ConsultationPaymentService->>ConsultationPaymentService: GetEmergencyFeeAsync(ping.ExpertId, cancellationToken)
        ConsultationPaymentService->>ConsultationPaymentService: PreparePendingPayOsTransactionAsync(userId, ping.Id, emergencyFee, ...)
        ConsultationPaymentService->>PaymentGateway: CreatePayOsLinkAsync(orderCode, emergencyFee, ...)
        PaymentGateway-->>ConsultationPaymentService: checkoutUrl + orderCode + paymentLinkId
        ConsultationPaymentService-->>ConsultationPaymentsController: ConsultationPaymentResponse(status=Pending)
        ConsultationPaymentsController-->>MemberApp: 200 OK Pending
        MemberApp->>PaymentGateway: Complete PayOS checkout
        alt Client fallback confirm
            MemberApp->>ConsultationPaymentsController: POST /api/consultations/payments/confirm
            ConsultationPaymentsController->>ConsultationPaymentService: ConfirmConsultationPaymentAsync(transactionId, cancellationToken)
            ConsultationPaymentService->>ConsultationPaymentService: ProcessConfirmedPayOsPaymentAsync(webhookData, cancellationToken)
        else PayOS webhook
            PaymentGateway->>ConsultationPaymentService: ProcessConsultationWebhookAsync(request, cancellationToken)
            ConsultationPaymentService->>ConsultationPaymentService: ProcessConfirmedPayOsPaymentAsync(webhookData, cancellationToken)
        end
        ConsultationPaymentService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(Id == transaction.ReferenceId, asNoTracking:false)
        ConsultationPaymentService->>ConsultationPingRequestRepository: Update(ping.Status = PendingExpertResponse, RequestedAt = UtcNow, ExpiresAt = RequestedAt + EmergencyRequestTtl)
        ConsultationPaymentService->>ExpertNotificationService: SendEmergencyRequestAsync(...)
        ExpertNotificationService-->>ExpertHub: EmergencyConsultationRequest
    end
```

#### ***3.3.5 Sequence Diagram Expert Accept or Reject Emergency Request***

```mermaid
sequenceDiagram
    actor ExpertApp
    actor MemberApp
    participant ExpertHub
    participant ConsultationInstantController
    participant EmergencyConsultationService
    participant ExpertNotificationService
    participant ConsultationPaymentService
    participant ConsultationLifecycleBackgroundService
    participant ConsultationPingRequestRepository
    participant ConsultationRepository
    participant ExpertTimeSlotRepository
    activate ExpertHub
    activate ConsultationInstantController
    activate EmergencyConsultationService
    activate ExpertNotificationService
    activate ConsultationPaymentService
    activate ConsultationLifecycleBackgroundService
    activate ConsultationPingRequestRepository
    activate ConsultationRepository
    activate ExpertTimeSlotRepository

    ExpertHub-->>ExpertApp: EmergencyConsultationRequest
    alt Accept
        ExpertApp->>ConsultationInstantController: POST /api/consultations/instant/{requestId}/accept
        ConsultationInstantController->>EmergencyConsultationService: AcceptEmergencyRequestAsync(requestId, expertId)
        EmergencyConsultationService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(Id == requestId, asNoTracking:false)
        EmergencyConsultationService->>EmergencyConsultationService: EnsurePendingStateForResponse(ping, now)
        EmergencyConsultationService->>ConsultationRepository: InsertAsync(new Consultation { Status = Ongoing, Type = Emergency, RoomId = "consultation-{consultationId}" })
        EmergencyConsultationService->>ExpertTimeSlotRepository: GetListAsync(overlapping available ExpertTimeSlot)
        EmergencyConsultationService->>ConsultationPingRequestRepository: Update(ping.Status = AcceptedByExpert, RespondedAt = now, ConsultationId = consultationId)
        EmergencyConsultationService->>ExpertTimeSlotRepository: Update(overlapping slots => Reserved)
        EmergencyConsultationService->>ExpertNotificationService: NotifyEmergencyRequestStatusChangedAsync(requestId, result)
        ExpertNotificationService-->>ExpertHub: EmergencyRequestStatusChanged(status=AcceptedByExpert)
        ExpertHub-->>MemberApp: EmergencyRequestStatusChanged(status=AcceptedByExpert)
        EmergencyConsultationService-->>ConsultationInstantController: EmergencyConsultationRequestResponse
        ConsultationInstantController-->>ExpertApp: 200 OK Accepted
    else Reject
        ExpertApp->>ConsultationInstantController: POST /api/consultations/instant/{requestId}/reject
        ConsultationInstantController->>EmergencyConsultationService: RejectEmergencyRequestAsync(requestId, expertId)
        EmergencyConsultationService->>ConsultationPingRequestRepository: FirstOrDefaultAsync(Id == requestId, asNoTracking:false)
        EmergencyConsultationService->>EmergencyConsultationService: EnsurePendingStateForResponse(ping, now)
        EmergencyConsultationService->>ConsultationPingRequestRepository: Update(ping.Status = DeclinedByExpert, RespondedAt = now)
        EmergencyConsultationService->>ConsultationPaymentService: RefundEmergencyEscrowAsync(requestId, reason)
        ConsultationPaymentService-->>EmergencyConsultationService: refund completed
        EmergencyConsultationService->>ExpertNotificationService: NotifyEmergencyRequestStatusChangedAsync(requestId, result)
        ExpertNotificationService-->>ExpertHub: EmergencyRequestStatusChanged(status=DeclinedByExpert)
        ExpertHub-->>MemberApp: EmergencyRequestStatusChanged(status=DeclinedByExpert)
        EmergencyConsultationService-->>ConsultationInstantController: EmergencyConsultationRequestResponse
        ConsultationInstantController-->>ExpertApp: 200 OK Declined
    else Timeout sweep
        ConsultationLifecycleBackgroundService->>ConsultationPaymentService: ExpireEmergencyRequestsAsync(stoppingToken)
        ConsultationPaymentService->>ConsultationPingRequestRepository: GetListAsync(Status in PendingExpertResponse or PendingPayment, expired by ExpiresAt/RequestedAt)
        ConsultationPaymentService->>ConsultationPingRequestRepository: Update(ping.Status = Expired, RespondedAt = now)
        opt Request was PendingExpertResponse
            ConsultationPaymentService->>ConsultationPaymentService: RefundEmergencyEscrowAsync(ping.Id, Emergency consultation request expired.)
        end
        ConsultationPaymentService->>ExpertNotificationService: NotifyEmergencyRequestStatusChangedAsync(ping.Id, statusData)
        ExpertNotificationService-->>ExpertHub: EmergencyRequestStatusChanged(status=Expired)
        ExpertHub-->>MemberApp: EmergencyRequestStatusChanged(status=Expired)
    end
```

#### ***3.3.6 Sequence Diagram Join Consultation Room and In-Room Interaction***

```mermaid
sequenceDiagram
    actor ParticipantApp
    participant VideoCallController
    participant ConsultationRepository
    participant LiveKitService
    participant LiveKitCloud
    participant ConsultationHub
    participant MediaController
    participant ChatMessageRepository
    activate VideoCallController
    activate ConsultationRepository
    activate LiveKitService
    activate LiveKitCloud
    activate ConsultationHub
    activate MediaController
    activate ChatMessageRepository

    ParticipantApp->>VideoCallController: POST /api/consultations/{consultationId}/video-token
    VideoCallController->>ConsultationRepository: FirstOrDefaultAsync(c.Id == consultationId)
    VideoCallController->>LiveKitService: GenerateAccessToken(identity, roomName, grants, metadata)
    LiveKitService-->>VideoCallController: token
    VideoCallController-->>ParticipantApp: ApiResponse<VideoTokenResponse>
    ParticipantApp->>LiveKitCloud: Join room consultation.RoomId

    ParticipantApp->>ConsultationHub: Connect /hubs/consultation?consultationId={consultationId}
    ConsultationHub-->>ParticipantApp: Authorized participant connection

    opt Upload image attachment
        ParticipantApp->>MediaController: POST /api/media/upload-image
        MediaController-->>ParticipantApp: ApiResponse<upload result>
    end

    ParticipantApp->>ConsultationHub: ReceiveMessage(content, attachmentUrl)
    ConsultationHub->>ConsultationHub: GetConsultationIdFromQuery() + GetCurrentUserId() + CheckRateLimit()
    ConsultationHub->>ChatMessageRepository: Persist ChatMessage
    ChatMessageRepository-->>ConsultationHub: ChatMessage saved
    ConsultationHub-->>ParticipantApp: ReceiveMessage

    ParticipantApp->>ConsultationHub: Signal(eventType, payload)
    ConsultationHub-->>ParticipantApp: Signal
```

#### ***3.3.7 Sequence Diagram End Consultation and Settlement (Narrative)***

```mermaid
sequenceDiagram
    actor ParticipantApp
    participant ConsultationsController
    participant ConsultationService
    participant ConsultationPaymentService
    participant ConsultationLifecycleBackgroundService
    participant BookingService
    participant LiveKitService
    participant ConsultationRepository
    participant ConsultationBookingRepository
    participant ExpertTimeSlotRepository
    activate ConsultationsController
    activate ConsultationService
    activate ConsultationPaymentService
    activate ConsultationLifecycleBackgroundService
    activate BookingService
    activate LiveKitService
    activate ConsultationRepository
    activate ConsultationBookingRepository
    activate ExpertTimeSlotRepository

    alt Explicit end
        ParticipantApp->>ConsultationsController: POST /api/consultations/{consultationId}/end
        ConsultationsController->>ConsultationService: EndConsultationAsync(consultationId, actorId)
        ConsultationService->>ConsultationRepository: FirstOrDefaultAsync(c.Id == consultationId, asNoTracking:false)
        ConsultationService->>ConsultationRepository: Update(consultation.Status = Completed, EndTime = UtcNow)
        opt Linked scheduled booking exists
            ConsultationService->>ConsultationBookingRepository: FirstOrDefaultAsync(b.ConsultationId == consultationId, asNoTracking:false)
            ConsultationService->>ConsultationBookingRepository: Update(booking.Status = Completed)
            ConsultationService->>ExpertTimeSlotRepository: FirstOrDefaultAsync(s.Id == booking.TimeSlotId, asNoTracking:false)
            ConsultationService->>ExpertTimeSlotRepository: Update(slot.Status = Booked)
        end
        ConsultationService->>ConsultationPaymentService: SettleConsultationEscrowAsync(consultationId)
        ConsultationPaymentService-->>ConsultationService: ExpertPayout + PlatformFee persisted
        ConsultationService-->>ConsultationsController: End completed
        ConsultationsController-->>ParticipantApp: 200 OK
    else Lifecycle fallback - scheduled
        ConsultationLifecycleBackgroundService->>BookingService: AutoCompleteElapsedScheduledConsultationsAsync(stoppingToken)
        BookingService->>ConsultationRepository: Query confirmed bookings whose slot has elapsed and consultation not completed
        BookingService-->>ParticipantApp: RoomExpiring (SignalR group consultation:{consultationId})
        BookingService->>LiveKitService: DeleteRoomAsync(consultation-{consultationId})
        BookingService->>ConsultationRepository: Update(booking.Status = Completed, consultation.Status = Completed, consultation.EndTime = slot.EndTime)
        BookingService->>ConsultationPaymentService: SettleConsultationEscrowAsync(consultationId, cancellationToken)
        ConsultationPaymentService-->>BookingService: ExpertPayout + PlatformFee persisted
    else Lifecycle fallback - emergency
        ConsultationLifecycleBackgroundService->>BookingService: AutoCompleteElapsedEmergencyConsultationsAsync(stoppingToken)
        BookingService->>ConsultationRepository: Query ongoing emergency consultations older than 30 minutes
        BookingService-->>ParticipantApp: RoomExpiring (SignalR group consultation:{consultationId})
        BookingService->>LiveKitService: DeleteRoomAsync(consultation-{consultationId})
        BookingService->>ConsultationRepository: Update(consultation.Status = Completed, consultation.EndTime = UtcNow)
        BookingService->>ConsultationPaymentService: SettleConsultationEscrowAsync(consultationId, cancellationToken)
        ConsultationPaymentService-->>BookingService: ExpertPayout + PlatformFee persisted
    end
```

#### ***3.3.8 Sequence Diagram Create Consultation Review (Narrative)***

```mermaid
sequenceDiagram
    actor MemberApp
    participant ConsultationsController
    participant ConsultationService
    participant ConsultationRepository
    participant UserFeedbackRepository
    participant ExpertProfileRepository
    activate ConsultationsController
    activate ConsultationService
    activate ConsultationRepository
    activate UserFeedbackRepository
    activate ExpertProfileRepository

    MemberApp->>ConsultationsController: POST /api/consultations/{consultationId}/reviews
    ConsultationsController->>ConsultationService: CreateConsultationReviewAsync(consultationId, raterId, request)
    ConsultationService->>ConsultationRepository: FirstOrDefaultAsync(c.Id == consultationId, asNoTracking:false)
    ConsultationService->>UserFeedbackRepository: FirstOrDefaultAsync(existing consultation feedback check)
    ConsultationService->>UserFeedbackRepository: InsertAsync(new UserFeedback { Type = Consultation })
    UserFeedbackRepository-->>ConsultationService: UserFeedback
    ConsultationService->>ExpertProfileRepository: FirstOrDefaultAsync(AccountId == consultation.CalleeId, asNoTracking:false)
    ConsultationService->>ExpertProfileRepository: Update(RatingCount, Rating)
    ConsultationService-->>ConsultationsController: UserFeedbackResponse + UpdatedAverageRating + UpdatedRatingCount
    ConsultationsController-->>MemberApp: 200 OK
```
