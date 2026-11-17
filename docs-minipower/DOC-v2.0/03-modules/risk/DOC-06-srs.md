# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Kiểm soát Rủi ro Thuế (RISK)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** ISO/IEC/IEEE 29148:2018 / IEEE 830  
> **Phân hệ:** `RISK` — Kiểm soát & Cảnh báo Rủi ro Thuế  
> **Căn cứ tài liệu gốc:** `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 6.13, 6.14), `[mSMI] Document API.docx`.  
> **Căn cứ mã nguồn thực tế:** `backend/service_risk/`, `backend/models/taxcodeclassificationappl.js`, `backend/models/taxcodeinfo.js`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| FR ID | Tên Yêu cầu | Mô tả Chi tiết | Mức ưu tiên | Source (UC / BR) |
|:---:|---|---|:---:|:---:|
| **RISK-FR-001** | Tự động phát hiện trạng thái pháp lý MST | Hệ thống **shall** tự động kiểm tra MST người bán từ cơ sở dữ liệu Thuế và gắn cờ cảnh báo đối với các trạng thái: Ngừng HĐ chưa đóng MST, Tạm ngừng kinh doanh, Không hoạt động tại địa chỉ đăng ký. | Must | RISK-UC-001<br>RISK-BR-001, 004 |
| **RISK-FR-002** | Đối soát Blacklist 1.500 DN rủi ro cao | Hệ thống **shall** tự động đối chiếu MST người bán với danh mục 1.500 doanh nghiệp rủi ro cao và hiển thị cờ đỏ cảnh báo nghiêm trọng. | Must | RISK-UC-001<br>RISK-BR-002 |
| **RISK-FR-003** | Cảnh báo Hóa đơn bị Hủy / Thay đổi | Hệ thống **shall** tự động cảnh báo khi hóa đơn mua vào bị bên bán hủy hoặc bị thay thế/điều chỉnh bởi hóa đơn khác trên Cổng Thuế. | Must | RISK-UC-003<br>RISK-BR-003 |
| **RISK-FR-004** | Kiểm tra Chữ ký số & Tính toàn vẹn XML | Hệ thống **shall** xác thực tính hợp lệ của chữ ký số người bán và tính toàn vẹn của tệp tin XML gốc khi tải về hoặc tải lên. | Must | RISK-UC-001<br>RISK-BR-005, 006 |
| **RISK-FR-005** | Tra cứu trạng thái hoạt động MST/CCCD | Hệ thống **shall** cung cấp giao diện và API tra cứu nhanh thông tin pháp lý của một MST hoặc CCCD bất kỳ trực tiếp từ cơ sở dữ liệu Thuế. | Must | RISK-UC-002<br>RISK-BR-001 |

---

## 2. Đặc tả Chi tiết Yêu cầu Chức năng

### 2.1. RISK-FR-001: Tự động Phát hiện Trạng thái Pháp lý Người bán
* **Mục tiêu:** Giám sát liên tục trạng thái pháp lý của các đối tác cung cấp hóa đơn đầu vào nhằm phát hiện sớm rủi ro hóa đơn bất hợp pháp.
* **Đầu vào:** Mã số thuế người bán (`header.nbmst`) của các hóa đơn mua vào và ngày lập hóa đơn (`header.tdlap`).
* **Xử lý:**
  1. Truy vấn bảng `taxcodeinfos` theo `mst = nbmst`.
  2. Đánh giá trạng thái:
     - Nếu mã trạng thái $\ne \text{'00'}$ (Đang hoạt động): Kích hoạt cảnh báo tương ứng (Tạm ngừng KD, Ngừng HĐ chưa đóng MST, Bỏ trốn khỏi địa chỉ kinh doanh).
     - Kiểm tra mốc thời gian: Nếu ngày lập hóa đơn rơi vào giai đoạn tạm ngừng hoặc sau ngày đóng MST (`RISK-BR-004`), nâng mức cảnh báo lên cấp độ **Nguy hiểm (Màu Đỏ)**.
  3. Cập nhật trường `warning.status`, `warning.is_risk = true` trên bản ghi hóa đơn.
* **Đầu ra:** Huy hiệu cảnh báo hiển thị trên giao diện danh sách hóa đơn mua vào và thống kê trên Dashboard rủi ro.

---

### 2.2. RISK-FR-002: Đối soát Blacklist 1.500 Doanh nghiệp Rủi ro cao
* **Mục tiêu:** Tự động ngăn chặn việc đưa hóa đơn từ các doanh nghiệp gian lận hóa đơn vào kê khai thuế.
* **Đầu vào:** `nbmst` của hóa đơn mua vào.
* **Xử lý:**
  1. Đối soát `nbmst` với bảng `taxcodeclassificationappls` (danh mục 1.500 DN do TCT công bố).
  2. Nếu trùng khớp:
     - Gán nhãn rủi ro mức độ cao nhất: `risk_level = CRITICAL` (Màu Đỏ).
     - Điền lý do: Số hiệu công văn của CQT, ngày ban hành công văn.
     - Khóa gợi ý khấu trừ: Cấm tự động phân bổ vào Tờ khai Mẫu 01/GTGT cho tới khi kế toán trưởng thẩm định hồ sơ thực tế (`RISK-BR-002`).
* **Đầu ra:** Biểu tượng cờ đỏ nổi bật trên bảng kèm tooltip: *"Cảnh báo: Người bán thuộc Danh mục 1.500 DN rủi ro cao của Tổng cục Thuế"*.

---

### 2.3. RISK-FR-003: Cảnh báo Hóa đơn bị Hủy / Thay thế / Điều chỉnh
* **Mục tiêu:** Giúp kế toán bên mua phát hiện ngay lập tức các hóa đơn đã nhận nhưng bị bên bán đơn phương sửa đổi hoặc hủy bỏ trên Cổng Thuế.
* **Đầu vào:** Sự thay đổi giá trị của trường `header.tthai` sau các phiên quét đồng bộ dữ liệu.
* **Xử lý:**
  1. Nếu `header.tthai === 4` (Bị hủy):
     - Phát cảnh báo khẩn cấp: Yêu cầu kế toán rà soát kỳ kê khai để điều chỉnh giảm thuế GTGT đầu vào nếu đã kê khai.
  2. Nếu `header.tthai === 2` (Điều chỉnh) hoặc `3` (Thay thế):
     - Tự động tra cứu hóa đơn mới qua mã tham chiếu hoặc thông báo sai sót (`TBSS`).
     - Tạo liên kết đối soát 2 chiều giữa hóa đơn cũ và hóa đơn mới để kế toán so sánh biến động số tiền/thuế (`RISK-BR-003`).
* **Đầu ra:** Danh sách chuyên biệt "Hóa đơn mua vào có thay đổi trạng thái" hiển thị trên màn hình Giám sát Thuế.

---

### 2.4. RISK-FR-004: Kiểm tra Chữ ký số & Tính Toàn vẹn File XML
* **Mục tiêu:** Thẩm định tính pháp lý điện tử của tệp hóa đơn XML theo chuẩn Luật Giao dịch Điện tử.
* **Xử lý:**
  1. Tính mã băm SHA-256 trên nội dung hóa đơn (`<DLHDon>`), đối chiếu với `<DigestValue>` trong thẻ chữ ký số. Cảnh báo vi phạm nếu băm sai lệch.
  2. Bóc tách chứng thư số của người bán: Kiểm tra thời hạn hiệu lực (`validFrom`, `validTo`) tại thời điểm ký (`signingTime`). Nếu ký khi chứng thư số hết hạn $\implies$ Gán nhãn chữ ký không hợp lệ (`RISK-BR-005`).
  3. So khớp tên người bán trên hóa đơn với tên trên chứng thư số và CSDL Thuế để phát hiện sai lệch mạo danh (`RISK-BR-006`).
* **Đầu ra:** Bảng kết luận thẩm định kỹ thuật số hiển thị chi tiết trong popup xem hóa đơn.

---

### 2.5. RISK-FR-005: Tra cứu Trạng thái Hoạt động MST/CCCD Trực tuyến
* **Mục tiêu:** Cung cấp công cụ tra cứu nhanh thông tin pháp lý cho kế toán viên trước khi ký kết hợp đồng hoặc nhận hóa đơn từ đối tác mới.
* **Đầu vào:** Mã số thuế (10 hoặc 13 số) hoặc số CCCD của người đại diện pháp luật.
* **Xử lý:**
  1. Gọi API `GET /api/tax-code/:mst/info` truy vấn cơ sở dữ liệu `taxcodeinfos`.
  2. Trả về thông tin đăng ký kinh doanh: Tên doanh nghiệp chính thức, địa chỉ trụ sở, người đại diện, ngày cấp phép hoạt động, cơ quan thuế quản lý trực tiếp, và trạng thái thuế hiện thời.
* **Đầu ra:** Thẻ thông tin trực quan hiển thị đầy đủ thông tin pháp nhân và huy hiệu trạng thái hoạt động.
