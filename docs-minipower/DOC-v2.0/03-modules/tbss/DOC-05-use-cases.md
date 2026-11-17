# DOC-05 — Ca sử dụng Phân hệ Quản lý Thông báo Sai sót (TBSS)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TBSS` — Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6.16), `SMI_Sơ đồ nghiệp vụ.json` (Trang 5), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Thông báo 04SS).

---

## 1. Danh sách Ca sử dụng (Use Cases)

| Mã Use Case | Tên Ca sử dụng | Tác nhân chính | Mức ưu tiên | Mô tả tóm tắt |
|:---:|---|:---:|:---:|---|
| **TBSS-UC-001** | Tra cứu & Lọc Danh sách Thông báo 04/SS | Kế toán viên | Must | Tìm kiếm các thông báo 04/SS Mua vào / Bán ra theo số thông báo, khoảng ngày gửi, CQT tiếp nhận. |
| **TBSS-UC-002** | Xem Bản thể hiện trực quan 04/SS | Kế toán viên | Must | Hiển thị mẫu in chuẩn của tờ Thông báo sai sót 04/SS với đầy đủ các cột chi tiết. |
| **TBSS-UC-003** | Tải Tệp XML & Bản in 04/SS | Kế toán viên | Should | Tải file XML gốc và file thể hiện của tờ thông báo 04/SS về máy tính. |
| **TBSS-UC-004** | Đồng bộ Thông báo 04/SS từ CQT | Hệ thống / Kế toán viên | Must | Tự động hoặc chủ động kéo dữ liệu thông báo 04/SS phát sinh trên Cổng Thuế về mSMI. |
| **TBSS-UC-005** | Truy xuất Liên kết Hóa đơn - 04/SS | Kế toán viên | Must | Từ tờ hóa đơn xem được thông báo 04/SS liên quan, hoặc từ thông báo 04/SS nhảy sang xem tờ hóa đơn gốc. |

---

## 2. Chi tiết Ca sử dụng

### 2.1. TBSS-UC-001: Tra cứu & Lọc Danh sách Thông báo 04/SS

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Người dùng đã đăng nhập hệ thống mSMI.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng chọn menu **Quản lý hóa đơn** -> **Thông báo hóa đơn sai sót (04/SS)**.
2. Người dùng chọn tab:
   - **Tab Thông báo Mua vào**: Các thông báo do nhà cung cấp gửi lên CQT liên quan đến hóa đơn của đơn vị.
   - **Tab Thông báo Bán ra**: Các thông báo do chính đơn vị gửi lên CQT.
3. Người dùng sử dụng các bộ lọc:
   - Khoảng ngày gửi thông báo (Từ ngày ... Đến ngày ...).
   - Số thông báo CQT cấp / Số thông báo của NNT.
   - Trạng thái xử lý của CQT (Đã tiếp nhận, Đã chấp nhận, Bị từ chối).
4. Hệ thống lọc và hiển thị danh sách dạng bảng: Số thông báo, Ngày thông báo, Tên NNT gửi, Mã CQT tiếp nhận, Số lượng hóa đơn sai sót trong tờ khai, Trạng thái CQT.

---

### 2.2. TBSS-UC-002: Xem Bản thể hiện trực quan 04/SS

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Đang ở màn hình danh sách thông báo 04/SS.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng bấm vào biểu tượng "Xem chi tiết" hoặc bấm vào dòng thông báo 04/SS.
2. Hệ thống mở cửa sổ (Modal / Tab mới) hiển thị bản thể hiện trực quan của Mẫu 04/SS-HĐĐT:
   - **Phần thông tin chung**: Tiêu đề "THÔNG BÁO HÓA ĐƠN ĐIỆN TỬ CÓ SAI SÓT", Tên NNT, MST, Địa chỉ, Số thông báo, Ngày gửi, Kính gửi CQT quản lý.
   - **Bảng danh sách chi tiết các hóa đơn sai sót**:
     - Cột 1: STT.
     - Cột 2: Mã cơ quan thuế cấp (nếu có).
     - Cột 3: Ký hiệu mẫu số hóa đơn và ký hiệu hóa đơn.
     - Cột 4: Số hóa đơn điện tử.
     - Cột 5: Ngày lập hóa đơn.
     - Cột 6: Loại áp dụng hóa đơn điện tử.
     - Cột 7: Tính chất thông báo (Hủy / Điều chỉnh / Thay thế / Giải trình).
     - Cột 8: Lý do sai sót cụ thể.
   - **Chữ ký điện tử**: Tên người ký, thời gian ký số.
3. Người dùng đọc đối chiếu thông tin và đóng cửa sổ khi xem xong.

---

### 2.3. TBSS-UC-003: Tải Tệp XML & Bản in 04/SS

- **Tác nhân:** Kế toán viên (ACT-002).
- **Mô tả:** Cho phép tải file XML thông điệp gốc của thông báo 04/SS (phục vụ đối soát kỹ thuật với ngành thuế) và in/tải bản PDF thể hiện để lưu trữ hồ sơ kế toán.

---

### 2.4. TBSS-UC-004: Đồng bộ Thông báo 04/SS từ CQT

- **Tác nhân:** Hệ thống (tiến trình tự động) hoặc Kế toán viên (kích hoạt thủ công).
- **Tiền điều kiện:** Đã cấu hình tài khoản CQT thành công.

#### Luồng sự kiện chính (Main Flow):
1. Khi đến lịch chạy nền hoặc khi người dùng bấm nút "Đồng bộ 04/SS":
2. Hệ thống gọi API sang Cổng HĐĐT của Tổng cục Thuế với khoảng thời gian được chỉ định.
3. Cổng Thuế trả về danh sách các thông báo 04/SS phát sinh.
4. Hệ thống tải toàn văn dữ liệu XML của từng thông báo 04/SS.
5. Lưu dữ liệu vào bảng `hdtbssrses`.
6. Thực hiện liên kết tự động tới các hóa đơn gốc trong kho dữ liệu mSMI (`TBSS-UC-005`).
7. Báo cáo số lượng thông báo 04/SS mới nhận được.

---

### 2.5. TBSS-UC-005: Truy xuất Liên kết Hóa đơn - 04/SS

- **Tác nhân:** Kế toán viên (ACT-002).
- **Mô tả:**
  - Tại màn hình danh sách Hóa đơn mua vào/bán ra: Với những hóa đơn có liên kết 04/SS, xuất hiện icon thông báo sai sót màu cam/đỏ. Bấm vào icon này sẽ mở thẳng bản thể hiện thông báo 04/SS liên quan.
  - Ngược lại, tại bản thể hiện thông báo 04/SS: Bấm vào dòng số hóa đơn sẽ mở màn hình chi tiết tờ hóa đơn gốc tương ứng.

