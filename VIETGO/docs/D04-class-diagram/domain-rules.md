# D04 — Class Diagram & Domain Rules | VietGo

## 1. Mục tiêu và phạm vi

Tài liệu mô tả cấu trúc lớp, trách nhiệm của từng lớp, quan hệ UML và các quy tắc nghiệp vụ trong hệ thống VietGo. Class Diagram được xây dựng dựa trên yêu cầu chức năng và thiết kế cơ sở dữ liệu D03.

Phạm vi mô hình bao gồm:

- **UC-01:** Lấy và so sánh báo giá vận chuyển GHN/GHTK.
- **UC-02:** Chọn dịch vụ và tạo vận đơn.
- **UC-03:** Theo dõi vận đơn và lịch sử vận chuyển.
- **UC-04:** Xem thông tin COD.
- **UC-05 (P1):** RepeatBox — quản lý kế hoạch gửi hàng định kỳ.
- **UC-06:** Tiếp nhận và xử lý khiếu nại vận chuyển.

UC-06 được bổ sung vào mô hình hiện có, không thay thế các chức năng UC-01 đến UC-05.

## 2. Tổ chức Class Diagram

### 2.1. Domain P0 — Thực thể nghiệp vụ

Bao gồm 12 class:

| Class | Trách nhiệm |
|---|---|
| User | Quản lý thông tin người dùng và quyền truy cập |
| RateRequest | Lưu yêu cầu báo giá |
| Parcel | Lưu thông tin kiện hàng |
| ParcelItem | Lưu các mặt hàng trong kiện |
| QuoteAttempt | Ghi nhận từng lần gọi API báo giá |
| Rate | Lưu phương án vận chuyển và chi phí |
| Carrier | Quản lý thông tin hãng vận chuyển |
| CarrierService | Quản lý dịch vụ của từng hãng |
| Shipment | Quản lý vận đơn và thông tin COD |
| BookingAttempt | Ghi nhận quá trình gọi API tạo vận đơn |
| TrackingEvent | Lưu lịch sử trạng thái vận chuyển |
| ShipmentStatus | Danh mục trạng thái vận chuyển chuẩn hóa |

### 2.2. Application Services P0

- **QuoteService:** Điều phối lấy báo giá từ các hãng và chuẩn hóa kết quả.
- **ShipmentService:** Điều phối tạo vận đơn, quản lý kết quả booking và xử lý timeout.
- **TrackingService:** Tra cứu trạng thái vận chuyển và lịch sử tracking.

### 2.3. Carrier Integration P0

- **CarrierAdapter:** Interface định nghĩa các thao tác tích hợp chung.
- **GHNAdapter:** Triển khai tích hợp vận chuyển GHN.
- **GHTKAdapter:** Triển khai tích hợp vận chuyển GHTK.
- **BookingResult:** DTO biểu diễn kết quả tạo vận đơn.

GHNAdapter và GHTKAdapter hiện thực CarrierAdapter, giúp các Application Service không phụ thuộc trực tiếp vào cấu trúc API riêng của từng hãng.

### 2.4. RepeatBox P1

- **ShipmentTemplate:** Lưu mẫu thông tin gửi hàng.
- **RepeatPlan:** Quản lý kế hoạch gửi hàng định kỳ.
- **RepeatOccurrence:** Biểu diễn từng kỳ gửi hàng.
- **RepeatBoxService:** Tạo bản nháp và quản lý các kỳ gửi.

RepeatBox chỉ hỗ trợ chuẩn bị vận đơn. Scheduler không tự động đặt vận đơn với hãng vận chuyển.

### 2.5. Value Types / Enums

Hệ thống sử dụng các enum:

- UserRole: USER, ADMIN.
- BookingStatus: PENDING, SUCCESS, FAILED, UNKNOWN.
- RepeatFrequency: WEEKLY, MONTHLY.
- RepeatPlanStatus: ACTIVE, PAUSED, ENDED.
- RepeatOccurrenceStatus: DRAFT_READY, SKIPPED, SHIPMENT_CREATED.

### 2.6. Complaint Module — UC-06

Complaint Module được bổ sung để hỗ trợ tiếp nhận, lưu trữ, theo dõi và xử lý khiếu nại liên quan đến vận đơn.

