# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Kết nối Cơ quan Thuế (TAXACC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** ISO/IEC/IEEE 29148:2018 / IEEE 830  
> **Phân hệ:** `TAXACC` — Kết nối Cơ quan Thuế & Đồng bộ HĐĐT  
> **Căn cứ tài liệu gốc:** `[mSMI] API Đăng ký, cập nhật tài khoản Thuế.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 10), `mSMI - Tài liệu thiết kế CSDL.docx` (`externalaccounts`, `externalaccountsyncs`).  
> **Căn cứ mã nguồn thực tế:** `backend/service_crawl_invoices/`, `backend/routes/accounts/`, `backend/models/externalaccount.js`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| FR ID | Tên Yêu cầu | Mô tả Chi tiết | Mức ưu tiên | Source (UC / BR) |
|:---:|---|---|:---:|:---:|
| **TAXACC-FR-001** | Xác thực & Đăng nhập Cổng Thuế | Hệ thống **shall** xác thực thông tin tài khoản với Cổng HĐĐT Tổng cục Thuế (`hoadondientu.gdt.gov.vn`), tự động xử lý Captcha và phản hồi kết quả theo bảng mã lỗi chuẩn. | Must | TAXACC-UC-001, 002<br>TAXACC-BR-001, 002 |
| **TAXACC-FR-002** | Cấu hình Lịch trình Tự động đồng bộ | Hệ thống **shall** cho phép thiết lập bật/tắt và chọn mốc giờ chạy tự động độc lập cho 2 luồng: Hóa đơn Mua vào và Hóa đơn Bán ra. | Must | TAXACC-UC-003<br>TAXACC-BR-004 |
| **TAXACC-FR-003** | Kích hoạt Đồng bộ Thủ công | Hệ thống **shall** cho phép người dùng chọn khoảng thời gian tùy ý (tự động chia nhỏ theo chu kỳ $\le 31$ ngày) và kích hoạt lệnh cào hóa đơn ngay lập tức. | Must | TAXACC-UC-004<br>TAXACC-BR-003 |
| **TAXACC-FR-004** | Quản lý Lịch sử Đồng bộ | Hệ thống **shall** lưu trữ và hiển thị chi tiết nhật ký mọi phiên cào dữ liệu (thời gian, kết quả, số hóa đơn lấy về, số hóa đơn tạo mới, lỗi phát sinh). | Should | TAXACC-UC-005<br>TAXACC-BR-005 |
| **TAXACC-FR-005** | API Đăng ký/Sửa tài khoản cho đối tác | Hệ thống **shall** cung cấp REST API `POST /create-tax-account` và `POST /update-tax-account` kèm header `apiToken` cho phép các ứng dụng ngoài tạo tài khoản CQT. | Must | TAXACC-UC-001, 002<br>TAXACC-BR-002 |

---

## 2. Đặc tả Chi tiết Yêu cầu Chức năng

### 2.1. TAXACC-FR-001: Xác thực & Đăng nhập Cổng Thuế
* **Mục tiêu:** Quản lý thông tin đăng nhập Cổng HĐĐT của Tổng cục Thuế, tự động vượt qua Captcha và chuẩn hóa kết quả xác thực.
* **Đầu vào:** Mã số thuế (`username`), Mật khẩu (`password`), Tùy chọn Proxy IP.
* **Xử lý:**
  1. Kiểm tra hợp lệ định dạng MST (10 hoặc 13 số).
  2. Mã hóa đối xứng mật khẩu bằng thuật toán AES-256-CBC với vector `iv` ngẫu nhiên trước khi lưu vào bảng `externalaccounts` (`TAXACC-BR-002`).
  3. Khi kiểm tra kết nối: Gọi engine bóc tách tải Captcha SVG từ Cổng Thuế, đưa qua module nhận diện ký tự OCR/AI.
  4. Gửi payload đăng nhập kèm Captcha; nhận phản hồi từ CQT và ánh xạ theo bảng mã lỗi chuẩn từ `00` đến `07` (`TAXACC-BR-001`).
* **Đầu ra:** Phản hồi JSON `{ errorCode: "00", message: "Thành công" }` hoặc mã lỗi tương ứng để hiển thị thông báo hướng dẫn cho người dùng.

---

### 2.2. TAXACC-FR-002: Cấu hình Lịch trình Tự động Đồng bộ (Cron Scheduler)
* **Mục tiêu:** Cho phép doanh nghiệp thiết lập tự động hóa quá trình cào hóa đơn định kỳ nền theo lịch cố định mà không cần thao tác thủ công.
* **Đầu vào:** Cờ bật/tắt tự động (`is_auto_sync`), khung giờ chạy (ví dụ: `02:00` hàng ngày), tần suất quét (mỗi 1 giờ, mỗi ngày, mỗi tuần).
* **Xử lý:**
  1. Lưu cấu hình lịch trình riêng biệt cho 2 luồng: Hóa đơn Mua vào (`auto_sync_purchase`) và Hóa đơn Bán ra (`auto_sync_sold`).
  2. Hệ thống PM2 Worker đọc cấu hình và lên lịch thực thi độc lập (`TAXACC-BR-004`).
  3. Mỗi phiên quét tự động chỉ lấy các hóa đơn phát sinh từ mốc `last_sync_date` đến thời điểm hiện tại.
* **Đầu ra:** Trạng thái cấu hình được lưu và tiến trình ngầm được tự động kích hoạt.

---

### 2.3. TAXACC-FR-003: Kích hoạt Đồng bộ Thủ công & Phân đoạn Thời gian
* **Mục tiêu:** Cho phép kế toán chủ động cào bù dữ liệu cho một khoảng thời gian bất kỳ khi cần rà soát gấp.
* **Đầu vào:** Khoảng ngày `tuNgay` (`ge`) và `denNgay` (`le`), chiều hóa đơn (Mua vào / Bán ra).
* **Xử lý:**
  1. Kiểm tra giới hạn 31 ngày: Nếu khoảng cách thời gian lớn hơn 31 ngày, hệ thống tự động băm nhỏ thành các cửa sổ con liên tiếp $\le 31$ ngày (`TAXACC-BR-003`).
  2. Đẩy job vào hàng đợi ưu tiên cao trong RabbitMQ để worker thực hiện ngay lập tức.
  3. Kênh WebSocket/SignalR đẩy trạng thái tiến độ thời gian thực (0% $\rightarrow$ 100%) về giao diện người dùng.
* **Đầu ra:** Danh sách hóa đơn mới được kéo về kho dữ liệu và hiển thị ngay trên bảng.

---

### 2.4. TAXACC-FR-004: Quản lý Nhật ký & Lịch sử Đồng bộ
* **Mục tiêu:** Cung cấp báo cáo audit log minh bạch về toàn bộ các phiên đồng bộ dữ liệu.
* **Xử lý:**
  1. Ghi nhận chi tiết từng phiên cào vào bảng `externalaccountsyncs` (`TAXACC-BR-005`): Thời gian bắt đầu, thời gian kết thúc, thời lượng chạy (`duration_ms`), số lượng hóa đơn cào được, số lượng hóa đơn tạo mới, số lượng hóa đơn cập nhật, trạng thái (`success` hoặc `error`).
  2. Nếu phát sinh lỗi kết nối hoặc CQT gián đoạn: Ghi rõ mã lỗi và nội dung phản hồi từ CQT.
* **Đầu ra:** Giao diện xem lịch sử đồng bộ dạng bảng có phân trang và bộ lọc theo trạng thái/thời gian.

---

### 2.5. TAXACC-FR-005: API Đăng ký & Cập nhật Tài khoản cho Đối tác
* **Mục tiêu:** Cho phép các hệ thống bên ngoài (phần mềm kế toán, ERP, cổng thanh toán) tự động tích hợp đăng ký tài khoản thuế vào mSMI qua REST API an toàn.
* **Đầu vào:**
  - Header: `apiToken` (Token xác thực ứng dụng đối tác).
  - Body: `{ username, password, company_name, tax_code }`.
* **Xử lý:**
  1. Xác thực `apiToken` với danh mục ứng dụng được cấp phép.
  2. Kiểm tra tính hợp lệ và sự tồn tại của tài khoản thuế.
  3. Tự động kiểm tra đăng nhập với CQT; nếu thành công tiến hành tạo mới hoặc cập nhật bản ghi trong `externalaccounts`.
* **Đầu ra:** Trả về kết quả HTTP `200 OK` kèm mã phản hồi chuẩn hóa.
