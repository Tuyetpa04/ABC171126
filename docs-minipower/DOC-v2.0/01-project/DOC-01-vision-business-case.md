# DOC-01 — Tầm nhìn & Năng lực Sản phẩm (Vision & Business Case)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Căn cứ tài liệu gốc:** `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `mSMI - Tài liệu HDSD...docx`, kết hợp thực tế triển khai `mTax`, `TNCN`, `kiemtrahoadon.vn`.

---

## 1. Tuyên bố Tầm nhìn Sản phẩm (Product Vision)

**mSMI (Minvoice SME Invoice)** là nền tảng quản trị rủi ro hóa đơn, thuế điện tử và tự động hóa kế toán thuế toàn diện dành cho doanh nghiệp, công ty dịch vụ kế toán và người nộp thuế tại Việt Nam.

Phần mềm đóng vai trò như một **Trung tâm Tuân thủ & Trợ lý Thuế Thông minh (Tax & Compliance Hub)**, giải quyết triệt để 4 bài toán lớn:
1. **Thu thập dữ liệu tự động đa kênh**: Kết nối tự động Cổng HĐĐT Tổng cục Thuế, Cổng Thuế điện tử (`eTax`), Cổng Dịch vụ công (`DVC`), Hộp thư Email doanh nghiệp, và Kênh tiếp nhận chứng từ khấu trừ thuế TNCN điện tử.
2. **Kiểm soát & Giảm thiểu Rủi ro Thuế**: Tự động phát hiện hóa đơn của doanh nghiệp bỏ trốn, doanh nghiệp rủi ro cao (Blacklist 1.500 DN), hóa đơn bị hủy/thay thế lén lút, hóa đơn sai lệch thông tin hoặc chữ ký số không hợp lệ. Cung cấp cổng tra cứu công khai `kiemtrahoadon.vn` thẩm định tính pháp lý tức thì cho mọi đối tượng.
3. **Quản trị Thuế Điện tử Tập trung (mTax)**: Giám sát toàn bộ vòng đời hồ sơ khai thuế (tiếp nhận/chấp nhận), đối soát dòng tiền nộp ngân sách nhà nước (Giấy nộp tiền NSNN) và cảnh báo thông báo thuế kịp thời.
4. **Tự động hóa Nghiệp vụ Kế toán**: Tự động lập Tờ khai thuế GTGT 01/GTGT, lập bảng kê mua vào/bán ra phân nhóm thuế suất, tính toán khấu trừ thuế TNCN và kết xuất dữ liệu linh hoạt sang các phần mềm kế toán thông dụng (MISA, FAST, Bravo...).

---

## 2. Mục tiêu Kinh doanh & Giá trị Cốt lõi

* **Đối với Doanh nghiệp mua hàng (Bên Mua - Trọng tâm cốt lõi)**:
  - Loại trừ 100% nguy cơ vô tình hạch toán và khấu trừ thuế cho các hóa đơn bất hợp pháp, hóa đơn của doanh nghiệp ngừng hoạt động hoặc hóa đơn đã bị bên bán hủy trên Cổng Thuế.
  - Bảo vệ tối đa chi phí hợp lý của doanh nghiệp khi quyết toán thuế TNDN và hoàn thuế GTGT.
* **Đối với Doanh nghiệp bán hàng (Bên Bán)**:
  - Tự động đối soát dữ liệu xuất hóa đơn với hệ thống cấp mã của Cục Thuế.
  - Gom đủ số liệu đầu ra để phục vụ việc lập Tờ khai thuế GTGT và theo dõi thông báo sai sót 04/SS.
* **Đối với Bộ phận Nhân sự / Kế toán Tiền lương**:
  - Tự động hóa việc thu thập, lưu trữ và tra cứu chứng từ khấu trừ thuế TNCN điện tử (Mẫu 03/TNCN), hỗ trợ quyết toán thuế TNCN cuối năm minh bạch, chính xác.
* **Đối với Đại lý Thuế & Công ty Dịch vụ Kế toán**:
  - Tiết kiệm 80% - 90% thời gian gom chứng từ, nhập liệu thủ công cho hàng chục khách hàng.
  - Giám sát trạng thái tờ khai và giấy nộp tiền của hàng trăm doanh nghiệp trên cùng một nền tảng quản trị tập trung.
* **Đối với Cộng đồng & Khách vãng lai**:
  - Tiếp cận cổng tra cứu `kiemtrahoadon.vn` miễn phí, kiểm tra nhanh chóng tính toàn vẹn và chữ ký số file XML hóa đơn nhận được từ nhà cung cấp.

---

## 3. Bốn Trụ cột Năng lực Chức năng (Capability Pillars)

```text
┌──────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 mSMI CAPABILITY PILLARS v2.0                                 │
├──────────────────────────┬──────────────────────────┬──────────────────────────┬─────────────┤
│ 1. NGUỒN DỮ LIỆU ĐA KÊNH │ 2. KIỂM SOÁT RỦI RO      │ 3. THUẾ ĐIỆN TỬ & TNCN   │ 4. TÍCH HỢP │
│ - Cổng HĐĐT TCT          │ - Soi trạng thái MST     │ - Tờ khai thuế (DVC/eTax)│ - Tờ khai 01│
│ - Cổng eTax & DVC        │ - Blacklist 1.500 DN     │ - Giấy nộp tiền NSNN     │ - Bảng kê   │
│ - Chứng từ TNCN CQT      │ - Cảnh báo HĐ Hủy/Sửa    │ - Thông báo thuế bước 1/2│ - Xuất PMKT │
│ - Hộp thư Email          │ - Soi chữ ký số & tên    │ - Chứng từ TNCN Mẫu 03   │ - API tĩnh  │
│ - Upload XML kiemtrahd   │ - Thẩm định XML độc lập  │ - Tra cứu CQT công khai  │ - MinIO S3  │
└──────────────────────────┴──────────────────────────┴──────────────────────────┴─────────────┘
```
