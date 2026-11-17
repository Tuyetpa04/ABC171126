# Traceability Matrix (Ma trận Gióng hàng Yêu cầu & Kiến trúc Toàn diện v2.0)

| Cập nhật | 2026-11-08 | Phiên bản | 2.0 | Trạng thái | Master Baseline Unified v2.0 |
|:---:|:---:|:---:|:---:|:---:|:---:|

Single source of truth về mối quan hệ gióng hàng đa chiều giữa:
1. **Trục Nghiệp vụ Chức năng (Business/Functional):**
$$\text{Module} \longrightarrow \text{Use Case (UC)} \longrightarrow \text{Business Rule (BR)} \longrightarrow \text{Functional Requirement (FR)} \longrightarrow \text{Acceptance Criteria (AC)}$$
2. **Trục Kiến trúc Kỹ thuật & Vận hành (Architectural/Runtime):**
$$\text{NFR} \longleftrightarrow \text{ADR} \longleftrightarrow \text{Technical Container} \longleftrightarrow \text{Runtime Pipeline} \longleftrightarrow \text{Verification Scenario}$$

---

## 1. Phân hệ Kết nối Cơ quan Thuế (`taxacc`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `TAXACC-FR-001` | Kết nối & Quản lý Tài khoản CQT | `TAXACC-UC-001` | `TAXACC-BR-001`, `002` | `TAXACC-AC-001` | `03-modules/taxacc/DOC-06-srs.md` | Must | Baseline |
| `TAXACC-FR-002` | Tự động Đồng bộ Nền HĐĐT | `TAXACC-UC-003` | `TAXACC-BR-004` | `TAXACC-AC-002` | `03-modules/taxacc/DOC-06-srs.md` | Must | Baseline |
| `TAXACC-FR-003` | Kích hoạt Đồng bộ Thủ công | `TAXACC-UC-004` | `TAXACC-BR-003` | `TAXACC-AC-003` | `03-modules/taxacc/DOC-06-srs.md` | Must | Baseline |
| `TAXACC-FR-004` | Lịch sử & Báo cáo Lỗi Đồng bộ | `TAXACC-UC-005` | `TAXACC-BR-005` | `TAXACC-AC-004` | `03-modules/taxacc/DOC-06-srs.md` | Should | Baseline |
| `TAXACC-FR-005` | Cấu hình Lịch trình Quét Nền | `TAXACC-UC-002` | `TAXACC-BR-004` | `TAXACC-AC-002` | `03-modules/taxacc/DOC-06-srs.md` | Could | Baseline |

---

## 2. Phân hệ Quản lý & Thao tác Hóa đơn (`invoic`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `INVOIC-FR-001` | Bộ lọc & Tra cứu Danh sách Hóa đơn | `INVOIC-UC-001` | `INVOIC-BR-001`, `002` | `INVOIC-AC-001` | `03-modules/invoic/DOC-06-srs.md` | Must | Baseline |
| `INVOIC-FR-002` | Xem Bản thể hiện Đồ họa & XML | `INVOIC-UC-002`, `003` | `INVOIC-BR-001` | `INVOIC-AC-001` | `03-modules/invoic/DOC-06-srs.md` | Must | Baseline |
| `INVOIC-FR-003` | Tải File Đơn lẻ & Đóng gói ZIP Hàng loạt | `INVOIC-UC-004` | `INVOIC-BR-004`, `007` | `INVOIC-AC-002` | `03-modules/invoic/DOC-06-srs.md` | Must | Baseline |
| `INVOIC-FR-004` | Gán Nhãn Phân loại Chi phí (Tags) | `INVOIC-UC-005` | `INVOIC-BR-008` | `INVOIC-AC-005` | `03-modules/invoic/DOC-06-srs.md` | Should | Baseline |
| `INVOIC-FR-005` | Gán Kỳ Kê khai Thuế & Hủy Kê khai | `INVOIC-UC-006` | `INVOIC-BR-003` | `INVOIC-AC-003`, `004` | `03-modules/invoic/DOC-06-srs.md` | Must | Baseline |
| `INVOIC-FR-006` | Đính kèm Hồ sơ & Quản lý Ẩn Cảnh báo | `INVOIC-UC-007` | `INVOIC-BR-005`, `006` | `INVOIC-AC-004` | `03-modules/invoic/DOC-06-srs.md` | Could | Baseline |
| `INVOIC-FR-007` | Ghi chú Nội bộ trên Hóa đơn | `INVOIC-UC-008` | `INVOIC-BR-001` | `INVOIC-AC-001` | `03-modules/invoic/DOC-06-srs.md` | Could | Baseline |

