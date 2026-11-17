# Business Glossary — Trạng thái & Thuật ngữ Hệ thống mSMI

| Version | Date | Author | Status |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

---

## 1. Trạng thái Tài liệu (DOC Status)

Áp dụng cho header của mọi tài liệu DOC và cột `Status` trong `doc-registry.md`:

| Trạng thái | Ý nghĩa | Khi nào áp dụng |
|---|---|---|
| **Draft** | Bản nháp — đang soạn thảo hoặc bổ sung nội dung | Khi tài liệu đang được phân tích hoặc chưa hoàn chỉnh |
| **Review** | Chờ rà soát — đã hoàn tất nội dung, chờ PO / R&D đối soát | Khi hoàn thành soạn thảo và chờ xác nhận |
| **Baseline** | Đã chốt chính thức — được phê duyệt làm mốc chuẩn hệ thống | Khi tài liệu đã được rà soát và đóng băng làm căn cứ phát triển |
| **Deprecated** | Lạc hậu / Không áp dụng nữa — chỉ lưu tham chiếu lịch sử | Áp dụng cho các sơ đồ/tài liệu phiên bản cũ đã bị thay thế |

---

## 2. Từ điển Thuật ngữ Nghiệp vụ mSMI

Toàn bộ các thuật ngữ và từ viết tắt xuất hiện trong hệ thống mSMI gồm 9 phân hệ:

| Thuật ngữ / Viết tắt | Tên đầy đủ / Định nghĩa nghiệp vụ | Ghi chú & Ý nghĩa trong mSMI |
|---|---|---|
| **mSMI** | Minvoice SME Invoice | Hệ thống quản lý hóa đơn điện tử, kiểm soát rủi ro và tuân thủ thuế toàn diện của Minvoice. |
| **CQT** | Cơ quan Thuế | Cơ quan thuế quản lý trực tiếp doanh nghiệp (Tổng cục Thuế, Cục Thuế, Chi cục Thuế). |
| **TCT** | Tổng cục Thuế | Cổng thông tin hóa đơn điện tử quốc gia (`hoadondientu.gdt.gov.vn`). |
| **HĐĐT** | Hóa đơn điện tử | Hóa đơn được lập, gửi, nhận, lưu trữ và quản lý bằng phương tiện điện tử theo NĐ 123 và TT 78. |
| **HĐ Mua vào (Purchase)** | Hóa đơn đầu vào | Hóa đơn do nhà cung cấp xuất cho doanh nghiệp, dùng để hạch toán chi phí và khấu trừ thuế GTGT. |
| **HĐ Bán ra (Sold)** | Hóa đơn đầu ra | Hóa đơn do doanh nghiệp xuất cho khách hàng, dùng để ghi nhận doanh thu và tính thuế GTGT phải nộp. |
| **SCO** | Hóa đơn từ máy tính tiền | Hóa đơn điện tử khởi tạo từ máy tính tiền có kết nối chuyển dữ liệu điện tử với cơ quan thuế. |
| **04/SS-HĐĐT** | Thông báo hóa đơn điện tử có sai sót | Mẫu biểu thông báo gửi cơ quan thuế khi phát hiện hóa đơn đã lập có sai sót (Hủy, Điều chỉnh, Thay thế, Giải trình). |
| **Tờ khai 01/GTGT** | Tờ khai thuế giá trị gia tăng | Mẫu tờ khai thuế GTGT theo phương pháp khấu trừ áp dụng cho người nộp thuế sản xuất kinh doanh thông thường. |
| **NNT** | Người nộp thuế | Doanh nghiệp, tổ chức hoặc cá nhân có nghĩa vụ nộp thuế theo quy định của pháp luật. |
| **MST** | Mã số thuế | Dãy số gồm 10 hoặc 13 chữ số được cơ quan thuế cấp cho người nộp thuế để quản lý thuế. |
| **Doanh nghiệp rủi ro cao** | Blacklist 1.500 DN | Danh mục các doanh nghiệp có rủi ro cao về hóa đơn do Tổng cục Thuế ban hành và cảnh báo toàn quốc. |
| **XML gốc** | Extensible Markup Language | Tệp dữ liệu chứa nội dung hóa đơn gốc có gắn chữ ký số của người bán và mã xác thực của cơ quan thuế (có giá trị pháp lý cao nhất). |
| **Bản thể hiện (PDF/HTML)** | Visual Representation | Bản hiển thị trực quan các chỉ tiêu nội dung hóa đơn để người dùng dễ đọc (mô phỏng hóa đơn giấy). |
| **Gán nhãn (Tag)** | Nhãn phân loại nội bộ | Thẻ màu gắn lên hóa đơn để phân loại mục đích sử dụng (chi phí tiếp khách, hàng dự án, chi phí hợp lý...). |
| **Gán kỳ kê khai** | Gắn mốc tính thuế | Thao tác đánh dấu hóa đơn thuộc kỳ kê khai thuế nào (Tháng/Quý) để tự động tổng hợp số liệu lên Tờ khai 01/GTGT. |
| **PMKT** | Phần mềm kế toán | Các ứng dụng kế toán doanh nghiệp bên ngoài (như MISA, FAST, Bravo, SmartKey...). |
| **mTax** | Hệ thống Thuế Điện tử & Dịch vụ công | Phân hệ kết nối Cổng Thuế điện tử (`thuedientu.gdt.gov.vn`) và Cổng DVC (`dichvucong.gdt.gov.vn`) để quản lý Tờ khai, Giấy nộp tiền và Thông báo thuế. |
| **eTax** | Thuế điện tử doanh nghiệp | Cổng kê khai và nộp thuế qua mạng của Tổng cục Thuế dành cho doanh nghiệp. |
| **DVC** | Cổng Dịch vụ công Quốc gia / TCT | Cổng dịch vụ công phục vụ nộp hồ sơ, thủ tục hành chính thuế và tra cứu thông báo. |
| **TNCN** | Thuế Thu nhập Cá nhân | Phân hệ quản lý và đồng bộ chứng từ khấu trừ thuế TNCN điện tử theo quy chuẩn mới. |
| **Mẫu 03/TNCN** | Chứng từ khấu trừ thuế TNCN điện tử | Mẫu biểu chứng từ điện tử quy định tại Nghị định 123/2020/NĐ-CP và Thông tư 78/2021/TT-BTC. |
| **KTHD / kiemtrahoadon.vn** | Cổng Kiểm tra Hóa đơn Công khai | Trang web độc lập cho phép người nhận hóa đơn tra cứu trạng thái CQT và thẩm định XML không cần đăng nhập. |
| **MinIO S3** | Object Storage lưu trữ tệp | Hạ tầng lưu trữ đối tượng phân tán tách biệt tệp nhị phân XML/PDF khỏi MongoDB nhằm tối ưu tốc độ truy vấn metadata. |
