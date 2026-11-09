# 4.2. Thiết kế các bảng dữ liệu hệ thống VietGo

Cơ sở dữ liệu VietGo gồm 18 bảng: giữ nguyên 15 bảng gốc (12 bảng P0 và 3 bảng mở rộng RepeatBox thuộc P1), đồng thời bổ sung 3 bảng Complaint cho UC-06. Bảng `users` có thêm trường `role` để hỗ trợ quản trị cơ bản ở P1.

**Bảng 4.1. Danh sách thực thể và các thuộc tính trong cơ sở dữ liệu VietGo**

| **Table** | **Fields** | **Description** |
|---|---|---|
| **users** | user_id (PK), full_name, email, password_hash, phone, role, created_at | Quản lý tài khoản, thông tin đăng nhập và vai trò USER/ADMIN (quản trị cơ bản P1). |
| **rate_requests** | rate_request_id (PK), user_id (FK), sender_name, sender_phone, sender_address, sender_province_code, sender_ward_code, receiver_name, receiver_phone, receiver_address, receiver_province_code, receiver_ward_code, payer, cod_amount, created_at | Lưu dữ liệu đầu vào báo giá: người gửi/nhận, địa chỉ, phí do ai trả và số tiền COD (UC-01). |
| **parcels** | parcel_id (PK), rate_request_id (FK), item_description, weight_grams, length_cm, width_cm, height_cm, declared_value | Lưu thông tin kiện hàng, bao gồm trọng lượng (gram), kích thước (cm) và giá trị khai báo (VND). Mỗi yêu cầu báo giá liên kết tối đa một kiện hàng theo thiết kế hiện tại. |
| **parcel_items** | parcel_item_id (PK), parcel_id (FK), item_name, quantity, item_price, item_code | Chi tiết các mặt hàng thuộc kiện: tên, số lượng, giá và mã sản phẩm để đáp ứng yêu cầu của hãng. |
| **carriers** | carrier_id (PK), carrier_code, carrier_name, is_active | Danh mục hãng vận chuyển GHN và GHTK; trạng thái khả dụng của từng hãng. |
| **carrier_services** | service_id (PK), carrier_id (FK), service_code, service_name, is_active | Danh mục dịch vụ theo từng carrier, tránh nhầm dịch vụ giữa các hãng. |
| **quote_attempts** | quote_attempt_id (PK), rate_request_id (FK), carrier_id (FK), request_status, error_code, error_message, is_mock, requested_at, completed_at | Ghi nhận từng lần gọi API báo giá và lỗi/timeout, cho phép hai hãng phản hồi độc lập. |
| **rates** | rate_id (PK), quote_attempt_id (FK), carrier_id (FK), service_id (FK), carrier_rate_id, shipping_fee, surcharge_fee, service_fee, total_fee, fee_data_status, eta_min_days, eta_max_days, eta_expected_at, eta_source, eta_raw, is_available, quoted_at, expires_at | Lưu kết quả báo giá gồm phí vận chuyển, ETA, dịch vụ, tính khả dụng và thời hạn hiệu lực. Khóa ngoại ghép bảo đảm kết quả báo giá và dịch vụ thuộc cùng một hãng vận chuyển. |
| **shipment_statuses** | status_code (PK), status_name, is_final | Danh mục trạng thái vận chuyển chuẩn hóa dùng chung cho tracking và vận đơn. |
| **shipments** | shipment_id (PK), booking_key, user_id (FK), rate_id (FK), carrier_id (FK), vietgo_tracking_code, carrier_tracking_code, booking_status, shipment_status (FK), sender_name, sender_phone, sender_address, sender_province_code, sender_ward_code, receiver_name, receiver_phone, receiver_address, receiver_province_code, receiver_ward_code, payer, parcel_snapshot, cod_amount, cod_data_status, cod_status, cod_carrier_status, cod_source, cod_updated_at, confirmed_shipping_fee, confirmed_total_fee, tracking_sync_status, tracking_last_checked_at, tracking_last_error, status_updated_at, created_at, updated_at | Lưu thông tin vận đơn VietGo, kết quả đặt hãng, dữ liệu người gửi/nhận, kiện hàng, COD, phí và trạng thái vận chuyển. Khóa ngoại ghép bảo đảm hãng vận chuyển của đơn phù hợp với báo giá đã chọn. |
| **booking_attempts** | attempt_id (PK), shipment_id (FK), attempt_no, attempt_status, carrier_reference, error_code, error_message, attempted_at, completed_at | Lưu lịch sử các lần gọi API tạo vận đơn và kết quả PENDING/SUCCESS/FAILED/UNKNOWN; hỗ trợ kiểm soát việc gọi lại API và phát hiện nguy cơ tạo đơn trùng. |
| **tracking_events** | tracking_event_id (PK), shipment_id (FK), carrier_event_id, event_fingerprint, carrier_status, normalized_status (FK), description, event_time, received_at | Lịch sử trạng thái gốc/chuẩn hóa và thời gian nhận cập nhật vận chuyển. |
| **shipment_templates** | template_id (PK), user_id (FK), template_name, sender_name, sender_phone, sender_address, sender_province_code, sender_ward_code, receiver_name, receiver_phone, receiver_address, receiver_province_code, receiver_ward_code, item_description, items_snapshot, weight_grams, length_cm, width_cm, height_cm, declared_value, payer, cod_amount, is_active, created_at, updated_at | [P1 – RepeatBox] Mẫu đơn dùng lại gồm gửi/nhận, hàng hóa và COD, thuộc một chủ sở hữu. |
| **repeat_plans** | repeat_plan_id (PK), template_id (FK), user_id (FK), plan_name, frequency, day_of_week, day_of_month, start_date, end_date, next_run_date, plan_status, created_at, updated_at | [P1 – RepeatBox] Lưu kế hoạch gửi hàng định kỳ theo tuần/tháng, ngày chạy tiếp theo và trạng thái ACTIVE/PAUSED/ENDED. Khóa ngoại ghép bảo đảm kế hoạch sử dụng mẫu đơn thuộc đúng người dùng sở hữu. |
| **repeat_occurrences** | occurrence_id (PK), repeat_plan_id (FK), scheduled_for, occurrence_status, draft_snapshot, rate_request_id (FK), shipment_id (FK), created_at, updated_at | [P1 – RepeatBox] Mỗi kỳ gửi có bản nháp nội bộ; UNIQUE(repeat_plan_id,scheduled_for) ngăn Scheduler tạo trùng; chỉ nối shipment sau SUCCESS. |
| **complaint_tickets** | complaint_id (PK), shipment_id (FK), created_by_user_id (FK), assigned_to_user_id (FK), complaint_code, category, subject, description, status, forwarding_method, forwarding_status, carrier_ticket_id, carrier_reference, idempotency_key, last_forwarded_at, resolved_at, closed_at, created_at, updated_at | [UC-06 – Complaint] Ticket khiếu nại gắn với vận đơn; lưu người tạo, CSKH phụ trách, trạng thái xử lý/chuyển tiếp, mã ticket VietGo và mã ticket hãng nếu có. GHN chuyển tiếp qua API, GHTK qua CSKH thủ công; `carrier_ticket_id` cho phép NULL. |
| **complaint_evidence** | evidence_id (PK), complaint_id (FK), uploaded_by_user_id (FK), file_name, mime_type, file_size_bytes, storage_key, checksum_sha256, evidence_note, uploaded_at | [UC-06 – Complaint] Thông tin chứng từ/ảnh đính kèm; lưu khóa tệp riêng tư, định dạng, kích thước, người tải lên và thời điểm. |
| **complaint_history** | history_id (PK), complaint_id (FK), actor_user_id (FK), actor_type, event_type, from_status, to_status, message, event_metadata, occurred_at | [UC-06 – Complaint] Nhật ký tiếp nhận, bổ sung chứng từ, gửi carrier, timeout, cập nhật trạng thái, xử lý và đóng ticket; lưu tác nhân cùng thời gian phát sinh. |

