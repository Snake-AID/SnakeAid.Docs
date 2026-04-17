---
doc_role: analysis
operation_id: 01-INIT-consulting
type: INIT
status: draft
created_at: 2026-03-05
---

# Consultation API Endpoints & Entity Graph Design

Based on the `Consultation Wireframe.md` and the Multi-Agent Design Review, here are the required API endpoints and their entity interactions.

## 1. Users ("Người dùng") Series

### 1.1. Consultation Homepage & General

**`GET /api/consultations/scheduled`**

- **Purpose**: Get current/upcoming and past bookings.
- **Entity Graph**: `Account` (Caller) -> `ConsultationBooking` (Include `Expert`, `ExpertProfile`, `TimeSlot`).
- **Filters**: By `Status` (PendingPayment, Confirmed) and `Consultation.Status`.

### 1.2. Expert Directory

**`GET /api/v1/experts`**

- **Purpose**: Get list of verified experts with filters.
- **Entity Graph**: `ExpertProfile` (Include `Account`, `Specializations.Specialization`).
- **Response Data**: Avatar, Name, Specializations, Verification Status, Rating, ConsultationFee, IsOnline.

**`GET /api/v1/experts/{expertId}`**

- **Purpose**: Get detailed expert profile.
- **Entity Graph**: `ExpertProfile` (Include `Account`, `Specializations`, `ExpertCertificate`).

**`GET /api/v1/experts/{expertId}/reviews`**

- **Purpose**: Get expert reviews.
- **Entity Graph**: `UserFeedback` where `TargetId == expertId` AND `TargetType == Expert`. Include `Account` (Reviewer).

### 1.3. Booking & Availability

**`GET /api/v1/experts/{expertId}/time-slots`**

- **Purpose**: Get available slots for booking.
- **Entity Graph**: `ExpertTimeSlot` where `Status == Available` and `StartTime > Now`.

**`POST /api/consultations/scheduled`**

- **Purpose**: Book a scheduled consultation.
- **Request Payload**: `ExpertId`, `TimeSlotId`, `ProblemDescription` (Note: DB schema update required for note).
- **Entity Graph**: Read `ExpertTimeSlot` (Optimistic Concurrency check via `Version`) -> Create `ConsultationBooking` (Status = PendingPayment).

**`POST /api/v1/consultations/emergency`**

- **Purpose**: Request an immediate (emergency) consultation.
- **Entity Graph**: Create `ConsultationPingRequest`. The request is then broadcast via `ExpertHub` to online experts.

### 1.4. In-Room & Actions

**`POST /api/videocall/livekit-token/{consultationId}`** (Existing LiveKit Integration)

- **Purpose**: Get WebRTC Token to join the video room via LiveKit Cloud.
- **Entity Graph**: Verifies `Consultation` ownership. Uses `VideoCallController`.

**`POST /api/v1/consultations/{consultationId}/end`**

- **Purpose**: End the consultation.
- **Entity Graph**: Update `Consultation` (Status -> Completed, EndTime = Now). Free up corresponding resources.

**`POST /api/v1/consultations/{consultationId}/reviews`**

- **Purpose**: Submit post-consultation rating.
- **Entity Graph**: Create `UserFeedback`.

---

## 2. Experts ("Chuyên gia") Series

### 2.1. Settings & Schedule

**`PUT /api/v1/experts/me/settings`**

- **Purpose**: Update consultation fees (Live and Scheduled) and bio.
- **Entity Graph**: Update `ExpertProfile`.

**`POST /api/v1/experts/me/time-slots/bulk`**

- **Purpose**: Create available time slots in bulk for a specific week based on the expert's schedule configuration. Supports dynamic, non-fixed time blocks per day.
- **Entity Graph**: Create multiple `ExpertTimeSlot` records. Overwrites or merges with existing available slots for the specified date range.

### 2.2. Emergency Requests

**`POST /api/consultations/instant/{requestId}/accept`**

- **Purpose**: Expert accepts incoming emergency ping.
- **Entity Graph**:
  - Update `ConsultationPingRequest`.
  - Create `Consultation` (Status -> Ongoing).
  - _Slot Paradox Handling_: Trigger domain event to mark overlapping `ExpertTimeSlot` as `Reserved`.

**`POST /api/consultations/instant/{requestId}/reject`**

- **Purpose**: Reject the ping.
- **Entity Graph**: Update `ConsultationPingRequest`.

### 2.3. Consultation Tools

**`GET /api/snake-species/search`**

- **Purpose**: "Tra cứu rắn" side-screen.
- **Entity Graph**: `SnakeSpecies` (Include `SpeciesVenom`, `SpeciesAntivenom`).

---

## 3. SignalR Hubs Architecture

Following the Separation of Concerns, we will use two distinct Hubs:

### 3.1. `ExpertHub` (`/hubs/expert`)

- **Purpose**: Global presence and emergency ping routing for experts.
- **Functions**:
  - `JoinAsExpert`: Registers connection, updates `ExpertProfile.IsOnline = true` in DB, maintains `ConnectedExperts` dictionary.
  - `OnDisconnectedAsync`: Updates `ExpertProfile.IsOnline = false`.
  - `EmergencyPing`: Broadcasts `ConsultationPingRequest` targeted to online, available experts.

### 3.2. `ConsultationHub` (`/hubs/consultation`)

- **Purpose**: Scoped room context for an active consultation session. Requires `?consultationId={id}` during connection.
- **Functions**:
  - Validates user (Caller/Callee logic based on `Consultation` lookup).
  - `ReceiveMessage`: Persists to `ChatMessage` and broadcasts to the room group.
  - `Signal`: Exchange real-time UI state (e.g., mic/cam toggled, typing indicators).
