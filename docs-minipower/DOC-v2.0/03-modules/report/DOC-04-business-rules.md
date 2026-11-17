# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Báo cáo Thuế & Kết xuất Kế toán (REPORT)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `REPORT` — Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất Kế toán Động  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 9), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 4), `SMI_Sơ đồ nghiệp vụ.json` (Trang 11), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Báo cáo, Tờ khai 01GTGT), `mSMI - Tài liệu thiết kế CSDL.docx` (`tokhaithues`, `reportconfigs`).  
> **Căn cứ mã nguồn thực tế:** `backend/service_report/`, `backend/routes/reports/`, `backend/models/tokhaithue.js`.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| ID | Tên Quy tắc | Mô tả Tóm tắt Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **REPORT-BR-001** | Phân nhóm Thuế suất Bảng kê | Bảng kê Hóa đơn Mua vào / Bán ra bắt buộc gom nhóm chi tiết theo các mức thuế suất: Không chịu thuế, 0%, 5%, 8%, 10% và Thuế suất khác; tính tổng tiền chưa thuế và tiền thuế tương ứng. | Computation | Must | REPORT-UC-001<br>REPORT-FR-001 |
| **REPORT-BR-002** | Công thức Tính Tờ khai 01/GTGT | Tự động tính toán các chỉ tiêu từ [21] đến [43] của Tờ khai Thuế GTGT Mẫu 01/GTGT căn cứ trên các hóa đơn được gán kỳ kê khai và đủ điều kiện khấu trừ. | Computation | Must | REPORT-UC-002<br>REPORT-FR-002 |
| **REPORT-BR-003** | Cú pháp Công thức Động `calc()` & `formatDate()` | Cột Excel xuất phần mềm kế toán hỗ trợ định nghĩa linh hoạt qua cú pháp hàm `calc(expression)` và `formatDate(field, format)`. Hỗ trợ các phép toán số học `+`, `-`, `*`, `/` và nối chuỗi ký tự. | Computation | Must | REPORT-UC-004<br>REPORT-FR-004 |
| **REPORT-BR-004** | Phân cấp Mẫu Báo cáo Kế toán | Cung cấp 2 tầng cấu hình mẫu xuất Excel: Cấu hình Mẫu chuẩn hệ thống (dành cho các phần mềm MISA, FAST, Bravo...) và Cấu hình Người dùng tùy biến theo tài khoản. | Authorization | Should | REPORT-UC-004<br>REPORT-FR-004 |
| **REPORT-BR-005** | Điều kiện Đưa vào Kỳ Báo cáo | Chỉ những hóa đơn có trường `taxPeriod` trùng khớp với kỳ báo cáo và cờ `isCancelDeclare = false` mới được tính vào số liệu bảng kê và tờ khai thuế. | Validation | Must | REPORT-UC-001<br>REPORT-FR-001 |
| **REPORT-BR-006** | Lưu trữ Snapshot & Xuất XML HTKK | Cho phép lưu vết snapshot tờ khai theo kỳ vào bảng `tokhaithues` để bảo toàn dữ liệu lịch sử và kết xuất tệp XML tương thích phần mềm Hỗ trợ Kê khai (HTKK) của Tổng cục Thuế. | Integrity | Must | REPORT-UC-003<br>REPORT-FR-003 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ (Detailed Business Rules Specification)

### 2.1. REPORT-BR-001: Phân nhóm Thuế suất Bảng kê Mua vào / Bán ra
* **Statement**: Bảng kê Hóa đơn Mua vào và Bán ra bắt buộc phải bóc tách và tổng hợp dữ liệu theo 6 nhóm thuế suất chuẩn quy định tại Luật Thuế GTGT:
  1. Nhóm 1: Hàng hóa dịch vụ Không chịu thuế (KCT).
  2. Nhóm 2: Thuế suất 0% (Hàng hóa xuất khẩu, vận tải quốc tế).
  3. Nhóm 3: Thuế suất 5% (Mặt hàng thiết yếu, nông sản sơ chế, y tế...).
  4. Nhóm 4: Thuế suất 8% (Chính sách giảm thuế GTGT theo Nghị quyết Quốc hội).
  5. Nhóm 5: Thuế suất 10% (Thuế suất phổ thông).
  6. Nhóm 6: Thuế suất khác / Không tính thuế.
