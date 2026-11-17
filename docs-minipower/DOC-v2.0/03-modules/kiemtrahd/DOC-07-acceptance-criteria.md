# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn (KIEMTRAHD)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** BDD (Behavior-Driven Development) — Given / When / Then.  
> **Phân hệ:** `KIEMTRAHD` — Cổng Kiểm tra & Tra cứu Hóa đơn Điện tử Công khai.

---

## 1. Danh mục Kịch bản Nghiệm thu (Acceptance Criteria Register)

| Mã AC | Tên Kịch bản Nghiệm thu | Functional Requirement | Trọng yếu |
|:---:|---|---|:---:|
| **KTHD-AC-001** | Tra cứu trực tiếp hóa đơn công khai từ CQT thành công | `KTHD-FR-001` | Must |
| **KTHD-AC-002** | Tự động chuẩn hóa ký hiệu 7 ký tự và rẽ nhánh Mẫu 6 | `KTHD-FR-001`, `KTHD-BR-002` | Must |
| **KTHD-AC-003** | Thẩm định tính toàn vẹn và chữ ký số file XML hợp lệ | `KTHD-FR-002` | Must |
| **KTHD-AC-004** | Xem bản thể hiện song song và in ấn đối với khách vãng lai | `KTHD-FR-003`, `KTHD-BR-004` | Must |
| **KTHD-AC-005** | Tra cứu lại CQT và đồng bộ trạng thái hóa đơn nội bộ | `KTHD-FR-004`, `KTHD-BR-005` | Should |

---

## 2. Đặc tả Chi tiết Từng Kịch bản Nghiệm thu

### KTHD-AC-001: Tra cứu trực tiếp hóa đơn công khai từ CQT thành công
```gherkin
Kịch bản: Khách vãng lai tra cứu hóa đơn GTGT hợp lệ trên Cổng Thuế
  Giả sử Người dùng đang ở trang chủ tra cứu "https://kiemtrahoadon.vn"
  Và Ảnh Captcha SVG từ Cơ quan Thuế đã được nạp thành công
  Khi Người dùng nhập MST người bán "0101234567"
  Và Chọn loại hóa đơn "Mẫu 1 - Hóa đơn giá trị gia tăng"
  Và Nhập ký hiệu "C24TAA", số hóa đơn "12345"
  Và Nhập tổng tiền thanh toán "11000000"
  Và Nhập chính xác chuỗi ký tự Captcha hiển thị trên màn hình
  Và Bấm nút "Tìm kiếm"
  Thì Hệ thống gửi yêu cầu proxy sang Cổng Thuế
  Và Trả về kết quả hiển thị trạng thái hóa đơn "Hóa đơn mới tạo lập"
  Và Trạng thái xử lý của CQT là "Đã cấp mã"
  Và Tự động làm mới một mã Captcha mới trên form tra cứu
```

### KTHD-AC-002: Tự động chuẩn hóa ký hiệu và rẽ nhánh Mẫu 6 (Vé/thẻ)
```gherkin
Kịch bản 1: Người dùng nhập ký hiệu hóa đơn 7 ký tự
  Giả sử Người dùng đang ở màn hình tra cứu hóa đơn
  Khi Người dùng dán chuỗi "1C24TUU" vào ô "Ký hiệu hóa đơn"
  Thì Hệ thống tự động cắt bỏ ký tự đầu "1", chuyển thành "C24TUU" (6 ký tự viết hoa)

Kịch bản 2: Người dùng chọn loại hóa đơn Mẫu 6 (Vé/thẻ điện tử)
  Giả sử Người dùng đang chọn loại hóa đơn
  Khi Người dùng chọn "Mẫu 6 - Vé, thẻ, chứng từ điện tử"
  Thì Trường nhập "Tổng tiền thanh toán" được ẩn đi
  Và Trường chọn ngày "Ngày lập hóa đơn" xuất hiện bắt buộc nhập
```

### KTHD-AC-003: Thẩm định tính toàn vẹn và chữ ký số file XML hợp lệ
```gherkin
Kịch bản: Tải lên file XML hóa đơn có chữ ký số hợp lệ và mã CQT
  Giả sử Người dùng chọn tệp tin XML "HD_0101234567_C24TAA_12345.xml"
  Khi Người dùng kéo-thả tệp tin vào khu vực tải lên
  Thì Hệ thống phân tích cú pháp và hiển thị thông báo kiểm tra thành công
  Và Cột trạng thái chữ ký số hiển thị tích xanh "Chữ ký số hợp lệ"
  Và Hiển thị đúng Tên đơn vị ký và Nhà cung cấp dịch vụ chứng thực chữ ký số
  Và Cờ kiểm tra tính toàn vẹn XML "xmlnven" đạt giá trị hợp lệ
  Và Hiển thị mã Cơ quan Thuế cấp kèm trạng thái "Đã cấp mã"
```

### KTHD-AC-004: Xem bản thể hiện song song và in ấn đối với khách vãng lai
```gherkin
Kịch bản: Khách vãng lai xem và in bản thể hiện sau khi kiểm tra XML
  Giả sử Người dùng chưa đăng nhập hệ thống (khách vãng lai me=null) vừa tải lên file XML thành công
  Khi Hệ thống mở hộp thoại đối soát toàn màn hình (Split view)
  Thì Nửa bên trái hiển thị bản in hóa đơn HTML tiêu chuẩn có đầy đủ thông tin bên bán, bên mua và chi tiết dòng hàng
  Và Nửa bên phải hiển thị chi tiết các tiêu chí kiểm tra pháp lý
  Và Hệ thống chỉ hiển thị 2 nút tác vụ: "In hóa đơn" và "In kết quả kiểm tra"
  Và Toàn bộ các nút nội bộ ("Tải XML gốc", "Thêm ghi chú", "Gắn nhãn") đều bị ẩn
  Khi Người dùng bấm nút "In hóa đơn"
  Thì Hộp thoại in của trình duyệt được kích hoạt với định dạng in tối ưu không bị vỡ khung
```

### KTHD-AC-005: Tra cứu lại CQT và đồng bộ trạng thái hóa đơn nội bộ
```gherkin
Kịch bản: Kế toán viên kiểm tra lại hóa đơn nội bộ bị hủy trên Cổng Thuế
  Giả sử Kế toán viên đã đăng nhập vào hệ thống mSMI và mở chi tiết một hóa đơn có trạng thái "Gốc"
  Khi Kế toán bấm nút "Tra cứu CQT" trong hộp thoại kiểm tra nhanh
  Và Nhập mã Captcha để xác thực
  Thì Hệ thống truy vấn Cổng Thuế và phát hiện trạng thái thực tế là "Hóa đơn đã hủy"
  Và Hệ thống tự động kích hoạt cập nhật trạng thái mới nhất vào hóa đơn nội bộ
  Và Giao diện lập tức đổi màu trạng thái hóa đơn sang đỏ "Đã hủy" kèm thời gian đối soát mới nhất
```