---

## 3. Phân hệ Kiểm soát & Cảnh báo Rủi ro Thuế (`risk`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `RISK-FR-001` | Kiểm tra Trạng thái MST Người bán | `RISK-UC-001` | `RISK-BR-001`, `002` | `RISK-AC-001` | `03-modules/risk/DOC-06-srs.md` | Must | Baseline |
| `RISK-FR-002` | Đối soát Danh mục 1.500 DN Rủi ro cao | `RISK-UC-002` | `RISK-BR-003` | `RISK-AC-002` | `03-modules/risk/DOC-06-srs.md` | Must | Baseline |
| `RISK-FR-003` | Giám sát Chuỗi Hóa đơn Thay đổi / Hủy | `RISK-UC-003` | `RISK-BR-004` | `RISK-AC-003` | `03-modules/risk/DOC-06-srs.md` | Must | Baseline |
| `RISK-FR-004` | Xác thực Tính toàn vẹn XML & Chữ ký số | `RISK-UC-004` | `RISK-BR-005` | `RISK-AC-004` | `03-modules/risk/DOC-06-srs.md` | Must | Baseline |
| `RISK-FR-005` | Soi lệch Tên Pháp lý Doanh nghiệp | `RISK-UC-004` | `RISK-BR-006` | `RISK-AC-004` | `03-modules/risk/DOC-06-srs.md` | Should | Baseline |

---

## 4. Phân hệ Quản lý Hộp thư Hóa đơn (`mailbx`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `MAILBX-FR-001` | Cấu hình & Xác thực Hộp thư | `MAILBX-UC-001` | `MAILBX-BR-001` | `MAILBX-AC-001` | `03-modules/mailbx/DOC-06-srs.md` | Must | Baseline |
| `MAILBX-FR-002` | Quét Nền & Lọc Tệp Đính kèm | `MAILBX-UC-002` | `MAILBX-BR-002` | `MAILBX-AC-002`, `005` | `03-modules/mailbx/DOC-06-srs.md` | Must | Baseline |
| `MAILBX-FR-003` | Giải nén ZIP & Bóc tách XML | `MAILBX-UC-004` | `MAILBX-BR-003` | `MAILBX-AC-003` | `03-modules/mailbx/DOC-06-srs.md` | Must | Baseline |
| `MAILBX-FR-004` | Khử trùng & Đồng bộ Hóa đơn | `MAILBX-UC-004` | `MAILBX-BR-004` | `MAILBX-AC-004` | `03-modules/mailbx/DOC-06-srs.md` | Must | Baseline |
| `MAILBX-FR-005` | Quản lý Danh sách Thư & Tải File | `MAILBX-UC-003`, `005` | `MAILBX-BR-005` | `MAILBX-AC-002` | `03-modules/mailbx/DOC-06-srs.md` | Should | Baseline |

---

