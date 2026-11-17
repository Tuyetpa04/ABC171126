# ADR-003 — Tách Lưu trữ Tệp XML và PDF sang MinIO S3 Object Storage

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-08-25 |
| **Deciders** | Solution Architect, Database Administrator |
| **Phân hệ ảnh hưởng** | `xml-service/`, `pdf-services/`, `backend/`, MongoDB |

---

### 1. Bối cảnh
Trước đây, toàn bộ chuỗi ký tự XML hóa đơn gốc và các tệp đính kèm được lưu trực tiếp dưới dạng trường chuỗi (`raw`, `attachments`) trong MongoDB Document `invoiceraws`.
- Kích thước cơ sở dữ liệu MongoDB phình to nhanh chóng (vượt ngưỡng hàng trăm Gigabytes chỉ sau vài tháng).
- Bộ nhớ RAM của server MongoDB bị chiếm dụng bởi dữ liệu nhị phân không tham gia vào điều kiện lọc/tìm kiếm (queries/indexes).
- Tốc độ sao lưu (backup) và phục hồi thảm họa (restore) của MongoDB bị chậm nghiêm trọng.

### 2. Quyết định
Chúng tôi quyết định tách rời hoàn toàn dữ liệu tệp nhị phân (XML, PDF) ra khỏi cơ sở dữ liệu MongoDB và chuyển sang lưu trữ trên **MinIO Object Storage** (chuẩn tương thích Amazon S3 API):
- MongoDB chỉ lưu trữ metadata, các trường nghiệp vụ cần truy vấn và đường dẫn URI trỏ tới object trong MinIO.
- Xây dựng cơ chế phân vùng bucket (Zone Partitioning): dữ liệu cũ trước tháng 10/2026 đặt tại bucket `minio-default`, dữ liệu từ tháng 10/2026 trở đi được phân vùng vào bucket theo quý/tháng (ví dụ: `minio-202610`).

### 3. Lý do
- Giảm 80% dung lượng lưu trữ của MongoDB, giải phóng bộ nhớ RAM phục vụ cho việc lập chỉ mục (indexing) và tìm kiếm tốc độ cao.
- Chi phí lưu trữ Object Storage rẻ hơn nhiều lần so với chi phí lưu trữ trên SSD hiệu năng cao của Database Server.

**[Kỹ thuật]**
- Microservice `xml-service` và `pdf-services` sử dụng AWS SDK for .NET kết nối trực tiếp tới MinIO Endpoint.
- Định dạng đặt tên Object Key chuẩn hóa: `${tenantId}/${taxYear}/${taxMonth}/${invoiceKey}.xml` để tránh nghẽn I/O trên filesystem của MinIO.
- Hỗ trợ mã hóa lưu trữ (Encryption at rest) đối với các tệp đính kèm nhạy cảm.

### 4. Hệ quả
- **Tích cực:** Tốc độ query danh sách hóa đơn trên giao diện tăng hơn 4 lần; chi phí mở rộng hạ tầng lưu trữ giảm mạnh; việc backup CSDL MongoDB diễn ra nhanh chóng.
- **Đánh đổi:** Cần quản lý tính toàn vẹn 2 pha (2-phase consistency): đảm bảo upload MinIO thành công trước khi cập nhật đường dẫn vào MongoDB.

