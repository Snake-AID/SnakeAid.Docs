Bản cập nhật với terminology mới:

| # | Mô tả | URL cũ | URL mới |
|---|-------|--------|---------|
| 1 | Tạo đặt lịch | `POST api/consultation-bookings` | `POST api/consultations/scheduled` |
| 2 | Lấy lịch hẹn của user | `GET api/users/me/consultation-bookings` | `GET api/users/me/consultations/scheduled` |
| 3 | Lấy lịch hẹn của expert | `GET api/experts/me/consultation-bookings` | `GET api/experts/me/consultations/scheduled` |
| 4 | Thanh toán lịch hẹn | `POST api/consultation-bookings/{id}/payments` | `POST api/consultations/scheduled/{id}/payments` |
| 5 | Tạo tư vấn ngay | `POST api/consultations/emergency-requests` | `POST api/consultations/instant` |
| 6 | Expert accept | `POST api/consultations/emergency-requests/{id}/accept` | `POST api/consultations/instant/{id}/accept` |
| 7 | Expert reject | `POST api/consultations/emergency-requests/{id}/reject` | `POST api/consultations/instant/{id}/reject` |
| 8 | Thanh toán tư vấn ngay | `POST api/consultations/emergency-requests/{id}/payments` | `POST api/consultations/instant/{id}/payments` |
| 9 | Xác nhận thanh toán | `POST api/consultation-payments/confirm-payment` | `POST api/consultations/payments/confirm` |
| 10 | Kết thúc consultation | `POST api/consultations/{id}/end` | *(giữ nguyên)* |
| 11 | Lấy consultation của user | `GET api/users/me/consultations` | *(giữ nguyên)* |
| 12 | Tạo review | `POST api/consultations/{id}/reviews` | *(giữ nguyên)* |
| 13 | Xem review | `GET api/consultations/{id}/reviews` | *(giữ nguyên)* |
| 14 | Lấy video token | `POST api/videocall/livekit-token/{id}` | `POST api/consultations/{id}/video-token` |

Pattern rõ ràng hơn:
- `api/consultations/scheduled/*` — flow đặt lịch
- `api/consultations/instant/*` — flow tư vấn ngay
- `api/consultations/{id}/*` — thao tác trên consultation đã tạo (end, reviews, video-token)
- `api/{role}/me/consultations/*` — tài nguyên cá nhân

Hai từ `scheduled` và `instant` đối xứng nhau, đọc URL là hiểu ngay flow nào.