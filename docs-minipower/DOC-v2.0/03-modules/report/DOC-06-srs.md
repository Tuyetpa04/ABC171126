# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Báo cáo Thuế & Kết xuất Kế toán (REPORT)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `REPORT` — Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất Kế toán Động  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 9), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 4), `SMI_Sơ đồ nghiệp vụ.json` (Trang 11), `mSMI - Tài liệu thiết kế CSDL.docx` (`tokhaithues`, `reportconfigs`).

---

## 1. Yêu cầu Chức năng (Functional Requirements)

| Mã FR | Tên Chức năng | Mô tả chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **REPORT-FR-001** | Bảng kê Hóa đơn Gom nhóm Thuế | Tổng hợp hóa đơn theo kỳ, phân tách rõ ràng từng nhóm thuế suất (0%, 5%, 8%, 10%, không chịu thuế) kèm tổng tiền chưa thuế và tiền thuế. | Must | REPORT-UC-001<br>REPORT-BR-001, 005 |
| **REPORT-FR-002** | Tính toán Tự động Tờ khai 01/GTGT | Động cơ tính toán thu thập dữ liệu từ các hóa đơn đã gán kỳ kê khai để tự động điền các chỉ tiêu [21] đến [43] của Mẫu 01/GTGT. | Must | REPORT-UC-002<br>REPORT-BR-002 |
| **REPORT-FR-003** | Snapshot & Kết xuất XML HTKK | Cho phép lưu vết snapshot tờ khai vào bảng `tokhaithues` và xuất file XML theo chuẩn XSD của phần mềm HTKK Tổng cục Thuế. | Must | REPORT-UC-003<br>REPORT-BR-006 |
| **REPORT-FR-004** | Trình Cấu hình Xuất Excel Động | Cung cấp công cụ phân tích cú pháp biểu thức `calc()` và `formatDate()` để sinh file Excel tùy biến theo mẫu import của các PM kế toán (MISA/FAST). | Should | REPORT-UC-004<br>REPORT-BR-003, 004 |
| **REPORT-FR-005** | Biểu đồ & Báo cáo Giám sát Thuế | Báo cáo trực quan hóa biểu đồ doanh thu - chi phí qua các tháng và danh sách cảnh báo hóa đơn bị người bán hủy/thay thế/điều chỉnh trong kỳ. | Could | REPORT-UC-005<br>REPORT-BR-001 |

---

## 2. Đặc tả Chi tiết Từng Yêu cầu

### 2.1. REPORT-FR-001: Bảng kê Hóa đơn Gom nhóm Thuế
- **Dữ liệu đầu vào:**
  - `taxPeriod`: Kỳ tính thuế (ví dụ `2026-Q3` hoặc `2026-10`).
  - `type`: `PURCHASE` (Mua vào) hoặc `SOLD` (Bán ra).
- **Thuật toán xử lý:**
  - Lấy tất cả hóa đơn thỏa mãn: `taxPeriod == input` VÀ `isCancelDeclare != true` VÀ `status != "CANCELLED"`.
  - Duyệt qua từng dòng hàng hóa (`details`):
    - Phân nhóm theo thuế suất: `KCT` (Không chịu thuế), `0%`, `5%`, `8%`, `10%`, `Khác`.
    - Cộng dồn thành tiền chưa thuế (`thtien`) và tiền thuế (`tgtthue`) cho từng nhóm.
  - Tính tổng cộng toàn bảng kê:
    $$\text{TotalAmount} = \sum \text{thtien}; \quad \text{TotalVAT} = \sum \text{tgtthue}; \quad \text{TotalPayment} = \text{TotalAmount} + \text{TotalVAT}$$

### 2.2. REPORT-FR-002: Tính toán Tự động Tờ khai 01/GTGT
- **Logic tự động điền:**
  - Chỉ tiêu [22] = $\sum \text{thtien}$ của tất cả hóa đơn Mua vào trong kỳ.
  - Chỉ tiêu [23] = $\sum \text{tgtthue}$ của tất cả hóa đơn Mua vào trong kỳ.
  - Chỉ tiêu [25] = Thuế GTGT đầu vào hợp lệ được khấu trừ (mặc định = [23] trừ các hóa đơn bị đánh dấu không đủ điều kiện khấu trừ).
  - Chỉ tiêu [26] = Doanh thu bán ra nhóm KCT.
  - Chỉ tiêu [27] = Doanh thu bán ra nhóm 0%.
  - Chỉ tiêu [29], [30] = Doanh thu & thuế bán ra nhóm 5%.
  - Chỉ tiêu [31], [32] = Doanh thu & thuế bán ra nhóm 10%.
  - Chỉ tiêu [32a] = Doanh thu & thuế bán ra nhóm 8%.
  - Chỉ tiêu [34] = $[26] + [27] + [29] + [31] + [32a]$.
  - Chỉ tiêu [35] = $[30] + [32] + \text{thuế } [32a]$.
  - Chỉ tiêu [40] = $\max(0, [35] - [25] - [21])$.
  - Chỉ tiêu [43] = $\max(0, [25] + [21] - [35])$.

