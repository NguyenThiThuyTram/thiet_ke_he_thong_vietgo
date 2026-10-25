# VIETGO - D03 Data Dictionary (P0)

**Trạng thái:** Bản thiết kế đề xuất đã chỉnh sửa, cần nhóm xác nhận cùng D01/D02/D06 trước khi coi là nghiệm thu.

**Nguồn đối chiếu:** `schema (1).dbml` do nhóm cung cấp; tài liệu Google Drive *ĐỒ ÁN VIETGO* (D03, UC-01..UC-04); bảng *API Mapping GHN - GHTK - VietGo* (tab Mapping).

**Quy ước:** PostgreSQL định hướng; `bigint` tiền ở đơn vị VND nguyên, `int` cân nặng ở gram, kích thước `decimal(8,2)` cm; thời điểm lưu `timestamp` (khi triển khai nên thống nhất timezone UTC / `timestamptz`). NULL mang ý nghĩa chưa có thông tin chứ không phải 0.

## 1. Danh mục bảng và ánh xạ nghiệp vụ

| Bảng | Công dụng | Chức năng |
|---|---|---|
| `users` | Đăng ký/đăng nhập, sở hữu dữ liệu | Account; UC-01..04 |
| `rate_requests` | Yêu cầu báo giá, snapshot đầu vào | UC-01 |
| `parcels` | Thông tin vật lý một kiện | UC-01, UC-02 |
| `parcel_items` | Danh sách sản phẩm và số lượng | UC-01, UC-02 |
| `carriers` | Danh mục GHN/GHTK | UC-01..04 |
| `carrier_services` | Mã dịch vụ theo từng hãng | UC-01, UC-02 |
| `quote_attempts` | Ghi nhận gọi báo giá kể cả lỗi | UC-01 |
| `rates` | Phương án cước/ETA có thể so sánh | UC-01, UC-02 |
| `shipment_statuses` | Trạng thái vận chuyển chuẩn VietGo | UC-03 |
| `shipments` | Vận đơn, phí, COD, trạng thái hiện tại | UC-02..04 |
| `booking_attempts` | Lịch sử gọi API tạo đơn, timeout, unknown | UC-02 |
| `tracking_events` | Timeline tracking đầy đủ | UC-03 |

## 2. Từ điển từng bảng / cột

### 2.1. `users`

**Mục đích:** Đăng ký/đăng nhập, sở hữu dữ liệu. **Sử dụng:** Account; UC-01..04.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `user_id` | `bigint` | PK | Không | `—` | tự tăng | Mã định danh tài khoản VietGo | — |
| `full_name` | `varchar(100)` | — | Không | `—` | — | Tên người sử dụng | — |
| `email` | `varchar(255)` | — | Không | `—` | UNIQUE | Email đăng nhập (không trùng) | — |
| `password_hash` | `varchar(255)` | — | Không | `—` | — | Mật khẩu đã băm an toàn | — |
| `phone` | `varchar(20)` | — | Có | `—` | — | Số điện thoại của tài khoản | — |
| `created_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm tạo tài khoản | — |

### 2.2. `rate_requests`

**Mục đích:** Yêu cầu báo giá, snapshot đầu vào. **Sử dụng:** UC-01.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `rate_request_id` | `bigint` | PK | Không | `—` | tự tăng | Mã yêu cầu báo giá | — |
| `user_id` | `bigint` | FK → `users.user_id` | Không | `—` | — | Tài khoản sở hữu yêu cầu | — |
| `sender_name` | `varchar(100)` | — | Không | `—` | — | Tên người gửi | — |
| `sender_phone` | `varchar(20)` | — | Không | `—` | — | Số điện thoại người gửi | — |
| `sender_address` | `text` | — | Không | `—` | — | Địa chỉ lấy hàng chi tiết | — |
| `sender_province_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã tỉnh/thành gửi đã chuẩn hóa | — |
| `sender_ward_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã phường/xã gửi đã chuẩn hóa | — |
| `receiver_name` | `varchar(100)` | — | Không | `—` | — | Tên người nhận | — |
| `receiver_phone` | `varchar(20)` | — | Không | `—` | — | Số điện thoại người nhận | — |
| `receiver_address` | `text` | — | Không | `—` | — | Địa chỉ giao hàng chi tiết | — |
| `receiver_province_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã tỉnh/thành nhận đã chuẩn hóa | — |
| `receiver_ward_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã phường/xã nhận đã chuẩn hóa | — |
| `payer` | `varchar(20)` | — | Không | `'SHOP'` | SHOP / RECEIVER | Bên thanh toán cước: SHOP hoặc RECEIVER | — |
| `cod_amount` | `bigint` | — | Không | `0` | ≥ 0 | Số tiền yêu cầu thu hộ (0 nếu không COD) | VND |
| `created_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm gửi yêu cầu báo giá | — |

