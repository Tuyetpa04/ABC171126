# DOC-04 — Quy tắc Nghiệp vụ Phân hệ Quản lý Hóa đơn (INVOIC)

| Phiên bản | Ngày | Tác giả | Trạng thái |
|:---:|:---:|:---:|:---:|
| 2.0 | 2026-11-08 | BA | Baseline Unified v2.0 |

> **Phân hệ:** `INVOIC` — Quản lý Hóa đơn Mua vào / Bán ra  
> **Căn cứ tài liệu gốc:** `mSMI - Tài liệu HDSD...docx` (Mục 6), `mSMI - Tài liệu thiết kế CSDL.docx` (Danh mục hóa đơn, chứng từ đính kèm, nhãn phân loại), `App hóa đơn vào_ ra SMI.xlsx`, `[mSMI] Document API.docx`.  
> **Căn cứ pháp lý:** Nghị định 123/2020/NĐ-CP, Thông tư 78/2021/TT-BTC, Thông tư 80/2021/TT-BTC.

---

## 1. Danh mục Quy tắc Nghiệp vụ (Business Rules Register)

| ID | Tên Quy tắc | Mô tả Tóm tắt Rule | Loại Rule | Mức ưu tiên | Trace (UC / FR) |
|:---:|---|---|:---:|:---:|:---:|
| **INVOIC-BR-001** | Khóa định danh Hóa đơn & Chống trùng lặp | Định danh duy nhất bằng tổ hợp 4 tiêu chí nghiệp vụ: Mã số thuế người bán, Ký hiệu mẫu số, Ký hiệu hóa đơn, Số hóa đơn. Áp dụng cơ chế hợp nhất bảo toàn nguyên vẹn dữ liệu kế toán khi đồng bộ lại. | Validation / Integrity | Must | INVOIC-UC-001<br>INVOIC-FR-001 |
| **INVOIC-BR-002** | Phân loại Chiều Hóa đơn & Máy tính tiền | Phân định rạch ròi 4 luồng dữ liệu độc lập: Hóa đơn Mua vào, Hóa đơn Bán ra, Hóa đơn Mua vào máy tính tiền, Hóa đơn Bán ra máy tính tiền. | Validation / Security | Must | INVOIC-UC-001<br>INVOIC-FR-001 |
| **INVOIC-BR-003** | Quy tắc Kê khai Thuế & Khóa kỳ Báo cáo | Khi gán ngày kê khai, hóa đơn được chốt vào kỳ tính thuế (tháng/quý) và tính vào Tờ khai 01/GTGT. Khi hủy kê khai, hóa đơn tự động rút khỏi bảng kê, đảm bảo tính chuẩn xác cho số liệu thuế. | Calculation / State | Must | INVOIC-UC-006<br>INVOIC-FR-005 |
| **INVOIC-BR-004** | Đóng gói Tải hàng loạt & Truy xuất Đa tầng | Giới hạn tối đa 100 hóa đơn/lần tải nén ZIP; áp dụng quy trình 5 bước tìm nạp bản thể hiện PDF để đảm bảo kế toán luôn tải đủ cả tệp XML gốc và tệp PDF thể hiện. | Processing / Output | Must | INVOIC-UC-004<br>INVOIC-FR-003 |
| **INVOIC-BR-005** | Quyền Ẩn Cảnh báo & Lưu vết Kiểm toán | Kế toán viên có quyền ẩn cảnh báo rủi ro sau khi đã xác minh thực tế; hệ thống bắt buộc lưu vết trạng thái rủi ro ban đầu để phục vụ thanh tra, kiểm toán thuế. | Authorization / Audit | Should | INVOIC-UC-001<br>INVOIC-FR-006 |
| **INVOIC-BR-006** | Giới hạn & Kiểm soát Tệp Đính kèm Chứng từ | Cho phép đính kèm hồ sơ chứng từ (PDF, PNG, JPG, DOCX, XLSX dung lượng $\le$ 10MB); áp dụng tiêu chuẩn mã hóa lưu trữ an toàn để bảo mật số liệu tài chính doanh nghiệp. | Security / Storage | Should | INVOIC-UC-007<br>INVOIC-FR-006 |
| **INVOIC-BR-007** | Đồng bộ Chi tiết theo Nhu cầu Tức thời | Vận hành đồng bộ 2 giai đoạn (Tổng quan $\rightarrow$ Chi tiết); khi người dùng mở xem hoặc tải hóa đơn chưa hoàn tất chi tiết, hệ thống tự động ưu tiên xử lý ngay lập tức trong vòng 5 giây. | Optimization / Queue | Must | INVOIC-UC-004<br>INVOIC-FR-003 |
| **INVOIC-BR-008** | Quản lý & Gán Nhãn Phân loại Chi phí | Cho phép doanh nghiệp tự định nghĩa danh mục nhãn màu và gán nhiều nhãn cho hóa đơn phục vụ phân loại hạch toán chi phí nội bộ và trích xuất báo cáo quản trị. | Classification / Metadata | Should | INVOIC-UC-005<br>INVOIC-FR-004 |

