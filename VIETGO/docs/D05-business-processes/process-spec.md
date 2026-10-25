# D05 — BPMN + Process Description | VIETGO (ĐÃ CẬP NHẬT)

**Căn cứ:** FR/NFR nhóm chốt (10 FR, 5 NFR), D02 Use Cases bản chốt, và template D05 của file hướng dẫn.

## 1. Thông tin chung

| Mục | Thông tin |
| --- | --- |
| Tên output | D05 - BPMN + Process Description (VietGo) |
| Phạm vi | P0: 2 quy trình chuyên sâu WF-01 (UC-01) và WF-02 (UC-02). RepeatBox UC-05 là P1, không vẽ riêng trong D05. |
| Nguồn nghiệp vụ | FR/NFR nhóm vừa chốt (bản 3 trang) + D02 Use Case Diagram & Specifications (bản 8 trang). |
| Yêu cầu gốc | Hai file BPMN 2.0, PNG/PDF, process-spec.md; mỗi bước có Actor/Lane, Action, Input, Output, Business Rule, Exception. |
| Cách phân lane | Người gửi hàng và Hệ thống VietGo nằm trong pool VietGo; GHN/GHTK mô hình thành pool bên ngoài, tương tác bằng message flow. |
| Không tự suy diễn | Không định nghĩa endpoint/payload carrier, cách tra cứu tham chiếu, thời hạn hiệu lực Rate, chi tiết thuật toán khóa chống trùng hoặc retry. |

## 2. WF-01 — Lấy và so sánh báo giá GHN/GHTK (UC-01)

Tiền điều kiện: biểu mẫu có thể truy cập; carrier được cấu hình; không đòi hỏi có vận đơn. Trigger: người gửi yêu cầu Tra cứu cước. Hậu điều kiện thành công: có ít nhất một Rate hợp lệ và chưa tạo vận đơn.

