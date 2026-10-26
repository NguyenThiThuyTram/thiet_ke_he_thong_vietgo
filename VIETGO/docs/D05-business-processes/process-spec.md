# D05 — BPMN + Process Description | VIETGO

**Phiên bản cập nhật:** 26/10/2026  
**Phạm vi chỉnh sửa:** Đồng bộ tên Start/End Event và Task theo `WF01-quote(1).bpmn` và `WF02-create-shipment(1).bpmn`; làm rõ ánh xạ giữa sơ đồ và bảng Process Description. **Không thay đổi luồng xử lý, gateway, pool/lane, business rule hoặc exception.**  
**Căn cứ nghiệp vụ kế thừa:** FR/NFR nhóm chốt (10 FR, 5 NFR), D02 Use Cases bản chốt và template D05 — theo `process-spec.md` gốc. Các tài liệu FR/NFR, D02 không được đính kèm trong lần cập nhật này, nên mã tham chiếu dưới đây được giữ từ bản cũ, chưa được kiểm tra độc lập.

## 1. Thông tin chung

| Mục | Thông tin |
| --- | --- |
| Tên output | D05 — BPMN + Process Description (VietGo) |
| Phạm vi | P0: 2 quy trình WF-01 (UC-01) và WF-02 (UC-02). RepeatBox UC-05 thuộc P1, không vẽ riêng trong D05. |
| File BPMN đối chiếu | `WF01-quote(1).bpmn`, `WF02-create-shipment(1).bpmn` |
| File mô tả quy trình | `process-spec-updated.md` |
| Cách phân lane | Trong pool VietGo có hai lane **Người gửi hàng** và **Hệ thống VietGo**. GHN/GHTK là pool ngoài, kết nối qua message flow. |
| Quy ước nhãn | **Event** mang tên sự kiện/kết quả, không có tiền tố “Bắt đầu” / “Kết thúc”. **Task** có tên ngắn, ưu tiên động từ + đối tượng. Phần nghiệp vụ chi tiết giữ trong các cột Input/Output, Business Rule và Exception. |
| Giới hạn | Không suy diễn endpoint/payload carrier, thời hạn hiệu lực Rate, thuật toán chống trùng, cơ chế xác minh hoặc retry an toàn. |

## 2. WF-01 — Lấy và so sánh báo giá GHN/GHTK (UC-01)

**Tiền điều kiện:** Người gửi có thể mở biểu mẫu; GHN và GHTK đã được cấu hình tích hợp; chưa cần có vận đơn.  
**Trigger:** Người gửi yêu cầu tra cứu cước.  
**Hậu điều kiện thành công:** Có ít nhất một Rate hợp lệ được chọn và chuyển sang UC-02; **chưa tạo Shipment**.

### 2.1. Start/End Event (tên khớp BPMN)

| BPMN ID | Loại | Tên hiển thị | Ý nghĩa |
| --- | --- | --- | --- |
| `Q_Start` | Start Event | **Yêu cầu tra cứu cước** | Kích hoạt quy trình lấy báo giá. |
| `Q_EndNone` | End Event | **Không có dịch vụ phù hợp** | Không tìm được Rate phù hợp theo nhánh nghiệp vụ. |
| `Q_EndError` | End Event | **Tra cứu báo giá thất bại** | Cả hai API lỗi hoặc timeout theo nhánh phân loại trên sơ đồ. |
| `Q_EndOK` | End Event | **Đã chọn báo giá để tạo vận đơn** | Có Rate được chọn để chuyển sang UC-02. |

### 2.2. Process Description — các Task

> Cột **Action** là **tên Task ngắn đúng như trong BPMN**. Các cột còn lại diễn giải nghiệp vụ từ tài liệu cũ. Bước được đánh số theo luồng đọc chính và nhóm nhánh, không phải trình tự thực thi bắt buộc (hai API chạy song song).

