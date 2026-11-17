# DOC-13 — Yêu cầu Phi Chức năng Hệ thống mSMI (NFR)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | Solution Architect | Baseline Unified v2.0 |

> **Tiêu chuẩn tham khảo:** ISO/IEC 25010 (Software Product Quality); Adjunct of SAD (DOC-08).  
> **Căn cứ đối chiếu:** Hiện trạng mã nguồn và kiến trúc hàng đợi tại `documents/c4/`, `documents/adr/`.

---

## 1. Ma trận Yêu cầu Phi Chức năng (NFR Matrix)

| Mã NFR | Nhóm chất lượng | Tiêu chí kỹ thuật & Chỉ số đo lường | Giải pháp Kiến trúc đáp ứng |
|:---:|---|---|---|
| **NFR-PERF-001** | Hiệu năng Cào Hóa đơn | Hệ thống cào và phân tích tối thiểu 500.000 hóa đơn/ngày; độ trễ xử lý hàng đợi dưới 30 giây khi cào tự động. | Cụm RabbitMQ kết hợp `header-service` đa luồng và 16 consumer PM2. |
| **NFR-PERF-002** | Độ trễ Cào Thủ công (Manual Sync) | Thao tác cào hóa đơn tức thì từ Portal phải được đưa vào hàng đợi ưu tiên cao và nhận kết quả đầu tiên dưới 10 giây; cửa sổ theo dõi UI 15 giây. | Tách riêng queue `invoice-raw-header` với prefetch = 30 độc lập với job nền (`ADR-004`), polling MongoDB từng ID. |
| **NFR-PERF-003** | Tải Hàng loạt PDF & Fallback HTML | Xử lý tải hàng loạt PDF; nếu thiếu file gốc từ CQT, tự động fallback sang worker thread render Pug HTML mẫu 1–6; timeout 120 giây; không làm sập batch. | Worker thread nội bộ trong Backend API, in tuần tự từng tờ, xuất file biên bản lỗi `_bao_cao_loi.txt`. |
| **NFR-RES-001** | Khả năng Chịu lỗi & Vượt Rate Limit | Khi Cổng Thuế trả về HTTP 403/429 hoặc timeout, hệ thống tự động luân chuyển proxy/bot trong vòng 3 giây. | `proxy-forwarder-service` xoay vòng dải IP (`ADR-002`) và mạng lưới Bot Desktop SignalR. |
| **NFR-RES-002** | Kiểm soát Khuếch đại Tải & Nhất quán Message | Khi CQT trả 500/503/timeout, PFS dừng failover và PRA giới hạn retry; 100% câu thông báo hiển thị tập trung tại PRA, không leak raw body ra UI. | Chuẩn hóa tầng PRA dựng thông điệp hiển thị, BE/FE chuyển tiếp nguyên văn (`ADR-007`). |
| **NFR-STOR-001** | Tối ưu Lưu trữ & Dung lượng CSDL | Không làm phình MongoDB quá 20% dung lượng tệp nhị phân; bảo toàn thời gian truy vấn danh sách hóa đơn dưới 500ms. | Tách toàn bộ tệp XML và PDF sang MinIO S3 Object Storage (`ADR-003`). |
| **NFR-SEC-001** | An toàn Mật khẩu & Token | 100% mật khẩu CQT, App Password và JWT Token lưu trong CSDL bắt buộc phải được mã hóa chuẩn mật mã học. | Module mã hóa đối xứng 2 chiều trước khi lưu xuống bảng `externalaccounts`. |
| **NFR-ISO-001** | Tính Cách ly Phân hệ Thuế (mTax) | Sự cố nghẽn hàng đợi hoặc lỗi dữ liệu của phân hệ Thuế điện tử không được ảnh hưởng đến phân hệ HĐĐT. | Tách riêng RabbitMQ Exchange `mtax_crawl` và database MongoDB `mtax`. |
| **NFR-AVAIL-001**| Độ Sẵn sàng Hạ tầng (Availability) | Hệ thống API Portal đạt uptime tối thiểu 99.5% trong giờ hành chính. | PM2 auto-restart khi crash, trần memory heap 4GB, cụm RabbitMQ HA. |

---

## 2. Thông số Cấu hình Runtime Chi tiết

### 2.1. Cấu hình Prefetch & Concurrency RabbitMQ
- Queue cào thủ công (`consumer_invoice_header`): `prefetch = 30` (xử lý ưu tiên, không giữ quá nhiều message chưa hoàn thành).
- Queue cào tự động ngày (`consumer_invoice_header_auto_daily`): `prefetch = 20`.
- Queue cào tự động tháng/năm (`consumer_invoice_header_auto_month`, `_year`): `prefetch = 15` và `10` (tránh nghẽn RAM khi parse bảng kê lớn).
- Consumer chi tiết (`consumer_invoice_detail`): `prefetch = 20`.
- Consumer XML & PDF: `prefetch = 10` (xử lý stream nhị phân và upload S3).
- Chính sách NACK & Retry consumer: Nack tối đa 3 lần và ghi nhận log trước khi ACK; kích hoạt cơ chế Data Repair đối với các hóa đơn bị commit muộn.

### 2.2. Chính sách Timeout & Giới hạn Bộ nhớ (Timeouts & Memory Limits)
- HTTP Timeout cào CQT qua Proxy: `ConnectTimeout = 20s`, `Forwarding.TimeoutSeconds = 120s`.
- Cấu hình Proxy Pool PFS: `ProxyMaxFailures = 3`, `ProxyBanMinutes = 1`, `ProxyBatchSize = 3`.
- Timeout tải hàng loạt PDF fallback: 120 giây; tối đa 2 lượt/process để chống tràn heap.
- Cửa sổ theo dõi UI cho cào thủ công: 15 giây polling trạng thái qua Mongo.
- Node.js Process Memory Limit: `--max-old-space-size=4096` (4GB) trên tiến trình `node_www` và các worker nặng; tự động restart định kỳ qua PM2 cron để triệt tiêu hiện tượng memory leak.