### 2.3. REPORT-FR-003: Snapshot & Kết xuất XML HTKK
- **Bảng CSDL `tokhaithues`:**
  - Lưu trường: `taxCode`, `taxPeriod`, `data` (object chứa toàn bộ các chỉ tiêu [21] đến [43]), `invoiceIds` (danh sách ID hóa đơn tham gia), `status` (Bản nháp / Đã chốt), `createdDate`, `updatedDate`.
- **Đầu ra XML:**
  - Cấu trúc XML đúng chuẩn thông điệp Tờ khai 01/GTGT của Tổng cục Thuế (phiên bản TT 80/2021/TT-BTC).
  - Thẻ gốc: `<HSoThueDTu>`, bao gồm `<TTinChung>` và `<CTieuTKhaiChinh>`.

### 2.4. REPORT-FR-004: Trình Cấu hình Xuất Excel Động
- **Cấu hình mẫu bảng kê (Cấu hình linh hoạt theo phần mềm kế toán):**
  - Mỗi mẫu cấu hình gồm: Tên phần mềm kế toán mục tiêu (MISA, FAST, Bravo, Vacom...) và Danh sách định nghĩa các cột xuất dữ liệu.
  - Mỗi cột xuất dữ liệu được định nghĩa bằng các thuộc tính nghiệp vụ:
    * **Tên tiêu đề cột:** Tên cột hiển thị trên file Excel (ví dụ: *"Số chứng từ"*, *"Ngày hóa đơn"*, *"Tiền thuế"*).
    * **Biểu thức/Quy tắc lấy dữ liệu:** Công thức ánh xạ dữ liệu (ví dụ: Lấy theo Số hóa đơn, Ngày lập, hoặc Tổng tiền thanh toán).
    * **Thứ tự hiển thị:** Vị trí sắp xếp cột từ trái sang phải trên bảng tính Excel (1, 2, 3...).
    * **Độ rộng cột:** Kích thước cột hiển thị (tính theo ký tự) để trình bày đẹp mắt khi mở file.
- **Bộ thông dịch biểu thức (Formula Parser):**
  - Hỗ trợ biến số ngữ cảnh: `detail` (chi tiết dòng hóa đơn), `invoice` (thông tin chung hóa đơn).
  - Hỗ trợ hàm định dạng `formatDate(field, pattern)`.
  - Hỗ trợ toán tử số học: `+`, `-`, `*`, `/`.
  - Tự động sinh file `.xlsx` bằng thư viện streaming Excel để đảm bảo không bị tràn bộ nhớ khi xuất hàng vạn dòng hóa đơn.

### 2.5. REPORT-FR-005: Biểu đồ & Báo cáo Giám sát Thuế (Dashboard Analytics)
- **Mục tiêu:** Cung cấp báo cáo phân tích trực quan hóa số liệu doanh thu, chi phí và thuế GTGT biến động theo thời gian, hỗ trợ kế toán trưởng và ban giám đốc ra quyết định.
- **Xử lý:**
  1. Tổng hợp dữ liệu theo chuỗi thời gian (12 tháng gần nhất hoặc 4 quý trong năm tài chính):
     - Biểu đồ cột chồng (Stacked Bar Chart): Doanh số bán ra phân theo từng nhóm thuế suất (0%, 5%, 8%, 10%).
     - Biểu đồ đường (Line Chart): Xu hướng biến động thuế GTGT đầu vào được khấu trừ so với thuế GTGT đầu ra phải nộp.
  2. Báo cáo Giám sát Hóa đơn Rủi ro theo kỳ:
     - Thống kê tỷ lệ hóa đơn mua vào có cảnh báo rủi ro (DN bỏ trốn, Blacklist 1.500 DN).
     - Danh sách hóa đơn mua vào bị bên bán hủy hoặc sửa đổi sau thời điểm kê khai để kế toán chủ động lập tờ khai điều chỉnh bổ sung kịp thời.
- **Đầu ra:** Bảng điều khiển Dashboard tương tác trực quan với các widget biểu đồ và danh sách cảnh báo có thể nhấp chuột để chuyển nhanh sang màn hình xử lý chi tiết.


