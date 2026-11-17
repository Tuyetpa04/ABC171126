# Ma trận Ánh xạ Tài liệu C4 sang Chuẩn MiniPower

| Cập nhật | 2026-11-08 | Phiên bản | 2.0 | Trạng thái | Master Baseline (Cover 100% C4) |
|:---:|:---:|:---:|:---:|:---:|:---:|

Tài liệu này đối chiếu chi tiết toàn bộ các tệp tài liệu kiến trúc trong `documents/c4/` và `documents/adr/` sang đúng cấu trúc hồ sơ kỹ thuật chuẩn MiniPower trong bộ hồ sơ `DOC-v2.0/`.

---

## 1. Ánh xạ Cấp độ C1 & C2 (Context & Containers)

| Tệp tài liệu nguồn gốc (`documents/c4/`) | Cấp độ C4 | Tài liệu chuẩn hóa MiniPower tương ứng | Phần mục cụ thể |
|---|:---:|---|---|
| `c4/01-system-context.md` | C1 | `04-platform/DOC-08-sad.md`<br>`04-platform/DOC-10-integration-specification.md` | §1.4 Tổng quan kiến trúc & §4.1 Sơ đồ Ngữ cảnh<br>Toàn bộ tài liệu tích hợp ngoài (INT-001..007) |
| `c4/02-container.md` | C2 | `04-platform/DOC-08-sad.md`<br>`04-platform/DOC-17-deployment-guide.md` | §4.1 Sơ đồ Container & Bảng ma trận 23 containers kỹ thuật<br>Toàn bộ cấu hình 47 PM2, RabbitMQ, MinIO Multi-Zone |
| `c4/lich-su-c1-truoc-doi-chieu-2026-11-02.md` | C1 | `00-governance/doc-versioning.md` | Lưu tham chiếu lịch sử phiên bản cũ (Deprecated) |

---

## 2. Ánh xạ Cấp độ C3 (23 Components / Services Catalog)

Theo nguyên tắc tinh gọn và chuẩn mực tài liệu kiến trúc MiniPower, tài liệu giải pháp tránh sao chép 1-1 hàng chục tệp component rời rạc. Toàn bộ các quy tắc nghiệp vụ kỹ thuật chuyên sâu (Business Logic, Validation Gates, Routing Queues, CAS Update, Multi-Zone Storage) của **23 Services Cốt lõi** từ `documents/c4/03-components/` được hợp nhất và chuẩn hóa trực tiếp vào **`04-platform/DOC-08-sad.md`** (§4.1.3 Bảng Ma trận Kiểm kê 23 Containers & §4.1.5 Đặc tả Quy tắc Nghiệp vụ Kỹ thuật Chuyên sâu 23 Services Cốt lõi):