### 2.3. `parcels`

**Mục đích:** Thông tin vật lý một kiện. **Sử dụng:** UC-01, UC-02.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `parcel_id` | `bigint` | PK | Không | `—` | tự tăng | Mã kiện hàng | — |
| `rate_request_id` | `bigint` | FK → `rate_requests.rate_request_id` | Không | `—` | UNIQUE | Yêu cầu báo giá sở hữu kiện hàng | — |
| `item_description` | `text` | — | Có | `—` | — | Mô tả tổng quát kiện hàng (khác chi tiết mặt hàng) | — |
| `weight_grams` | `int` | — | Không | `—` | > 0 | Khối lượng kiện dùng để báo giá | gram |
| `length_cm` | `decimal(8,2)` | — | Có | `—` | > 0 nếu có | Chiều dài kiện | cm |
| `width_cm` | `decimal(8,2)` | — | Có | `—` | > 0 nếu có | Chiều rộng kiện | cm |
| `height_cm` | `decimal(8,2)` | — | Có | `—` | > 0 nếu có | Chiều cao kiện | cm |
| `declared_value` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Giá trị khai báo/bảo hiểm của kiện | VND |

### 2.4. `parcel_items`

**Mục đích:** Danh sách sản phẩm và số lượng. **Sử dụng:** UC-01, UC-02.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `parcel_item_id` | `bigint` | PK | Không | `—` | tự tăng | Mã dòng mặt hàng | — |
| `parcel_id` | `bigint` | FK → `parcels.parcel_id` | Không | `—` | một parcel có ≥ 1 item theo nghiệp vụ | Kiện hàng chứa mặt hàng | — |
| `item_name` | `varchar(255)` | — | Không | `—` | — | Tên sản phẩm/loại hàng | — |
| `quantity` | `int` | — | Không | `—` | > 0 | Số đơn vị sản phẩm | sản phẩm |
| `item_price` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Đơn giá sản phẩm nếu có | VND |
| `item_code` | `varchar(100)` | — | Có | `—` | — | Mã SKU của shop nếu có | — |

### 2.5. `carriers`

**Mục đích:** Danh mục GHN/GHTK. **Sử dụng:** UC-01..04.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `carrier_id` | `bigint` | PK | Không | `—` | tự tăng | Mã định danh hãng trong VietGo | — |
| `carrier_code` | `varchar(20)` | — | Không | `—` | UNIQUE; P0 = GHN/GHTK | Mã hãng dùng khi tích hợp | — |
| `carrier_name` | `varchar(100)` | — | Không | `—` | — | Tên hãng vận chuyển | — |
| `is_active` | `boolean` | — | Không | `true` | — | Đánh dấu hãng đang khả dụng | — |

### 2.6. `carrier_services`

**Mục đích:** Mã dịch vụ theo từng hãng. **Sử dụng:** UC-01, UC-02.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `service_id` | `bigint` | PK | Không | `—` | tự tăng | Mã định danh dịch vụ | — |
| `carrier_id` | `bigint` | FK → `carriers.carrier_id` | Không | `—` | — | Hãng cung cấp dịch vụ | — |
| `service_code` | `varchar(60)` | — | Không | `—` | — | Mã dịch vụ gốc của từng hãng | — |
| `service_name` | `varchar(100)` | — | Không | `—` | — | Tên dịch vụ hiển thị cho khách | — |
| `is_active` | `boolean` | — | Không | `true` | — | Dịch vụ có đang được sử dụng | — |