| Bước | Actor/Lane | Action | Input | Output | Business Rule | Exception |
| --- | --- | --- | --- | --- | --- | --- |
| 01 | Người gửi | Mở biểu mẫu, nhập địa chỉ gửi/nhận, liên hệ, kiện hàng, khối lượng, kích thước khi cần, giá trị và COD nếu có | Thông tin đơn hàng | Dữ liệu yêu cầu báo giá | FR-01; UC-01 bước 1–2 | Thiếu thông tin → yêu cầu bổ sung (E1) |
| 02 | Người gửi | Chọn “Tra cứu cước” | Dữ liệu vừa nhập | Yêu cầu tra cứu | UC-01 trigger | Chưa gửi API nếu biểu mẫu sai |
| 03 | VietGo | Kiểm tra trường bắt buộc, định dạng và tính hợp lệ | Yêu cầu tra cứu | Kết quả hợp lệ/không hợp lệ | FR-01; UC-01 bước 3 | Sai/thiếu → thông báo rõ trường, cho sửa (E1) |
| 04 | VietGo | Chuẩn hóa địa chỉ, đơn vị và trường dữ liệu cho từng hãng | Dữ liệu hợp lệ | Payload phù hợp GHN và GHTK | FR-02; NFR-05 | Không gửi payload không hợp lệ |
| 05 | VietGo → GHN API | Gửi yêu cầu báo giá GHN độc lập | Payload GHN | Giá, dịch vụ hoặc lỗi GHN | FR-02; NFR-01/03/05 | Lỗi/timeout ≤10 giây không chặn GHTK |
| 06 | VietGo → GHTK API | Gửi yêu cầu báo giá GHTK độc lập | Payload GHTK | Giá, dịch vụ hoặc lỗi GHTK | FR-02; NFR-01/03/05 | Lỗi/timeout ≤10 giây không chặn GHN |
| 07 | VietGo | Kiểm tra phản hồi GHN, chuẩn hóa giá, phí, ETA, khả dụng | Response GHN | Rate GHN chuẩn hóa hoặc lỗi | FR-03; NFR-04/05 | Sai định dạng → loại phương án, ghi lỗi (E3) |
| 08 | VietGo | Kiểm tra phản hồi GHTK, chuẩn hóa giá, phí, ETA, khả dụng | Response GHTK | Rate GHTK chuẩn hóa hoặc lỗi | FR-03; NFR-04/05 | Sai định dạng → loại phương án, ghi lỗi (E3) |
| 09 | VietGo | Tổng hợp trạng thái hai nhánh báo giá | Hai nhóm kết quả/lỗi | Danh sách các phương án hợp lệ | UC-01 bước 5; NFR-03 | Mỗi carrier lỗi độc lập |
| 10 | VietGo | Loại phương án không hỗ trợ tuyến/kiện, phân biệt phí và ETA thiếu | Rate đã chuẩn hóa | Rate có thể chọn và nhãn thiếu dữ liệu | FR-03; NFR-04; UC-01 A2–A4 | ETA: “Chưa có dữ liệu ETA”; phí: “Chưa có dữ liệu”; không tự điền 0 |
| 11 | VietGo | Kiểm tra có ít nhất một Rate hợp lệ hay không | Danh sách sau lọc | Rẽ nhánh có/không có báo giá | UC-01 A1/A5/E2 | Phân biệt không có dịch vụ với lỗi kỹ thuật |
| 12 | VietGo | Nếu không có Rate, phân loại nguyên nhân cả hai API lỗi hay không có dịch vụ | Trạng thái hai carrier | Nguyên nhân nghiệp vụ/kỹ thuật | UC-01 A5/E2/E3; FR-08 | A5: không lỗi kỹ thuật; E2/E3: ghi log, không giá giả |
| 13 | VietGo | Nếu chỉ một hãng có Rate, thông báo hãng còn lại tạm không khả dụng | Rate còn lại | Cảnh báo kèm báo giá hợp lệ | UC-01 A1; NFR-03 | Không chặn báo giá có căn cứ |
| 14 | VietGo | Hiển thị hãng, dịch vụ, cước chính, phụ phí, tổng phí, ETA bằng VNĐ | Rate hợp lệ, nhãn dữ liệu thiếu | Bảng báo giá so sánh | FR-03; NFR-04 | Không dự đoán ETA, không tính giá giả, không tự cộng trùng |
| 15 | Người gửi | So sánh và chọn Rate hợp lệ | Bảng báo giá | Rate được chọn cho UC-02 | UC-01 bước 7; FR-04 (liên kết) | Không cho chọn dịch vụ không khả dụng |
| 16 | VietGo | Kết thúc UC-01, chuyển Rate đã chọn sang UC-02; chưa tạo Shipment | Rate được chọn | Điều kiện đầu vào cho UC-02 | UC-01 hậu điều kiện | Nếu không có Rate: kết thúc với thông báo/cho nhập lại |

### Nhánh Alternate / Exception — UC-01

| Mã | Điều kiện | Xử lý |
| --- | --- | --- |
| A1 | Chỉ một carrier có Rate hợp lệ | Vẫn hiển thị Rate hãng đó và cảnh báo hãng còn lại |
| A2 | Thiếu ETA hợp lệ | Hiển thị “Chưa có dữ liệu ETA”, không tự dự đoán |
| A3 | Thiếu thành phần phí | Hiển thị “Chưa có dữ liệu”, không thay bằng 0 |
| A4 | Không hỗ trợ tuyến/kiện | Không cho chọn phương án đó |
| A5 | Response hợp lệ nhưng không có dịch vụ | Thông báo không có dịch vụ phù hợp; cho nhập lại |
| E1 | Thiếu/sai đầu vào | Nêu trường sai; không gọi API |
| E2 | Cả hai API lỗi/timeout | Ghi nhận lỗi, thông báo không thể lấy giá, cho thử lại |
| E3 | Dữ liệu API sai định dạng | Loại phương án, ghi lỗi tích hợp; không dựng giá giả |

