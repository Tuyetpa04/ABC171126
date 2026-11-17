# DOC Registry — mSMI (Hồ sơ Chuẩn hóa Toàn diện v2.0)

| Cập nhật | 2026-11-08 | Phiên bản | 2.0 (Master Baseline Unified) |
|:---:|:---:|:---:|:---:|

Đăng ký tài liệu toàn diện hệ thống chuẩn hóa theo tiêu chuẩn MiniPower:
- **Tiêu chuẩn:** Bộ hồ sơ giải pháp chuẩn MiniPower (`DOC-01` đến `DOC-17`).
- **Phạm vi nghiệp vụ:** Toàn bộ 9 phân hệ nghiệp vụ hoàn chỉnh (6 phân hệ gốc + 3 phân hệ phát sinh `mtax`, `tncn`, `kiemtrahd`).
- **Phạm vi kỹ thuật & kiến trúc:** Toàn bộ kiến trúc C4 đối chiếu từ thực tế mã nguồn (23 Containers, 47 PM2 processes, 8 Pipelines, 7 ADRs, 7 Cổng tích hợp, 25 NFRs).
- **Trạng thái tài liệu:** `Master Baseline Unified v2.0` (Đã chuẩn hóa, hợp nhất và ký duyệt).

---

## 1. governance (Quản trị Dùng chung)

| Path | Mô tả | Ver | Người thực hiện | Status |
|---|---|:---:|:---:|:---:|
| `00-governance/business-glossary.md` | Thuật ngữ nghiệp vụ HĐĐT, CQT, 04/SS, SCO, 1.500 DN rủi ro cao và quy tắc DOC | 2.0 | BA | Baseline Unified v2.0 |
| `00-governance/architecture-glossary.md` | Thuật ngữ kỹ thuật hệ thống, C2 Container, C3 Component, RabbitMQ, PM2, MinIO, CAS, Circuit Breaker | 2.0 | SA | Baseline Unified v2.0 |
| `00-governance/doc-versioning.md` | Quy tắc đánh mã ID Business & Architecture Spine, định dạng Version và Change Log | 2.0 | BA / SA | Baseline Unified v2.0 |

---

## 2. project (Tài liệu Dự án & Nghiệp vụ Cấp cao)

| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `01-project/DOC-01-vision-business-case.md` | 01 | 2.0 | BA | Baseline | Yes | ☑ |
| `01-project/DOC-02-stakeholder-analysis.md` | 02 | 2.0 | BA | Baseline | Yes | ☑ |
| `01-project/DOC-03-brd.md` | 03 | 2.0 | BA | Baseline | Yes | ☑ |

---

## 3. modules (Đặc tả Nghiệp vụ Từng Phân hệ)

### 3.1. taxacc — Phân hệ Kết nối Cơ quan Thuế & Đồng bộ HĐĐT
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/taxacc/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/taxacc/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/taxacc/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/taxacc/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.2. invoic — Phân hệ Quản lý & Thao tác Hóa đơn (Mua vào / Bán ra)
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/invoic/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/invoic/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/invoic/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/invoic/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.3. risk — Phân hệ Kiểm soát & Cảnh báo Rủi ro Thuế
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/risk/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/risk/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/risk/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/risk/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.4. mailbx — Phân hệ Quản lý Hộp thư & Bóc tách Hóa đơn từ Email
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/mailbx/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mailbx/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mailbx/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mailbx/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.5. tbss — Phân hệ Quản lý Thông báo Hóa đơn Sai sót (Mẫu 04/SS-HĐĐT)
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/tbss/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tbss/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tbss/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tbss/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.6. report — Phân hệ Báo cáo Thuế, Tờ khai 01/GTGT & Kết xuất Kế toán
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/report/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/report/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/report/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/report/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.7. mtax — Phân hệ Quản lý Thuế Điện tử & Dịch vụ công CQT
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/mtax/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mtax/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mtax/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/mtax/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.8. tncn — Phân hệ Quản lý Chứng từ Khấu trừ Thuế TNCN Điện tử
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/tncn/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tncn/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tncn/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/tncn/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

