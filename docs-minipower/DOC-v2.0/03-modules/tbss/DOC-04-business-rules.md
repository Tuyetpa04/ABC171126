# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Quản lý Thông báo Sai sót (TBSS)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TBSS` — Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6.16), `SMI_Sơ đồ nghiệp vụ.json` (Trang 5), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Thông báo 04SS), `mSMI - Tài liệu thiết kế CSDL.docx` (Bảng `hdtbssrses`).

---

## 1. Danh mục Quy tắc Nghiệp vụ

| ID | Tên Quy tắc | Mô tả Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **TBSS-BR-001** | Phân loại Chiều Thông báo 04/SS | Hệ thống chia tách rõ rệt 2 luồng: Thông báo 04/SS Mua vào (bên bán gửi lên CQT giải trình/hủy/điều chỉnh hóa đơn xuất cho doanh nghiệp) và 04/SS Bán ra (doanh nghiệp lập gửi CQT). | Validation | Must | TBSS-UC-001<br>TBSS-FR-001 |
| **TBSS-BR-002** | Bảng Mã Tính chất Sai sót | Căn cứ Nghị định 123/2020/NĐ-CP, tính chất sai sót trong thông báo 04/SS bắt buộc ánh xạ theo 4 giá trị chuẩn: `1` - Hủy, `2` - Điều chỉnh, `3` - Thay thế, `4` - Giải trình. | Validation | Must | TBSS-UC-002<br>TBSS-FR-002 |
| **TBSS-BR-003** | Liên kết Tự động với Hóa đơn gốc | Khi tiếp nhận thông báo 04/SS từ CQT, hệ thống tự động tìm kiếm hóa đơn gốc tương ứng theo cặp `(sellerTaxCode, khmshdon, khhdon, shdon)` để liên kết dữ liệu và cập nhật trạng thái cảnh báo trên hóa đơn. | Integrity | Must | TBSS-UC-005<br>TBSS-FR-005 |
| **TBSS-BR-004** | Toàn vẹn Dữ liệu XML & Bản thể hiện | Dữ liệu thông báo 04/SS phải được lưu trữ nguyên vẹn dưới định dạng XML gốc của CQT; hệ thống hỗ trợ sinh bản thể hiện trực quan (HTML/PDF) mô phỏng đúng mẫu in chuẩn Nghị định 123. | Computation | Must | TBSS-UC-002<br>TBSS-FR-002 |
| **TBSS-BR-005** | Đồng bộ Trạng thái Xử lý từ CQT | Ghi nhận chính xác phản hồi tiếp nhận từ Cơ quan Thuế đối với tờ thông báo 04/SS: `Tiếp nhận`, `Chấp nhận`, `Không chấp nhận`. | Validation | Should | TBSS-UC-001<br>TBSS-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ

### 2.1. TBSS-BR-001: Phân loại Chiều Thông báo 04/SS
- **Thông báo 04/SS Mua vào**: Giúp kế toán doanh nghiệp mua phát hiện kịp thời các trường hợp người bán đã đơn phương gửi thông báo sai sót/hủy hóa đơn lên cơ quan thuế mà không thông báo cho bên mua.
- **Thông báo 04/SS Bán ra**: Quản lý lịch sử các thông báo sai sót do chính doanh nghiệp đã gửi lên Tổng cục Thuế để theo dõi trạng thái phê duyệt của cơ quan thuế quản lý trực tiếp.

### 2.2. TBSS-BR-002: Bảng Mã Tính chất Sai sót
Mỗi dòng hóa đơn sai sót được kê khai trong thông báo 04/SS phải mang một trong các mã tính chất sau:

| Mã tính chất (`tchat`) | Tên tính chất | Ý nghĩa nghiệp vụ |
|:---:|---|---|
| `1` | Hủy | Hóa đơn đã lập có sai sót và bị hủy bỏ hoàn toàn. |
| `2` | Điều chỉnh | Hóa đơn có sai sót về tiền, thuế, số lượng cần lập hóa đơn điều chỉnh. |
| `3` | Thay thế | Hóa đơn bị hủy và được thay thế bằng một hóa đơn mới hoàn toàn. |
| `4` | Giải trình | Hóa đơn chỉ sai sót về tên, địa chỉ người mua nhưng không sai mã số thuế và các nội dung khác; người bán giải trình với CQT mà không cần lập hóa đơn điều chỉnh/thay thế. |

