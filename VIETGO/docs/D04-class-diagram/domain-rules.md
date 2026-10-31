# D04 — Class Diagram & Domain Rules | VietGo

## 1. Mục đích và phạm vi

- Một **UML Class Diagram tổng thể**, tập trung vào hệ thống VietGo MVP P0: tài khoản, tra cứu và so sánh báo giá GHN/GHTK (UC-01), tạo vận đơn (UC-02), tracking (UC-03), xem thông tin COD (UC-04).
- RepeatBox (UC-05) là **P1 ngoài MVP**, được tách package trong cùng sơ đồ, không lẫn với luồng bắt buộc P0.
- Đối chiếu với tab **ĐỒ ÁN VIETGO**, **ERD**, **CUSTOMIZE**, **FR - NFR** trong tài liệu nhóm; đối chiếu yêu cầu mục **3.4 Class Diagram** của template thầy.
- Sơ đồ mô tả lớp thiết kế, **không phải bản ERD**. Các entity tương ứng với 15 bảng D03; có thêm application service, adapter, enum và DTO để làm rõ hành vi, không có nghĩa tạo thêm bảng CSDL.

## 2. Các package và vai trò

| Package | Class chính | Trách nhiệm |
|---|---|---|
| DOMAIN — P0 | `User`, `RateRequest`, `Parcel`, `ParcelItem`, `Carrier`, `CarrierService`, `QuoteAttempt`, `Rate`, `Shipment`, `BookingAttempt`, `TrackingEvent`, `ShipmentStatus` | Lưu dữ liệu và quy tắc nghiệp vụ cốt lõi; tương ứng 12 bảng P0 trong D03 |
| APPLICATION SERVICES — P0 | `QuoteService`, `ShipmentService`, `TrackingService` | Điều phối logic UC-01/02/03 và dữ liệu COD của `Shipment` cho UC-04 |
| CARRIER INTEGRATION — P0 | `CarrierAdapter`, `GHNAdapter`, `GHTKAdapter`, `BookingResult` | Độc lập chi tiết API từng hãng; chuẩn hóa báo giá, kết quả tạo đơn, tracking |
| REPEATBOX — P1 | `ShipmentTemplate`, `RepeatPlan`, `RepeatOccurrence`, `RepeatBoxService` | Lưu mẫu, lập kế hoạch, tạo bản nháp nội bộ, chống kỳ trùng |

`Address` không tách thành bảng/lớp domain riêng trong bản này vì D03 đang lưu **snapshot** trường gửi/nhận trong `rate_requests`, `shipments` và `shipment_templates`. `BookingStatus` là **enum trạng thái API tạo đơn**, còn `ShipmentStatus` là **danh mục trạng thái vận chuyển**. `BookingResult` là DTO phản hồi adapter, không phải bảng dữ liệu.

## 3. Quy tắc nghiệp vụ theo domain

### 3.1. UC-01 — Báo giá và so sánh

1. `RateRequest` thuộc đúng `User`, dữ liệu nhập được kiểm tra trước khi gọi API.
2. Mỗi `RateRequest` có tối đa một `Parcel` theo giả định hiện tại của D03, mỗi kiện có thể có nhiều `ParcelItem`.
3. `QuoteService` gọi `CarrierAdapter` cho GHN và GHTK độc lập; lỗi một hãng không được làm hỏng kết quả hợp lệ của hãng kia.
4. Mỗi lần yêu cầu hãng được lưu trong `QuoteAttempt`; `Rate` thuộc một lần thử báo giá và một `CarrierService` phù hợp với `Carrier`.
5. Không tự gán phí thiếu là 0; không tự đoán ETA. `Rate.isSelectable()` kiểm tra dịch vụ khả dụng, phí cần thiết và hiệu lực báo giá.

### 3.2. UC-02 — Xác nhận và tạo vận đơn

1. Người gửi chọn `Rate` hợp lệ rồi chủ động xác nhận. `ShipmentService` chỉ tạo booking với carrier tương ứng.
2. `Shipment.bookingKey` là khóa **UNIQUE, bất biến cho một yêu cầu xác nhận**; các thao tác bấm lặp dùng lại khóa đó. Mỗi khóa tạo tối đa một vận đơn thành công.
3. `BookingAttempt` ghi các lần gọi API (`attemptNo`, trạng thái, mã tham chiếu, thời gian). Không cho hai lần gọi tạo đơn **đồng thời** cùng một `bookingKey`.
4. `BookingStatus`: `PENDING` (đang xử lý), `SUCCESS` (đã xác nhận tạo), `FAILED` (carrier đã từ chối rõ), `UNKNOWN` (timeout/lỗi mà chưa kết luận đã tạo hay chưa).
5. Nếu carrier **đã xác nhận tạo**, lưu mã vận đơn hãng và chuyển `SUCCESS`, không retry.
6. Nếu `UNKNOWN`, **kiểm tra kết quả từ carrier trước khi retry** bằng mã tham chiếu/đối chiếu API khi hãng hỗ trợ. Chỉ thử lại khi có xác nhận chưa tạo **và** cơ chế hãng bảo đảm retry an toàn; nếu chưa thể xác minh, giữ `UNKNOWN`, không tự retry.
7. `bookingStatus` (kết quả tạo vận đơn) **khác** `shipmentStatus` (trạng thái vận chuyển); không gộp hai khái niệm.
8. Lưu snapshot cước, thông tin gửi/nhận, kiện hàng, COD theo dữ liệu có căn cứ. Không báo thành công giả.

### 3.3. UC-03 — Theo dõi vận đơn