* **Condition**: Khi người dùng mở xem hoặc xuất file Excel Bảng kê Hóa đơn Mua vào / Bán ra theo kỳ kê khai (Tháng/Quý).
* **Action / Algorithm**:
  - Với mỗi dòng chi tiết hàng hóa (`detail`) của các hóa đơn có `data.nkekhai` thuộc kỳ:
    * Ánh xạ trường thuế suất `tsuat`: Chuẩn hóa các biểu diễn `"10%"`, `"10"`, `0.1` về nhóm chuẩn `10%`.
    * Cộng dồn doanh số chưa thuế: $\text{TotalNet}_{\text{rate}} = \sum \text{thtien}$.
    * Cộng dồn tiền thuế GTGT: $\text{TotalVAT}_{\text{rate}} = \sum \text{tgtthue}$.
  - Tính dòng tổng cộng toàn bảng kê:
    $$\text{GrandTotalNet} = \sum_{\text{all rates}} \text{TotalNet}_{\text{rate}}, \quad \text{GrandTotalVAT} = \sum_{\text{all rates}} \text{TotalVAT}_{\text{rate}}$$
* **Schema / Fields**: `invoiceraws.detail.{tsuat, thtien, tgtthue}`, `invoiceraws.data.nkekhai`.

---

### 2.2. REPORT-BR-002: Ma trận Công thức Tờ khai Mẫu 01/GTGT
Căn cứ Thông tư 80/2021/TT-BTC, hệ thống tự động tổng hợp số liệu lên Tờ khai Thuế GTGT Mẫu 01/GTGT:

| Chỉ tiêu | Tên Chỉ tiêu trên Tờ khai 01/GTGT | Nguồn dữ liệu & Công thức tính toán |
|:---:|---|---|
| **[21]** | Thuế GTGT chưa khấu trừ hết kỳ trước chuyển sang | Lấy từ Chỉ tiêu [43] của Tờ khai kỳ trước liền kề đã chốt hoặc do Kế toán viên nhập tay. |
| **[22]** | Tổng giá trị HHDV mua vào trong kỳ | Tổng tiền chưa thuế của tất cả HĐ Mua vào có gán kỳ tính thuế hiện tại và hợp lệ. |
| **[23]** | Tổng thuế GTGT của HHDV mua vào | Tổng tiền thuế GTGT của tất cả HĐ Mua vào trong kỳ. |
| **[24]** | Thuế GTGT của HHDV mua vào dùng cho SXKD chịu thuế và không chịu thuế | Mặc định tính bằng Chỉ tiêu [23] (hoặc cho phép Kế toán điều chỉnh phân bổ). |
| **[25]** | Tổng số thuế GTGT đầu vào được khấu trừ kỳ này | Tổng thuế GTGT của các hóa đơn mua vào hợp lệ, phục vụ hoạt động SXKD chịu thuế (loại trừ các hóa đơn bị loại khấu trừ hoặc doanh nghiệp bỏ trốn). |
| **[26]** | HHDV bán ra không chịu thuế GTGT | Tổng doanh thu chưa thuế của HĐ Bán ra thuộc nhóm thuế suất "Không chịu thuế". |
| **[27]** | HHDV bán ra chịu thuế suất 0% | Tổng doanh thu chưa thuế của HĐ Bán ra thuộc nhóm thuế suất "0%". |
| **[29]** | Doanh thu HHDV bán ra chịu thuế suất 5% | Tổng doanh thu chưa thuế của HĐ Bán ra thuộc nhóm thuế suất "5%". |
| **[30]** | Thuế GTGT của HHDV bán ra chịu thuế suất 5% | Tổng tiền thuế GTGT của các HĐ Bán ra có thuế suất "5%". |
| **[31]** | Doanh thu HHDV bán ra chịu thuế suất 10% | Tổng doanh thu chưa thuế của HĐ Bán ra thuộc nhóm thuế suất "10%". |
| **[32]** | Thuế GTGT của HHDV bán ra chịu thuế suất 10% | Tổng tiền thuế GTGT của các HĐ Bán ra có thuế suất "10%". |
| **[32a]** | Doanh thu và thuế GTGT bán ra áp dụng thuế suất khác (8%) | Tổng doanh thu và thuế GTGT của HĐ Bán ra thuộc diện giảm thuế theo chính sách 8%. |
| **[34]** | Tổng doanh thu HHDV bán ra | $[34] = [26] + [27] + [28] + [29] + [31] + [32a]$ |
| **[35]** | Tổng thuế GTGT của HHDV bán ra | $[35] = [30] + [32] + [33]$ |
| **[40]** | Thuế GTGT phải nộp trong kỳ | $\text{Nếu } [35] - ([25] + [21]) > 0 \implies [40] = [35] - ([25] + [21])$; ngược lại $[40] = 0$. |
| **[43]** | Thuế GTGT còn được khấu trừ chuyển kỳ sau | $\text{Nếu } ([25] + [21]) - [35] > 0 \implies [43] = ([25] + [21]) - [35]$; ngược lại $[43] = 0$. |

