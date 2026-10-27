# VIETGO — SHIPMENT TRANSITION RULES

**Module:** UC-03 — Theo dõi trạng thái vận chuyển  
**Phiên bản:** 1.0 — bản thiết kế đề xuất, 26/10/2026  
**Phạm vi:** Vận đơn GHN/GHTK đã được VietGo tạo thành công. Không bao gồm vòng đời kế hoạch RepeatBox, thanh toán hoặc đối soát COD.

## 1. Mục đích và quy ước

Tài liệu mô tả điều kiện chuyển trạng thái của `shipmentStatus` sau khi VietGo nhận dữ liệu tracking từ GHN/GHTK, chuẩn hóa bằng **Status Mapping**, rồi kiểm tra bằng **State Machine**. Đây là **quy tắc thiết kế nội bộ VietGo**, không phải cam kết rằng GHN/GHTK luôn phát sinh sự kiện theo đúng thứ tự.

- **API Mapping:** chuyển đổi cấu trúc dữ liệu VietGo ↔ GHN/GHTK.
- **Status Mapping:** ánh xạ `carrier + raw_status` → trạng thái chuẩn VietGo.
- **Transition Rules:** quyết định có chấp nhận cập nhật từ trạng thái hiện tại sang trạng thái mới hay không.
- **State Machine:** mô tả trực quan các luồng chuyển trạng thái hợp lệ.
- **Booking và shipment là hai vòng đời khác nhau:** `bookingStatus=SUCCESS` chỉ cho phép khởi tạo shipment; không đồng nghĩa với `DELIVERED`.

### Danh mục trạng thái

| Mã | Diễn giải | Nhóm |
|---|---|---|
| `WAITING_PICKUP` | Chờ hãng lấy hàng | Đang xử lý |
| `PICKED_UP` | Hãng xác nhận đã lấy hàng | Đang xử lý |
| `IN_TRANSIT` | Đang vận chuyển | Đang xử lý |
| `OUT_FOR_DELIVERY` | Đang giao cho người nhận | Đang xử lý |
| `DELIVERY_FAILED` | Lần giao không thành công | Ngoại lệ có thể phục hồi |
| `REDELIVERY` | Đã duyệt giao lại (trạng thái nội bộ đề xuất) | Ngoại lệ có thể phục hồi |
| `RETURNING` | Đang hoàn hàng | Đang xử lý |
| `NEEDS_ATTENTION` | Cần xác minh/xử lý sự cố (trạng thái nội bộ đề xuất) | Ngoại lệ |
| `DELIVERED` | Giao thành công | Kết thúc |
| `RETURNED` | Hoàn hàng hoàn tất | Kết thúc |
| `CANCELLED` | Hủy hợp lệ đã được hãng xác nhận | Kết thúc |

`FINAL STATE`/`END_STATE` trong sơ đồ chỉ là điểm kết thúc trực quan, **không phải** giá trị lưu vào `shipmentStatus`.

## 2. Transition Rules — bảng quy tắc chuyển trạng thái