## 3. WF-02 — Chọn dịch vụ và tạo vận đơn (UC-02)

Tiền điều kiện: người gửi đã đăng nhập, có ít nhất một Rate hợp lệ từ UC-01. Trigger: xác nhận tạo vận đơn. Hậu điều kiện thành công: một Shipment lưu an toàn, có các mã/thông tin hãng xác nhận.

| Bước | Actor/Lane | Action | Input | Output | Business Rule | Exception |
| --- | --- | --- | --- | --- | --- | --- |
| 01 | Người gửi | Chọn một Rate hợp lệ sau UC-01 | Kết quả báo giá đã hợp lệ | Rate được chọn | FR-04; UC-02 bước 1 | Không cho chọn dịch vụ không còn khả dụng (A2) |
| 02 | VietGo | Hiển thị trang xác nhận người gửi/nhận, kiện hàng, COD, hãng, dịch vụ, phí | Rate và dữ liệu đơn | Bản tóm tắt để xác nhận | FR-04; UC-02 bước 2 | Trường tùy chọn thiếu → “Chưa có dữ liệu” (A3) |
| 03 | Người gửi | Kiểm tra, quyết định chỉnh sửa hoặc xác nhận | Bản tóm tắt | Thông tin đã rà soát | UC-02 bước 3; A1 | Chỉnh sửa ảnh hưởng giá/dịch vụ → quay UC-01 lấy báo giá mới |
| 04 | Người gửi | Nhấn “Xác nhận/Tạo vận đơn” | Rate và thông tin xác nhận | Yêu cầu tạo vận đơn | UC-02 trigger | Chưa xác nhận → không gọi API |
| 05 | VietGo | Xác minh phiên đăng nhập, quyền, dữ liệu, khả dụng và hiệu lực Rate | Tài khoản, đơn hàng, Rate | Đủ/không đủ điều kiện tạo | FR-01/04; NFR-02 | Hết hiệu lực, thiếu quyền, sai dữ liệu → yêu cầu sửa/chọn lại (A2) |
| 06 | VietGo | Tìm hoặc tạo booking_key duy nhất, bất biến cho lần xác nhận | Yêu cầu booking | booking_key gắn yêu cầu | UC-02 Business Rules | Không sinh booking_key mới cho cùng lần xác nhận |
| 07 | VietGo | Kiểm tra đã tồn tại booking_key hay chưa | booking_key và booking_status đã lưu | Quyết định yêu cầu mới/trùng | UC-02 A4; NFR-03 | Chặn hai booking đồng thời bằng cùng yêu cầu |
| 08 | VietGo | Nếu trùng: trả kết quả tùy booking_status, không gọi carrier lần nữa | booking_status có sẵn | SUCCESS → đơn cũ; PENDING/UNKNOWN → đang xử lý/xác minh; FAILED → đã từ chối | UC-02 A4; NFR-03 | Không tự tạo đơn mới khi bấm lặp/retry |
| 09 | VietGo | Đối với yêu cầu mới: lưu PENDING và booking_attempts | booking_key, dữ liệu xác nhận | Dấu vết trạng thái đang xử lý | UC-02 Business Rules | Lưu vết lần gửi để phục vụ xác minh |
| 10 | VietGo → carrier được chọn | Backend gửi yêu cầu booking đến GHN HOẶC GHTK | Yêu cầu đơn, booking_key, Rate | Response hoặc lỗi/timeout | FR-04; NFR-01/02 | Không đưa token API ra trình duyệt; tối đa 10 giây/yêu cầu |
| 11 | VietGo | Phân loại phản hồi: thành công có căn cứ, từ chối, chưa xác định | API response | Nhánh kết quả booking | UC-02 bước 6/E1–E3; NFR-03 | Không suy ra thành công từ phản hồi thiếu căn cứ |
| 12 | VietGo | Nếu thành công: lưu Shipment, mã VietGo, mã carrier nếu có, phí, COD, trạng thái | Phản hồi tạo đơn hợp lệ | Vận đơn và chi tiết đã lưu | FR-04; FR-07 liên quan; UC-02 bước 7 | Lỗi lưu hoặc dữ liệu thiếu căn cứ → không báo hoàn tất (E3) |
| 13 | VietGo | Chỉ sau khi lưu an toàn, đánh dấu SUCCESS và hiển thị kết quả | Shipment đã được xác nhận/lưu | Thông báo thành công, mã/chi tiết đơn | FR-04; UC-02 bước 8 | Trường tùy chọn chưa có → “Chưa có dữ liệu” (A3) |
| 14 | VietGo | Carrier từ chối hợp lệ: đặt FAILED, thông báo lý do | Phản hồi từ chối và lý do | Kết quả tạo đơn thất bại | FR-08; UC-02 E1 | Không báo tạo thành công |
| 15 | VietGo | Timeout/mất kết nối/phản hồi chưa rõ: đặt UNKNOWN | Thời điểm timeout, booking_key | Trạng thái UNKNOWN + lịch sử attempts | UC-02 E2/E3; NFR-03 | UNKNOWN ≠ FAILED; giữ booking_key, không tự retry |
| 16 | VietGo | Thông báo “Đang xác minh kết quả tạo vận đơn” | UNKNOWN | Thông tin chờ xác minh cho người gửi | UC-02 E2 | Không hiển thị thành công giả |
| 17 | VietGo ↔ carrier | Nếu API hỗ trợ, truy vấn/đối chiếu bằng mã tham chiếu trước retry | booking_key, booking_attempts, tham chiếu | Kết quả xác minh có căn cứ hoặc không rõ | UC-02 E2 | Không tự đặt endpoint hay cách truy vấn hãng khi tài liệu chưa có |
| 18 | VietGo | Nếu carrier xác nhận đã tạo: lưu Shipment, đặt SUCCESS, trả mã/chi tiết | Kết quả đã tạo được xác minh | Vận đơn đã xác nhận | UC-02 E2; NFR-03 | Không gửi lại booking khi đã tồn tại |
| 19 | VietGo | Nếu carrier xác nhận chưa tạo và cơ chế hãng bảo đảm retry an toàn: retry có kiểm soát cùng key | Kết quả chưa tạo + căn cứ retry an toàn | Một booking_attempt mới dùng cùng booking_key | UC-02 E2; Business Rules | Không mặc định có hỗ trợ kiểm tra/retry an toàn |
| 20 | VietGo | Nếu không thể xác minh hoặc không an toàn: giữ UNKNOWN, chuyển kiểm tra thủ công | Kết quả không xác định | Trạng thái UNKNOWN chưa kết thúc nghiệp vụ | UC-02 E2; NFR-03 | Không tự động retry, không báo FAILED giả |
| 21 | VietGo | Nếu lỗi lưu hoặc khôi phục dữ liệu: lưu dấu vết, không công bố hoàn tất | Response hợp lệ hoặc trạng thái cần khôi phục | Cảnh báo cần kiểm tra/khôi phục an toàn | UC-02 E3; NFR-04 | Không đánh dấu SUCCESS khi chưa bảo đảm dữ liệu |
| 22 | VietGo | Kết thúc với kết quả có căn cứ: SUCCESS, FAILED hoặc PENDING/UNKNOWN | Trạng thái booking đã xác minh/lưu | Hiển thị trạng thái đúng sự thật | UC-02 hậu điều kiện | Trạng thái vận chuyển Shipment khác booking_status |