| Bước | BPMN ID | Actor/Lane | Action | Input | Output | Business Rule | Exception |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 01 | `Q_Form` | Người gửi hàng | **Nhập thông tin gửi hàng** | Địa chỉ gửi/nhận, liên hệ, kiện hàng, khối lượng/kích thước khi cần, giá trị, COD (nếu có) | Dữ liệu yêu cầu báo giá | FR-01; UC-01 bước 1–2 | Thiếu thông tin → yêu cầu bổ sung (E1). |
| 02 | `Q_Search` | Người gửi hàng | **Chọn “Tra cứu cước”** | Dữ liệu vừa nhập | Yêu cầu tra cứu | UC-01 trigger | Chưa gửi API nếu dữ liệu chưa hợp lệ. |
| 03 | `Q_Validate` | Hệ thống VietGo | **Kiểm tra thông tin gửi hàng** | Yêu cầu tra cứu | Kết quả hợp lệ/không hợp lệ | FR-01; UC-01 bước 3: kiểm tra trường bắt buộc, định dạng và tính hợp lệ | Sai/thiếu → đi nhánh `Q_Invalid` (E1). |
| 04 (E1) | `Q_Invalid` | Hệ thống VietGo | **Thông báo dữ liệu không hợp lệ** | Danh sách trường sai/thiếu | Thông báo để nhập lại | UC-01 E1: chỉ rõ trường cần sửa | Quay về `Q_Form`; không gọi API khi đầu vào sai. |
| 05 | `Q_Prepare` | Hệ thống VietGo | **Chuẩn hóa dữ liệu gửi hãng** | Dữ liệu đã hợp lệ | Dữ liệu phù hợp với GHN và GHTK | FR-02; NFR-05: chuẩn hóa địa chỉ, đơn vị, trường dữ liệu theo từng API | Không gửi payload không hợp lệ. |
| 06 (song song) | `Q_GHN` | Hệ thống VietGo → GHN API | **Lấy báo giá GHN** | Dữ liệu chuẩn hóa cho GHN | Response báo giá hoặc lỗi GHN | FR-02; NFR-01/03/05: gọi hãng độc lập, timeout tối đa 10 giây | GHN lỗi/timeout không chặn GHTK (E2/E3). |
| 07 (song song) | `Q_GHTK` | Hệ thống VietGo → GHTK API | **Lấy báo giá GHTK** | Dữ liệu chuẩn hóa cho GHTK | Response báo giá hoặc lỗi GHTK | FR-02; NFR-01/03/05: gọi hãng độc lập, timeout tối đa 10 giây | GHTK lỗi/timeout không chặn GHN (E2/E3). |
| 08 | `Q_GHNMap` | Hệ thống VietGo | **Kiểm tra báo giá GHN** | Response GHN | Rate GHN chuẩn hóa hoặc lỗi | FR-03; NFR-04/05: kiểm tra giá, phí, ETA, khả dụng | Sai định dạng → loại Rate lỗi và ghi log (E3). |
| 09 | `Q_GHTKMap` | Hệ thống VietGo | **Kiểm tra báo giá GHTK** | Response GHTK | Rate GHTK chuẩn hóa hoặc lỗi | FR-03; NFR-04/05: kiểm tra giá, phí, ETA, khả dụng | Sai định dạng → loại Rate lỗi và ghi log (E3). |
| 10 | `Q_Filter` | Hệ thống VietGo | **Lọc báo giá không phù hợp** | Kết quả hai hãng sau khi hoàn tất hai nhánh | Danh sách Rate có thể chọn, nhãn thông tin thiếu | FR-03; NFR-04; UC-01 A2–A4: loại tuyến/kiện không hỗ trợ | Thiếu ETA → “Chưa có dữ liệu ETA”; thiếu phí → “Chưa có dữ liệu”; không thay bằng 0. |
| 11 (A5) | `Q_NoService` | Hệ thống VietGo | **Thông báo không có dịch vụ** | Không có Rate hợp lệ; nhánh không phải cả hai API lỗi | Thông báo không có dịch vụ phù hợp | UC-01 A5; phân biệt không có dịch vụ với lỗi tích hợp | Kết thúc tại `Q_EndNone`; có thể tra cứu lại bằng yêu cầu mới. |
| 12 (E2/E3) | `Q_BothFail` | Hệ thống VietGo | **Thông báo lỗi tra cứu cước** | Cả hai API lỗi/timeout theo nhánh `Q_NoType` | Thông báo không thể lấy báo giá | UC-01 E2/E3; FR-08: ghi log lỗi | Kết thúc tại `Q_EndError`; có thể thử lại. |
| 13 (A1) | `Q_Notice` | Hệ thống VietGo | **Cảnh báo hãng chưa khả dụng** | Chỉ một hãng có Rate hợp lệ | Cảnh báo về hãng còn lại | UC-01 A1; NFR-03 | Không chặn Rate có căn cứ của hãng còn hoạt động. |
| 14 | `Q_Display` | Hệ thống VietGo | **Hiển thị so sánh phí và ETA** | Rate hợp lệ, nhãn dữ liệu thiếu | Bảng hãng/dịch vụ/cước/phụ phí/tổng phí/ETA (VNĐ) | FR-03; NFR-04 | Không dựng giá giả, không tự dự đoán ETA hoặc cộng trùng phí. |
| 15 | `Q_Select` | Người gửi hàng | **Chọn báo giá hợp lệ** | Bảng so sánh | Rate được chọn để sang UC-02 | UC-01 bước 7; liên kết FR-04 | Không chọn dịch vụ không khả dụng; chưa tạo Shipment. |

