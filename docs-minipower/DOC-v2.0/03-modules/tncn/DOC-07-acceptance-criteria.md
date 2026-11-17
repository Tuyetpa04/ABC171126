# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Chứng từ Thuế TNCN (TNCN)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** BDD (Behavior-Driven Development) — Given / When / Then.  
> **Phân hệ:** `TNCN` — Quản lý Chứng từ Khấu trừ Thuế Thu nhập Cá nhân Điện tử.

---

## 1. Danh mục Kịch bản Nghiệm thu (Acceptance Criteria Register)

| Mã AC | Tên Kịch bản Nghiệm thu | Functional Requirement | Trọng yếu |
|:---:|---|:---:|:---:|
| **TNCN-AC-001** | Đồng bộ chứng từ TNCN từ Cổng Thuế thành công | `TNCN-FR-001` | Must |
| **TNCN-AC-002** | Tra cứu nhanh theo số CCCD hoặc MST cá nhân | `TNCN-FR-002` | Must |
| **TNCN-AC-003** | Bóc tách chính xác các chỉ tiêu thu nhập và tiền thuế khấu trừ | `TNCN-FR-003` | Must |
| **TNCN-AC-004** | Xuất bản in PDF chứng từ chuẩn Mẫu số 03/TNCN | `TNCN-FR-004` | Must |
| **TNCN-AC-005** | Cơ chế khử trùng chống lặp bản ghi khi cào lại | `TNCN-FR-001` | Must |

---

## 2. Đặc tả Chi tiết Từng Kịch bản Nghiệm thu

### TNCN-AC-001: Đồng bộ chứng từ TNCN thủ công
```gherkin
Kịch bản: Kế toán cào chứng từ thuế TNCN tháng 09/2026
  Giả sử Doanh nghiệp đã đăng nhập tài khoản CQT thành công
  Khi Kế toán mở hộp thoại "Đồng bộ Chứng từ TNCN"
  Và Chọn khoảng ngày từ "01/09/2026" đến "30/09/2026"
  Và Bấm "Xác nhận đồng bộ"
  Thì Hệ thống chia khoảng ngày thành các chunk tối đa 7 ngày
  Và Gửi yêu cầu qua Proxy/Bot tới Cổng Thuế
  Và Trả về danh sách các chứng từ phát sinh trong tháng 9
  Và Số lượng chứng từ mới phát hiện được cập nhật vào lịch sử đồng bộ
```

### TNCN-AC-002: Tra cứu theo số CCCD của người lao động
```gherkin
Kịch bản: Tìm chứng từ của nhân sự chưa có MST cá nhân
  Giả sử Hệ thống đã đồng bộ danh sách chứng từ của đơn vị
  Khi Kế toán nhập số Căn cước công dân "001095012345" vào ô tìm kiếm
  Thì Hệ thống lọc và trả về đúng các chứng từ cấp cho nhân viên có CCCD trên
  Và Hiển thị rõ: Họ và tên, Khoản thu nhập, Số tiền thuế đã khấu trừ
```

### TNCN-AC-003: Bóc tách chính xác số tiền thuế khấu trừ
```gherkin
Kịch bản: Đối soát tính toán chỉ tiêu tiền thuế
  Giả sử Chứng từ số "00000125" có tổng thu nhập chịu thuế là "50.000.000 VNĐ"
  Và Cá nhân thuộc diện khấu trừ 10% (thu nhập vãng lai)
  Khi Kế toán mở chi tiết chứng từ
  Thì Hệ thống hiển thị:
    | Chỉ tiêu | Số tiền |
    | Tổng thu nhập chịu thuế | 50.000.000 VNĐ |
    | Số thuế TNCN đã khấu trừ | 5.000.000 VNĐ |
    | Số tiền còn được nhận | 45.000.000 VNĐ |
  Và Tổng tiền thuế + Số tiền còn nhận bằng đúng Tổng thu nhập chịu thuế
```

### TNCN-AC-004: Xuất bản in PDF chứng từ Mẫu 03/TNCN
```gherkin
Kịch bản: Tải bản in PDF phục vụ quyết toán thuế cho cá nhân
  Giả sử Kế toán chọn một chứng từ bất kỳ trong danh sách
  Khi Kế toán bấm nút "Xem bản in" hoặc "Tải PDF"
  Thì Hệ thống hiển thị bản thể hiện đồ họa chuẩn Mẫu số 03/TNCN
  Và Thể hiện đầy đủ tên cơ quan thuế, tên tổ chức chi trả, thông tin cá nhân và chữ ký số
  Và Cho phép tải file PDF về máy tính để gửi cho người lao động
```

### TNCN-AC-005: Cơ chế khử trùng chống lặp bản ghi khi cào lại (TNCN-FR-001)
```gherkin
Kịch bản: Chống nhân bản trùng lặp chứng từ khi thực hiện quét lại
  Giả sử Chứng từ số "00000125" mẫu "03/TNCN" ký hiệu "C26TAA" đã tồn tại trong CSDL
  Khi Kế toán thực hiện đồng bộ lại khoảng thời gian có chứa chứng từ này
  Thì Hệ thống tính toán khóa duy nhất `${mst_to_chuc}_03/TNCN_C26TAA_125`
  Và Phát hiện bản ghi đã tồn tại trong bảng chungtutncns
  Và Hệ thống không tạo thêm bản ghi mới mà chỉ cập nhật thông tin nếu có thay đổi
  Và Tổng số lượng bản ghi trong cơ sở dữ liệu không bị tăng ảo
```