Module gồm 6 class:

| Class | Trách nhiệm |
|---|---|
| Complaint | Biểu diễn phiếu khiếu nại và trạng thái xử lý |
| ComplaintEvidence | Quản lý bằng chứng, hình ảnh và tài liệu |
| ComplaintHistory | Ghi lịch sử thay đổi và phản hồi |
| ComplaintService | Điều phối nghiệp vụ khiếu nại |
| ComplaintRepository | Lưu trữ và truy xuất dữ liệu khiếu nại |
| CarrierSupportGateway | Điều phối chuyển tiếp yêu cầu đến kênh hỗ trợ hãng |

Complaint Module dùng chung cho GHN và GHTK nhưng hỗ trợ hai phương thức chuyển tiếp khác nhau.

## 3. Business Rules — Quy tắc nghiệp vụ

### 3.1. UC-01 — Báo giá và so sánh vận chuyển

**BR-01:** Mỗi RateRequest thuộc về một User và có thể phát sinh nhiều QuoteAttempt.

**BR-02:** Mỗi QuoteAttempt ghi nhận một lần gọi báo giá đến Carrier cụ thể. Kết quả báo giá của GHN và GHTK phải được xử lý độc lập.

**BR-03:** Mỗi Rate phải thuộc đúng CarrierService và Carrier được sử dụng trong yêu cầu báo giá.

**BR-04:** VietGo không tự suy đoán ETA hoặc thành phần chi phí nếu hãng không cung cấp.

**BR-05:** Phương án vận chuyển chỉ được lựa chọn khi đáp ứng điều kiện khả dụng và hiệu lực của Rate.

### 3.2. UC-02 — Tạo vận đơn

**BR-06:** Mỗi yêu cầu xác nhận tạo vận đơn sử dụng bookingKey duy nhất để hạn chế việc tạo vận đơn trùng lặp.

**BR-07:** bookingStatus có bốn trạng thái: PENDING, SUCCESS, FAILED và UNKNOWN.

**BR-08:** Khi hãng trả kết quả thành công có căn cứ và hệ thống lưu vận đơn an toàn, bookingStatus được chuyển thành SUCCESS.

**BR-09:** Khi hãng từ chối rõ ràng, bookingStatus được chuyển thành FAILED.

**BR-10:** Khi timeout, mất kết nối hoặc phản hồi không đủ căn cứ xác nhận, bookingStatus phải là UNKNOWN, không được tự kết luận SUCCESS hoặc FAILED.

**BR-11:** Với bookingStatus UNKNOWN, VietGo cần xác minh kết quả với hãng trước khi quyết định thử lại.

**BR-12:** Không được tạo nhiều yêu cầu booking đồng thời cho cùng một lần xác nhận.

**BR-13:** Theo ràng buộc D03 đã thống nhất, một Rate được tham chiếu bởi tối đa một Shipment.

### 3.3. UC-03 — Theo dõi vận đơn

**BR-14:** Người dùng chỉ được xem vận đơn mà mình có quyền truy cập.

**BR-15:** TrackingEvent phải lưu thông tin trạng thái gốc từ hãng để phục vụ đối chiếu và kiểm tra.

**BR-16:** Trạng thái vận chuyển được chuẩn hóa thông qua ShipmentStatus.

**BR-17:** Khi gặp mã trạng thái chưa được nhận diện, VietGo không tự suy đoán trạng thái mới.

**BR-18:** bookingStatus và shipmentStatus là hai khái niệm độc lập. Tạo vận đơn thành công không đồng nghĩa với giao hàng thành công.

### 3.4. UC-04 — Thông tin COD

**BR-19:** Shipment lưu số tiền COD yêu cầu thu hộ.

**BR-20:** Thông tin COD do hãng cung cấp phải được thể hiện theo dữ liệu thực tế nhận được.

**BR-21:** Không tự kết luận COD đã thu hoặc đã đối soát khi chưa có dữ liệu xác nhận.

**BR-22:** Trong phạm vi MVP, VietGo không trực tiếp thực hiện thu hộ, chuyển tiền hoặc đối soát COD thay hãng vận chuyển.

### 3.5. UC-05 — RepeatBox P1