**Chú giải:** PK = khóa chính (Primary Key); FK = khóa ngoại (Foreign Key). Các khóa ngoại kép và ràng buộc UNIQUE được định nghĩa cụ thể trong `schema-vietgo-18-bang.dbml`. Phạm vi P1 là phần mở rộng, không thuộc chức năng nghiệm thu P0.

### 4.2.3. Ánh xạ quan hệ giữa mô hình ERD và lược đồ CSDL quan hệ

Mô hình ERD của VietGo được chuyển thành các bảng quan hệ theo nguyên tắc: mỗi thực thể tương ứng một bảng; mỗi thuộc tính tương ứng một cột; quan hệ giữa các thực thể được hiện thực bằng khóa ngoại (FK), khóa duy nhất (UNIQUE) và các ràng buộc nghiệp vụ. Bảng dưới đây liệt kê **27 quan hệ** được khai báo trong `schema-vietgo-18-bang.dbml`; hướng tham chiếu là từ **bảng con (FK)** đến **bảng cha (PK/UNIQUE)**.

**Bảng 4.2. Ánh xạ quan hệ ERD – CSDL quan hệ**

| STT | Bảng con / FK | Bảng cha / khóa được tham chiếu | Quan hệ và ý nghĩa |
|---|---|---|---|
| 1 | `rate_requests.user_id` | `users.user_id` | 1:N – Một tài khoản tạo nhiều yêu cầu báo giá. |
| 2 | `parcels.rate_request_id` | `rate_requests.rate_request_id` | 1:0..1 – Một yêu cầu báo giá có tối đa một kiện do `rate_request_id` UNIQUE. |
| 3 | `parcel_items.parcel_id` | `parcels.parcel_id` | 1:N – Một kiện chứa nhiều mặt hàng. |
| 4 | `carrier_services.carrier_id` | `carriers.carrier_id` | 1:N – Một hãng có nhiều dịch vụ. |
| 5 | `quote_attempts.rate_request_id` | `rate_requests.rate_request_id` | 1:N – Một yêu cầu báo giá được gọi API nhiều lần. |
| 6 | `quote_attempts.carrier_id` | `carriers.carrier_id` | 1:N – Một hãng nhận nhiều lần gọi báo giá. |
| 7 | `rates.(quote_attempt_id, carrier_id)` | `quote_attempts.(quote_attempt_id, carrier_id)` | FK kép – Báo giá phải thuộc đúng hãng được gọi API. |
| 8 | `rates.(service_id, carrier_id)` | `carrier_services.(service_id, carrier_id)` | FK kép – Dịch vụ được báo giá phải thuộc đúng hãng. |
| 9 | `shipments.(rate_id, carrier_id)` | `rates.(rate_id, carrier_id)` | FK kép – Vận đơn phải chọn đúng báo giá và hãng tương ứng. |
| 10 | `shipments.user_id` | `users.user_id` | 1:N – Một tài khoản có nhiều vận đơn. |
| 11 | `shipments.shipment_status` | `shipment_statuses.status_code` | N:1, có thể NULL – Vận đơn sử dụng trạng thái chuẩn hóa. |
| 12 | `booking_attempts.shipment_id` | `shipments.shipment_id` | 1:N – Một vận đơn có thể có nhiều lần thử booking. |
| 13 | `tracking_events.shipment_id` | `shipments.shipment_id` | 1:N – Một vận đơn có nhiều sự kiện tracking. |
| 14 | `tracking_events.normalized_status` | `shipment_statuses.status_code` | N:1, có thể NULL – Sự kiện được ánh xạ trạng thái nếu nhận diện được. |
| 15 | `shipment_templates.user_id` | `users.user_id` | 1:N, P1 – Một người dùng lưu nhiều mẫu gửi hàng. |
| 16 | `repeat_plans.user_id` | `users.user_id` | 1:N, P1 – Một người dùng tạo nhiều kế hoạch định kỳ. |
| 17 | `repeat_plans.(template_id, user_id)` | `shipment_templates.(template_id, user_id)` | FK kép, P1 – Kế hoạch chỉ được tham chiếu mẫu thuộc cùng chủ sở hữu. |
| 18 | `repeat_occurrences.repeat_plan_id` | `repeat_plans.repeat_plan_id` | 1:N, P1 – Mỗi kế hoạch sinh nhiều kỳ gửi. |
| 19 | `repeat_occurrences.rate_request_id` | `rate_requests.rate_request_id` | FK có thể NULL, P1 – Kỳ gửi liên kết yêu cầu báo giá mới khi phát sinh. |
| 20 | `repeat_occurrences.shipment_id` | `shipments.shipment_id` | FK có thể NULL, UNIQUE, P1 – Kỳ gửi chỉ liên kết vận đơn được xác nhận; một vận đơn thuộc tối đa một kỳ gửi. |
| 21 | `complaint_tickets.shipment_id` | `shipments.shipment_id` | 1:N – Một vận đơn có thể phát sinh nhiều ticket; chỉ chủ vận đơn được tạo khiếu nại (kiểm tra ở tầng nghiệp vụ). |
| 22 | `complaint_tickets.created_by_user_id` | `users.user_id` | 1:N – Một người dùng có thể tạo nhiều ticket khiếu nại. |
| 23 | `complaint_tickets.assigned_to_user_id` | `users.user_id` | 1:N, có thể NULL – Một nhân sự CSKH có thể phụ trách nhiều ticket; việc kiểm tra vai trò thuộc tầng ứng dụng. |
| 24 | `complaint_evidence.complaint_id` | `complaint_tickets.complaint_id` | 1:N – Một ticket có nhiều bằng chứng. |
| 25 | `complaint_evidence.uploaded_by_user_id` | `users.user_id` | 1:N – Ghi nhận người tải chứng từ lên. |
| 26 | `complaint_history.complaint_id` | `complaint_tickets.complaint_id` | 1:N – Một ticket có nhiều sự kiện lịch sử xử lý. |
| 27 | `complaint_history.actor_user_id` | `users.user_id` | 1:N, có thể NULL – Ghi nhận người thực hiện thay đổi; NULL đối với sự kiện hệ thống hoặc callback hợp lệ. |