---

## 2. Chi tiết Quy tắc Nghiệp vụ (Detailed Business Rules Specification)

### 2.1. INVOIC-BR-001 — Khóa định danh Hóa đơn & Chống trùng lặp (Unique Key & Deduplication)
* **Phát biểu quy tắc (Statement):** Mỗi hóa đơn điện tử trong phạm vi một đơn vị/tài khoản doanh nghiệp phải là duy nhất. Tuyệt đối không tạo bản ghi nhân bản khi hệ thống đồng bộ định kỳ từ Cơ quan Thuế hoặc khi tiếp nhận tệp từ nhiều nguồn hộp thư điện tử khác nhau.
* **Điều kiện áp dụng (Condition):** Kích hoạt khi phát sinh lệnh đồng bộ dữ liệu từ Cổng Thuế (đồng bộ tổng quan hoặc cào chi tiết), bóc tách tệp XML từ Hộp thư tự động (`MAILBX`), hoặc khi người dùng tra cứu trực tiếp từ Cổng Tổng cục Thuế (`KIEMTRAHD`).
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Tổ hợp định danh duy nhất (Unique Business Key):**
     Một tờ hóa đơn được định danh độc bản dựa trên sự kết hợp của 4 thông tin nghiệp vụ:
     * **Mã số thuế bên bán** (`nbmst`): Mã số thuế của người xuất hóa đơn.
     * **Ký hiệu mẫu số hóa đơn** (`khmshdon`): Ví dụ `1` (Hóa đơn GTGT), `2` (Hóa đơn bán hàng).
     * **Ký hiệu hóa đơn** (`khhdon`): Ví dụ `C24TAV`, `1C24MAA`.
     * **Số hóa đơn** (`shdon`): Chuẩn hóa dưới dạng số tự nhiên (loại bỏ các số 0 vô nghĩa ở đầu, ví dụ `00000125` $\rightarrow$ `125`).
  2. **Cơ chế Hợp nhất & Bảo toàn Dữ liệu (Data Merge & Preservation):**
     * **Trường hợp Hóa đơn đã tồn tại trong hệ thống:**
       * Cập nhật thông tin nghiệp vụ mới nhất từ Cơ quan Thuế (như ngày ký duyệt, trạng thái hóa đơn mới: Bị hủy, Bị thay thế, Bị điều chỉnh).
       * Cập nhật danh sách dòng hàng hóa chi tiết nếu bản ghi trước đó mới chỉ có thông tin tổng quan.
       * Cập nhật các liên kết tệp tài liệu: File PDF gốc, File XML dữ liệu, File bản in HTML.
       * **Bảo toàn nguyên vẹn 100% dữ liệu kế toán đã can thiệp:** Giữ nguyên Ngày kê khai thuế, Ghi chú nghiệp vụ nội bộ, Nhãn phân loại chi phí, và Trạng thái ẩn cảnh báo rủi ro do người dùng đã thao tác.
     * **Trường hợp Hóa đơn mới:**
       * Khởi tạo hóa đơn vào kho dữ liệu của đơn vị.
       * Thiết lập trạng thái kê khai mặc định là: **Chưa kê khai**.
       * Tự động đưa hóa đơn vào quy trình rà soát rủi ro thuế (đối soát trạng thái người bán, kiểm tra danh sách doanh nghiệp rủi ro cao).
