# DOC-06 — Đặc tả Yêu cầu Phần mềm Phân hệ Thuế Điện tử (MTAX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MTAX` — Quản lý Thuế Điện tử & Dịch vụ công Tổng cục Thuế  
> **Căn cứ mã nguồn thực tế:** `documents/features/mtax/`, `documents/c4/04-code/pipeline-mtax.md`, `mtax-api/`, `mtax-fe/`.

---

## 1. Yêu cầu Chức năng (Functional Requirements Register)

| Mã FR | Tên Chức năng | Mô tả Chi tiết | Mức ưu tiên | Trace (UC / BR) |
|:---:|---|---|:---:|:---:|
| **MTAX-FR-001** | Xác thực & Kết nối Tài khoản eTax/DVC | Quản lý thông tin đăng nhập Cổng Thuế điện tử (`username`, `password`), tự động duy trì phiên làm việc và bảo mật 2 chiều. | Must | MTAX-UC-001<br>MTAX-BR-001, 006 |
| **MTAX-FR-002** | Đồng bộ & Tra cứu Tờ khai Thuế | Cào dữ liệu tờ khai thuế theo 2 nguồn `DVC` và `ETAX`, hỗ trợ lọc đa tiêu chí (loại tờ khai, kỳ nộp, trạng thái CQT). | Must | MTAX-UC-002<br>MTAX-BR-002, 003 |
| **MTAX-FR-003** | Đồng bộ & Quản lý Giấy nộp tiền NSNN | Kéo thông tin chứng từ nộp tiền vào Kho bạc Nhà nước, hiển thị số tiền, mã chương, mã tiểu mục nộp thuế và tải file PDF/XML. | Must | MTAX-UC-003<br>MTAX-BR-002, 004 |
| **MTAX-FR-004** | Đồng bộ & Quản lý Thông báo Thuế | Kéo toàn bộ thông báo tiếp nhận (bước 1) và chấp nhận/từ chối (bước 2) từ CQT; liên kết trực tiếp với tờ khai liên quan. | Must | MTAX-UC-004<br>MTAX-BR-002, 005 |
| **MTAX-FR-005** | Tải Tệp XML Gốc & Xuất Báo cáo Excel | Hỗ trợ tải file XML tờ khai nguyên vẹn (đơn lẻ hoặc đóng gói ZIP) để mở qua phần mềm iTaxViewer; hỗ trợ xuất bảng kê danh sách ra tệp Excel. | Must | MTAX-UC-005<br>MTAX-BR-002, 004 |
| **MTAX-FR-006** | Đồng bộ Danh mục Tờ khai Đăng ký | Quét danh sách các loại tờ khai mà người nộp thuế đã đăng ký nộp qua mạng để làm bộ lọc master data. | Should | MTAX-UC-002<br>MTAX-BR-002 |

---

## 2. Đặc tả Chi tiết Yêu cầu Chức năng

### 2.1. MTAX-FR-001: Xác thực & Kết nối Tài khoản eTax/DVC
* **Mục tiêu:** Cho phép người dùng cấu hình tài khoản truy cập Cổng Thuế điện tử (`thuedientu.gdt.gov.vn`) hoặc Cổng Dịch vụ công Quốc gia để hệ thống tự động cào dữ liệu nghĩa vụ thuế.
* **Đầu vào (Inputs):**
  - Cổng nguồn: Chọn `ETAX` hoặc `DVC`.
  - Mã số thuế: Chuỗi 10 hoặc 13 số.
  - Tên đăng nhập phụ (nếu có, ví dụ: `${MST}-QL` đối với tài khoản phân quyền quản lý).
  - Mật khẩu đăng nhập.
* **Xử lý (Processing):**
  1. Kiểm tra định dạng đầu vào; mã hóa đối xứng mật khẩu bằng thuật toán `AES-256-CBC` trước khi lưu vào bảng `mtax_accounts`.
  2. Kích hoạt tính năng **"Kiểm tra kết nối"**: Gọi API crawler gửi thông tin đăng nhập kèm giải mã Captcha sang Cổng eTax/DVC.
  3. Nếu xác thực thành công, lưu trữ session token/cookie vào bộ nhớ đệm an toàn, thiết lập cơ chế tự động refresh token định kỳ ngầm.
  4. Phân lập hoàn toàn phiên xác thực của mTax với phiên cào hóa đơn điện tử (`MTAX-BR-006`).
