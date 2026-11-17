# DOC-12 — Đặc tả API Tích hợp mSMI (API Specification)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | BA / Solution Architect | Baseline Unified v2.0 |

> **Phân loại tài liệu:** Platform API Specification (Đặc tả API Tích hợp Lõi)  
> **Căn cứ tài liệu gốc:** `[mSMI] Document API.docx`, `[mSMI] API Đăng ký, cập nhật tài khoản Thuế.docx`.

---

## 1. Tổng quan & Môi trường

| Mục | Môi trường Kiểm thử (Test / UAT) | Môi trường Sản xuất (Production) |
|---|---|---|
| **Base URL Quản lý Hóa đơn** | `https://test-qlhd.minvoice.com.vn` | `https://qlhd.minvoice.com.vn` |
| **Base URL Tra cứu MST Thuế** | `https://mst.minvoice.com.vn` | `https://mst.minvoice.com.vn` |
| **Định dạng dữ liệu** | `application/json` (RESTful) | `application/json` (RESTful) |
| **Chuẩn tài liệu** | OpenAPI 3.0 / REST | OpenAPI 3.0 / REST |

---

## 2. Xác thực & Phân quyền (Authentication)

- **Cơ chế xác thực:** Hệ thống sử dụng cơ chế Static API Token được cấp phát riêng cho từng tài khoản đơn vị/doanh nghiệp.
- **Vị trí truyền:** Header của HTTP Request.
- **Tên Header:** `apiToken: <Giá trị Token được cấp phát>`
- **Cấu hình Token:**
  - Token được sinh ngẫu nhiên khi khai báo trong phần mềm, đảm bảo không trùng lặp.
  - Hỗ trợ thiết lập ngày hết hạn (sau ngày này token tự động vô hiệu) hoặc để trống hạn dùng vĩnh viễn.
  - Một đơn vị có thể tạo nhiều Token để cấp cho nhiều phần mềm vệ tinh (MISA, FAST, Bravo, ERP nội bộ).

---

## 3. Danh mục Endpoint

| STT | Phương thức | Đường dẫn API (Endpoint) | Tên Chức năng Nghiệp vụ | Trace (UC / FR) |
|:---:|:---:|---|---|:---:|
| **1** | `POST` | `/erp/qlhd-api/create-tax-account` | Đăng ký & kết nối tài khoản Cổng Thuế mới | TAXACC-FR-001 |
| **2** | `POST` | `/erp/qlhd-api/edit-tax-account` | Cập nhật / đăng nhập lại tài khoản Cổng Thuế | TAXACC-FR-001 |
| **3** | `GET` | `/erp/qlhd-api/invoices` | Lấy danh sách hóa đơn theo điều kiện lọc | INVOIC-FR-001 |
| **4** | `GET` | `/erp/qlhd-api/invoices/modified` | Lấy danh sách hóa đơn có phát sinh thay đổi | INVOIC-FR-007 |
| **5** | `GET` | `/erp/qlhd-api/invoices/{id}/download/invoice.xml` | Tải file XML hóa đơn gốc | INVOIC-FR-003 |
| **6** | `GET` | `/erp/qlhd-api/invoices/{id}/download/invoice.html`| Tải bản in PDF / HTML thể hiện hóa đơn | INVOIC-FR-002 |
| **7** | `GET` | `/erp/qlhd-api/invoices/{id}/warning` | Lấy thông tin cảnh báo rủi ro hóa đơn | RISK-FR-003 |
| **8** | `GET` | `/api/System/SearchTaxCodeV2` | Tra cứu trạng thái hoạt động của MST từ CQT | RISK-FR-001 |
| **9** | `POST` | `/api/v2/declaration/crawl` | Kích hoạt cào tờ khai thuế từ eTax / DVC | MTAX-FR-002 |
| **10** | `GET` | `/api/v2/declaration/list` | Lấy danh sách tờ khai thuế đã đồng bộ | MTAX-FR-002 |
| **11** | `GET` | `/api/v2/payment/list` | Lấy danh sách giấy nộp tiền NSNN | MTAX-FR-003 |
| **12** | `GET` | `/api/v2/notice/list` | Lấy danh sách thông báo thuế từ CQT | MTAX-FR-004 |
| **13** | `GET` | `/api/chung-tu-tncn` | Tra cứu danh sách chứng từ khấu trừ thuế TNCN | TNCN-FR-002 |
| **14** | `POST` | `/api/chung-tu-tncn/crawl` | Kích hoạt cào chứng từ TNCN từ CQT | TNCN-FR-001 |
| **15** | `GET` | `/query/guest-invoices` | Tra cứu trạng thái hóa đơn CQT công khai | KTHD-FR-001 |
| **16** | `GET` | `/external_accounts/captcha` | Lấy mã Captcha đồ họa SVG từ CQT | KTHD-FR-001 |
| **17** | `POST` | `/upload_xml_data` | Tải lên và thẩm định pháp lý tệp XML công khai | KTHD-FR-002 |
| **18** | `PATCH` | `/invoices/:id/tthai` | Cập nhật đồng bộ trạng thái mới từ CQT | KTHD-FR-004 |