### 2.7. `quote_attempts`

**Mục đích:** Ghi nhận gọi báo giá kể cả lỗi. **Sử dụng:** UC-01.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `quote_attempt_id` | `bigint` | PK | Không | `—` | tự tăng | Mã lần gọi API báo giá | — |
| `rate_request_id` | `bigint` | FK → `rate_requests.rate_request_id` | Không | `—` | — | Yêu cầu báo giá của lần gọi | — |
| `carrier_id` | `bigint` | FK → `carriers.carrier_id` | Không | `—` | — | Hãng được gọi API | — |
| `request_status` | `varchar(30)` | — | Không | `—` | PENDING / SUCCESS / NO_SERVICE / FAILED / TIMEOUT / INVALID_RESPONSE | Trạng thái xử lý gọi API, kể cả lỗi và timeout | — |
| `error_code` | `varchar(60)` | — | Có | `—` | — | Mã lỗi trả về/hệ thống | — |
| `error_message` | `text` | — | Có | `—` | — | Thông điệp lỗi hoặc lý do thất bại | — |
| `is_mock` | `boolean` | — | Không | `false` | — | Đánh dấu phản hồi mô phỏng phục vụ demo | — |
| `requested_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm bắt đầu gọi API | — |
| `completed_at` | `timestamp` | — | Có | `—` | — | Thời điểm kết thúc nếu xác định được | — |

### 2.8. `rates`

**Mục đích:** Phương án cước/ETA có thể so sánh. **Sử dụng:** UC-01, UC-02.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `rate_id` | `bigint` | PK | Không | `—` | tự tăng | Mã phương án báo giá | — |
| `quote_attempt_id` | `bigint` | FK → `quote_attempts.quote_attempt_id` | Không | `—` | — | Lần gọi API sinh ra phương án | — |
| `carrier_id` | `bigint` | FK → `quote_attempts.carrier_id`, `carrier_services.carrier_id` | Không | `—` | — | Hãng vận chuyển của phương án; bị ràng buộc trùng hãng gọi API và dịch vụ | — |
| `service_id` | `bigint` | FK → `carrier_services.service_id` | Không | `—` | phải cùng carrier với lần gọi báo giá | Dịch vụ vận chuyển của phương án | — |
| `carrier_rate_id` | `varchar(100)` | — | Có | `—` | — | Mã báo giá gốc của hãng nếu có | — |
| `shipping_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Phí vận chuyển cơ bản | VND |
| `surcharge_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Phụ phí nếu được hãng trả về | VND |
| `service_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Phí dịch vụ nếu được hãng trả về | VND |
| `total_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Tổng cước báo giá theo hãng | VND |
| `fee_data_status` | `varchar(30)` | — | Không | `'UNAVAILABLE'` | COMPLETE / PARTIAL / UNAVAILABLE | Mức độ đầy đủ của dữ liệu cước | — |
| `eta_min_days` | `int` | — | Có | `—` | ≥ 0 | ETA nhỏ nhất theo ngày | ngày |
| `eta_max_days` | `int` | — | Có | `—` | ≥ 0 | ETA lớn nhất theo ngày | ngày |
| `eta_expected_at` | `timestamp` | — | Có | `—` | — | Thời điểm giao hàng dự kiến nếu hãng cung cấp | — |
| `eta_source` | `varchar(60)` | — | Có | `—` | — | Nguồn tính ETA hoặc tên API | — |
| `eta_raw` | `text` | — | Có | `—` | — | ETA gốc, giữ lại nếu không phân tích được | — |
| `is_available` | `boolean` | — | Không | `false` | — | Phương án có thể chọn để tạo vận đơn hay không | — |
| `quoted_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm lưu phương án báo giá | — |
| `expires_at` | `timestamp` | — | Có | `—` | — | Thời điểm hết hiệu lực phương án nếu có | — |

### 2.9. `shipment_statuses`