**BR-23:** Mỗi RepeatPlan được tạo từ một ShipmentTemplate thuộc quyền sở hữu hợp lệ của người dùng.

**BR-24:** Mỗi kỳ gửi được biểu diễn bằng RepeatOccurrence.

**BR-25:** Cặp giá trị (repeatPlanId, scheduledFor) phải duy nhất để tránh tạo trùng kỳ gửi.

**BR-26:** Scheduler chỉ tạo bản nháp nội bộ và không tự gọi API booking.

**BR-27:** Người dùng phải kiểm tra thông tin, lấy báo giá mới và xác nhận trước khi tạo vận đơn.

**BR-28:** RepeatOccurrence chỉ được liên kết với Shipment khi việc tạo vận đơn đã được xác nhận thành công.

### 3.6. UC-06 — Tiếp nhận và xử lý khiếu nại

**BR-29 — Quyền tạo khiếu nại**

Chỉ người dùng có quyền truy cập vận đơn mới được tạo khiếu nại liên quan đến vận đơn đó.

Mỗi Complaint phải liên kết với một Shipment tồn tại trong hệ thống.

**BR-30 — Kiểm tra thông tin**

Trước khi tạo khiếu nại, ComplaintService phải kiểm tra dữ liệu bắt buộc như lý do, nội dung mô tả và vận đơn liên quan.

Nếu dữ liệu không hợp lệ, hệ thống từ chối yêu cầu và thông báo nội dung cần chỉnh sửa.

**BR-31 — Quản lý bằng chứng**

Mỗi Complaint có thể chứa nhiều ComplaintEvidence.

Tệp bằng chứng cần được kiểm tra định dạng, kích thước và điều kiện hợp lệ trước khi chấp nhận.

Bằng chứng phải được liên kết với đúng Complaint.

**BR-32 — Tạo phiếu khiếu nại**

Sau khi dữ liệu được xác thực, hệ thống tạo Complaint và lưu thông tin qua ComplaintRepository.

Phiếu khiếu nại có định danh nội bộ của VietGo, độc lập với mã ticket do hãng vận chuyển cung cấp.

**BR-33 — Quản lý trạng thái**

Complaint.status biểu diễn trạng thái xử lý khiếu nại.

Các trạng thái nghiệp vụ đề xuất gồm:

- OPEN: Khiếu nại đã được tạo.
- IN_REVIEW: CSKH đang kiểm tra.
- NEED_MORE_INFO: Cần bổ sung thông tin hoặc chứng từ.
- FORWARDED: Đã chuyển yêu cầu đến hãng.
- RESOLVED: Đã có kết quả xử lý.
- CLOSED: Khiếu nại đã đóng.

Đây là bộ trạng thái đề xuất cho VietGo, cần đồng bộ với D03 và quy trình BPMN WF-06 trước khi triển khai.

**BR-34 — Lịch sử xử lý**

Mọi thay đổi trạng thái quan trọng và hoạt động xử lý khiếu nại cần được ghi nhận bằng ComplaintHistory.

Lịch sử có thể bao gồm hành động, trạng thái trước và sau, thời điểm cập nhật và người thực hiện.

Không được xóa lịch sử nghiệp vụ chỉ vì Complaint đã đóng.

**BR-35 — Chuyển tiếp khiếu nại GHN**

Với vận đơn GHN, CarrierSupportGateway có thể tích hợp API ticket chính thức của GHN khi hệ thống có token và quyền truy cập phù hợp.

Các thao tác tích hợp bao gồm tạo ticket, tra cứu chi tiết, phản hồi và tiếp nhận callback theo tài liệu API được xác minh.

Mã ticket do GHN cung cấp được lưu trong carrierTicketId khi có dữ liệu hợp lệ.

**BR-36 — Chuyển tiếp khiếu nại GHTK**

Với vận đơn GHTK, hệ thống sử dụng quy trình chuyển tiếp qua CSKH và kênh hỗ trợ chính thức của GHTK.

Do chưa xác minh API khiếu nại công khai của GHTK, VietGo không tự tạo hoặc giả định endpoint khiếu nại cho hãng.

CSKH có trách nhiệm ghi nhận việc chuyển tiếp và cập nhật phản hồi vào VietGo.

