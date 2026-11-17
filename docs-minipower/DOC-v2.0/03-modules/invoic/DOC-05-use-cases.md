# DOC-05 — Ca sử dụng Phân hệ Quản lý Hóa đơn (INVOIC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `INVOIC` — Quản lý Hóa đơn Mua vào / Bán ra  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6), `App hóa đơn vào_ ra SMI.xlsx` (Sheet Hóa đơn).

---

## 1. Danh sách Ca sử dụng (Use Cases Register)

| UC ID | Tên Use Case | Tác nhân chính | Mức ưu tiên | Trace (BR / FR) |
|:---:|---|:---:|:---:|:---:|
| **INVOIC-UC-001** | Tra cứu & Lọc danh sách hóa đơn | ACT-001 (Kế toán) | Must | INVOIC-BR-001, BR-002<br>INVOIC-FR-001 |
| **INVOIC-UC-002** | Xem bản thể hiện hóa đơn (HTML/PDF) | ACT-001, ACT-002 | Must | INVOIC-FR-002 |
| **INVOIC-UC-003** | Xem cấu trúc file XML gốc | ACT-001 (Kế toán) | Should | INVOIC-FR-002 |
| **INVOIC-UC-004** | Tải hóa đơn (Đơn lẻ & Hàng loạt ZIP) | ACT-001 (Kế toán) | Must | INVOIC-BR-004, BR-007<br>INVOIC-FR-003 |
| **INVOIC-UC-005** | Gán nhãn phân loại hóa đơn (Tags) | ACT-001 (Kế toán) | Should | INVOIC-BR-008<br>INVOIC-FR-004 |
| **INVOIC-UC-006** | Gán ngày kê khai thuế / Hủy kê khai | ACT-001 (Kế toán) | Must | INVOIC-BR-003<br>INVOIC-FR-005 |
| **INVOIC-UC-007** | Đính kèm tài liệu chứng từ | ACT-001 (Kế toán) | Could | INVOIC-BR-006<br>INVOIC-FR-006 |
| **INVOIC-UC-008** | Ghi chú nội bộ hóa đơn | ACT-001 (Kế toán) | Could | INVOIC-FR-007 |

---

## 2. Đặc tả Chi tiết Ca sử dụng

### 2.1. INVOIC-UC-001: Tra cứu & Lọc Danh sách Hóa đơn
* **Actor**: ACT-001 (Kế toán viên).
* **Preconditions**: Kế toán đã đăng nhập, chọn phân hệ Hóa đơn Mua vào hoặc Bán ra.
* **Basic Path**:
  1. Kế toán truy cập màn hình Danh sách hóa đơn.
  2. Kế toán thiết lập các tiêu chí lọc: Khoảng ngày lập (`tdlap`), trạng thái kê khai, tên/MST người bán, thẻ nhãn hoặc trạng thái rủi ro.
  3. Kế toán bấm **"Tìm kiếm"**.
  4. Hệ thống tải dữ liệu phân trang và hiển thị danh sách hóa đơn tương ứng kèm thông tin tóm tắt: Ký hiệu, số HĐ, ngày lập, người bán/mua, tổng tiền, tiền thuế, trạng thái CQT.
* **Alternate Path**: Nếu không tìm thấy kết quả phù hợp, hiển thị thông báo "Không có dữ liệu phù hợp với bộ lọc".

---

### 2.2. INVOIC-UC-002: Xem Bản thể hiện Hóa đơn (HTML/PDF)
* **Actor**: ACT-001 (Kế toán), ACT-002 (Kế toán trưởng).
* **Basic Path**:
  1. Kế toán nhấp vào biểu tượng xem (hoặc số hóa đơn) trên danh sách.
  2. Hệ thống kiểm tra dữ liệu chi tiết dòng hàng (`assets.detail.done`).
     - *Nếu đã có chi tiết*: Render tức thì giao diện đồ họa hóa đơn đầy đủ (tiêu đề, danh mục hàng hóa, đơn giá, thuế suất, tổng tiền, thông tin chữ ký số).
     - *Nếu chưa có chi tiết*: Tự động kích hoạt cơ chế ưu tiên cào detail `PATCH /:id/priority` (`INVOIC-BR-007`) và hiển thị thông báo tải dữ liệu trong giây lát.
  3. Người dùng có thể thực hiện in trực tiếp ra máy in hoặc chuyển đổi xem giữa các hóa đơn liên quan.

---

