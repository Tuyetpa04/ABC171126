# Hồ sơ Tài liệu Chuẩn hóa Hệ thống mSMI — Phiên bản Hợp nhất Master v2.0

> **Dự án:** mSMI — Minvoice SME Invoice
> **Phiên bản:** `2.0` (Unified Master Baseline — Hợp nhất Toàn diện Nghiệp vụ 9 Phân hệ & Toàn bộ Kiến trúc C4)
> **Ngày phê duyệt:** `2026-11-08` (Cập nhật 08/11/2026)
> **Chuẩn áp dụng:** Quy ước quản trị tài liệu MiniPower (`DOC-01` đến `DOC-17`, `docs-skeleton`)
> **Quy mô hệ thống:** **9 Phân hệ Nghiệp vụ Hoàn chỉnh** + **23 Technical Containers & 47 PM2 Apps** + **7 Cổng Tích hợp Ngoại vi** + **7 ADRs Cốt lõi**.

---

## 1. Cấu trúc Hồ sơ Tài liệu Master Baseline v2.0

```text
DOC-v2.0/
├── README.md                                    # Mục lục tổng quan & Quy ước hợp nhất Master v2.0
├── 00-governance/                               # Quản trị tài liệu & Từ điển thuật ngữ
│   ├── business-glossary.md                     # Từ điển thuật ngữ HĐĐT, CQT, 04/SS, mTax, TNCN, MinIO S3
│   ├── architecture-glossary.md                 # Từ điển kỹ thuật hệ thống, RabbitMQ, PM2, CAS, DLQ, Circuit Breaker
│   └── doc-versioning.md                        # Quy ước phiên bản v2.0, header metadata & Business/Tech ID Spine
├── 01-project/                                  # Hồ sơ cấp dự án
│   ├── DOC-01-vision-business-case.md           # Tầm nhìn nền tảng, 4 trụ cột năng lực & đối tượng thụ hưởng
│   ├── DOC-02-stakeholder-analysis.md           # Danh mục 8 tác nhân (ACT-001..008) & Ma trận RACI
│   └── DOC-03-brd.md                            # Yêu cầu nghiệp vụ tổng thể & Module Index 9 phân hệ
├── 03-modules/                                  # Chi tiết 9 Bounded Context / Phân hệ Nghiệp vụ
│   ├── taxacc/                                  # 1. Kết nối CQT & Tài khoản thuế HĐĐT (DOC-04..07)
│   ├── invoic/                                  # 2. Quản lý Hóa đơn Mua vào / Bán ra & MinIO S3 (DOC-04..07)
│   ├── risk/                                    # 3. Kiểm soát Rủi ro Thuế & Xác thực Chữ ký số (DOC-04..07)
│   ├── mailbx/                                  # 4. Hộp thư Nhận Hóa đơn Tự động Gmail/IMAP (DOC-04..07)
│   ├── tbss/                                    # 5. Thông báo Hóa đơn Sai sót Mẫu 04/SS-HĐĐT (DOC-04..07)
│   ├── report/                                  # 6. Báo cáo Thuế, Tờ khai 01/GTGT & PM Kế toán (DOC-04..07)
│   ├── mtax/                                    # 7. Quản lý Thuế Điện tử & Dịch vụ công CQT (DOC-04..07)
│   ├── tncn/                                    # 8. Chứng từ Khấu trừ Thuế TNCN Mẫu 03/TNCN (DOC-04..07)
│   └── kiemtrahd/                               # 9. Cổng Tra cứu HĐ Công khai kiemtrahoadon.vn (DOC-04..07)
├── 04-platform/                                 # Hồ sơ nền tảng, Kiến trúc Giải pháp & Triển khai
│   ├── DOC-08-sad.md                            # Kiến trúc Tổng thể (SAD 4+1 Views, 23 Containers, 47 PM2, 8 Pipelines)
│   ├── DOC-09-adr/                              # Thư mục 7 Quyết định Kiến trúc chuẩn hóa (ADR-001..007)
│   │   ├── ADR-001-rabbitmq-event-driven.md
│   │   ├── ADR-002-proxy-forwarder-rotation.md
│   │   ├── ADR-003-minio-object-storage.md
│   │   ├── ADR-004-split-header-queues.md
│   │   ├── ADR-005-hybrid-dotnet-node.md
│   │   ├── ADR-006-batch-job-shared-connection.md
│   │   ├── ADR-007-chuan-hoa-http-status-message-cqt.md
│   │   └── README.md
│   ├── DOC-10-integration-specification.md      # Đặc tả 7 cổng tích hợp ngoại vi (INT-001..INT-007)
│   ├── DOC-11-data-model.md                     # Mô hình CSDL 32 collections & Lưu trữ MinIO S3
│   ├── DOC-12-api-specification.md              # Đặc tả 18 REST APIs chuẩn tích hợp OpenAPI
│   ├── DOC-13-nfr.md                            # 25 Yêu cầu Phi chức năng & 5 Kịch bản xác thực thực nghiệm
│   └── DOC-17-deployment-guide.md               # Hướng dẫn Triển khai, Vận hành PM2 & Kế hoạch Khắc phục Thảm họa (DR)
└── 05-traceability/                             # Kiểm soát truy vết & Danh mục quản lý
    ├── doc-registry.md                          # Document Inventory toàn diện (DOC-01..DOC-17)
    ├── trace-matrix.md                          # Ma trận Gióng hàng Đa chiều (Chức năng FR/UC & Kiến trúc NFR/ADR)
    └── c4-to-minipower-mapping.md               # Ma trận Truy vết Đối soát 100% C4 gốc sang chuẩn MiniPower
```