* **Xử lý ngoại lệ (Exception Handling):**
  - Nếu hóa đơn đầu vào bị thiếu một trong các thông tin định danh bắt buộc (thiếu MST người bán hoặc thiếu số hóa đơn), hệ thống từ chối tiếp nhận và ghi nhận nhật ký lỗi đồng bộ để kế toán viên kiểm tra lại nguồn cấp dữ liệu.
* **Căn cứ pháp lý & Nguồn:**
  - Điều 10 Nghị định 123/2020/NĐ-CP (Quy định về ký hiệu mẫu số, ký hiệu hóa đơn và số hóa đơn).
  - Tài liệu thiết kế CSDL mSMI (Bảng hóa đơn `invoiceraws`).

---

### 2.2. INVOIC-BR-002 — Phân loại Chiều Hóa đơn & Máy tính tiền (Invoice Direction & SCO)
* **Phát biểu quy tắc (Statement):** Hệ thống phân định rạch ròi 4 nhóm luồng hóa đơn độc lập, đảm bảo không nhầm lẫn số liệu kế toán đầu vào - đầu ra và tờ khai thuế của doanh nghiệp:
  1. Hóa đơn Mua vào thông thường (`purchase`)
  2. Hóa đơn Bán ra thông thường (`sold`)
  3. Hóa đơn Mua vào khởi tạo từ máy tính tiền (`purchase_sco`)
  4. Hóa đơn Bán ra khởi tạo từ máy tính tiền (`sold_sco`)
* **Điều kiện áp dụng (Condition):** Khi đồng bộ tự động từ các phân hệ riêng biệt của Cổng Thuế hoặc khi người dùng tải tệp XML lên hệ thống.
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Quy tắc phân định Chiều Hóa đơn (Mua vào / Bán ra):**
     * Nếu **Mã số thuế bên bán** trùng khớp với Mã số thuế của đơn vị đang quản lý: Tự động phân loại luồng **Hóa đơn Bán ra**.
     * Nếu **Mã số thuế bên mua** trùng khớp với Mã số thuế của đơn vị đang quản lý: Tự động phân loại luồng **Hóa đơn Mua vào**.
     * *Trường hợp giao dịch nội bộ (giữa trụ sở chính và chi nhánh có cùng MST 10 số):* Hệ thống căn cứ theo mã số chi nhánh 3 số ở đuôi hoặc nguồn tiếp nhận từ phân hệ tra cứu tương ứng của Cổng Thuế để xếp đúng luồng.
  2. **Quy tắc nhận diện Hóa đơn Khởi tạo từ Máy tính tiền (SCO):**
     * Căn cứ ký tự thứ 5 của Ký hiệu hóa đơn: Nếu mang ký tự `M` (ví dụ: `1C24MAA`, `2C24MTT`) theo quy chuẩn Thông tư 78/2021/TT-BTC, hoặc có cờ xác nhận máy tính tiền từ Cơ quan Thuế: Hệ thống tự động phân loại vào nhóm **Máy tính tiền** (`purchase_sco` hoặc `sold_sco`).
     * Hóa đơn máy tính tiền được quản lý theo cơ chế riêng: Không bắt buộc có chữ ký số của người bán khi lập, mã của CQT là chuỗi ký tự cấp tự động theo dải số đã đăng ký.
  3. **Kiểm soát độc lập luồng hiển thị:**
     * Mọi chức năng tìm kiếm, xem danh sách, kết xuất báo cáo và đối soát rủi ro đều phân tách riêng biệt theo từng chiều hóa đơn, tránh tuyệt đối việc tính nhầm chi phí mua vào sang doanh thu bán ra.
* **Căn cứ pháp lý & Nguồn:**
  - Điều 8 Thông tư 78/2021/TT-BTC (Quy định về hóa đơn điện tử khởi tạo từ máy tính tiền có mã của cơ quan thuế).

---

