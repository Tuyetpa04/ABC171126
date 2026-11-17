# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Kiểm soát Rủi ro Thuế (RISK)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `RISK` — Kiểm soát & Cảnh báo Rủi ro Thuế  
> **Căn cứ tài liệu gốc:** `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 6.13, 6.14), `mSMI - Tài liệu thiết kế CSDL.docx` (`taxcodeclassificationappls`, `taxcodeinfos`, `invoiceraws`), `[mSMI] Document API.docx`.  
> **Căn cứ mã nguồn thực tế:** `backend/service_risk/`, `backend/routes/invoices/`, `backend/models/taxcodeclassificationappl.js`.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| ID | Tên Quy tắc | Mô tả Tóm tắt Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **RISK-BR-001** | Trạng thái Pháp lý Người bán | Tự động tra cứu MST người bán trên cơ sở dữ liệu quốc gia về người nộp thuế; cảnh báo nếu trạng thái là: Ngừng hoạt động chưa đóng MST, Tạm ngừng kinh doanh, Không hoạt động tại địa chỉ đăng ký (bỏ trốn). | Inference / State | Must | RISK-UC-001<br>RISK-FR-001 |
| **RISK-BR-002** | Rà soát Blacklist 1.500 DN Rủi ro cao | Đối chiếu `nbmst` với bảng danh mục 1.500 doanh nghiệp rủi ro cao về hóa đơn của Tổng cục Thuế. Nếu khớp, phát cờ đỏ cảnh báo mức độ Nguy hiểm và ngăn chặn khấu trừ. | Validation / Match | Must | RISK-UC-001<br>RISK-FR-002 |
| **RISK-BR-003** | Hóa đơn bị Hủy / Thay thế / Điều chỉnh | Cảnh báo ngay lập tức nếu hóa đơn mua vào có trạng thái CQT bị Hủy (`tthai = 4`), Thay thế (`tthai = 3`), hoặc Điều chỉnh (`tthai = 2`), tự động tạo chuỗi liên kết đối soát. | Event / Consistency | Must | RISK-UC-001<br>RISK-FR-003 |
| **RISK-BR-004** | Bất thường Mốc thời gian Lập hóa đơn | Cảnh báo vi phạm nghiêm trọng nếu ngày lập hóa đơn (`tdlap`) rơi vào khoảng thời gian người bán đang tạm ngừng hoạt động hoặc trước ngày thành lập đăng ký kinh doanh. | Temporal / Audit | Must | RISK-UC-001<br>RISK-FR-001 |
| **RISK-BR-005** | Tính Toàn vẹn XML & Chữ ký số | Cảnh báo nếu dữ liệu XML bị sửa đổi làm sai lệch hash toàn vẹn, chứng thư số của người bán hết hạn hoặc ngày ký số trễ hơn so với quy định pháp luật. | Security / Crypto | Must | RISK-UC-001<br>RISK-FR-004 |
| **RISK-BR-006** | Soi lệch Tên Doanh nghiệp & Địa chỉ | So khớp tự động tên người bán/người mua trên hóa đơn với dữ liệu gốc của CQT; phát cảnh báo nếu lệch chính tả, ký tự hoa/thường, hoặc sai khác địa chỉ kinh doanh. | Text Match / AI | Should | RISK-UC-001<br>RISK-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ (Detailed Business Rules Specification)

### 2.1. RISK-BR-001: Trạng thái Pháp lý Người bán (Seller Legal Status)
* **Statement**: Bất kỳ hóa đơn mua vào nào xuất phát từ nhà cung cấp có trạng thái pháp lý bất thường trên cơ sở dữ liệu quốc gia về người nộp thuế đều phải được tự động phát hiện và cảnh báo tức thời.
* **Condition**: Kích hoạt khi hóa đơn mua vào được đồng bộ về hoặc khi chạy tác vụ quét rủi ro định kỳ theo bảng `taxcodeinfos`.
* **Action / Algorithm**:
  1. Lấy mã số thuế người bán: `sellerMST = invoice.header.nbmst`.
  2. Truy vấn vào bảng `taxcodeinfos` theo `mst = sellerMST`.
  3. Đánh giá trạng thái pháp lý (`status_code`):
     - `00` - Đang hoạt động bình thường: Trạng thái Hợp lệ.
     - `01` - Ngừng hoạt động nhưng chưa hoàn thành thủ tục đóng mã số thuế: **Cảnh báo Vàng** (Nguy cơ rủi ro cao).
     - `02` - Tạm ngừng kinh doanh có thời hạn: **Cảnh báo Cam**.
     - `03` - Người nộp thuế không hoạt động tại địa chỉ đã đăng ký (Doanh nghiệp bỏ trốn): **Cảnh báo Đỏ** (Mức độ Nguy hiểm tối đa).
     - `05` - Đã chấm dứt hiệu lực mã số thuế (Đã đóng MST): **Cảnh báo Đỏ**.
  4. Nếu phát hiện vi phạm:
     - Gán giá trị vào `warning.status = "Doanh nghiệp không hoạt động tại địa chỉ đăng ký"` (hoặc thông điệp tương ứng).
     - Đặt cờ `warning.is_risk = true`.
     - Lưu thông tin lịch sử thay đổi trạng thái vào mảng `warning.risk_details`.
* **Schema / Fields**: `taxcodeinfos.{mst, name, status_code, status_name, closed_date, open_date}`, `invoiceraws.warning`.
* **Source**: `mSMI - Tài liệu HDSD...docx` (Mục 6.13), Thông tư 105/2020/TT-BTC.

---

### 2.2. RISK-BR-002: Rà soát Blacklist 1.500 Doanh nghiệp Rủi ro cao
* **Statement**: Bất kỳ hóa đơn mua vào nào có người bán nằm trong Danh mục doanh nghiệp rủi ro cao về hóa đơn (danh sách đen 1.500 DN do Tổng cục Thuế công bố) phải bị gắn cờ cảnh báo đỏ nghiêm trọng để ngăn chặn nguy cơ khấu trừ khống và bị truy thu thuế.
* **Condition**: Khi phát sinh hóa đơn mua vào mới hoặc khi hệ thống cập nhật phiên bản Blacklist mới từ Cơ quan Thuế.
* **Action / Algorithm**:
  1. Đối soát Mã số thuế bên bán (`nbmst`) với Danh mục doanh nghiệp rủi ro cao đang có hiệu lực trong hệ thống (danh mục 1.500 doanh nghiệp do Tổng cục Thuế ban hành).
  2. Nếu phát hiện bên bán thuộc danh mục rủi ro cao:
     - Gán nhãn cảnh báo mức độ cao nhất: **Màu Đỏ (Cực kỳ nguy hiểm)**.
     - Cập nhật thông điệp cảnh báo: *"Doanh nghiệp thuộc danh mục rủi ro cao 1.500 DN của Tổng cục Thuế"*.
     - Lưu chi tiết căn cứ pháp lý cảnh báo: Số công văn của CQT (ví dụ: Công văn số 1798/TCT-TTKT), ngày ban hành và cơ quan thuế ra văn bản.
     - Khóa tự động: Không cho phép tự động phân bổ hóa đơn này vào Tờ khai 01/GTGT; yêu cầu kế toán trưởng phải thẩm định hồ sơ giải trình thực tế trước khi gán kê khai.
* **Schema / Fields**: `taxcodeclassificationappls.{mst, name, dispatch_no, dispatch_date, reason}`, `invoiceraws.warning`.
* **Source**: Công văn số 1798/TCT-TTKT của Tổng cục Thuế về rà soát hóa đơn rủi ro cao, `backend/models/taxcodeclassificationappl.js`.

---

### 2.3. RISK-BR-003: Cảnh báo Hóa đơn bị Hủy hoặc Điều chỉnh / Thay thế
* **Statement**: Hệ thống phải thông báo tức thì cho kế toán mua vào khi hóa đơn đã nhận bị bên bán đơn phương hủy bỏ hoặc lập hóa đơn điều chỉnh/thay thế trên Cổng Thuế mà bên mua chưa nắm được thông tin.
* **Condition**: Khi phiên đồng bộ dữ liệu phát hiện thuộc tính `header.tthai` của hóa đơn bị thay đổi giá trị từ Cơ quan Thuế.
* **Action / Algorithm**:
  1. Kiểm tra mã trạng thái CQT (`header.tthai`):
     - `tthai === 1`: Hóa đơn Mới lập (Bình thường).
     - `tthai === 2`: Hóa đơn Bị điều chỉnh.
     - `tthai === 3`: Hóa đơn Bị thay thế.
     - `tthai === 4`: Hóa đơn Bị hủy.
     - `tthai === 5`: Hóa đơn Bị giải trình sai sót.
  2. Nếu `tthai === 4` (Bị hủy):
     - Bật cờ cảnh báo: `warning.status = "Hóa đơn đã bị hủy trên Cổng Tổng cục Thuế"`.
     - Nếu hóa đơn này đã được gán `nkekhai` trong kỳ tính thuế trước, hệ thống phát cảnh báo `CRITICAL`: Yêu cầu kế toán lập tờ khai điều chỉnh/bổ sung giảm thuế đầu vào.
  3. Nếu `tthai === 2` hoặc `tthai === 3`:
     - Tự động tìm kiếm hóa đơn điều chỉnh/thay thế tương ứng thông qua số tham chiếu hoặc thông báo sai sót (`TBSS`).
     - Gắn liên kết chuỗi vết (Traceability Chain) giữa hóa đơn gốc và hóa đơn mới để kế toán dễ dàng đối soát song song.
* **Schema / Fields**: `invoiceraws.header.tthai`, `invoiceraws.related_invoices`.
* **Source**: Nghị định 123/2020/NĐ-CP (Điều 19 về xử lý hóa đơn sai sót).

---

### 2.4. RISK-BR-004: Bất thường Mốc thời gian Lập hóa đơn (Timeline Anomaly)
* **Statement**: Cảnh báo vi phạm nghiêm trọng tính pháp lý nếu ngày lập hóa đơn (`tdlap`) rơi vào khoảng thời gian người bán không được phép hoạt động kinh doanh (trước ngày cấp phép thành lập hoặc trong thời gian đang tạm ngừng kinh doanh).
* **Condition**: Kích hoạt khi bóc tách thông tin ngày lập hóa đơn từ tệp XML và đối soát với thông tin thời gian biểu của người nộp thuế.
* **Action / Algorithm**:
  1. Bóc tách ngày lập hóa đơn: `invoiceDate = new Date(invoice.header.tdlap)`.
  2. Lấy thông tin thời gian đăng ký doanh nghiệp của bên bán từ `taxcodeinfos`:
     - `establishedDate`: Ngày thành lập / cấp mã số thuế lần đầu.
     - `suspendFrom`: Ngày bắt đầu tạm ngừng kinh doanh.
     - `suspendTo`: Ngày kết thúc tạm ngừng kinh doanh.
  3. Quy tắc kiểm tra logic mốc thời gian:
     - **Vi phạm 1 (Trước thành lập)**: Nếu `invoiceDate < establishedDate` $\implies$ Cảnh báo Đỏ: "Hóa đơn lập trước ngày doanh nghiệp người bán được cấp phép thành lập".
     - **Vi phạm 2 (Trong thời gian tạm ngừng)**: Nếu `suspendFrom <= invoiceDate <= suspendTo` $\implies$ Cảnh báo Đỏ: "Hóa đơn lập trong thời gian doanh nghiệp người bán đang đăng ký tạm ngừng kinh doanh".
     - **Vi phạm 3 (Sau ngày đóng MST)**: Nếu `closedDate && invoiceDate > closedDate` $\implies$ Cảnh báo Đỏ: "Hóa đơn lập sau ngày người bán đã hoàn tất thủ tục đóng mã số thuế".
* **Schema / Fields**: `invoiceraws.header.tdlap`, `taxcodeinfos.{open_date, suspend_from, suspend_to, closed_date}`.
* **Source**: Luật Doanh nghiệp 2020, Nghị định 01/2021/NĐ-CP về đăng ký doanh nghiệp.

---

### 2.5. RISK-BR-005: Tính Toàn vẹn XML & Chữ ký số (Digital Signature & Hash Integrity)
* **Statement**: Dữ liệu tệp XML hóa đơn điện tử phải đảm bảo tính toàn vẹn tuyệt đối (chống sửa đổi byte) và chữ ký số của đơn vị phát hành phải hợp lệ tại thời điểm ký theo Luật Giao dịch Điện tử.
* **Condition**: Khi người dùng tải file XML lên hoặc khi nhận file XML từ kênh Email / Cổng Thuế.
* **Action / Algorithm**:
  1. **Kiểm tra Cấu trúc XML (XSD Validation)**: Xác thực cây thẻ XML theo đúng chuẩn Quyết định 1450/QĐ-TCT. Nếu thẻ bị thiếu hoặc sai định dạng cú pháp $\implies$ Gán cờ `ERR_INVALID_XML_SCHEMA`.
  2. **Kiểm tra Tính Toàn vẹn Băm (Digest Value Verification)**:
     - Tính toán mã băm SHA-256 trên khối dữ liệu nội dung hóa đơn (`<DLHDon>`).
     - So sánh với giá trị trong thẻ `<DigestValue>`. Nếu không trùng khớp $\implies$ Cảnh báo vi phạm nghiêm trọng: "Dữ liệu hóa đơn đã bị can thiệp, chỉnh sửa trái phép sau khi ký số".
  3. **Kiểm tra Chứng thư số (Digital Certificate Validation)**:
     - Trích xuất thời hạn hiệu lực của chứng thư số: `validFrom` và `validTo`.
     - Lấy thời điểm ký số: `signingTime` từ thẻ `<SigningTime>`.
     - Kiểm tra điều kiện hiệu lực:
       $$\text{validFrom} \le \text{signingTime} \le \text{validTo}$$
     - Nếu `signingTime > validTo` hoặc `signingTime < validFrom`: Cảnh báo "Chữ ký số đã hết hiệu lực tại thời điểm ký".
  4. **Kiểm tra Độ trễ Ký số**: Nếu khoảng cách giữa ngày ký và ngày lập vượt quá hạn mức quy định (ví dụ ký trễ quá nhiều ngày), hệ thống phát cảnh báo rủi ro về thời điểm xuất hóa đơn theo Thông tư 78.
* **Schema / Fields**: `invoiceraws.warning.xml_integrity`, `invoiceraws.warning.signature_valid`, `invoiceraws.header.nky`, `invoiceraws.header.tdlap`.
* **Source**: Nghị định 123/2020/NĐ-CP (Điều 10), Luật Giao dịch điện tử số 20/2023/QH15.

---

### 2.6. RISK-BR-006: Soi lệch Tên Doanh nghiệp & Địa chỉ (Entity Match Discrepancy)
* **Statement**: Hệ thống tự động so khớp chuỗi ký tự tên và địa chỉ người nộp thuế ghi trên hóa đơn với cơ sở dữ liệu đăng ký thuế chính thức để phát hiện các trường hợp sai lệch chính tả, mạo danh hoặc hóa đơn xuất sai đơn vị mua.
* **Condition**: Kích hoạt khi đồng bộ hóa đơn hoặc khi người dùng tra cứu chi tiết hóa đơn.
* **Action / Algorithm**:
  1. Chuẩn hóa chuỗi văn bản (Text Normalization):
     - Loại bỏ khoảng trắng thừa (trim & collapse spaces).
     - Đưa về dạng chữ thường Unicode chuẩn NFC (hoặc NFD).
     - Chuẩn hóa các từ viết tắt phổ biến: "TNHH" $\leftrightarrow$ "Trách Nhiệm Hữu Hạn", "CP" $\leftrightarrow$ "Cổ Phần", "MTV" $\leftrightarrow$ "Một Thành Viên".
  2. Tính toán độ tương đồng chuỗi (Levenshtein Distance / Jaro-Winkler):
     - So khớp tên người bán trên HĐ (`header.nbten`) với tên pháp nhân trong CSDL Thuế (`taxcodeinfos.name`).
     - So khớp tên người mua trên HĐ (`header.nmten`) với tên đăng ký của tài khoản doanh nghiệp (`account.company_name`).
  3. Đánh giá mức độ sai lệch:
     - Độ tương đồng $\ge 95\%$: Chấp nhận hợp lệ (sai lệch dấu chấm, phẩy không đáng kể).
     - Độ tương đồng từ $70\%$ đến dưới $95\%$: **Cảnh báo Vàng** — "Tên doanh nghiệp trên hóa đơn có sự khác biệt nhỏ so với tên đăng ký thuế, cần kiểm tra lại".
     - Độ tương đồng $< 70\%$: **Cảnh báo Đỏ** — "Sai lệch nghiêm trọng tên pháp nhân, nghi vấn hóa đơn ghi sai tên đơn vị thụ hưởng".
* **Schema / Fields**: `invoiceraws.header.nbten`, `invoiceraws.header.nmten`, `taxcodeinfos.name`, `invoiceraws.warning.name_mismatch`.
* **Source**: `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `backend/service_risk/index.js`.
