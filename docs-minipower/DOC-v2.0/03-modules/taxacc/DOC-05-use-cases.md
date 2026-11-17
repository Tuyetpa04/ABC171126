# DOC-05 — Ca sử dụng Phân hệ Kết nối Cơ quan Thuế (TAXACC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TAXACC` — Kết nối Cơ quan Thuế & Đồng bộ HĐĐT  
> **Căn cứ tài liệu gốc:** `[mSMI] API Đăng ký, cập nhật tài khoản Thuế.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 10).

---

## 1. Danh sách Ca sử dụng (Use Cases Register)

| UC ID | Tên Use Case | Tác nhân chính | Mức ưu tiên | Trace (BR / FR) |
|:---:|---|:---:|:---:|:---:|
| **TAXACC-UC-001** | Khai báo tài khoản Cổng Thuế mới | ACT-001 (Kế toán), ACT-003 (Đại lý) | Must | TAXACC-BR-001, BR-002<br>TAXACC-FR-001 |
| **TAXACC-UC-002** | Sửa / Cập nhật tài khoản Cổng Thuế | ACT-001 (Kế toán) | Must | TAXACC-BR-001, BR-002<br>TAXACC-FR-001 |
| **TAXACC-UC-003** | Thiết lập cấu hình lịch tự động đồng bộ | ACT-001 (Kế toán) | Must | TAXACC-BR-004<br>TAXACC-FR-002 |
| **TAXACC-UC-004** | Kích hoạt đồng bộ hóa đơn thủ công | ACT-001 (Kế toán) | Must | TAXACC-BR-003<br>TAXACC-FR-003 |
| **TAXACC-UC-005** | Xem nhật ký và lịch sử đồng bộ CQT | ACT-001, ACT-002 | Should | TAXACC-BR-005<br>TAXACC-FR-004 |

---

## 2. Đặc tả Chi tiết Ca sử dụng

### 2.1. TAXACC-UC-001: Khai báo Tài khoản Cổng Thuế Mới
* **Actor**: ACT-001 (Kế toán viên), ACT-003 (Đại lý thuế).
* **Basic Path**:
  1. Người dùng truy cập menu "Kết nối Cơ quan thuế" -> Bấm nút **"Thêm tài khoản"**.
  2. Người dùng nhập: Tên doanh nghiệp, Mã số thuế (Username) và Mật khẩu đăng nhập Cổng HĐĐT của Tổng cục Thuế.
  3. Bấm nút **"Lưu & Đăng nhập"**.
  4. Hệ thống tải Captcha từ CQT, giải mã Captcha và gửi yêu cầu đăng nhập.
  5. CQT xác thực thành công (mã `00`) kèm JWT token phiên làm việc.
  6. Hệ thống mã hóa AES-256-CBC mật khẩu, lưu vào bảng `externalaccounts` (`TAXACC-BR-002`) và kích hoạt trạng thái "Đang kết nối".
  7. Hiển thị thông báo: *"Thêm tài khoản thuế thành công"*.
* **Alternate Path (Lỗi xác thực)**: Nếu CQT trả về mã lỗi (`01` đến `07`), hệ thống hiển thị thông điệp tương ứng (theo `TAXACC-BR-001`) và cho phép người dùng kiểm tra lại thông tin.

---

### 2.2. TAXACC-UC-002: Sửa / Cập nhật Tài khoản Cổng Thuế
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Người dùng mở danh sách tài khoản thuế -> Chọn tài khoản cần sửa -> Bấm **"Chỉnh sửa"**.
  2. Người dùng cập nhật mật khẩu mới (trường hợp doanh nghiệp vừa đổi mật khẩu trên trang của TCT).
  3. Bấm **"Kiểm tra & Cập nhật"**.
  4. Hệ thống tiến hành xác thực lại với CQT; nếu thành công sẽ mã hóa lại mật khẩu mới và cập nhật trạng thái hoạt động bình thường.

---

### 2.3. TAXACC-UC-003: Thiết lập Cấu hình Lịch Tự động Đồng bộ
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Tại màn hình cấu hình tài khoản thuế, người dùng mở tab **"Lịch đồng bộ tự động"**.
  2. Người dùng bật công tắc "Tự động đồng bộ Hóa đơn Mua vào" và "Tự động đồng bộ Hóa đơn Bán ra".
  3. Chọn khung giờ chạy nền (ví dụ: `01:00` sáng hàng ngày).
  4. Bấm **"Lưu cấu hình"**; hệ thống cập nhật các cờ `auto_sync_purchase`, `auto_sync_sold` và kích hoạt scheduler trong PM2 Worker (`TAXACC-BR-004`).

---

### 2.4. TAXACC-UC-004: Kích hoạt Đồng bộ Hóa đơn Thủ công
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán bấm nút **"Đồng bộ ngay"** trên thanh công cụ danh sách hóa đơn.
  2. Chọn chiều hóa đơn (Mua vào hoặc Bán ra) và chọn khoảng ngày cần cào bù (ví dụ: từ `01/01/2026` đến `31/03/2026`).
  3. Hệ thống kiểm tra khoảng thời gian: Tự động chia nhỏ thành các đợt $\le 31$ ngày (`TAXACC-BR-003`).
  4. Đẩy yêu cầu vào hàng đợi ưu tiên cao của RabbitMQ và hiển thị thanh tiến trình đồng bộ thời gian thực.
  5. Khi hoàn tất, hệ thống thông báo tổng số hóa đơn mới được lấy về kho.

---

### 2.5. TAXACC-UC-005: Xem Nhật ký và Lịch sử Đồng bộ CQT
* **Actor**: ACT-001 (Kế toán), ACT-002 (Kế toán trưởng).
* **Basic Path**:
  1. Người dùng truy cập tab **"Lịch sử đồng bộ"** của tài khoản thuế.
  2. Hệ thống tải danh sách các phiên cào từ bảng `externalaccountsyncs` (`TAXACC-BR-005`).
  3. Hiển thị chi tiết từng phiên: Thời gian bắt đầu/kết thúc, thời lượng chạy (`duration_ms`), số hóa đơn quét được, số hóa đơn tạo mới, trạng thái (`Thành công` hoặc `Lỗi`).
  4. Người dùng có thể nhấp vào dòng lỗi để xem thông điệp chi tiết trả về từ Cổng Thuế phục vụ khắc phục sự cố.