### 2.3. Gateway và các nhánh quyết định

| BPMN ID | Tên Gateway trên BPMN | Điều kiện / hướng xử lý |
| --- | --- | --- |
| `Q_Valid` | Thông tin hợp lệ? | **Có** → `Q_Prepare`; **Không** → `Q_Invalid` → `Q_Form`. |
| `Q_Fork` | Gọi 2 hãng độc lập | Rẽ song song sang `Q_GHN` và `Q_GHTK`. |
| `Q_Join` | Đã nhận / xử lý cả hai nhánh | Hợp hai nhánh đã xử lý, sau đó vào `Q_Filter`. |
| `Q_Any` | Có Rate hợp lệ? | **Có** → `Q_Partial`; **Không** → `Q_NoType`. |
| `Q_NoType` | Lỗi kỹ thuật cả hai? | **Có** → `Q_BothFail` → `Q_EndError`; **Không; API hợp lệ** → `Q_NoService` → `Q_EndNone`. |
| `Q_Partial` | Chỉ một hãng có Rate? | **Có** → `Q_Notice` → `Q_Display`; **Không** → `Q_Display`. |

### 2.4. Alternate / Exception — UC-01

| Mã | Điều kiện | Xử lý |
| --- | --- | --- |
| A1 | Chỉ một carrier có Rate hợp lệ | Vẫn hiển thị Rate của hãng đó và cảnh báo hãng còn lại. |
| A2 | Thiếu ETA hợp lệ | Hiển thị “Chưa có dữ liệu ETA”, không tự dự đoán. |
| A3 | Thiếu thành phần phí | Hiển thị “Chưa có dữ liệu”, không tự thay bằng 0. |
| A4 | Không hỗ trợ tuyến/kiện | Không cho chọn phương án đó. |
| A5 | Response hợp lệ nhưng không có dịch vụ | Thông báo không có dịch vụ phù hợp, cho nhập lại/tra cứu mới. |
| E1 | Thiếu hoặc sai đầu vào | Nêu trường sai, không gọi API; quay lại nhập thông tin. |
| E2 | Cả hai API lỗi/timeout | Ghi nhận lỗi, thông báo không thể lấy giá, cho thử lại. |
| E3 | Dữ liệu API sai định dạng | Loại phương án lỗi, ghi lỗi tích hợp, không dựng giá giả. |

## 3. WF-02 — Chọn dịch vụ và tạo vận đơn (UC-02)

**Tiền điều kiện:** Người gửi đã đăng nhập và có ít nhất một Rate hợp lệ từ UC-01.  
**Trigger:** Người gửi xác nhận tạo vận đơn sau khi rà soát thông tin.  
**Hậu điều kiện thành công:** Shipment đã được lưu an toàn; thông tin/mã vận đơn được hãng xác nhận ở mức dữ liệu thực có.

### 3.1. Start/End Event (tên khớp BPMN)

