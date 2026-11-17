# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Kết nối Cơ quan Thuế (TAXACC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|-----------|------|---------|------------|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `TAXACC` — Kết nối Cơ quan Thuế & Đồng bộ HĐĐT  
> **Căn cứ tài liệu gốc:** `[mSMI] API Đăng ký, cập nhật tài khoản Thuế.docx`, `mSMI - Tài liệu HDSD...docx` (Mục 10), `mSMI - Tài liệu thiết kế CSDL.docx` (`externalaccounts`, `externalaccountsyncs`).  
> **Căn cứ mã nguồn thực tế:** `backend/service_crawl_invoices/`, `backend/routes/accounts/`, `backend/models/externalaccount.js`.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| ID | Tên Quy tắc | Mô tả Tóm tắt Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **TAXACC-BR-001** | Bảng Mã lỗi Đăng nhập CQT | Quá trình xác thực với Cổng HĐĐT Tổng cục Thuế bắt buộc phân loại phản hồi theo bảng mã lỗi chuẩn (`00` đến `07`) để hiển thị thông báo chính xác cho người dùng. | Validation | Must | TAXACC-UC-001<br>TAXACC-FR-001 |
| **TAXACC-BR-002** | Mã hóa Mật khẩu Tài khoản Thuế | Mật khẩu tài khoản Thuế của doanh nghiệp bắt buộc phải được mã hóa đối xứng AES-256-CBC trước khi lưu vào CSDL; chỉ giải mã trong RAM khi gửi request đăng nhập sang CQT. | Security / Crypto | Must | TAXACC-UC-001<br>TAXACC-FR-001 |
| **TAXACC-BR-003** | Giới hạn Khoảng ngày Đồng bộ | Mỗi phiên tra cứu gửi sang Cổng Thuế không vượt quá 31 ngày (`le - ge <= 31 days`); tự động băm nhỏ thành các cửa sổ con nếu người dùng chọn khoảng thời gian dài hơn. | Validation / Constraint | Must | TAXACC-UC-004<br>TAXACC-FR-003 |
| **TAXACC-BR-004** | Tách biệt Độc lập Luồng Mua / Bán | Cơ chế tự động đồng bộ chạy độc lập giữa luồng Hóa đơn Mua vào (`sync_invoices_purchase`) và Bán ra (`sync_invoices_sold`), lỗi luồng này không làm tắc nghẽn luồng kia. | Architecture / Isolation | Must | TAXACC-UC-003<br>TAXACC-FR-002 |
| **TAXACC-BR-005** | Ghi nhận Lịch sử & Audit Đồng bộ | Mọi phiên đồng bộ (tự động hoặc thủ công) đều bắt buộc ghi vết vào bảng `externalaccountsyncs` (thời gian, số lượng hóa đơn, độ trễ, trạng thái và lỗi chi tiết). | Audit / Traceability | Must | TAXACC-UC-005<br>TAXACC-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ (Detailed Business Rules Specification)

### 2.1. TAXACC-BR-001: Bảng Mã lỗi Đăng nhập CQT
Quá trình xác thực thông tin tài khoản thuế trên Cổng Dịch vụ Hóa đơn điện tử của Tổng cục Thuế bắt buộc chuẩn hóa theo bảng mã lỗi:

| Mã lỗi (`errorCode`) | Thông điệp nghiệp vụ hiển thị (`message`) | Hướng xử lý kỹ thuật & Hướng dẫn người dùng |
|:---:|---|---|
| `00` | Thành công | Lưu thông tin tài khoản vào bảng `externalaccounts`, kích hoạt cơ chế đồng bộ tự động. |
| `01` | Bạn chưa nhập đủ thông tin username/password | Bắt buộc điền đầy đủ cả Mã số thuế và Mật khẩu trên biểu mẫu. |
| `02` | Username không đúng định dạng mã số thuế | Kiểm tra định dạng MST: 10 chữ số (doanh nghiệp) hoặc 13 chữ số có gạch nối `-` (chi nhánh). |
| `03` | Mật khẩu phải bao gồm chữ hoa, chữ thường, chữ số, ký tự đặc biệt | Yêu cầu người dùng kiểm tra lại quy chuẩn mật khẩu theo chính sách của Tổng cục Thuế. |
| `04` | Không lấy được captcha từ CQT | Tự động thử lại kết nối 3 lần qua Proxy IP khác hoặc thông báo kiểm tra đường truyền CQT. |
| `05` | Trang web của CQT gián đoạn, vui lòng thử lại sau | Cổng Thuế bảo trì hoặc quá tải; lưu trạng thái tạm dừng và lập lịch thử lại sau 15 phút. |
| `06` | Không xác thực được Captcha, vui lòng thử lại | Tự động lấy lại Captcha SVG/hình ảnh mới và gửi lại cho module OCR/giải mã. |
| `07` | Tên đăng nhập hoặc mật khẩu không đúng | Thông báo người dùng kiểm tra đăng nhập trực tiếp trên trang `hoadondientu.gdt.gov.vn`. |

