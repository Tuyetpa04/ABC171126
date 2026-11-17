# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Chứng từ Khấu trừ Thuế TNCN (TNCN)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TNCN` — Quản lý Chứng từ Khấu trừ Thuế Thu nhập Cá nhân Điện tử  
> **Căn cứ pháp lý:** Nghị định số 123/2020/NĐ-CP & Thông tư số 78/2021/TT-BTC (Mẫu số 03/TNCN).  
> **Căn cứ mã nguồn thực tế:** `backend/service/chung-tu-tncn/`, `backend/models/chung_tu_tncn.js`, `documents/c4/04-code/pipeline-chung-tu-tncn.md`.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| Mã Rule | Tên Quy tắc | Mô tả Tóm tắt | Loại Rule | Mức ưu tiên | Trace |
|:---:|---|---|:---:|:---:|:---:|
| **TNCN-BR-001** | Quy chuẩn Mẫu biểu 03/TNCN | Toàn bộ chứng từ khấu trừ thuế TNCN điện tử phải tuân thủ chuẩn Mẫu số 03/TNCN (ký hiệu mẫu, ký hiệu chứng từ, số chứng từ dạng số nguyên tăng liên tục). | Validation | Must | TNCN-UC-001<br>TNCN-FR-001 |
| **TNCN-BR-002** | Bóc tách Thông tin Đối tượng Khấu trừ | Bắt buộc thu thập và phân loại chính xác: MST cá nhân hoặc số CCCD/Hộ chiếu, Họ tên, Quốc tịch, và Trạng thái Cư trú (Cá nhân cư trú / Không cư trú). | Data Integrity | Must | TNCN-UC-002<br>TNCN-FR-002 |
| **TNCN-BR-003** | Quy tắc Bóc tách Chỉ tiêu Thu nhập & Thuế | Mỗi chứng từ bắt buộc chứa: Khoản thu nhập chịu thuế, Thời điểm trả thu nhập (Tháng/Năm), Tổng thu nhập chịu thuế, Tổng thu nhập tính thuế, Số thuế TNCN đã khấu trừ, và Số tiền còn lại thực nhận. | Computation | Must | TNCN-UC-003<br>TNCN-FR-003 |
| **TNCN-BR-004** | Cơ chế Khóa Duy nhất & Khử trùng | Khóa định danh duy nhất của chứng từ: `${mst_to_chuc}_${khmshdon}_${khhdon}_${shdon}`. Ngăn chặn triệt để việc nhân bản trùng lặp bản ghi khi quét lại. | Constraint | Must | TNCN-UC-001<br>TNCN-FR-001 |
| **TNCN-BR-005** | Toàn vẹn Dữ liệu XML & Bản in Thể hiện | Dữ liệu gốc XML lưu trữ nguyên vẹn (gồm thẻ chữ ký số của tổ chức chi trả thu nhập); hỗ trợ kết xuất bản in thể hiện trực quan có giá trị đối soát cho cá nhân đi quyết toán thuế. | Structural | Must | TNCN-UC-004<br>TNCN-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ

### 2.1. TNCN-BR-001 & TNCN-BR-004: Khóa Định danh & Cấu trúc Mẫu 03/TNCN
Chứng từ khấu trừ thuế TNCN điện tử mang giá trị pháp lý tương đương biên lai thuế:
* **Ký hiệu mẫu số:** Cố định định dạng chuẩn CQT (ví dụ: `03/TNCN`).
* **Ký hiệu chứng từ:** Chuỗi ký tự quy định năm và ký hiệu nội bộ (ví dụ: `C26TAA`).
* **Số chứng từ:** Dãy số tự nhiên từ `1` đến `99999999` do tổ chức chi trả phát hành.
* Hệ thống lưu trữ khóa logic duy nhất:
  $$\text{Key} = \text{MST Tổ chức} + \text{"\_"} + \text{Ký hiệu mẫu} + \text{"\_"} + \text{Ký hiệu} + \text{"\_"} + \text{Số chứng từ}$$

### 2.2. TNCN-BR-002: Thông tin Cá nhân Người Nộp Thuế
Hệ thống cho phép lọc và đối soát danh tính người nộp thuế:
* Trường định danh: Ưu tiên Mã số thuế cá nhân (10 số). Nếu cá nhân chưa có MST thì lưu số Căn cước công dân (CCCD), Chứng minh nhân dân hoặc Hộ chiếu.
* Trạng thái cư trú: Đánh dấu cờ `is_resident` để phục vụ quyết toán thuế (cá nhân cư trú áp dụng biểu thuế lũy tiến hoặc khấu trừ 10%; cá nhân không cư trú khấu trừ 20%).

### 2.3. TNCN-BR-003: Chỉ tiêu Thu nhập & Tiền thuế
Mỗi chứng từ bóc tách cụ thể các dòng số liệu:
1. `khoanThuNhap`: Tiền lương, tiền công, thù lao dịch vụ, đầu tư vốn...
2. `thoiDiemTraThuNhap`: Ghi rõ tháng/năm chi trả thu nhập thực tế.
3. `tongThuNhapChiuThue`: Thu nhập chịu thuế đã trả cho cá nhân.
4. `baoHiemBatBuoc`: Các khoản đóng bảo hiểm bắt buộc được trừ.
5. `soThueDaKhauTru`: Số tiền thuế TNCN mà tổ chức đã khấu trừ giữ lại để nộp ngân sách nhà nước.
6. `soTienConDuocNhan`: Số tiền thực trả về tay cá nhân sau khi trừ thuế.

### 2.4. TNCN-BR-005: Toàn vẹn Dữ liệu XML & Bản in Thể hiện (XML & PDF Output)
* **Statement**: Toàn bộ chứng từ khấu trừ thuế TNCN điện tử bắt buộc phải bảo toàn tệp tin XML gốc có chữ ký số của tổ chức chi trả thu nhập; hệ thống hỗ trợ sinh bản in thể hiện (HTML/PDF) chuẩn theo Mẫu 03/TNCN quy định tại Nghị định 123/2020/NĐ-CP để cung cấp cho người lao động thực hiện quyết toán thuế.
* **Action / Algorithm**:
  1. **Lưu trữ XML gốc**: Lưu nội dung XML ký số vào bảng `chung_tu_tncn` hoặc lưu trên Object Storage MinIO. Khóa băm SHA-256 được tính toán để đảm bảo không bị chỉnh sửa dữ liệu.
  2. **Render Bản in Thể hiện**:
     - Hệ thống ánh xạ các trường dữ liệu vào phôi in chuẩn Mẫu số 03/TNCN.
     - Hiển thị đầy đủ thông tin tổ chức trả thu nhập, thông tin cá nhân nộp thuế, các chỉ tiêu số tiền chịu thuế, số tiền thuế đã khấu trừ, số tiền thực nhận.
     - Hiển thị dấu kiểm tra chữ ký số hợp lệ của tổ chức chi trả.
  3. **Xuất Bản in & Tải xuống**: Hỗ trợ xuất tệp tin PDF riêng lẻ hoặc kết xuất danh sách chứng từ khấu trừ thuế TNCN ra file Excel theo kỳ quyết toán để doanh nghiệp nộp báo cáo tình hình sử dụng chứng từ khấu trừ thuế TNCN.
* **Schema / Fields**: `chung_tu_tncn.{xml_data, pdf_path, status, digital_signature}`.