* **Đầu ra (Outputs):** Thông báo trạng thái kết nối thành công, ngày kết nối, hiển thị huy hiệu "Đã kết nối" trên giao diện quản trị.
* **Xử lý ngoại lệ:** Nếu sai thông tin đăng nhập, phản hồi mã lỗi `ERR_MTAX_AUTH_FAILED` kèm lý do cụ thể do Cổng Thuế trả về (mật khẩu không đúng, tài khoản bị tạm khóa, lỗi mạng CQT).

---

### 2.2. MTAX-FR-002: Đồng bộ & Tra cứu Tờ khai Thuế (`Declaration`)
* **Mục tiêu:** Tự động thu thập toàn bộ lịch sử các tờ khai thuế mà doanh nghiệp đã nộp qua mạng, cung cấp giao diện quản lý và tra cứu đa chiều.
* **Đầu vào:** Bộ lọc tìm kiếm: Loại tờ khai, Kỳ tính thuế (Tháng/Quý/Năm), Khoảng ngày nộp (`tuNgay` - `denNgay`), Trạng thái phê duyệt của CQT.
* **Xử lý:**
  1. Worker định kỳ lấy job từ hàng đợi RabbitMQ `mtax_crawl`, gửi request tra cứu lịch sử nộp tờ khai sang Cổng Thuế.
  2. Phân trang dữ liệu thu thập được từ CQT và bóc tách thành các bản ghi trong bộ sưu tập `mtax_declarations`.
  3. Ánh xạ trạng thái tiếp nhận theo chuẩn: `1` - Chờ tiếp nhận, `2` - Đã tiếp nhận (bước 1), `3` - Đã chấp nhận (bước 2), `4` - Không chấp nhận (`MTAX-BR-003`).
* **Đầu ra (Dữ liệu hiển thị trên bảng):**
  1. `tenToKhai`: Tên loại tờ khai (ví dụ: *Tờ khai thuế giá trị gia tăng - Mẫu 01/GTGT*).
  2. `maToKhai`: Mã mẫu biểu (ví dụ: `01/GTGT`, `01/MBAI`, `02/TNDN`).
  3. `kyTinhThue`: Kỳ tính thuế (ví dụ: *Quý 3/2026*, *Tháng 10/2026*).
  4. `loaiToKhai`: `0` - Chính thức; `>0` - Bổ sung lần thứ N.
  5. `ngayNop`: Ngày giờ nộp hồ sơ thành công lên Cổng Thuế.
  6. `maGiaoDich`: Dãy số định danh giao dịch duy nhất do Tổng cục Thuế cấp.
  7. `coQuanThue`: Tên Chi cục Thuế / Cục Thuế quản lý tiếp nhận.
  8. `trangThai`: Huy hiệu trạng thái trực quan (`Đã tiếp nhận`, `Đã chấp nhận`, `Bị từ chối`).
  9. `thaoTac`: Xem chi tiết, nút tải XML gốc, nút mở văn bản thông báo CQT đi kèm.

---

### 2.3. MTAX-FR-003: Đồng bộ & Quản lý Giấy nộp tiền NSNN (`Payment`)
* **Mục tiêu:** Kéo và lưu trữ toàn bộ các chứng từ nộp tiền vào Ngân sách Nhà nước đã được Kho bạc Nhà nước xác nhận thành công.
* **Đầu vào:** Kỳ nộp thuế, mã tiểu mục, khoảng ngày phát sinh nộp tiền, ngân hàng trích nợ.
* **Xử lý:**
  1. Thu thập dữ liệu từ phân hệ nộp thuế của Cổng eTax.
  2. Bóc tách các trường tài chính bắt buộc: Mã chương, mã tiểu mục mục lục ngân sách, cơ quan thu, tài khoản Kho bạc thụ hưởng, số tiền nộp (`MTAX-BR-004`).
  3. Tải và lưu trữ tệp XML/PDF chứng từ nộp thuế thành công có chữ ký số của Ngân hàng thương mại và Kho bạc Nhà nước.
