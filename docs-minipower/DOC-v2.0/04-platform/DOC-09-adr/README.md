# DOC-09 — Sổ bộ Quyết định Kiến trúc Hệ thống (ADR Register)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | Solution Architect | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** **Michael Nygard ADR Standard** (Context, Decision, Status, Consequences).  
> **Căn cứ đối chiếu:** Mã nguồn và các quyết định kiến trúc tại `documents/adr/` và `documents/c4/04-code/`.

---

## Danh mục Quyết định Kiến trúc (ADR Catalog)

| Mã ADR | Tiêu đề Quyết định | Ngày ban hành | Trạng thái | Phân hệ ảnh hưởng |
|:---:|---|:---:|:---:|:---:|
| **[ADR-001](./ADR-001-rabbitmq-event-driven.md)** | Chuyển dịch từ Cron Poller sang RabbitMQ Event-Driven | 2026-08-15 | `Accepted` | Backend, ETL, Workers |
| **[ADR-002](./ADR-002-proxy-forwarder-rotation.md)** | Sử dụng Proxy Forwarder luân phiên IP chống chặn CQT | 2026-08-20 | `Accepted` | Proxy Service, Crawl Bot |
| **[ADR-003](./ADR-003-minio-object-storage.md)** | Tách lưu trữ tệp XML và PDF sang MinIO S3 Object Storage | 2026-08-25 | `Accepted` | XML/PDF Services, Mongo |
| **[ADR-004](./ADR-004-split-header-queues.md)** | Tách riêng 5 hàng đợi Header độc lập theo loại chu kỳ job | 2026-08-27 | `Accepted` | Header Service, Consumers |
| **[ADR-005](./ADR-005-hybrid-dotnet-node.md)** | Áp dụng Kiến trúc Lai: .NET Core cho Workers & Node.js cho API | 2026-09-01 | `Accepted` | Toàn hệ thống Platform |
| **[ADR-006](./ADR-006-batch-job-shared-connection.md)** | Tối ưu sử dụng chung kết nối RabbitMQ trong Batch Job | 2026-10-15 | `Accepted` | Batch Job, RabbitMQ |
| **[ADR-007](./ADR-007-chuan-hoa-http-status-message-cqt.md)** | Chuẩn hóa HTTP Status và Tập trung Dựng Message Lỗi CQT tại PRA | 2026-11-04 | `Accepted` | PFS, PRA, Backend API, Frontend |

