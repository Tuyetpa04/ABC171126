# ADR-007 — Chuẩn hóa HTTP Status và Tập trung Dựng Message Lỗi CQT tại PRA

| Mục | Giá trị |
|---|---|
| **Status** | `Accepted` |
| **Date** | 2026-11-04 |
| **Deciders** | Solution Architect, Core Backend Lead, Frontend Lead |
| **Phân hệ ảnh hưởng** | `proxy-forwarder-service/` (PFS), `proxy-request-api/` (PRA), `backend/`, `frontend/` |

---

### 1. Bối cảnh
Trước đây, cơ chế xử lý lỗi khi gọi Cổng Cơ quan Thuế (CQT) gặp phải nhiều vấn đề gây suy giảm hiệu năng và trải nghiệm người dùng:
1. **Lệch pha và phân mảnh câu thông báo:** Thông báo lỗi hiển thị cho người dùng được sinh và ánh xạ (mapping) rải rác ở nhiều lớp khác nhau (PFS, PRA, BE và FE). FE và BE tự ý dịch lại hoặc bọc thêm chuỗi khiến thông điệp không nhất quán.
2. **Khuếch đại tải khi CQT lỗi (Storming/Retry amplification):**
   - PFS từng chỉ dừng failover với một số status nhất định; khi CQT trả về HTTP 503, request có thể bị thử qua toàn bộ pool proxy.
   - Khi đọc body response lỗi bị gián đoạn, lỗi HTTP bị hiểu nhầm thành lỗi kết nối mạng (network transport error) và tiếp tục kích hoạt failover.
   - PRA tiếp tục retry độc lập dù PFS đã kết thúc failover trong từng lượt.
3. **Mất nguyên nhân gốc:** Response lỗi từ CQT chứa XML hoặc HTML thô bị ném thẳng ra UI hoặc bị ghi đè thành câu chung chung `"Kết nối bị gián đoạn..."`.

### 2. Quyết định
Chúng tôi quyết định chuẩn hóa toàn bộ chu trình xử lý lỗi CQT với nguyên tắc cốt lõi: **Proxy Request API (PRA) là nơi duy nhất chịu trách nhiệm dựng thông điệp hiển thị cho người dùng. Backend và Frontend tuyệt đối chỉ chuyển tiếp và hiển thị nguyên văn message từ PRA.**

Quy định phân công trách nhiệm đa tầng:
- **Proxy Forwarder Service (PFS):**
  - Giữ nguyên HTTP status code gốc từ CQT (`IsSuccessStatusCode`).
  - Phân biệt rõ ràng giữa 4 trạng thái: HTTP Response hợp lệ, Timeout, Lỗi kết nối (Connection Refused/Reset), và Trạng thái Pool (`ALL_PROXIES_BANNED`).
  - Dừng failover ngay lập tức khi nhận được response lỗi xác định từ CQT (HTTP 500, 503) hoặc timeout; trả mã nguyên nhân kỹ thuật qua header `X-CQT-Error-Code` và `X-CQT-Timeout`.
  - Tuyệt đối không tự dựng câu thông báo tiếng Việt hướng người dùng.
- **Proxy Request API (PRA) — Nguồn duy nhất dựng Message:**
  - Ánh xạ tập trung từ mã status/mã kỹ thuật sang bảng câu thông báo chuẩn:
    * `401`: *"Phiên đăng nhập CQT đã hết hạn [401]. Hệ thống sẽ tự động đăng nhập lại (nếu được bật) hoặc vui lòng đăng nhập lại."*
    * `429`: *"Cơ quan thuế đang giới hạn số lượng yêu cầu do hệ thống gửi liên tục [429]. Vui lòng thử lại sau."*
    * `500` / `502` / `503`: *"CQT trả về lỗi [<Mã lỗi>]. Vui lòng thử lại sau."*
    * `504` hoặc Timeout PFS: *"CQT không phản hồi trong thời gian chờ [504]. Vui lòng thử lại sau."*
    * `ALL_PROXIES_BANNED`: *"Kết nối đến CQT hiện không khả dụng do đang bị giới hạn truy cập. Vui lòng thử lại sau."*
  - Không đưa raw exception body của CQT vào message hiển thị; đưa diagnostic chi tiết vào metadata riêng.