### 2.3. INVOIC-BR-003 — Quy tắc Kê khai Thuế & Khóa kỳ Báo cáo (`nkekhai`)
* **Phát biểu quy tắc (Statement):** Một tờ hóa đơn chỉ được phân bổ vào Bảng kê mua vào/bán ra và Tờ khai thuế GTGT Mẫu 01/GTGT khi đã được kế toán viên xác nhận Ngày kê khai thuế hợp lệ.
* **Điều kiện áp dụng (Condition):** Kế toán viên chọn một hoặc nhiều hóa đơn trên danh sách và thực hiện chức năng "Gán ngày kê khai" hoặc "Hủy ngày kê khai".
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Nghiệp vụ Gán Ngày Kê khai:**
     * Kế toán viên chọn ngày kê khai (ví dụ ngày `31/03/2026`).
     * Hệ thống tự động xác định kỳ tính thuế tương ứng:
       * Theo Tháng (ví dụ: Tháng 03/2026) nếu doanh nghiệp kê khai theo tháng.
       * Theo Quý (ví dụ: Quý 1/2026) nếu doanh nghiệp kê khai theo quý.
     * Trạng thái hóa đơn chuyển thành: **Đã kê khai** (hiển thị kèm nhãn kỳ tính thuế cụ thể trên giao diện).
     * Số liệu tiền hàng, tiền thuế và thuế suất của hóa đơn tự động được cộng dồn vào các chỉ tiêu tương ứng trên Bảng kê và Tờ khai 01/GTGT của kỳ đó.
  2. **Nghiệp vụ Hủy Ngày Kê khai:**
     * Kế toán viên thực hiện chức năng "Hủy kê khai" đối với hóa đơn đã chọn.
     * Trạng thái hóa đơn chuyển về: **Chưa kê khai**.
     * Hóa đơn lập tức bị loại bỏ khỏi Bảng kê và Tờ khai 01/GTGT của kỳ tính thuế đã gán trước đó, đảm bảo không làm sai lệch nghĩa vụ thuế GTGT phải nộp hoặc số thuế còn được khấu trừ chuyển kỳ sau.
* **Xử lý ngoại lệ (Exception Handling):**
  - Nếu hóa đơn có trạng thái trên Cơ quan Thuế là "Đã bị hủy" hoặc "Bị thay thế", hệ thống bắt buộc hiển thị hộp thoại cảnh báo rủi ro loại trừ chi phí và yêu cầu kế toán trưởng xác nhận trước khi cho phép gán ngày kê khai.
* **Căn cứ pháp lý & Nguồn:**
  - Luật Quản lý Thuế số 38/2019/QH14, Thông tư 80/2021/TT-BTC (Hướng dẫn lập tờ khai thuế GTGT Mẫu 01/GTGT).

---

### 2.4. INVOIC-BR-004 — Đóng gói Tải hàng loạt & Truy xuất Đa tầng (Batch Download ZIP)
* **Phát biểu quy tắc (Statement):** Khi kế toán viên yêu cầu tải hàng loạt hóa đơn, hệ thống hỗ trợ gom tối đa 100 hóa đơn/lần vào một tệp nén ZIP duy nhất; áp dụng quy trình 5 bước tìm nạp bản thể hiện để tối đa hóa tỷ lệ tải thành công cả tệp XML gốc và tệp PDF thể hiện.
* **Điều kiện áp dụng (Condition):** Kế toán viên chọn các hóa đơn trên danh sách và bấm chọn "Tải nhiều XML", "Tải nhiều PDF" hoặc "Tải tệp ZIP tổng hợp".
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Giới hạn số lượng:**
     * Mỗi lần tải hàng loạt cho phép chọn tối đa **100 hóa đơn**.
     * Trường hợp danh sách chọn vượt quá 100 hóa đơn, hệ thống thông báo yêu cầu người dùng thu hẹp bộ lọc hoặc tải thành nhiều đợt để đảm bảo tốc độ đường truyền và tính toàn vẹn của tệp nén.
  2. **Quy chuẩn đặt tên tệp hóa đơn bên trong file ZIP:**
     * Định dạng chuẩn: `[MST_Người_Bán]_[MST_Người_Mua]_[Ký_Hiệu]_[Số_Hóa_Đơn].[Định_Dạng]`
     * Giúp kế toán dễ dàng sắp xếp, tìm kiếm chứng từ theo từng nhà cung cấp và từng số hóa đơn.
  3. **Quy trình 5 bước tìm nạp Bản thể hiện PDF:**
     * *Bước 1 (Kho nhà cung cấp giải pháp):* Kiểm tra tệp PDF gốc lưu trữ từ nhà cung cấp giải pháp HĐĐT đã đồng bộ về.
     * *Bước 2 (Hộp thư điện tử):* Nếu bước 1 chưa có, kiểm tra tệp PDF đính kèm bóc tách từ email nhận hóa đơn của doanh nghiệp.
     * *Bước 3 (Kho lưu trữ đám mây hệ thống):* Kiểm tra bản thể hiện PDF đã được hệ thống lưu trữ tại phân vùng dữ liệu đơn vị.
     * *Bước 4 (Hồ sơ chứng từ đính kèm):* Tìm kiếm tệp bản thể hiện PDF do người dùng tải lên bổ sung thủ công.
     * *Bước 5 (Tạo tức thời từ dữ liệu chuẩn):* Nếu cả 4 bước trên đều không tìm thấy tệp sẵn có nhưng hóa đơn đã có dữ liệu dòng hàng chi tiết từ Cơ quan Thuế, hệ thống tự động chuyển đổi bản thể hiện chuẩn sang tệp PDF tương đương để đưa vào file nén.
  4. **Xử lý khi thiếu tệp:**
     * Nếu một số hóa đơn chưa tải được file chi tiết do Cổng Thuế chưa phản hồi, hệ thống vẫn nén toàn bộ các hóa đơn đã có sẵn.
     * Tự động tạo tệp văn bản `DanhSachHoaDonChuaTaiDuoc.txt` đặt tại thư mục gốc của file ZIP, liệt kê rõ số hóa đơn, ngày lập và lý do chưa tải được để kế toán tiện theo dõi và tải bù sau.
