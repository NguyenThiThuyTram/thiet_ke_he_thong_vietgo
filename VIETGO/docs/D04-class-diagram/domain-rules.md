# D04 — Class Diagram & Domain Rules | VietGo

## 1. Mục đích và phạm vi

Class Diagram mô tả cấu trúc lớp, thuộc tính, phương thức và các mối quan hệ trong hệ thống VietGo, tập trung vào các chức năng thuộc MVP P0 và phần mở rộng RepeatBox P1.

**Phạm vi thiết kế:**

- **UC-01 — Báo giá và so sánh:** Tra cứu cước vận chuyển từ GHN/GHTK, chuẩn hóa và so sánh các phương án báo giá.
- **UC-02 — Tạo vận đơn:** Người dùng lựa chọn dịch vụ, xác nhận đặt đơn và theo dõi kết quả booking.
- **UC-03 — Theo dõi vận chuyển:** Xem trạng thái hiện tại và lịch sử vận chuyển.
- **UC-04 — Thông tin COD:** Quản lý và hiển thị số tiền thu hộ theo thông tin vận đơn.
- **UC-05 — RepeatBox (P1):** Quản lý mẫu đơn, lập lịch gửi hàng định kỳ và tạo bản nháp nội bộ.

Class Diagram gồm **28 thành phần (class, interface và enum), 27 quan hệ UML**, được chia thành 5 package.

Sơ đồ được xây dựng từ các yêu cầu chức năng, Use Case và mô hình dữ liệu ERD D03 của VietGo. Đây là **mô hình thiết kế lớp**, không phải sơ đồ cơ sở dữ liệu; các service, adapter, DTO và enum không mặc nhiên tương ứng với bảng trong CSDL.

---

## 2. Các package và trách nhiệm

| Package | Thành phần | Trách nhiệm |
|---|---|---|
| **Domain P0** | User, RateRequest, Parcel, ParcelItem, QuoteAttempt, Rate, Carrier, CarrierService, Shipment, BookingAttempt, TrackingEvent, ShipmentStatus | Biểu diễn thực thể và quy tắc nghiệp vụ cốt lõi |
| **Application Services P0** | QuoteService, ShipmentService, TrackingService | Điều phối xử lý báo giá, tạo vận đơn và theo dõi |
| **Carrier Integration P0** | CarrierAdapter, GHNAdapter, GHTKAdapter, BookingResult | Tích hợp và chuẩn hóa phản hồi API từ GHN/GHTK |
| **RepeatBox P1** | ShipmentTemplate, RepeatPlan, RepeatOccurrence, RepeatBoxService | Quản lý gửi hàng định kỳ, tạo bản nháp và liên kết vận đơn |
| **Value Types / Enum** | UserRole, BookingStatus, RepeatFrequency, RepeatPlanStatus, RepeatOccurrenceStatus | Định nghĩa các tập giá trị và trạng thái sử dụng trong hệ thống |

### 2.1. Domain P0

Các class thuộc Domain P0 lưu trữ dữ liệu nghiệp vụ và thực hiện những thao tác trực tiếp trên thực thể.

- `User`: Thông tin người dùng và quyền truy cập.
- `RateRequest`: Yêu cầu tra cứu báo giá.
- `Parcel`: Thông tin kiện hàng.
- `ParcelItem`: Các mặt hàng thuộc kiện hàng.
- `QuoteAttempt`: Ghi nhận từng lần gọi API báo giá.
- `Rate`: Phương án cước vận chuyển nhận được.
- `Carrier`: Hãng vận chuyển GHN/GHTK.
- `CarrierService`: Dịch vụ vận chuyển thuộc hãng.
- `Shipment`: Vận đơn do người dùng tạo.
- `BookingAttempt`: Lịch sử các lần yêu cầu tạo vận đơn.
- `TrackingEvent`: Sự kiện cập nhật vận chuyển.
- `ShipmentStatus`: Danh mục trạng thái vận chuyển chuẩn hóa.

### 2.2. Application Services P0

- `QuoteService`: Xử lý yêu cầu báo giá và chuẩn hóa kết quả.
- `ShipmentService`: Điều phối tạo vận đơn và kiểm tra kết quả booking trước khi retry.
- `TrackingService`: Tra cứu trạng thái hiện tại và lịch sử vận chuyển.

### 2.3. Carrier Integration P0

`CarrierAdapter` định nghĩa các phương thức chung để làm việc với hãng vận chuyển:

- `getRates()`: Lấy báo giá.
- `createBooking()`: Gửi yêu cầu tạo vận đơn.
- `findBooking()`: Kiểm tra kết quả booking.
- `getTracking()`: Lấy thông tin vận chuyển.

`GHNAdapter` và `GHTKAdapter` hiện thực interface `CarrierAdapter`. `BookingResult` là DTO chứa kết quả booking, không phải thực thể lưu trữ độc lập.

### 2.4. RepeatBox P1

- `ShipmentTemplate`: Mẫu thông tin gửi hàng.
- `RepeatPlan`: Kế hoạch gửi hàng định kỳ.
- `RepeatOccurrence`: Một kỳ gửi cụ thể theo kế hoạch.
- `RepeatBoxService`: Xử lý việc tạo bản nháp và bỏ qua kỳ gửi.

### 2.5. Value Types / Enum

| Enum | Giá trị |
|---|---|
| `UserRole` | USER, ADMIN |
| `BookingStatus` | PENDING, SUCCESS, FAILED, UNKNOWN |
| `RepeatFrequency` | WEEKLY, MONTHLY |
| `RepeatPlanStatus` | ACTIVE, PAUSED, ENDED |
| `RepeatOccurrenceStatus` | DRAFT_READY, SKIPPED, SHIPMENT_CREATED |

---

## 3. Quy tắc nghiệp vụ theo Use Case

### 3.1. UC-01 — Báo giá và so sánh

**Các class liên quan:** `User`, `RateRequest`, `Parcel`, `ParcelItem`, `QuoteAttempt`, `Carrier`, `CarrierService`, `Rate`, `QuoteService`.

**Quy tắc nghiệp vụ:**

1. Một `User` có thể tạo nhiều `RateRequest`, nhưng mỗi yêu cầu báo giá thuộc một người dùng.
2. `RateRequest` chứa tối đa một `Parcel` theo mô hình hiện tại.
3. Mỗi `Parcel` có thể chứa nhiều `ParcelItem`.
4. Một `RateRequest` có thể phát sinh nhiều `QuoteAttempt` khi hệ thống tra cứu cước.
5. Mỗi `QuoteAttempt` thuộc một `Carrier` cụ thể.
6. Một `QuoteAttempt` có thể nhận nhiều phương án `Rate`.
7. Mỗi `Rate` tương ứng với một `CarrierService`.
8. `QuoteService` gọi API thông qua `CarrierAdapter` để lấy và chuẩn hóa báo giá.
9. Lỗi API của một hãng không làm mất kết quả hợp lệ từ hãng còn lại.
10. Không tự gán phí chưa có dữ liệu thành 0 hoặc tự dự đoán thời gian giao hàng.

**Phương thức tiêu biểu:**

- `RateRequest.validate()`: Kiểm tra tính hợp lệ của yêu cầu.
- `Parcel.validateDimensions()`: Kiểm tra thông tin kiện hàng.
- `ParcelItem.subtotal()`: Tính thành tiền mặt hàng.
- `QuoteAttempt.recordResult()`: Ghi nhận kết quả tra cứu.
- `Rate.isSelectable()`: Kiểm tra khả năng lựa chọn phương án.
- `QuoteService.getQuotes()`: Truy vấn báo giá.
- `QuoteService.normalizeQuotes()`: Chuẩn hóa dữ liệu.

### 3.2. UC-02 — Xác nhận và tạo vận đơn

**Các class liên quan:** `User`, `Rate`, `Shipment`, `BookingAttempt`, `ShipmentService`, `CarrierAdapter`, `BookingResult`.

**Quy tắc nghiệp vụ:**

1. Người dùng phải lựa chọn một `Rate` hợp lệ trước khi xác nhận tạo vận đơn.
2. Một `User` có thể sở hữu nhiều `Shipment`; mỗi vận đơn thuộc một người dùng.
3. Một `Rate` có thể được tham chiếu bởi nhiều `Shipment` theo mô hình hiện tại.
4. `Shipment.bookingKey` là khóa duy nhất dùng để hạn chế tạo vận đơn trùng.
5. Một `Shipment` có thể ghi nhận nhiều `BookingAttempt`.
6. Mỗi `BookingAttempt` thuộc đúng một `Shipment`.
7. Hệ thống không gửi đồng thời nhiều yêu cầu tạo đơn cho cùng một `bookingKey`.
8. `BookingStatus` có bốn giá trị:
   - `PENDING`: Đang xử lý.
   - `SUCCESS`: Carrier đã xác nhận tạo vận đơn.
   - `FAILED`: Carrier từ chối rõ ràng.
   - `UNKNOWN`: Chưa xác định được kết quả do timeout hoặc lỗi kết nối.
