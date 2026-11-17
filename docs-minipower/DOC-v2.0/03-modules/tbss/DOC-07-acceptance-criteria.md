# DOC-07 — Tiêu chí Nghiệm thu Phân hệ Quản lý Thông báo Sai sót (TBSS)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TBSS` — Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6.16), `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Thông báo 04SS), `SMI_Sơ đồ nghiệp vụ.json` (Trang 5).

---

## 1. Danh sách Tiêu chí Nghiệm thu (Acceptance Criteria)

| Mã AC | Tiêu đề Kịch bản | Ca sử dụng liên quan | Yêu cầu liên quan |
|:---:|---|:---:|:---:|
| **TBSS-AC-001** | Lọc và phân loại danh sách Thông báo 04/SS | TBSS-UC-001 | TBSS-FR-001 |
| **TBSS-AC-002** | Xem bản thể hiện trực quan thông báo 04/SS | TBSS-UC-002 | TBSS-FR-002 |
| **TBSS-AC-003** | Tự động gắn cờ cảnh báo sai sót lên tờ hóa đơn gốc | TBSS-UC-005 | TBSS-FR-005 |
| **TBSS-AC-004** | Tải tệp XML và xuất PDF thông báo 04/SS | TBSS-UC-003 | TBSS-FR-003 |

---

## 2. Đặc tả Kịch bản Nghiệm thu (Gherkin Scenarios)

### TBSS-AC-001: Lọc và phân loại danh sách Thông báo 04/SS

```gherkin
Kịch bản: Tra cứu thông báo 04/SS phía mua vào trong tháng 10
  Cho rằng Kế toán viên đang ở màn hình Quản lý Thông báo sai sót 04/SS
  Khi Kế toán viên chọn tab "Thông báo Mua vào"
  Và Kế toán viên chọn khoảng ngày gửi từ "01/10/2026" đến "31/10/2026"
  Và Kế toán viên bấm nút "Tìm kiếm"
  Thì Hệ thống trả về danh sách các thông báo 04/SS do nhà cung cấp gửi lên CQT trong tháng 10
  Và Bảng kết quả hiển thị rõ: Số thông báo, Ngày thông báo, MST người bán, Số lượng hóa đơn sai sót, Trạng thái CQT.
```

---

### TBSS-AC-002: Xem bản thể hiện trực quan thông báo 04/SS

```gherkin
Kịch bản: Mở xem chi tiết bản thể hiện Mẫu 04/SS-HĐĐT
  Cho rằng Kế toán viên đang xem danh sách thông báo 04/SS
  Khi Kế toán viên bấm vào dòng thông báo số "TB-04SS/001"
  Thì Hệ thống mở màn hình xem chi tiết bản thể hiện Mẫu 04/SS-HĐĐT
  Và Bản thể hiện hiển thị đúng tiêu đề "THÔNG BÁO HÓA ĐƠN ĐIỆN TỬ CÓ SAI SÓT"
  Và Bảng chi tiết liệt kê đủ danh sách các hóa đơn có sai sót
  Và Tính chất sai sót hiển thị rõ ràng bằng chữ (Hủy, Điều chỉnh, Thay thế hoặc Giải trình)
  Và Cột lý do hiển thị đúng nội dung giải trình mà bên bán đã gửi lên CQT.
```

---

### TBSS-AC-003: Tự động gắn cờ cảnh báo sai sót lên tờ hóa đơn gốc

```gherkin
Kịch bản: Tự động liên kết và bật cờ cảnh báo khi phát hiện thông báo 04/SS
  Cho rằng Hóa đơn mua vào số 0000088 ký hiệu 1C24TBB của nhà cung cấp A đang ở trạng thái bình thường
  Khi Tiến trình đồng bộ kéo về 1 thông báo 04/SS trong đó nhà cung cấp A khai báo Hủy hóa đơn số 0000088
  Thì Hệ thống tự động liên kết thông báo 04/SS này với hóa đơn số 0000088 trong CSDL
  Và Trạng thái tờ hóa đơn số 0000088 trên màn hình danh sách tự động chuyển thành "Bị hủy theo 04/SS"
  Và Hệ thống hiển thị icon cảnh báo màu đỏ cạnh số hóa đơn
  Khi Kế toán viên bấm vào icon cảnh báo
  Thì Hệ thống chuyển hướng mở thẳng bản thể hiện thông báo 04/SS liên quan.
```

---

### TBSS-AC-004: Tải tệp XML và xuất PDF thông báo 04/SS

```gherkin
Kịch bản: Tải tệp XML gốc của thông báo 04/SS phục vụ thanh tra thuế
  Cho rằng Kế toán viên đang mở xem chi tiết một thông báo 04/SS
  Khi Kế toán viên bấm nút "Tải XML"
  Thì Hệ thống tải về máy tính file ".xml" nguyên bản thông điệp gửi CQT
  Và Cấu trúc XML chứa đầy đủ chữ ký số và dữ liệu các dòng hóa đơn sai sót
  Khi Kế toán viên bấm nút "Xuất PDF"
  Thì Hệ thống kết xuất bản thể hiện Mẫu 04/SS thành file PDF chuẩn trang in A4.
```

