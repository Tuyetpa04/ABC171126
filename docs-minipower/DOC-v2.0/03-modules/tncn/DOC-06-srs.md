# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Chứng từ Thuế TNCN (TNCN)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TNCN` — Quản lý Chứng từ Khấu trừ Thuế Thu nhập Cá nhân Điện tử  
> **Căn cứ mã nguồn thực tế:** `backend/service/chung-tu-tncn/`, `backend/models/chung_tu_tncn.js`, `documents/c4/04-code/pipeline-chung-tu-tncn.md`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| Mã FR | Tên Chức năng | Mô tả Chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **TNCN-FR-001** | Đồng bộ Chứng từ TNCN từ CQT | Tự động quét theo lịch (chu kỳ 5s chia chunk 7 ngày) hoặc cào thủ công theo yêu cầu từ API `GET /api/pitw` của Cổng Thuế. | Must | TNCN-UC-001<br>TNCN-BR-001, 004 |
| **TNCN-FR-002** | Tra cứu & Bộ lọc Đa tiêu chí | Lọc chứng từ theo: Mã số thuế cá nhân, CCCD/Hộ chiếu, Họ tên người nộp thuế, Khoảng ngày lập, Ký hiệu chứng từ. | Must | TNCN-UC-002<br>TNCN-BR-002 |
| **TNCN-FR-003** | Bóc tách Chỉ tiêu Thu nhập & Thuế | Bóc tách và lưu trữ chi tiết: Tổng thu nhập chịu thuế, Tổng thu nhập tính thuế, Số thuế TNCN đã khấu trừ, Số tiền thực nhận. | Must | TNCN-UC-003<br>TNCN-BR-003 |
| **TNCN-FR-004** | Xuất Bản thể hiện PDF & Tải XML Gốc | Render trực quan mẫu chứng từ 03/TNCN theo quy định của Bộ Tài chính; cho phép tải tệp XML gốc lưu trữ MinIO S3. | Must | TNCN-UC-004<br>TNCN-BR-005 |
| **TNCN-FR-005** | Xác thực Toàn vẹn & Chữ ký số | Kiểm tra chữ ký số của đơn vị chi trả thu nhập trên XML chứng từ, ghi nhận cờ toàn vẹn dữ liệu. | Should | TNCN-UC-003<br>TNCN-BR-005 |

---

## 2. Đặc tả Chi tiết Yêu cầu Chức năng

### 2.1. TNCN-FR-001: Đồng bộ Chứng từ Khấu trừ Thuế TNCN từ CQT
* **Mục tiêu:** Tự động kết nối và tải toàn bộ chứng từ khấu trừ thuế TNCN điện tử do tổ chức phát hành từ Cổng Tổng cục Thuế.
* **Xử lý:**
  1. Sử dụng tài khoản thuế đã xác thực (`externalaccounts`), gọi endpoint `/api/pitw` của CQT.
  2. Áp dụng chiến lược chia nhỏ khoảng thời gian theo chu kỳ 7 ngày/lần cào nhằm tối ưu tốc độ và không làm nghẽn API.
  3. Xây dựng khóa duy nhất `${mst_to_chuc}_${khmshdon}_${khhdon}_${shdon}` (`TNCN-BR-004`) để chống trùng lặp dữ liệu khi quét lại.
* **Đầu ra:** Các bản ghi chứng từ mới được lưu vào bảng `chungtutncns`.

---

### 2.2. TNCN-FR-002: Tra cứu & Bộ lọc Đa tiêu chí
* **Mục tiêu:** Cung cấp công cụ tra cứu chứng từ khấu trừ thuế TNCN theo người lao động nhanh chóng.
* **Đầu vào:** Mã số thuế cá nhân (10 số), Số CCCD/Hộ chiếu, Họ tên người nộp thuế, Năm tính thuế, Ký hiệu chứng từ.
* **Xử lý:**
  1. Hỗ trợ tìm kiếm mờ (Fuzzy Search) theo họ tên người nộp thuế có hỗ trợ tiếng Việt có dấu/không dấu.
  2. Hỗ trợ lọc theo trạng thái cư trú (`isResident = true/false`) để phân loại biểu thuế khấu trừ (`TNCN-BR-002`).
* **Đầu ra:** Danh sách chứng từ thỏa mãn tiêu chí tìm kiếm với tốc độ phản hồi $\le 1\text{s}$.

---

### 2.3. TNCN-FR-003: Bóc tách Chỉ tiêu Thu nhập & Thuế Khấu trừ
* **Mục tiêu:** Bóc tách chính xác toàn bộ các số liệu tài chính từ cấu trúc tệp XML theo chuẩn Mẫu số 03/TNCN quy định tại Nghị định 123/2020/NĐ-CP.
* **Xử lý:**
  1. Phân tích cây thẻ XML để trích xuất:
     - `khoanThuNhap`: Khoản thu nhập chịu thuế (tiền lương, thù lao, đầu tư vốn...).
     - `thoiDiemTra`: Tháng/năm chi trả thu nhập thực tế.
     - `tongThuNhapChiuThue`: Tổng thu nhập đã trả.
     - `tongThuNhapTinhThue`: Thu nhập sau giảm trừ.
     - `soThueDaKhauTru`: Tiền thuế TNCN đã khấu trừ giữ lại.
     - `soTienConDuocNhan`: Số tiền thực chi trả về tay cá nhân.
  2. Kiểm tra tính cân đối toán học: $\text{soTienConDuocNhan} = \text{tongThuNhapChiuThue} - \text{soThueDaKhauTru} - \text{baoHiemBatBuoc}$.
