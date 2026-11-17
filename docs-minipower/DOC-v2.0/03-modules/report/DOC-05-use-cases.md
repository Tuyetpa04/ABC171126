# DOC-05 — Ca sử dụng Phân hệ Báo cáo Thuế & Kết xuất Kế toán (REPORT)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `REPORT` — Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất Kế toán Động  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 9), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 4), `SMI_Sơ đồ nghiệp vụ.json` (Trang 11), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Báo cáo, Tờ khai 01GTGT).

---

## 1. Danh sách Ca sử dụng (Use Cases)

| Mã Use Case | Tên Ca sử dụng | Tác nhân chính | Mức ưu tiên | Mô tả tóm tắt |
|:---:|---|:---:|:---:|---|
| **REPORT-UC-001** | Lập Bảng kê Hóa đơn Mua vào / Bán ra | Kế toán viên | Must | Kết xuất bảng tổng hợp hóa đơn mua vào hoặc bán ra theo kỳ kê khai, phân nhóm thuế suất. |
| **REPORT-UC-002** | Tự động lập Tờ khai Thuế GTGT 01/GTGT | Kế toán viên | Must | Hệ thống tự động tính các chỉ tiêu [21] - [43] từ danh sách hóa đơn đã gán kỳ kê khai. |
| **REPORT-UC-003** | Khóa sổ & Xuất XML Tờ khai nộp HTKK | Kế toán viên | Must | Chốt số liệu tờ khai, lưu snapshot vào CSDL và xuất file XML để nộp qua phần mềm HTKK của CQT. |
| **REPORT-UC-004** | Cấu hình Mẫu Excel Kế toán Động | Kế toán viên / Quản trị viên | Should | Thiết lập công thức hàm `calc()`, `formatDate()` để xuất bảng kê Excel khớp mẫu PM kế toán (MISA/FAST). |
| **REPORT-UC-005** | Xem Báo cáo Biểu đồ Phân tích Mua - Bán | Kế toán trưởng / Ban Giám đốc | Could | Xem biểu đồ so sánh tương quan doanh thu, chi phí thuế, và danh sách hóa đơn bị hủy/thay thế gần đây. |

---

## 2. Chi tiết Ca sử dụng

### 2.1. REPORT-UC-001: Lập Bảng kê Hóa đơn Mua vào / Bán ra

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Đã có hóa đơn trong kho và được gán kỳ kê khai.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng vào menu **Báo cáo** -> **Bảng kê hóa đơn**.
2. Người dùng chọn loại bảng kê: **Bảng kê mua vào** hoặc **Bảng kê bán ra**.
3. Người dùng chọn kỳ kê khai: Theo Tháng (ví dụ `Tháng 10/2026`) hoặc Theo Quý (ví dụ `Quý 3/2026`).
4. Người dùng bấm **Xem báo cáo**.
5. Hệ thống truy vấn toàn bộ hóa đơn có `taxPeriod` khớp với kỳ đã chọn (loại trừ các hóa đơn bị hủy hoặc hủy kê khai).
6. Hệ thống gom nhóm danh sách hóa đơn theo các nhóm thuế suất (0%, 5%, 8%, 10%, không chịu thuế).
7. Hệ thống hiển thị bảng kê với dòng tổng tiền chưa thuế và tổng tiền thuế của từng nhóm và tổng cộng toàn kỳ.
8. Người dùng có thể bấm nút **Xuất Excel** để tải tệp bảng kê về máy.

---

### 2.2. REPORT-UC-002: Tự động lập Tờ khai Thuế GTGT 01/GTGT

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Người dùng đã hoàn tất việc gán kỳ kê khai cho các hóa đơn mua vào và bán ra của kỳ tính thuế.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng vào menu **Báo cáo** -> **Tờ khai thuế GTGT (01/GTGT)**.
2. Người dùng chọn Kỳ kê khai thuế (ví dụ: `Quý 3/2026`).
3. Hệ thống tự động thu thập số liệu hóa đơn mua vào và bán ra trong kỳ và tính toán:
   - Chỉ tiêu [22] (Tổng giá trị HHDV mua vào), [23] (Tổng thuế GTGT mua vào), [25] (Thuế đầu vào được khấu trừ).
   - Chỉ tiêu [26], [27], [29], [30], [31], [32], [32a], [34], [35] cho nhóm bán ra.
   - Tự động lấy Chỉ tiêu [21] từ tờ khai kỳ trước (nếu có).
   - Tính toán Chỉ tiêu [40] (Số thuế phải nộp) hoặc [43] (Số thuế còn được khấu trừ).
