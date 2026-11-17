# ADR-002 — Sử dụng Proxy Forwarder Luân phiên IP Chống Chặn CQT

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-08-20 |
| **Deciders** | Solution Architect, Security/Network Lead |
| **Phân hệ ảnh hưởng** | `proxy-forwarder-service/`, `proxy-request-api/`, `crawl-etl-service/` |

---

### 1. Bối cảnh
Cổng thông tin Hóa đơn điện tử của Tổng cục Thuế (`hoadondientu.gdt.gov.vn`) áp dụng chính sách giới hạn tần suất truy vấn (rate limiting) rất chặt chẽ và thường xuyên tạm khóa địa chỉ IP công khai của server mSMI khi phát hiện hàng chục nghìn request dồn dập trong khoảng thời gian ngắn.

### 2. Quyết định
Chúng tôi quyết định xây dựng và triển khai một dịch vụ chuyên trách mang tên **Proxy Forwarder Service (`proxy-forwarder-service`)**:
- Tập hợp một mạng lưới danh sách các địa chỉ IP Proxy đáng tin cậy.
- Mọi request cào dữ liệu qua kênh server sẽ được định tuyến qua Proxy Forwarder Service để tự động luân phiên (round-robin / health-check) IP trước khi gửi sang Cổng Thuế.
- Tự động loại bỏ tạm thời các IP bị CQT trả về mã lỗi HTTP 403 (Forbidden), 429 (Too Many Requests) hoặc timeout.

### 3. Lý do
- Bảo vệ IP gốc của hệ thống mSMI không bao giờ bị đưa vào blacklist của ngành thuế.
- Duy trì tính liên tục của các phiên cào tự động 24/7 mà không phụ thuộc vào một đường truyền duy nhất.

**[Kỹ thuật]**
- Service được viết bằng C# ASP.NET Core để tối ưu hóa hiệu năng network I/O (`HttpClientFactory` kết hợp `SocketsHttpHandler`).
- Hỗ trợ cả Proxy tĩnh và Proxy động có xác thực (Basic Auth / IP Whitelist).
- Tích hợp fallback sang luồng Bot Desktop SignalR khi toàn bộ dải IP server gặp sự cố nghẽn mạng nghiêm trọng.

### 4. Hệ quả
- **Tích cực:** Tỷ lệ cào hóa đơn thành công duy trì trên 98.5%, không còn tình trạng toàn hệ thống bị dừng do nghẽn IP CQT.
- **Đánh đổi:** Tăng chi phí thuê hạ tầng proxy và phát sinh thêm một độ trễ nhỏ (khoảng 100 - 300ms) trên mỗi HTTP handshake.

