# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn (KIEMTRAHD)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `KIEMTRAHD` — Cổng Kiểm tra & Tra cứu Hóa đơn Điện tử Công khai  
> **Căn cứ mã nguồn thực tế:** `kiemtrahoadon/`, `documents/c4/03-components/kiemtra-hd-portal.md`, `service/proxy-service/`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| Mã FR | Tên Chức năng | Mô tả Chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **KTHD-FR-001** | Tra cứu Trạng thái HĐ Công khai qua CQT | Tiếp nhận thông tin HĐ và Captcha SVG, gọi proxy sang Cổng Thuế để lấy trạng thái pháp lý, trạng thái cấp mã, thông báo sai sót và HĐ liên quan. | Must | KTHD-UC-001<br>KTHD-BR-001, 002 |
| **KTHD-FR-002** | Thẩm định Tính Toàn vẹn File XML HĐĐT | Tiếp nhận tệp XML tải lên, bóc tách cấu trúc XSD, phân tích chữ ký số người bán, thời hạn chứng thư số, mã CQT và cảnh báo rủi ro người bán. | Must | KTHD-UC-002<br>KTHD-BR-003 |
| **KTHD-FR-003** | Trực quan hóa Bản thể hiện & In ấn Công khai | Render bản in HTML chuẩn từ XML trong giao diện chia đôi (Split view) kèm bảng thẩm định pháp lý; hỗ trợ in hóa đơn và in kết quả kiểm tra. | Must | KTHD-UC-003<br>KTHD-BR-004 |
| **KTHD-FR-004** | Đồng bộ Trạng thái CQT cho Hóa đơn Nội bộ | Cung cấp giao diện tra cứu lại trạng thái CQT từ màn hình chi tiết HĐ nội bộ, tự động ghi đè trạng thái mới nhất khi phát hiện sai lệch. | Should | KTHD-UC-004<br>KTHD-BR-005 |
| **KTHD-FR-005** | Tra cứu Định danh Doanh nghiệp Nguồn CQT | Tự động truy vấn thông tin người bán theo MST qua dịch vụ định danh thuế, trả về tên công ty chính thức, địa chỉ và trạng thái hoạt động. | Must | KTHD-UC-001..002<br>KTHD-BR-003 |

---

## 2. Đặc tả Chi tiết Dữ liệu & Giao diện

### 2.1. KTHD-FR-001: Biểu mẫu Tra cứu Hóa đơn CQT Công khai
- **Trường nhập liệu đầu vào:**
  1. `mst` (Mã số thuế bên bán): Bắt buộc, chuỗi ký tự số 10 hoặc 13 số.
  2. `type` (Loại hóa đơn): Bắt buộc, dropdown chọn 1 trong 8 mẫu biểu chuẩn (Mẫu 1: GTGT, Mẫu 2: Bán hàng, Mẫu 3: Bán tài sản công, Mẫu 4: Dự trữ quốc gia, Mẫu 5: Khác, Mẫu 6: Vé/thẻ, 06_01: PXK kiêm vận chuyển nội bộ, 06_02: PXK gửi bán đại lý).
  3. `khhdon` (Ký hiệu hóa đơn): Bắt buộc, tối đa 6 ký tự viết hoa. Tự động cắt ký tự đầu nếu người dùng nhập 7 ký tự (ví dụ: `1C24TUU` → `C24TUU`).
  4. `shdon` (Số hóa đơn): Bắt buộc, chuỗi từ 1 đến 8 chữ số.
  5. `tgtttbso` (Tổng tiền thanh toán): Bắt buộc nếu loại HĐ ≠ 6; số tiền tối đa 14 chữ số phần nguyên và 7 chữ số thập phân.
  6. `tdlap` (Ngày lập hóa đơn): Bắt buộc nếu loại HĐ = 6; bộ chọn ngày (Calendar) ISO 8601.
  7. `captcha` (Mã kiểm tra): Bắt buộc, chuỗi ký tự hiển thị từ SVG do CQT sinh.
- **Trường dữ liệu kết quả trả về:**
  1. `tthai`: Tên trạng thái hóa đơn (Hóa đơn mới tạo lập, Hóa đơn bị thay thế, Hóa đơn bị điều chỉnh, Hóa đơn đã hủy...).
  2. `ttxly`: Trạng thái xử lý của Cơ quan Thuế (Đã cấp mã, Không cấp mã, Chờ cấp mã...).
  3. `tbssList`: Danh sách mã Thông báo sai sót Mẫu 04/SS gửi lên CQT gắn liền với hóa đơn này.
  4. `relatedInvoices`: Danh sách số hóa đơn điều chỉnh/thay thế tương ứng.

### 2.2. KTHD-FR-002 & KTHD-FR-003: Cấu trúc Dữ liệu Thẩm định File XML (`XmlData`)
Khi bóc tách file XML, hệ thống lưu trữ và trả về đối tượng thẩm định gồm các chỉ mục pháp lý:

| Nhóm Tiêu chí | Khóa Dữ liệu (Flag) | Kiểu Dữ liệu | Ý nghĩa Nghiệp vụ & Điều kiện Đạt |
|---|---|:---:|---|
| **Cấu trúc XML** | `xmlnven` | Boolean | `true`: File nguyên vẹn, tuân thủ XSD QĐ 1450/QĐ-TCT. |
| | `ignone_nbcks` | Boolean | `false`: File không bị can thiệp, dữ liệu khớp hoàn toàn với chữ ký số. |
| **Chữ ký số Người bán** | `nbcks` | Object | Trích xuất chữ ký số người bán. |
| | `nbcks.SigningBy` | String | Tên pháp nhân sở hữu chứng thư số. |
| | `nbcks.Issuer` | String | Tên nhà cung cấp chứng thư số công cộng (CA). |
| | `nkhdhle` | Boolean | `true`: Chứng thư số hợp lệ tại thời điểm ký. |
| | `nkhdtnlap` | Boolean | `true`: Ngày ký nằm trong khoảng hiệu lực của chứng thư số. |
| **Cấp mã Cơ quan Thuế** | `dcma` | Boolean | `true`: Hóa đơn đã được Cơ quan Thuế cấp mã. |
| | `cqtcma` | String | Chuỗi mã số của Cơ quan Thuế cấp cho hóa đơn. |
| | `cqtcmatnky` | String | Mã tiếp nhận theo kỳ của CQT. |
| | `is_sco` | Boolean | Phân loại hóa đơn từ máy tính tiền khởi tạo. |
| **Định danh Người nộp thuế** | `owner_match` | Boolean | `true`: MST và Tên đơn vị trên HĐ khớp hoàn toàn với CSDL Thuế. |
| | `owner_name` | String | Tên doanh nghiệp chính thức theo CSDL Tổng cục Thuế. |

### 2.3. KTHD-FR-003: Quy cách Giao diện Đối soát (Split View Dialog)
* **Khung hiển thị:** Mở hộp thoại toàn màn hình (Maximized Dialog) chia đôi tỷ lệ 55% - 45%:
  * **Cột trái (55%):** Nhúng thẻ Iframe an toàn gọi API `/upload_xml_data/:id/html` để nạp bản in hóa đơn đồ họa tiêu chuẩn có hoa văn chìm và mã vạch.
  * **Cột phải (45%):** Bảng tổng hợp trạng thái thẩm định với các thẻ trực quan:
    * Thẻ Trạng thái Chữ ký số (Dấu tích xanh nếu toàn bộ điều kiện chữ ký đạt, cảnh báo đỏ nếu hết hạn/không hợp lệ).
    * Thẻ Thông tin Bên bán & Đối soát CSDL Thuế.
    * Thẻ Tình trạng Cấp mã Cơ quan Thuế.
  * **Thanh tác vụ trên cùng (Action Bar):**
    * Nút **"In hóa đơn"** (Biểu tượng Máy in): Kích hoạt lệnh in Iframe bản thể hiện.
    * Nút **"In kết quả kiểm tra"**: Kích hoạt lệnh in phiếu kết quả thẩm định.
    * Nút **"Đóng"**: Thoát khỏi màn hình xem chi tiết.

### 2.4. KTHD-FR-004: Đồng bộ Trạng thái CQT cho Hóa đơn Nội bộ
* **Mục tiêu:** Cung cấp tính năng cho người dùng đã đăng nhập (kế toán doanh nghiệp) thực hiện tra cứu lại trạng thái thực tế từ CQT ngay trên màn hình chi tiết hóa đơn.
* **Xử lý:**
  1. Khi người dùng nhấn nút **"Kiểm tra trạng thái CQT"**, hệ thống gọi API sang Cổng Thuế để lấy thông tin trạng thái mới nhất (`result.tthai`).
  2. So sánh trạng thái trả về với dữ liệu đang lưu trong bảng `invoiceraws`:
     - Nếu phát hiện sai lệch (ví dụ: CSDL ghi nhận trạng thái 1 - Mới lập, nhưng CQT trả về 4 - Đã bị hủy):
     - Hệ thống tự động ghi đè trạng thái mới nhất vào CSDL (`patchTthai`).
     - Ghi nhận lịch sử kiểm tra vào nhật ký audit để kế toán đối soát (`KTHD-BR-005`).
* **Đầu ra:** Cập nhật ngay trạng thái mới trên giao diện người dùng và thông báo kết quả đối soát.

### 2.5. KTHD-FR-005: Tra cứu Định danh Doanh nghiệp Nguồn CQT
* **Mục tiêu:** Tự động đối soát và xác thực thông tin pháp nhân người bán dựa trên nguồn dữ liệu định danh chính thức của Cơ quan Thuế.
* **Xử lý:**
  1. Trích xuất mã số thuế người bán từ hóa đơn (`mst`).
  2. Truy vấn trực tiếp dịch vụ dữ liệu người nộp thuế để lấy: Tên pháp nhân đầy đủ, địa chỉ trụ sở đăng ký, tình trạng hoạt động (đang hoạt động, tạm ngừng, đóng MST).
  3. So khớp độ tương đồng tên trên hóa đơn và tên đăng ký; phát cảnh báo nếu phát hiện sai lệch mạo danh hoặc lỗi chính tả.
* **Đầu ra:** Dữ liệu định danh chuẩn hóa được hiển thị trực tiếp trong khung kết quả thẩm định hóa đơn.