* **Nguồn tham chiếu:**
  - `App hóa đơn vào_ ra SMI.xlsx` (Sheet Hóa đơn), Hướng dẫn sử dụng phần mềm quản lý hóa đơn mSMI.

---

### 2.5. INVOIC-BR-005 — Quyền Ẩn Cảnh báo & Lưu vết Kiểm toán (`hide_status`)
* **Phát biểu quy tắc (Statement):** Kế toán viên phụ trách có quyền ẩn cảnh báo rủi ro đối với những hóa đơn đã được đơn vị kiểm tra, xác minh chứng từ thực tế hợp lệ; hệ thống bắt buộc lưu vết trạng thái rủi ro ban đầu để phục vụ công tác thanh tra, kiểm toán thuế sau này.
* **Điều kiện áp dụng (Condition):** Kế toán viên mở xem chi tiết hóa đơn có cảnh báo rủi ro (cảnh báo vàng/đỏ) và thực hiện thao tác "Ẩn cảnh báo".
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Quy tắc phân quyền thao tác:**
     * Chỉ người dùng sở hữu tài khoản quản lý đơn vị hoặc nhân viên được cấp quyền quản lý hóa đơn của MST đó mới được thực hiện chức năng này.
  2. **Quy trình Chuyển đổi Trạng thái Cảnh báo:**
     * **Khi kế toán chọn "Ẩn cảnh báo":**
       * Hệ thống lưu trữ trạng thái rủi ro hiện tại (ví dụ: *"Người bán tạm ngừng kinh doanh"*, *"Hóa đơn có rủi ro cao theo danh sách 1.500 DN"*) vào mục lịch sử cảnh báo gốc.
       * Chuyển trạng thái hiển thị của hóa đơn trên danh sách làm việc thành: **Hợp lệ (Đã xác minh)**.
       * Hóa đơn được đưa ra khỏi danh sách cần xử lý gấp trên màn hình Giám sát Rủi ro Thuế.
     * **Khi kế toán chọn "Bỏ ẩn cảnh báo":**
       * Hệ thống khôi phục nguyên trạng mức độ cảnh báo rủi ro ban đầu từ dữ liệu lịch sử gốc.
       * Hóa đơn xuất hiện trở lại trên danh sách theo dõi rủi ro để tiếp tục kiểm tra, xử lý.
  3. **Lưu vết phục vụ kiểm toán:**
     * Mọi thao tác ẩn/bỏ ẩn đều được ghi lại ngày giờ, tên người thực hiện và lý do xác minh để phục vụ việc giải trình số liệu khi cơ quan thuế quyết toán.
* **Nguồn tham chiếu:**
  - `Tính năng phần mềm quản lý rủi ro hóa đơn mSMI.docx`, `App hóa đơn vào_ ra SMI.xlsx`.

---

