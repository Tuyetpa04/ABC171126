# ADR-005 — Áp dụng Kiến trúc Lai: .NET Core cho Workers & Node.js cho API

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-09-01 |
| **Deciders** | Chief Architect, Tech Leads |
| **Phân hệ ảnh hưởng** | Toàn bộ giải pháp Platform |

---

### 1. Bối cảnh
Hệ thống mSMI có hai nhóm tác vụ mang tính chất hoàn toàn trái ngược nhau:
1. Nhóm tác vụ Web / API / BFF: Tương tác với giao diện React, quản lý người dùng, phân quyền, cấu hình giao diện và tích hợp REST API đơn giản.
2. Nhóm tác vụ Xử lý Dữ liệu Nặng (Compute/IO Intensive): Bóc tách hàng triệu dòng Excel, parse cây DOM XML dung lượng lớn, giải mã base64 PDF, kiểm tra chữ ký số mật mã học (cryptography) và duy trì kết nối WebSocket hàng nghìn Bot Desktop.
Nếu dùng toàn bộ Node.js cho nhóm 2, single-thread event loop của Node.js dễ bị block khi parse XML/PDF lớn dẫn đến crash server. Nếu dùng toàn bộ C# cho cả Web API, tốc độ phát triển tính năng giao diện sẽ chậm hơn.

### 2. Quyết định
Chúng tôi quyết định chuẩn hóa mô hình **Kiến trúc Lai (Hybrid Architecture)**:
- **Node.js (Express.js)**: Chịu trách nhiệm toàn bộ tầng Web Server, Backend for Frontend (BFF), Authentication JWT, Quản lý tài khoản và CRUD nghiệp vụ thông thường.
- **C# (.NET Core / .NET 10)**: Chịu trách nhiệm toàn bộ tầng Microservices xử lý dữ liệu nặng (`header-service`, `xml-service`, `pdf-services`, `crawl-etl-service`, `signalr-service`, `minvoice-plugin`).
- Hai bên giao tiếp bất đồng bộ qua RabbitMQ Message Broker.

### 3. Lý do
- Tận dụng thế mạnh đa luồng thực thụ (true multi-threading), quản lý bộ nhớ tối ưu (Span<T>, Memory<T>) và hệ sinh thái thư viện xử lý tài liệu mạnh mẽ của nền tảng .NET cho worker.
- Duy trì tốc độ phát triển ứng dụng (developer velocity) nhanh chóng và linh hoạt của hệ sinh thái Node.js/JavaScript cho tầng API.

### 4. Hệ quả
- **Tích cực:** Tốc độ xử lý bóc tách XML/PDF tăng hơn 300%; không còn hiện tượng block event loop làm treo API của người dùng web.
- **Đánh đổi:** Đội ngũ phát triển cần nắm vững cả hai hệ sinh thái công nghệ (Node.js và C#); quy trình CI/CD cần xây dựng cho cả 2 môi trường build.