## 5. Phân hệ Quản lý Thông báo Sai sót (`tbss`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `TBSS-FR-001` | Tra cứu Danh sách Thông báo 04/SS | `TBSS-UC-001` | `TBSS-BR-001` | `TBSS-AC-001` | `03-modules/tbss/DOC-06-srs.md` | Must | Baseline |
| `TBSS-FR-002` | Hiển thị Bản thể hiện 04/SS | `TBSS-UC-002` | `TBSS-BR-002`, `004` | `TBSS-AC-002` | `03-modules/tbss/DOC-06-srs.md` | Must | Baseline |
| `TBSS-FR-003` | Tải Dữ liệu XML & File Thể hiện | `TBSS-UC-003` | `TBSS-BR-004` | `TBSS-AC-004` | `03-modules/tbss/DOC-06-srs.md` | Should | Baseline |
| `TBSS-FR-004` | Tự động Đồng bộ 04/SS từ CQT | `TBSS-UC-004` | `TBSS-BR-005` | `TBSS-AC-001` | `03-modules/tbss/DOC-06-srs.md` | Must | Baseline |
| `TBSS-FR-005` | Liên kết Hai chiều Hóa đơn - 04/SS | `TBSS-UC-005` | `TBSS-BR-003` | `TBSS-AC-003` | `03-modules/tbss/DOC-06-srs.md` | Must | Baseline |

---

## 6. Phân hệ Báo cáo Thuế & Kết xuất Kế toán (`report`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `REPORT-FR-001` | Bảng kê Hóa đơn Gom nhóm Thuế | `REPORT-UC-001` | `REPORT-BR-001`, `005` | `REPORT-AC-001` | `03-modules/report/DOC-06-srs.md` | Must | Baseline |
| `REPORT-FR-002` | Tính toán Tự động Tờ khai 01/GTGT | `REPORT-UC-002` | `REPORT-BR-002` | `REPORT-AC-002` | `03-modules/report/DOC-06-srs.md` | Must | Baseline |
| `REPORT-FR-003` | Snapshot & Kết xuất XML HTKK | `REPORT-UC-003` | `REPORT-BR-006` | `REPORT-AC-004` | `03-modules/report/DOC-06-srs.md` | Must | Baseline |
| `REPORT-FR-004` | Trình Cấu hình Xuất Excel Động | `REPORT-UC-004` | `REPORT-BR-003`, `004` | `REPORT-AC-003` | `03-modules/report/DOC-06-srs.md` | Should | Baseline |
| `REPORT-FR-005` | Biểu đồ & Báo cáo Giám sát Thuế | `REPORT-UC-005` | `REPORT-BR-001` | `REPORT-AC-001` | `03-modules/report/DOC-06-srs.md` | Could | Baseline |

---

## 7. Phân hệ Quản lý Thuế Điện tử (`mtax`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `MTAX-FR-001` | Xác thực & Kết nối Tài khoản eTax/DVC | `MTAX-UC-001` | `MTAX-BR-001`, `006` | `MTAX-AC-001` | `03-modules/mtax/DOC-06-srs.md` | Must | Baseline |
| `MTAX-FR-002` | Đồng bộ & Tra cứu Tờ khai Thuế | `MTAX-UC-002` | `MTAX-BR-002`, `003` | `MTAX-AC-002`, `003` | `03-modules/mtax/DOC-06-srs.md` | Must | Baseline |
| `MTAX-FR-003` | Đồng bộ & Quản lý Giấy nộp tiền NSNN | `MTAX-UC-003` | `MTAX-BR-002`, `004` | `MTAX-AC-004` | `03-modules/mtax/DOC-06-srs.md` | Must | Baseline |
| `MTAX-FR-004` | Đồng bộ & Quản lý Thông báo Thuế | `MTAX-UC-004` | `MTAX-BR-002`, `005` | `MTAX-AC-003` | `03-modules/mtax/DOC-06-srs.md` | Must | Baseline |
| `MTAX-FR-005` | Tải Tệp XML Gốc & Xuất Báo cáo Excel | `MTAX-UC-005` | `MTAX-BR-002`, `004` | `MTAX-AC-005` | `03-modules/mtax/DOC-06-srs.md` | Must | Baseline |
| `MTAX-FR-006` | Đồng bộ Danh mục Tờ khai Đăng ký | `MTAX-UC-002` | `MTAX-BR-002` | `MTAX-AC-002` | `03-modules/mtax/DOC-06-srs.md` | Should | Baseline |

