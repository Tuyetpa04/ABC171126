# Architecture Glossary — Thuật ngữ Kiến trúc Hệ thống mSMI

| Version | Date | Author | Status |
|---------|------|--------|--------|
| 1.0 | 2026-11-03 | Solution Architect | Baseline |

---

## 1. Thuật ngữ Mô hình C4 & MiniPower

| Thuật ngữ | Định nghĩa | Ý nghĩa trong hệ thống mSMI |
|---|---|---|
| **C1 — System Context** | Cấp độ 1 của mô hình C4: Bối cảnh hệ thống | Xem toàn bộ mSMI là một khối hộp đen (black-box), mô tả tương tác với người dùng và các hệ thống ngoài. |
| **C2 — Container Diagram** | Cấp độ 2 của mô hình C4: Sơ đồ Container | Mô tả các khối thực thi và lưu trữ độc lập (React Frontend, Node.js API, .NET Workers, MongoDB, RabbitMQ, Redis, MinIO). |
| **C3 — Component Diagram** | Cấp độ 3 của mô hình C4: Sơ đồ Thành phần | Mô tả cấu trúc module, controller, service, repository bên trong từng container. |
| **C4 — Code Diagram** | Cấp độ 4 của mô hình C4: Sơ đồ Mã nguồn | Mô tả chi tiết các class, interface, method, sequence diagram và call chain của các pipeline nghiệp vụ trọng yếu. |
| **SAD (DOC-08)** | Software Architecture Document | Tài liệu Kiến trúc Giải pháp tổng thể theo mô hình 4+1 Views của Philippe Kruchten (Logical, Process, Development, Physical, Scenarios). |
| **ADR (DOC-09)** | Architecture Decision Record | Bản ghi nhận các quyết định kiến trúc quan trọng (bối cảnh, phương án lựa chọn, hậu quả, trạng thái). |
| **NFR (DOC-13)** | Non-Functional Requirements | Yêu cầu phi chức năng: hiệu năng, tính sẵn sàng, bảo mật, khả năng mở rộng và khả năng phục hồi. |

---

## 2. Thuật ngữ Kỹ thuật & Hạ tầng mSMI

| Thuật ngữ / Khái niệm | Ý nghĩa kỹ thuật | Vai trò trong hệ thống mSMI |
|---|---|---|
| **Crawl Bot Desktop (`minvoice-plugin`)** | Ứng dụng Desktop viết bằng C# chạy trên máy client/desktop | Kết nối Cổng HĐĐT của Tổng cục Thuế thông qua giao thức SignalR, hỗ trợ giải quyết bài toán chặn IP và môi trường mạng. |
| **SignalR Hub (`signalr-service`)** | Dịch vụ điều phối kết nối thời gian thực (ASP.NET Core) | Quản lý kết nối hai chiều với hàng loạt Bot Desktop, duy trì trạng thái bot rảnh/bận qua Redis cache. |
| **Crawl ETL Service (`crawl-etl-service`)** | Worker xử lý trích xuất và điều phối (.NET) | Lắng nghe các queue yêu cầu cào hóa đơn từ RabbitMQ, đóng gói `MessageCrawl` và phân phối sang SignalR Hub hoặc Proxy Forwarder. |
| **Proxy Forwarder (`proxy-forwarder-service`)** | Dịch vụ luân phiên HTTP Proxy (.NET) | Thực hiện chuyển tiếp request sang Cổng Thuế qua mạng lưới IP xoay vòng để tránh bị CQT khóa IP/rate limit. |
| **Header Service (`header-service`)** | Worker xử lý danh sách hóa đơn (.NET) | Phân tích cú pháp danh sách hóa đơn cào về, định tuyến sang 5 hàng đợi độc lập (`auto-daily`, `auto-month`, `auto-year`, `retry`, `manual`). |
| **XML Service (`xml-service`)** | Worker xử lý tệp XML (.NET) | Giải mã base64 XML, kiểm tra tính toàn vẹn (integrity), bóc tách chi tiết dòng hàng hóa và upload lưu trữ vào MinIO S3. |
| **PDF Services (`pdf-services`)** | Microservice render & lưu trữ PDF (.NET 10) | Nhận queue PDF, kiểm tra header `%PDF`, xác định phân vùng bucket MinIO và cập nhật metadata vào MongoDB. |
| **PM2 Ecosystem** | Trình quản lý tiến trình Node.js trên server | Điều phối 47 ứng dụng tiến trình chạy ngầm (1 web server, 29 workers cào dữ liệu, 16 consumers RabbitMQ). |
| **RabbitMQ Exchange & Queues** | Message Broker trung tâm | Sử dụng Direct/Topic Exchange (`crawl-invoice-raw`, `mtax_crawl`) để tách biệt luồng xử lý và đảm bảo tính chịu lỗi, chống tắc nghẽn. |
| **MinIO Object Storage** | Hệ thống lưu trữ đối tượng tương thích S3 | Thay thế việc lưu trực tiếp file nhị phân vào MongoDB, chia vùng theo bucket (`minio-default`, `minio-202610`). |
| **Hangfire (`batch-job`)** | Framework quản lý tác vụ nền định kỳ (.NET) | Quản lý lịch quét định kỳ, theo dõi trạng thái job phân tán với storage trên MongoDB và Redis. |
| **mTax Pipeline** | Hệ thống con quản lý Thuế điện tử | Chạy trên exchange riêng `mtax_crawl` và cơ sở dữ liệu Mongo riêng `mtax`, độc lập với luồng Hóa đơn điện tử. |