**Giải thích và ràng buộc chính.** Khóa chính (PK) định danh duy nhất một bản ghi, còn khóa ngoại (FK) bảo đảm bản ghi liên kết tồn tại. Với các FK kép ở hàng 7, 8, 9 và 17, bảng cha phải có khóa duy nhất tương ứng; nhờ đó VietGo không thể ghép báo giá/dịch vụ khác hãng hoặc dùng mẫu RepeatBox của người dùng khác. Mối quan hệ `rate_requests` – `parcels` hiện giới hạn một kiện cho mỗi yêu cầu báo giá; đây là giả định cần nhóm xác nhận trước khi nghiệm thu.

**Kiểm soát phát sinh dữ liệu.** Để tránh tạo đơn trùng, `shipments.booking_key` là UNIQUE và `booking_attempts` có UNIQUE `(shipment_id, attempt_no)`. Mỗi vận đơn chỉ được có tối đa một lần gọi API chưa giải quyết tại một thời điểm; trạng thái UNKNOWN phải được đối chiếu với hãng trước khi gửi lại. Với RepeatBox P1, `repeat_occurrences` có UNIQUE `(repeat_plan_id, scheduled_for)` để Scheduler không tạo trùng kỳ gửi; Scheduler chỉ sinh bản nháp nội bộ, không tự động booking hãng.