---

## 4. Chi tiết Đặc tả Từng Endpoint

### 4.1. POST `/erp/qlhd-api/create-tax-account`
Đăng ký thông tin tài khoản Cổng HĐĐT của Tổng cục Thuế (`hoadondientu.gdt.gov.vn`) vào mSMI.

- **Headers:** `Content-Type: application/json`, `apiToken: <token>`
- **Request Body:**
```json
{
  "username": "0106026495-999",
  "password": "Password@123",
  "sync_invoices_sold": true,
  "init_sync_sold": "2026-10-01 00:00:00",
  "sync_invoices_purchase": true,
  "init_sync_purchase": "2026-10-01 00:00:00"
}
```
- **Tham số Request:**
  - `username` (string, required): Mã số thuế đăng nhập Cổng Thuế.
  - `password` (string, required): Mật khẩu tài khoản Cổng Thuế.
  - `sync_invoices_sold` (boolean): Có bật đồng bộ hóa đơn bán ra không.
  - `init_sync_sold` (datetime, required if sync_invoices_sold=true): Mốc ngày bắt đầu cào bán ra.
  - `sync_invoices_purchase` (boolean): Có bật đồng bộ hóa đơn mua vào không.
  - `init_sync_purchase` (datetime, required if sync_invoices_purchase=true): Mốc ngày bắt đầu cào mua vào.

- **Response Body (200 OK):**
```json
{
  "_id": "6475c67a9a7ce3cf085e86cc",
  "errorCode": "00",
  "message": "Thành công",
  "sync_invoices_sold": {
    "sync": true,
    "init_sync": "2026-10-01 00:00:00"
  },
  "sync_invoices_purchase": {
    "sync": true,
    "init_sync": "2026-10-01 00:00:00"
  }
}
```

---

### 4.2. POST `/erp/qlhd-api/edit-tax-account`
Cập nhật lại mật khẩu hoặc cấu hình ngày đồng bộ cho tài khoản Thuế đã tồn tại.

- **Headers:** `Content-Type: application/json`, `apiToken: <token>`
- **Request Body:**
```json
{
  "_id": "6475c67a9a7ce3cf085e86cc",
  "password": "NewPassword@123",
  "sync_invoices_sold": true,
  "init_sync_sold": "2026-10-01 00:00:00",
  "sync_invoices_purchase": true,
  "init_sync_purchase": "2026-10-01 00:00:00"
}
```
- **Response Body (200 OK):** Tương tự endpoint create-tax-account.

---

### 4.3. GET `/erp/qlhd-api/invoices`
Lấy danh sách hóa đơn theo điều kiện lọc và phân trang.

- **Headers:** `apiToken: <token>`
- **Query Parameters:**
  - `page` (int, default: 1): Số trang cần lấy.
  - `size` (int, required, max: 200, default: 50): Số bản ghi trên một trang.
  - `invoiceType` (string, required): `INPUT_ELECTRONIC_INVOICE` (Mua vào) hoặc `OUTPUT_ELECTRONIC_INVOICE` (Bán ra).
  - `invoiceReleaseDateFrom` (date, dd/MM/yyyy): Lọc theo ngày lập từ ngày.
  - `invoiceReleaseDateTo` (date, dd/MM/yyyy): Lọc theo ngày lập đến ngày.
  - `generalNotation` (string): Ký hiệu hóa đơn (ví dụ: `1C26TAA`).
  - `generalInvoiceNo` (int): Số hóa đơn cụ thể.
  - `sellerTaxNo` (string): Mã số thuế người bán.
  - `sellerName` (string): Tên bên bán.
  - `buyerTaxNo` (string): Mã số thuế người mua.
  - `buyerName` (string): Tên bên mua.

