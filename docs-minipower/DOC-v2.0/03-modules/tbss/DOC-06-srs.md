# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Quản lý Thông báo Sai sót (TBSS)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TBSS` — Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6.16), `SMI_Sơ đồ nghiệp vụ.json` (Trang 5), `mSMI - Tài liệu thiết kế CSDL.docx` (Bảng `hdtbssrses`).

---

## 1. Yêu cầu Chức năng (Functional Requirements)

| Mã FR | Tên Chức năng | Mô tả chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **TBSS-FR-001** | Tra cứu Danh sách Thông báo 04/SS | Cung cấp giao diện lọc và phân trang thông báo 04/SS theo 2 tab Mua vào / Bán ra, hỗ trợ lọc theo ngày, số TB, trạng thái CQT. | Must | TBSS-UC-001<br>TBSS-BR-001 |
| **TBSS-FR-002** | Hiển thị Bản thể hiện 04/SS | Render trực quan mẫu thông báo 04/SS-HĐĐT chuẩn quy định, hiển thị đầy đủ danh sách các hóa đơn sai sót và lý do. | Must | TBSS-UC-002<br>TBSS-BR-002, 004 |
| **TBSS-FR-003** | Tải Dữ liệu XML & File Thể hiện | Endpoint cho phép tải file XML gốc của thông báo 04/SS và tải bản thể hiện dưới định dạng PDF hoặc HTML. | Should | TBSS-UC-003<br>TBSS-BR-004 |
| **TBSS-FR-004** | Tự động Đồng bộ 04/SS từ CQT | Tích hợp API Cổng Thuế để định kỳ cào thông báo 04/SS phát sinh về CSDL `hdtbssrses`. | Must | TBSS-UC-004<br>TBSS-BR-005 |
| **TBSS-FR-005** | Liên kết Hai chiều Hóa đơn - 04/SS | Tự động quét và gắn cờ cảnh báo sai sót lên hóa đơn trong `invoiceraws`, hỗ trợ click chuyển qua lại giữa 2 màn hình. | Must | TBSS-UC-005<br>TBSS-BR-003 |

---

## 2. Đặc tả Chi tiết Từng Yêu cầu

### 2.1. TBSS-FR-001: Tra cứu Danh sách Thông báo 04/SS
- **Tham số đầu vào lọc:**
  - `type`: `PURCHASE` (Mua vào) hoặc `SOLD` (Bán ra).
  - `fromDate`, `toDate`: Khoảng ngày lập/gửi thông báo.
  - `noticeNumber`: Số thông báo của NNT hoặc số CQT tiếp nhận.
  - `status`: Trạng thái xử lý (`1`: Đã tiếp nhận, `2`: CQT chấp nhận, `3`: CQT không chấp nhận).
- **Kết quả trả về:** Danh sách bản ghi phân trang, gồm: Mã ID, Ngày thông báo, Số thông báo, Tên người bán/mua, Số lượng hóa đơn có sai sót trong thông báo, Trạng thái.

### 2.2. TBSS-FR-002: Hiển thị Bản thể hiện 04/SS
- **Cấu trúc hiển thị HTML:**
  - Biểu mẫu tuân theo Mẫu số 04/SS-HĐĐT ban hành kèm theo Nghị định số 123/2020/NĐ-CP.
  - Bảng chi tiết hóa đơn:
    - Hiển thị đầy đủ các cột: STT, Mã CQT cấp, Ký hiệu mẫu số & ký hiệu hóa đơn, Số hóa đơn, Ngày lập, Tính chất sai sót (hiển thị chuỗi tiếng Việt: Hủy / Điều chỉnh / Thay thế / Giải trình tương ứng với mã số 1, 2, 3, 4), Lý do sai sót.
  - Hiển thị chữ ký số của người gửi thông báo.

### 2.3. TBSS-FR-003: Tải Dữ liệu XML & File Thể hiện
- Cho phép người dùng bấm nút:
  - `Tải XML`: Trả về file `.xml` nguyên bản thông điệp CQT.
  - `In / Xuất PDF`: Chuyển đổi bản thể hiện HTML thành file `.pdf` có thể in ấn hoặc lưu trữ.

### 2.4. TBSS-FR-004: Tự động Đồng bộ 04/SS từ CQT
- Sử dụng token xác thực từ phân hệ `TAXACC`.
- Gửi truy vấn lấy danh sách thông báo 04/SS theo mốc thời gian.
- Phân tích cú pháp XML thông báo và ghi dữ liệu vào bảng `hdtbssrses`.
- Ghi nhận lịch sử phiên cào vào `externalaccountsyncs`.

### 2.5. TBSS-FR-005: Liên kết Hai chiều Hóa đơn - 04/SS
- Khi có một thông báo 04/SS mới được lưu:
  - Quét từng dòng hóa đơn sai sót trong danh sách.
  - Tìm bản ghi hóa đơn gốc trong `invoiceraws` khớp MST, Mẫu số, Ký hiệu, Số hóa đơn.
  - Gắn thuộc tính `hasTbss = true` và `tbssInfo = { id, tchat, lydo, ngaytb }` vào hóa đơn gốc.
  - Trên màn hình danh sách hóa đơn, hiển thị biểu tượng thông báo sai sót để kế toán nhận diện nhanh chóng.

