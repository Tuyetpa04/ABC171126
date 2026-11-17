# DOC-17 — Hướng dẫn Triển khai & Cấu hình Hạ tầng Runtime (Deployment Guide)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | Solution Architect / DevOps Lead | Baseline Unified v2.0 |

> **Căn cứ tài liệu gốc:** `documents/c4/02-container.md`, `backend/ecosystem.config.js`, `documents/c4/03-components/`, `documents/adr/`.

---

## 1. Tổng quan Mô hình Triển khai Hạ tầng

Hệ thống mSMI bao gồm cụm máy chủ ứng dụng (Application Servers) điều phối 47 tiến trình PM2, cụm Microservices .NET Core, hệ thống Message Broker RabbitMQ, cơ sở dữ liệu MongoDB, bộ đệm Redis và lưu trữ đối tượng MinIO S3.

---

## 2. Cấu hình Cụm Tiến trình PM2 (Node.js Backend — 47 Apps)

Toàn bộ backend Node.js được quản lý thông qua tệp `backend/ecosystem.config.js` gồm **47 tiến trình độc lập**:

### 2.1. Nhóm Web Server & API Gateway (1 App)
| Tên App PM2 | Script thực thi | Port | Cấu hình Memory & Restart | Vai trò |
|---|---|:---:|---|---|
| `node_www` | `bin/www` | 3000 | `--max-old-space-size=4096`, cron restart 04:00 AM | Máy chủ API phục vụ Web Portal và CRM ngoài; chứa worker thread render fallback HTML mẫu 1–6 sang PDF khi tải hàng loạt |

### 2.2. Nhóm Consumers Lắng nghe RabbitMQ (16 Apps)
| STT | Tên App PM2 | Script thực thi | Queue lắng nghe | Prefetch QoS | Vai trò & Trách nhiệm |
|:---:|---|---|---|:---:|---|
| 1 | `consumer_invoice_header` | `consume/invoiceHeader.js` | `invoice-raw-header` | 30 | Ghi danh sách HĐ cào thủ công/mặc định vào MongoDB (CAS update) |
| 2 | `consumer_invoice_header_retry` | `consume/invoiceHeader.js` | `invoice-raw-header-retry` | 15 | Xử lý cào bù danh sách HĐ (retry / excel-log) |
| 3 | `consumer_invoice_header_auto_daily` | `consume/invoiceHeader.js` | `invoice-raw-header-auto-daily` | 20 | Ghi danh sách HĐ quét tự động hàng ngày (`daily`) |
| 4 | `consumer_invoice_header_auto_month` | `consume/invoiceHeader.js` | `invoice-raw-header-auto-month` | 15 | Ghi danh sách HĐ quét tự động hàng tháng (`month`) |
| 5 | `consumer_invoice_header_auto_year` | `consume/invoiceHeader.js` | `invoice-raw-header-auto-year` | 10 | Ghi danh sách HĐ quét tự động hàng năm (`year`) |
| 6 | `consumer_invoice_detail` | `consume/invoiceDetail.js` | `invoice-raw` | 20 | Bóc tách chi tiết mặt hàng/thuế, áp dụng CAS & Data Repair |
| 7 | `consumer_hdtbssres` | `consume/hdtbssresExcel.js` | `hdtbssres-excel` | 1 | Tiêu thụ bảng kê Excel thông báo sai sót 04/SS |
| 8 | `consumer_detail_hdtbssres` | `consume/detailHdtbssres.js` | `detail-hdtbssres` | 1 | Tiêu thụ chi tiết từng thông báo sai sót 04/SS |
| 9 | `consumer_purchase_hdtbssres` | `consume/purchaseHdtbssrses.js` | `purchase-hdtbssres` | 1 | Tiêu thụ thông báo sai sót hóa đơn mua vào |
| 10 | `consumer_sync_ttinlquan` | `consume/syncTtinlquan.js` | `sync-ttinlquan` | 1 | Tiêu thụ thông tin liên quan (HĐ điều chỉnh/thay thế) |
| 11 | `consumer_chung_tu_tncn_header` | `consume/chungTuTncnHeader.js` | `chung-tu-tncn-list` | 5 | Tiêu thụ danh sách chứng từ khấu trừ thuế TNCN thủ công |
| 12 | `consumer_chung_tu_tncn_header_auto_daily` | `consume/chungTuTncnHeader.js` | `chung-tu-tncn-list-auto-daily` | 20 | Tiêu thụ danh sách chứng từ TNCN quét hàng ngày |
| 13 | `consumer_chung_tu_tncn_header_auto_month` | `consume/chungTuTncnHeader.js` | `chung-tu-tncn-list-auto-month` | 15 | Tiêu thụ danh sách chứng từ TNCN quét hàng tháng |
| 14 | `consumer_chung_tu_tncn_header_auto_year` | `consume/chungTuTncnHeader.js` | `chung-tu-tncn-list-auto-year` | 10 | Tiêu thụ danh sách chứng từ TNCN quét hàng năm |
| 15 | `consumer_chung_tu_tncn_header_retry` | `consume/chungTuTncnHeader.js` | `chung-tu-tncn-list-retry` | 15 | Tiêu thụ danh sách chứng từ TNCN cào lại (retry) |
| 16 | `consumer_chung_tu_tncn_xml` | `consume/chungTuTncnXml.js` | `chung-tu-tncn-xml` | 3 | Tiêu thụ XML chứng từ TNCN, upload MinIO và lưu DB |

