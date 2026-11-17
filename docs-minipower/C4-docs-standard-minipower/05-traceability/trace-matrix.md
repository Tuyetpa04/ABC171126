# Ma trận Gióng hàng Kiến trúc (Architectural Traceability Matrix)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-07 | Solution Architect | Master Baseline (Cover 100% C4) |

---

## 1. Mục đích & Nguyên tắc Gióng hàng (Traceability Scope)

Ma trận Gióng hàng Kiến trúc (Architectural Traceability Matrix) liên kết đa chiều giữa:
1. **Yêu cầu Phi chức năng (NFR)** — Định nghĩa chất lượng hệ thống tại `DOC-13-nfr.md`.
2. **Quyết định Kiến trúc (ADR)** — Giải pháp thiết kế được chấp thuận tại `DOC-09-adr/`.
3. **Container & Component (C2 / C3)** — Đơn vị mã nguồn và runtime trong `DOC-08-sad.md` & `DOC-17-deployment-guide.md`.
4. **Tiến trình Runtime (Pipelines)** — 8 luồng thực thi `PIPE-001` đến `PIPE-008` trong `DOC-08-sad.md`.
5. **Kịch bản Xác thực (+1 Scenarios)** — Phương án kiểm thử và chỉ số đánh giá `SC-001` đến `SC-005`.

---

## 2. Ma trận Gióng hàng Cốt lõi: NFR $\leftrightarrow$ ADR $\leftrightarrow$ Container $\leftrightarrow$ Pipeline $\leftrightarrow$ Kịch bản

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

## 3. Ma trận Quyết định Kiến trúc & Vị trí Mã nguồn Triển khai

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

## 4. Ma trận Tích hợp Bên ngoài (External Integrations Traceability)

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

## 5. Ma trận Kiểm soát Rủi ro Kỹ thuật (Technical Debt & Risks)

| Mã Rủi ro | Nguy cơ Kỹ thuật | Giải pháp Kiến trúc Xử lý | Kiểm soát tại File Mã nguồn / Cấu hình |
|:---:|---|---|---|
| **RSK-001** | CQT đổi Captcha v3 / OTP | Cơ chế Fallback Desktop Bot + Alert | `minvoice-plugin/`, `signalr-service/` |
| **RSK-002** | Rò rỉ RAM trên tiến trình Node.js | Giới hạn 4GB RAM + Cron Restart PM2 | `backend/ecosystem.config.js` (`--max-old-space-size=4096`) |
| **RSK-003** | Mất gói tin RabbitMQ khi Worker restart | Requeue unacknowledged + DLQ Dead-letter | Cấu hình AckMode = Manual trong `.NET` & `amqplib` |