| ID | Từ trạng thái | Sự kiện/điều kiện | Sang trạng thái | Xử lý |
|---|---|---|---|---|
| TR-01 | Chưa có shipment | Booking thành công; có định danh carrier và đơn VietGo | `WAITING_PICKUP` | Tạo shipment một lần, lưu mapping mã đơn |
| TR-02 | `WAITING_PICKUP` | Hãng xác nhận lấy hàng | `PICKED_UP` | Ghi nhận thời điểm và raw event |
| TR-03 | `PICKED_UP` | Hãng xác nhận điều phối/vận chuyển | `IN_TRANSIT` | Cập nhật tracking |
| TR-04 | `IN_TRANSIT` | Hãng xác nhận bắt đầu giao | `OUT_FOR_DELIVERY` | Cập nhật tracking |
| TR-05 | `OUT_FOR_DELIVERY` | Hãng xác nhận giao thành công | `DELIVERED` | Đánh dấu kết thúc |
| TR-06 | `WAITING_PICKUP` | Hãng xác nhận hủy; chính sách cho phép | `CANCELLED` | Đánh dấu kết thúc |
| TR-07 | `OUT_FOR_DELIVERY` | Hãng xác nhận giao thất bại | `DELIVERY_FAILED` | Lưu lý do thất bại |
| TR-08 | `DELIVERY_FAILED` | Giao lại được duyệt theo quy trình VietGo/carrier | `REDELIVERY` | Ghi quyết định giao lại |
| TR-09 | `REDELIVERY` | Hãng xác nhận bắt đầu giao lại | `OUT_FOR_DELIVERY` | Tăng số lần giao nếu có dữ liệu |
| TR-10 | `DELIVERY_FAILED` | Hãng xác nhận bắt đầu hoàn | `RETURNING` | Chuyển sang luồng hoàn |
| TR-11 | `IN_TRANSIT` | Hãng xác nhận chuyển hoàn trực tiếp | `RETURNING` | Ghi lý do chuyển hoàn |
| TR-12 | `RETURNING` | Hãng xác nhận hoàn hàng hoàn tất | `RETURNED` | Đánh dấu kết thúc |
| TR-13 | `IN_TRANSIT` | Nhận sự kiện sự cố đã xác minh | `NEEDS_ATTENTION` | Lưu raw status, mở xử lý sự cố |
| TR-14 | `NEEDS_ATTENTION` | Có bằng chứng hãng đã xử lý và tiếp tục vận chuyển | `IN_TRANSIT` | Ghi người/quy tắc xác nhận phục hồi |

**Lưu ý thiết kế:** Bảng trên phản ánh các mũi tên trong State Machine hiện tại. Trên thực tế, carrier có thể bỏ qua trạng thái trung gian, gửi event trễ hoặc báo sự cố tại bước khác. Những trường hợp đó **không tự động ép vào luồng trên**; cần bổ sung transition có căn cứ hoặc chuyển sang xử lý ngoại lệ. `REDELIVERY` không phải mã trạng thái carrier mặc định.

## 3. Business Rules

| ID | Quy tắc | Mô tả/điều kiện kiểm tra |
|---|---|---|
| BR-01 | Tách booking và shipment | Chỉ tạo shipment sau khi booking được xác nhận thành công; booking lỗi/không rõ kết quả không tự tạo shipment. |
| BR-02 | Chuẩn hóa trước khi chuyển | Mọi sự kiện carrier phải đi qua Status Mapping và kiểm tra Transition Rules trước khi cập nhật `shipmentStatus`. |
| BR-03 | Lưu bằng chứng gốc | Lưu `carrier`, `raw_status`, `carrier_order_id`, thời gian sự kiện, thời gian nhận, lý do và tham chiếu payload. |
| BR-04 | Không suy đoán trạng thái | Mã lạ hoặc không có mapping: giữ trạng thái hiện tại, ghi nhận để xác minh. |
| BR-05 | Trạng thái kết thúc | `DELIVERED`, `RETURNED`, `CANCELLED` là terminal trong phạm vi MVP; không tự chuyển sang trạng thái khác. |
| BR-06 | Giao thất bại chưa kết thúc | `DELIVERY_FAILED` có thể dẫn đến giao lại hoặc hoàn hàng khi có sự kiện xác nhận. |
| BR-07 | Hủy phải được xác nhận | Yêu cầu hủy không đồng nghĩa `CANCELLED`; chỉ cập nhật sau khi carrier xác nhận. |
| BR-08 | Chống cập nhật trùng | Dùng event ID của hãng nếu có, hoặc khóa suy ra từ carrier + order + raw status + event time; xử lý idempotent. |
| BR-09 | Kiểm tra thứ tự sự kiện | Sự kiện đến trễ không được ghi đè trạng thái mới hơn; đối chiếu timestamp, thứ tự và trạng thái hiện tại. |
| BR-10 | Phân biệt COD/đối soát | Trạng thái tài chính, ví dụ GHTK `status_id=6`, không được ghi đè `shipmentStatus`. |
| BR-11 | Sự kiện thông tin shipper | Các mã GHTK được phân loại là thông tin shipper (ví dụ `123`, `127`, `128`, `45`, `49`, `410`) không tự cập nhật `shipmentStatus` nếu chưa có trạng thái đơn chính thức. |
| BR-12 | Bảo toàn lịch sử | Mỗi lần thay đổi được ghi vào `shipment_status_history`; không xóa raw event cũ. |
| BR-13 | RepeatBox tách riêng | Scheduler chỉ tạo đơn nháp cho kỳ gửi; trạng thái kế hoạch định kỳ và `approval_status` không thuộc Shipment State Machine. |
| BR-14 | Đồng bộ hãng vận chuyển | Khi carrier chưa phản hồi hoặc API timeout, không tự giả định đã giao/hủy; cần retry hoặc đối soát trạng thái. |

