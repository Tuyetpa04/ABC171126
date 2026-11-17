# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Kết nối Cơ quan Thuế (TAXACC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** Gherkin BDD (Given / When / Then)  
> **Phân hệ:** `TAXACC` — Kết nối Cơ quan Thuế & Đồng bộ HĐĐT

---

## 1. Danh mục Tiêu chí Chấp nhận

| AC ID | FR ID | Tên Kịch bản Nghiệm thu | Mức ưu tiên |
|:---:|:---:|---|:---:|
| **TAXACC-AC-001** | TAXACC-FR-001 | Đăng ký tài khoản thuế thành công với thông tin hợp lệ | Must |
| **TAXACC-AC-002** | TAXACC-FR-001 | Trả về mã lỗi 07 khi người dùng nhập sai mật khẩu CQT | Must |
| **TAXACC-AC-003** | TAXACC-FR-003 | Kích hoạt đồng bộ thủ công kéo đúng số hóa đơn mới | Must |
| **TAXACC-AC-004** | TAXACC-FR-004 | Ghi nhận đầy đủ thông số phiên chạy vào lịch sử đồng bộ | Should |

---

## 2. Kịch bản Gherkin Chi tiết

### TAXACC-AC-001: Khai báo tài khoản thuế thành công (TAXACC-FR-001)
```gherkin
Feature: Thêm mới tài khoản Cổng HĐĐT Tổng cục Thuế
  As a Kế toán viên (ACT-001)
  I want to Kết nối tài khoản thuế của doanh nghiệp vào mSMI
  So that Hệ thống có thể tự động tải hóa đơn điện tử về

  Scenario: Nhập đúng tài khoản và mật khẩu trang thuế
    Given Kế toán đang ở màn hình Thêm tài khoản thuế
    When Kế toán nhập MST "0106026495" và mật khẩu hợp lệ
    And Kế toán bấm nút "Lưu & Đăng nhập"
    Then Hệ thống vượt Captcha và xác thực thành công với Cổng Thuế
    And Phản hồi mã kết quả "00" (Thành công)
    And Tài khoản mới xuất hiện trên danh sách với trạng thái Đang hoạt động
```

### TAXACC-AC-002: Báo lỗi khi sai mật khẩu thuế (TAXACC-FR-001)
```gherkin
Feature: Xử lý ngoại lệ khi sai thông tin đăng nhập CQT
  As a Kế toán viên (ACT-001)
  I want to Nhận được thông báo lỗi rõ ràng khi gõ sai mật khẩu
  So that Tôi có thể điều chỉnh lại thông tin chính xác

  Scenario: Người dùng nhập sai mật khẩu thuế
    Given Kế toán nhập MST "0106026495" và mật khẩu sai "WrongPass@123"
    When Kế toán bấm "Lưu & Đăng nhập"
    Then Cổng Thuế từ chối xác thực
    And Hệ thống phản hồi mã lỗi "07"
    And Màn hình hiển thị thông báo "Tên đăng nhập hoặc mật khẩu không đúng"
```

### TAXACC-AC-003: Kích hoạt đồng bộ thủ công kéo đúng số hóa đơn mới (TAXACC-FR-003)
```gherkin
Feature: Kích hoạt đồng bộ hóa đơn thủ công theo khoảng thời gian
  As a Kế toán viên (ACT-001)
  I want to Bấm nút đồng bộ để kéo hóa đơn phát sinh trong kỳ
  So that Dữ liệu hóa đơn trong mSMI được cập nhật tức thì

  Scenario: Đồng bộ thủ công thành công hóa đơn mua vào trong tháng
    Given Tài khoản thuế "0106026495" đang ở trạng thái hoạt động bình thường
    When Kế toán chọn đồng bộ hóa đơn "Mua vào" từ "01/03/2026" đến "31/03/2026"
    And Kế toán nhấn nút "Đồng bộ ngay"
    Then Hệ thống gửi yêu cầu cào dữ liệu qua RabbitMQ worker
    And Thanh tiến trình hiển thị hoàn tất 100%
    And Hệ thống thông báo "Đã đồng bộ thành công: 25 hóa đơn mới"
```

### TAXACC-AC-004: Ghi nhận đầy đủ thông số phiên chạy vào lịch sử đồng bộ (TAXACC-FR-004)
```gherkin
Feature: Ghi nhận lịch sử audit log đồng bộ CQT
  As a Quản trị viên hệ thống (ACT-001)
  I want to Xem lại nhật ký chi tiết các phiên cào dữ liệu
  So that Tôi kiểm soát tiến độ và khắc phục sự cố khi có lỗi

  Scenario: Kiểm tra bản ghi nhật ký sau khi phiên đồng bộ kết thúc
    Given Một phiên đồng bộ tự động vừa hoàn tất lúc 02:15:30
    When Quản trị viên truy cập tab "Lịch sử đồng bộ"
    Then Hệ thống hiển thị bản ghi mới nhất với trạng thái "Thành công"
    And Bản ghi thể hiện rõ: thời gian bắt đầu, thời gian kết thúc, thời lượng chạy (duration_ms)
    And Số lượng hóa đơn cào được khớp chính xác với số bản ghi thực tế
```