* **Đầu ra:** Dữ liệu số liệu tài chính được lưu trữ đầy đủ trong CSDL.

---

### 2.4. TNCN-FR-004: Xuất Bản thể hiện PDF & Tải XML Gốc
* **Mục tiêu:** Tạo bản in thể hiện trực quan có giá trị pháp lý để cấp phát cho người lao động đi quyết toán thuế TNCN.
* **Xử lý:**
  1. Render trực tiếp mẫu chứng từ Mẫu số 03/TNCN dạng HTML/PDF theo phôi quy chuẩn của Bộ Tài chính.
  2. Hỗ trợ tải tệp tin XML gốc lưu trữ trên MinIO S3 để kiểm tra chữ ký số.
  3. Cung cấp tính năng kết xuất danh sách chứng từ ra bảng tính Excel phục vụ làm báo cáo tình hình sử dụng chứng từ khấu trừ thuế TNCN.
* **Đầu ra:** Tệp PDF bản in thể hiện hoặc file XML tải về thành công.

---

### 2.5. TNCN-FR-005: Xác thực Toàn vẹn & Chữ ký số Chứng từ
* **Mục tiêu:** Đảm bảo chứng từ điện tử không bị giả mạo và chữ ký số của tổ chức chi trả thu nhập là hợp lệ.
* **Xử lý:**
  1. Xác thực cấu trúc tệp XML theo đúng lược đồ XSD quy định cho chứng từ TNCN điện tử.
  2. Thẩm định tính toàn vẹn của thẻ chữ ký số `<Signature>` và kiểm tra thời hạn chứng thư số của đơn vị chi trả tại thời điểm ký (`TNCN-BR-005`).
* **Đầu ra:** Ghi nhận cờ `is_valid_signature = true` trên chứng từ.

---

### 2.6. Đặc tả Cấu trúc Bảng Dữ liệu Chứng từ (`chungtutncns`)

| Tên trường | Kiểu dữ liệu | Bắt buộc | Mô tả & Ý nghĩa nghiệp vụ |
|---|:---:|:---:|---|
| `_id` | ObjectId | Có | Định danh bản ghi duy nhất. |
| `user` | ObjectId | Có | FK trỏ `useraccounts._id` (Doanh nghiệp chi trả). |
| `account` | ObjectId | Có | FK trỏ `externalaccounts._id` (Tài khoản Thuế). |
| `key` | String | Có | Khóa duy nhất: `${mst_to_chuc}_${khmshdon}_${khhdon}_${shdon}` (Unique Index). |
| `soChungTu` | Number | Có | Số chứng từ khấu trừ thuế TNCN (dạng số nguyên). |
| `kyHieu` | String | Có | Ký hiệu chứng từ (ví dụ: `C26TAA`). |
| `mauSo` | String | Có | Cố định `03/TNCN`. |
| `ngayLap` | Date | Có | Thời điểm lập chứng từ khấu trừ thuế. |
| `hoTenNNT` | String | Có | Họ và tên người nhận thu nhập được khấu trừ thuế. |
| `mstNNT` | String | Không | Mã số thuế cá nhân của người nhận thu nhập (10 số). |
| `cccdNNT` | String | Không | Số Căn cước công dân / Hộ chiếu (nếu chưa có MST cá nhân). |
| `quocTich` | String | Có | Quốc tịch (mặc định: `Việt Nam`). |
| `isResident` | Boolean | Có | `true`: Cá nhân cư trú; `false`: Cá nhân không cư trú. |
| `khoanThuNhap` | String | Có | Tên khoản thu nhập (Tiền lương, thù lao, đầu tư vốn...). |
| `thoiDiemTra` | String | Có | Tháng/năm chi trả thu nhập (ví dụ: `Tháng 09/2026`). |
| `tongThuNhapChiuThue` | Number | Có | Tổng thu nhập chịu thuế đã chi trả (VNĐ). |
| `tongThuNhapTinhThue` | Number | Có | Thu nhập tính thuế sau khi trừ các khoản giảm trừ (VNĐ). |
| `soThueDaKhauTru` | Number | Có | Số tiền thuế TNCN đã giữ lại nộp vào ngân sách. |
| `soTienConDuocNhan` | Number | Có | Số tiền thực trả cho cá nhân sau khi trừ thuế. |
| `xmlKey` | String | Không | Đường dẫn lưu trữ tệp XML gốc trên cụm MinIO S3. |