---

## 8. Phân hệ Quản lý Chứng từ Khấu trừ Thuế TNCN (`tncn`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `TNCN-FR-001` | Đồng bộ Chứng từ TNCN Tự động từ CQT | `TNCN-UC-001` | `TNCN-BR-001`, `002` | `TNCN-AC-001` | `03-modules/tncn/DOC-06-srs.md` | Must | Baseline |
| `TNCN-FR-002` | Tra cứu & Bộ lọc Đa tiêu chí | `TNCN-UC-002` | `TNCN-BR-003` | `TNCN-AC-002` | `03-modules/tncn/DOC-06-srs.md` | Must | Baseline |
| `TNCN-FR-003` | Xem Chi tiết Khoản Thu nhập & Khấu trừ | `TNCN-UC-003` | `TNCN-BR-004` | `TNCN-AC-003` | `03-modules/tncn/DOC-06-srs.md` | Must | Baseline |
| `TNCN-FR-004` | Tải Tệp XML Gốc & Bản thể hiện PDF | `TNCN-UC-004` | `TNCN-BR-005` | `TNCN-AC-004` | `03-modules/tncn/DOC-06-srs.md` | Must | Baseline |
| `TNCN-FR-005` | Đối soát & Cảnh báo Sai lệch Định danh | `TNCN-UC-003` | `TNCN-BR-003` | `TNCN-AC-005` | `03-modules/tncn/DOC-06-srs.md` | Should | Baseline |

---

## 9. Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn Công khai (`kiemtrahd`)

| Mã FR | Tên Chức năng (FR) | UC liên quan | Quy tắc Nghiệp vụ (BR) | Tiêu chí Nghiệm thu (AC) | Đường dẫn DOC-06 | Mức ưu tiên | Trạng thái |
|---|---|---|---|---|---|:---:|:---:|
| `KTHD-FR-001` | Tra cứu Trạng thái HĐ Công khai qua CQT | `KTHD-UC-001` | `KTHD-BR-001`, `002` | `KTHD-AC-001`, `002` | `03-modules/kiemtrahd/DOC-06-srs.md` | Must | Baseline |
| `KTHD-FR-002` | Thẩm định Tính Toàn vẹn File XML HĐĐT | `KTHD-UC-002` | `KTHD-BR-003` | `KTHD-AC-003` | `03-modules/kiemtrahd/DOC-06-srs.md` | Must | Baseline |
| `KTHD-FR-003` | Trực quan hóa Bản thể hiện & In ấn Công khai | `KTHD-UC-003` | `KTHD-BR-004` | `KTHD-AC-004` | `03-modules/kiemtrahd/DOC-06-srs.md` | Must | Baseline |
| `KTHD-FR-004` | Đồng bộ Trạng thái CQT cho Hóa đơn Nội bộ | `KTHD-UC-004` | `KTHD-BR-005` | `KTHD-AC-005` | `03-modules/kiemtrahd/DOC-06-srs.md` | Should | Baseline |
| `KTHD-FR-005` | Tra cứu Định danh Doanh nghiệp Nguồn CQT | `KTHD-UC-001..002` | `KTHD-BR-003` | `KTHD-AC-001` | `03-modules/kiemtrahd/DOC-06-srs.md` | Must | Baseline |

---

## 10. Phân hệ Nền tảng & Tích hợp (`platform`)