**Mục đích:** Trạng thái vận chuyển chuẩn VietGo. **Sử dụng:** UC-03.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `status_code` | `varchar(30)` | PK | Không | `—` | — | Mã trạng thái chuẩn VietGo | — |
| `status_name` | `varchar(100)` | — | Không | `—` | — | Nhãn hiển thị trạng thái chuẩn | — |
| `is_final` | `boolean` | — | Không | `false` | — | Đây có phải trạng thái cuối trong vòng đời vận đơn | — |

### 2.10. `shipments`

**Mục đích:** Vận đơn, phí, COD, trạng thái hiện tại. **Sử dụng:** UC-02..04.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `shipment_id` | `bigint` | PK | Không | `—` | tự tăng | Mã vận đơn nội bộ (PK) | — |
| `booking_key` | `varchar(100)` | — | Không | `—` | UNIQUE; định danh ổn định, chống tạo lặp | Khóa idempotency duy nhất cho một yêu cầu booking logic | — |
| `user_id` | `bigint` | FK → `users.user_id` | Không | `—` | phải khớp chủ rate request | Chủ sở hữu vận đơn (phải trùng chủ yêu cầu báo giá) | — |
| `rate_id` | `bigint` | FK → `rates.rate_id` | Không | `—` | UNIQUE; mỗi rate tối đa 1 shipment | Báo giá đã được chọn để tạo vận đơn | — |
| `carrier_id` | `bigint` | FK → `rates.carrier_id` | Không | `—` | phải khớp carrier của rate | Hãng được chọn, bị ràng buộc trùng hãng của báo giá | — |
| `vietgo_tracking_code` | `varchar(100)` | — | Không | `—` | UNIQUE | Mã vận đơn do VietGo tạo | — |
| `carrier_tracking_code` | `varchar(100)` | — | Có | `—` | — | Mã vận đơn do hãng cấp khi xác nhận | — |
| `booking_status` | `varchar(30)` | — | Không | `'PENDING'` | PENDING / SUCCESS / FAILED / UNKNOWN | Trạng thái giao dịch tạo đơn với API | — |
| `shipment_status` | `varchar(30)` | FK → `shipment_statuses.status_code` | Có | `—` | — | Trạng thái vận chuyển hiện tại được chuẩn hóa | — |
| `sender_name` | `varchar(100)` | — | Không | `—` | — | Tên người gửi tại thời điểm đặt đơn | — |
| `sender_phone` | `varchar(20)` | — | Không | `—` | — | Số điện thoại gửi tại thời điểm đặt đơn | — |
| `sender_address` | `text` | — | Không | `—` | — | Địa chỉ gửi tại thời điểm đặt đơn | — |
| `sender_province_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã tỉnh/thành gửi khi đặt đơn | — |
| `sender_ward_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã phường/xã gửi khi đặt đơn | — |
| `receiver_name` | `varchar(100)` | — | Không | `—` | — | Tên người nhận tại thời điểm đặt đơn | — |
| `receiver_phone` | `varchar(20)` | — | Không | `—` | — | Số điện thoại nhận tại thời điểm đặt đơn | — |
| `receiver_address` | `text` | — | Không | `—` | — | Địa chỉ nhận tại thời điểm đặt đơn | — |
| `receiver_province_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã tỉnh/thành nhận khi đặt đơn | — |
| `receiver_ward_code` | `varchar(40)` | — | Không | `—` | đúng danh mục địa giới VietGo | Mã phường/xã nhận khi đặt đơn | — |
| `payer` | `varchar(20)` | — | Không | `—` | SHOP / RECEIVER | Bên chịu cước đã được khách xác nhận | — |
| `parcel_snapshot` | `jsonb` | — | Không | `—` | — | Bản chụp kiện hàng và các mặt hàng lúc đặt đơn | — |
| `cod_amount` | `bigint` | — | Không | `0` | ≥ 0 | Số tiền COD đã xác nhận trên đơn | VND |
| `cod_data_status` | `varchar(30)` | — | Không | `'NOT_APPLICABLE'` | NOT_APPLICABLE / PENDING / AVAILABLE / UNSUPPORTED / ERROR | Tình trạng có dữ liệu COD hay không | — |
| `cod_status` | `varchar(40)` | — | Có | `—` | — | Trạng thái nghiệp vụ thu hộ đã chuẩn hóa | — |
| `cod_carrier_status` | `varchar(100)` | — | Có | `—` | — | Trạng thái COD nguyên gốc của hãng | — |
| `cod_source` | `varchar(50)` | — | Có | `—` | — | Nguồn trả dữ liệu COD | — |
| `cod_updated_at` | `timestamp` | — | Có | `—` | — | Lần cuối nhận dữ liệu COD | — |
| `confirmed_shipping_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Cước vận chuyển hãng xác nhận sau tạo đơn | VND |
| `confirmed_total_fee` | `bigint` | — | Có | `—` | ≥ 0 nếu có | Tổng phí hãng xác nhận sau tạo đơn | VND |
| `tracking_sync_status` | `varchar(30)` | — | Có | `—` | SUCCESS / NO_UPDATE / ERROR | Kết quả đồng bộ tracking lần gần nhất | — |
| `tracking_last_checked_at` | `timestamp` | — | Có | `—` | — | Lần cuối kiểm tra tracking từ hãng | — |
| `tracking_last_error` | `text` | — | Có | `—` | — | Thông báo lỗi đồng bộ tracking gần nhất | — |
| `status_updated_at` | `timestamp` | — | Có | `—` | — | Lần cuối trạng thái vận chuyển thay đổi | — |
| `created_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm tạo đối tượng vận đơn | — |
| `updated_at` | `timestamp` | — | Có | `—` | — | Thời điểm sửa đổi gần nhất | — |

### 2.11. `booking_attempts`

**Mục đích:** Lịch sử gọi API tạo đơn, timeout, unknown. **Sử dụng:** UC-02.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `attempt_id` | `bigint` | PK | Không | `—` | tự tăng | Mã lần thử gọi API tạo vận đơn | — |
| `shipment_id` | `bigint` | FK → `shipments.shipment_id` | Không | `—` | — | Vận đơn VietGo của lần gọi | — |
| `attempt_no` | `int` | — | Không | `—` | > 0 | Thứ tự lần thử trong cùng một booking | lần |
| `attempt_status` | `varchar(30)` | — | Không | `—` | PENDING / SUCCESS / FAILED / UNKNOWN | Trạng thái thực hiện của lần thử | — |
| `carrier_reference` | `varchar(100)` | — | Có | `—` | — | Mã tham chiếu từ carrier nếu có | — |
| `error_code` | `varchar(60)` | — | Có | `—` | — | Mã lỗi gọi API tạo đơn | — |
| `error_message` | `text` | — | Có | `—` | — | Mô tả lỗi hoặc timeout | — |
| `attempted_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm bắt đầu lần gọi | — |
| `completed_at` | `timestamp` | — | Có | `—` | — | Thời điểm kết thúc lần gọi nếu biết | — |