| Tệp Component nguồn (`c4/03-components/`) | Container ID | Công nghệ chính | Vị trí Chuẩn hóa Đặc tả trong MiniPower |
|---|:---:|:---:|---|
| `03-components/msmi-portal.md` | `CNT-001` | React 18, Redux | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/mtax-fe.md` | `CNT-002` | React SPA | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/kiemtra-hd-portal.md` | `CNT-003` | React SPA | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/ops-dashboard.md` | `CNT-004` | React Admin | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/msmi-backend-api.md` | `CNT-005` | Node.js Express | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.3) |
| `03-components/msmi-backend-workers.md` | `CNT-006` | Node.js (29 PM2 apps) | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.4, §4.1.5.3) |
| `03-components/msmi-queue-consumers.md`| `CNT-007` | Node.js (16 PM2 apps) | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.4, §4.1.5.3) |
| `03-components/msmi-http-satellite.md` | `CNT-008` | Node.js Socket.IO | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.3) |
| `03-components/crawl-data-api.md` | `CNT-009` | ASP.NET Core 8 | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/proxy-request-api.md` | `CNT-010` | ASP.NET Core 8 | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/proxy-forwarder-service.md` | `CNT-011` | ASP.NET Core 9 | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/signalr-service.md` | `CNT-012` | ASP.NET Core 8 Hub | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/minvoice-plugin` | `CNT-013` | C# Desktop Bot | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/header-service.md` | `CNT-014` | .NET 6 Worker/API | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.1) |
| `03-components/xml-service.md` | `CNT-015` | .NET 6 Worker | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.1) |
| `03-components/pdf-services.md` | `CNT-016` | .NET 10 Worker | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.1) |
| `03-components/log-crawl-service.md` | `CNT-017` | .NET 6 Worker | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.1) |
| `03-components/crawl-etl-service.md` | `CNT-018` | .NET 6 Worker | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/certification-service.md`| `CNT-019` | .NET 6 Worker/API | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/captcha.md` | `CNT-020` | ASP.NET Core 8 Web API | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.2) |
| `03-components/batch-job.md` | `CNT-021` | .NET 8 Hangfire | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/mtax-api.md` | `CNT-022` | ASP.NET Core 8 Web API | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |
| `03-components/dashboard-api.md` | `CNT-023` | NestJS | `04-platform/DOC-08-sad.md` (§4.1.3, §4.1.5.4) |

> **Ghi chú về tệp legacy:** Tệp `documents/c4/03-components/legacy-backend.md` trong C4 cũ chỉ mang tính chất lưu vết lịch sử các đoạn script cào Node.js ban đầu. Nó không phải là một container/service độc lập của hệ thống hiện tại, và toàn bộ nội dung của nó đã được kế thừa vào đặc tả của `msmi-backend-workers` (`CNT-006`) và `msmi-queue-consumers` (`CNT-007`) trong `DOC-08-sad.md`.

---

## 3. Ánh xạ Cấp độ C4 Code & Pipelines

Theo chuẩn mực kiến trúc MiniPower (dựa trên Kruchten 4+1 Views), tài liệu kiến trúc cấp giải pháp tập trung vào các luồng nghiệp vụ runtime liên-container, không sao chép lại toàn bộ chi tiết mã nguồn cấp hàm (method level) để tránh nợ kỹ thuật và lỗi thời tài liệu:

| Nhóm Tài liệu C4 Code nguồn (`c4/04-code/`) | Bản chất Kỹ thuật | Chuẩn hóa Thay thế trong MiniPower |
|---|---|---|
| **Các Pipelines Runtime chính** (`pipeline-hddt-sync-full.md`, `pipeline-portal-manual-crawl.md`, `pipeline-mtax.md`, `pipeline-tbss.md`, `pipeline-chung-tu-tncn.md`, `pipeline-xml-sync.md`, `pipeline-proxy-crawl.md`...) | Luồng dữ liệu và sự kiện liên-container | **Hợp nhất & Chuẩn hóa toàn diện tại `04-platform/DOC-08-sad.md` (§4.2 Process/Runtime View)** với 8 Sequence Diagrams chuẩn GitLab và bảng đặc tả các bước thực thi. |
| **Đặc tả Chi tiết Hàm & Call Chain** (`msmi-backend-api.md`, `crawl-data-api.md`, `header-service.md`, `xml-service.md`...) | Tên hàm, tham số, call chain nội bộ trong 1 container | Bảo toàn trong **Mã nguồn thực tế (Codebase)**, tài liệu docstring/comments và Git history; ranh giới và trách nhiệm của từng service được quy định tại **`04-platform/DOC-08-sad.md` (§4.1.5)**. |
| **Biên bản Kỹ thuật & Báo cáo Sự cố** (`2026-08-...`, `2026-09-...`, `issue-...`) | Báo cáo sự cố tạm thời trong quá trình phát triển | Kế thừa các quyết định giải pháp bền vững vào **`04-platform/DOC-09-adr/`** (`ADR-001` đến `ADR-007`) và **`DOC-13-nfr.md`**. |

---

## 4. Ánh xạ Bộ Quyết định Kiến trúc (ADR)

| Tệp ADR nguồn (`documents/adr/`) | Mã ADR chuẩn MiniPower | Tiêu đề quyết định chuẩn hóa |
|---|:---:|---|
| Quyết định chuyển sang Message Broker | `ADR-001` | Chuyển dịch từ Cron Poller sang RabbitMQ Event-Driven |
| `2026-10-13-adr-bo-retry-http-500-proxy-forwarder.md` | `ADR-002` | Sử dụng Proxy Forwarder luân phiên IP chống chặn CQT |
| Quyết định tách file nhị phân khỏi Mongo | `ADR-003` | Tách lưu trữ tệp XML và PDF sang MinIO S3 Object Storage |
| `2026-08-27-split-invoice-raw-header-queue-plan.md` | `ADR-004` | Tách riêng 5 hàng đợi Header độc lập theo loại chu kỳ job |
| Quyết định kết hợp .NET và Node.js | `ADR-005` | Áp dụng Kiến trúc Lai: .NET Core cho Workers & Node.js cho API |
| `2026-10-15-adr-batch-job-dung-chung-connection-rabbitmq.md` | `ADR-006` | Tối ưu sử dụng chung kết nối RabbitMQ trong Batch Job |
| `2026-11-04-chuan-hoa-http-status-message-cqt.md` | `ADR-007` | Chuẩn hóa HTTP Status và Tập trung Dựng Message Lỗi CQT tại PRA |

---

## 5. Ánh xạ Báo cáo Rà soát, Hướng dẫn & Kiểm kê Mã nguồn Mới (05/11/2026)

| Tệp nguồn gốc mới trong `documents/` | Nội dung rà soát kỹ thuật | Ánh xạ chuẩn hóa MiniPower |
|---|---|---|
| `qa/2026-11-05-ra-soat-ba-thay-doi-c4.md` | Rà soát 3 thay đổi lớn: Tải nhiều PDF fallback Pug mẫu 1-6, persistence đồng bộ thủ công/repair, chuẩn hóa lỗi CQT | `04-platform/DOC-08-sad.md` (§4.2 PIPE-002, PIPE-003-PDF), `DOC-13-nfr.md` (§1 NFR-PERF-003, NFR-RES-002), `ADR-007` |
| `guides/architecture-rules.md` | 10 nguyên tắc kiến trúc: Cấm phụ thuộc ngược, chuẩn hóa hợp đồng API, bảo mật | `00-governance/doc-versioning.md`, `04-platform/DOC-08-sad.md` (§5) |
| `maintenance/source-audit/` (18 files: `anh-xa-c3-nguon-ma.md`, `anh-xa-c4-nguon-ma.md`, `kiem-ke-queue-crawl-etl.md`...) | Bảng kiểm kê và đối soát 1-1 giữa component/pipeline C4 với file code nguồn thực tế | `05-traceability/c4-to-minipower-mapping.md` (§2 & §3) + `05-traceability/trace-matrix.md` |