* **Đầu ra (Dữ liệu hiển thị):**
  1. `soChungTu`: Số giấy nộp tiền / Số tham chiếu giao dịch.
  2. `ngayNop`: Thời điểm tiền được trừ vào tài khoản ngân hàng và chuyển sang Kho bạc.
  3. `khoBac`: Tên Kho bạc Nhà nước thụ hưởng.
  4. `coQuanThu`: Cơ quan thuế quản lý khoản thu.
  5. `maTieuMuc`: Mã tiểu mục mục lục ngân sách (ví dụ: `1701` - Thuế GTGT; `1052` - Thuế TNDN).
  6. `soTien`: Số tiền nộp (định dạng phân cách hàng nghìn, VNĐ).
  7. `trangThai`: `Nộp thuế thành công` (có dấu xác nhận của ngân hàng & Kho bạc).
  8. `thaoTac`: Tra cứu chi tiết giao dịch nộp thuế, tải file chứng từ nộp thuế / XML gốc.

---

### 2.4. MTAX-FR-004: Đồng bộ & Quản lý Thông báo Thuế (`Notice`)
* **Mục tiêu:** Thu thập các văn bản thông báo chính thức do Cơ quan Thuế phát hành gửi tới người nộp thuế và liên kết tự động với tờ khai tương ứng.
* **Xử lý:**
  1. Phân loại thông báo thu thập được:
     - *Thông báo bước 1:* Thông báo tiếp nhận hồ sơ khai thuế điện tử (gửi trong vòng 15 phút sau khi nộp).
     - *Thông báo bước 2:* Thông báo chấp nhận hoặc không chấp nhận hồ sơ khai thuế điện tử (gửi trong vòng 1 ngày làm việc).
     - *Thông báo nộp thuế:* Thông báo về số tiền thuế phải nộp, tiền phạt chậm nộp (nếu có).
  2. Phân tích mã hồ sơ khai thuế hoặc mã giao dịch điện tử để thiết lập liên kết hai chiều với bản ghi tờ khai thuế trong bảng `mtax_declarations` (`MTAX-BR-005`).
* **Đầu ra:** Giao diện hiển thị danh sách văn bản thông báo kèm nút mở xem trực tiếp nội dung PDF/HTML và xem tờ khai gốc liên đới.

---

### 2.5. MTAX-FR-005: Tải Tệp XML Gốc & Xuất Báo cáo Excel
* **Mục tiêu:** Hỗ trợ kết xuất tệp XML gốc để người dùng có thể mở kiểm tra bằng ứng dụng iTaxViewer của Tổng cục Thuế và xuất báo cáo tổng hợp danh mục ra Excel.
* **Đầu vào:** Lựa chọn tải 1 tệp hoặc tích chọn nhiều tờ khai/giấy nộp tiền để tải hàng loạt.
* **Xử lý:**
  1. **Tải tệp XML đơn lẻ:** Truy xuất từ kho lưu trữ Object Storage MinIO và pipe dữ liệu về client với đúng tên tệp chuẩn do CQT ban hành.
  2. **Tải hàng loạt (Batch Download):** Đóng gói toàn bộ các file XML được chọn vào 1 tệp nén `.zip`, đặt tên theo cấu trúc `{MST}_HoSoThue_{TuNgay}_{DenNgay}.zip`.
  3. **Xuất báo cáo Excel:** Tổng hợp các cột: Tên tờ khai, kỳ tính thuế, mã giao dịch, ngày nộp, trạng thái CQT, số tiền nộp (đối với giấy nộp tiền) vào bảng tính Excel định dạng chuẩn để phục vụ lưu trữ nội bộ.
* **Đầu ra:** Stream tải file ZIP/XML/Excel với Header `Content-Disposition: attachment`.

---

### 2.6. MTAX-FR-006: Đồng bộ Danh mục Tờ khai Đăng ký (`Category`)
* **Mục tiêu:** Quét toàn bộ danh mục các loại tờ khai mà người nộp thuế đã đăng ký nộp qua mạng với Cơ quan Thuế quản lý trực tiếp.
* **Xử lý:**
  1. Gọi API Cổng Thuế để lấy danh sách loại tờ khai đã đăng ký (Tờ khai GTGT tháng/quý, tờ khai quyết toán thuế TNDN, tờ khai thuế môn bài...).
  2. Lưu trữ vào bảng `mtax_categories` làm dữ liệu Master Data.
  3. Sử dụng danh mục này để:
     - Tự động điền danh sách gợi ý trong dropdown bộ lọc tra cứu tờ khai.
     - Cảnh báo người dùng nếu đến hạn nộp tờ khai định kỳ mà hệ thống chưa ghi nhận tờ khai nộp thành công trên Cổng Thuế.
* **Đầu ra:** Bảng danh mục loại tờ khai đăng ký nộp qua mạng hiển thị trạng thái hiệu lực (`Đang nộp`, `Ngừng nộp`).
