# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn (KIEMTRAHD)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `KIEMTRAHD` — Cổng Kiểm tra & Tra cứu Hóa đơn Điện tử Công khai  
> **Căn cứ mã nguồn thực tế:** `kiemtrahoadon/`, `documents/c4/03-components/kiemtra-hd-portal.md`, `service/proxy-service/` (Node.js/Express).

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| Mã Rule | Tên Quy tắc | Mô tả Tóm tắt | Loại Rule | Mức ưu tiên | Trace |
|:---:|---|---|:---:|:---:|:---:|
| **KTHD-BR-001** | Tra cứu Công khai Không Cần Định danh | Cho phép mọi tổ chức, cá nhân tra cứu tính hợp lệ của hóa đơn mà không cần đăng nhập. Bắt buộc giải mã Captcha SVG từ Cơ quan Thuế để chống lạm dụng. | Security / Access | Must | KTHD-UC-001<br>KTHD-FR-001 |
| **KTHD-BR-002** | Rẽ nhánh & Chuẩn hóa Tham số Tra cứu CQT | Chuẩn hóa ký hiệu (cắt ký tự đầu nếu 7 ký tự). Bắt buộc nhập Tổng tiền thanh toán với Mẫu 1–5, hoặc Ngày lập đối với Mẫu 6 (vé/thẻ). | Validation | Must | KTHD-UC-001<br>KTHD-FR-001 |
| **KTHD-BR-003** | Kiểm tra Toàn diện Tính Pháp lý File XML | Kiểm tra 5 tiêu chí: Cấu trúc XML chuẩn XSD, tính toàn vẹn chữ ký số người bán, thời hạn chứng thư số, mã CQT cấp và so khớp MST người bán với CSDL Thuế. | Integrity / Compliance | Must | KTHD-UC-002<br>KTHD-FR-002 |
| **KTHD-BR-004** | Phân quyền Hiển thị Khách vãng lai | Khách vãng lai chỉ được xem bản thể hiện HTML, in hóa đơn và in kết quả kiểm tra. Ẩn toàn bộ nút tải XML gốc, ghi chú, gắn nhãn doanh nghiệp. | Authorization | Must | KTHD-UC-003<br>KTHD-FR-003 |
| **KTHD-BR-005** | Đồng bộ Trạng thái CQT Ngược về Hệ thống | Khi người dùng có phiên nội bộ kiểm tra trạng thái từ CQT, nếu trạng thái CQT thay đổi, hệ thống cập nhật tự động vào CSDL hóa đơn lưu trữ. | Consistency | Should | KTHD-UC-004<br>KTHD-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ

### 2.1. KTHD-BR-001: Tra cứu Công khai & Bảo vệ Captcha
* Hệ thống vận hành tại cổng công khai độc lập (domain `kiemtrahoadon.vn`), không yêu cầu người dùng phải đăng ký tài khoản hay duy trì phiên đăng nhập (`me = null`).
* Để chống việc khai thác dữ liệu tự động (crawler/botnet), mọi lượt tra cứu trực tiếp tới Cổng Dịch vụ Hóa đơn điện tử của Cơ quan Thuế bắt buộc kèm mã kiểm tra hình ảnh (`Captcha`).
* Captcha được sinh trực tiếp dưới dạng đồ họa vector SVG từ Cơ quan Thuế và tự động làm mới ngay sau mỗi lần gửi yêu cầu tra cứu (dù thành công hay thất bại).

### 2.2. KTHD-BR-002: Chuẩn hóa & Rẽ nhánh Tham số Tra cứu
Form tra cứu trực tiếp CQT yêu cầu xác thực các thông tin đầu vào theo Nghị định 123/2020/NĐ-CP:
1. **Mã số thuế người bán (`mst`):** Đúng định dạng 10 số (doanh nghiệp) hoặc 13 số (chi nhánh/đơn vị trực thuộc).
2. **Loại hóa đơn (`type`):** Gồm 8 nhóm phân loại:
   * Mẫu 1: Hóa đơn giá trị gia tăng.
   * Mẫu 2: Hóa đơn bán hàng.
   * Mẫu 3: Hóa đơn bán tài sản công.
   * Mẫu 4: Hóa đơn bán hàng dự trữ quốc gia.
   * Mẫu 5: Hóa đơn khác (tem, vé, thẻ...).
   * Mẫu 6: Chứng từ điện tử / Vé điện tử.
   * Mẫu 06_01: Phiếu xuất kho kiêm vận chuyển nội bộ.
   * Mẫu 06_02: Phiếu xuất kho gửi bán hàng đại lý.
3. **Ký hiệu hóa đơn (`khhdon`):**
   * Tối đa 6 ký tự viết hoa (ví dụ: `C24TAA`, `K24TYY`).
   * *Quy tắc chuẩn hóa tự động:* Nếu người dùng nhập 7 ký tự (do bao gồm chữ số loại hóa đơn phía trước như `1C24TAA`), hệ thống tự động cắt bỏ ký tự đầu tiên để lấy 6 ký tự hợp lệ phía sau.
