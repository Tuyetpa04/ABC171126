# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Quản lý Hộp thư & Bóc tách Hóa đơn từ Email (MAILBX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MAILBX` — Quản lý Hộp thư & Tự động bóc tách hóa đơn từ Email  
> **Căn cứ tài liệu gốc:** `TÀI LIỆU HƯỚNG DẪN ĐỒNG BỘ HÓA ĐƠN QUA EMAIL.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 7 & Mục 4), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 1), `mSMI - Tài liệu thiết kế CSDL.docx` (`gmailaccounts`, `externalaccounts`, `attachments`, `invoiceraws`).

---

## 1. Danh mục Quy tắc Nghiệp vụ

| ID | Tên Quy tắc | Mô tả Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **MAILBX-BR-001** | Chuẩn hóa Giao thức Xác thực Hộp thư | Hỗ trợ 2 phương thức kết nối: Google OAuth2 / App Password cho Gmail; giao thức IMAP/SSL (Port 993) cho Email Server doanh nghiệp. Mật khẩu phải được mã hóa trước khi lưu trữ. | Authorization | Must | MAILBX-UC-001<br>MAILBX-FR-001 |
| **MAILBX-BR-002** | Bộ lọc Nhận diện Thư chứa Hóa đơn | Hệ thống chỉ kích hoạt bóc tách đối với các email có tệp đính kèm mang định dạng `.xml` hoặc file nén `.zip`. Các thư văn bản thuần túy hoặc thư spam không chứa định dạng trên sẽ bị bỏ qua. | Validation | Must | MAILBX-UC-002<br>MAILBX-FR-002 |
| **MAILBX-BR-003** | Giải nén ZIP & Kiểm tra Cấu trúc XML | Tệp nén `.zip` đính kèm phải được tự động giải nén. Tất cả các file `.xml` tìm thấy bên trong phải tuân thủ chuẩn thông điệp hóa đơn điện tử Nghị định 123 / Thông tư 78 của Tổng cục Thuế. | Computation | Must | MAILBX-UC-004<br>MAILBX-FR-003 |
| **MAILBX-BR-004** | Chống Trùng lặp Hóa đơn Bóc tách | Khi bóc tách thành công một hóa đơn từ email, hệ thống kiểm tra khóa duy nhất `(sellerTaxCode, invoiceTemplate, invoiceSeries, invoiceNumber)`. Nếu hóa đơn đã tồn tại (đã crawl từ CQT hoặc email khác), không tạo trùng bản ghi hóa đơn mà chỉ gắn tệp đính kèm vào hóa đơn hiện có. | Integrity | Must | MAILBX-UC-004<br>MAILBX-FR-003 |
| **MAILBX-BR-005** | Ghi vết Lịch sử Đồng bộ Email | Mọi phiên quét hộp thư phải ghi nhận nhật ký vào CSDL (`gmailaccounts` / `externalaccountsyncs`): Thời gian quét, tổng số email mới nhận, số email chứa XML (`hasMimeXml`), số hóa đơn bóc tách thành công, danh sách lỗi nếu có. | Validation | Must | MAILBX-UC-002<br>MAILBX-FR-005 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ

### 2.1. MAILBX-BR-001: Chuẩn hóa Giao thức Xác thực Hộp thư
- **Gmail**:
  - Người dùng có thể kết nối thông qua nút "Đăng nhập bằng Google" (OAuth2 token scope: `mail.google.com` hoặc `gmail.readonly`).
  - Hoặc sử dụng App Password (Mật khẩu ứng dụng 16 ký tự) sau khi đã kích hoạt xác thực 2 bước (2FA) trên tài khoản Google cá nhân/doanh nghiệp.
- **Mail Server nội bộ (IMAP)**:
  - Khai báo: `imap_host` (ví dụ `imap.zoho.com`, `mail.company.com`), `imap_port` (mặc định `993`), `use_ssl = true`, `username`, `encrypted_password`.
  - Phải có nút **Kiểm tra kết nối (Test Connection)** trước khi lưu để xác thực tính đúng đắn của thông tin đăng nhập.

### 2.2. MAILBX-BR-002: Bộ lọc Nhận diện Thư chứa Hóa đơn
- Hệ thống định kỳ quét thư mục Hộp thư đến (Inbox) hoặc nhãn được chỉ định.
- Chỉ xử lý các thư chưa đọc hoặc thư phát sinh sau mốc thời gian đồng bộ gần nhất.
- Kiểm tra MIME Type và phần mở rộng (extension) của tệp đính kèm:
  - Có đuôi `.xml` (hoặc MIME `application/xml`, `text/xml`).
  - Hoặc có đuôi `.zip` (hoặc MIME `application/zip`, `application/x-zip-compressed`).
- Nếu email chỉ có tệp `.pdf` hoặc `.jpg` mà không có `.xml`/`.zip`, thư vẫn được lưu vào danh sách email nhưng trường `hasMimeXml` gắn giá trị `false` và không chuyển vào kho hóa đơn tự động.

### 2.3. MAILBX-BR-003: Giải nén ZIP & Kiểm tra Cấu trúc XML
- Khi phát hiện tệp nén `.zip`, cơ chế trích xuất chạy ngầm tự động bung tệp vào thư mục tạm hoặc RAM buffer:
  - Quét duyệt toàn bộ cây thư mục trong file ZIP để tìm tất cả các file có đuôi `.xml`.
  - Đọc nội dung root node XML để xác thực thẻ HĐĐT: `<HDon>`, `<Invoice>`, hoặc định dạng theo Quyết định 1450/QĐ-TCT.
  - Nếu file ZIP có mật khẩu (password-protected) hoặc bị lỗi tệp nén (corrupt archive), ghi nhận trạng thái lỗi bóc tách để người dùng xử lý thủ công.

### 2.4. MAILBX-BR-004: Chống Trùng lặp Hóa đơn Bóc tách
- Khóa định danh hóa đơn toàn hệ thống:
  $$\text{InvoiceID} = \text{MST Bán} + \text{"\_"} + \text{Mẫu số} + \text{"\_"} + \text{Ký hiệu} + \text{"\_"} + \text{Số hóa đơn}$$
- Nếu `InvoiceID` đã tồn tại trong bảng `invoiceraws`:
  1. Giữ nguyên dữ liệu hóa đơn gốc đã lưu trước đó.
  2. Bổ sung liên kết tệp đính kèm XML/PDF từ email vào mảng `attachments` của hóa đơn.
  3. Cập nhật cờ nguồn gốc: Hóa đơn được ghi nhận có xuất hiện qua kênh Email.
- Nếu `InvoiceID` chưa tồn tại trong `invoiceraws`:
  1. Tạo mới bản ghi hóa đơn trong `invoiceraws`.
  2. Trích xuất đầy đủ thông tin: Người bán, người mua, ngày lập, chi tiết dòng hàng hóa, thuế suất, tổng tiền.
  3. Tự động kích hoạt luồng kiểm tra rủi ro (`RISK-BR-001` đến `RISK-BR-006`).

### 2.5. MAILBX-BR-005: Ghi vết Lịch sử Đồng bộ Email (Email Sync Audit Log)
- **Statement**: Mọi phiên quét hộp thư định kỳ hoặc kích hoạt thủ công đều bắt buộc phải ghi vết đầy đủ vào bảng `gmailaccounts` hoặc `externalaccountsyncs` để người dùng và quản trị viên nắm rõ tiến độ và tỷ lệ thành công của quá trình bóc tách.
- **Condition**: Khi tác vụ quét email bắt đầu kết nối tới máy chủ thư điện tử (IMAP hoặc Gmail API) cho tới khi hoàn tất phân tích toàn bộ các thư mới.
- **Action / Algorithm**:
  1. Ghi nhận mốc thời gian bắt đầu quét và cập nhật trạng thái `sync_status = 'processing'`.
  2. Thống kê định lượng sau mỗi phiên:
     - `total_emails_scanned`: Tổng số email mới được tải về phân tích trong phiên.
     - `emails_with_xml`: Số lượng email có đính kèm tệp XML hoặc ZIP chứa hóa đơn (`hasMimeXml = true`).
     - `invoices_extracted`: Số lượng hóa đơn bóc tách thành công và chuyển vào bảng `invoiceraws`.
     - `errors_count`: Số lượng email không thể giải nén hoặc tệp XML sai định dạng.
  3. Lưu chi tiết danh sách lỗi (nếu có): Tiêu đề thư, người gửi, thời gian nhận thư, lý do thất bại (ví dụ: "File ZIP có mật khẩu bảo vệ", "File XML sai cấu trúc hóa đơn điện tử").
  4. Cập nhật mốc `last_sync_date = new Date()` để các phiên quét tiếp theo chỉ lấy thư phát sinh sau thời điểm này.
- **Schema / Fields**: `gmailaccounts.{last_sync_date, sync_status, last_sync_stats}`, `externalaccountsyncs`.