4. Hệ thống điền đầy đủ các số liệu lên biểu mẫu Tờ khai 01/GTGT chuẩn.
5. Người dùng kiểm tra, có thể điều chỉnh thủ công các chỉ tiêu đặc thù (như phân bổ khấu trừ [24], điều chỉnh giảm [37], tăng [38]) nếu cần.

---

### 2.3. REPORT-UC-003: Khóa sổ & Xuất XML Tờ khai nộp HTKK

- **Tác nhân:** Kế toán viên (ACT-002).
- **Tiền điều kiện:** Đã xem và hoàn tất kiểm tra số liệu Tờ khai 01/GTGT.

#### Luồng sự kiện chính (Main Flow):
1. Tại màn hình Tờ khai 01/GTGT, người dùng bấm nút **Lưu tờ khai**.
2. Hệ thống tạo một bản ghi snapshot trong bảng `tokhaithues` lưu toàn bộ thông số các chỉ tiêu và danh sách ID hóa đơn cấu thành tại thời điểm lưu.
3. Người dùng bấm nút **Xuất XML (HTKK)**.
4. Hệ thống sinh file XML theo đúng cấu trúc XSD chuẩn của phần mềm Hỗ trợ Kê khai của Tổng cục Thuế.
5. Trình duyệt tải file XML về máy tính. Kế toán viên có thể mở file này trực tiếp trên phần mềm HTKK hoặc nộp trực tiếp qua Cổng Thuế điện tử (thuedientu.gdt.gov.vn).

---

### 2.4. REPORT-UC-004: Cấu hình Mẫu Excel Kế toán Động

- **Tác nhân:** Kế toán viên (ACT-002) hoặc Quản trị viên (ACT-001).

#### Luồng sự kiện chính (Main Flow):
1. Người dùng vào menu **Cấu hình** -> **Cấu hình Bảng kê Kế toán**.
2. Người dùng chọn phần mềm kế toán cần tích hợp (MISA SME / MISA AMIS / FAST Accounting / Bravo / Tự định nghĩa).
3. Hệ thống hiển thị danh sách các cột Excel dự kiến xuất ra:
   - Tên cột hiển thị (Header).
   - Thứ tự cột (STT).
   - Công thức tính toán (Formula).
4. Người dùng có thể:
   - Thêm cột mới hoặc xóa cột không cần thiết.
   - Nhập công thức: ví dụ cột "Số chứng từ" gán `calc(detail.shdon)`, cột "Ngày hạch toán" gán `formatDate(detail.tdlap, 'DD/MM/YYYY')`, cột "Tài khoản Nợ" gán `calc('1561')`.
5. Người dùng bấm **Kiểm tra công thức** để xem trước dữ liệu mẫu (Preview 3 dòng đầu tiên).
6. Người dùng bấm **Lưu mẫu cấu hình**.
7. Khi xuất Excel tại màn hình Hóa đơn hoặc Bảng kê, hệ thống áp dụng cấu hình này để tạo file Excel.

---

### 2.5. REPORT-UC-005: Xem Biểu đồ & Báo cáo Giám sát Thuế (Dashboard Analytics)

- **Tác nhân:** Kế toán trưởng (ACT-002), Giám đốc tài chính (ACT-001).
- **Tiền điều kiện:** Doanh nghiệp đã đồng bộ và phát sinh dữ liệu hóa đơn mua vào, bán ra trong năm tài chính.

#### Luồng sự kiện chính (Main Flow):
1. Người dùng vào menu **Báo cáo** -> **Giám sát Thuế & Dashboard**.
2. Người dùng chọn năm tài chính hoặc khoảng thời gian phân tích (ví dụ: `Năm 2026`).
3. Hệ thống tổng hợp dữ liệu và trực quan hóa:
   - Biểu đồ phân tích doanh thu bán ra theo từng nhóm thuế suất (0%, 5%, 8%, 10%).
   - Biểu đồ xu hướng đối chiếu thuế GTGT đầu vào được khấu trừ so với thuế GTGT đầu ra phải nộp qua các tháng.
   - Thống kê tỷ lệ hóa đơn có rủi ro thuế (doanh nghiệp bỏ trốn, Blacklist 1.500 DN).
   - Danh sách hóa đơn mua vào bị bên bán hủy hoặc sửa đổi trong kỳ kê khai hiện tại.
4. Người dùng có thể nhấp chuột vào từng phần tử biểu đồ để lọc nhanh danh sách hóa đơn chi tiết cấu thành.


