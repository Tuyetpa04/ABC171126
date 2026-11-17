# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Thuế Điện tử (MTAX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MTAX` — Quản lý Thuế Điện tử & Dịch vụ công Tổng cục Thuế  
> **Căn cứ mã nguồn thực tế:** `documents/features/mtax/`, `documents/c4/04-code/pipeline-mtax.md`, `mtax-api/`, `mtax-fe/`.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| Mã Rule | Tên Quy tắc | Mô tả Tóm tắt | Loại Rule | Mức ưu tiên | Trace |
|:---:|---|---|:---:|:---:|:---:|
| **MTAX-BR-001** | Phân loại Cổng Nguồn Dữ liệu Thuế | Hệ thống chia tách 2 nguồn cổng dịch vụ thuế: `DVC` (Cổng Dịch vụ công Quốc gia/TCT) và `ETAX` (Cổng Thuế điện tử `thuedientu.gdt.gov.vn`). | Validation | Must | MTAX-UC-001<br>MTAX-FR-001 |
| **MTAX-BR-002** | Bốn Nhóm Nghiệp vụ Dữ liệu Thuế | Dữ liệu thuế thu thập bắt buộc được phân vào 4 danh mục: Tờ khai thuế (`Declaration`), Giấy nộp tiền (`Payment`), Thông báo thuế (`Notice`), và Danh mục tờ khai đăng ký (`Category`). | Structural | Must | MTAX-UC-002..005<br>MTAX-FR-002..005 |
| **MTAX-BR-003** | Trạng thái Xử lý Tờ khai của CQT | Ánh xạ chuẩn hóa trạng thái tờ khai theo các mốc phê duyệt của Cơ quan Thuế: `1` - Chờ tiếp nhận, `2` - Đã tiếp nhận, `3` - Đã chấp nhận, `4` - Không chấp nhận. | State | Must | MTAX-UC-002<br>MTAX-FR-002 |
| **MTAX-BR-004** | Tính Toàn vẹn Giấy nộp tiền & Mã tiểu mục | Mỗi bản ghi Giấy nộp tiền NSNN bắt buộc lưu trữ: Số giấy nộp tiền, Ngày nộp, Kho bạc Nhà nước, Mã chương, Mã tiểu mục (Mục lục NSNN), Số tiền nộp, và File XML chứng từ nộp thuế thành công. | Integrity | Must | MTAX-UC-003<br>MTAX-FR-003 |
| **MTAX-BR-005** | Liên kết Hai chiều Tờ khai & Thông báo Thuế | Khi có Thông báo tiếp nhận/chấp nhận từ CQT trả về, hệ thống tự động gắn mã tham chiếu nối Thông báo (`Notice`) với Tờ khai thuế (`Declaration`) tương ứng theo mã hồ sơ khai thuế. | Constraint | Should | MTAX-UC-004<br>MTAX-FR-004 |
| **MTAX-BR-006** | Độc lập Phiên Xác thực Thuế điện tử | Tài khoản Thuế điện tử (MST + Mật khẩu eTax/DVC) là tài khoản định danh riêng biệt với tài khoản HĐĐT; hệ thống duy trì phiên làm việc độc lập và không dùng chung token của luồng hóa đơn. | Security | Must | MTAX-UC-001<br>MTAX-FR-001 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ

### 2.1. MTAX-BR-001 & MTAX-BR-002: Bốn Nhóm Dữ liệu Thuế
Hệ thống mTax quản lý dữ liệu tập trung theo 4 thực thể chính:
1. **Tờ khai thuế (`Declaration`):** Gồm toàn bộ tờ khai thuế môn bài, tờ khai thuế GTGT, tờ khai tạm tính TNDN, quyết toán TNDN/TNCN và báo cáo tình hình sử dụng hóa đơn (BC26).
2. **Giấy nộp tiền NSNN (`Payment`):** Chứng từ điện tử chứng minh doanh nghiệp đã thực hiện nghĩa vụ nộp tiền vào Kho bạc Nhà nước qua các ngân hàng thương mại liên kết.
3. **Thông báo của Cơ quan Thuế (`Notice`):** Các văn bản phản hồi tự động của CQT (Thông báo tiếp nhận hồ sơ khai thuế điện tử, Thông báo chấp nhận/không chấp nhận tờ khai, Thông báo nợ thuế).
4. **Danh mục Tờ khai Đăng ký (`Category`):** Danh sách các loại tờ khai mà người nộp thuế đã đăng ký nộp qua mạng với Cơ quan Thuế quản lý trực tiếp.