| BPMN ID | Loại | Tên hiển thị | Ý nghĩa |
| --- | --- | --- | --- |
| `S_Start` | Start Event | **Nhận Rate từ UC-01** | Bắt đầu luồng lựa chọn/xác nhận từ Rate đã tra cứu. |
| `S_EndRerate` | End Event | **Quay lại UC-01 để báo giá** | Cần chỉnh sửa dẫn tới tra cứu lại. |
| `S_EndInvalid` | End Event | **Chưa đủ điều kiện tạo đơn** | Thông tin, quyền hoặc Rate không hợp lệ. |
| `S_EndDupOK` | End Event | **Vận đơn đã tồn tại** | Trả Shipment có sẵn, không gửi booking lại. |
| `S_EndDupFail` | End Event | **Yêu cầu trước đã thất bại** | Trả trạng thái FAILED của yêu cầu đã có. |
| `S_EndDupHold` | End Event | **Yêu cầu đang được xử lý** | Trả trạng thái PENDING/UNKNOWN đã lưu, không đặt trùng. |
| `S_EndRejected` | End Event | **Hãng vận chuyển từ chối** | Hãng từ chối tạo đơn có căn cứ. |
| `S_EndSaveFail` | End Event | **Cần khôi phục dữ liệu** | Lỗi lưu hoặc chưa bảo đảm toàn vẹn dữ liệu. |
| `S_EndSuccess` | End Event | **Vận đơn được tạo thành công** | Đã lưu an toàn, booking_status = SUCCESS. |
| `S_EndConfirmed` | End Event | **Đã xác minh tạo vận đơn** | Hãng xác nhận đã tạo sau nhánh đối chiếu; đã lưu kết quả. |
| `S_EndManual` | End Event | **Chờ xác minh thủ công** | Chuyển xử lý thủ công; UNKNOWN **không** phải thành công/thất bại. |

### 3.2. Process Description — các Task

> Tên trong cột **Action** trùng với BPMN. Nhánh `S_Retry` chỉ được phép khi đã xác minh **chưa tạo đơn** và có bằng chứng retry an toàn từ hãng.

