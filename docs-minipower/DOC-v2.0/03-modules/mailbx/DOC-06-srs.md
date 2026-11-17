# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Quản lý Hộp thư & Bóc tách Hóa đơn từ Email (MAILBX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MAILBX` — Quản lý Hộp thư & Tự động bóc tách hóa đơn từ Email  
> **Căn cứ tài liệu gốc:** `TÀI LIỆU HƯỚNG DẪN ĐỒNG BỘ HÓA ĐƠN QUA EMAIL.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 7 & Mục 4), `mSMI - Tài liệu thiết kế CSDL.docx` (`gmailaccounts`, `externalaccounts`, `attachments`).

---

## 1. Yêu cầu Chức năng (Functional Requirements)

| Mã FR | Tên Chức năng | Mô tả chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **MAILBX-FR-001** | Cấu hình & Xác thực Hộp thư | Cho phép thêm, sửa, xóa cấu hình hòm thư (Gmail OAuth2 / App Password, IMAP/SSL Port 993); có nút Test Connection kiểm tra đăng nhập trước khi lưu. | Must | MAILBX-UC-001<br>MAILBX-BR-001 |
| **MAILBX-FR-002** | Quét Nền & Lọc Tệp Đính kèm | Chạy tiến trình ngầm định kỳ kết nối hộp thư, đọc email mới và kiểm tra định dạng tệp đính kèm (`.xml`, `.zip`), gắn cờ `hasMimeXml`. | Must | MAILBX-UC-002<br>MAILBX-BR-002 |
| **MAILBX-FR-003** | Giải nén ZIP & Bóc tách XML | Tự động giải nén file ZIP, đọc dữ liệu các file XML hóa đơn bên trong và chuyển đổi thành cấu trúc hóa đơn mSMI. | Must | MAILBX-UC-004<br>MAILBX-BR-003 |
| **MAILBX-FR-004** | Khử trùng & Đồng bộ Hóa đơn | Kiểm tra khóa định danh hóa đơn, tránh trùng lặp bản ghi trong `invoiceraws`, bổ sung file đính kèm vào hóa đơn có sẵn. | Must | MAILBX-UC-004<br>MAILBX-BR-004 |
| **MAILBX-FR-005** | Quản lý Danh sách Thư & Tải File | Màn hình hiển thị danh sách thư nhận được, tìm kiếm/lọc, xem chi tiết nội dung thư và tải các tệp đính kèm gốc. | Should | MAILBX-UC-003<br>MAILBX-UC-005 |

---

## 2. Đặc tả Chi tiết Từng Yêu cầu

### 2.1. MAILBX-FR-001: Cấu hình & Xác thực Hộp thư
- **Dữ liệu đầu vào:**
  - Loại hộp thư: `Gmail` hoặc `Custom IMAP`.
  - Nếu Gmail: Cho phép chọn luồng OAuth2 Redirect hoặc nhập Email + App Password (16 ký tự).
  - Nếu IMAP: `Server Host` (string), `Port` (number, mặc định `993`), `SSL` (boolean, mặc định `true`), `Username` (string), `Password` (string).
- **Quy trình xử lý:**
  - Khi bấm "Kiểm tra kết nối": Hệ thống gọi hàm `testConnection()`. Nếu server trả lời xác thực thành công, trả về HTTP 200 kèm thông điệp "Kết nối thành công".
  - Mật khẩu trước khi lưu xuống CSDL phải được mã hóa theo chuẩn mã hóa hệ thống.
  - Mỗi đơn vị/MST có thể cấu hình 1 hoặc nhiều tài khoản hộp thư nhận hóa đơn.

### 2.2. MAILBX-FR-002: Quét Nền & Lọc Tệp Đính kèm
- **Cơ chế kích hoạt:** Cron Job định kỳ (ví dụ mỗi 10 hoặc 15 phút một lần tùy cấu hình hệ thống).
- **Xử lý:**
  - Kết nối tới hòm thư, gửi truy vấn tìm kiếm các thư mới có mốc thời gian lớn hơn thời điểm quét lần cuối.
  - Phân tích danh sách `parts` của thư:
    - Nếu phát hiện tệp có định dạng mở rộng `.xml` hoặc `.zip`, đánh dấu cờ `hasMimeXml = true`.
    - Tải nội dung tệp và lưu vào hệ thống tệp đính kèm cục bộ/cloud storage.
  - Lưu bản ghi thư vào CSDL kèm thông tin: `messageId`, `sender`, `recipient`, `subject`, `receivedDate`, `hasMimeXml`.

### 2.3. MAILBX-FR-003: Giải nén ZIP & Bóc tách XML
- **Giải nén:**
  - Nhận luồng byte của tệp `.zip`.
  - Giải nén tự động không đòi hỏi tương tác người dùng.
  - Tìm kiếm toàn bộ các tệp `.xml` bên trong thư mục giải nén.
- **Bóc tách XML:**
  - Parse DOM của từng file XML.
  - Trích xuất các thẻ dữ liệu hóa đơn theo chuẩn QĐ 1450/QĐ-TCT:
    - Mã số thuế người bán, tên người bán, địa chỉ người bán.
    - Mẫu số (`khmshdon`), Ký hiệu (`khhdon`), Số hóa đơn (`shdon`), Ngày lập (`tdlap`).
    - Mã tra cứu hóa đơn, chữ ký số (Certificate Subject, Serial, Signing Time).
    - Danh sách chi tiết hàng hóa: Tên hàng, ĐVT, Số lượng, Đơn giá, Thành tiền, Thuế suất GTGT.
    - Tổng tiền chưa thuế, tổng tiền thuế, tổng tiền thanh toán.

### 2.4. MAILBX-FR-004: Khử trùng & Đồng bộ Hóa đơn
- Tìm kiếm bản ghi trong bảng `invoiceraws` theo bộ khóa:
  $$\text{Query} = \{ \text{nbmst}: \text{MST\_Bán}, \text{khmshdon}: \text{Mẫu\_Số}, \text{khhdon}: \text{Ký\_Hiệu}, \text{shdon}: \text{Số\_HĐ} \}$$
- Nếu tìm thấy:
  - Bổ sung tệp đính kèm XML và PDF vào danh sách `attachments` của hóa đơn.
  - Không ghi đè các trường thông tin kế toán đã chỉnh sửa (như nhãn, ghi chú, trạng thái kê khai).
- Nếu không tìm thấy:
  - Chèn bản ghi mới vào `invoiceraws` với nguồn gốc `source = "EMAIL"`.
  - Gọi service kiểm tra rủi ro để đánh giá trạng thái pháp lý người bán và đối chiếu danh mục 1500 DN.

### 2.5. MAILBX-FR-005: Quản lý Danh sách Thư & Tải File
- Giao diện danh sách: Phân trang (10, 20, 50 dòng/trang).
- Hiển thị badge màu:
  - `Có XML`: Màu xanh lá cây.
  - `Không có XML`: Màu xám.
  - `Đã tạo hóa đơn`: Màu xanh dương.
- Cho phép tải tệp tin đính kèm trực tiếp thông qua endpoint an toàn có kiểm tra quyền truy cập.