1. Người dùng chỉ xem `Shipment` thuộc quyền sở hữu; admin P1 cần kiểm tra vai trò.
2. `TrackingService` dùng API hãng qua adapter, ánh xạ `TrackingEvent.carrierStatus` sang trạng thái chuẩn `ShipmentStatus` khi nhận diện được.
3. Trạng thái chưa nhận diện hoặc lỗi API không được tự gán `Đã giao` hoặc ghi đè trạng thái đáng tin cậy.
4. Lịch sử `TrackingEvent` là tập sự kiện riêng; có thể trống trong khi trạng thái hiện tại vẫn hiển thị được.

### 3.4. UC-04 — COD

1. `Shipment.codAmount` là **số tiền yêu cầu thu hộ**, không phải cước vận chuyển.
2. `Shipment.codStatus` chỉ hiển thị khi có dữ liệu từ hãng. Không suy ra đã thu/đối soát chỉ từ trạng thái `Đã giao`.
3. VietGo P0 **không trực tiếp thu, chuyển hoặc đối soát tiền COD**.

### 3.5. UC-05 — RepeatBox (P1)

1. Mỗi `ShipmentTemplate` thuộc một `User`; mỗi `RepeatPlan` dùng **một mẫu đơn** và thuộc cùng chủ sở hữu.
2. `RepeatPlan` hỗ trợ `WEEKLY`/`MONTHLY` và `ACTIVE`/`PAUSED`/`ENDED`. Kế hoạch `PAUSED` không sinh kỳ mới.
3. `RepeatOccurrence` có ràng buộc `UNIQUE(repeatPlanId, scheduledFor)`; scheduler chạy lại không tạo thêm kỳ/bản nháp trùng.
4. Scheduler và `RepeatBoxService` chỉ tạo bản nháp nội bộ `DRAFT_READY`, **không tự động đặt vận đơn GHN/GHTK**.
5. Người dùng mở bản nháp, lấy **báo giá mới** qua UC-01 và tự xác nhận đặt đơn qua UC-02.
6. `RepeatOccurrence.shipment` chỉ liên kết khi carrier xác nhận booking `SUCCESS`. Có thể bỏ qua kỳ (`SKIPPED`).
7. Việc sửa mẫu đơn chỉ áp dụng cho các kỳ tương lai, không tự sửa vận đơn carrier đã tạo.

## 4. Quan hệ UML và ánh xạ D03

- **Association (`--`)**: User–Shipment, Carrier–CarrierService, Rate–Shipment, Shipment–ShipmentStatus, v.v.
- **Composition (`*--`)**: `RateRequest`–`Parcel` (0..1), `Parcel`–`ParcelItem`, `RateRequest`–`QuoteAttempt`, `QuoteAttempt`–`Rate`, `Shipment`–`BookingAttempt`/`TrackingEvent`, `RepeatPlan`–`RepeatOccurrence`. Tính sở hữu vòng đời trong sơ đồ lớp là quyết định thiết kế nghiệp vụ, không tự động suy ra `ON DELETE CASCADE` ở DB.
- **Realization (`<|..`)**: `GHNAdapter` và `GHTKAdapter` hiện thực `CarrierAdapter`.
- **Dependency (`..>`)**: service sử dụng entity/adapter; các lớp dùng enum.
- **Multiplicity** thể hiện theo quan hệ hiện hành D03; đặc biệt `RateRequest`–`Parcel` là `1`–`0..1`, `RepeatOccurrence`–`Shipment` có liên kết tùy chọn.

Các lớp thêm phương thức mô tả **trách nhiệm thiết kế** chứ không khẳng định đã có code triển khai. Các kiểu `UUID`, `Decimal`, `DateTime`, `JSON` là kiểu ở mức mô hình; khi code/DB cần ánh xạ cụ thể.

## 5. Đối chiếu yêu cầu và file nộp

| Yêu cầu | Thể hiện trong class diagram |
|---|---|
| FR-01 | `User`, `UserRole` |
| FR-02–08 | `RateRequest`, `Parcel`, `QuoteAttempt`, `Rate`, `QuoteService`, adapters |
| FR-09–12, FR-17 | `Shipment`, `BookingAttempt`, `BookingStatus`, `ShipmentService` |
| FR-13–15 | `Shipment`, `TrackingEvent`, `ShipmentStatus`, `TrackingService` |
| FR-16 | COD snapshot/status thuộc `Shipment` |
| FR-19–24 (P1) | `ShipmentTemplate`, `RepeatPlan`, `RepeatOccurrence`, `RepeatBoxService` |

### Nộp GitHub

```text
docs/
└── D04-class-diagram/
    ├── domain-class.puml
    └── domain-rules.md
```

Đưa hình PNG/PDF vào mục **3.4 Class Diagram** trong báo cáo. Template thầy yêu cầu thể hiện các lớp (entity/controller/service/model), thuộc tính/phương thức và quan hệ UML. Sơ đồ đã thể hiện entity/service/adapter; **controller được lược bỏ có chủ đích** vì D04 tập trung domain và controller sẽ xuất hiện trong D07 Sequence và D08 Architecture. Nếu thầy yêu cầu lớp controller trực tiếp trong D04, bổ sung `QuoteController`, `ShipmentController`, `TrackingController` ở package Presentation.

### Nguồn đối chiếu

- Google Docs nhóm: tab `ĐỒ ÁN VIETGO` (D04), `ERD`, `CUSTOMIZE`, `FR - NFR`.
- Template thầy: https://github.com/trinhtin/ec312/blob/main/project-template.md, mục 3.4.