---

### 2.2. TAXACC-BR-002: Mã hóa Mật khẩu Tài khoản Thuế (AES-256-CBC)
* **Statement**: Mật khẩu tài khoản Thuế của người dùng là thông tin nhạy cảm cấp độ tối mật, bắt buộc phải được mã hóa trước khi ghi vào CSDL và tuyệt đối không bao giờ xuất hiện ở dạng plain text trong nhật ký hệ thống.
* **Condition**: Kích hoạt khi người dùng tạo mới tài khoản thuế (`POST /external-accounts`) hoặc cập nhật mật khẩu (`PUT /external-accounts/:id`).
* **Action / Algorithm**:
  1. Sinh ngẫu nhiên Vector khởi tạo `iv` (16 bytes) cho mỗi lần mã hóa.
  2. Áp dụng thuật toán mã hóa đối xứng `AES-256-CBC` kết hợp với khóa bí mật ứng dụng (`APP_SECRET_KEY`) cấu hình trong biến môi trường máy chủ.
  3. Lưu chuỗi mật mã hóa (`encryptedPassword`) và chuỗi `iv` vào bảng `externalaccounts`.
  4. Khi Worker cào hóa đơn cần gửi yêu cầu đăng nhập sang CQT:
     - Đọc dữ liệu từ DB, giải mã trong RAM.
     - Sử dụng mật khẩu trong bộ nhớ đệm để gửi payload đăng nhập lấy Token JWT từ CQT.
     - Sau khi nhận Token hoặc gặp lỗi, lập tức xóa biến plain text khỏi bộ nhớ RAM để chống rò rỉ (Garbage Collection).
  5. Cấm tuyệt đối việc in log mật khẩu (`console.log`, log file, Winston logger) ở bất kỳ tầng nào.
* **Schema / Fields**: `externalaccounts.password` (chuỗi hex/base64 mã hóa), `externalaccounts.iv`.

---

### 2.3. TAXACC-BR-003: Giới hạn Khoảng ngày Đồng bộ (31-day Window Constraint)
* **Statement**: Cổng HĐĐT của Tổng cục Thuế áp đặt giới hạn kỹ thuật: Mỗi lượt request tra cứu danh sách hóa đơn chỉ được phép có khoảng cách tối đa 31 ngày giữa ngày bắt đầu (`ge`) và ngày kết thúc (`le`).
* **Condition**: Khi hệ thống thực hiện đồng bộ định kỳ nền hoặc khi người dùng yêu cầu "Đồng bộ thủ công" cho một khoảng thời gian dài (theo Quý hoặc theo Năm).
* **Action / Algorithm**:
  1. Kiểm tra điều kiện:
     $$\Delta t = \text{Date}(le) - \text{Date}(ge)$$
  2. Nếu $\Delta t \le 31\text{ ngày}$: Gửi 1 request tra cứu đơn lẻ với tham số `ge` và `le`.
  3. Nếu $\Delta t > 31\text{ ngày}$: Hệ thống tự động kích hoạt thuật toán băm nhỏ cửa sổ thời gian (Date Chunking):
     - Chia khoảng thời gian lớn thành chuỗi các khoảng thời gian con liên tiếp $[ge_1, le_1], [ge_2, le_2], \dots, [ge_n, le_n]$, mỗi khoảng $\le 31$ ngày.
     - Ví dụ: Yêu cầu đồng bộ Quý 1/2026 (01/01/2026 – 31/03/2026) được băm thành:
       * Lượt 1: `01/01/2026` đến `31/01/2026` (31 ngày)
       * Lượt 2: `01/02/2026` đến `28/02/2026` (28 ngày)
       * Lượt 3: `01/03/2026` đến `31/03/2026` (31 ngày)
     - Mỗi lượt tra cứu con được thực hiện tuần tự với khoảng nghỉ ngắt quãng (jitter delay 1.5–3 giây) để chống nghẽn và vượt qua bộ lọc Rate Limit của Cổng Thuế.
* **Exception**: Nếu người dùng nhập ngày kết thúc trước ngày bắt đầu (`le < ge`), hệ thống chặn ngay tại tầng giao diện và báo lỗi `ERR_INVALID_DATE_RANGE`.
* **Schema / Fields**: Query parameters `ge` (Greater or Equal), `le` (Less or Equal).

