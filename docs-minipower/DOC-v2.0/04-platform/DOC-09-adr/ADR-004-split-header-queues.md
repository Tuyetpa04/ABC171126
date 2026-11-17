# ADR-004 — Tách Riêng 5 Hàng đợi Header Độc lập theo Loại Chu kỳ Job

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-08-27 |
| **Deciders** | Solution Architect, Core Backend Lead |
| **Phân hệ ảnh hưởng** | `header-service/`, `backend/consume/invoiceHeader.js`, RabbitMQ |

---

### 1. Bối cảnh
Trước đây, toàn bộ kết quả bóc tách danh sách hóa đơn (Header) từ các nguồn đều được đổ chung vào một hàng đợi duy nhất trên RabbitMQ mang tên `invoice-raw-header`.
- Khi đến kỳ chốt sổ hoặc khi có người dùng cấu hình cào lại dữ liệu của cả năm (`job_year`), một lượng lớn (hàng chục nghìn) message cào năm đổ dồn vào hàng đợi.
- Hàng đợi bị nghẽn (head-of-line blocking), khiến các job cào phát sinh theo ngày (`job_daily`) và đặc biệt là thao tác cào thủ công tức thì của người dùng trên giao diện web bị xếp hàng chờ hàng giờ đồng hồ mới được xử lý.

### 2. Quyết định
Chúng tôi quyết định tách hàng đợi Header thành **5 queue độc lập** trên Exchange `crawl-invoice-raw`:
1. `invoice-raw-header`: Dành riêng cho luồng cào thủ công tức thì từ Portal (Ưu tiên cao nhất).
2. `invoice-raw-header-retry`: Dành cho các tác vụ cào bù, retry khi gặp lỗi.
3. `invoice-raw-header-auto-daily`: Dành cho cron job tự động quét ngày (tick ngắn).
4. `invoice-raw-header-auto-month`: Dành cho job tự động đối soát theo tháng.
5. `invoice-raw-header-auto-year`: Dành cho job cào quét lịch sử cả năm.

Phía `backend/ecosystem.config.js`, phân bổ 5 tiến trình PM2 riêng biệt tương ứng với các mức hạn ngạch tiếp nhận (prefetch count) khác nhau:
- Consumer thủ công (`consumer_invoice_header`): Prefetch = 30.
- Consumer daily (`consumer_invoice_header_auto_daily`): Prefetch = 20.
- Consumer retry (`consumer_invoice_header_retry`): Prefetch = 15.
- Consumer month (`consumer_invoice_header_auto_month`): Prefetch = 15.
- Consumer year (`consumer_invoice_header_auto_year`): Prefetch = 10.

### 3. Lý do
- Đảm bảo tính công bằng (Fairness) và trải nghiệm người dùng: Thao tác bấm tay trên giao diện luôn được phản hồi ngay lập tức, không bao giờ bị nghẽn bởi các job chạy nền nặng nề.

**[Kỹ thuật]**
- Microservice `header-service` bổ sung module `InvoiceHeaderQueueRouter.cs` để phân tích thuộc tính metadata của gói tin và dispatch tới đúng Routing Key tương ứng.
- Cấu hình Dead Letter Exchange (DLX) riêng cho từng queue để cô lập các bản ghi lỗi.

### 4. Hệ quả
- **Tích cực:** Độ trễ cào thủ công từ giao diện giảm từ vài chục phút xuống dưới 10 giây; hệ thống tự động cân bằng tải hoàn hảo giữa tác vụ nặng và tác vụ nhẹ.
- **Đánh đổi:** Tăng số lượng tiến trình PM2 trên server từ 1 lên 5 tiến trình consumer cho mảng Header.