| Bước | BPMN ID | Actor/Lane | Action | Input | Output | Business Rule | Exception |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 01 | `S_Select` | Người gửi hàng | **Chọn Rate hợp lệ** | Các Rate từ UC-01 | Rate được chọn | FR-04; UC-02 bước 1 | Không cho chọn Rate không còn khả dụng (A2). |
| 02 | `S_Summary` | Hệ thống VietGo | **Hiển thị thông tin vận đơn** | Rate được chọn và dữ liệu đơn hàng | Tóm tắt người gửi/nhận, kiện, COD, hãng, dịch vụ, phí | FR-04; UC-02 bước 2 | Trường tùy chọn chưa có → “Chưa có dữ liệu” (A3). |
| 03 | `S_Review` | Người gửi hàng | **Kiểm tra thông tin vận đơn** | Bản tóm tắt | Quyết định sửa hoặc xác nhận | UC-02 bước 3; A1 | Nếu thay đổi ảnh hưởng cước/dịch vụ → nhánh `S_ReturnUC01`. |
| 04 (A1) | `S_ReturnUC01` | Hệ thống VietGo | **Chỉnh sửa và báo giá lại** | Yêu cầu sửa thông tin | Điều hướng về UC-01 để lấy Rate mới | UC-02 A1 | Đi tới `S_EndRerate`; không dùng Rate cũ khi dữ liệu tính phí thay đổi. |
| 05 | `S_Confirm` | Người gửi hàng | **Xác nhận tạo vận đơn** | Thông tin đã kiểm tra và Rate | Yêu cầu booking | UC-02 trigger | Chưa xác nhận thì không gọi API carrier. |
| 06 | `S_Check` | Hệ thống VietGo | **Kiểm tra điều kiện tạo đơn** | Phiên đăng nhập, quyền, dữ liệu đơn, Rate | Kết quả đủ/không đủ điều kiện | FR-01/04; NFR-02: kiểm tra đăng nhập, quyền, khả dụng và hiệu lực Rate | Hết hạn, thiếu quyền, sai dữ liệu → `S_Invalid` (A2). |
| 07 (A2) | `S_Invalid` | Hệ thống VietGo | **Yêu cầu sửa hoặc chọn Rate khác** | Lý do không đủ điều kiện | Hướng dẫn chỉnh sửa/chọn Rate mới | UC-02 A2 | Đi tới `S_EndInvalid`; không gửi booking. |
| 08 | `S_Key` | Hệ thống VietGo | **Xác lập booking_key** | Yêu cầu booking được xác nhận | booking_key bất biến theo cùng yêu cầu | UC-02 Business Rules | Không sinh key mới khi người dùng xác nhận lại cùng yêu cầu. |
| 09 (A4) | `S_Status` | Hệ thống VietGo | **Kiểm tra trạng thái booking** | booking_key đã tồn tại | booking_status đã lưu | UC-02 A4; NFR-03 | Chỉ dùng cho nhánh key trùng; không book lại. |
| 10 (A4) | `S_DupOK` | Hệ thống VietGo | **Trả lại vận đơn đã lưu** | booking_status = SUCCESS | Shipment hiện có | UC-02 A4; NFR-03 | Đi tới `S_EndDupOK`; không gọi hãng lần nữa. |
| 11 (A4) | `S_DupHold` | Hệ thống VietGo | **Thông báo yêu cầu đang xử lý** | booking_status = PENDING/UNKNOWN | Thông báo đang xử lý hoặc xác minh | UC-02 A4; NFR-03 | Đi tới `S_EndDupHold`; không tạo booking mới. |
| 12 (A4) | `S_DupFail` | Hệ thống VietGo | **Thông báo yêu cầu đã thất bại** | booking_status = FAILED | Lý do/tình trạng từ chối đã lưu | UC-02 A4; FR-08 | Đi tới `S_EndDupFail`; không tự động gửi lại. |
| 13 | `S_Prepare` | Hệ thống VietGo | **Lưu yêu cầu PENDING** | booking_key mới và dữ liệu đã xác nhận | booking_status = PENDING; booking_attempts | UC-02 Business Rules: lưu dấu vết trước khi gọi hãng | Chống gửi trùng cùng một yêu cầu. |
| 14 | `S_Call` | Hệ thống VietGo → GHN hoặc GHTK API | **Gửi yêu cầu tạo vận đơn** | Rate, thông tin đơn và booking_key | Phản hồi carrier hoặc lỗi/timeout | FR-04; NFR-01/02: gọi đúng hãng đã chọn, timeout tối đa 10 giây; token giữ phía server | Timeout/phản hồi không rõ → `S_Unknown` (E2/E3). |
| 15 | `S_Save` | Hệ thống VietGo | **Lưu thông tin vận đơn** | Phản hồi thành công có căn cứ | Shipment, mã VietGo/mã hãng nếu có, phí, COD, trạng thái | FR-04; FR-07 liên quan; UC-02 bước 7 | Lỗi lưu/thiếu căn cứ → `S_SaveFail` (E3). |
| 16 (E1) | `S_Reject` | Hệ thống VietGo | **Ghi nhận hãng từ chối** | Phản hồi từ chối hợp lệ và lý do | booking_status = FAILED; thông báo lỗi | FR-08; UC-02 E1 | Đi tới `S_EndRejected`; không báo thành công. |
| 17 (E2/E3) | `S_Unknown` | Hệ thống VietGo | **Ghi nhận trạng thái UNKNOWN** | Timeout, mất kết nối hoặc phản hồi chưa đủ căn cứ | booking_status = UNKNOWN, giữ booking_key/attempts | UC-02 E2/E3; NFR-03 | UNKNOWN ≠ FAILED; không tự retry. |
| 18 | `S_Complete` | Hệ thống VietGo | **Hoàn tất tạo vận đơn** | Shipment được lưu an toàn | booking_status = SUCCESS; mã và chi tiết đã xác nhận | FR-04; UC-02 bước 8; A3 | Chỉ hiển thị SUCCESS sau khi lưu an toàn; đến `S_EndSuccess`. |
| 19 (E3) | `S_SaveFail` | Hệ thống VietGo | **Ghi nhận lỗi lưu vận đơn** | Kết quả API hoặc dữ liệu lưu chưa bảo đảm | Dấu vết lỗi và yêu cầu khôi phục | UC-02 E3; NFR-04 | Không đánh dấu SUCCESS; đến `S_EndSaveFail`. |
| 20 (E2) | `S_Notify` | Hệ thống VietGo | **Thông báo đang xác minh** | booking_status = UNKNOWN | Thông báo “Đang xác minh kết quả tạo vận đơn” | UC-02 E2 | Không công bố thành công khi chưa có căn cứ. |
| 21 (E2) | `S_Reconcile` | Hệ thống VietGo ↔ carrier | **Đối chiếu kết quả với hãng** | booking_key, booking_attempts, tham chiếu nếu có | Kết quả xác minh: đã tạo/chưa tạo/chưa xác định | UC-02 E2 | Chỉ tra cứu/đối chiếu theo tính năng hãng thực sự hỗ trợ; không tự giả định endpoint. |
| 22 (E2) | `S_Confirmed` | Hệ thống VietGo | **Lưu vận đơn đã xác minh** | Carrier xác nhận đã tạo đơn | Shipment lưu an toàn; booking_status = SUCCESS | UC-02 E2; NFR-03 | Không retry nếu đã tạo; tới `S_EndConfirmed`. |
| 23 (E2) | `S_Manual` | Hệ thống VietGo | **Chuyển xử lý thủ công** | Không xác minh được, hoặc không chứng minh được retry an toàn | Giữ UNKNOWN; chuyển người phụ trách kiểm tra | UC-02 E2; NFR-03 | Không coi UNKNOWN là FAILED; tới `S_EndManual`. |
| 24 (E2) | `S_Retry` | Hệ thống VietGo | **Thử lại với booking_key cũ** | Carrier xác nhận **chưa tạo** và bảo đảm retry an toàn | Một booking_attempt mới, giữ nguyên booking_key | UC-02 E2; Business Rules | Chỉ lúc đủ điều kiện; quay về `S_Call`, không sinh key mới. |