## 4. Exception Handling

| ID | Tình huống | Phát hiện | Cách xử lý đề xuất |
|---|---|---|---|
| EX-01 | Raw status chưa được ánh xạ | Không tìm thấy trong Status Mapping | Giữ trạng thái hiện tại; lưu event; cảnh báo bổ sung mapping. |
| EX-02 | Chuyển trạng thái không hợp lệ | Cặp `from → to` không nằm trong quy tắc | Không ghi đè; lưu vào hàng chờ kiểm tra, đối chiếu API carrier. |
| EX-03 | Webhook trùng | Trùng event ID/khóa idempotency | Trả ACK an toàn; không ghi thêm transition trùng. |
| EX-04 | Webhook đến sai thứ tự | Timestamp cũ hơn event đã chấp nhận | Lưu lịch sử nhưng không tự rollback trạng thái; xác minh khi mâu thuẫn. |
| EX-05 | Mất kết nối/timeout carrier | API lỗi, timeout, HTTP 5xx | Retry có backoff; kiểm tra lại bằng API tra cứu; không tự gán trạng thái thành công. |
| EX-06 | Carrier báo giao thất bại | Mapping thành `DELIVERY_FAILED` | Lưu lý do; chờ quyết định giao lại hoặc hoàn hàng. |
| EX-07 | Ngoại lệ, mất, hư hỏng | GHN `exception`/`lost`/`damage` hoặc mã tương đương đã xác minh | Đưa vào `NEEDS_ATTENTION` nếu transition hợp lệ; yêu cầu xác minh và theo dõi xử lý. |
| EX-08 | Yêu cầu hủy bị từ chối | Carrier không xác nhận hủy | Giữ shipment status cũ; ghi nhận kết quả hủy thất bại. |
| EX-09 | Sự kiện sau trạng thái terminal | Nhận event mới sau `DELIVERED`/`RETURNED`/`CANCELLED` | Không đổi trạng thái tự động; chuyển kiểm tra đối soát/correction thủ công. |
| EX-10 | Event tài chính hoặc shipper info | Mã không biểu thị trạng thái vận đơn chính thức | Lưu vào lịch sử sự kiện/phân hệ COD; không cập nhật `shipmentStatus`. |
| EX-11 | Không tìm thấy đơn tương ứng | Không map được carrier order ID với VietGo | Lưu event vào hàng chờ; tra cứu lại mã đơn, không tạo shipment giả. |
| EX-12 | Carrier báo trạng thái trung gian bị bỏ qua | Ví dụ từ `WAITING_PICKUP` trực tiếp `IN_TRANSIT` | Xác minh qua API tracking; chỉ cho phép nếu nhóm bổ sung transition được phê duyệt. |

## 5. Quy trình xử lý sự kiện tracking

1. VietGo nhận webhook hoặc polling response từ GHN/GHTK.
2. Kiểm tra tính hợp lệ của nguồn, mã đơn, cấu trúc dữ liệu và khả năng xác thực (nếu carrier hỗ trợ).
3. Lưu raw event, kiểm tra idempotency và thứ tự sự kiện.
4. Áp dụng Status Mapping; nếu là sự kiện tài chính/thông tin shipper thì phân luồng riêng.
5. So sánh `currentStatus` và `mappedStatus` theo Transition Rules.
6. Nếu hợp lệ: cập nhật trạng thái trong transaction và ghi lịch sử. Nếu không hợp lệ: giữ trạng thái hiện tại, lưu exception để đối soát.
7. Trả ACK theo yêu cầu tích hợp; các tác vụ nặng xử lý bất đồng bộ nếu phù hợp.