9. Khi booking ở trạng thái `UNKNOWN`, `ShipmentService` cần kiểm tra kết quả từ carrier trước khi quyết định retry.
10. Không tự retry khi chưa xác minh được carrier đã tạo vận đơn hay chưa.
11. `BookingStatus` thể hiện kết quả tạo vận đơn, khác với `ShipmentStatus` thể hiện tiến trình vận chuyển.

**Phương thức tiêu biểu:**

- `Shipment.markBookingUnknown()`: Đánh dấu kết quả booking chưa xác định.
- `Shipment.confirmBooking()`: Ghi nhận kết quả carrier xác nhận.
- `BookingAttempt.recordResponse()`: Ghi nhận phản hồi booking.
- `ShipmentService.createShipment()`: Điều phối tạo vận đơn.
- `ShipmentService.checkCarrierBeforeRetry()`: Kiểm tra kết quả carrier trước khi thử lại.

### 3.3. UC-03 — Theo dõi vận chuyển

**Các class liên quan:** `User`, `Shipment`, `TrackingEvent`, `ShipmentStatus`, `TrackingService`.

**Quy tắc nghiệp vụ:**

1. Người dùng chỉ được xem những `Shipment` mà mình có quyền truy cập.
2. Một `Shipment` có thể có nhiều `TrackingEvent`.
3. Mỗi `TrackingEvent` thuộc một `Shipment` cụ thể.
4. `TrackingEvent.carrierStatus` lưu trạng thái gốc từ hãng vận chuyển.
5. `TrackingEvent.normalizedStatus` biểu diễn trạng thái sau khi chuẩn hóa, nếu nhận diện được.
6. `ShipmentStatus` lưu danh mục trạng thái vận chuyển chung.
7. Một `Shipment` có thể tham chiếu trạng thái vận chuyển hiện tại khi có dữ liệu.
8. Nếu API lỗi hoặc không nhận diện được trạng thái mới, hệ thống không tự chuyển vận đơn sang trạng thái `Đã giao`.
9. Lịch sử vận chuyển có thể chưa có sự kiện, ngay cả khi vận đơn đã được tạo.

**Phương thức tiêu biểu:**

- `User.canAccess()`: Kiểm tra quyền truy cập.
- `TrackingEvent.normalize()`: Chuẩn hóa trạng thái.
- `ShipmentStatus.isTerminal()`: Kiểm tra trạng thái kết thúc.
- `TrackingService.getCurrentStatus()`: Xem trạng thái hiện tại.
- `TrackingService.getHistory()`: Xem lịch sử vận chuyển.

### 3.4. UC-04 — Xem thông tin COD

**Class liên quan:** `Shipment`.

**Quy tắc nghiệp vụ:**

1. `Shipment.codAmount` biểu diễn số tiền yêu cầu thu hộ.
2. COD không phải cước vận chuyển và không được gộp với phí giao hàng.
3. Thông tin COD cần được hiển thị dựa trên dữ liệu đáng tin cậy.
4. Không suy luận COD đã thu hoặc đã đối soát chỉ vì vận đơn có trạng thái `Đã giao`.
5. Trong phạm vi MVP P0, VietGo không trực tiếp thu, chuyển hoặc đối soát tiền COD.

**Lưu ý thiết kế:** Class `Shipment` trong sơ đồ hiện tại có thuộc tính `codAmount`, nhưng **chưa khai báo `codStatus`**. Vì vậy, sơ đồ thể hiện số tiền COD; trạng thái thu hộ từ carrier cần được bổ sung trong thiết kế nếu nhóm muốn mô hình hóa chi tiết hơn.

### 3.5. UC-05 — RepeatBox (P1)

**Các class liên quan:** `User`, `ShipmentTemplate`, `RepeatPlan`, `RepeatOccurrence`, `RateRequest`, `Shipment`, `RepeatBoxService`.

**Quy tắc nghiệp vụ:**

