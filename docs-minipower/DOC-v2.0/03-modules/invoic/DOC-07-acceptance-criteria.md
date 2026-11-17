# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Quản lý Hóa đơn (INVOIC)

| Phiên bản | Ngày      | Tác giả | Trạng thái          |
| ----------- | ---------- | --------- | --------------------- |
| 2.0         | 2026-11-08 | BA        | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** Gherkin BDD (Given / When / Then)
> **Phân hệ:** `INVOIC` — Quản lý Hóa đơn Mua vào / Bán ra

---

## 1. Danh mục Tiêu chí Chấp nhận

|          AC ID          |     FR ID     | Tên Kịch bản Nghiệm thu                                        | Mức ưu tiên |
| :---------------------: | :-----------: | ------------------------------------------------------------------ | :------------: |
| **INVOIC-AC-001** | INVOIC-FR-001 | Lọc danh sách hóa đơn theo khoảng ngày và loại hóa đơn |      Must      |
| **INVOIC-AC-002** | INVOIC-FR-003 | Tải hàng loạt hóa đơn định dạng ZIP thành công          |      Must      |
| **INVOIC-AC-003** | INVOIC-FR-005 | Gán ngày kê khai thuế và đối soát số liệu lên Tờ khai  |      Must      |
| **INVOIC-AC-004** | INVOIC-FR-005 | Hủy kê khai đưa trạng thái hóa đơn về Chưa kê khai     |      Must      |
| **INVOIC-AC-005** | INVOIC-FR-004 | Gán nhãn phân loại cho nhiều hóa đơn cùng lúc            |     Should     |

---

## 2. Kịch bản Gherkin Chi tiết

### INVOIC-AC-001: Lọc danh sách hóa đơn theo khoảng ngày và loại hóa đơn (INVOIC-FR-001)

```gherkin
Feature: Lọc danh sách hóa đơn theo nhiều tiêu chí
  As a Kế toán viên (ACT-001)
  I want to Lọc hóa đơn theo khoảng thời gian và phân loại
  So that Tôi có thể tìm kiếm nhanh chứng từ cần đối soát

  Scenario: Lọc thành công hóa đơn máy tính tiền mua vào trong tháng
    Given Kế toán đang ở phân hệ Hóa đơn Mua vào
    When Kế toán chọn loại hóa đơn là "Máy tính tiền (SCO)"
    And Kế toán chọn khoảng ngày lập từ "01/03/2026" đến "31/03/2026"
    And Kế toán bấm nút "Tìm kiếm"
    Then Hệ thống chỉ hiển thị các hóa đơn có type = "purchase_sco"
    And Tất cả hóa đơn trả về đều có ngày tdlap nằm trong tháng 03/2026
```

### INVOIC-AC-002: Tải hàng loạt hóa đơn dạng ZIP (INVOIC-FR-003)

```gherkin
Feature: Tải hàng loạt hóa đơn về máy tính
  As a Kế toán viên (ACT-001)
  I want to Chọn nhiều hóa đơn và tải về cùng lúc
  So that Tôi có thể lưu trữ hồ sơ giải trình quyết toán thuế nhanh chóng

  Scenario: Tải hàng loạt thành công cả file XML và PDF
    Given Kế toán đang ở màn hình Danh sách hóa đơn mua vào
    And Kế toán đã lọc ra danh sách hóa đơn của tháng 03/2026
    When Kế toán tích chọn 50 hóa đơn
    And Kế toán bấm nút "Tải hóa đơn" và chọn định dạng "Cả XML và PDF"
    Then Hệ thống thực hiện nén và tải về file zip thành công
    And File zip giải nén ra chứa đúng 50 file XML và 50 file PDF tương ứng
```

### INVOIC-AC-003: Gán ngày kê khai thuế (INVOIC-FR-005)

```gherkin
Feature: Gán ngày kê khai thuế cho hóa đơn
  As a Kế toán viên (ACT-001)
  I want to Gán mốc ngày kê khai cho các hóa đơn hợp lệ
  So that Hóa đơn tự động được tổng hợp vào Tờ khai thuế GTGT của kỳ đó

  Scenario: Gán ngày kê khai thành công
    Given Hóa đơn số 0001234 đang có trạng thái "Chưa kê khai" (nkekhai = null)
    When Kế toán chọn hóa đơn và gán ngày kê khai là "2026-03-31"
    Then Trường nkekhai của hóa đơn được cập nhật giá trị "2026-03-31"
    And Trạng thái hiển thị trên bảng đổi thành "Đã kê khai"
    And Giá trị tiền thuế của hóa đơn này được cộng dồn vào Tờ khai Mẫu 01/GTGT Quý 1/2026
```

### INVOIC-AC-004: Hủy ngày kê khai thuế (INVOIC-FR-005)

```gherkin
Feature: Hủy ngày kê khai thuế đã gán cho hóa đơn
  As a Kế toán viên (ACT-001)
  I want to Hủy kê khai một tờ hóa đơn đã gán nhầm
  So that Số liệu tờ khai thuế GTGT được cập nhật chính xác

  Scenario: Hủy kê khai thành công
    Given Hóa đơn số 0001234 đang có trạng thái "Đã kê khai" với nkekhai = "2026-03-31"
    When Kế toán chọn hóa đơn và bấm nút "Hủy kê khai"
    Then Trường nkekhai của hóa đơn được đặt về giá trị null
    And Trạng thái hiển thị trên bảng đổi về "Chưa kê khai"
    And Giá trị tiền thuế của hóa đơn này lập tức bị trừ khỏi Tờ khai 01/GTGT Quý 1/2026
```

### INVOIC-AC-005: Gán nhãn phân loại cho nhiều hóa đơn cùng lúc (INVOIC-FR-004)

```gherkin
Feature: Gán nhãn phân loại chi phí hàng loạt
  As a Kế toán viên (ACT-001)
  I want to Gán nhãn nội bộ cho nhiều hóa đơn trong một thao tác
  So that Tôi có thể phân nhóm chi phí dự án nhanh chóng

  Scenario: Gán nhãn thành công cho danh sách hóa đơn
    Given Kế toán đã tạo nhãn "Chi phí Dự án Alpha" với màu xanh lục
    When Kế toán tích chọn 10 hóa đơn trên danh sách
    And Kế toán chọn gán nhãn "Chi phí Dự án Alpha"
    Then Mảng tags của cả 10 hóa đơn đều được bổ sung ObjectId của nhãn Dự án Alpha
    And Giao diện bảng hiển thị huy hiệu nhãn màu xanh lục trên 10 hóa đơn đó
```
