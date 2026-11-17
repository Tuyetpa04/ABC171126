# Doc Versioning — Quy ước Phiên bản & Header Tài liệu

| Version | Date | Author | Status |
|:---:|:---:|:---:|:---:|
| 1.0 | 2026-11-03 | BA | First Baseline (6 Core Modules) |
| 2.0 | 2026-11-08 | BA / Solution Architect | Master Baseline Unified v2.0 (9 Modules & C4 Architecture) |

---

## 1. Quy ước Đánh phiên bản (Semantic Versioning for Docs)

Hồ sơ tài liệu tuân thủ quy ước versioning 2 thành phần `Major.Minor`:

* **`0.x` (Draft / In-progress)**: Tài liệu đang trong giai đoạn soạn thảo, khảo sát hoặc chưa qua phê duyệt.
* **`1.0` (First Baseline)**: Phiên bản chuẩn hóa đầu tiên từ nguồn tài liệu gốc gồm 6 phân hệ hóa đơn cốt lõi.
* **`1.x` (Minor changes)**: Cập nhật, làm rõ chi tiết hoặc bổ sung ví dụ minh họa mà không làm thay đổi phạm vi nghiệp vụ lớn.
* **`2.0` (Master Baseline Unified — Hiện tại)**: Hợp nhất toàn diện giữa Nghiệp vụ Gốc (9 phân hệ hoàn chỉnh) và Toàn bộ Kiến trúc Hệ thống C4 (`DOC-08`, `DOC-09`, `DOC-10`, `DOC-11`, `DOC-12`, `DOC-13`, `DOC-17`), tạo thành một bộ hồ sơ giải pháp đồng nhất duy nhất (Single Source of Truth) kết nối đầy đủ từ Nghiệp vụ (BA) đến Kiến trúc & Hạ tầng Kỹ thuật (SA/DevOps).

---

## 2. Tiêu chuẩn Header bắt buộc của mọi file DOC

Mọi tệp tài liệu trong hệ thống bắt buộc phải có bảng metadata đầu trang:

```markdown
# DOC-XX — [Tên Tài liệu]

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | BA / Solution Architect | Baseline Unified v2.0 |
```

---

## 3. Quy tắc Mã định danh (ID Spine)

### 3.1. Danh mục 9 Mã Phân hệ Nghiệp vụ (`{MOD}`)
1. **`TAXACC`**: Phân hệ Kết nối CQT & Quản lý Tài khoản Thuế
2. **`MAILBX`**: Phân hệ Hộp thư Email Nhận Hóa đơn
3. **`INVOIC`**: Phân hệ Quản lý & Thao tác Hóa đơn Mua vào / Bán ra
4. **`RISK`**: Phân hệ Kiểm soát & Cảnh báo Rủi ro Thuế
5. **`TBSS`**: Phân hệ Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)
6. **`REPORT`**: Phân hệ Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất PMKT
7. **`MTAX`**: Phân hệ Quản lý Thuế Điện tử & Dịch vụ công CQT
8. **`TNCN`**: Phân hệ Quản lý Chứng từ Khấu trừ Thuế TNCN Điện tử
9. **`KTHD`**: Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn Công khai (`kiemtrahoadon.vn`)

### 3.2. Quy cách Đánh mã Đối tượng Nghiệp vụ (Business Spine)
* **Business Rules (Quy tắc nghiệp vụ)**: `{MOD}-BR-NNN` (ví dụ: `INVOIC-BR-001`, `MTAX-BR-002`, `TNCN-BR-001`)
* **Use Cases (Ca sử dụng)**: `{MOD}-UC-NNN` (ví dụ: `TAXACC-UC-001`, `KTHD-UC-002`)
* **Functional Requirements (Yêu cầu chức năng)**: `{MOD}-FR-NNN` (ví dụ: `REPORT-FR-001`, `TNCN-FR-003`)
* **Acceptance Criteria (Tiêu chí nghiệm thu BDD)**: `{MOD}-AC-NNN` (ví dụ: `INVOIC-AC-001`, `KTHD-AC-003`)
* **Tác nhân người dùng (Actor)**: `ACT-NNN` (`ACT-001` đến `ACT-008`)

### 3.3. Quy cách Đánh mã Đối tượng Kiến trúc & Kỹ thuật (Architectural Spine)
* **Technical Container (Vỏ bọc kỹ thuật C2)**: `CNT-NNN` (`CNT-001` đến `CNT-023`)
* **Component & Microservice (Thành phần logic C3)**: `CMP-NNN` (`CMP-001` đến `CMP-029`)
* **External Integration (Cổng tích hợp ngoại vi)**: `INT-NNN` (`INT-001` đến `INT-007`)
* **Runtime Pipeline (Tiến trình vận hành liên-container)**: `PIPE-NNN` (`PIPE-001` đến `PIPE-008`)
* **Architecture Decision Record (Quyết định kiến trúc)**: `ADR-NNN` (`ADR-001` đến `ADR-007`)
* **Non-Functional Requirement (Yêu cầu phi chức năng)**: `NFR-{CATEGORY}-NNN` (ví dụ: `NFR-PERF-001`, `NFR-RES-001`, `NFR-SEC-001`...)
* **Technical Risk (Rủi ro kỹ thuật)**: `RSK-NNN` (`RSK-001` đến `RSK-003`)
* **Verification Scenario (Kịch bản kiểm chứng thực nghiệm)**: `SC-NNN` (`SC-001` đến `SC-005`)

Nguyên tắc bất biến: 
* **Nghiệp vụ:** Mọi `AC` phải trỏ về `FR`; Mọi `FR` phải trỏ về `UC` hoặc `BR` tương ứng.
* **Kiến trúc:** Mọi `NFR` phải có `ADR` bảo trợ giải pháp kỹ thuật, được triển khai tại `Container/Component` cụ thể, vận hành qua `Pipeline` và kiểm chứng bằng `Scenario` tương ứng.