| Mã Thực thể / API | Loại thành phần | Mục đích & Nghiệp vụ liên quan | Đường dẫn DOC | Mức ưu tiên | Trạng thái |
|---|:---:|---|---|:---:|:---:|
| `invoiceraws`, `attachments`, `tags`, `xmldatas` | Data Model | Lưu trữ lõi hóa đơn, XML và tệp đính kèm | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `externalaccounts`, `gmailaccounts` | Data Model | Tài khoản CQT và Hộp thư tiếp nhận | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `tokhaithues`, `reportconfigs` | Data Model | Tờ khai thuế GTGT và cấu hình xuất kế toán | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `chungtutncns` | Data Model | Dữ liệu chứng từ khấu trừ thuế TNCN điện tử | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `declarations`, `payments`, `notices` | Data Model | Tờ khai, giấy nộp tiền và thông báo thuế mTax | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `taxcodeclassificationappls` | Data Model | Danh mục 1.500 DN rủi ro cao theo công văn CQT | `04-platform/DOC-11-data-model.md` | Must | Baseline |
| `POST /create-tax-account`, `POST /edit-tax-account` | REST API | Kết nối & cập nhật tài khoản thuế qua API | `04-platform/DOC-12-api-specification.md` | Must | Baseline |
| `GET /invoices`, `GET /invoices/modified` | REST API | Lấy danh sách hóa đơn thường và hóa đơn thay đổi | `04-platform/DOC-12-api-specification.md` | Must | Baseline |
| `GET /invoices/:id/download/:file` | REST API | Tải file XML hóa đơn gốc và bản in PDF/HTML | `04-platform/DOC-12-api-specification.md` | Must | Baseline |
| `GET /invoices/:id/warning` | REST API | Chi tiết các chỉ số chấm điểm rủi ro XML hóa đơn | `04-platform/DOC-12-api-specification.md` | Must | Baseline |
| `GET /api/System/SearchTaxCodeV2` | REST API | Tra cứu tình trạng hoạt động MST từ Cơ quan thuế | `04-platform/DOC-12-api-specification.md` | Must | Baseline |

---

## 11. Ma trận Gióng hàng Kiến trúc Cốt lõi: NFR $\leftrightarrow$ ADR $\leftrightarrow$ Container $\leftrightarrow$ Pipeline $\leftrightarrow$ Kịch bản