---

### 2.3. REPORT-BR-003: Cú pháp Công thức Xuất Excel Kế toán Động
* **Statement**: Cột trong mẫu xuất Excel kế toán hỗ trợ định nghĩa linh hoạt bằng các biểu thức ánh xạ động, cho phép doanh nghiệp tùy biến khớp định dạng nhập liệu của mọi phần mềm kế toán trên thị trường.
* **Cú pháp hàm `calc()`**:
  - Lấy trường dữ liệu hóa đơn: `calc(detail.shdon)` $\rightarrow$ Số hóa đơn.
  - Lấy hằng số chuỗi: `calc('HDBR')` hoặc `calc('131')` (mã tài khoản kế toán).
  - Phép tính số học: `calc(detail.thtien + detail.tgtthue)` $\rightarrow$ Tổng tiền thanh toán.
  - Nối chuỗi ký tự: `calc(detail.khmshdon + detail.khhdon)` $\rightarrow$ Mẫu số ghép ký hiệu.
* **Cú pháp hàm `formatDate()`**:
  - Cú pháp: `formatDate(field, 'FORMAT_PATTERN')`.
  - Ví dụ: `formatDate(detail.tdlap, 'DD/MM/YYYY')` xuất ra `28/10/2026`.
  - Hỗ trợ các định dạng: `DD/MM/YYYY`, `YYYY-MM-DD`, `DD-MM-YYYY`.
* **Schema / Fields**: `reportconfigs.columns.{column_name, expression, format}`.

---

### 2.4. REPORT-BR-004: Phân cấp Mẫu Báo cáo Kế toán (System & Custom Templates)
* **Statement**: Hệ thống phân cấp rõ 2 cấp độ mẫu cấu hình xuất dữ liệu kế toán: Mẫu chuẩn hệ thống (Read-Only) và Mẫu người dùng tự định nghĩa (User Customization).
* **Condition**: Khi người dùng vào màn hình cấu hình xuất Excel hoặc thực hiện xuất dữ liệu sang phần mềm kế toán.
* **Action / Algorithm**:
  1. **Mẫu Chuẩn Hệ thống (`is_system = true`)**:
     - Do Quản trị viên hệ thống cài sẵn cho các phần mềm kế toán thông dụng: MISA SME, MISA AMIS, FAST Accounting, Bravo, Smart Pro, Effect.
     - Người dùng chỉ có quyền sử dụng hoặc sao chép (Clone) thành mẫu mới, không được sửa/xóa trực tiếp mẫu gốc.
  2. **Mẫu Tùy biến Doanh nghiệp (`is_system = false`)**:
     - Lưu theo tài khoản doanh nghiệp (`account = accountId`).
     - Người dùng được phép thêm, sửa, xóa các cột, điều chỉnh công thức `calc()`, thay đổi thứ tự cột cho phù hợp với phần mềm kế toán nội bộ đặc thù.
