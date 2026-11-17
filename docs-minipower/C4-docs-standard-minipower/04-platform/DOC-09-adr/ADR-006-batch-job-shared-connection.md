# ADR-006 — Tối ưu Sử dụng Chung Kết nối RabbitMQ trong Batch Job

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-10-15 |
| **Deciders** | Solution Architect, DevOps Lead |
| **Phân hệ ảnh hưởng** | `batch-job/`, RabbitMQ Cluster |

---

### 1. Bối cảnh
Dịch vụ `batch-job` sử dụng framework Hangfire trong .NET để kích hoạt hàng loạt tác vụ nền định kỳ (quét hóa đơn, kiểm tra trạng thái MST, đồng bộ token). 
- Ở thiết kế ban đầu, mỗi một job con được khởi tạo lại tạo mới một TCP Connection riêng tới RabbitMQ (`IConnection.CreateModel()`).
- Khi Hangfire dispatch hàng nghìn job cùng lúc, số lượng kết nối TCP tới RabbitMQ tăng đột biến (vượt ngưỡng connection limit), gây lỗi cạn kiệt socket (Socket Exhaustion) và làm treo server RabbitMQ.

### 2. Quyết định
Chúng tôi quyết định áp dụng mẫu thiết kế **Singleton Connection Pool** cho RabbitMQ trong `batch-job`:
- Khởi tạo một kết nối vật lý duy nhất (`IConnection`) dưới dạng Singleton Lifecycle tại thời điểm khởi động ứng dụng (`Program.cs`).
- Mỗi job Hangfire khi chạy chỉ mở một kênh logic tạm thời (`IModel / Channel`) từ kết nối dùng chung đó và đóng kênh ngay sau khi hoàn thành việc publish message.

### 3. Lý do
- Giảm số lượng kết nối TCP đồng thời tới RabbitMQ từ hàng nghìn kết nối xuống còn đúng 1 kết nối duy nhất từ service `batch-job`.
- Tiết kiệm bộ nhớ và tài nguyên socket của cả máy chủ ứng dụng lẫn máy chủ message broker.

### 4. Hệ quả
- **Tích cực:** Loại bỏ 100% lỗi `SocketException` và lỗi quá tải connection trên RabbitMQ; tăng độ ổn định của hệ thống lên mức tối đa.
- **Đánh đổi:** Cần quản lý cơ chế tự động kết nối lại (`AutomaticRecoveryEnabled = true`) trên kết nối Singleton khi có sự cố mạng.