**Pseudo-rule:** `mappedStatus = map(carrier, rawStatus)` → `isAllowed(currentStatus, mappedStatus)` → `updateShipmentAndHistory()`; nếu không hợp lệ → `recordException()`.

## 6. Acceptance Test Cases

| TC | Given | When | Then |
|---|---|---|---|
| TC-01 | Booking thành công | Tạo shipment | `WAITING_PICKUP` |
| TC-02 | `WAITING_PICKUP` | Carrier xác nhận lấy | `PICKED_UP` |
| TC-03 | `OUT_FOR_DELIVERY` | Carrier xác nhận giao thành công | `DELIVERED`, terminal |
| TC-04 | `OUT_FOR_DELIVERY` | Carrier báo giao thất bại | `DELIVERY_FAILED`, không terminal |
| TC-05 | `DELIVERY_FAILED` | Giao lại được duyệt | `REDELIVERY` |
| TC-06 | `RETURNING` | Carrier xác nhận hoàn tất | `RETURNED`, terminal |
| TC-07 | `WAITING_PICKUP` | Carrier xác nhận hủy | `CANCELLED`, terminal |
| TC-08 | Bất kỳ trạng thái | Raw status chưa nhận diện | Giữ trạng thái cũ, ghi exception |
| TC-09 | Đã xử lý event | Nhận lại cùng event | Không cập nhật trùng |
| TC-10 | `DELIVERED` | Nhận event cũ `IN_TRANSIT` | Không rollback |
| TC-11 | `IN_TRANSIT` | GHN báo `lost` đã xác minh | `NEEDS_ATTENTION` |
| TC-12 | Shipment đang vận chuyển | Nhận mã đối soát GHTK | Không đổi `shipmentStatus` |

## 7. Điểm cần nhóm xác nhận trước khi chốt

1. **`REDELIVERY`:** đây là trạng thái nội bộ bổ sung; cần thêm vào Status Mapping/định nghĩa canonical nếu giữ trong State Machine.
2. **`NEEDS_ATTENTION`:** bảng mapping có thể gộp nhiều lỗi vào đây, nhưng State Machine hiện mới vẽ transition từ `IN_TRANSIT`; cần mở rộng có kiểm soát nếu muốn hỗ trợ sự cố ở giai đoạn khác.
3. **Bỏ qua trạng thái trung gian:** bản sơ đồ cuối hiện chưa có nhánh `WAITING_PICKUP → IN_TRANSIT`, dù các bản đề xuất trước từng có; nhóm phải chọn một phiên bản thống nhất.
4. **Các quy tắc phục hồi:** chỉ chuyển từ `NEEDS_ATTENTION` sang `IN_TRANSIT` khi có bằng chứng carrier xác nhận; trường hợp hoàn/hủy/sự cố không phục hồi cần đặc tả riêng.
5. **Giới hạn nguồn:** quy tắc trên là đặc tả nội bộ dựa vào sơ đồ nhóm đang dùng và mapping đã đề xuất; chưa phải bằng chứng API thật đã được kiểm thử. Cần kiểm tra với tài liệu chính thức và token/sandbox (hoặc Mock API).

## 8. Tài liệu tham chiếu

- API Mapping GHN – GHTK – VietGo (file nhóm cung cấp).
- Status Mapping và Mapping Notes (sheet bổ sung của nhóm).
- State Machine PlantUML — phiên bản trao đổi cuối cùng.
- GHN Developer Docs: https://developer.ghn.vn/en/docs/order/create
- GHTK API Docs: https://api.ghtk.vn/docs/submit-order/submit-order-express/

> **Ghi chú:** Những bảng trên là tài liệu thiết kế phục vụ đồ án. Các chi tiết về sự kiện, trường webhook và quyền thực thi cần được xác minh với tài liệu API hiện hành trước khi triển khai production.