1. Một `User` có thể sở hữu nhiều `ShipmentTemplate`.
2. Một `ShipmentTemplate` có thể được nhiều `RepeatPlan` sử dụng.
3. Mỗi `RepeatPlan` thuộc một mẫu đơn cụ thể.
4. `RepeatPlan` hỗ trợ chu kỳ `WEEKLY` hoặc `MONTHLY`.
5. `RepeatPlan` có các trạng thái `ACTIVE`, `PAUSED` và `ENDED`.
6. Một `RepeatPlan` có thể sinh nhiều `RepeatOccurrence`.
7. Cần bảo đảm mỗi cặp `(repeatPlanId, scheduledFor)` chỉ phát sinh một kỳ gửi.
8. `RepeatBoxService` chỉ tạo bản nháp nội bộ khi đến lịch.
9. Không tự động gọi GHN/GHTK để tạo vận đơn khi đến kỳ.
10. Người dùng cần lấy báo giá mới thông qua `RateRequest` trước khi xác nhận gửi.
11. `RepeatOccurrence` chỉ liên kết với `Shipment` khi booking đã thành công.
12. Người dùng có thể bỏ qua một kỳ gửi bằng trạng thái `SKIPPED`.

**Phương thức tiêu biểu:**

- `ShipmentTemplate.updateSnapshot()`: Cập nhật dữ liệu mẫu.
- `RepeatPlan.pause()`: Tạm dừng kế hoạch.
- `RepeatPlan.resume()`: Tiếp tục kế hoạch.
- `RepeatOccurrence.markSkipped()`: Bỏ qua kỳ gửi.
- `RepeatOccurrence.linkShipment()`: Liên kết vận đơn.
- `RepeatBoxService.createDraftForDuePlan()`: Tạo bản nháp theo lịch.
- `RepeatBoxService.skipOccurrence()`: Bỏ qua kỳ gửi.

---

## 4. Các quan hệ UML trong Class Diagram

### 4.1. Association

Association biểu diễn mối liên hệ giữa hai class bằng đường liền.

| Quan hệ | Multiplicity | Ý nghĩa |
|---|---|---|
| User — RateRequest | `1 — 0..*` | Người dùng tạo yêu cầu báo giá |
| QuoteAttempt — Carrier | `0..* — 1` | Mỗi lần tra cứu thuộc một hãng |
| Carrier — CarrierService | `1 — 0..*` | Hãng cung cấp nhiều dịch vụ |
| CarrierService — Rate | `1 — 0..*` | Dịch vụ có thể xuất hiện trong nhiều báo giá |
| User — Shipment | `1 — 0..*` | Người dùng sở hữu vận đơn |
| Rate — Shipment | `1 — 0..*` | Vận đơn được tạo dựa trên phương án đã chọn |
| Shipment — ShipmentStatus | `0..* — 0..1` | Vận đơn tham chiếu trạng thái hiện tại |
| TrackingEvent — ShipmentStatus | `0..* — 0..1` | Sự kiện có thể được ánh xạ sang trạng thái chuẩn |
| User — ShipmentTemplate | `1 — 0..*` | Người dùng sở hữu các mẫu RepeatBox |
| ShipmentTemplate — RepeatPlan | `1 — 0..*` | Kế hoạch sử dụng mẫu đơn |
| RepeatOccurrence — Shipment | `0..1 — 0..1` | Kỳ lặp liên kết vận đơn sau booking thành công |
| RepeatOccurrence — RateRequest | `0..1 — 0..1` | Kỳ lặp liên kết yêu cầu báo giá mới |

### 4.2. Composition

Composition biểu diễn quan hệ toàn thể – thành phần, sử dụng đường liền và hình thoi đen tại phía class sở hữu.

| Class sở hữu | Thành phần | Multiplicity |
|---|---|---|
| RateRequest | Parcel | `1 — 0..1` |
| Parcel | ParcelItem | `1 — 0..*` |
| RateRequest | QuoteAttempt | `1 — 0..*` |
| QuoteAttempt | Rate | `1 — 0..*` |
| Shipment | BookingAttempt | `1 — 0..*` |
| Shipment | TrackingEvent | `1 — 0..*` |
| RepeatPlan | RepeatOccurrence | `1 — 0..*` |

**Lưu ý:** Composition thể hiện quan hệ sở hữu về mặt thiết kế đối tượng, không mặc nhiên yêu cầu xóa dây chuyền dữ liệu trong CSDL.

### 4.3. Dependency

Dependency sử dụng đường nét đứt có mũi tên, thể hiện một class cần sử dụng class khác.

| Class sử dụng | Class được sử dụng |
|---|---|
| QuoteService | CarrierAdapter |
| ShipmentService | CarrierAdapter |
| TrackingService | CarrierAdapter |
| RepeatBoxService | RepeatPlan |
| RepeatBoxService | RepeatOccurrence |
| CarrierAdapter | BookingResult |

