# Hồ sơ Tài liệu Kiến trúc Hệ thống mSMI (Chuẩn hóa MiniPower)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-07 | Solution Architect | Master Baseline (Cover 100% C4) |

> **Dự án:** mSMI — Minvoice SME Invoice  
> **Chế độ hồ sơ:** `maintain` (Hệ thống hiện hữu — Chuẩn hóa tài liệu Kiến trúc Kỹ thuật thực tế)  
> **Chuẩn áp dụng:** Cấu trúc tài liệu tiêu chuẩn MiniPower (`04-platform/` & `05-traceability/`)  
> **Nguồn dữ liệu gốc:** Toàn bộ tài liệu kiến trúc đối chiếu mã nguồn thực tế tại `documents/c4/` (C1 Context, C2 Container, C3 Components, C4 Code) và `documents/adr/`.
> **Mức độ bao phủ:** **100% C4 Model** được chuẩn hóa có cấu trúc trong `DOC-08 SAD` (23 containers, 47 PM2 processes, 24 components catalog, 8 core pipelines) và các tài liệu platform vệ tinh (`DOC-09..17`).

---

## 1. Cấu trúc Hồ sơ Tài liệu Kiến trúc

```text
C4-docs-standard-minipower/
├── README.md                                    # Mục lục kiến trúc & quy ước chuyển đổi C4 -> MiniPower
├── 00-governance/                               # Quản trị kiến trúc & Thuật ngữ
│   ├── architecture-glossary.md                 # Thuật ngữ kỹ thuật (C1-C4, Crawl Bot, ETL, Proxy Forwarder, MinIO...)
│   └── doc-versioning.md                        # Quy ước phiên bản tài liệu kiến trúc
├── 04-platform/                                 # Bộ hồ sơ Kiến trúc Kỹ thuật Chuẩn MiniPower
│   ├── DOC-08-sad.md                            # Tài liệu Kiến trúc Giải pháp Toàn diện (Master SAD - 4+1 Views)
│   ├── components/                              # 24 Tài liệu Đặc tả Component Kỹ thuật (COMP-001..024)
│   │   ├── README.md                            # Mục lục & Bảng kiểm kê 24 components
│   │   ├── COMP-001-msmi-portal.md              # Đặc tả mSMI Portal (React SPA)
│   │   ├── COMP-005-msmi-backend-api.md         # Đặc tả Backend API Express (node_www)
│   │   ├── COMP-014-header-service.md           # Đặc tả Header Service bóc tách Excel
│   │   ├── COMP-015-xml-service.md              # Đặc tả XML Service & verify chữ ký số
│   │   ├── COMP-016-pdf-services.md             # Đặc tả PDF Multi-Zone Service (.NET 10)
│   │   └── ...                                  # Toàn bộ 24 components
│   ├── DOC-09-adr/                              # Sổ bộ Quyết định Kiến trúc (ADR-001..007)
│   ├── DOC-10-integration-specification.md      # Đặc tả tích hợp 7 hệ thống bên ngoài (INT-001..007)
│   ├── DOC-13-nfr.md                            # Yêu cầu phi chức năng (Thông lượng, Rate limit, TTL, Retry)
│   └── DOC-17-deployment-guide.md               # Hướng dẫn triển khai (47 ứng dụng PM2, RabbitMQ, MinIO)
└── 05-traceability/                             # Kiểm soát Truy vết Kiến trúc
    ├── doc-registry.md                          # Sổ đăng ký danh mục tài liệu & trạng thái ký duyệt
    ├── c4-to-minipower-mapping.md               # Bảng ánh xạ: C1, C2, C3, C4 -> MiniPower DOCs
    └── trace-matrix.md                          # Ma trận truy vết: NFR -> ADR -> Container -> Pipeline
```

---

## 2. Ma trận Ánh xạ Toàn diện Mô hình C4 sang Khung Chuẩn MiniPower

| Mức độ C4 Model | Tài liệu Nguồn gốc trong `documents/c4/` | Tài liệu Chuẩn hóa Thay thế trong `C4-docs-standard-minipower` |
|:---:|---|---|
| **C1 — System Context** | `c4/01-system-context.md` | `04-platform/DOC-08-sad.md` (§1 & §4.1) + `04-platform/DOC-10-integration-specification.md` |
| **C2 — Container Diagram** | `c4/02-container.md` | `04-platform/DOC-08-sad.md` (§4.1 & §4.4) + `04-platform/DOC-17-deployment-guide.md` |
| **C3 — Component Diagrams** | `c4/03-components/*.md` (24 files) | Hợp nhất và chuẩn hóa chuyên sâu tại `04-platform/DOC-08-sad.md` (§4.1.3 Kiểm kê 23 containers & §4.1.5 Quy tắc nghiệp vụ chuyên sâu 23 services cốt lõi). |
| **C4 — Code & Pipelines** | `c4/04-code/*.md` (54 files) | Toàn bộ các luồng runtime lớn được tổng hợp và chuẩn hóa tại `04-platform/DOC-08-sad.md` (§4.2 Process/Runtime View: 8 Core Pipelines). Chi tiết cấp hàm (method-level) được bảo toàn trong codebase và Git history. |
| **ADR (Quyết định)** | `documents/adr/*.md` (9 files) | `04-platform/DOC-09-adr/` (`ADR-001` đến `ADR-007` + `README.md`) |
| **NFR (Phi chức năng)** | `c4/` rate limits, timeouts, queues | `04-platform/DOC-13-nfr.md` |