### 2.12. `tracking_events`

**Mục đích:** Timeline tracking đầy đủ. **Sử dụng:** UC-03.

| Cột | Kiểu dữ liệu | PK/FK | Nullable | Default | Ràng buộc | Ý nghĩa nghiệp vụ | Đơn vị |
|---|---|---|---|---|---|---|---|
| `tracking_event_id` | `bigint` | PK | Không | `—` | tự tăng | Mã sự kiện tracking nội bộ | — |
| `shipment_id` | `bigint` | FK → `shipments.shipment_id` | Không | `—` | — | Vận đơn liên quan | — |
| `carrier_event_id` | `varchar(100)` | — | Có | `—` | — | ID sự kiện gốc từ hãng nếu có | — |
| `event_fingerprint` | `varchar(128)` | — | Không | `—` | định danh ổn định, chống tạo lặp | Dấu vân tay chống ghi lặp cùng sự kiện | — |
| `carrier_status` | `varchar(100)` | — | Không | `—` | — | Trạng thái gốc từ GHN/GHTK | — |
| `normalized_status` | `varchar(30)` | FK → `shipment_statuses.status_code` | Có | `—` | — | Trạng thái chuẩn VietGo; có thể chưa map | — |
| `description` | `text` | — | Có | `—` | — | Diễn giải sự kiện từ hãng | — |
| `event_time` | `timestamp` | — | Có | `—` | — | Thời điểm sự kiện tại hãng (có thể thiếu) | — |
| `received_at` | `timestamp` | — | Không | ``now()`` | — | Thời điểm VietGo tiếp nhận sự kiện | — |

