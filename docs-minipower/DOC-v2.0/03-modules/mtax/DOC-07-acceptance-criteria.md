# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Thuế Điện tử (MTAX)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** BDD (Behavior-Driven Development) — Given / When / Then.  
> **Phân hệ:** `MTAX` — Quản lý Thuế Điện tử & Dịch vụ công.

---

## 1. Danh mục Kịch bản Nghiệm thu (Acceptance Criteria Register)

| Mã AC | Tên Kịch bản Nghiệm thu | Functional Requirement | Trọng yếu |
|:---:|---|:---:|:---:|
| **MTAX-AC-001** | Kết nối & Duy trì phiên tài khoản Thuế điện tử thành công | `MTAX-FR-001` | Must |
| **MTAX-AC-002** | Đồng bộ và phân trang danh sách tờ khai từ nguồn DVC/eTax | `MTAX-FR-002` | Must |
| **MTAX-AC-003** | Cảnh báo tờ khai bị CQT từ chối kèm văn bản thông báo | `MTAX-FR-002`, `004` | Must |
| **MTAX-AC-004** | Tra cứu Giấy nộp tiền và bóc tách đầy đủ mã tiểu mục | `MTAX-FR-003` | Must |
| **MTAX-AC-005** | Tải tệp XML tờ khai gốc và xuất báo cáo danh sách Excel | `MTAX-FR-005` | Must |

---

## 2. Đặc tả Chi tiết Từng Kịch bản Nghiệm thu

### MTAX-AC-001: Kết nối tài khoản Thuế điện tử thành công
```gherkin
Kịch bản: Thêm mới tài khoản eTax hợp lệ
  Giả sử Người dùng đang ở màn hình Cấu hình tài khoản Thuế điện tử
  Khi Người dùng nhập Mã số thuế "0101234567"
  Và Nhập tên đăng nhập "0101234567-QL" và Mật khẩu eTax chính xác
  Và Bấm nút "Kiểm tra kết nối & Lưu"
  Thì Hệ thống gửi yêu cầu xác thực sang Cổng Thuế điện tử
  Và Trả về thông báo "Kết nối tài khoản Thuế điện tử thành công"
  Và Trạng thái tài khoản chuyển sang màu xanh "Đang hoạt động"
```

### MTAX-AC-002: Đồng bộ danh sách tờ khai thuế
```gherkin
Kịch bản: Bấm đồng bộ tờ khai thuế Quý 3/2026
  Giả sử Doanh nghiệp đã kết nối tài khoản eTax đang hoạt động
  Khi Kế toán chọn nguồn "ETAX", chọn kỳ tính thuế "Quý 3/2026"
  Và Bấm nút "Đồng bộ ngay"
  Thì Hệ thống kích hoạt pipeline cào dữ liệu qua mtax-api
  Và Sau tối đa 15 giây, danh sách hiển thị các tờ khai đã nộp trong kỳ (01/GTGT, 01/MBAI...)
  Và Mỗi dòng hiển thị đầy đủ Ngày nộp, Mã giao dịch, Lần nộp và Trạng thái xử lý của CQT
```

### MTAX-AC-003: Cảnh báo tờ khai bị CQT từ chối
```gherkin
Kịch bản: Tờ khai bị CQT phản hồi Không chấp nhận
  Giả sử Tờ khai 01/GTGT có mã giao dịch "TCT123456" bị CQT từ chối do sai chữ ký số
  Khi Kế toán mở danh sách Tờ khai thuế
  Thì Hệ thống hiển thị huy hiệu màu đỏ "Không chấp nhận" tại cột Trạng thái
  Khi Kế toán nhấp vào liên kết "Xem thông báo từ chối"
  Thì Hệ thống mở chi tiết Thông báo bước 2 của CQT
  Và Nêu rõ nội dung lý do: "Chữ ký số của người nộp thuế không hợp lệ tại thời điểm ký"
```

### MTAX-AC-004: Tra cứu Giấy nộp tiền và mã tiểu mục
```gherkin
Kịch bản: Kiểm tra số tiền thuế GTGT đã nộp qua Kho bạc
  Giả sử Kế toán đang ở tab "Giấy nộp tiền NSNN"
  Khi Kế toán lọc theo mã tiểu mục "1701" (Thuế GTGT hàng nội địa) và năm "2026"
  Thì Hệ thống trả về danh sách các lần nộp tiền thuế GTGT thành công
  Và Hiển thị chính xác Số chứng từ, Ngày nộp, Kho bạc Nhà nước thụ hưởng và Số tiền nộp
  Khi Kế toán bấm nút "Tải XML"
  Thì Tệp XML gốc chứa chữ ký số xác nhận của Kho bạc và Ngân hàng được tải về máy tính
```

### MTAX-AC-005: Tải tệp XML tờ khai gốc và xuất báo cáo Excel
```gherkin
Kịch bản: Tải tệp XML tờ khai để mở qua iTaxViewer
  Giả sử Kế toán đang ở danh sách Tờ khai thuế và tìm thấy tờ khai 01/GTGT
  Khi Kế toán bấm nút "Tải XML" tại dòng tờ khai
  Thì Hệ thống tải tệp XML nguyên bản đã nộp về máy tính của người dùng
  Và Tệp XML này mở hợp lệ và hiển thị đầy đủ chỉ tiêu trên ứng dụng iTaxViewer

Kịch bản: Xuất danh sách báo cáo tờ khai ra tệp Excel
  Giả sử Danh sách tờ khai đang hiển thị 15 kết quả sau khi lọc theo năm 2026
  Khi Kế toán bấm nút "Xuất Excel"
  Thì Hệ thống kết xuất và tải về tệp Excel chứa toàn bộ thông tin các cột trong danh sách
```