| Mã NFR | Tiêu chí Yêu cầu | Quyết định Kiến trúc | Container / Thành phần Chịu trách nhiệm | Luồng Thực thi (Pipeline) | Kịch bản Xác thực | Chỉ số Đo lường & Nghiệm thu |
|:---:|---|:---:|---|:---:|:---:|---|
| **NFR-PERF-001** | Cào & phân tích $\ge$ 500.000 HĐ/ngày; trễ hàng đợi < 30s | **ADR-001**<br/>(RabbitMQ Event-driven)<br/>**ADR-005**<br/>(Hybrid .NET/Node) | `backend` (47 PM2 Apps),<br/>`crawl-etl-service`,<br/>`header-service`,<br/>`xml-service` | `PIPE-001`<br/>(Auto Crawl) | **SC-001**<br/>(Cao điểm 1.000 HĐ) | Thông lượng $\ge$ 500k HĐ/ngày;<br/>Prefetch queue = 20-30;<br/>Độ trễ trung bình < 15s |
| **NFR-PERF-002** | Độ trễ cào thủ công Portal < 10s cho hóa đơn đầu tiên | **ADR-004**<br/>(Split Header Queues) | `portal` (React),<br/>`backend` (`POST /account/:id/sync`),<br/>`signalr-service`,<br/>`minvoice-plugin` | `PIPE-002`<br/>(Manual Crawl) | **SC-001**<br/>(Cao điểm) | Queue `invoice-raw-header` ưu tiên cao;<br/>Thời gian bot nhận job < 2s;<br/>Kết quả đầu tiên < 10s |
| **NFR-PERF-003** | Tải hàng loạt PDF; tự động fallback Pug mẫu 1–6 khi thiếu file gốc | **ADR-005**<br/>(Node Worker Fallback) | `backend` (Worker thread trong `node_www`),<br/>Pug templates 1–6 | `PIPE-003-PDF`<br/>(Batch PDF Render) | **SC-004**<br/>(Fallback PDF) | Timeout 120s; in từng tờ; xuất `_bao_cao_loi.txt`; không sập batch tải |
| **NFR-RES-001** | Tự động đổi IP Proxy trong 3s khi gặp HTTP 403/429/Timeout | **ADR-002**<br/>(Proxy Forwarder Rotation) | `proxy-forwarder-service`,<br/>`proxy-request-api`,<br/>`minvoice-plugin` | `PIPE-001`,<br/>`PIPE-002` | **SC-002**<br/>(Cổng Thuế chặn IP) | Phát hiện lỗi và đổi proxy trong 3 giây;<br/>Tỷ lệ retry thành công $\ge$ 98% |
| **NFR-RES-002** | Dừng failover khi lỗi CQT, PRA nguồn duy nhất dựng message lỗi | **ADR-007**<br/>(Chuẩn hóa Message CQT) | `proxy-forwarder-service`,<br/>`proxy-request-api`,<br/>`backend`, `frontend` | `PIPE-002`,<br/>`PIPE-008` (Proxy Forwarding) | **SC-005**<br/>(Lỗi CQT đồng nhất) | 201 tests đạt; không leak stack trace; chặn bão retry 503/timeout |
| **NFR-STOR-001** | Dung lượng Mongo không tăng quá 20% do file nhị phân; query < 500ms | **ADR-003**<br/>(MinIO S3 Object Storage) | `xml-service`,<br/>`pdf-services`,<br/>MinIO Cluster (`minio-default`, `minio-202610`) | `PIPE-007`<br/>(XML & Storage) | **SC-003**<br/>(Khởi động lại worker) | 100% XML và PDF đẩy sang S3;<br/>MongoDB chỉ giữ đường dẫn S3 Key;<br/>Query hóa đơn < 300ms |
| **NFR-SEC-001** | Mã hóa 100% mật khẩu CQT, token phiên trong CSDL | **ADR-005**<br/>(Node BFF / Crypto) | `backend` (`login-account-cqt.js`),<br/>MongoDB `externalaccounts` | `PIPE-001`,<br/>`PIPE-002` | Audit & Penetration Test | Thuật toán mã hóa đối xứng 2 chiều;<br/>Không lộ Plaintext trong log/DB |
| **NFR-ISO-001** | Cô lập phân hệ Thuế (mTax), nghẽn thuế không ảnh hưởng HĐĐT | **ADR-001**<br/>(RabbitMQ Isolation) | `mtax-fe`,<br/>`mtax-api`,<br/>RabbitMQ Exchange `mtax_crawl`,<br/>MongoDB logical DB `mtax` | `PIPE-004`<br/>(mTax Scraping) | Kịch bản mTax tải cực đại | 0 giọt tin nào từ `mtax` lọt sang `crawl-invoice-raw`;<br/>Nghẽn tờ khai không làm chậm HĐĐT |
| **NFR-AVAIL-001**| Uptime API $\ge$ 99.5%; không tràn bộ nhớ RAM | **ADR-005**,<br/>**ADR-006**<br/>(Connection Pooling) | PM2 Cluster (47 tiến trình),<br/>`batch-job` (Hangfire),<br/>RabbitMQ HA Cluster | Toàn bộ luồng | **SC-003**<br/>(Worker restart) | Cấu hình trần 4GB Node.js;<br/>PM2 auto-restart khi crash;<br/>Không lỗi cạn kiệt RabbitMQ connection |

---

## 12. Ma trận Quyết định Kiến trúc & Vị trí Mã nguồn Triển khai