### 2.6. INVOIC-BR-006 — Giới hạn & Kiểm soát Tệp Đính kèm Chứng từ
* **Phát biểu quy tắc (Statement):** Hồ sơ chứng từ đính kèm theo hóa đơn (Hợp đồng kinh tế, Biên bản nghiệm thu bàn giao, Phiếu xuất kho, Giấy báo nợ, Ủy nhiệm chi) phải được kiểm soát chặt chẽ về định dạng, dung lượng tối đa và lưu trữ bảo mật để đảm bảo an toàn bí mật số liệu tài chính doanh nghiệp.
* **Điều kiện áp dụng (Condition):** Kế toán viên tải tệp lên hoặc xóa tệp đính kèm tại màn hình xem chi tiết hóa đơn.
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Quy định về Định dạng & Kích thước Hợp lệ:**
     * **Định dạng cho phép:** Tệp văn bản PDF (`.pdf`), tệp hình ảnh chứng từ (`.png`, `.jpg`, `.jpeg`), tệp bảng tính/soạn thảo (`.docx`, `.xlsx`).
     * **Chặn tuyệt đối:** Các tệp thực thi, tệp mã lệnh có nguy cơ mất an toàn thông tin (`.exe`, `.bat`, `.cmd`, `.sh`, `.js`).
     * **Dung lượng tối đa:** Không quá **10 MB** cho mỗi tệp tải lên.
     * **Số lượng tối đa:** Cho phép lưu trữ tối đa **10 tệp đính kèm** trên cùng một tờ hóa đơn.
  2. **Tiêu chuẩn An toàn Lưu trữ (At-Rest Security):**
     * Toàn bộ tệp chứng từ khi lưu trữ vào kho dữ liệu hệ thống đều được áp dụng cơ chế mã hóa dữ liệu an toàn theo chuẩn công nghiệp để chống lộ lọt thông tin khi bị sao chép trái phép.
     * Chỉ những người dùng có thẩm quyền truy cập hợp lệ mới được phép xem và tải tệp đã giải mã về máy tính.
* **Xử lý ngoại lệ (Exception Handling):**
  - Nếu tệp tải lên vượt quá giới hạn 10 MB hoặc sai định dạng cho phép, hệ thống từ chối tiếp nhận và thông báo rõ nguyên nhân cho người dùng trên màn hình thao tác.
* **Nguồn tham chiếu:**
  - `mSMI - Tài liệu thiết kế CSDL.docx` (Bảng tài liệu đính kèm), Quy chuẩn bảo mật hệ thống mSMI.

---

### 2.7. INVOIC-BR-007 — Đồng bộ Chi tiết theo Nhu cầu Tức thời (`priority`)
* **Phát biểu quy tắc (Statement):** Hệ thống triển khai quy trình đồng bộ hóa đơn 2 giai đoạn để tối ưu hiệu năng và tránh bị Cổng Thuế chặn truy cập: Giai đoạn 1 đồng bộ thông tin tiêu đề hóa đơn, Giai đoạn 2 đồng bộ chi tiết danh mục hàng hóa. Khi kế toán viên phát sinh nhu cầu xem hoặc tải một hóa đơn chưa hoàn tất chi tiết, hệ thống tự động ưu tiên xử lý tức thì trong vòng 5 giây.
* **Điều kiện áp dụng (Condition):** Khi kế toán viên bấm mở xem chi tiết một hóa đơn, xem bản in, hoặc bấm tải tệp mà hóa đơn đó chưa có sẵn danh sách dòng hàng hóa chi tiết.
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Giai đoạn 1 (Đồng bộ danh sách tổng quan):**
     * Trong các phiên quét tự động định kỳ, hệ thống ưu tiên quét nhanh toàn bộ tiêu đề hóa đơn để kế toán nắm được số lượng, tổng tiền và ngày lập mới nhất từ Cơ quan Thuế.
  2. **Giai đoạn 2 (Đồng bộ chi tiết tự động):**
     * Các hóa đơn được đưa vào hàng đợi nền để robot tự động tải lần lượt tệp chi tiết từng dòng hàng.
  3. **Cơ chế Ưu tiên Tức thời theo Nhu cầu Người dùng:**
     * Khi người dùng nhấp chuột vào một hóa đơn cụ thể trên màn hình làm việc, hệ thống nhận diện đây là hóa đơn đang có nhu cầu xử lý gấp.
     * Hóa đơn này lập tức được đưa lên vị trí ưu tiên cao nhất trong hàng đợi xử lý.
     * Robot đồng bộ ưu tiên kết nối tới Cơ quan Thuế để lấy đầy đủ chi tiết dòng hàng trong vòng không quá **5 giây**.
     * Màn hình tự động cập nhật hiển thị đầy đủ bảng hàng hóa mà người dùng không cần phải tải lại trang.