### 3.3. Gateway và các nhánh quyết định

| BPMN ID | Tên Gateway trên BPMN | Điều kiện / hướng xử lý |
| --- | --- | --- |
| `S_Edit` | Cần sửa thông tin? | **Có** → `S_ReturnUC01` → `S_EndRerate`; **Không** → `S_Confirm`. |
| `S_Valid` | Đủ điều kiện? | **Có** → `S_Key`; **Không** → `S_Invalid` → `S_EndInvalid`. |
| `S_Dup` | booking_key đã tồn tại? | **Có** → `S_Status`/`S_DupState`; **Không** → `S_Prepare`. |
| `S_DupState` | booking_status? | **SUCCESS** → `S_DupOK`; **PENDING/UNKNOWN** → `S_DupHold`; **FAILED** → `S_DupFail`. |
| `S_Result` | Kết quả từ carrier? | **Thành công** → `S_Save`; **Từ chối** → `S_Reject`; **Timeout/chưa rõ** → `S_Unknown`. |
| `S_SaveOK` | Lưu an toàn thành công? | **Có** → `S_Complete`; **Không** → `S_SaveFail`. |
| `S_VResult` | Kết quả xác minh? | **Đã tạo** → `S_Confirmed`; **Chưa tạo** → `S_CheckSafe`; **Không xác minh được** → `S_Manual`. |
| `S_CheckSafe` | Xác nhận chưa tạo và retry an toàn? | **Có; hãng bảo đảm** → `S_Retry` → `S_Call`; **Không** → `S_Manual`. |

### 3.4. Alternate / Exception — UC-02

| Mã | Điều kiện | Xử lý |
| --- | --- | --- |
| A1 | Chỉnh sửa trước xác nhận | Nếu thay đổi ảnh hưởng phí/dịch vụ, quay UC-01 để báo giá mới. |
| A2 | Rate hết hạn/hết khả dụng hoặc dữ liệu không đạt | Yêu cầu tra cứu/chọn lại; không tạo theo giá cũ. |
| A3 | Thiếu trường tùy chọn | Chỉ hiển thị thông tin có căn cứ; phần còn lại “Chưa có dữ liệu”. |
| A4 | Xác nhận nhiều lần | Dùng lại booking_key; SUCCESS trả đơn cũ, PENDING/UNKNOWN chờ xử lý, FAILED trả trạng thái đã lưu. |
| E1 | Carrier từ chối | Đặt FAILED, hiển thị lý do có căn cứ. |
| E2 | Timeout/mất kết nối | Đặt UNKNOWN, giữ key/attempts; xác minh trước retry; không thể xác minh → thủ công. |
| E3 | Phản hồi thiếu căn cứ hoặc lỗi lưu | Không báo hoàn tất; ghi dấu vết và xử lý khôi phục an toàn. |