**Đối chiếu ERD – DBML.** Hình ERD tổng thể phải thể hiện đủ 18 bảng và 27 mối quan hệ nói trên. Bản Mermaid được tạo từ `schema-vietgo-18-bang.dbml` và bao gồm đủ 18 bảng, giữ nguyên tên và thứ tự thuộc tính trong schema. Tuy nhiên, hình Mermaid/ảnh ERD không biểu diễn trọn vẹn định nghĩa FK kép, UNIQUE nhiều cột và các CHECK/trigger; `schema-vietgo-18-bang.dbml` cùng phần quy tắc CSDL là nguồn tham chiếu chính cho các ràng buộc này. Khi xuất hình nộp báo cáo, cần chọn nền sáng và màu đường nối tương phản để nhìn rõ các quan hệ.

### 4.2.4. Bổ sung ràng buộc nghiệp vụ Complaint (UC-06)

- `complaint_tickets.shipment_id` là FK bắt buộc; người tạo (`created_by_user_id`) phải là người sở hữu vận đơn (`shipments.user_id`). Đây là điều kiện kiểm tra liên bảng/tầng nghiệp vụ, không được đảm bảo chỉ bởi hai FK riêng lẻ.
- `complaint_tickets.carrier_ticket_id` có thể NULL. Chỉ lưu mã ticket hãng khi hãng thực sự xác nhận; không tự tạo mã API/ticket GHTK. `forwarding_method` phân biệt `GHN_API` và `GHTK_MANUAL`.
- `idempotency_key` và `complaint_code` UNIQUE để chống gửi trùng và định danh ticket VietGo. Trường `forwarding_status = UNKNOWN` không cho phép tự động gửi lại khi chưa đối chiếu.
- `complaint_evidence.storage_key` UNIQUE, lưu đường dẫn nội bộ thay vì URL công khai. `file_size_bytes` tính bằng byte và phải > 0; file cần được kiểm tra định dạng, dung lượng, phân quyền.
- `complaint_history` là nhật ký sự kiện; không nên sửa/xóa bản ghi đã phát sinh. `actor_user_id` có thể NULL nếu `actor_type` là `SYSTEM` hoặc `CARRIER`.
- Các giá trị trạng thái/loại khiếu nại và quyền CSKH nêu trong ghi chú DBML cần được kiểm tra bằng CHECK/logic ứng dụng khi triển khai PostgreSQL.