---

### 2.4. TAXACC-BR-004: Tách biệt Độc lập Luồng Mua / Bán (Process Isolation)
* **Statement**: Quá trình tự động đồng bộ hóa đơn Mua vào (`purchase`) và Bán ra (`sold`) phải chạy độc lập trên các tiến trình runtime PM2 riêng biệt; sự cố gián đoạn của luồng này tuyệt đối không được ảnh hưởng đến luồng kia.
* **Condition**: Khi hệ thống khởi chạy các cron worker định kỳ ngầm.
* **Action / Algorithm**:
  1. Hệ thống phân chia thành 2 cron job độc lập:
     - Worker 1: `cron_sync_invoices_purchase` chuyên quét hóa đơn người mua.
     - Worker 2: `cron_sync_invoices_sold` chuyên quét hóa đơn người bán.
  2. Mỗi luồng quản lý con trỏ thời gian đồng bộ độc lập trong bảng `externalaccounts`:
     - `last_sync_purchase`: Mốc thời gian hoàn tất đồng bộ mua vào gần nhất.
     - `last_sync_sold`: Mốc thời gian hoàn tất đồng bộ bán ra gần nhất.
  3. Khóa chống chạy trùng lặp (Distributed Mutex): Mỗi tài khoản có 2 cờ lock riêng biệt `is_syncing_purchase` và `is_syncing_sold`. Nếu luồng mua đang chạy, luồng bán vẫn được phép chạy song song mà không xung đột khóa.
  4. Cô lập lỗi (Fault Isolation): Nếu Cổng Thuế phát sinh lỗi ở tab hóa đơn mua vào (ví dụ lỗi 500 do tải lượng lớn), worker mua vào chuyển sang trạng thái chờ backoff, trong khi worker bán ra vẫn tiếp tục cào hóa đơn bán ra bình thường.
* **Schema / Fields**: `externalaccounts.last_sync_purchase`, `externalaccounts.last_sync_sold`, `externalaccounts.is_syncing_purchase`, `externalaccounts.is_syncing_sold`.

---

### 2.5. TAXACC-BR-005: Ghi nhận Lịch sử & Audit Đồng bộ (Sync Audit Logging)
* **Statement**: Mọi phiên đồng bộ phát sinh trong hệ thống (tự động theo lịch hoặc thủ công do người dùng bấm nút) đều bắt buộc phải ghi nhận bản ghi nhật ký chi tiết vào bảng `externalaccountsyncs` để phục vụ giám sát vận hành và đối soát SLA.
* **Condition**: Khi bất kỳ phiên cào hóa đơn nào được khởi tạo cho tới khi kết thúc hoàn toàn.
* **Action / Algorithm**:
  1. Khi bắt đầu phiên đồng bộ: Khởi tạo bản ghi nhật ký ghi nhận các thông tin:
     * **Tài khoản đơn vị:** Mã định danh đơn vị/doanh nghiệp đang đồng bộ.
     * **Chiều dữ liệu:** Mua vào (`purchase`) hoặc Bán ra (`sold`).
     * **Phương thức kích hoạt:** Tự động theo lịch (`cron`) hoặc Thao tác thủ công (`manual`).
     * **Thời gian bắt đầu:** Mốc ngày giờ bắt đầu gửi lệnh tới CQT.
     * **Khoảng thời gian tra cứu:** Từ ngày (`ge`) đến ngày (`le`).
     * **Trạng thái ban đầu:** Đang xử lý (`running`).
  2. Trong quá trình đồng bộ: Tích lũy các chỉ số thống kê:
     - `total_crawled`: Tổng số hóa đơn quét được từ Cổng Thuế.
     - `total_inserted`: Số hóa đơn mới được thêm vào DB.
     - `total_updated`: Số hóa đơn cũ được cập nhật trạng thái/chi tiết.
  3. Khi kết thúc phiên đồng bộ:
     - Cập nhật `end_time = new Date()`.
     - Tính toán `duration_ms = end_time - start_time`.
     - Nếu thành công: Cập nhật `status = 'success'`.
     - Nếu thất bại: Cập nhật `status = 'error'`, lưu mã lỗi và nội dung chi tiết vào trường `error_message`.
* **Schema / Fields**: Bảng `externalaccountsyncs.{account, user, type, trigger_by, start_time, end_time, duration_ms, status, total_crawled, total_inserted, total_updated, error_message}`.
* **Source**: `mSMI - Tài liệu thiết kế CSDL.docx` (Bảng `externalaccountsyncs`), `backend/service_crawl_invoices/index.js`.