### 2.3. TBSS-BR-003: Liên kết Tự động với Hóa đơn gốc
- Khóa đối soát liên kết:
  $$\text{InvoiceRef} = \{ \text{MST Bán}, \text{Mẫu số HĐ}, \text{Ký hiệu HĐ}, \text{Số HĐ} \}$$
- Khi thông báo 04/SS được đồng bộ về:
  1. Duyệt từng dòng hóa đơn con trong mảng `dshdon` của thông báo 04/SS.
  2. Truy vấn vào bảng `invoiceraws`.
  3. Nếu tìm thấy hóa đơn gốc:
     - Gắn trường liên kết `tbssRefId = thông báo 04/SS._id`.
     - Cập nhật trạng thái xử lý của hóa đơn tương ứng với tính chất sai sót (Hóa đơn bị hủy / Hóa đơn có thông báo điều chỉnh / Hóa đơn có giải trình).
     - Kích hoạt cờ cảnh báo rủi ro trên hóa đơn để kế toán chú ý khi xem danh sách.

### 2.4. TBSS-BR-004: Toàn vẹn Dữ liệu XML & Bản thể hiện Mẫu 04/SS-HĐĐT
- **Statement**: Mọi tờ thông báo sai sót thu thập từ CQT hoặc do doanh nghiệp gửi đi bắt buộc phải lưu trữ toàn vẹn tệp XML gốc ký số của CQT/Doanh nghiệp; hệ thống cung cấp giao diện trực quan hóa bản thể hiện chuẩn theo Mẫu 04/SS-HĐĐT quy định tại Nghị định 123/2020/NĐ-CP.
- **Action / Algorithm**:
  1. Dữ liệu XML gốc được bóc tách và lưu vào trường `xml_data` hoặc lưu trữ trên MinIO/Object Storage.
  2. Hệ thống bóc tách các trường: Số thông báo của CQT (`so_tbss`), Ngày thông báo (`ngay_tbss`), Cơ quan thuế quản lý trực tiếp (`ten_cqt`), và mảng các dòng hóa đơn sai sót `dshdon`.
  3. Render bản thể hiện HTML/PDF: Tự động điền dữ liệu vào phôi in biểu mẫu Mẫu số 04/SS-HĐĐT, hiển thị đầy đủ tiêu đề, bảng kê danh sách hóa đơn sai sót, tính chất sai sót, lý do sai sót và chữ ký số điện tử của người nộp thuế/Cơ quan thuế.
- **Schema / Fields**: `hdtbssrses.{xml_data, html_content, dshdon, so_tbss, ngay_tbss}`.

### 2.5. TBSS-BR-005: Đồng bộ Trạng thái Xử lý từ CQT (CQT Approval State)
- **Statement**: Hệ thống phải theo dõi và cập nhật liên tục trạng thái phản hồi tiếp nhận từ Cổng Dịch vụ Hóa đơn điện tử của Cơ quan Thuế đối với từng tờ thông báo 04/SS do doanh nghiệp gửi đi.
- **Action / Algorithm**:
  - Ánh xạ mã trạng thái xử lý phản hồi từ CQT:
    * `1` - **Đã tiếp nhận**: CQT đã nhận được tệp thông báo điện tử và đang kiểm tra tính hợp lệ.
    * `2` - **Đã chấp nhận**: CQT chấp nhận nội dung thông báo sai sót; thông báo có giá trị pháp lý chính thức.
    * `3` - **Không chấp nhận**: CQT từ chối thông báo kèm thông báo lý do (sai cấu trúc, chữ ký số không hợp lệ, thông tin hóa đơn không khớp).
  - Tự động cập nhật trạng thái vào bảng `hdtbssrses` và gửi thông báo qua giao diện/Email cho kế toán phụ trách để kịp thời xử lý nếu bị từ chối.
- **Schema / Fields**: `hdtbssrses.{status, cqt_response_code, cqt_response_message, updated_at}`.


