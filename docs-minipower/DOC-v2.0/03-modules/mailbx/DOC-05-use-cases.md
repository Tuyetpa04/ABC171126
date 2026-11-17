# DOC-05 — Ca sử dụng Phân hệ Quản lý Hộp thư & Bóc tách Hóa đơn từ Email (MAILBX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MAILBX` — Quản lý Hộp thư & Tự động bóc tách hóa đơn từ Email  
> **Căn cứ tài liệu gốc:** `TÀI LIỆU HƯỚNG DẪN ĐỒNG BỘ HÓA ĐƠN QUA EMAIL.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 7), `SMI_Sơ đồ nghiệp vụ.json` (Trang 1, 12).

---

## 1. Danh sách Ca sử dụng (Use Cases)

| Mã Use Case | Tên Ca sử dụng | Tác nhân chính | Mức ưu tiên | Mô tả tóm tắt |
|:---:|---|:---:|:---:|---|
| **MAILBX-UC-001** | Kết nối & Cấu hình Hộp thư | Kế toán viên / Quản trị viên | Must | Khai báo tài khoản Gmail hoặc IMAP Server để nhận hóa đơn đầu vào. |
| **MAILBX-UC-002** | Tự động quét & Tiếp nhận Email nền | Hệ thống (Robot nền) | Must | Quét định kỳ hộp thư, lọc các email có tệp đính kèm hóa đơn. |
| **MAILBX-UC-003** | Tra cứu danh sách & Xem chi tiết Email | Kế toán viên | Should | Xem danh sách email đã nhận, lọc theo người gửi, trạng thái có file XML. |
| **MAILBX-UC-004** | Tự động bóc tách HĐĐT & Giải nén ZIP | Hệ thống (Robot nền) | Must | Tự động bung file ZIP, trích xuất dữ liệu XML hóa đơn lưu vào kho mSMI. |
| **MAILBX-UC-005** | Tải tệp đính kèm Email gốc | Kế toán viên | Could | Tải file XML, PDF hoặc file ZIP gốc gửi kèm trong thư về máy tính. |

---

## 2. Chi tiết Ca sử dụng

### 2.1. MAILBX-UC-001: Kết nối & Cấu hình Hộp thư

- **Tác nhân:** Kế toán viên (ACT-002) hoặc Quản trị viên (ACT-001).
- **Tiền điều kiện:** Người dùng đã đăng nhập hệ thống mSMI.
- **Hậu điều kiện:** Tài khoản hộp thư được lưu vào bảng `gmailaccounts` hoặc `externalaccounts`, trạng thái kết nối `Active`.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng truy cập menu **Cấu hình** -> **Hộp thư hóa đơn**.
2. Người dùng chọn loại hòm thư cần kết nối:
   - **Cách 1: Kết nối Gmail qua Google OAuth2**: Bấm nút "Đăng nhập bằng Google", đồng ý cấp quyền truy cập hộp thư.
   - **Cách 2: Kết nối Gmail qua App Password**: Nhập địa chỉ Gmail và mật khẩu ứng dụng 16 ký tự của Google.
   - **Cách 3: Kết nối Máy chủ Thư nội bộ (IMAP)**: Nhập Host IMAP, Port (mặc định 993), bật SSL, tên đăng nhập, mật khẩu hòm thư.
3. Người dùng bấm nút **Kiểm tra kết nối**.
4. Hệ thống thực hiện handshake thử nghiệm với mail server. Nếu thành công, hiển thị thông báo "Kết nối thành công".
5. Người dùng bấm **Lưu cấu hình**. Hệ thống kích hoạt tiến trình lắng nghe/quét định kỳ cho hòm thư này.

#### Luồng thay thế (Alternative Flow):
- **4a. Thông tin xác thực không hợp lệ:** Hệ thống báo lỗi "Không thể kết nối tới máy chủ thư điện tử. Vui lòng kiểm tra lại tài khoản/mật khẩu ứng dụng hoặc cài đặt IMAP".

---

### 2.2. MAILBX-UC-002: Tự động quét & Tiếp nhận Email nền

- **Tác nhân:** Hệ thống (Robot nền / Cron Job).
- **Tiền điều kiện:** Đã có ít nhất 1 tài khoản hòm thư đang ở trạng thái kích hoạt.
- **Hậu điều kiện:** Danh sách email mới được lưu vào CSDL; các email chứa XML được chuyển sang bước bóc tách dữ liệu.

#### Luồng sự kiện chính (Main Flow):
1. Đến chu kỳ quét định kỳ, hệ thống kết nối tới hộp thư thông qua IMAP hoặc Google API.
2. Hệ thống tìm kiếm các email mới phát sinh kể từ mốc thời gian quét cuối cùng.
3. Với mỗi email mới tìm thấy:
   - Đọc thông tin header: Người gửi (`from`), Người nhận (`to`), Tiêu đề thư (`subject`), Thời gian nhận (`receivedDate`).
   - Kiểm tra tệp đính kèm (Attachments).
   - Nếu có tệp `.xml` hoặc `.zip`, đánh dấu cờ `hasMimeXml = true`. Ngược lại, gán `hasMimeXml = false`.
   - Lưu thông tin thư và lưu trữ tệp đính kèm vào hệ thống lưu trữ (Storage).
4. Hệ thống cập nhật thời gian hoàn thành phiên quét và số lượng thư nhận được vào lịch sử đồng bộ.

---

### 2.3. MAILBX-UC-003: Tra cứu danh sách & Xem chi tiết Email

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Người dùng đã đăng nhập hệ thống.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng truy cập menu **Hộp thư hóa đơn** -> **Danh sách Email**.
2. Hệ thống hiển thị bảng danh sách các email đã nhận:
   - Người gửi, Tiêu đề thư, Ngày nhận, Trạng thái đính kèm (`Có XML`, `Không có XML`), Trạng thái bóc tách (`Đã tạo hóa đơn`, `Chờ xử lý`, `Lỗi`).
3. Người dùng có thể tìm kiếm theo: Tên người gửi, email người gửi, khoảng ngày nhận, hoặc lọc theo cờ có tệp XML.
4. Người dùng bấm vào một dòng email để xem chi tiết:
   - Hiển thị toàn văn nội dung email (Text / HTML).
   - Danh sách các tệp đính kèm gửi kèm trong thư kèm dung lượng file.

---

### 2.4. MAILBX-UC-004: Tự động bóc tách HĐĐT & Giải nén ZIP

- **Tác nhân:** Hệ thống (Robot nền).
- **Tiền điều kiện:** Email đã được lưu và có cờ `hasMimeXml = true`.
- **Hậu điều kiện:** Dữ liệu hóa đơn được ghi vào bảng `invoiceraws`, các quy tắc kiểm tra rủi ro được kích hoạt.

#### Luồng sự kiện chính (Main Flow):
1. Hệ thống tiếp nhận các email có `hasMimeXml = true`.
2. Nếu tệp đính kèm là `.zip`: Hệ thống tự động giải nén, duyệt tìm các file `.xml`.
3. Nếu tệp đính kèm là `.xml`: Đưa trực tiếp vào luồng xử lý parser.
4. Hệ thống phân tích cấu trúc XML theo chuẩn thông điệp HĐĐT (QĐ 1450 / TT 78):
   - Đọc MST Người bán, Tên người bán, Địa chỉ người bán.
   - Đọc Mẫu số (`khmshdon`), Ký hiệu (`khhdon`), Số hóa đơn (`shdon`), Ngày lập (`tdlap`).
   - Đọc chi tiết các dòng hàng hóa, số lượng, đơn giá, thành tiền, thuế suất, tiền thuế GTGT.
5. Kiểm tra trùng lặp theo `MAILBX-BR-004`:
   - Nếu chưa tồn tại: Thêm mới bản ghi vào `invoiceraws`, đánh dấu nguồn tiếp nhận là `Email`, lưu tệp XML và PDF đính kèm vào mảng `attachments`.
   - Nếu đã tồn tại: Cập nhật tệp đính kèm vào bản ghi có sẵn, ghi nhận hóa đơn đã có thêm kênh xác nhận từ Email.
6. Kích hoạt quy tắc rủi ro tự động (`RISK-BR-001` đến `RISK-BR-006`).

#### Luồng ngoại lệ (Exception Flow):
- **4a. File XML không đúng chuẩn định dạng HĐĐT:** Ghi nhận cờ lỗi bóc tách cho email, lưu nguyên trạng file đính kèm để người dùng mở kiểm tra thủ công.

---

### 2.5. MAILBX-UC-005: Tải tệp đính kèm Email gốc

- **Tác nhân:** Kế toán viên (ACT-002).
- **Mô tả:** Cho phép người dùng bấm tải trực tiếp tệp `.xml`, `.pdf` hoặc `.zip` nguyên bản được gửi từ nhà cung cấp về máy tính để lưu trữ đối chứng.