## 4. Quy tắc và giới hạn nghiệp vụ (kế thừa từ bản cũ)

- **Ánh xạ FR hiện hành theo bản nhóm chốt:** WF-01 liên quan FR-01 (dữ liệu đầu vào), FR-02 (Multi-Carrier Shipping Quotation), FR-03 (Shipping Rate Comparison & ETA), FR-08 (thông báo lỗi). WF-02 liên quan FR-01 (tài khoản), FR-04 (Shipment Booking), FR-08 (thông báo lỗi); FR-07 liên quan lưu thông tin COD và FR-05 liên quan xem chi tiết đơn.
- **Không trộn mã FR giữa các phiên bản:** Tài liệu D02 trước đây dùng các mã chi tiết như FR-02–FR-08, FR-09–FR-12, FR-17 từ SRS cũ. Tài liệu này kế thừa mã FR-01–FR-10 theo bộ FR được nhóm chốt nêu trong `process-spec.md` cũ.
- **WF-01:** Hai hãng được gọi độc lập; lỗi một hãng không chặn hãng kia. Phân biệt không có dịch vụ (A5) với lỗi cả hai API (E2). Không tự đoán ETA hoặc thay dữ liệu phí thiếu bằng 0.
- **WF-02:** `booking_key` bất biến cho cùng một yêu cầu. `booking_status` gồm `PENDING`, `SUCCESS`, `FAILED`, `UNKNOWN`. Trạng thái `UNKNOWN` không đồng nghĩa `FAILED`; không tự động book lại khi PENDING/UNKNOWN.
- **WF-02 — xác minh và retry:** Khi timeout, ghi UNKNOWN, giữ key và booking_attempts. Chỉ đối chiếu carrier bằng tham chiếu nếu hãng hỗ trợ. Chỉ retry khi hãng xác nhận chưa tạo **và** chứng minh được retry an toàn; nếu không thì giữ UNKNOWN để xử lý thủ công.
- **NFR:** NFR-01 — timeout tối đa 10 giây/yêu cầu API báo giá hoặc tạo đơn. NFR-02 — bảo vệ token ở server. NFR-03 — tránh trùng booking, tránh báo thành công giả. NFR-04 — toàn vẹn dữ liệu. NFR-05 — tích hợp adapter độc lập, ghi lỗi.
- **Phạm vi BPMN:** `booking_status` phản ánh trạng thái tạo đơn, khác `shipment_status` trong vận chuyển. Không đưa thanh toán trực tuyến, đối soát COD hoặc RepeatBox scheduler vào hai quy trình P0.

## 5. Đối chiếu thay đổi và file bàn giao

| Hạng mục | WF-01 | WF-02 |
| --- | --- | --- |
| Start/End Event | 1 Start + 3 End đã dùng tên sự kiện/kết quả, không có tiền tố “Bắt đầu/Kết thúc”. | 1 Start + 10 End đã dùng tên sự kiện/kết quả, không có tiền tố “Bắt đầu/Kết thúc”. |
| Task | 15 Task, tên trên sơ đồ được trình bày nguyên văn ở §2.2. | 24 Task, tên trên sơ đồ được trình bày nguyên văn ở §3.2. |
| Gateway | 6 Gateway, mô tả ở §2.3; không đổi tên/luồng. | 8 Gateway, mô tả ở §3.3; không đổi tên/luồng. |
| Nội dung chi tiết | Business Rule và Exception được giữ ở các bảng mô tả. | Giữ các nhánh idempotency, UNKNOWN, xác minh và retry an toàn. |

**Các file dùng để đối chiếu trong lần sửa này:** `WF01-quote(1).bpmn`, `WF02-create-shipment(1).bpmn`, `process-spec.md` (bản cũ).  
**File kết quả:** `process-spec-updated.md`.  
**Lưu ý bàn giao:** Bản cũ có liệt kê PNG/PDF và DOCX/PDF báo cáo; việc cập nhật lần này chỉ tạo file Markdown và không xác nhận các bản xuất đó đã được cập nhật theo tên mới.
