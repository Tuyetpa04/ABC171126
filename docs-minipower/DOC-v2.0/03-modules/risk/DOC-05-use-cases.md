# DOC-05 — Ca sử dụng Phân hệ Kiểm soát Rủi ro Thuế (RISK)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `RISK` — Kiểm soát & Cảnh báo Rủi ro Thuế  
> **Căn cứ tài liệu gốc:** `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 6.13, 6.14).

---

## 1. Danh sách Ca sử dụng (Use Cases Register)

| UC ID | Tên Use Case | Tác nhân chính | Mức ưu tiên | Trace (BR / FR) |
|:---:|---|:---:|:---:|:---:|
| **RISK-UC-001** | Xem cảnh báo rủi ro trên hóa đơn | ACT-001 (Kế toán), ACT-002 (Kế toán trưởng) | Must | RISK-BR-001 đến BR-006<br>RISK-FR-001 đến FR-004 |
| **RISK-UC-002** | Tra cứu trạng thái hoạt động MST/CCCD | ACT-001 (Kế toán) | Must | RISK-BR-001<br>RISK-FR-005 |
| **RISK-UC-003** | Đối soát lịch sử hóa đơn bị thay đổi | ACT-001, ACT-002 | Must | RISK-BR-003<br>RISK-FR-003 |
| **RISK-UC-004** | Ẩn cảnh báo hóa đơn đã xác minh | ACT-001 (Kế toán) | Should | RISK-FR-001<br>INVOIC-BR-005 |

---

## 2. Đặc tả Chi tiết Ca sử dụng

### 2.1. RISK-UC-001: Xem Cảnh báo Rủi ro trên Hóa đơn Mua vào
* **Actor**: ACT-001 (Kế toán viên), ACT-002 (Kế toán trưởng).
* **Preconditions**: Hóa đơn mua vào đã được đồng bộ về hệ thống và chạy qua engine đánh giá rủi ro.
* **Basic Path**:
  1. Người dùng mở màn hình Danh sách hóa đơn mua vào hoặc phân hệ Giám sát Rủi ro.
  2. Hệ thống hiển thị các cột chỉ số rủi ro: Huy hiệu cờ cảnh báo (Đỏ/Cam/Vàng).
  3. Người dùng nhấp vào biểu tượng cảnh báo trên dòng hóa đơn.
  4. Hệ thống hiển thị popup chi tiết phân tích rủi ro:
     - Tình trạng pháp lý người bán tại thời điểm xuất HĐ (`RISK-BR-001`, `RISK-BR-004`).
     - Tình trạng trong danh sách đen 1.500 DN rủi ro cao (`RISK-BR-002`).
     - Kết quả thẩm định chữ ký số và tính toàn vẹn XML (`RISK-BR-005`).
     - Kết quả đối soát tên pháp nhân và địa chỉ (`RISK-BR-006`).
* **Outputs**: Báo cáo chi tiết các tiêu chí cảnh báo giúp kế toán đưa ra quyết định chấp nhận hay loại trừ hóa đơn khỏi kỳ kê khai.

---

### 2.2. RISK-UC-002: Tra cứu Trạng thái Hoạt động MST/CCCD
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán nhập Mã số thuế hoặc số CCCD của doanh nghiệp / người đại diện pháp luật vào ô tìm kiếm.
  2. Bấm nút **"Kiểm tra MST"**.
  3. Hệ thống gửi yêu cầu tra cứu sang cổng dữ liệu Thuế (API `SearchTaxCodeV2`).
  4. Hệ thống hiển thị bảng thông tin chi tiết:
     - Tên doanh nghiệp, địa chỉ trụ sở đăng ký.
     - Tên người đại diện pháp luật, số CCCD/CMND.
     - Ngày bắt đầu hoạt động, ngày cấp giấy phép.
     - Trạng thái hoạt động: *Đang hoạt động (Màu xanh)*, *Tạm ngừng kinh doanh (Màu cam)*, *Ngừng hoạt động/Khóa MST (Màu đỏ)*.
     - Cơ quan thuế quản lý trực tiếp.
* **Alternate Path (AF-1 - MST không tồn tại)**: Nếu CQT báo MST không tồn tại, hiển thị thông báo "Mã số thuế không hợp lệ hoặc chưa được ghi nhận trên cổng Thuế".

---

### 2.3. RISK-UC-003: Đối soát Lịch sử Hóa đơn bị Thay đổi / Điều chỉnh
* **Actor**: ACT-001 (Kế toán), ACT-002 (Kế toán trưởng).
* **Basic Path**:
  1. Người dùng truy cập tab **"Hóa đơn có thay đổi trạng thái"**.
  2. Danh sách hiển thị các hóa đơn mua vào bị người bán hủy (`tthai = 4`), thay thế (`tthai = 3`) hoặc điều chỉnh (`tthai = 2`).
  3. Người dùng nhấp vào nút **"Xem liên kết"**; hệ thống mở giao diện đối soát song song giữa hóa đơn gốc và hóa đơn điều chỉnh/thay thế mới.
  4. Người dùng kiểm tra chênh lệch số lượng, đơn giá, tiền thuế để phục vụ điều chỉnh sổ sách kế toán.

---

### 2.4. RISK-UC-004: Ẩn Cảnh báo Hóa đơn đã Xác minh Thực tế
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán kiểm tra hồ sơ thực tế của một hóa đơn đang bị cảnh báo rủi ro (ví dụ: đối tác chỉ thay đổi địa điểm kinh doanh nhưng CQT cập nhật chậm).
  2. Kế toán bấm nút **"Ẩn cảnh báo"** trên màn hình chi tiết hóa đơn.
  3. Hệ thống yêu cầu nhập lý do xác minh (ví dụ: *Đã có biên bản kiểm tra thực tế tại kho hàng*).
  4. Hệ thống lưu `warning.hide_status = true`, chuyển trạng thái hiển thị thành "Hợp lệ", đồng thời bảo lưu trạng thái gốc vào `warning.origin_status` (`INVOIC-BR-005`).
  5. Hóa đơn được loại khỏi danh sách cảnh báo cần xử lý trên Dashboard.