4. **Số hóa đơn (`shdon`):** Chuỗi ký số từ 1 đến 8 chữ số (ví dụ: `00000001` hoặc `1`).
5. **Rẽ nhánh điều kiện kiểm tra tài chính:**
   * **Nếu `khmshdon ≠ 6`:** Bắt buộc nhập trường `Tổng tiền thanh toán` (`tgtttbso`). Kiểm tra hợp lệ tối đa 14 chữ số phần nguyên và 7 chữ số phần thập phân.
   * **Nếu `khmshdon = 6`:** Ẩn trường Tổng tiền thanh toán; bắt buộc người dùng chọn `Ngày lập hóa đơn` (`tdlap`) định dạng ngày tháng năm.

### 2.3. KTHD-BR-003: Tiêu chuẩn Kiểm tra File XML Hóa đơn Điện tử
Khi người dùng tải lên một tệp tin hóa đơn định dạng `.xml`, hệ thống phân tích cú pháp và kiểm định theo 5 tiêu chí pháp lý độc lập:
1. **Tính nguyên vẹn của cấu trúc tệp XML (`xmlnven`):**
   * Tệp XML phải tuân thủ nghiêm ngặt chuẩn cấu trúc dữ liệu hóa đơn điện tử của Tổng cục Thuế (theo Quyết định 1450/QĐ-TCT).
   * Cờ cảnh báo `ignone_nbcks = false` khẳng định tệp không bị can thiệp, chỉnh sửa nội dung sau khi ký số.
2. **Tính hợp lệ của Chữ ký số Người bán (`nbcks`):**
   * Bóc tách thông tin chứng thư số: Tên đơn vị ký (`SigningBy`), Tổ chức cung cấp dịch vụ chứng thực chữ ký số công cộng (`Issuer` - ví dụ: VNPT, Viettel, M-Invoice CA...).
   * Đối chiếu thời hạn hiệu lực của chứng thư số: Chứng thư số phải còn hạn sử dụng tại thời điểm lập hóa đơn (`nkhdhle = true`, `nkhdtnlap = true`).
3. **Trạng thái Cấp mã của Cơ quan Thuế:**
   * Tự động nhận diện loại hóa đơn: Ký hiệu bắt đầu bằng chữ `C` là hóa đơn **Có mã của CQT**; bắt đầu bằng chữ `K` là hóa đơn **Không có mã của CQT**.
   * Với hóa đơn có mã: Kiểm tra sự tồn tại của chuỗi mã CQT cấp (`dcma`, `cqtcma`, `cqtcmatnky`, `is_sco`). Nếu hóa đơn có mã mà thiếu mã CQT, hệ thống phát cảnh báo vi phạm pháp lý.
4. **So khớp Định danh Người nộp thuế (`owner_match`):**
   * Hệ thống tự động gọi dịch vụ cơ sở dữ liệu quốc gia về người nộp thuế để đối soát MST và Tên đơn vị bán.
   * Cảnh báo nếu tên trên hóa đơn sai khác so với tên đăng ký chính thức với cơ quan thuế (`owner_name`).
5. **Phân tích Rủi ro Doanh nghiệp:**
   * Cảnh báo nếu MST người bán thuộc danh sách cảnh báo rủi ro cao về thuế, ngừng hoạt động nhưng chưa hoàn thành thủ tục đóng MST, hoặc doanh nghiệp bỏ trốn.

### 2.4. KTHD-BR-004: Phân quyền Trải nghiệm Khách vãng lai
* **Giao diện Trực quan Song song (Split View):**
  * Nửa bên trái: Hiển thị bản thể hiện đồ họa HTML đầy đủ của hóa đơn (render từ dữ liệu XML theo mẫu phôi in tiêu chuẩn).
  * Nửa bên phải: Hiển thị chi tiết bảng kết luận thẩm định tính hợp lệ của hóa đơn, thông tin chữ ký số và trạng thái thuế.
* **Giới hạn Thao tác Khách vãng lai (`me = null`):**
  * Cho phép: Xem bản thể hiện, Xem kết quả kiểm tra, In hóa đơn (In bản thể hiện), In kết quả kiểm tra.
  * Nghiêm cấm / Ẩn hoàn toàn: Nút tải file XML dữ liệu gốc, Thêm ghi chú nội bộ, Gắn nhãn quản lý (`tag`), Lưu trữ vào kho dữ liệu doanh nghiệp.

### 2.5. KTHD-BR-005: Đồng bộ Trạng thái Hóa đơn Nội bộ
* Khi người dùng đã đăng nhập hệ thống nội bộ (kế toán doanh nghiệp) truy cập xem hóa đơn qua màn hình chi tiết (`#/invoice-info?id=...`), người dùng có quyền nhấn nút **Tra cứu trạng thái CQT**.
* Nếu trạng thái thực tế trả về từ CQT khác với trạng thái đang lưu trữ trong cơ sở dữ liệu (`result.tthai !== invoiceInfo.tthai`), hệ thống tự động ghi đè cập nhật trạng thái mới nhất (`patchTthai`) và lưu vết lịch sử kiểm tra.

