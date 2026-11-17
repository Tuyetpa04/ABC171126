# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Kiểm soát Rủi ro Thuế (RISK)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** Gherkin BDD (Given / When / Then)  
> **Phân hệ:** `RISK` — Kiểm soát & Cảnh báo Rủi ro Thuế

---

## 1. Danh mục Tiêu chí Chấp nhận

| AC ID | FR ID | Tên Kịch bản Nghiệm thu | Mức ưu tiên |
|:---:|:---:|---|:---:|
| **RISK-AC-001** | RISK-FR-002 | Bật cờ cảnh báo đỏ khi người bán thuộc Danh sách 1.500 DN rủi ro cao | Must |
| **RISK-AC-002** | RISK-FR-001 | Cảnh báo khi người bán đang trong tình trạng Tạm ngừng kinh doanh | Must |
| **RISK-AC-003** | RISK-FR-003 | Phát hiện và cảnh báo hóa đơn mua vào bị bên bán Hủy trên Cổng Thuế | Must |
| **RISK-AC-004** | RISK-FR-005 | Tra cứu MST trả về đầy đủ thông tin pháp lý từ Thuế | Must |

---

## 2. Kịch bản Gherkin Chi tiết

### RISK-AC-001: Cảnh báo người bán thuộc Danh mục 1.500 DN (RISK-FR-002)
```gherkin
Feature: Cảnh báo doanh nghiệp người bán thuộc Blacklist 1.500 DN rủi ro cao
  As a Kế toán viên (ACT-001)
  I want to Hệ thống tự động báo động đỏ khi hóa đơn đến từ nhà cung cấp rủi ro
  So that Tôi dừng thanh toán và không kê khai khấu trừ thuế cho hóa đơn này

  Scenario: Hóa đơn mua vào có MST trùng với danh mục rủi ro cao
    Given Trong hệ thống có danh mục 1.500 DN rủi ro chứa MST "0101234567"
    When Hệ thống đồng bộ một hóa đơn mua vào mới có MST người bán là "0101234567"
    Then Hệ thống tự động gắn trạng thái cảnh báo rủi ro cao
    And Trên màn hình danh sách hóa đơn xuất hiện biểu tượng cờ đỏ cảnh báo
    And Tooltip hiển thị nội dung "Người bán nằm trong Danh mục rủi ro cao về thuế"
```

### RISK-AC-002: Cảnh báo khi người bán đang trong tình trạng Tạm ngừng kinh doanh (RISK-FR-001)
```gherkin
Feature: Cảnh báo người bán có trạng thái pháp lý bất thường
  As a Kế toán viên (ACT-001)
  I want to Nhận được cảnh báo khi hóa đơn lập trong giai đoạn đối tác tạm ngừng KD
  So that Tôi xác minh tính hợp pháp của giao dịch trước khi thanh toán

  Scenario: Ngày lập hóa đơn nằm trong khoảng thời gian tạm ngừng kinh doanh
    Given Doanh nghiệp người bán "0312345678" có trạng thái tạm ngừng KD từ 01/01/2026 đến 30/06/2026
    When Hệ thống tiếp nhận hóa đơn mua vào có ngày lập là "15/03/2026" từ người bán này
    Then Hệ thống kích hoạt cờ cảnh báo rủi ro màu cam
    And Ghi nhận thông điệp cảnh báo "Hóa đơn lập trong thời gian doanh nghiệp người bán đang tạm ngừng KD"
```

### RISK-AC-003: Cảnh báo hóa đơn bị HỦY sau khi xuất (RISK-FR-003)
```gherkin
Feature: Cảnh báo hóa đơn mua vào bị bên bán Hủy trên Cổng Thuế
  As a Kế toán viên (ACT-001)
  I want to Biết ngay khi một hóa đơn mua vào bị hủy bỏ
  So that Tôi không kê khai hóa đơn rác vào kỳ tính thuế

  Scenario: Bên bán thực hiện hủy hóa đơn trên Cổng Thuế
    Given Hóa đơn số 0005678 đã được đồng bộ vào mSMI trước đó với trạng thái Gốc
    When Phiên đồng bộ CQT định kỳ phát hiện trạng thái hóa đơn trên Cổng Thuế đổi sang "Bị hủy" (tthai = 4)
    Then Hệ thống cập nhật trạng thái hóa đơn trong mSMI thành "Hóa đơn đã bị hủy"
    And Hiển thị cờ cảnh báo trên dòng hóa đơn
    And Hóa đơn này tự động xuất hiện trong Báo cáo "Hóa đơn có biến động gần đây"
```

### RISK-AC-004: Tra cứu MST trả về đầy đủ thông tin pháp lý từ Thuế (RISK-FR-005)
```gherkin
Feature: Tra cứu trực tuyến thông tin mã số thuế doanh nghiệp
  As a Kế toán viên (ACT-001)
  I want to Nhập mã số thuế để xem tình trạng hoạt động thực tế
  So that Tôi kiểm tra tính chính xác của đối tác trước khi lập hợp đồng

  Scenario: Tra cứu thành công một mã số thuế đang hoạt động
    Given Kế toán đang ở màn hình Tra cứu MST doanh nghiệp
    When Kế toán nhập mã số thuế "0100109106" và nhấn nút "Kiểm tra MST"
    Then Hệ thống truy vấn CSDL Thuế và hiển thị chính xác tên doanh nghiệp, địa chỉ trụ sở
    And Hiển thị trạng thái hoạt động là "Đang hoạt động" kèm dấu tích xanh
```