**BR-37 — Xử lý lỗi và timeout**

Khi yêu cầu đến hãng thất bại, timeout hoặc chưa nhận được phản hồi, hệ thống phải giữ lại Complaint và thông tin xử lý đã lưu.

Không được thông báo đã chuyển tiếp thành công khi chưa có căn cứ.

Trường hợp chưa rõ kết quả phải được kiểm tra trước khi gửi lại nhằm hạn chế tạo ticket trùng.

**BR-38 — Xử lý bất đồng bộ**

Việc hãng vận chuyển tiếp nhận và phản hồi khiếu nại có thể diễn ra sau thời điểm VietGo tạo Complaint.

VietGo cần hỗ trợ cập nhật trạng thái trong những lần xử lý tiếp theo mà không yêu cầu người dùng gửi lại toàn bộ khiếu nại.

**BR-39 — Đóng khiếu nại**

Complaint chỉ được chuyển sang CLOSED khi đáp ứng điều kiện đóng theo quy trình nghiệp vụ.

Trước khi đóng, kết quả xử lý phải được lưu và lịch sử cập nhật phải được ghi nhận.

**BR-40 — Phân quyền CSKH**

Người dùng thông thường được tạo và theo dõi khiếu nại thuộc quyền của mình.

Nhân viên CSKH được tiếp nhận, yêu cầu bổ sung chứng từ, chuyển tiếp và cập nhật trạng thái theo quyền được cấp.

Các thao tác cập nhật phải được kiểm tra quyền ở phía backend.

## 4. UML Relationships

### 4.1. Association

Association thể hiện mối liên hệ giữa các đối tượng trong hệ thống.

| Class A | Class B | Multiplicity | Tên quan hệ |
|---|---|---|---|
| User | RateRequest | 1 — 0..* | creates |
| Carrier | CarrierService | 1 — 0..* | provides |
| CarrierService | Rate | 1 — 0..* | offered in |
| User | Shipment | 1 — 0..* | owns |
| Rate | Shipment | 1 — 0..1 | selected for |
| User | ShipmentTemplate | 1 — 0..* | owns |
| ShipmentTemplate | RepeatPlan | 1 — 0..* | used by |
| User | Complaint | 1 — 0..* | submits |
| Shipment | Complaint | 1 — 0..* | has complaints |

Một User có thể gửi nhiều Complaint. Mỗi Complaint thuộc về một User và liên kết với đúng một Shipment.

### 4.2. Composition

Composition biểu diễn quan hệ sở hữu thành phần trong mô hình nghiệp vụ.

| Class chủ | Class thành phần | Multiplicity | Tên quan hệ |
|---|---|---|---|
| RateRequest | Parcel | 1 — 0..1 | has |
| Parcel | ParcelItem | 1 — 0..* | contains |
| RateRequest | QuoteAttempt | 1 — 0..* | logs |
| QuoteAttempt | Rate | 1 — 0..* | returns |
| Shipment | BookingAttempt | 1 — 0..* | records |
| Shipment | TrackingEvent | 1 — 0..* | has history |
| RepeatPlan | RepeatOccurrence | 1 — 0..* | schedules |
| Complaint | ComplaintEvidence | 1 — 0..* | has evidence |
| Complaint | ComplaintHistory | 1 — 0..* | records history |

Trong Class Diagram, hình thoi đen của hai quan hệ ComplaintEvidence và ComplaintHistory được đặt tại phía Complaint.

Quan hệ Composition mô tả sở hữu trong mô hình đối tượng; không đồng nghĩa với việc được phép xóa vĩnh viễn bằng chứng hoặc lịch sử trong cơ sở dữ liệu. Quy định lưu trữ và xóa dữ liệu phải được xác định riêng.

### 4.3. Directed Association

| Nguồn | Đích | Multiplicity | Tên quan hệ |
|---|---|---|---|
| QuoteAttempt | Carrier | 0..* → 1 | for |
| Shipment | ShipmentStatus | 0..* → 0..1 | current status |
| TrackingEvent | ShipmentStatus | 0..* → 0..1 | mapped to |
| RepeatOccurrence | Shipment | 0..1 → 0..1 | links after success |
| RepeatOccurrence | RateRequest | 0..1 → 0..1 | re-quotes via |