- **Backend (Node.js API):**
  - Giữ nguyên trường `message` nhận được từ PRA; unwrap exception an toàn và chuyển tiếp nguyên văn về client; không tự ý map lại thành câu tiếng Việt riêng.
- **Frontend (React Portal):**
  - Hiển thị trực tiếp nội dung `message` nhận được lên Toast / Alert; không tự ý viết thêm hậu tố, không mapping lại lỗi 401/429/5xx.
  - Chỉ xử lý thông báo timeout nội bộ khi bản thân request từ FE tới BE bị rớt mạng trước khi nhận được response.

### 3. Lý do
- **Trải nghiệm người dùng đồng nhất:** Kế toán luôn nhận được thông điệp rõ ràng, biết chính xác là do phiên đăng nhập hết hạn, CQT đang quá tải hay mạng proxy đang bị hạn chế.
- **Ngăn chặn khuếch đại tải:** Tránh việc hệ thống spam liên tục hàng chục request tới CQT khi CQT đang trả về HTTP 503 hoặc timeout, giảm thiểu tình trạng treo và quá tải CPU.
- **Dễ bảo trì và debug:** Toàn bộ từ điển câu thông báo chỉ nằm tại 1 nơi duy nhất (PRA), khi cần chỉnh sửa câu chữ không phải sửa đồng thời cả 4 repository (PFS, PRA, BE, FE).

**[Kỹ thuật]**
- Sử dụng header `X-CQT-Error-Code: ALL_PROXIES_BANNED` từ PFS khi toàn bộ proxy đều bị ban và fallback thất bại.
- Sử dụng header `X-CQT-Timeout: true` từ PFS để PRA nhận biết timeout thực tế và chặn retry vô ích.
- Áp dụng cấu hình: `ProxyMaxFailures = 3`, `ProxyBanMinutes = 1`, `ProxyBatchSize = 3`, `ConnectTimeout = 20s`, `Forwarding.TimeoutSeconds = 120s`.
- Bảo toàn logic nghiệp vụ Auto-login: Nhận diện mã `[401]` trong message để kích hoạt luồng tự động đăng nhập lại tài khoản Thuế mà không làm đứt chuỗi hiển thị.

### 4. Các phương án đã xem xét
| Phương án | Ưu điểm | Nhược điểm | Kết luận |
|---|---|---|---|
| **A — PRA làm nguồn duy nhất (Được chọn)** | Thống nhất 100% câu chữ; BE/FE mỏng và đơn giản; dễ kiểm thử đơn vị | PRA phải hiểu các mã kỹ thuật của PFS | **Chấp thuận** |
| **B — FE tự ánh xạ theo status code** | Backend chỉ trả mã số | FE không phân biệt được lỗi timeout do mạng nội bộ hay timeout từ CQT; logic lặp lại nếu có Mobile App | Loại bỏ |
| **C — PFS dựng sẵn câu thông báo** | Dựng ngay tại nơi gọi CQT | Vi phạm nguyên tắc Single Responsibility của tầng Forwarder; PFS là service xử lý mạng tốc độ cao, không nên quản lý UI/UX text | Loại bỏ |

### 5. Hệ quả
- **Tích cực:** Triệt tiêu hoàn toàn tình trạng lệch pha thông báo; 201 testcases (PFS 105/105, PRA 57/57, BE 12/12, FE 17/17) đều pass; bảo vệ hạ tầng không bị bão retry khi CQT nghẽn.
- **Đánh đổi:** Cần đảm bảo các biến môi trường `API_TRA_CUU` ở backend luôn trỏ chính xác qua PRA thay vì gọi tắt trực tiếp sang PFS.
- **Rủi ro:** Khi mất mạng nội bộ trước khi đến PRA, FE cần có thông điệp fallback cục bộ riêng để không bị nhầm lẫn với lỗi từ CQT.

### 6. Tuân thủ & Tác động NFR
- **NFR-RES-001 (Khả năng chịu lỗi & Luân phiên):** Dừng failover đúng lúc, bảo toàn quota proxy.
- **NFR-SEC-001 (Bảo mật thông tin):** Loại bỏ rủi ro rò rỉ cấu trúc hệ thống hoặc stack trace trong raw body CQT ra ngoài người dùng cuối.