### 2.2. MTAX-BR-003: Quy chuẩn Trạng thái Tiếp nhận
Mỗi tờ khai đồng bộ về được gán nhãn trạng thái trực quan:
* 🟡 **Chờ tiếp nhận:** Hồ sơ vừa được gửi lên cổng, Cổng Thuế chưa sinh mã giao dịch.
* 🔵 **Đã tiếp nhận (Thông báo bước 1):** CQT xác nhận đã nhận được hồ sơ điện tử trong vòng 15 phút kể từ khi gửi.
* 🟢 **Đã chấp nhận (Thông báo bước 2):** CQT kiểm tra tính hợp lệ về mặt chữ ký số và dữ liệu biểu mẫu, chấp nhận tờ khai chính thức có giá trị pháp lý.
* 🔴 **Không chấp nhận:** CQT từ chối hồ sơ kèm văn bản thông báo nêu rõ lý do lỗi (sai cấu trúc, chữ ký số hết hạn...).

### 2.3. MTAX-BR-004: Tiêu chuẩn Dữ liệu Giấy Nộp Tiền
Bản ghi Giấy nộp tiền bắt buộc bóc tách các trường tài chính:
* `paymentNo`: Số chứng từ / Số tham chiếu nộp thuế.
* `treasury`: Kho bạc Nhà nước nơi thụ hưởng ngân sách (tỉnh/thành phố, quận/huyện).
* `taxPeriod`: Kỳ nộp thuế tương ứng.
* `budgetSubItem`: Mã tiểu mục nộp tiền (ví dụ `1701` - Thuế GTGT hàng sản xuất kinh doanh trong nước, `1052` - Thuế TNDN).
* `amount`: Số tiền nộp (VNĐ).

### 2.4. MTAX-BR-005: Liên kết Hai chiều Tờ khai & Thông báo Thuế (Bidirectional Link)
* **Statement**: Khi hệ thống thu thập được các Thông báo tiếp nhận/chấp nhận từ Cơ quan Thuế, hệ thống phải tự động phân tích mã giao dịch hồ sơ để liên kết Thông báo (`Notice`) với Tờ khai thuế (`Declaration`) tương ứng, tạo thành luồng hồ sơ khép kín.
* **Action / Algorithm**:
  1. Bóc tách mã hồ sơ khai thuế (`ho_so_id` hoặc `ma_giao_dich`) từ nội dung Thông báo của CQT.
  2. Tìm kiếm trong bảng Tờ khai thuế bản ghi có cùng mã giao dịch hoặc cùng tổ hợp:
     $$\text{Key} = \{ \text{MST}, \text{Mã loại tờ khai}, \text{Kỳ tính thuế}, \text{Lần nộp} \}$$
  3. Nếu tìm thấy:
     - Gắn trường tham chiếu `notice_ids` vào tờ khai thuế tương ứng.
     - Cập nhật trạng thái phê duyệt của tờ khai theo kết luận của thông báo bước 2 (Đã chấp nhận hoặc Không chấp nhận).
     - Cho phép người dùng bấm xem văn bản thông báo của CQT ngay tại màn hình danh sách tờ khai.
* **Schema / Fields**: `mtax_declarations.{notice_ids, approval_status}`, `mtax_notices.{declaration_id, transaction_code}`.

### 2.5. MTAX-BR-006: Độc lập Phiên Xác thực Thuế điện tử (Session Security Isolation)
* **Statement**: Tài khoản giao dịch Thuế điện tử (gồm MST và Mật khẩu đăng nhập Cổng eTax/Dịch vụ công Quốc gia) là thông tin định danh độc lập hoàn toàn với tài khoản HĐĐT; hệ thống duy trì phiên làm việc riêng biệt và không chia sẻ token/phiên xác thực với luồng hóa đơn.
* **Action / Algorithm**:
  1. Thông tin đăng nhập Cổng eTax được lưu trữ riêng trong bảng `mtax_accounts` và được mã hóa đối xứng AES-256-CBC.
  2. Module `mtax-api` duy trì session cookie và token phiên làm việc độc lập với dịch vụ crawl HĐĐT.
  3. Mọi thao tác đồng bộ mTax chạy qua hàng đợi RabbitMQ riêng biệt (`mtax_crawl`), đảm bảo sự cố nghẽn mạng hoặc hết hạn token bên phân hệ Thuế điện tử không bao giờ làm ảnh hưởng tới luồng cào hóa đơn cốt lõi.
* **Schema / Fields**: `mtax_accounts.{mst, password_encrypted, iv, session_token, last_login}`.