### 4.4. Realization

Realization được thể hiện bằng đường nét đứt, đầu tam giác rỗng hướng về interface.

| Class triển khai | Interface |
|---|---|
| GHNAdapter | CarrierAdapter |
| GHTKAdapter | CarrierAdapter |

Hai adapter triển khai giao diện chung, giúp VietGo xử lý khác biệt giữa API của GHN và GHTK mà không cần thay đổi logic nghiệp vụ cốt lõi.

---

## 5. Quy ước thiết kế UML

### 5.1. Cấu trúc class

Mỗi class được chia thành ba phần:

1. Tên class.
2. Danh sách thuộc tính và kiểu dữ liệu.
3. Danh sách phương thức, tham số và kiểu trả về.

**Ký hiệu quyền truy cập:**

- `-`: Private — thuộc tính hoặc phương thức nội bộ.
- `+`: Public — phương thức có thể được sử dụng từ bên ngoài class.

### 5.2. Multiplicity

| Ký hiệu | Ý nghĩa |
|---|---|
| `1` | Đúng một đối tượng |
| `0..1` | Không có hoặc một đối tượng |
| `0..*` | Không có hoặc nhiều đối tượng |

Multiplicity được ghi ở hai đầu quan hệ Association và Composition.

### 5.3. Kiểu dữ liệu

Sơ đồ sử dụng các kiểu dữ liệu ở mức thiết kế, bao gồm:

- `UUID`: Định danh đối tượng trong mô hình hiện tại.
- `String`: Chuỗi ký tự.
- `Integer`: Số nguyên.
- `Decimal`: Số thập phân.
- `Boolean`: Giá trị đúng/sai.
- `Date`: Ngày.
- `DateTime`: Ngày và thời gian.

**Lưu ý:** Kiểu `UUID` trong Class Diagram cần được đối chiếu với ERD D03 cuối cùng để bảo đảm thống nhất kiểu khóa chính và khóa ngoại.

---

## 6. Đối chiếu với yêu cầu chức năng

| Nhóm yêu cầu | Class liên quan |
|---|---|
| FR-01 — Tài khoản | User, UserRole |
| FR-02–08 — Báo giá và so sánh | RateRequest, Parcel, ParcelItem, QuoteAttempt, Rate, Carrier, CarrierService, QuoteService |
| FR-09–12, FR-17 — Tạo vận đơn | Shipment, BookingAttempt, BookingStatus, ShipmentService |
| FR-13–15 — Theo dõi vận chuyển | Shipment, TrackingEvent, ShipmentStatus, TrackingService |
| FR-16 — COD | Shipment |
| FR-19–24 — RepeatBox P1 | ShipmentTemplate, RepeatPlan, RepeatOccurrence, RepeatBoxService |

---

## 7. Kết luận

Class Diagram VietGo mô tả các thực thể nghiệp vụ, thành phần xử lý logic và lớp tích hợp với GHN/GHTK.

Sơ đồ phân biệt rõ:

- **Domain Entity:** Lưu dữ liệu và quy tắc nghiệp vụ.
- **Application Service:** Điều phối các chức năng chính.
- **Carrier Adapter:** Tích hợp API vận chuyển.
- **DTO:** Truyền kết quả giữa các thành phần.
- **Enum:** Biểu diễn các giá trị và trạng thái.
- **RepeatBox P1:** Phần mở rộng được tách khỏi nghiệp vụ MVP P0.

Việc kết hợp Association, Composition, Dependency và Realization giúp thể hiện cấu trúc và trách nhiệm của các lớp, đồng thời hỗ trợ phát triển tiếp Sequence Diagram và System Architecture.

---

## 8. File bàn giao D04

```text
docs/
└── D04-class-diagram/
    ├── domain-class.puml
    └── domain-rules.md
```

- `domain-class.puml`: Mã nguồn UML Class Diagram.
- `domain-rules.md`: Mô tả các class, quan hệ và quy tắc nghiệp vụ.

**Tài liệu đối chiếu:**

- Tài liệu nhóm VietGo: ĐỒ ÁN VIETGO, ERD, CUSTOMIZE, FR - NFR.
- Use Case Specifications D02.
- Template đồ án môn học: https://github.com/trinhtin/ec312/blob/main/project-template.md — mục 3.4 Class Diagram.