### 2.3. Nhóm Background Workers & Pollers (29 Apps)
| STT | Tên App PM2 | Script thực thi | Chu kỳ / Cron | Target Queue / Service | Vai trò & Trách nhiệm |
|:---:|---|---|---|---|---|
| 1 | `service_1_detail_purchase` | `service/service_1_detail_purchase.js` | 2.5s (`cron_restart: '0 * * * *'`) | `crawl-invoice-raw` | Quét HĐ mua vào thông thường thiếu detail để cào chi tiết |
| 2 | `service_1_detail_purchase_sco` | `service/service_1_detail_purchase_sco.js` | 2.5s (`cron_restart: '0 * * * *'`) | `crawl-invoice-raw` | Quét HĐ mua vào SCO máy tính tiền thiếu detail |
| 3 | `service_1_detail_sold` | `service/service_1_detail_sold.js` | 2.5s (`cron_restart: '0 * * * *'`) | `crawl-invoice-raw` | Quét HĐ bán ra thông thường thiếu detail |
| 4 | `service_1_detail_sold_sco` | `service/service_1_detail_sold_sco.js` | 2.5s (`cron_restart: '0 * * * *'`) | `crawl-invoice-raw` | Quét HĐ bán ra SCO máy tính tiền thiếu detail |
| 5 | `service_1_purchase` | `service/service_1_purchase.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét tài khoản mua vào đến hạn, tạo job cào Excel |
| 6 | `service_1_purchase_sco` | `service/service_1_purchase_sco.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét tài khoản mua vào SCO đến hạn, tạo job cào Excel |
| 7 | `service_1_sold` | `service/service_1_sold.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét tài khoản bán ra thông thường đến hạn |
| 8 | `service_1_sold_sco` | `service/service_1_sold_sco.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét tài khoản bán ra SCO đến hạn |
| 9 | `service_1_purchase_auto_poller` | `service/service_1_purchase_auto_poller.js` | 5s | Queue auto-sync nội bộ | Poller quét TK mua vào chạy lịch daily/month/year |
| 10 | `service_1_purchase_auto_worker` | `service/service_1_purchase_auto_worker.js` | Queue | `crawl-invoice-raw-excel-auto` | Worker đóng gói payload mua vào đẩy sang RabbitMQ |
| 11 | `service_1_purchase_sco_auto_poller` | `service/service_1_purchase_sco_auto_poller.js` | 5s | Queue auto-sync nội bộ | Poller quét TK mua vào SCO chạy lịch daily/month/year |
| 12 | `service_1_purchase_sco_auto_worker` | `service/service_1_purchase_sco_auto_worker.js` | Queue | `crawl-invoice-raw-excel-auto` | Worker đóng gói payload mua vào SCO |
| 13 | `service_1_sold_auto_poller` | `service/service_1_sold_auto_poller.js` | 5s | Queue auto-sync nội bộ | Poller quét TK bán ra chạy lịch daily/month/year |
| 14 | `service_1_sold_auto_worker` | `service/service_1_sold_auto_worker.js` | Queue | `crawl-invoice-raw-excel-auto` | Worker đóng gói payload bán ra |
| 15 | `service_1_sold_sco_auto_poller` | `service/service_1_sold_sco_auto_poller.js` | 5s | Queue auto-sync nội bộ | Poller quét TK bán ra SCO chạy lịch daily/month/year |
| 16 | `service_1_sold_sco_auto_worker` | `service/service_1_sold_sco_auto_worker.js` | Queue | `crawl-invoice-raw-excel-auto` | Worker đóng gói payload bán ra SCO |
| 17 | `service_1_purchase_log` | `service/service_1_purchase_log.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét và đồng bộ nhật ký cào HĐ mua vào |
| 18 | `service_1_purchase_sco_log` | `service/service_1_purchase_sco_log.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét và đồng bộ nhật ký cào HĐ mua vào SCO |
| 19 | `service_1_sold_log` | `service/service_1_sold_log.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét và đồng bộ nhật ký cào HĐ bán ra |
| 20 | `service_1_sold_sco_log` | `service/service_1_sold_sco_log.js` | 5s | `crawl-invoice-raw-excel-auto` | Quét và đồng bộ nhật ký cào HĐ bán ra SCO |
| 21 | `service_1_chung_tu_tncn` | `service/service_1_chung_tu_tncn.js` | 5s | `crawl-chung-tu-tncn*` | Quản lý đồng bộ chứng từ TNCN: danh sách, log, XML |
| 22 | `service_1_chung_tu_tncn_auto_poller` | `service/service_1_chung_tu_tncn_auto_poller.js` | 5s | Queue auto-sync nội bộ | Poller quét TK đến hạn đồng bộ chứng từ TNCN tự động |
| 23 | `service_1_chung_tu_tncn_auto_worker` | `service/service_1_chung_tu_tncn_auto_worker.js` | Queue | `crawl-chung-tu-tncn*` | Worker đóng gói job cào chứng từ TNCN |
| 24 | `service_token_health` | `service/service_token_health.js` | 5s (50 TK/lần) | Detonator $\rightarrow$ Reactor | Kiểm tra tính sống còn token JWT CQT, tự relogin |
| 25 | `service_tbss` | `service/service_tbss.js` | 6s (`cron_restart: '0 * * * *'`) | `crawl-hdtbssres*` | Quét thông báo sai sót 04/SS mua vào |
| 26 | `service_tbss_sold_explanation` | `service/service_tbss_sold_explanation.js` | 5s | `crawl-hdtbssres*` | Quét thông báo sai sót 04/SS bán ra (giải trình) |
| 27 | `service_tbss_sold_sco_explanation` | `service/service_tbss_sold_sco_explanation.js` | 7s | `crawl-hdtbssres*` | Quét thông báo sai sót 04/SS bán ra SCO (giải trình) |
| 28 | `service_2` | `service/service_2.js` | Đa chu kỳ | Socket.IO Reactor | Tiện ích hệ thống: auto-login CQT, email IMAP, interval |
| 29 | `service_3` | `service/service_3.js` | Đa chu kỳ | Cổng TCT / API ngoài | Quét và cập nhật trạng thái pháp lý MST doanh nghiệp |

### 2.4. Nhóm Satellite Egress Proxy (1 App)
| Tên App PM2 | Script thực thi | Protocol | Vai trò |
|---|---|:---:|---|
| `satellite` | `reactor/satellite.js` | Socket.IO Reactor | Egress Gateway trung chuyển kết nối ra ngoài và điều tiết lưu lượng |

---

## 3. Cấu hình Cụm Microservices .NET Core

Các service C# được triển khai dưới dạng Systemd Services hoặc Docker Containers:

| Microservice | Công nghệ | File cấu hình chính | Cổng Port / Giao thức |
|---|:---:|---|:---:|
| `crawl-etl-service` | .NET 8 Worker | `appsettings.json` | RabbitMQ Consumer |
| `header-service` | .NET 8 Worker | `appsettings.json` | RabbitMQ Consumer |
| `xml-service` | .NET 8 Worker | `appsettings.json` | RabbitMQ Consumer |
| `pdf-services` | .NET 10 Worker | `appsettings.json` | RabbitMQ Consumer |
| `proxy-forwarder-service` | ASP.NET Core 8 | `appsettings.json` | Port 5005 (HTTP/REST); `Forwarding.TimeoutSeconds=120`, `ConnectTimeout=20s`, `ProxyBatchSize=3` |
| `proxy-request-api` | ASP.NET Core 8 | `appsettings.json` | Port 5006 (HTTP/REST); Đích trỏ của biến môi trường `API_TRA_CUU` từ Backend API; Nguồn duy nhất dựng message lỗi CQT |
| `signalr-service` | ASP.NET Core 8 | `appsettings.json` | Port 5001 (WebSocket / REST) |
| `mtax-api` | ASP.NET Core 8 | `appsettings.json` | Port 5010 (HTTP/REST & RabbitMQ) |
| `batch-job` | .NET 8 + Hangfire | `appsettings.json` | Port 5008 (Hangfire Dashboard) |

---

## 4. Cấu hình Message Broker RabbitMQ

- **Exchange chính:**
  1. `crawl-invoice-raw`: Type `topic` (hoặc `direct`), `durable = true`. Dùng điều phối toàn bộ luồng HĐĐT.
  2. `mtax_crawl`: Type `direct`, `durable = true`. Dùng riêng cho phân hệ Thuế điện tử.
- **Quy tắc Binding Queues:**
  - `crawl-invoice-raw-excel-auto-*` $\rightarrow$ Nhận lệnh cào từ AutoWorker.
  - `invoice-raw-excel-auto` $\rightarrow$ Nhận kết quả bảng kê từ Bot/Proxy.
  - `invoice-raw-header-*` $\rightarrow$ 5 queue phân luồng danh sách hóa đơn.
  - `invoice-raw-pdf` & `response-invoice-raw-xml` $\rightarrow$ Nhận file nhị phân đẩy sang MinIO.

---

## 5. Cấu hình Lưu trữ & Cơ sở Dữ liệu

### 5.1. Cụm MongoDB
Hệ thống sử dụng MongoDB Replica Set kết nối qua 3 Named Connections:
1. `DB_MSMI_CONNECTION`: Kết nối database chính `HDDV` (lưu trữ `invoiceraws`, `attachments`, `tags`, `useraccounts`...).
2. `DB_CRAWL_DATA_CONNECTION`: Kết nối database Thuế điện tử `mtax`.
3. `STATS_CONNECTION`: Kết nối database phục vụ thống kê tổng hợp số liệu Dashboard.

### 5.2. MinIO S3 Object Storage
- **Endpoint:** `http://minio.internal:9000` (hoặc Cloud S3).
- **Buckets bắt buộc:**
  - `minio-default`: Chứa các hóa đơn lịch sử phát sinh trước tháng 10/2026.
  - `minio-202610`: Chứa hóa đơn phát sinh từ tháng 10/2026 trở đi (phân vùng theo chu kỳ).
- **Cấu hình Policy:** Bucket để chế độ Private, chỉ cho phép truy cập qua Pre-signed URL hoặc API Gateway có xác thực.

### 5.3. Redis Cluster
- **Cấu hình:** Port 6379, thiết lập `maxmemory-policy: allkeys-lru`.
- **Key Namespaces:**
  - `session:*`: Bộ đệm phiên làm việc người dùng.
  - `bot:available:*`: Trạng thái rảnh/bận của Bot Desktop SignalR.
  - `hangfire:*`: Trạng thái các recurring jobs của `batch-job`.

