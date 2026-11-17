# ADR-001 — Chuyển dịch từ Cron Poller sang RabbitMQ Event-Driven

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-08-15 |
| **Deciders** | Solution Architect, Core Backend Lead |
| **Phân hệ ảnh hưởng** | `backend/`, `crawl-etl-service/`, `header-service/` |

---

### 1. Bối cảnh
Trước đây, hệ thống mSMI sử dụng cơ chế cron job chạy định kỳ trong Node.js (`service_1_purchase.js`, `service_1_sold.js`) để kết nối tuần tự tới từng tài khoản thuế và cào dữ liệu. Khi số lượng doanh nghiệp và lượng hóa đơn tăng vọt, cơ chế này bộc lộ các nhược điểm nghiêm trọng:
- Thời gian chạy kéo dài, job sau chồng chéo lên job trước.
- Khi một tài khoản bị lỗi hoặc CQT bị nghẽn mạng, toàn bộ luồng cào phía sau bị chặn đứng (blocking).
- Không có cơ chế điều phối tải (load leveling) và khó mở rộng theo chiều ngang (horizontal scaling).

### 2. Quyết định
Chúng tôi quyết định chuyển đổi toàn bộ cơ chế thu thập và bóc tách dữ liệu sang kiến trúc **Hướng sự kiện (Event-Driven Architecture)** sử dụng **RabbitMQ Message Broker**:
- Tách luồng thành Producer (đẩy yêu cầu vào hàng đợi) và Consumer (tiếp nhận và xử lý độc lập).
- Sử dụng Exchange `crawl-invoice-raw` làm trục xương sống điều phối thông điệp.
- Worker Node.js và C# chỉ giao tiếp qua message queue, không gọi đồng bộ trực tiếp.

### 3. Lý do
- Đảm bảo tính sẵn sàng cao: Nếu một worker bị sập, message vẫn được lưu giữ an toàn trên RabbitMQ và được worker khác xử lý tiếp.
- Khả năng mở rộng dễ dàng: Có thể tăng số lượng consumer khi lượng hóa đơn tăng đột biến mà không cần sửa đổi logic nghiệp vụ.

**[Kỹ thuật]**
- Sử dụng Direct & Topic Exchange trong RabbitMQ.
- Thiết lập cờ `durable = true` cho queues và `delivery_mode = 2` (persistent) cho message để chống mất dữ liệu khi restart broker.
- Áp dụng cơ chế Manual ACK: Chỉ xác nhận hoàn thành (`basicAck`) sau khi dữ liệu đã được ghi thành công xuống MongoDB hoặc MinIO.

### 4. Hệ quả
- **Tích cực:** Tốc độ đồng bộ tăng gấp nhiều lần, loại bỏ hoàn toàn hiện tượng nghẽn luồng; kiểm soát chính xác trạng thái từng phiên cào.
- **Đánh đổi:** Hệ thống cần duy trì thêm hạ tầng cụm RabbitMQ; việc debug luồng yêu cầu công cụ tracing thay vì đọc log tuần tự đơn thuần.