### Nhánh Alternate / Exception — UC-02

| Mã | Điều kiện | Xử lý |
| --- | --- | --- |
| A1 | Chỉnh sửa trước xác nhận | Nếu thay đổi ảnh hưởng phí/dịch vụ, lấy báo giá mới |
| A2 | Rate hết hạn/hết khả dụng | Yêu cầu tra cứu/chọn lại; không tạo theo giá cũ |
| A3 | Thiếu trường tùy chọn | Chỉ hiển thị trường đã xác nhận; trường khác “Chưa có dữ liệu” |
| A4 | Xác nhận nhiều lần | Dùng lại booking_key; PENDING/UNKNOWN không book lại; SUCCESS trả đơn cũ |
| E1 | Carrier từ chối | Đặt FAILED; hiện lý do có căn cứ |
| E2 | Timeout/mất kết nối | Đặt UNKNOWN; giữ key/attempts; xác minh trước retry; không thể xác minh → thủ công |
| E3 | Không đủ căn cứ phản hồi/lỗi lưu | Không báo hoàn tất; lưu dấu vết và xử lý khôi phục an toàn |

## 4. Quy tắc và giới hạn

- BẢN FR ĐÃ CHỐT: WF-01 gắn FR-01 (dữ liệu đầu vào), FR-02 (Multi-Carrier Shipping Quotation), FR-03 (Shipping Rate Comparison & ETA), FR-08 (thông báo lỗi). WF-02 gắn FR-01 (tài khoản), FR-04 (Shipment Booking), FR-08 (thông báo lỗi); FR-07 liên quan khi lưu thông tin COD và FR-05 ở bước xem chi tiết đơn.
- Chú ý mã FR: tài liệu D02 đang ghi các mã chi tiết kiểu FR-02–FR-08 hoặc FR-09–FR-12, FR-17 từ bộ SRS trước. Bản này lấy mã FR-01–FR-10 theo file FR mới được nhóm chốt, KHÔNG coi các mã chi tiết cũ là mã hiện hành.
- WF-01: gọi cả hai hãng độc lập; lỗi của một hãng không chặn hãng còn lại. Không có Rate do không hỗ trợ dịch vụ là tình huống nghiệp vụ (A5), khác lỗi cả hai API (E2). Khi hãng không trả ETA hoặc một trường phí, hiển thị chưa có dữ liệu; không tự đoán và không tự thay bằng 0.
- WF-02: booking_key bất biến và tái sử dụng trên cùng lần xác nhận. Các booking_status là PENDING, SUCCESS, FAILED, UNKNOWN. UNKNOWN không đồng nghĩa FAILED. PENDING/UNKNOWN không tự tạo yêu cầu mới hoặc tự retry.
- WF-02: timeout → ghi UNKNOWN, giữ key và booking_attempts; kiểm tra hãng qua mã tham chiếu NẾU hãng hỗ trợ. Chỉ retry sau khi hãng xác nhận chưa tạo và bảo đảm retry an toàn. Nếu không xác minh được, giữ UNKNOWN và chuyển kiểm tra thủ công.
- NFR-01 theo file mới: mỗi yêu cầu API báo giá hoặc tạo vận đơn timeout tối đa 10 giây. NFR-02 bảo vệ token phía server; NFR-03 bảo đảm không báo thành công giả/không trùng; NFR-04 toàn vẹn dữ liệu; NFR-05 tích hợp adapter độc lập/ghi lỗi.
- BPMN diễn tả LUỒNG NGHIỆP VỤ. Booking_status phản ánh kết quả tạo đơn, không phải shipment_status phản ánh vận chuyển. Không mô tả thanh toán trực tuyến, đối soát COD hay RepeatBox scheduler trong hai WF P0.

## 5. File bàn giao

`WF01-quote.bpmn`, `WF02-create-shipment.bpmn`, `WF01-quote.png/.pdf`, `WF02-create-shipment.png/.pdf`, `process-spec.md`, `D05-Process-Description.docx/.pdf`, `D05-VietGo-Hoan-Chinh.pdf`.