| Mã ADR | Tiêu đề Quyết định | Thư mục Mã nguồn Hiện hữu | Lớp Triển khai Cụ thể |
|:---:|---|---|---|
| **ADR-001** | Chuyển Poller sang RabbitMQ Event-Driven | `backend/consume/`, `crawl-etl-service/` | Lớp Queue Event Consumer & Producer (RabbitMQ Client) |
| **ADR-002** | Proxy Forwarder luân phiên IP đa tầng | `proxy-forwarder-service/`, `proxy-request-api/` | Bộ lọc điều hướng HttpWebRequest & Rotation Manager |
| **ADR-003** | Tách tệp nhị phân XML/PDF sang MinIO S3 | `xml-service/`, `pdf-services/` | Module tích hợp AWS S3 SDK & MinioZoneRegistryState |
| **ADR-004** | Phân tách 5 hàng đợi Header độc lập | `header-service/`, `backend/ecosystem.config.js` | Topic Routing Key & 5 Worker Consumers riêng biệt |
| **ADR-005** | Kiến trúc Lai .NET Worker + Node.js API | `backend/` kết hợp cụm service `.NET Core` | Tách rời BFF API (Node) và Compute/IO Worker (.NET) |
| **ADR-006** | Pool kết nối RabbitMQ dùng chung Batch Job | `batch-job/` | Singleton IConnection Factory & Channel Pooling trong Hangfire |
| **ADR-007** | Chuẩn hóa HTTP Status và Message lỗi CQT tại PRA | `proxy-request-api/`, `proxy-forwarder-service/`, `backend/`, `frontend/` | Bộ điều phối thông điệp hiển thị tập trung tại PRA, dừng failover PFS |

---

## 13. Ma trận Tích hợp Bên ngoài (External Integrations Traceability)

| Mã Giao tiếp | Đối tác Tích hợp | Phương thức Kỹ thuật | Container Tiếp nhận | Tài liệu Đặc tả |
|:---:|---|---|---|:---:|
| **INT-001** | Cổng HĐĐT Tổng cục Thuế | HTTP REST / JSON / XML via Proxy | `minvoice-plugin`, `crawl-data-api` | `DOC-10-integration-specification.md` §2.1 |
| **INT-002** | Cổng Thuế điện tử (mTax TCT) | HTTPS Web Scraper / Session Cookie | `mtax-api`, `minvoice-plugin` | `DOC-10-integration-specification.md` §2.2 |
| **INT-003** | Dịch vụ Giải mã Captcha (TwoCaptcha) | REST API (POST image / GET token) | `captcha`, `proxy-forwarder-service` | `DOC-10-integration-specification.md` §2.3 |
| **INT-004** | Nhà cung cấp Proxy xoay vòng | SOCKS5 / HTTP Proxy Auth | `proxy-forwarder-service` | `DOC-10-integration-specification.md` §2.4 |
| **INT-005** | Cổng Email (Gmail OAuth2 / IMAP) | IMAP SSL, Google API Client | `backend` (`service_email_invoice`) | `DOC-10-integration-specification.md` §2.5 |
| **INT-006** | Nhà cung cấp HĐĐT độc lập | REST API / SOAP XML | `backend` (`service_crawl_invoice_other`) | `DOC-10-integration-specification.md` §2.6 |
| **INT-007** | Hệ thống CRM & Phần mềm Kế toán | Inbound REST API (`apiToken`) | `backend` (`routes/`) | `DOC-10-integration-specification.md` §2.7 |

---

## 14. Ma trận Kiểm soát Rủi ro Kỹ thuật (Technical Debt & Risks)

| Mã Rủi ro | Nguy cơ Kỹ thuật | Giải pháp Kiến trúc Xử lý | Kiểm soát tại File Mã nguồn / Cấu hình |
|:---:|---|---|---|
| **RSK-001** | CQT đổi Captcha v3 / OTP | Cơ chế Fallback Desktop Bot + Alert | `minvoice-plugin/`, `signalr-service/` |
| **RSK-002** | Rò rỉ RAM trên tiến trình Node.js | Giới hạn 4GB RAM + Cron Restart PM2 | `backend/ecosystem.config.js` (`--max-old-space-size=4096`) |
| **RSK-003** | Mất gói tin RabbitMQ khi Worker restart | Requeue unacknowledged + DLQ Dead-letter | Cấu hình AckMode = Manual trong `.NET` & `amqplib` |