- **Response Body (200 OK):**
```json
{
  "status": 200,
  "message": "Tìm kiếm/Lọc danh sách Hóa đơn thành công. Tổng số lượng Hóa đơn tìm thấy là: 1 bản ghi.",
  "totalInvoice": 1,
  "totalPage": 1,
  "numberOfInvoicePerPage": 50,
  "currentPage": 1,
  "numberOfInvoiceInCurrentPage": 1,
  "listInvoice": [
    {
      "_id": "64b0f924e23cb20012ab9901",
      "nbmst": "0106026495",
      "nbten": "CÔNG TY TNHH HÓA ĐƠN ĐIỆN TỬ M-INVOICE",
      "khmshdon": 1,
      "khhdon": "1C26TYY",
      "shdon": 125,
      "tdlap": "2026-10-15T08:30:00Z",
      "nmmst": "0101234567",
      "nmten": "CÔNG TY TNHH THƯƠNG MẠI DỊCH VỤ AN BÌNH",
      "tgtcthue": 10000000,
      "tgtthue": 1000000,
      "tgtttbso": 11000000,
      "tgtttbchu": "Mười một triệu đồng chẵn",
      "tthai": 1,
      "type": "purchase",
      "hdhhdvu": [
        {
          "stt": "1",
          "ten": "Dịch vụ phần mềm quản lý hóa đơn",
          "dvtinh": "Gói",
          "sluong": 1,
          "dgia": 10000000,
          "thtien": 10000000,
          "tsuat": "10%",
          "tthue": 1000000
        }
      ]
    }
  ]
}
```

---

### 4.4. GET `/erp/qlhd-api/invoices/modified`
Lấy danh sách các hóa đơn phát sinh sự kiện thay đổi: Bị thay thế (`tthai = 4`), Bị điều chỉnh (`tthai = 5`), hoặc Bị hủy (`tthai = 6`).

- **Query Parameters:** Giống endpoint `/erp/qlhd-api/invoices`.
- **Response Body:** Trả về danh sách hóa đơn kèm mảng `hdonLquans` và các thông tin liên quan đến hóa đơn điều chỉnh/thay thế mới.

---

### 4.5. GET `/erp/qlhd-api/invoices/{id}/download/invoice.xml`
Tải về tệp XML nguyên bản của tờ hóa đơn.

- **Headers:** `apiToken: <token>`
- **Path Parameters:** `id` (`_id` MongoDB của hóa đơn).
- **Response:** XML Stream (`Content-Type: application/xml`).

---

### 4.6. GET `/erp/qlhd-api/invoices/{id}/download/invoice.html`
Tải bản thể hiện đồ họa mô phỏng mẫu in của tờ hóa đơn.

- **Headers:** `apiToken: <token>`
- **Path Parameters:** `id` (`_id` MongoDB của hóa đơn).
- **Response:** HTML Content (`Content-Type: text/html`).

---

### 4.7. GET `/erp/qlhd-api/invoices/{id}/warning`
Lấy chi tiết bảng điểm và các chỉ số kiểm tra tính hợp lệ của dữ liệu XML và rủi ro thuế.

- **Headers:** `apiToken: <token>`
- **Path Parameters:** `id` (`_id` của hóa đơn).
- **Response Body (200 OK):**
```json
{
  "status": "Cảnh báo",
  "origin_status": "Cảnh báo",
  "score": 1,
  "nkhdhle": 0,
  "nkhdtnlap": 0,
  "ttcks": 0,
  "xmlnven": 0,
  "cqtcma": 0,
  "cqtcmatnky": 0,
  "nbdksd": 0,
  "dcma": 0,
  "Date": "2026-10-15T09:00:00.000Z",
  "warning_Array": [
    "Sai tên đơn vị mua hàng so với tên đăng ký trên Cơ quan Thuế."
  ],
  "is_sco": false,
  "ignone_nbcks": false,
  "has_xml": true,
  "owner_match": 1,
  "owner_name": "CÔNG TY TNHH THƯƠNG MẠI DỊCH VỤ AN BÌNH"
}
```

