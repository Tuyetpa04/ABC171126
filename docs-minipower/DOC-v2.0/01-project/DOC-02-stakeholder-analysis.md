# DOC-02 — Phân tích Tác nhân & Các bên liên quan (Stakeholder Analysis)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Căn cứ tài liệu gốc & mã nguồn:** `mSMI - Tài liệu HDSD...docx`, `kiemtrahoadon/`, `mtax-fe/`, `service/chung-tu-tncn/`.

---

## 1. Danh mục Tác nhân Người dùng & Hệ thống (Actors)

| Actor ID | Tên Tác nhân | Vai trò & Trách nhiệm trong hệ thống mSMI | Nhu cầu chính |
|:---:|---|---|---|
| **ACT-001** | **Kế toán viên** | Thao tác hàng ngày: Tra cứu hóa đơn, xem bản in, tải XML, gán nhãn chi phí, gán ngày kê khai thuế, đính kèm chứng từ, kiểm tra cảnh báo rủi ro, theo dõi tờ khai eTax và giấy nộp tiền, tra cứu chứng từ TNCN, kết xuất Excel sang PMKT. | Gom hóa đơn nhanh, không phải gõ tay, cảnh báo sớm hóa đơn bị hủy/sai sót, tự động hóa tờ khai. |
| **ACT-002** | **Kế toán trưởng / Chủ DN** | Giám sát tổng quan Dashboard: Tình hình mua/bán, rủi ro thuế doanh nghiệp, nghĩa vụ nộp ngân sách (Giấy nộp tiền), phê duyệt Tờ khai thuế GTGT 01/GTGT trước khi nộp CQT. | Báo cáo trực quan, an toàn pháp lý thuế, kiểm soát dòng tiền thuế và đối tác. |
| **ACT-003** | **Đại lý / Kế toán Dịch vụ** | Quản lý đa tài khoản khách hàng doanh nghiệp, gán nhân viên phụ trách từng MST, theo dõi tình trạng tuân thủ thuế của hàng chục công ty. | Quản lý tập trung 50-200 công ty trên cùng một nền tảng, phân quyền rành mạch. |
| **ACT-004** | **Quản trị viên (Admin)** | Cấu hình tham số hệ thống, tạo tài khoản đại lý, quản lý gói cước (Licenses), thiết lập mẫu bảng kê mặc định cho toàn hệ thống. | Quản trị vận hành ổn định, phân bổ hạn mức hóa đơn. |
| **ACT-005** | **Hệ thống Phần mềm ngoài** | Phần mềm kế toán (MISA, FAST), hệ thống ERP, CRM kết nối qua REST API sử dụng API Token tĩnh để tự động lấy hóa đơn và cảnh báo rủi ro. | Tích hợp dữ liệu tự động, liên tục, định dạng chuẩn. |
| **ACT-006** | **Cổng HĐĐT Tổng cục Thuế** | Hệ thống bên ngoài (`hoadondientu.gdt.gov.vn`) cung cấp dữ liệu hóa đơn gốc, thông báo sai sót 04/SS và thông tin trạng thái mã số thuế. | Nhận request tra cứu hợp lệ, phản hồi dữ liệu qua Captcha/Token. |
| **ACT-007** | **Khách vãng lai (Guest User)** | Tổ chức, cá nhân bên ngoài không có tài khoản mSMI, truy cập `kiemtrahoadon.vn` để tra cứu trạng thái hóa đơn trực tiếp từ CQT hoặc tải file XML lên để kiểm tra chữ ký số và tính hợp pháp. | Tra cứu tức thì, giao diện trực quan, không cần đăng nhập. |
| **ACT-008** | **Cổng Thuế điện tử & DVC** | Các cổng dịch vụ công thuế quốc gia (`thuedientu.gdt.gov.vn` và `dichvucong.gdt.gov.vn`) cung cấp tờ khai, thông báo thuế và giấy nộp tiền NSNN. | Tiếp nhận phiên xác thực định danh thuế và cung cấp dữ liệu hồ sơ. |

---

## 2. Ma trận Trách nhiệm Nghiệp vụ (RACI Matrix)

| Nhóm Nghiệp vụ | ACT-001 (Kế toán) | ACT-002 (Kế toán trưởng) | ACT-003 (Đại lý) | ACT-004 (Admin) | ACT-007 (Khách vãng lai) |
|---|:---:|:---:|:---:|:---:|:---:|
| Khai báo & Kết nối tài khoản CQT (HĐĐT) | **R** | **A** | **R** | **I** | — |
| Kết nối Hộp thư nhận hóa đơn (Email) | **R** | **A** | **R** | **I** | — |
| Xử lý hóa đơn (Gán nhãn, kê khai, đính kèm) | **R** | **A** | **R** | **I** | — |
| Kiểm tra & Xử lý cảnh báo rủi ro thuế | **R** | **A** | **R** | **I** | — |
| Xem & Tải thông báo sai sót 04/SS | **R** | **A** | **R** | **I** | — |
| Lập & Khóa sổ Tờ khai thuế 01/GTGT | **R** | **A** | **R** | **I** | — |
| Cấu hình mẫu xuất Excel sang PMKT | **R** | **A** | **R** | **A (Mẫu)** | — |
| Kết nối & Đồng bộ Tờ khai eTax/DVC (`mTax`) | **R** | **A** | **R** | **I** | — |
| Quản lý Giấy nộp tiền NSNN & Thông báo Thuế | **R** | **A** | **R** | **I** | — |
| Đồng bộ & Tra cứu Chứng từ TNCN (`TNCN`) | **R** | **A** | **R** | **I** | — |
| Tra cứu công khai & Kiểm tra XML (`kiemtrahd`) | **I** | **I** | **I** | **I** | **R** |
| Phân quyền nhân viên quản lý theo MST | **I** | **A** | **A** | **C** | — |
| Quản lý Bản quyền & Gói cước | **I** | **I** | **C** | **A** | — |

*Ghi chú: **R** - Responsible (Thực hiện); **A** - Accountable (Chịu trách nhiệm/Duyệt); **C** - Consulted (Tham vấn); **I** - Informed (Nhận thông tin).*