## 3. Khóa, quan hệ và chuẩn hóa

**Các quan hệ chính:** `users (1) → (N) rate_requests`; `rate_requests (1) → (0..1) parcels` ở mức lược đồ (nghiệp vụ hợp lệ cần đúng một kiện); `parcels (1) → (N) parcel_items`; `rate_requests (1) → (N) quote_attempts`; `quote_attempts (1) → (N) rates`; `rates (1) → (0..1) shipments`; `shipments (1) → (N) booking_attempts / tracking_events`. `carriers`, `carrier_services`, `shipment_statuses` là bảng danh mục.

**Khóa và tính đúng hãng:** `rates.carrier_id` được lặp có chủ đích và được khóa kép kiểm tra đồng thời với `quote_attempts` và `carrier_services`. `shipments.carrier_id` lặp có chủ đích để ràng buộc đúng hãng của `rates` và đặt UNIQUE theo `(carrier_id, carrier_tracking_code)`. Đây là hai thuộc tính suy diễn có ràng buộc, không được phép cập nhật tùy ý.

**1NF:** Mỗi ô là giá trị nguyên tử; danh sách mặt hàng tách vào `parcel_items`. **2NF:** Từng bảng thực thể sử dụng PK định danh, các cột nghiệp vụ phụ thuộc vào PK; các unique/key nhiều cột chỉ phục vụ nhất quán. **3NF:** Thông tin carrier và dịch vụ không lặp tên hãng ở `rates`/`shipments`; trạng thái chuẩn dùng danh mục `shipment_statuses`. Ngoại lệ lưu có chủ đích là thông tin chụp tại thời điểm đặt đơn trong `shipments` (không thay đổi nếu yêu cầu báo giá thay đổi) và hai `carrier_id` lặp bị ràng buộc.

**Địa chỉ:** P0 dùng cột địa chỉ + mã tỉnh/phường trên request, và snapshot các cột này trên shipment. Không tự tạo bảng `addresses` khi chưa có UC sổ địa chỉ. **Phí:** tổng tiền và từng thành phần riêng; NULL = chưa được hãng cung cấp, 0 = hãng xác nhận miễn phí. **COD:** `cod_amount` khác với `cod_status`; `cod_data_status` thể hiện mức độ có/không có dữ liệu từ hãng.

## 4. Quy tắc liên bảng cần lưu ý

1. **Chủ sở hữu:** `shipments.user_id` phải bằng `rate_requests.user_id` của rate đã chọn; `constraints-postgresql.sql` có trigger minh họa. Không chỉ kiểm tra ở giao diện.
2. **Đúng hãng:** quote_attempt, carrier_service, rate và shipment phải cùng carrier thông qua các FK kép.
3. **Không tạo trùng:** `booking_key` giữ nguyên qua cùng một yêu cầu logic; khi trạng thái booking UNKNOWN do timeout, phải tra cứu/đối chiếu trước khi thử lại, không tự tạo booking mới.
4. **Báo giá thiếu dữ liệu:** vẫn có thể lưu `rates` với phí/ETA NULL, nhưng chỉ hiển thị và cho phép chọn nếu dữ liệu đủ theo quy tắc nghiệp vụ. `quote_attempts` lưu các lần NO_SERVICE/FAILED/TIMEOUT ngay cả khi không có rate.
5. **Tracking chưa map:** vẫn lưu `carrier_status` gốc và `normalized_status = NULL`; hiển thị trạng thái gốc/nhãn chưa xác định, không làm mất lịch sử. `event_time` có thể NULL, sắp timeline theo `COALESCE(event_time, received_at)`.
6. **COD:** `cod_amount = 0` nên đi cùng `NOT_APPLICABLE`; `cod_amount > 0` ban đầu PENDING, chỉ AVAILABLE nếu đã nhận trạng thái. Không coi `cod_status IS NULL` đồng nghĩa đã thu hộ hoặc chưa thu hộ.
7. **Tính bất biến:** các trường địa chỉ, payer, parcel_snapshot được chụp tại lúc booking, không ghi đè theo request gốc về sau.
8. **Ít nhất một mặt hàng:** bắt buộc nghiệp vụ có ≥ 1 `parcel_items` trước khi gọi carrier; giới hạn này không bảo đảm bằng một FK đơn thuần.

