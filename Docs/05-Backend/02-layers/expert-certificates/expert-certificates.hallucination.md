# Expert Certificates Hallucination

## Purpose

File này không dùng để chốt thiết kế.

File này dùng để gom các điểm còn chưa chốt, các risk chưa được lock, và các ambiguity cần research tiếp trước khi được nâng cấp thành decision trong:

- `expert-certificates.introduction.md`
- `expert-certificates.roadmap.md`
- `expert-certificates.sourcecode.md`
- `expert-certificates.useguide.md`

Nguyên tắc:

- research trước
- chốt decision sau
- chỉ đẩy kết luận đã đủ chắc về bộ docs chính

## Current Direction Summary

Current planned direction đã tương đối rõ:

- thêm `MediaReferenceType.CertExpert`
- thêm `ExpertProfile.IsVerified`
- xây CRUD cho `ExpertCertificate`
- admin route dùng `/api/admin/expert/certificates`

Nhưng vẫn còn một số điểm chưa được khóa hoàn toàn ở mức implementation detail và contract detail.

## Risk 1. Attachment Source Of Truth

### Current ambiguity

Chưa chốt certificate attachment sẽ lấy nguồn dữ liệu chính từ đâu:

- `ExpertCertificate.CertificateUrl`
- hay `ReportMedia` với `ReferenceId + MediaReferenceType`

### Why this matters

Nếu không khóa sớm, implementation rất dễ rơi vào trạng thái:

- ghi URL vào `ExpertCertificate`
- đồng thời cũng tạo `ReportMedia`
- nhưng không có rule rõ cái nào là nguồn dữ liệu thật

### Recommended default

- trong phase hiện tại, giữ `ExpertCertificate.CertificateUrl` là source of truth
- `ReportMedia` chỉ đóng vai trò taxonomy support nếu thực sự cần media flow riêng

## Risk 2. Meaning Of `ReferenceId` For `CertExpert`

### Current ambiguity

Nếu có dùng `ReportMedia` cho certificate flow, `ReferenceId` sẽ trỏ tới:

- `ExpertProfile.AccountId`
- hay `ExpertCertificate.Id`

### Why this matters

Quyết định này ảnh hưởng trực tiếp tới:

- upload order
- list/query logic
- delete cleanup logic
- khả năng tìm media theo expert hay theo certificate record

### Recommended default

- nếu certificate media cần bám theo từng certificate record, ưu tiên `ExpertCertificate.Id`
- nếu chỉ cần media bucket ở cấp expert profile, mới cân nhắc `ExpertProfile.AccountId`

## Risk 3. Upload Workflow Ordering

### Current ambiguity

Chưa khóa luồng nào là chính:

- upload file trước rồi tạo `ExpertCertificate`
- hay tạo `ExpertCertificate` trước rồi mới upload/attach file

### Why this matters

Với `ReportMedia` theo kiểu soft reference:

- upload trước dễ sinh orphan media nếu chưa có `ReferenceId` cuối cùng
- tạo record trước giúp attach rõ hơn, nhưng làm client flow thành nhiều bước

### Recommended default

- nếu chỉ dùng `CertificateUrl`, cho phép upload file trước rồi gửi URL vào create certificate
- nếu dùng `ReportMedia` như attachment registry thật, nên tạo `ExpertCertificate` trước để có `ReferenceId` ổn định

## Risk 4. Cleanup Policy

### Current ambiguity

Chưa chốt khi xóa certificate thì:

- có xóa media liên quan hay không
- nếu có, xóa đồng bộ hay best-effort cleanup

### Why this matters

Do `ReportMedia` không có FK relation:

- xóa `ExpertCertificate` không tự kéo theo cleanup media
- nếu không có service-level cleanup rule, media cũ sẽ bị orphan

### Recommended default

- nếu phase hiện tại chỉ dùng `CertificateUrl`, chưa bắt buộc cleanup `ReportMedia`
- nếu phase sau dùng `ReportMedia` thật, phải thêm explicit cleanup policy trong service

## Risk 5. Verification Recalculation Edge Cases

### Current ambiguity

Rule tổng quát đã có:

- có ít nhất một cert `Verified` thì `ExpertProfile.IsVerified = true`

Nhưng edge cases chưa khóa:

- expert sửa cert đã `Verified` thì có reset về `Pending` không
- admin đổi `Verified -> Rejected` thì recalculation chạy ra sao
- xóa cert đã verified cuối cùng thì khi nào cập nhật `IsVerified`

### Why this matters

Nếu không khóa rõ, `ExpertProfile.IsVerified` rất dễ thành snapshot stale.

### Recommended default

- mọi thay đổi material do expert thực hiện trên cert đã verified nên reset về `Pending`
- mọi path create, update, verify, reject, delete đều phải gọi một hàm recalculation tập trung

## Risk 6. Self Profile Contract Exposure

### Current ambiguity

Hiện public expert response đã có `isVerified`, nhưng self profile chưa có.

Điểm chưa chốt:

- phase này có mở rộng `ExpertMyProfileResponse` để thêm `isVerified` luôn không
- hay chỉ sửa public expert listing/detail trước

### Why this matters

Nếu admin review certificate là user-visible state mà self profile vẫn không expose `isVerified`, mobile app sẽ phải suy luận gián tiếp từ API khác.

### Recommended default

- thêm `isVerified` vào `ExpertMyProfileResponse` ngay trong cùng phase

## Risk 7. Admin CRUD Scope

### Current ambiguity

Admin CRUD đã có route direction, nhưng business scope chưa khóa:

- admin có được create certificate thay expert không
- hay admin chỉ được review/update/delete certificate đã do expert tạo

### Why this matters

Điểm này làm thay đổi:

- request contract admin create
- ownership assumptions
- audit trail semantics
- cách mobile/admin UI hiển thị flow nhập cert

### Recommended default

- nếu muốn scope chặt và ít ambiguity hơn, phase đầu chỉ cho admin review/update/delete
- chỉ giữ admin create nếu có nhu cầu vận hành rõ ràng

## Promotion Rule

Chỉ khi một risk ở file này đã được chốt đủ rõ:

- option được chọn
- impact được hiểu
- edge cases chính đã được khóa

thì mới chuyển nội dung tương ứng sang các file decision chính.