### 2.3. INVOIC-UC-003: Xem Cấu trúc File XML Gốc
* **Actor**: ACT-001 (Kế toán viên / Chuyên viên đối soát).
* **Basic Path**:
  1. Tại màn hình xem chi tiết hóa đơn, người dùng bấm nút **"Xem XML gốc"**.
  2. Hệ thống mở popup hiển thị toàn bộ nội dung tệp tin XML nguyên vẹn theo chuẩn QĐ 1450/QĐ-TCT.
  3. Người dùng kiểm tra các thẻ thông tin kỹ thuật, khối dữ liệu `<DLHDon>` và thẻ băm chữ ký số `<Signature>`.

---

### 2.4. INVOIC-UC-004: Tải Hóa đơn Đơn lẻ & Hàng loạt (Batch Download ZIP)
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán lọc ra danh sách hóa đơn cần tải (ví dụ: hóa đơn trong Quý 1/2026).
  2. Kế toán tích chọn các hóa đơn cần tải (tối đa 100 hóa đơn/lần).
  3. Kế toán bấm **"Tải hóa đơn"**, chọn định dạng: *Chỉ XML*, *Chỉ PDF* hoặc *Cả XML và PDF*.
  4. Hệ thống thực hiện giải thuật Fallback 5 tầng (`INVOIC-BR-004`), nén các tệp tin vào thư mục `hoa-don/` của file ZIP.
  5. Trình duyệt tự động tải tệp tin ZIP về máy với tên chuẩn `{MST}_{LoaiHD}_{TuNgay}_{DenNgay}.zip`.
* **Alternate Path (AF-1 - Khuyết tệp)**: Nếu có hóa đơn chưa kịp cào xong file PDF/XML, hệ thống vẫn nén các hóa đơn đã có và sinh file log `BaoCaoLoi_MissingAssets.txt` báo danh sách số HĐ bị khuyết.

---

### 2.5. INVOIC-UC-005: Gán Nhãn Phân loại Hóa đơn (Tags)
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán tích chọn một hoặc nhiều hóa đơn trên danh sách.
  2. Kế toán bấm nút **"Gán nhãn"** trên thanh công cụ.
  3. Hệ thống hiển thị hộp thoại chọn các nhãn màu quản trị (ví dụ: *Chi phí tiếp khách*, *Vật tư dự án X*).
  4. Kế toán chọn một hoặc nhiều nhãn và xác nhận.
  5. Hệ thống lưu mảng `tags` vào CSDL (`INVOIC-BR-008`) và hiển thị huy hiệu nhãn màu tương ứng trên bảng.

---

### 2.6. INVOIC-UC-006: Gán Ngày Kê khai Thuế / Hủy Kê khai
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán chọn các hóa đơn hợp lệ cần kê khai thuế.
  2. Kế toán bấm nút **"Gán ngày kê khai"**.
  3. Hệ thống hiển thị hộp thoại chọn ngày (mặc định lấy ngày cuối kỳ tính thuế hiện hành).
  4. Kế toán xác nhận; hệ thống lưu `data.nkekhai`, đổi trạng thái hiển thị thành "Đã kê khai".
  5. Số liệu hóa đơn tự động được tổng hợp vào Bảng kê và Tờ khai Mẫu 01/GTGT của kỳ đó (`INVOIC-BR-003`).
* **Alternate Path (Hủy kê khai)**: Kế toán bấm "Hủy kê khai", hệ thống đặt `data.nkekhai = null`, loại hóa đơn ra khỏi kỳ kê khai thuế.

---

### 2.7. INVOIC-UC-007: Đính kèm Tài liệu Chứng từ (Attachments)
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Tại giao diện chi tiết hóa đơn, kế toán chuyển sang tab **"Tài liệu đính kèm"**.
  2. Kế toán kéo thả hoặc chọn tệp tin chứng từ từ máy tính (hợp đồng, biên bản bàn giao, phiếu chi $\le 10\text{ MB}$).
  3. Hệ thống kiểm tra định dạng hợp lệ, tự động mã hóa đối xứng AES-256-CBC thành tệp `.atm` và lưu vào đĩa cứng (`INVOIC-BR-006`).
  4. Tệp tin hiển thị trong danh mục đính kèm kèm kích thước và ngày tải lên.

---

### 2.8. INVOIC-UC-008: Ghi chú Nội bộ trên Hóa đơn
* **Actor**: ACT-001 (Kế toán viên).
* **Basic Path**:
  1. Kế toán nhấp vào biểu tượng "Ghi chú" trên dòng hóa đơn tương ứng.
  2. Nhập nội dung diễn giải hạch toán hoặc thông tin trao đổi nội bộ.
  3. Bấm **"Lưu ghi chú"**; hệ thống cập nhật trường `comment` vào cơ sở dữ liệu (`INVOIC-FR-007`).
  4. Biểu tượng ghi chú đổi màu để các thành viên khác trong phòng kế toán nhận biết.