### 3.9. kiemtrahd — Phân hệ Cổng Tra cứu & Kiểm tra Hóa đơn Công khai
| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `03-modules/kiemtrahd/DOC-04-business-rules.md` | 04 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/kiemtrahd/DOC-05-use-cases.md` | 05 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/kiemtrahd/DOC-06-srs.md` | 06 | 2.0 | BA | Baseline | Yes | ☑ |
| `03-modules/kiemtrahd/DOC-07-acceptance-criteria.md` | 07 | 2.0 | BA | Baseline | Yes | ☑ |

---

## 4. platform (Tài liệu Nền tảng Kỹ thuật, Kiến trúc Giải pháp & API)

| Path | DOC | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| `04-platform/DOC-08-sad.md` | 08 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-09-adr/` (`ADR-001`..`007`) | 09 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-10-integration-specification.md` | 10 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-11-data-model.md` | 11 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-12-api-specification.md` | 12 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-13-nfr.md` | 13 | 2.0 | SA | Baseline | Yes | ☑ |
| `04-platform/DOC-17-deployment-guide.md` | 17 | 2.0 | SA / DevOps | Baseline | Yes | ☑ |

---

## 5. traceability (Hồ sơ Truy vết & Giống hàng Đa chiều)

| Path | Mô tả | Ver | Người thực hiện | Status | Baseline | Sign-off |
|---|---|:---:|:---:|:---:|:---:|:---:|
| `05-traceability/doc-registry.md` | Danh mục quản trị toàn diện hồ sơ tài liệu hệ thống (DOC-01..17) | 2.0 | BA / SA | Baseline | Yes | ☑ |
| `05-traceability/trace-matrix.md` | Ma trận gióng hàng 2 chiều: Nghiệp vụ (FR ↔ UC ↔ BR ↔ AC) & Kiến trúc (NFR ↔ ADR ↔ Container ↔ Pipeline ↔ Scenario) | 2.0 | BA / SA | Baseline | Yes | ☑ |
| `05-traceability/c4-to-minipower-mapping.md` | Ma trận đối soát 100% nguồn tài liệu C4 gốc sang chuẩn hồ sơ MiniPower | 2.0 | SA | Baseline | Yes | ☑ |

---

## 6. Danh mục Tài liệu Gốc Outdated (Legacy Sources Inventory)

| STT | Tên tệp gốc trong `documents-sources-outdated` | Định dạng | Module tương ứng | Trạng thái chuẩn hóa |
|:---:|---|:---:|:---:|:---:|
| 1 | `[mSMI] API Đăng ký, cập nhật tài khoản Thuế.docx` | DOCX | `taxacc`, `platform` | Chuẩn hóa vào TAXACC + DOC-12 |
| 2 | `[mSMI] Document API.docx` | DOCX | `invoic`, `risk`, `platform` | Chuẩn hóa vào INVOIC, RISK + DOC-12 |
| 3 | `App hóa đơn vào_ ra SMI.xlsx` | XLSX | Toàn bộ các module | Chuẩn hóa vào BRD + DOC-04/05/06 |
| 4 | `Luồng crawl data mSMI.json` | JSON | `taxacc`, `risk` | Chuẩn hóa vào TAXACC-UC + DOC-11 |
| 5 | `mSMI - Hướng dẫn xử lý các lỗi hay gặp phải.docx` | DOCX | `taxacc`, `mailbx`, `report` | Chuẩn hóa vào TAXACC, MAILBX, REPORT |
| 6 | `mSMI - Tài liệu HDSD phần mềm quản lý rủi ro hóa đơn.docx`| DOCX | Toàn bộ 6 modules | Chuẩn hóa vào 6 modules UC/FR/BR |
| 7 | `mSMI - Tài liệu thiết kế CSDL.docx` | DOCX | `platform` | Chuẩn hóa toàn diện vào DOC-11 |
| 8 | `SMI_Sơ đồ nghiệp vụ.json` | JSON | Toàn bộ 6 modules | Chuẩn hóa vào BRD + Use Cases |
| 9 | `TÀI LIỆU HƯỚNG DẪN  ĐỒNG BỘ HÓA ĐƠN QUA EMAIL.docx` | DOCX | `mailbx` | Chuẩn hóa vào MAILBX (DOC-04..07) |
| 10 | `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx` | DOCX | `risk` | Chuẩn hóa vào RISK (DOC-04..07) |