* **Schema / Fields**: `reportconfigs.{account, user, template_name, is_system, columns}`.

---

### 2.5. REPORT-BR-005: Điều kiện Đưa vào Kỳ Báo cáo & Khóa Số liệu
* **Statement**: Một hóa đơn chỉ được phép tính vào số liệu Bảng kê và Tờ khai thuế GTGT khi thỏa mãn đồng thời các điều kiện nghiệp vụ nghiêm ngặt.
* **Condition**: Khi hệ thống kích hoạt tính toán báo cáo thuế theo kỳ `targetPeriod`.
* **Action / Algorithm**:
  - **Điều kiện cần và đủ để đưa vào kỳ tính thuế**:
    1. Hóa đơn có `data.nkekhai ≠ null` và tháng/quý của `data.nkekhai` trùng với `targetPeriod`.
    2. Cờ trạng thái `isCancelDeclare === false`.
    3. Hóa đơn không bị đánh dấu xóa logic (`is_deleted === false`).
    4. Không tính trùng các hóa đơn đã được thay thế hoặc điều chỉnh giảm về 0.
  - **Cơ chế Khóa kỳ (Period Lock)**:
    - Khi kế toán bấm "Khóa số liệu kỳ kê khai", hệ thống ghi nhận cờ `is_locked = true` cho kỳ thuế đó trong bảng `tokhaithues`.
    - Sau khi đã khóa kỳ, mọi thao tác gán thêm hóa đơn mới vào kỳ này hoặc hủy kê khai hóa đơn cũ đều bị hệ thống chặn, bắt buộc phải mở khóa kỳ trước khi hiệu chỉnh.
* **Schema / Fields**: `tokhaithues.is_locked`, `invoiceraws.data.nkekhai`.

---

### 2.6. REPORT-BR-006: Lưu trữ Snapshot & Xuất XML HTKK (HTKK Integration)
* **Statement**: Hệ thống hỗ trợ chụp ảnh nhanh (Snapshot) toàn bộ số liệu tờ khai tại thời điểm chốt sổ lưu vào bảng `tokhaithues` để bảo toàn tính lịch sử và kết xuất tệp tin XML chuẩn XSD nộp vào phần mềm HTKK của Tổng cục Thuế.
* **Condition**: Kế toán bấm "Lưu tờ khai" hoặc "Xuất file XML nộp cơ quan thuế".
* **Action / Algorithm**:
  1. **Lưu trữ Snapshot (`tokhaithues`)**:
     - Lưu toàn bộ giá trị các chỉ tiêu từ [21] đến [43] kèm danh sách `_id` các hóa đơn cấu thành tại thời điểm lưu.
     - Nếu sau này có hóa đơn bị sửa đổi, số liệu của tờ khai snapshot đã lưu không bị thay đổi, giúp kế toán so sánh phát hiện chênh lệch giữa sổ sách và thực tế.
  2. **Kết xuất Tệp tin XML HTKK**:
     - Tuân thủ nghiêm ngặt cấu trúc XSD của Tổng cục Thuế quy định cho Tờ khai 01/GTGT (Mẫu theo Thông tư 80/2021/TT-BTC).
     - Định dạng thẻ XML: `<HSoKhaiThue>`, `<TTinChung>`, `<CTieuTKhai>`, `<BangKeMuaVao>`, `<BangKeBanRa>`.
     - File XML xuất ra có thể mở trực tiếp và nhập khẩu thành công 100% vào ứng dụng HTKK máy trạm của Tổng cục Thuế mà không phát sinh lỗi cấu trúc.
* **Schema / Fields**: Bảng `tokhaithues.{account, tax_period, snapshot_data, invoice_ids, created_at, created_by}`.
* **Source**: Thông tư 80/2021/TT-BTC, Định dạng thông điệp dữ liệu kê khai thuế điện tử của TCT.
