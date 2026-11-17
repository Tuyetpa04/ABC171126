# DOC Registry — Hồ sơ Kiến trúc Kỹ thuật mSMI

| Cập nhật | 2026-11-07 |
|:---:|:---:|

Đăng ký danh mục tài liệu kỹ thuật nền tảng (`04-platform`) và quản trị truy vết (`00-governance`, `05-traceability`).
- **Phạm vi kỹ thuật:** Mô hình C4 (Context C1, Container C2, Components C3, Pipelines/Code C4), Sổ bộ Quyết định Kiến trúc (ADR), Báo cáo Rà soát Kiểm thử (QA), và Kiểm kê Mã nguồn (Source Audit).
- **Trạng thái phê duyệt:** `Master Baseline` v2.0 (Cover 100% C4 Architecture).

---

## 1. governance (Quản trị Dùng chung)

| Path | Mô tả tài liệu | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|---|:---:|:---:|:---:|:---:|:---:|
| `00-governance/architecture-glossary.md` | Từ điển thuật ngữ kỹ thuật, viết tắt (C1-C4, Crawl Bot, MinIO, SignalR...) | 1.0 | Solution Architect | Baseline | Yes | ☑ |
| `00-governance/doc-versioning.md` | Quy ước phiên bản tài liệu kiến trúc & cấu trúc mã định danh kỹ thuật | 1.0 | Solution Architect | Baseline | Yes | ☑ |

---

## 2. platform (Tài liệu Kiến trúc & Hạ tầng Kỹ thuật)

| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `04-platform/DOC-08-sad.md` | 08 | 2.0 | Solution Architect | Baseline | Yes | ☑ |
| `04-platform/DOC-09-adr/` (7 files) | 09 | 2.0 | Solution Architect | Baseline | Yes | ☑ |
| `04-platform/DOC-10-integration-specification.md` | 10 | 1.1 | Solution Architect | Baseline | Yes | ☑ |
| `04-platform/DOC-13-nfr.md` | 13 | 1.1 | Solution Architect | Baseline | Yes | ☑ |
| `04-platform/DOC-17-deployment-guide.md` | 17 | 1.1 | DevOps Lead | Baseline | Yes | ☑ |

---

## 3. traceability (Ma trận Truy vết & Ánh xạ C4)

| Path | Mô tả tài liệu | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|---|:---:|:---:|:---:|:---:|:---:|
| `05-traceability/c4-to-minipower-mapping.md` | Bảng ánh xạ đối chiếu 1-1 từ 70+ file C4/ADR gốc sang các DOC chuẩn hóa | 2.0 | Solution Architect | Baseline | Yes | ☑ |
| `05-traceability/trace-matrix.md` | Ma trận gióng hàng đa chiều: NFR $\leftrightarrow$ ADR $\leftrightarrow$ Container $\leftrightarrow$ Pipeline $\leftrightarrow$ Kịch bản | 2.0 | Solution Architect | Baseline | Yes | ☑ |

