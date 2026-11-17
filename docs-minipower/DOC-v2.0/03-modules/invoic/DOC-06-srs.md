# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Quản lý Hóa đơn (INVOIC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** ISO/IEC/IEEE 29148:2018 / IEEE 830  
> **Phân hệ:** `INVOIC` — Quản lý Hóa đơn Mua vào / Bán ra  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx`, `[mSMI] Document API.docx`, `App hóa đơn vào_ ra SMI.xlsx`.  
> **Căn cứ mã nguồn thực tế:** `backend/routes/invoices/index.js`, `backend/models/invoiceraw.js`, `backend/utils/invoice.js`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| FR ID | Tên Yêu cầu | Mô tả Chi tiết | Mức ưu tiên | Source (UC / BR) |
|:---:|---|---|:---:|:---:|
| **INVOIC-FR-001** | Tra cứu & Bộ lọc Đa tiêu chí | Hệ thống **shall** cung cấp bộ lọc danh sách hóa đơn theo: Khoảng ngày lập, ngày ký, kỳ kê khai, người bán/người mua, trạng thái rủi ro, nhãn gán, loại hóa đơn (thường/máy tính tiền). | Must | INVOIC-UC-001<br>INVOIC-BR-001, 002 |
| **INVOIC-FR-002** | Xem Bản thể hiện & Cấu trúc XML | Hệ thống **shall** hiển thị trực quan bản thể hiện HTML/PDF của hóa đơn (đầy đủ dòng hàng, thuế suất, tổng tiền) và cho phép xem trực tiếp cấu trúc XML gốc. | Must | INVOIC-UC-002, 003<br>INVOIC-BR-001 |
| **INVOIC-FR-003** | Tải Hóa đơn Đơn lẻ & Hàng loạt ZIP | Hệ thống **shall** cho phép tải file XML gốc, file PDF riêng lẻ và đóng gói hàng loạt tối đa 100 hóa đơn vào tệp nén ZIP tải về thông qua cơ chế fallback 5 tầng. | Must | INVOIC-UC-004<br>INVOIC-BR-004, 007 |
| **INVOIC-FR-004** | Gán Nhãn Phân loại Chi phí (Tags) | Hệ thống **shall** cho phép người dùng gán một hoặc nhiều nhãn màu cho hóa đơn để phân loại chi phí nội bộ và hỗ trợ lọc nhanh theo nhãn. | Should | INVOIC-UC-005<br>INVOIC-BR-008 |
| **INVOIC-FR-005** | Quản lý Ngày Kê khai Thuế | Hệ thống **shall** cho phép cập nhật trường `nkekhai` cho một hoặc hàng loạt hóa đơn; hỗ trợ thao tác Hủy kê khai để đưa `nkekhai` về null. | Must | INVOIC-UC-006<br>INVOIC-BR-003 |
| **INVOIC-FR-006** | Đính kèm Hồ sơ Chứng từ & Ẩn Cảnh báo | Hệ thống **shall** cho phép upload tài liệu đính kèm (hợp đồng, biên bản) mã hóa an toàn và cho phép ẩn cờ cảnh báo rủi ro có bảo lưu vết trạng thái gốc. | Could | INVOIC-UC-007<br>INVOIC-BR-005, 006 |
| **INVOIC-FR-007** | Ghi chú Nội bộ trên Hóa đơn | Hệ thống **shall** cho phép kế toán ghi chú diễn giải nội bộ trên từng hóa đơn để các nhân viên kế toán trao đổi thông tin hạch toán. | Could | INVOIC-UC-008<br>INVOIC-BR-001 |

---

## 2. Đặc tả Chi tiết Yêu cầu Chức năng

### 2.1. INVOIC-FR-001: Tra cứu & Bộ lọc Đa tiêu chí (Search & Filtering)
* **Mục tiêu:** Cung cấp thanh công cụ tìm kiếm và lọc dữ liệu hóa đơn linh hoạt với hiệu năng cao trên tập dữ liệu hàng chục nghìn hóa đơn.
* **Đầu vào (Inputs):**
  - Tham số phân luồng: `account` (ID tài khoản thuế), `type` (`purchase`, `sold`, `purchase_sco`, `sold_sco`).
  - Tiêu chí lọc nâng cao:
    * Khoảng ngày lập hóa đơn (`tdlap` từ ngày - đến ngày).
    * Khoảng ngày ký số (`nky`).
    * Trạng thái kê khai thuế: Đã kê khai / Chưa kê khai / Theo kỳ kê khai cụ thể.
    * Mã số thuế hoặc Tên đối tác (Người bán `nbmst`/`nbten` đối với mua vào; Người mua `nmmst`/`nmten` đối với bán ra).
    * Mức độ rủi ro thuế: Hóa đơn hợp lệ / Có rủi ro / Doanh nghiệp bỏ trốn / Thuộc danh mục 1.500 DN.
    * Nhãn phân loại (`tags` mảng ObjectIds).
* **Xử lý (Processing):**
  1. API `GET /invoices/account/:account/type/:type` bóc tách tham số `lazyLoadEvent` (phân trang PrimeNG `first`, `rows`, `sortField`, `sortOrder`).
  2. Xây dựng MongoDB Query bằng `QueryBuilder`:
     - Kiểm tra quyền: `query.user = req.user._id` (hoặc `req.user.parent` đối với `employee`).
     - Tự động bổ sung index hints và điều kiện `$in` cho trường `tags`.
  3. Thực hiện truy vấn song song `countDocuments` và `find` có `.skip()` và `.limit()`.
* **Đầu ra (Outputs):** Đối tượng JSON `{ totalRecords: number, items: InvoiceRaw[] }`.

---

### 2.2. INVOIC-FR-002: Xem Bản thể hiện & Cấu trúc XML (Viewer)
* **Mục tiêu:** Cho phép kế toán xem trực quan bản in thể hiện hóa đơn đầy đủ thông tin pháp lý và soi cấu trúc XML gốc để kiểm tra tính toàn vẹn.
* **Xử lý:**
  1. **Xem Bản thể hiện HTML (`GET /invoices/:id/invoice-info-html`):**
     - Đọc dữ liệu chi tiết dòng hàng từ trường `detail` trong `invoiceraws`.
     - Nếu trường `assets.detail.done === false`, tự động kích hoạt `PATCH /:id/priority` đẩy ưu tiên cào detail lên `10` (`INVOIC-BR-007`).
     - Render HTML thông qua engine template EJS, mô phỏng đúng mẫu hóa đơn chuẩn: Quốc hiệu, tiêu ngữ, tên hóa đơn, mẫu số, ký hiệu, số, bảng danh mục hàng hóa, thuế suất, tổng tiền thanh toán bằng số/chữ, chữ ký số.
  2. **Xem XML Gốc:**
     - Truy xuất nội dung XML từ trường `xml` hoặc MinIO S3.
     - Hiển thị trên popup có tô màu cú pháp (Syntax Highlighter), hỗ trợ tìm kiếm thẻ và kiểm tra các khối chữ ký số `<Signature>`.
* **Đầu ra:** Giao diện xem trước mượt mà, hỗ trợ phóng to/thu nhỏ và lệnh in trực tiếp từ trình duyệt.

---

### 2.3. INVOIC-FR-003: Tải Hóa đơn Đơn lẻ & Hàng loạt dạng ZIP (Batch Download)
* **Mục tiêu:** Cung cấp tính năng tải tệp tin hóa đơn (XML, PDF, HTML) đơn lẻ hoặc đóng gói số lượng lớn phục vụ giải trình thuế và lưu trữ chứng từ.
* **Đầu vào:** Danh sách các `_id` hóa đơn được chọn qua checkbox (`ids=id1,id2,...`), tham số loại tệp `type = pdf | xml | html | zip`.
* **Xử lý:**
  1. Giới hạn tối đa **100 hóa đơn** trên mỗi lượt tải hàng loạt (`limit = 100`).
  2. Áp dụng cơ chế **Fallback 5 tầng truy xuất file PDF** (`createPdfDownloadRequest` theo `INVOIC-BR-004`):
     - Tầng 1: PDF gốc MinIO TCGP (`pdf_tcgp`).
     - Tầng 2: PDF gốc từ email bóc tách (`tryReadGmailPdfBuffer`).
     - Tầng 3: PDF MinIO Email (`pdf`).
     - Tầng 4: Attachment mã hóa AES trong CSDL (`.atm`).
     - Tầng 5: Render tức thời bằng Headless Chrome/Puppeteer từ dữ liệu HTML.
  3. Nén các tệp tin vào thư mục `hoa-don/` trong file ZIP bằng thư viện `JSZip`, đặt tên file chuẩn `{nbmst}_{nmmst}_{khmshdon}{khhdon}_{shdon}.ext`.
  4. Nếu có hóa đơn bị thiếu file do CQT chưa đồng bộ xong chi tiết, hệ thống vẫn nén các hóa đơn thành công và sinh file log `BaoCaoLoi_MissingAssets.txt` đặt tại thư mục gốc của file ZIP.
* **Đầu ra:** Stream file ZIP tải về máy tính người dùng (`Content-Type: application/zip`).

---

### 2.4. INVOIC-FR-004: Gán Nhãn Phân loại Chi phí (Tags)
* **Mục tiêu:** Cho phép kế toán gắn nhãn đánh dấu phân loại chi phí nội bộ cho từng hóa đơn hoặc nhiều hóa đơn cùng lúc.
* **Đầu vào:** Danh sách hóa đơn được chọn, danh sách các nhãn phân loại được áp dụng.
* **Xử lý:**
  1. Cập nhật danh sách nhãn phân loại cho hóa đơn được chỉ định.
  2. Hỗ trợ gán nhãn hàng loạt: Chọn nhiều hóa đơn trên bảng và chọn nhãn từ menu tiện ích; hệ thống tự động bổ sung thêm nhãn mới mà vẫn bảo toàn các nhãn hiện có của hóa đơn.
  3. Khi một nhãn bị xóa khỏi danh mục hệ thống, hệ thống tự động gỡ bỏ liên kết của nhãn đó khỏi toàn bộ các hóa đơn đã được gán trước đây (`INVOIC-BR-008`).
* **Đầu ra:** Huy hiệu nhãn màu hiển thị trực quan ngay trên từng dòng hóa đơn trong danh sách.

---

### 2.5. INVOIC-FR-005: Quản lý Ngày Kê khai Thuế & Khóa Kỳ (`nkekhai`)
* **Mục tiêu:** Quản lý vòng đời kê khai thuế của hóa đơn, xác định hóa đơn nào đã được hạch toán vào Tờ khai Mẫu 01/GTGT.
* **Đầu vào:** Mã hóa đơn, mốc ngày kê khai `inputDate` (hoặc cờ yêu cầu Hủy kê khai).
* **Xử lý:**
  1. **Thao tác Gán kê khai (`PATCH /invoices/:id/nkekhai`):**
     - Lưu `data.nkekhai = inputDate`.
     - Tự động sinh nhãn `data.nkekhai_label = "Tháng MM/YYYY"` hoặc `"Quý Q/YYYY"`.
     - Đổi trạng thái hiển thị: "Đã kê khai".
     - Hóa đơn được đưa vào công thức tính toán chỉ tiêu [22], [23], [25] (đối với mua vào) hoặc [26] đến [32a] (đối với bán ra) của phân hệ `REPORT`.
  2. **Thao tác Hủy kê khai:**
     - Đặt `data.nkekhai = null`, loại bỏ hóa đơn khỏi Bảng kê và Tờ khai thuế tương ứng.
* **Đầu ra:** Bản ghi hóa đơn được cập nhật và giao diện phản hồi trạng thái mới tức thì.

---

### 2.6. INVOIC-FR-006: Đính kèm Hồ sơ Chứng từ & Quản lý Ẩn Cảnh báo
* **Mục tiêu:** Quản lý các tài liệu gốc đi kèm hóa đơn và quản lý quyền ẩn cảnh báo rủi ro có bảo lưu vết.
* **Xử lý:**
  1. **Tệp đính kèm (`attachments`):**
     - Kiểm tra định dạng (PDF, hình ảnh, Office) và dung lượng ($\le 10\text{ MB}$).
     - Tự động mã hóa AES-256-CBC thành tệp `.atm` và lưu vết vào bảng `attachments` (`INVOIC-BR-006`).
     - Cho phép xem trước hoặc giải mã tải về khi người dùng có thẩm quyền bấm mở.
  2. **Ẩn Cảnh báo (`PATCH /invoices/:id/hide_status`):**
     - Khi người dùng bật "Ẩn cảnh báo": Ghi đè `warning.status = "Hợp lệ"`, lưu trạng thái rủi ro ban đầu vào `warning.origin_status`, đặt cờ `warning.hide_status = true` (`INVOIC-BR-005`).
     - Khi người dùng bỏ ẩn: Khôi phục lại trạng thái từ `warning.origin_status`.
* **Đầu ra:** Thông báo thao tác thành công và cập nhật lại giao diện danh sách hóa đơn.

---

### 2.7. INVOIC-FR-007: Ghi chú Nội bộ trên Hóa đơn (Internal Comments)
* **Mục tiêu:** Tạo kênh trao đổi thông tin hạch toán nội bộ giữa các kế toán viên phụ trách.
* **Đầu vào:** `comment` (chuỗi văn bản ghi chú, tối đa 500 ký tự).
* **Xử lý:** API `PATCH /invoices/:id/comment` ghi đè trường `comment` của hóa đơn thuộc phạm vi tài khoản quản trị.
* **Đầu ra:** Biểu tượng ghi chú đổi màu trên danh sách; tooltip hiển thị nội dung ghi chú khi rê chuột qua.
