# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Báo cáo Thuế & Kết xuất Kế toán (REPORT)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `REPORT` — Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất Kế toán Động  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 9), `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` (Mục 4), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Báo cáo, Tờ khai 01GTGT).

---

## 1. Danh sách Tiêu chí Nghiệm thu (Acceptance Criteria)

| Mã AC | Tiêu đề Kịch bản | Ca sử dụng liên quan | Yêu cầu liên quan |
|:---:|---|:---:|:---:|
| **REPORT-AC-001** | Lập Bảng kê Hóa đơn Mua vào phân nhóm thuế suất chính xác | REPORT-UC-001 | REPORT-FR-001 |
| **REPORT-AC-002** | Tự động tính toán các chỉ tiêu Tờ khai Thuế 01/GTGT | REPORT-UC-002 | REPORT-FR-002 |
| **REPORT-AC-003** | Cấu hình và xuất file Excel động theo mẫu MISA | REPORT-UC-004 | REPORT-FR-004 |
| **REPORT-AC-004** | Lưu snapshot tờ khai và xuất tệp XML tương thích HTKK | REPORT-UC-003 | REPORT-FR-003 |

---

## 2. Đặc tả Kịch bản Nghiệm thu (Gherkin Scenarios)

### REPORT-AC-001: Lập Bảng kê Hóa đơn Mua vào phân nhóm thuế suất chính xác

```gherkin
Kịch bản: Kết xuất Bảng kê Mua vào có nhiều mức thuế suất trong Quý 3
  Cho rằng Doanh nghiệp có 10 hóa đơn mua vào hợp lệ được gán kỳ kê khai "Quý 3/2026"
  Trong đó gồm: 2 hóa đơn thuế 5%, 5 hóa đơn thuế 10%, 2 hóa đơn thuế 8% và 1 hóa đơn Không chịu thuế
  Khi Kế toán viên chọn xem "Bảng kê Mua vào" cho kỳ "Quý 3/2026"
  Thì Bảng kê hiển thị phân chia thành 4 nhóm thuế suất riêng biệt
  Và Dòng tổng cộng tiền chưa thuế và tiền thuế của từng nhóm thuế suất khớp chính xác số liệu chi tiết
  Và Tổng cộng toàn bảng kê bằng tổng của tất cả các nhóm thuế suất cộng lại.
```

---

### REPORT-AC-002: Tự động tính toán các chỉ tiêu Tờ khai Thuế 01/GTGT

```gherkin
Kịch bản: Tự động tổng hợp số liệu điền lên Tờ khai 01/GTGT
  Cho rằng Kỳ tính thuế "Tháng 10/2026" có tổng tiền hàng mua vào là 100.000.000 đ, thuế đầu vào khấu trừ là 10.000.000 đ
  Và Doanh thu bán ra thuế suất 10% là 200.000.000 đ, thuế đầu ra là 20.000.000 đ
  Và Thuế GTGT chưa khấu trừ hết kỳ trước chuyển sang [21] là 2.000.000 đ
  Khi Kế toán viên mở màn hình "Tờ khai thuế 01/GTGT" cho kỳ "Tháng 10/2026"
  Thì Hệ thống tự động điền Chỉ tiêu [22] = 100.000.000 đ
  Và Hệ thống tự động điền Chỉ tiêu [23] = 10.000.000 đ, [25] = 10.000.000 đ
  Và Hệ thống tự động điền Chỉ tiêu [31] = 200.000.000 đ, [32] = 20.000.000 đ, [34] = 200.000.000 đ, [35] = 20.000.000 đ
  Và Hệ thống tự động tính Chỉ tiêu [40] (Số thuế phải nộp) = [35] - [25] - [21] = 8.000.000 đ.
```

---

### REPORT-AC-003: Cấu hình và xuất file Excel động theo mẫu MISA

```gherkin
Kịch bản: Xuất dữ liệu hóa đơn ra Excel theo mẫu import của phần mềm kế toán MISA
  Cho rằng Kế toán viên đã cấu hình cột "Số hóa đơn" có công thức calc(detail.shdon)
  Và Cột "Ngày hóa đơn" có công thức formatDate(detail.tdlap, 'DD/MM/YYYY')
  Và Cột "Mã nghiệp vụ" có công thức calc('MUA_HANG')
  Khi Kế toán viên bấm "Xuất Excel Kế toán" từ danh sách Hóa đơn mua vào
  Thì Hệ thống sinh ra tệp Excel có đúng 3 cột theo cấu hình đã lưu
  Và Giá trị ngày hiển thị đúng định dạng Ngày/Tháng/Năm (ví dụ 15/10/2026)
  Và Giá trị cột "Mã nghiệp vụ" được gán cố định chuỗi "MUA_HANG" cho tất cả các dòng.
```

---

### REPORT-AC-004: Lưu snapshot tờ khai và xuất tệp XML tương thích HTKK

```gherkin
Kịch bản: Khóa sổ tờ khai Quý 3 và xuất file XML nộp cơ quan thuế
  Cho rằng Kế toán viên đã kiểm tra số liệu Tờ khai 01/GTGT Quý 3/2026
  Khi Kế toán viên bấm nút "Lưu tờ khai"
  Thì Hệ thống tạo bản ghi snapshot trong CSDL, đánh dấu trạng thái "Đã chốt"
  Khi Kế toán viên bấm nút "Xuất XML (HTKK)"
  Thì Tệp XML được tải về máy tính
  Và Tệp XML này có thể mở trực tiếp và đọc thành công trên ứng dụng HTKK của Tổng cục Thuế mà không báo lỗi cú pháp.
```

