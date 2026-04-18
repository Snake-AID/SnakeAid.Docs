# AGENTS.md

## Purpose

Quy ước ngắn gọn để viết tiếp unit test documentation cho flow consultation trong `Report 5`.

Áp dụng cho:
- sheet `function`
- từng sheet `testcase`

Nguồn format chuẩn:
- `SnakeAid Report5_Unit Test.xlsx - nháp phần funtion.csv`
- `SnakeAid Report5_Unit Test.xlsx - CreateSnakebiteIncident.csv`
- `1 ViewExperts.csv` là template trình bày chuẩn cho các testcase sheet trong thư mục này

## Core Rule

Luôn bám format workbook mẫu hiện có.

Không được:
- đổi layout
- đổi tên cột chính
- viết testcase theo kiểu narrative tự do
- gộp nhiều business action vào một testcase sheet nếu có thể tách độc lập

## File Naming

- function sheet luôn đặt tên là `Function.csv`
- testcase sheet đặt theo format: `<sheet order number> <TestCaseName>.csv`
- `<sheet order number>` là số dùng để sắp xếp theo đúng thứ tự sheet trong workbook gốc
- `TestCaseName` dùng tên function/business action ngắn gọn, ví dụ `1 ViewExperts.csv`
- giá trị trong cột `Sheet Name` vẫn là tên sheet logic, không cần kèm `.csv`

## 1. Function Sheet Rules

### Required columns

Giữ đúng thứ tự:
- `No`
- `Requirement Name`
- `Class Name`
- `Function Name`
- `Function Code(Optional)`
- `Sheet Name`
- `Description`
- `Pre-Condition`

### Writing rules

- Mỗi dòng là một function nghiệp vụ có thể test độc lập
- `Description` viết theo business action, ví dụ: `Member view the expert directory`
- `Pre-Condition` viết dạng bullet, mỗi dòng bắt đầu bằng `-`
- `Sheet Name` phải map rõ sang sheet testcase
- `Function Code` phải duy nhất

### Consultation naming

Ưu tiên tên ngắn, rõ, theo business step:
- `ViewExperts`
- `CreateScheduledBooking`
- `PayScheduledBooking`
- `CreateEmergencyConsultationRequest`
- `PayEmergencyConsultationRequest`
- `AcceptEmergencyConsultationRequest`
- `RejectEmergencyConsultationRequest`
- `JoinConsultationRoom`
- `SendConsultationMessage`
- `EndConsultation`
- `CreateConsultationReview`

Không dùng tên quá technical nếu chưa thật sự cần.

## 2. Testcase Sheet Rules

### One sheet per function

Mỗi testcase sheet chỉ đại diện cho một function.

### Required header fields

Phải có:
- `Function Code`
- `Function Name`
- `Created By`
- `Executed By`
- `Lines of code`
- `Lack of test cases`
- `Test requirement`
- `Passed`
- `Failed`
- `Untested`
- `N/A/B`
- `Total Test Cases`

### Default header values

- `Function Name`: luôn phải điền
- `Executed By`: mặc định `KhiemNVD`

### CSV presentation rule

- Lấy `1 ViewExperts.csv` làm chuẩn trình bày trực tiếp
- Các testcase sheet mới phải bắt chước format của file này
- Chỉ thay nội dung testcase, không tự phát minh layout mới
- Nếu có mâu thuẫn giữa rule mô tả và `1 ViewExperts.csv`, ưu tiên `1 ViewExperts.csv`

### Matrix layout

Mỗi testcase là một cột:
- `UTCID01`
- `UTCID02`
- `UTCID03`
- ...

Các dòng điều kiện/expected result nằm bên trái và dùng `O` để đánh dấu testcase cover dòng đó.

### Left-side layout rule

- Phần body phải theo cùng nhịp cột như `1 ViewExperts.csv`
- Mẫu tham chiếu:
  - `Condition,Precondition,,,,,,,`
  - `,,Can connect with server,O,O,...`
  - `,PageNumber,,,,,,,`
  - `,,1,O,O,...`
  - `Confirm,Return,,,,,,,`
  - `Result,,"Type(N : Normal, A : Abnormal, B : Boundary)",...`
- Không đổi sang layout mới dù có vẻ gọn hơn

### Expected structure

Giữ thứ tự logic:
- metadata header
- hàng `UTCIDxx`
- `Condition`
- `Precondition`
- input/query/body fields
- `Confirm/Return`
- `Exception`
- `Log message`
- `Result`

## 3. Content Rules

### Focus level

Chỉ ghi những gì có ý nghĩa ở mức API/business flow:
- auth / role nếu có
- request validity
- resource existence
- valid status
- filter / sort / paging
- payment state
- room join eligibility
- review eligibility

Không sa vào internal implementation detail.

### Actor clarity

Actor phải rõ:
- `Member`
- `Expert`
- `Admin`
- `Participant`

### Sequence-first decomposition

Các function phải follow sequence diagram theo flow.

Một sequence diagram có thể sinh ra nhiều testcase sheet.

Không ép một sequence diagram thành một sheet duy nhất.

## 4. Result Rules

### Type meanings

- `N` = Normal
- `A` = Abnormal
- `B` = Boundary

### Default execution rules

Cho report hiện tại, mặc định:
- tất cả testcase đều `Passed`
- `Passed/Failed` luôn là `P` cho mọi testcase
- `Executed Date` là ngày ngẫu nhiên từ `04/06` trở đi
- `Defect ID` để trống
- summary phải phản ánh đúng:
  - `Passed = Total Test Cases`
  - `Failed = 0`
  - `Untested = 0`

### Result section

Phải luôn có các dòng:
- `Type(N : Normal, A : Abnormal, B : Boundary)`
- `Passed/Failed`
- `Executed Date`
- `Defect ID`

## 5. Minimum Coverage Rule

Mỗi function nên có:
- ít nhất 1 normal case
- abnormal cases cho validation / invalid state / invalid input nếu phù hợp
- boundary cases nếu field có range hoặc limit rõ ràng

## 6. Final Checklist

Trước khi hoàn thành một function/testcase sheet mới, kiểm tra:

- [ ] function đã có dòng trong sheet `function`
- [ ] `Function Code` là duy nhất
- [ ] `Sheet Name` khớp testcase sheet
- [ ] tên file đúng format `Function.csv` hoặc `<number> <TestCaseName>.csv`
- [ ] header testcase có `Function Name`
- [ ] `Executed By` là `KhiemNVD`
- [ ] format testcase sheet bám theo `1 ViewExperts.csv`
- [ ] body vẫn giữ ma trận cũ, không bị redesign
- [ ] có cột `UTCIDxx`
- [ ] có đủ normal / abnormal / boundary khi phù hợp
- [ ] result section có đủ `Passed/Failed`, `Executed Date`, `Defect ID`
- [ ] tất cả testcase đang `P`
- [ ] summary trên cùng khớp tổng số testcase

## Default Decision

Nếu có mâu thuẫn giữa:
- format đẹp hơn
- format giống workbook mẫu hơn

thì chọn format giống workbook mẫu hơn.
