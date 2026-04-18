# roadmap.md

## Purpose

File này là nguồn sự thật để triển khai toàn bộ `Consultation_UnitTest` mà không cần phụ thuộc vào lịch sử hội thoại.

Dùng file này để:
- biết toàn bộ phạm vi công việc
- theo dõi tiến độ hiện tại
- xác định sheet number tiếp theo
- resume công việc sau này

## Source Files

- Sequence source: [SnakeAid_Report4 SequenceDiagram SnakeExpertConsultation.md](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 4/Sequence Diagram/Consultation/SnakeAid_Report4 SequenceDiagram SnakeExpertConsultation.md)
- Rule file: [AGENTS.md](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/Consultation_UnitTest/AGENTS.md)
- Function sheet: [Function.csv](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/Consultation_UnitTest/Function.csv)

## Naming Convention

- Function sheet: `Function.csv`
- Testcase sheet: `<number> <TestCaseName>.csv`
- `number` dùng để sort theo thứ tự sheet trong workbook gốc

## Progress Legend

- `TODO`: chưa làm
- `IN PROGRESS`: đang làm
- `DONE`: đã tạo file testcase sheet
- `BLOCKED`: tạm dừng do thiếu thông tin hoặc cần xác minh thêm

## Current Status

- Current phase: `3.3 Consultation`
- Current progress: `12 testcase sheets done`
- Last completed sheet: `12 NotifyEmergencyRequest.csv`
- Next recommended sheet: `13 AcceptEmergencyConsultationRequest`

## Execution Rules

- Luôn follow `AGENTS.md`
- Decompose theo sequence flow, không ép mỗi sequence thành một sheet duy nhất
- Mỗi business function có 1 testcase sheet riêng
- Sau mỗi lần tạo sheet mới:
  - cập nhật `Function.csv`
  - tạo file testcase theo naming convention
  - cập nhật trạng thái trong roadmap này

## Master Plan

### 3.3.2 Sequence Diagram View List Experts and Presence

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 1 | ViewExperts | `1 ViewExperts.csv` | DONE | `GET /api/experts` |
| 2 | ExpertPresence | `2 ExpertPresence.csv` | DONE | `JoinAsMember`, `JoinAsExpert`, presence update |

### 3.3.3 Sequence Diagram Create and Pay Scheduled Booking

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 3 | CreateScheduledBooking | `3 CreateScheduledBooking.csv` | DONE | create booking |
| 4 | PayScheduledBookingWithWallet | `4 PayScheduledBookingWithWallet.csv` | DONE | wallet branch |
| 5 | PayScheduledBookingWithPayOS | `5 PayScheduledBookingWithPayOS.csv` | DONE | PayOS branch |
| 6 | ConfirmScheduledBookingPayment | `6 ConfirmScheduledBookingPayment.csv` | DONE | confirm / webhook completion |

### 3.3.4 Sequence Diagram Create, Pay, and Notify Emergency Consultation Request

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 7 | CreateEmergencyConsultationRequest | `7 CreateEmergencyConsultationRequest.csv` | DONE | create request |
| 8 | JoinEmergencyRequestRoom | `8 JoinEmergencyRequestRoom.csv` | DONE | member joins room |
| 9 | PayEmergencyRequestWithWallet | `9 PayEmergencyRequestWithWallet.csv` | DONE | wallet branch |
| 10 | PayEmergencyRequestWithPayOS | `10 PayEmergencyRequestWithPayOS.csv` | DONE | PayOS branch |
| 11 | ConfirmEmergencyRequestPayment | `11 ConfirmEmergencyRequestPayment.csv` | DONE | confirm / webhook completion |
| 12 | NotifyEmergencyRequest | `12 NotifyEmergencyRequest.csv` | DONE | send request to expert |

### 3.3.5 Sequence Diagram Expert Accept or Reject Emergency Request

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 13 | AcceptEmergencyConsultationRequest | `13 AcceptEmergencyConsultationRequest.csv` | TODO | expert accepts |
| 14 | RejectEmergencyConsultationRequest | `14 RejectEmergencyConsultationRequest.csv` | TODO | expert rejects |
| 15 | ExpireEmergencyConsultationRequest | `15 ExpireEmergencyConsultationRequest.csv` | TODO | timeout sweep |
| 16 | NotifyEmergencyRequestStatusChanged | `16 NotifyEmergencyRequestStatusChanged.csv` | TODO | accepted / declined / expired notification |

### 3.3.6 Sequence Diagram Join Consultation Room and In-Room Interaction

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 17 | GenerateConsultationVideoToken | `17 GenerateConsultationVideoToken.csv` | TODO | video token |
| 18 | JoinConsultationRoom | `18 JoinConsultationRoom.csv` | TODO | SignalR room connect |
| 19 | UploadConsultationAttachment | `19 UploadConsultationAttachment.csv` | TODO | optional media upload |
| 20 | SendConsultationMessage | `20 SendConsultationMessage.csv` | TODO | in-room message |
| 21 | SendConsultationSignal | `21 SendConsultationSignal.csv` | TODO | signal event |

### 3.3.7 Sequence Diagram End Consultation and Settlement

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 22 | EndConsultation | `22 EndConsultation.csv` | TODO | explicit end |
| 23 | SettleConsultationEscrow | `23 SettleConsultationEscrow.csv` | TODO | payout settlement |
| 24 | AutoCompleteScheduledConsultation | `24 AutoCompleteScheduledConsultation.csv` | TODO | lifecycle fallback scheduled |
| 25 | AutoCompleteEmergencyConsultation | `25 AutoCompleteEmergencyConsultation.csv` | TODO | lifecycle fallback emergency |

### 3.3.8 Sequence Diagram Create Consultation Review

| No. | Function | Planned File | Status | Notes |
| --- | --- | --- | --- | --- |
| 26 | CreateConsultationReview | `26 CreateConsultationReview.csv` | TODO | create review |
| 27 | UpdateExpertRatingAfterReview | `27 UpdateExpertRatingAfterReview.csv` | TODO | rating update |

## Resume Procedure

Khi resume:
1. Mở `roadmap.md`
2. Tìm dòng đầu tiên có trạng thái `TODO`
3. Đọc sequence tương ứng trong file sequence source
4. Cập nhật `Function.csv`
5. Tạo testcase sheet theo đúng số thứ tự đã định
6. Đổi trạng thái trong roadmap sang `DONE`
7. Cập nhật `Last completed sheet` và `Next recommended sheet`

## Update Log

- `2026-04-18`: Created roadmap and initialized full Consultation Unit Test Sheet plan.
- `2026-04-18`: Confirmed completed sheet `1 ViewExperts.csv`.
- `2026-04-18`: Completed sheet `2 ExpertPresence.csv`.
- `2026-04-18`: Completed sheet `3 CreateScheduledBooking.csv`.
- `2026-04-18`: Completed phase `3.3.3` with sheets `4 PayScheduledBookingWithWallet.csv`, `5 PayScheduledBookingWithPayOS.csv`, and `6 ConfirmScheduledBookingPayment.csv`.
- `2026-04-18`: Completed phase `3.3.4` with sheets `7 CreateEmergencyConsultationRequest.csv` to `12 NotifyEmergencyRequest.csv`.