---

### 4.8. GET `https://mst.minvoice.com.vn/api/System/SearchTaxCodeV2?tax={taxCode}`
Tra cứu thông tin và trạng thái hoạt động của Người nộp thuế từ cơ sở dữ liệu Tổng cục Thuế.

- **Query Parameters:** `tax` (Mã số thuế cần kiểm tra, ví dụ `0106026495`).
- **Response Body (200 OK):**
```json
{
  "ten_cty": "CÔNG TY TNHH HÓA ĐƠN ĐIỆN TỬ M-INVOICE",
  "dia_chi": "Số nhà 16, ngõ 269/1, đường Giáp Bát, Phường Giáp Bát, Quận Hoàng Mai, TP Hà Nội",
  "ma_so_thue": "0106026495",
  "masothue_id": "0106026495",
  "cqthuecap_tinh": "",
  "cqthue_ql": "Chi cục Thuế Quận Hoàng Mai",
  "nguoi_dai_dien": "Nguyễn Văn A",
  "ngay_thanh_lap": "2012-09-12",
  "tthai": "00",
  "ten_tthai": "NNT đang hoạt động (đã được cấp GCN ĐKT)"
}
```

---

## 5. Bảng Danh mục Mã lỗi (Error Codes)

### 5.1. Bảng Mã lỗi Đăng ký Tài khoản Thuế
| Mã lỗi (`errorCode`) | Thông điệp nghiệp vụ (`message`) |
|:---:|---|
| `00` | Thành công |
| `01` | Bạn chưa nhập đủ thông tin username/password |
| `02` | Username không đúng định dạng mã số thuế |
| `03` | Mật khẩu phải bao gồm chữ hoa, chữ thường, chữ số, ký tự đặc biệt |
| `04` | Không lấy được captcha từ CQT |
| `05` | Trang web của CQT gián đoạn, vui lòng thử lại sau |
| `06` | Không xác thực được Captcha, vui lòng thử lại |
| `07` | Tên đăng nhập hoặc mật khẩu không đúng |
| `08` | Ngày đồng bộ mua vào không được lớn hơn ngày đồng bộ đã khai báo trước đó |
| `09` | Ngày đồng bộ bán ra không được lớn hơn ngày đồng bộ đã khai báo trước đó |
| `10` | Ngày đồng bộ không được để trống khi có đồng bộ bán ra |
| `11` | Ngày đồng bộ không được để trống khi có đồng bộ mua vào |
| `12` | Tài khoản thuế đã tồn tại trên phần mềm |

### 5.2. Bảng Mã Trạng thái Pháp lý Mã số thuế (`tthai`)
| Mã `tthai` | Tên trạng thái Người nộp thuế | Mức độ Rủi ro Hệ thống |
|:---:|---|:---:|
| `00` | NNT đang hoạt động (đã được cấp GCN ĐKT) | An toàn (Xanh) |
| `01` | NNT ngừng hoạt động và đã hoàn thành thủ tục chấm dứt hiệu lực MST | Nguy cơ cao (Đỏ) |
| `02` | NNT đã chuyển cơ quan thuế quản lý | Bình thường |
| `03` | NNT ngừng hoạt động nhưng chưa hoàn thành thủ tục đóng MST | Cảnh báo (Cam) |
| `04` | NNT đang hoạt động (chưa đủ thông tin đăng ký thuế) | Cảnh báo (Cam) |
| `05` | NNT tạm ngừng hoạt động, kinh doanh có thời hạn | Cảnh báo (Cam) |
| `06` | NNT không hoạt động tại địa chỉ đã đăng ký (Doanh nghiệp bỏ trốn) | Nguy cơ cao (Đỏ) |
| `07` | NNT chờ làm thủ tục phá sản | Nguy cơ cao (Đỏ) |