## 5. Truy vấn thực tế phục vụ nghiệp vụ (chỉ dẫn)

| Nghiệp vụ | Bảng truy vấn | Khóa / index cần dùng |
|---|---|---|
| UC-01 so sánh cước | `rate_requests → quote_attempts → rates → carrier_services → carriers` | `rate_request_id; quote_attempt_id; service_id` |
| UC-01 phân tích lỗi | `quote_attempts + carriers` | `(rate_request_id, carrier_id, requested_at)` |
| UC-02 tạo và xem vận đơn | `rates → shipments; shipments → booking_attempts` | `shipments.rate_id UNIQUE; booking_key UNIQUE` |
| Danh sách đơn của người dùng | `shipments` | `(user_id, created_at)` |
| UC-03 timeline | `tracking_events + shipment_statuses` | `(shipment_id, event_time); event_fingerprint` |
| UC-04 xem COD | `shipments` | `shipment_id; cod_data_status` |

## 6. Kiểm thử nghiệm thu dữ liệu

- [ ] UC-01: thử riêng tình huống cả GHN/GHTK thành công (2 quote_attempts), và tình huống một trong hai hãng lỗi/timeout (vẫn có quote_attempt nhưng có thể không có rates). Tài liệu ghi 2 hãng thành công + 1 hãng lỗi cần đối chiếu do P0 chỉ có 2 hãng.
- [ ] UC-01: chưa có ETA → eta_raw hoặc eta_expected_at/min/max NULL hợp lệ, UI phân biệt chưa có ETA.
- [ ] UC-02: thành công → shipment.booking_status SUCCESS và có carrier_tracking_code; lỗi từ chối → FAILED; timeout chưa rõ → UNKNOWN và lưu booking_attempts.
- [ ] UC-02: thử lại với cùng booking_key không tạo một shipment thứ hai; shipment.rate_id UNIQUE.
- [ ] UC-03: event không nhận diện → carrier_status gốc vẫn được lưu; normalized_status NULL; event trùng fingerprint bị chặn.
- [ ] UC-03: carrier lỗi → tracking_sync_status ERROR, timeline cũ còn nguyên.
- [ ] UC-04: có COD → cod_data_status AVAILABLE; chưa cập nhật → PENDING; hãng không hỗ trợ → UNSUPPORTED; không COD → NOT_APPLICABLE.
- [ ] Giá trị âm, weight_grams = 0, quantity = 0, payer ngoài danh mục hoặc tham chiếu sai carrier phải bị chặn.

## 7. Phần chưa được phê duyệt / ngoài P0

Ba bảng `shipment_templates`, `repeat_plans`, `repeat_occurrences` của bản cũ được **giữ nguyên hướng nghiệp vụ nhưng chuyển ra** `schema-p1-extension.dbml`. Chỉ nhập vào ERD chính thức khi D01 chính thức phê duyệt chức năng tạo đơn định kỳ. Chức năng Admin P1/đối soát COD chuyên sâu cũng không tự động bổ sung.

## 8. Nguồn và giới hạn

- Nguồn nội bộ: Google Docs *THIẾT KẾ HỆ THỐNG THƯƠNG MẠI ĐIỆN TỬ* → tab *ĐỒ ÁN VIETGO*, mục D03 và UC-01..04.
- API Mapping nội bộ: bảng *API Mapping GHN - GHTK - VietGo*, tab `Mapping` (địa chỉ, mặt hàng, COD, bên trả phí, tracking, ETA).
- Các ENUM, quy tắc snapshot, idempotency, FK kép, snapshot một kiện / request, bộ index và chính sách NULL là **đề xuất thiết kế** cần xác nhận với SRS / D06 / adapter GHN-GHTK trước khi code production.