---

## 2. Quy tắc Đặt Mã định danh (ID Spine) Thống nhất

### 2.1. Trục Nghiệp vụ Chức năng (Business Spine)
* **Mã Phân hệ (`{MOD}`)**: `TAXACC`, `INVOIC`, `RISK`, `MAILBX`, `TBSS`, `REPORT`, `MTAX`, `TNCN`, `KTHD`.
* **Quy tắc Nghiệp vụ (Business Rule)**: `{MOD}-BR-NNN` (ví dụ: `INVOIC-BR-001`, `MTAX-BR-002`, `TNCN-BR-001`).
* **Ca sử dụng (Use Case)**: `{MOD}-UC-NNN` (ví dụ: `TAXACC-UC-001`, `KTHD-UC-002`).
* **Yêu cầu chức năng (Functional Req)**: `{MOD}-FR-NNN` (ví dụ: `REPORT-FR-001`, `MTAX-FR-003`).
* **Tiêu chí nghiệm thu BDD (Acceptance Criteria)**: `{MOD}-AC-NNN` (ví dụ: `INVOIC-AC-001`, `KTHD-AC-003`).
* **Tác nhân người dùng (Actor)**: `ACT-NNN` (`ACT-001` đến `ACT-008`).

### 2.2. Trục Kiến trúc & Kỹ thuật Vận hành (Architecture Spine)
* **Vỏ bọc Kỹ thuật (Technical Container)**: `CNT-NNN` (`CNT-001` đến `CNT-023`).
* **Thành phần & Dịch vụ Logic (Component)**: `CMP-NNN` (`CMP-001` đến `CMP-029`).
* **Cổng Tích hợp Ngoại vi (Integration)**: `INT-NNN` (`INT-001` đến `INT-007`).
* **Tiến trình Vận hành Liên-container (Runtime Pipeline)**: `PIPE-NNN` (`PIPE-001` đến `PIPE-008`).
* **Quyết định Kiến trúc (ADR)**: `ADR-NNN` (`ADR-001` đến `ADR-007`).
* **Yêu cầu Phi chức năng (NFR)**: `NFR-{CATEGORY}-NNN` (ví dụ: `NFR-PERF-001`, `NFR-RES-001`...).
* **Kịch bản Kiểm chứng Thực nghiệm (Scenario)**: `SC-NNN` (`SC-001` đến `SC-005`).
* **Rủi ro Kỹ thuật (Technical Risk)**: `RSK-NNN` (`RSK-001` đến `RSK-003`).

---

## 3. Nguồn gốc Dữ liệu & Ma trận Ánh xạ (Traceability Mapping)

| Nhóm Tài liệu Nguồn | Nguồn Dữ liệu Gốc | Phân hệ Tương ứng trong `DOC-v2.0` |
| ------------------------------------------ | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **6 Phân hệ Nghiệp vụ Gốc** | 10 tệp Word, Excel, JSON trong `documents-sources-outdated` | `03-modules/taxacc/`<br/>`03-modules/invoic/`<br/>`03-modules/risk/`<br/>`03-modules/mailbx/`<br/>`03-modules/tbss/`<br/>`03-modules/report/` |
| **Phát sinh 1: Thuế Điện tử** | `mtax-fe/`, `mtax-api/`, `documents/features/mtax/` | `03-modules/mtax/` (DOC-04, 05, 06, 07) |
| **Phát sinh 2: Chứng từ TNCN** | `backend/service/chung-tu-tncn/`, `models/chung_tu_tncn.js` | `03-modules/tncn/` (DOC-04, 05, 06, 07) |
| **Phát sinh 3: Tra cứu Mở** | `kiemtrahoadon/` (React SPA domain `kiemtrahoadon.vn`) | `03-modules/kiemtrahd/` (DOC-04, 05, 06, 07) |
| **Cơ sở dữ liệu & MinIO S3** | `mSMI - Tài liệu thiết kế CSDL.docx` + `ADR-003 MinIO` | `04-platform/DOC-11-data-model.md` |
| **API Tích hợp Hệ thống ngoài** | `[mSMI] Document API.docx` + microservices REST API | `04-platform/DOC-12-api-specification.md` |
| **C1 System Context & Integrations** | `documents/c4/01-system-context.md` | `04-platform/DOC-08-sad.md` (§1.4, §4.1.1) + `DOC-10-integration-specification.md` |
| **C2 Containers & PM2 Deployment** | `documents/c4/02-container.md` + `backend/ecosystem.config.js` | `04-platform/DOC-08-sad.md` (§4.1.2..§4.1.4) + `DOC-17-deployment-guide.md` |
| **C3 23 Core Services Logic** | 22 files trong `documents/c4/03-components/` | `04-platform/DOC-08-sad.md` (§4.1.5 Logic 23 Services & §4.3.3 Bảng 29 Components) |
| **Runtime Pipelines (Workflows)** | `documents/c4/04-code/pipeline-*.md` | `04-platform/DOC-08-sad.md` (§4.2 8 Core Pipelines) |
| **Architecture Decisions (ADRs)** | `documents/adr/` & C4 Solution Plans | `04-platform/DOC-09-adr/` (ADR-001 đến ADR-007) |
| **Yêu cầu Phi chức năng (NFRs)** | C4 Architecture Constraints & Production SLAs | `04-platform/DOC-13-nfr.md` |