### 4.4. Dependency

Dependency là quan hệ phụ thuộc, thể hiện bằng đường nét đứt có đầu mũi tên mở hướng về thành phần được sử dụng.

| Class nguồn | Class đích | Tên quan hệ |
|---|---|---|
| QuoteService | CarrierAdapter | uses |
| ShipmentService | CarrierAdapter | uses |
| TrackingService | CarrierAdapter | uses |
| RepeatBoxService | RepeatPlan | manages |
| RepeatBoxService | RepeatOccurrence | creates |
| CarrierAdapter | BookingResult | returns |
| ComplaintService | Complaint | manages |
| ComplaintService | ComplaintRepository | uses |
| ComplaintService | CarrierSupportGateway | forwards via |
| ComplaintRepository | Complaint | persists |

ComplaintService chịu trách nhiệm điều phối nghiệp vụ, ComplaintRepository quản lý lưu trữ, còn CarrierSupportGateway đóng vai trò tích hợp với hệ thống hỗ trợ của hãng.

### 4.5. Realization

| Class triển khai | Interface |
|---|---|
| GHNAdapter | CarrierAdapter |
| GHTKAdapter | CarrierAdapter |

Realization được thể hiện bằng đường nét đứt và tam giác rỗng hướng về interface.

### 4.6. Quy tắc đọc multiplicity

- `1`: Đúng một đối tượng.
- `0..1`: Không có hoặc có tối đa một đối tượng.
- `0..*`: Không có hoặc có nhiều đối tượng.

Ví dụ: `Shipment 1 — 0..* Complaint` nghĩa là một Shipment có thể có nhiều khiếu nại, nhưng mỗi Complaint liên kết với đúng một Shipment.

## 5. Mapping giữa Class Diagram và ERD D03

### 5.1. Đồng bộ dữ liệu hiện có

Một số thuộc tính được thống nhất với D03:

| Class / thuộc tính | Kiểu dữ liệu trong D04 |
|---|---|
| User.userId | Long |
| Shipment.shipmentId | Long |
| Rate.rateId | Long |
| Shipment.userId | Long |
| Shipment.rateId | Long |
| BookingAttempt.shipmentId | Long |
| Shipment.shipmentStatus | String? |
| ShipmentStatus.statusCode | String |

Định danh của các class khác cần được kiểm tra với schema.dbml tương ứng. Không tự đổi toàn bộ UUID sang Long nếu chưa có căn cứ từ D03.

### 5.2. Mapping dữ liệu UC-06

Complaint Module dự kiến sử dụng ba bảng mới trong D03:

| Class | Bảng dữ liệu |
|---|---|
| Complaint | complaint_tickets |
| ComplaintEvidence | complaint_evidence |
| ComplaintHistory | complaint_history |

ComplaintService, ComplaintRepository và CarrierSupportGateway là các thành phần xử lý, truy xuất dữ liệu và tích hợp; không bắt buộc tương ứng với bảng dữ liệu riêng.

### 5.3. Các ràng buộc cần đồng bộ

- complaint_tickets.shipment_id tham chiếu đến shipments.shipment_id.
- Mỗi Complaint thuộc về một vận đơn.
- complaint_evidence.complaint_id liên kết với Complaint tương ứng.
- complaint_history.complaint_id liên kết với Complaint tương ứng.
- carrier_ticket_id được phép NULL để hỗ trợ trường hợp chưa có mã ticket từ hãng, đặc biệt là quy trình GHTK thủ công.
- Các thay đổi trạng thái quan trọng phải được ghi vào lịch sử xử lý.
- Dữ liệu khiếu nại phải được bảo vệ bằng kiểm tra quyền truy cập.

Tên cột, khóa chính, kiểu ID và ràng buộc chi tiết cần được xác nhận lại theo phiên bản schema.dbml mới nhất do nhóm phụ trách D03 cung cấp.

## 6. Quy tắc tích hợp GHN và GHTK trong UC-06

### 6.1. GHN

VietGo hỗ trợ kết nối hệ thống ticket GHN thông qua CarrierSupportGateway.

Những khả năng tích hợp được đưa vào thiết kế bao gồm:

- Tạo ticket khiếu nại.
- Tra cứu thông tin ticket.
- Gửi phản hồi hoặc bổ sung.
- Tiếp nhận callback cập nhật.

Việc tích hợp thực tế phụ thuộc vào tài liệu API, token và quyền được GHN cấp.

### 6.2. GHTK

VietGo chưa xác nhận được API khiếu nại GHTK công khai.

Do đó, quy trình được thiết kế theo phương án:

1. VietGo tiếp nhận và lưu khiếu nại.
2. Nhân viên CSKH kiểm tra nội dung.
3. CSKH chuyển tiếp đến kênh hỗ trợ chính thức của GHTK.
4. CSKH ghi nhận kết quả trao đổi.
5. VietGo cập nhật trạng thái và lịch sử xử lý.

Không xây dựng hoặc giả định một endpoint API khiếu nại GHTK chưa được xác minh.

### 6.3. Nguyên tắc chung

Complaint Module phải thống nhất cách lưu trữ và theo dõi khiếu nại, đồng thời cho phép các phương thức chuyển tiếp khác nhau theo từng hãng vận chuyển.

Các phản hồi lỗi, timeout hoặc chưa có kết quả không được tự chuyển thành trạng thái xử lý thành công.

## 7. Traceability — Liên kết yêu cầu và mô hình

| Use Case | Class chính | Chức năng |
|---|---|---|
| UC-01 | RateRequest, Rate, QuoteAttempt, QuoteService | Báo giá và so sánh |
| UC-02 | Shipment, BookingAttempt, ShipmentService | Tạo vận đơn |
| UC-03 | TrackingEvent, ShipmentStatus, TrackingService | Theo dõi vận đơn |
| UC-04 | Shipment | Thông tin COD |
| UC-05 | ShipmentTemplate, RepeatPlan, RepeatOccurrence, RepeatBoxService | RepeatBox |
| UC-06 | Complaint, ComplaintEvidence, ComplaintHistory, ComplaintService, ComplaintRepository, CarrierSupportGateway | Xử lý khiếu nại |

Các mã FR, NFR và AC cụ thể cần đối chiếu với D01 và D02 đã cập nhật UC-06 để bảo đảm ma trận truy vết nhất quán.

## 8. UML Conventions

Class Diagram sử dụng các quy ước:

- Dấu `-` biểu diễn thuộc tính hoặc phương thức private.
- Dấu `+` biểu diễn phương thức public.
- Association dùng đường liền.
- Composition dùng đường liền và hình thoi đen ở phía class chủ.
- Dependency dùng đường nét đứt với mũi tên mở.
- Realization dùng đường nét đứt với tam giác rỗng hướng đến interface.
- Multiplicity thể hiện số lượng đối tượng tham gia quan hệ.
- Các class có stereotype `service`, `repository` hoặc `gateway` thể hiện trách nhiệm kiến trúc.

Tên class sử dụng PascalCase; tên thuộc tính và phương thức sử dụng camelCase.

## 9. Deliverables

Cấu trúc file nộp D04:

```text
docs/
└── D04-class-diagram/
    ├── domain-class.puml
    └── domain-rules.md
```

Ngoài mã nguồn, Class Diagram cần được xuất thành hình ảnh PNG hoặc PDF để sử dụng trong báo cáo đồ án.

File domain-class.puml và domain-rules.md phải thống nhất về tên lớp, thuộc tính, kiểu dữ liệu, quan hệ và quy tắc nghiệp vụ.

## 10. Kết luận

Class Diagram D04 của VietGo mô tả các thành phần nghiệp vụ, dịch vụ ứng dụng và tích hợp GHN/GHTK trong một mô hình thống nhất.

Việc bổ sung Complaint Module cho UC-06 giúp VietGo quản lý vòng đời khiếu nại, lưu bằng chứng và lịch sử xử lý, hỗ trợ chuyển tiếp đến GHN thông qua API chính thức hoặc GHTK qua CSKH.

Thiết kế giữ nguyên các chức năng hiện có, mở rộng theo hướng phân tách trách nhiệm giữa Domain, Application Service, Repository và Gateway, đồng thời cần được đồng bộ với ERD D03, Use Case D02 và quy trình BPMN WF-06 trước khi triển khai.
