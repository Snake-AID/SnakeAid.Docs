# Snake Catching Business ERD

## ERD Diagram
```mermaid
erDiagram
  PATIENT {}
  RESCUER {}
  EXPERT {}
  SNAKE_SPECIES {}
  INCIDENT {}
  SYMPTOM_TRACK {}
  MEDIA {}
  RESCUE_MISSION {}
  RESCUE_STATUS {}
  RESCUE_TEAM {}
  PRICING_RULE {}
  DISPUTE {}
  RATING {}
  FACILITY {}
  CONSULTATION {} 
  PAYMENT {}

  PATIENT ||--o{ INCIDENT : bao_cao
  INCIDENT }o--|| SNAKE_SPECIES : xac_nhan_or_du_doan
  INCIDENT ||--o{ SYMPTOM_TRACK : theo_doi
  INCIDENT ||--o{ MEDIA : bang_chung_su_co
  INCIDENT }o--|| FACILITY : huong_dan_co_so_y_te

  INCIDENT ||--o{ RESCUE_MISSION : kich_hoat
  RESCUER ||--o{ RESCUE_MISSION : thuc_hien
  RESCUE_TEAM ||--o{ RESCUE_MISSION : phan_cong_bo_sung
  RESCUE_MISSION ||--o{ RESCUE_STATUS : tien_trinh_gps
  RESCUE_MISSION ||--o{ MEDIA : bang_chung_cuu_ho
  PRICING_RULE ||--o{ RESCUE_MISSION : ap_dung_phi
  RESCUE_MISSION }o--|| PAYMENT : thanh_toan_cuu_ho
  RESCUE_MISSION ||--o{ DISPUTE : tranh_chap_or_hoan_phi

  PATIENT ||--o{ CONSULTATION : yeu_cau
  EXPERT ||--o{ CONSULTATION : tu_van
  CONSULTATION ||--o{ MEDIA : ho_tro_chan_doan
  CONSULTATION }o--|| PAYMENT : phi_tu_van

  PATIENT ||--o{ PAYMENT : khoi_tao
  RESCUER ||--o{ PAYMENT : nhan_cuu_ho
  EXPERT ||--o{ PAYMENT : nhan_tu_van
  PAYMENT ||--o{ DISPUTE : yeu_cau_hoan

  PATIENT ||--o{ RATING : danh_gia
  RESCUER ||--o{ RATING : duoc_danh_gia
  EXPERT ||--o{ RATING : duoc_danh_gia
```

## Explanation
- **Báo sự cố & xác định loài**: `PATIENT` tạo `INCIDENT`; hệ thống xác nhận/ước đoán `SNAKE_SPECIES`, kèm `MEDIA` (ảnh/video) và `SYMPTOM_TRACK` để theo dõi diễn biến; `FACILITY` gợi ý cơ sở y tế phù hợp.
- **Kích hoạt và điều phối cứu hộ**: `INCIDENT` kích hoạt `RESCUE_MISSION`; `RESCUER` là người chính, `RESCUE_TEAM` bổ sung khi cần; `RESCUE_STATUS` ghi nhận tiến trình/tọa độ GPS; `MEDIA` lưu bằng chứng cứu hộ; `PRICING_RULE` áp dụng phí; `PAYMENT` xử lý thanh toán; `DISPUTE` mở khi có tranh chấp hoặc hoàn phí.
- **Tư vấn chuyên gia từ xa**: `PATIENT` yêu cầu `CONSULTATION`; `EXPERT` cung cấp tư vấn; `MEDIA` hỗ trợ chẩn đoán; `PAYMENT` giữ và giải phóng phí tư vấn; `DISPUTE` dùng chung cho yêu cầu hoàn/điều chỉnh.
- **Thanh toán & hoàn phí**: `PATIENT` khởi tạo `PAYMENT`; `RESCUER` nhận phí cứu hộ, `EXPERT` nhận phí tư vấn; `DISPUTE` cho phép yêu cầu hoàn hoặc điều chỉnh khi phát sinh khiếu nại.
- **Đánh giá chất lượng**: `PATIENT` tạo `RATING` cho `RESCUER` và `EXPERT` sau khi dịch vụ hoàn tất để phục vụ xếp hạng và ưu tiên điều phối.

## References
- [README.md](README.md)
- [Docs/00-Introduction/Introduction.md](Docs/00-Introduction/Introduction.md)
- [Docs/02-Architecture-Design/Context-Diagram.md](Docs/02-Architecture-Design/Context-Diagram.md)
