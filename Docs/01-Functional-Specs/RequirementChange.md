**BIÊN BẢN CUỘC HỌP – Thống nhất luồng vận hành hệ thống cứu hộ rắn cắn**

**1\. Mục tiêu cuộc họp**  
Thống nhất điều chỉnh mô hình vận hành và luồng xử lý sự cố trong hệ thống cứu hộ rắn cắn.

---

**2\. Mô hình người dùng và vai trò**

**2.1 Member (người dùng)**

* Luồng sử dụng **giữ nguyên như hiện tại**.

**2.2 Rescuer (đội cứu hộ)**

* Là **nhân sự của trung tâm**, được **cấp tài khoản trực tiếp**.

* **Không thực hiện đăng ký tài khoản** trên hệ thống.

**2.3 Admin / Manager**

* Có vai trò quản lý hệ thống.

**2.4 Operator (điều phối)**

* Là người **điều phối đội cứu hộ** tại trung tâm.

* **Trực hệ thống web tại trung tâm** để tiếp nhận và xử lý các yêu cầu.

---

**3\. Luồng xử lý sự cố rắn cắn**

**3.1 Tiếp nhận yêu cầu**

1. Member gửi **Rescue Request**.

2. **Operator** là người **nhìn thấy request đầu tiên** trên hệ thống.

**3.2 Xác minh sự cố**

* Operator **liên hệ lại Member** để xác minh thông tin.

* Thêm **state mới cho Incident: VERIFY**.

**Luồng trạng thái:**

REQUEST CREATED

        ↓

VERIFY (Operator xác minh với Member)

        ↓

ASSIGN RESCUER

**3.3 Điều phối rescuer**

Sau khi VERIFY:

1. Operator **assign Rescuer**.

2. Kiểm tra:

   * Rescuer **đang online hay không**

   * Rescuer **có trong ca trực hay không**

**3.4 Quản lý ca trực rescuer**

* Cần có **lịch làm việc (shift)** của rescuer.

* Trong ca trực:

  * Rescuer **không bắt buộc ở trung tâm**

  * Nhưng **phải online trên app** để nhận thông tin.

Cân nhắc:

* **Push Notification** khi có nhiệm vụ mới.

**3.5 Monitoring rescuer**

* Operator có **bản đồ theo dõi rescuer online**.

* Thấy vị trí rescuer **trên map monitoring**.

---

**4\. Luồng Rescue Ping Request**

Sau khi VERIFY và assign:

VERIFY

   ↓

RESCUE PING REQUEST (PENDING)

   ↓

ACCEPT (Rescuer nhận nhiệm vụ)

   ↓

RESCUE MISSION

**Trường hợp khác**

* **Member CANCEL** → huỷ yêu cầu.

* **MISSION ABORT** → rescuer huỷ nhiệm vụ.

**Quy định**

* Khi **Abort Mission**, rescuer **phải cung cấp lý do huỷ**.

---

**5\. Luồng bắt rắn (Snake Capture)**

Luồng xử lý tương tự:

1. Request được gửi lên hệ thống.

2. Operator **tiếp nhận và xác nhận request**.

3. Sau khi **confirm từ Operator**, mới **điều rescuer thực hiện nhiệm vụ**.

---

**6\. Phạm vi hệ thống**

**6.1 Hiện tại**

* Hệ thống **chỉ phục vụ 1 trung tâm cứu hộ** sử dụng.

**6.2 Chi phí**

* Chi phí vụ việc được tính theo:

Khoảng cách (Km)

từ Trung tâm cứu hộ → vị trí sự cố

**6.3 Quản lý huyết thanh**

* **Loại bỏ module quản lý huyết thanh** khỏi hệ thống.

---

**7\. Phạm vi nghiệp vụ**

**7.1 Điều trị y tế**

* **Không thuộc phạm vi hệ thống**.

* Hệ thống **chỉ hỗ trợ cứu hộ ban đầu**.

**7.2 Tư vấn chuyên gia**

* Có thể có **chuyên gia về rắn từ bên ngoài** tham gia hệ thống để **tư vấn chuyên môn**.

**7.3 Quy định động vật hoang dã**

* Sau khi bắt rắn:

  * Rescuer **không được giữ lại**.

  * Phải **bàn giao cho cơ quan quản lý động vật hoang dã của Nhà nước**.


Rescue mission từ vị trí sos về map hay là về trung tâm là 2 option (có thêm default option là trung tâm)