* **Xử lý ngoại lệ (Exception Handling):**
  - Nếu Cổng Thuế bị nghẽn bất thường dẫn đến gián đoạn kết nối, hệ thống ghi nhận trạng thái chờ xử lý lại và cho phép kế toán viên nhập mã tra cứu hóa đơn để tải trực tiếp từ cổng của nhà cung cấp giải pháp HĐĐT tương ứng.
* **Nguồn tham chiếu:**
  - Hướng dẫn vận hành hệ thống mSMI, `ADR-004: Tách Hàng Đợi Đồng Bộ Header Và Detail Hóa Đơn`.

---

### 2.8. INVOIC-BR-008 — Quản lý & Gán Nhãn Phân loại Chi phí (`tags`)
* **Phát biểu quy tắc (Statement):** Doanh nghiệp có quyền tự thiết lập danh mục nhãn màu phân loại nội bộ và gắn đồng thời nhiều nhãn cho một tờ hóa đơn nhằm phục vụ phân nhóm chi phí hạch toán (chi phí dự án, nguyên vật liệu, tài sản cố định, chi phí tiếp khách) và phục vụ trích xuất báo cáo quản trị chuyên sâu.
* **Điều kiện áp dụng (Condition):** Kế toán viên quản lý danh mục nhãn (thêm mới, đổi tên, chọn màu, xóa nhãn) hoặc thực hiện gán/gỡ nhãn cho một hoặc nhiều hóa đơn trên màn hình làm việc.
* **Hành động & Quy trình xử lý nghiệp vụ (Business Workflow):**
  1. **Quy định về Danh mục Nhãn:**
     * Mỗi nhãn thuộc quyền quản lý riêng của từng tài khoản doanh nghiệp.
     * Thuộc tính nhãn gồm: **Tên nhãn** (tối đa 50 ký tự, không được trùng lặp trong cùng một doanh nghiệp) và **Mã màu nhận diện** (hiển thị trực quan dạng huy hiệu màu trên danh sách hóa đơn).
  2. **Thao tác Gán nhãn cho Hóa đơn:**
     * Cho phép một hóa đơn được gán nhiều nhãn khác nhau (ví dụ: vừa thuộc nhãn *Dự án Landmark*, vừa thuộc nhãn *Chi phí văn phòng*).
     * Hỗ trợ gán nhãn hàng loạt: Kế toán viên có thể tích chọn nhiều hóa đơn cùng lúc trên danh sách và bấm nút "Gán nhãn" để áp dụng chung một nhãn nhanh chóng.
  3. **Lọc và Khai thác Báo cáo theo Nhãn:**
     * Thanh tìm kiếm hỗ trợ lọc hóa đơn theo một hoặc nhiều nhãn kết hợp.
     * Giúp kế toán tổng hợp nhanh tổng chi phí phát sinh theo từng dự án hoặc từng trung tâm chi phí trong kỳ tính thuế.
  4. **Toàn vẹn Dữ liệu khi Xóa Nhãn:**
     * Khi người dùng quyết định xóa bỏ một nhãn khỏi danh mục, hệ thống tự động gỡ bỏ liên kết của nhãn đó khỏi toàn bộ các hóa đơn đã được gán trước đây.
     * Thao tác xóa nhãn không làm ảnh hưởng đến dữ liệu gốc của tờ hóa đơn.
* **Xử lý ngoại lệ (Exception Handling):**
  - Không cho phép tạo trùng tên nhãn đã tồn tại trong cùng một tài khoản doanh nghiệp (hệ thống tự động cắt khoảng trắng thừa ở đầu/cuối và thông báo lỗi nếu tên bị trùng).
* **Nguồn tham chiếu:**
  - `App hóa đơn vào_ ra SMI.xlsx` (Sheet: Hóa đơn và Sheet: Tên hiển thị phần mềm và nhãn h).