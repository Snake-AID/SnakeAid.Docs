# Snake Catching Business ERD

## ERD Diagram
```mermaid
erDiagram
  USER {}
  PAYMENT_CARD {}
  SNAKEAID_WALLET {}
  PATIENT {}
  RESCUER {}
  EXPERT {}
  RATING {}
  PAYMENT {}
  PAYMENT_TRANSACTION {}
  DISPUTE {}
  PAYMENT_METHOD {}
  RESCUE_MISSION {}
  RESCUE_STATUS {}
  RESCUE_TEAM {}
  MEDIA {}
  SNAKE_REPORT {}
  SNAKE_SPECIES {}
  CONSULTATION {}
  CONSULTATION_NOTE {}
  SNAKE_AI_RECOGNITION {}
  RESCUE_TOOLS {}
  SAFETY_GUIDELINE {}
  SNAKE_QUANTITY {}
  SNAKE_PRICE_RULE {}
  RESCUE_CONDITION {}
  SNAKE_PRICE_OUTPUT {}
  REVENUE_SHARE_CONFIG {}

  USER ||--o{ PAYMENT_CARD : luu_the
  USER ||--o{ SNAKEAID_WALLET : vi_he_thong
  PATIENT }o--|| USER : system_account
  RESCUER }o--|| USER : system_account
  EXPERT }o--|| USER : system_account

  PATIENT ||--o{ RATING : danh_gia
  RESCUER ||--o{ RATING : duoc_danh_gia
  EXPERT ||--o{ RATING : duoc_danh_gia

  PATIENT ||--o{ SNAKE_REPORT : bao_cao
  SNAKE_REPORT }o--|| SNAKE_SPECIES : xac_nhan_or_du_doan
  SNAKE_AI_RECOGNITION }o--o{ SNAKE_REPORT : nhan_dien_ai
  SNAKE_REPORT ||--o{ MEDIA : bang_chung_su_co

  PATIENT ||--o{ CONSULTATION : yeu_cau
  EXPERT ||--o{ CONSULTATION : tu_van
  CONSULTATION ||--o{ CONSULTATION_NOTE : chuyen_gia_ghi_chu
  CONSULTATION ||--o{ MEDIA : ho_tro_chan_doan
  CONSULTATION ||--o{ SNAKE_REPORT : tu_van
  PAYMENT ||--o{ CONSULTATION : phi_tu_van

  PATIENT ||--o{ PAYMENT : khoi_tao
  RESCUER ||--o{ PAYMENT : nhan_cuu_ho
  EXPERT ||--o{ PAYMENT : nhan_tu_van
  PAYMENT }o--|| PAYMENT_TRANSACTION : thanh_toan

  SNAKE_REPORT ||--o{ RESCUE_MISSION : kich_hoat
  RESCUER ||--o{ RESCUE_MISSION : thuc_hien
  RESCUE_TEAM ||--o{ RESCUE_MISSION : phan_cong_bo_sung
  RESCUE_MISSION ||--o{ RESCUE_STATUS : tien_trinh_gps
  RESCUE_MISSION ||--o{ MEDIA : bang_chung_cuu_ho
  RESCUE_MISSION }o--|| PAYMENT_METHOD : lua_chon_phuong_thuc
  RESCUE_MISSION ||--o{ PAYMENT : thanh_toan
  RESCUE_MISSION ||--o{ DISPUTE : tranh_chap_or_hoan_phi
  RESCUE_MISSION ||--o{ SNAKE_QUANTITY : xac_dinh_so_luong_ran
  PAYMENT ||--o{ DISPUTE : yeu_cau_hoan

  RESCUE_CONDITION ||--o{ SNAKE_PRICE_RULE : dieu_kien
  RESCUE_CONDITION ||--o{ SNAKE_PRICE_OUTPUT : ket_qua_gia
  RESCUE_MISSION ||--o{ RESCUE_CONDITION : gan_luat_hop_le
  REVENUE_SHARE_CONFIG ||--o{ SNAKE_PRICE_OUTPUT : bang_phi
  SNAKE_SPECIES ||--o{ SNAKE_PRICE_RULE : dieu_kien
```




## Explanation
- **Báo cáo & xác định loài**: `PATIENT` tạo `SNAKE_REPORT`; hệ thống xác nhận/ước đoán `SNAKE_SPECIES` và lưu `MEDIA` chứng cứ sự cố (có thể kèm `SNAKE_AI_RECOGNITION`).
- **Kích hoạt cứu hộ**: `SNAKE_REPORT` kích hoạt `RESCUE_MISSION`; `RESCUER` thực hiện, `RESCUE_TEAM` bổ sung; theo dõi `RESCUE_STATUS`; lưu `MEDIA` cứu hộ; chọn `PAYMENT_METHOD`; `PAYMENT` thanh toán nhiệm vụ; `DISPUTE` xử lý tranh chấp/hoàn phí; `SNAKE_QUANTITY` ghi nhận số lượng thực tế; `RESCUE_TOOLS` và `SAFETY_GUIDELINE` gắn hướng dẫn/dụng cụ.
- **Tư vấn từ xa**: `PATIENT` yêu cầu `CONSULTATION`; `EXPERT` tư vấn; `MEDIA` hỗ trợ chẩn đoán; `CONSULTATION_NOTE` ghi chú phiên; `PAYMENT` giữ/giải phóng phí tư vấn.
- **Thanh toán & ví**: `USER` liên kết `PAYMENT_CARD` và `SNAKEAID_WALLET`; `PAYMENT_TRANSACTION` ghi giao dịch, gắn với `PAYMENT` và `RESCUE_MISSION`.
- **Đánh giá**: `PATIENT` tạo `RATING` cho `RESCUER` và `EXPERT` sau dịch vụ.
- **Bảng giá linh hoạt**: `SNAKE_PRICE_RULE` + `RESCUE_CONDITION` + `SNAKE_PRICE_OUTPUT`, áp dụng cho `RESCUE_MISSION`; tỷ lệ chia doanh thu cấu hình trong `REVENUE_SHARE_CONFIG`.

## References
- [README.md](README.md)
- [Docs/00-Introduction/Introduction.md](Docs/00-Introduction/Introduction.md)
- [Docs/02-Architecture-Design/Context-Diagram.md](Docs/02-Architecture-Design/Context-Diagram.md)
