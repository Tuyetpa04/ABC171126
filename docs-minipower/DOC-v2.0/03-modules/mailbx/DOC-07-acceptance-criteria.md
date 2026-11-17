# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Quản lý Hộp thư & Bóc tách Hóa đơn từ Email (MAILBX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `MAILBX` — Quản lý Hộp thư & Tự động bóc tách hóa đơn từ Email  
> **Căn cứ tài liệu gốc:** `TÀI LIỆU HƯỚNG DẪN ĐỒNG BỘ HÓA ĐƠN QUA EMAIL.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 7), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 1).

---

## 1. Danh sách Tiêu chí Nghiệm thu (Acceptance Criteria)

| Mã AC | Tiêu đề Kịch bản | Ca sử dụng liên quan | Yêu cầu liên quan |
|:---:|---|:---:|:---:|
| **MAILBX-AC-001** | Kết nối thành công Hộp thư Gmail qua App Password | MAILBX-UC-001 | MAILBX-FR-001 |
| **MAILBX-AC-002** | Bóc tách tự động email có tệp đính kèm XML | MAILBX-UC-002, 004 | MAILBX-FR-002, 003 |
| **MAILBX-AC-003** | Tự động giải nén tệp ZIP và bóc tách nhiều XML bên trong | MAILBX-UC-004 | MAILBX-FR-003 |
| **MAILBX-AC-004** | Khử trùng lặp khi hóa đơn email đã tồn tại trong CSDL | MAILBX-UC-004 | MAILBX-FR-004 |
| **MAILBX-AC-005** | Bỏ qua thư không chứa tệp hóa đơn hợp lệ | MAILBX-UC-002 | MAILBX-FR-002 |

---

## 2. Đặc tả Kịch bản Nghiệm thu (Gherkin Scenarios)

### MAILBX-AC-001: Kết nối thành công Hộp thư Gmail qua App Password

```gherkin
Kịch bản: Cấu hình và kết nối thành công tài khoản Gmail bằng App Password
  Cho rằng Kế toán viên đang ở màn hình Cấu hình Hộp thư hóa đơn
  Khi Kế toán viên nhập địa chỉ email "ketoan.doanhnghiep@gmail.com"
  Và Kế toán viên nhập Mật khẩu ứng dụng 16 ký tự hợp lệ
  Và Kế toán viên bấm nút "Kiểm tra kết nối"
  Thì Hệ thống kết nối thử nghiệm thành công tới máy chủ Google
  Và Hệ thống hiển thị thông báo "Kết nối thành công"
  Khi Kế toán viên bấm nút "Lưu"
  Thì Trạng thái hòm thư chuyển thành "Đang hoạt động" (Active)
  Và Mật khẩu được mã hóa an toàn trước khi lưu vào CSDL.
```

---

### MAILBX-AC-002: Bóc tách tự động email có tệp đính kèm XML

```gherkin
Kịch bản: Tiếp nhận email có file đính kèm XML hóa đơn hợp lệ
  Cho rằng Robot nền đang thực hiện phiên quét hòm thư tự động
  Khi Hộp thư nhận được 1 email mới từ nhà cung cấp có chứa file đính kèm "hoadon_123.xml"
  Thì Hệ thống phát hiện email có tệp XML và gán cờ hasMimeXml = true
  Và Hệ thống tự động phân tích cú pháp file "hoadon_123.xml"
  Và Hệ thống trích xuất chính xác MST người bán, Số hóa đơn, Ngày lập, Tổng tiền
  Và Một bản ghi mới được tạo trong bảng danh sách Hóa đơn mua vào
  Và Tệp "hoadon_123.xml" được lưu trữ an toàn trong kho dữ liệu mSMI.
```

---

### MAILBX-AC-003: Tự động giải nén tệp ZIP và bóc tách nhiều XML bên trong

```gherkin
Kịch bản: Tự động bung file nén ZIP chứa nhiều hóa đơn
  Cho rằng Nhà cung cấp gửi 1 email đính kèm tệp nén "hoadon_thang10.zip"
  Và Bên trong file ZIP có chứa 3 file XML hóa đơn hợp lệ và 3 file PDF thể hiện
  Khi Robot nền tiến hành quét và tải email này về
  Thì Hệ thống tự động giải nén tệp "hoadon_thang10.zip"
  Và Hệ thống nhận diện được cả 3 file XML hợp lệ
  Thì 3 bản ghi hóa đơn mới được trích xuất và thêm vào kho hóa đơn
  Và Mỗi hóa đơn đều được gắn kèm file XML và file PDF tương ứng.
```

---

### MAILBX-AC-004: Khử trùng lặp khi hóa đơn email đã tồn tại trong CSDL

```gherkin
Kịch bản: Email gửi hóa đơn đã từng được đồng bộ từ Cổng Cơ quan Thuế
  Cho rằng Hóa đơn số 0000045, ký hiệu 1C24TAA, MST bán 0101234567 đã có trong hệ thống từ nguồn CQT
  Khi Robot bóc tách email nhận được một hóa đơn có cùng MST bán 0101234567, ký hiệu 1C24TAA, số 0000045
  Thì Hệ thống nhận diện hóa đơn này đã tồn tại trong CSDL
  Và Hệ thống không tạo thêm một bản ghi hóa đơn trùng lặp mới
  Và Hệ thống bổ sung tệp đính kèm từ email vào bản ghi hóa đơn đã có
  Và Nguồn gốc hóa đơn được ghi nhận có xuất hiện qua cả 2 kênh CQT và Email.
```

---

### MAILBX-AC-005: Bỏ qua thư không chứa tệp hóa đơn hợp lệ

```gherkin
Kịch bản: Bỏ qua email tiếp thị hoặc email không có tệp hóa đơn
  Cho rằng Hộp thư nhận được 1 email thông báo không có tệp đính kèm hoặc chỉ có hình ảnh chữ ký
  Khi Robot tiến hành quét email này
  Thì Hệ thống xác định email không có tệp .xml và không có tệp .zip
  Và Hệ thống gán cờ hasMimeXml = false
  Và Không kích hoạt tiến trình bóc tách hóa đơn
  Và Thư được lưu vào danh sách email thông thường mà không tạo rác trong danh sách hóa đơn.
```

