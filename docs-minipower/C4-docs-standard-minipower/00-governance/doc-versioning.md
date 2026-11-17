# Doc Versioning — Quy ước Phiên bản Tài liệu Kiến trúc

| Version | Date | Author | Status |
|---------|------|--------|--------|
| 1.0 | 2026-11-03 | Solution Architect | Baseline |

---

## 1. Quy ước Đánh phiên bản Tài liệu Kiến trúc

Tài liệu kiến trúc tuân thủ Semantic Versioning 2 thành phần (`Major.Minor`):
* **`0.x` (Draft / In-progress)**: Bản phác thảo kiến trúc khi đang khảo sát mã nguồn hoặc PoC.
* **`1.0` (First Architecture Baseline)**: Mốc chuẩn hóa toàn diện đầu tiên đối chiếu chính xác với trạng thái mã nguồn và các pipeline thực tế của hệ thống.
* **`1.x` (Minor Revision)**: Cập nhật cấu hình runtime, bổ sung queue mới hoặc làm rõ chi tiết component mà không làm thay đổi mô hình tổng thể.
* **`2.0` (Major Revision)**: Tái cấu trúc lớn (ví dụ chuyển toàn bộ sang Kubernetes, thay thế Message Broker, hoặc hợp nhất hoàn toàn `mtax` vào core).

---

## 2. Tiêu chuẩn Header Bắt buộc của File Kiến trúc

Mọi tài liệu kiến trúc kỹ thuật bắt buộc có metadata đầu trang:

```markdown
# DOC-XX — [Tên Tài liệu]

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 1.0 | 2026-11-03 | Solution Architect | Baseline |
```

---

## 3. Quy ước Đặt Mã Định danh Kiến trúc (Architecture ID Spine)

* **Architecture Goals & Constraints (Mục tiêu kiến trúc)**: `AG-NNN` (ví dụ: `AG-001`, `AG-002`).
* **Architecture Decision Records (Quyết định kiến trúc)**: `ADR-NNN` (ví dụ: `ADR-001`, `ADR-002`).
* **Integration Points (Điểm tích hợp ngoài)**: `INT-NNN` (ví dụ: `INT-001`, `INT-002`).
* **Non-Functional Requirements (Yêu cầu phi chức năng)**: `NFR-NNN` (ví dụ: `NFR-PERF-001`, `NFR-SEC-001`).
* **Pipelines (Luồng thực thi chéo)**: `PIPE-NNN` (ví dụ: `PIPE-001` - Auto Sync HĐĐT).

