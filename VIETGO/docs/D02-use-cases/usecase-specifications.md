# D02 — Use Case Specifications | VietGo

**Phạm vi:** MVP P0 · **Actor chính:** Người gửi hàng · **Hệ thống tích hợp:** GHN/GHTK  
**Sơ đồ đi kèm:** `usecase-overview.puml`  
**Lưu ý:** P1 (RepeatBox, hủy vận đơn, quản trị, webhook) được phân biệt trên sơ đồ tổng thể và không nằm trong bốn đặc tả bắt buộc.

## UC-01 — Báo giá và so sánh

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-01 · WF-01 · P0 |
| **Use Case Name** | Báo giá và so sánh cước GHN/GHTK |
| **Actors** | Chính: Người gửi hàng. Phụ: GHN API, GHTK API. |
| **Preconditions** | Mở được biểu mẫu báo giá; VietGo có cấu hình kết nối carrier; người gửi có thể cung cấp dữ liệu đơn hàng hợp lệ. Chưa cần có vận đơn. |
| **Trigger** | Người gửi chọn **Tra cứu cước** sau khi nhập dữ liệu. |
| **Postconditions** | Nếu thành công: có ít nhất một phương án hợp lệ để lựa chọn; chưa phát sinh vận đơn. Nếu thất bại: có thông báo phù hợp, không tạo vận đơn. |
| **FR/AC liên quan** | BR-01, BR-02, BR-03, BR-06, BR-08; UR-02–UR-05, UR-13; FR-02–FR-08, FR-17; AC-01–AC-04, AC-08. |

### Main Flow
1. Người gửi mở biểu mẫu tra cứu cước.
2. Nhập nơi gửi/nhận, liên hệ, thông tin kiện hàng, trọng lượng/kích thước (nếu cần), giá trị khai báo và COD (nếu có).
3. Người gửi chọn **Tra cứu cước**; VietGo kiểm tra các trường bắt buộc và dữ liệu hợp lệ.
4. VietGo chuẩn hóa dữ liệu, gửi yêu cầu báo giá độc lập đến GHN và GHTK.
5. Hai hãng trả báo giá hợp lệ; VietGo chuẩn hóa hãng, dịch vụ, cước, phụ phí, tổng phí, tính khả dụng và ETA khi được cung cấp.
6. VietGo hiển thị bảng so sánh bằng VNĐ, nêu rõ trường có dữ liệu hoặc chưa có dữ liệu.
7. Người gửi lựa chọn phương án phù hợp để tiếp tục UC-02.

### Alternate Flow
- **A1 — Chỉ một hãng thành công:** Hiển thị giá của hãng đó và thông báo hãng còn lại không khả dụng; vẫn cho chọn dịch vụ hợp lệ.
- **A2 — Thiếu ETA:** Hiển thị **Chưa có dữ liệu ETA**; không tự dự đoán thời gian giao.
- **A3 — Thiếu khoản phí:** Ghi **Chưa có dữ liệu**, không mặc định khoản thiếu bằng 0.
- **A4 — Dịch vụ không hỗ trợ tuyến/kiện hàng:** Không cho chọn dịch vụ đó; hiển thị những phương án còn hợp lệ.
- **A5 — Hai hãng phản hồi hợp lệ nhưng không có dịch vụ phù hợp:** Hiển thị **Không có báo giá phù hợp**, cho sửa đầu vào và thử lại; không coi là lỗi kỹ thuật.

### Exception Flow
- **E1 — Dữ liệu nhập sai/thiếu:** Chỉ rõ trường cần sửa; không gửi yêu cầu API.
- **E2 — Hai API đều lỗi hoặc timeout:** Thông báo chưa lấy được báo giá, ghi nhận lỗi và cho phép thử lại; không tự sinh giá giả.
- **E3 — Phản hồi sai định dạng:** Bỏ phương án không thể chuẩn hóa và ghi lỗi tích hợp.

**Business Rules / NFR:** Giá lấy từ carrier; timeout từng API tối đa 10 giây theo yêu cầu thiết kế; lỗi một hãng không làm hỏng kết quả hãng khác; dữ liệu mock phải ghi nhãn rõ. NFR-02/03/07/08/10/13/14.

---

## UC-02 — Xác nhận và tạo vận đơn

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-02 · WF-02 · P0 |
| **Use Case Name** | Xác nhận và tạo vận đơn |
| **Actors** | Chính: Người gửi hàng. Phụ: API hãng được chọn (GHN hoặc GHTK). |
| **Preconditions** | Người gửi đã đăng nhập, có phương án báo giá hợp lệ từ UC-01 và đã chọn được dịch vụ phù hợp trước khi xác nhận. |
| **Trigger** | Người gửi chọn **Xác nhận/Tạo vận đơn**. |
| **Postconditions** | Thành công: một vận đơn được lưu với mã VietGo và thông tin carrier được xác nhận. Thất bại/chưa rõ: không báo thành công giả; giao dịch được lưu vết để xác minh. |
| **FR/AC liên quan** | BR-03, BR-08; UR-06–UR-08, UR-13; FR-03, FR-09–FR-12, FR-17; AC-05, AC-09. |

### Main Flow
1. Người gửi chọn dịch vụ hợp lệ từ kết quả báo giá UC-01.
2. VietGo hiển thị màn hình xác nhận: người gửi, người nhận, kiện hàng, hãng, dịch vụ, cước/phụ phí và COD nếu có.
3. Người gửi kiểm tra và xác nhận tạo vận đơn.
4. VietGo kiểm tra dữ liệu, quyền tài khoản, hiệu lực báo giá; khởi tạo hoặc tái sử dụng `booking_key` duy nhất cho lần xác nhận này, lưu trạng thái `PENDING`, và khóa xử lý đồng thời trên cùng `booking_key` để chống tạo trùng.
5. Backend VietGo gửi yêu cầu tạo đơn đến carrier đã chọn, kèm mã tham chiếu/idempotency key nếu API hãng hỗ trợ; ghi nhận `booking_key`, carrier, request và từng `booking_attempt` để đối chiếu; không lộ khóa API trên trình duyệt.
6. Carrier xác nhận hợp lệ và trả mã vận đơn hãng cùng các thông tin liên quan.
7. Khi carrier xác nhận đã tạo, VietGo gắn mã carrier với `booking_key`, cập nhật `booking_status = SUCCESS` và lưu vận đơn, phí, COD, trạng thái có căn cứ; một `booking_key` chỉ được liên kết tối đa một vận đơn thành công.
8. VietGo thông báo tạo thành công và hiển thị chi tiết vận đơn.

### Alternate Flow
- **A1 — Sửa trước khi xác nhận:** Người gửi quay lại sửa; VietGo kiểm tra và báo giá lại nếu thay đổi ảnh hưởng đến phí/dịch vụ.
- **A2 — Giá/dịch vụ hết hiệu lực:** Yêu cầu tra cứu và chọn lại; không tạo theo báo giá cũ.
- **A3 — Thiếu thông tin tùy chọn trong phản hồi:** Chỉ hiển thị phần được xác nhận, trường thiếu ghi **Chưa có dữ liệu**.
- **A4 — Xác nhận nhiều lần:** Các thao tác lặp của cùng lần xác nhận phải dùng lại `booking_key`; khi `PENDING` hoặc `UNKNOWN`, không phát sinh booking mới và hiển thị trạng thái đang xử lý/xác minh. Khi `SUCCESS`, trả về vận đơn đã có; chỉ tạo `booking_key` mới cho yêu cầu đặt đơn mới có chủ đích.

### Exception Flow
- **E1 — Carrier từ chối rõ ràng:** Đánh dấu `booking_status = FAILED`, lưu mã lỗi/lý do từ chối; thông báo cho người gửi, không báo thành công. Chỉ cho gửi một yêu cầu mới sau khi người gửi sửa/chọn lại và xác nhận.
- **E2 — Timeout/mất kết nối, kết quả chưa rõ:** Đánh dấu `booking_status = UNKNOWN` (không coi là FAILED), giữ nguyên `booking_key` và thông tin `booking_attempt`. Hiển thị **Đang xác minh kết quả tạo vận đơn**. Trước mọi lần retry, VietGo đối chiếu mã tham chiếu/booking key hoặc tìm đơn bên carrier qua API được hỗ trợ. Nếu xác nhận carrier **đã tạo đơn** → cập nhật `SUCCESS`, lưu mã vận đơn, **không gửi lại**. Nếu carrier **xác nhận chưa tạo** và retry an toàn theo cơ chế hãng hỗ trợ → mới cho retry có kiểm soát với cùng khóa tham chiếu khi hỗ trợ, không chạy đồng thời. Nếu carrier **không hỗ trợ tra cứu hoặc vẫn chưa xác định** → giữ `UNKNOWN`, chờ đối soát/kiểm tra thủ công; **không tự động retry**.
- **E3 — Phản hồi không đủ tin cậy hoặc lỗi lưu:** Giữ `UNKNOWN` nếu chưa xác minh được kết quả carrier; không báo thành công hay gửi lại mù quáng. Lưu vết giao dịch và ưu tiên tra cứu/đối soát mã vận đơn trước khi khôi phục dữ liệu.

**Business Rules / NFR:** `booking_key` là khóa duy nhất và bất biến cho một lần xác nhận tạo vận đơn; các lần bấm lặp và retry của cùng yêu cầu tái sử dụng khóa này. `booking_status`: `PENDING` (đang xử lý), `SUCCESS` (carrier xác nhận đã tạo), `FAILED` (carrier xác nhận từ chối/chưa tạo), `UNKNOWN` (timeout/lỗi khiến kết quả chưa thể kết luận). `UNKNOWN` không được tự chuyển sang `FAILED` hoặc tự retry; phải xác minh từ carrier trước. Khóa duy nhất/transaction ngăn hai tiến trình booking đồng thời, và lưu lịch sử `booking_attempts`. Timeout API tối đa 10 giây/yêu cầu theo thiết kế; không thu tiền online trong MVP.

---

## UC-03 — Theo dõi trạng thái vận chuyển

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-03 · WF-03 · P0 |
| **Use Case Name** | Theo dõi trạng thái vận chuyển |
| **Actors** | Chính: Người gửi hàng. Phụ: GHN API/GHTK API khi cung cấp trạng thái. |
| **Preconditions** | Người gửi đã đăng nhập; vận đơn tồn tại và thuộc quyền truy cập của tài khoản. |
| **Trigger** | Người gửi chọn xem trạng thái của một vận đơn. |
| **Postconditions** | Hiển thị trạng thái được xác minh và thời điểm cập nhật; khi chọn xem lịch sử mới hiển thị timeline. Lỗi cập nhật không làm hệ thống tự đổi trạng thái. |
| **FR/AC liên quan** | BR-04, BR-05, BR-08; UR-09–UR-11; FR-13–FR-15; AC-06, AC-09. |

### Main Flow
1. Người gửi mở **Vận đơn của tôi**; VietGo kiểm tra phiên và quyền truy cập.
2. VietGo hiển thị các vận đơn thuộc tài khoản, mã tham chiếu, hãng và trạng thái hiện tại khi có dữ liệu.
3. Người gửi chọn một vận đơn (có thể lọc danh sách trước).
4. VietGo lấy trạng thái đáng tin cậy đã lưu, có thể truy vấn dữ liệu mới từ API hãng khi phù hợp.
5. VietGo ánh xạ các trạng thái nhận diện được về trạng thái chung.
6. VietGo hiển thị **trạng thái hiện tại**, nguồn và thời điểm cập nhật. **Không tự động mở lịch sử**.

### Alternate Flow
- **A1 — Xem lịch sử vận chuyển (`«extend»` UC-03):** Người gửi chủ động chọn **Xem lịch sử vận chuyển**; VietGo tải tracking event, sắp theo thời gian và hiển thị timeline; người gửi có thể quay lại màn hình trạng thái.
- **A2 — Chưa có cập nhật mới:** Hiển thị trạng thái tin cậy gần nhất và thời điểm cập nhật, không tự suy đoán đã chuyển bước.
- **A3 — Chưa có tracking event:** Khi mở lịch sử ở A1, hiển thị **Chưa có lịch sử cập nhật**; trạng thái hiện tại vẫn xem được.
- **A4 — Trạng thái kết thúc:** Hiển thị trạng thái cuối theo dữ liệu hãng; không suy luận COD đã được đối soát.

### Exception Flow
- **E1 — Không có quyền:** Từ chối truy cập, không lộ dữ liệu vận đơn.
- **E2 — Không tìm thấy vận đơn:** Thông báo **Không tìm thấy vận đơn**.
- **E3 — API hãng lỗi/timeout:** Giữ trạng thái đáng tin cậy gần nhất, ghi thời điểm cập nhật và thông báo không thể làm mới.
- **E4 — Mã trạng thái chưa nhận diện:** Không tự gán thành **Đã giao**; giữ trạng thái đã xác minh và ghi nhận lỗi mapping.

**Business Rules / NFR:** Phải kiểm tra quyền trên từng vận đơn; không tự tạo tracking event; webhook thuộc P1; tra cứu/làm mới qua dữ liệu lưu hoặc API vẫn thuộc P0.

---

## UC-04 — Xem thông tin COD

| Trường | Nội dung |
|---|---|
| **Use Case ID** | UC-04 · WF-04 · P0 |
| **Use Case Name** | Xem thông tin COD |
| **Actors** | Chính: Người gửi hàng. Phụ: GHN/GHTK cung cấp dữ liệu COD khi có. |
| **Preconditions** | Người gửi đăng nhập, có quyền xem vận đơn và VietGo đã lưu dữ liệu vận đơn. |
| **Trigger** | Người gửi mở **Thông tin COD** trong chi tiết vận đơn. |
| **Postconditions** | Chỉ hiển thị thông tin COD có căn cứ; không thu/chuyển tiền hoặc đối soát tài chính từ thao tác xem. |
| **FR/AC liên quan** | BR-07, BR-08; UR-12; FR-16; AC-07, AC-09. |

### Main Flow
1. Người gửi mở chi tiết vận đơn và chọn **Thông tin COD**.
2. VietGo xác thực phiên và kiểm tra quyền sở hữu.
3. Đọc số tiền yêu cầu thu hộ `cod_amount`, phân biệt với cước vận chuyển.
4. Nếu carrier có cung cấp, đọc và chuẩn hóa `cod_status` cùng thời điểm cập nhật.
5. Hiển thị số tiền COD, trạng thái dữ liệu, nguồn/thời điểm cập nhật và phí liên quan nếu có; không gộp COD vào doanh thu/cước.

### Alternate Flow
- **A1 — Không thu hộ:** `cod_amount = 0`; hiển thị **Không thu hộ COD**.
- **A2 — Chưa có trạng thái COD:** Có số tiền yêu cầu thu hộ nhưng carrier chưa trả trạng thái; hiển thị **Chưa có dữ liệu trạng thái COD**.
- **A3 — Đã giao nhưng chưa xác nhận COD:** Không tự kết luận đã thu hoặc đã đối soát tiền.
- **A4 — Hãng không hỗ trợ trạng thái COD:** Hiển thị số COD và ghi rõ **Hãng chưa hỗ trợ dữ liệu trạng thái COD**.

### Exception Flow
- **E1 — Không có quyền:** Từ chối truy cập dữ liệu COD.
- **E2 — Thiếu/sai dữ liệu:** Thông báo chưa khả dụng; không mặc định giá trị bị thiếu bằng 0.
- **E3 — Carrier lỗi:** Hiển thị dữ liệu đáng tin cậy đã lưu kèm thời điểm cập nhật; không khẳng định thanh toán đã hoàn tất.

**Business Rules / NFR:** `cod_amount` khác `cod_status`; COD thu hộ khác doanh thu/phí vận chuyển; VietGo không trực tiếp thu hộ hoặc tự chuyển COD về shop trong MVP.

---

## Ghi chú về phạm vi và sơ đồ

- **Use Case tổng thể:** Tài khoản, báo giá, lựa chọn dịch vụ, tạo/quản lý vận đơn, theo dõi trạng thái và lịch sử, thông tin COD.
- **Xem lịch sử vận chuyển:** Quan hệ `History ..> UC03 : <<extend>>`, không phải `include`; người gửi chủ động mở lịch sử.
- **P1:** UC-05 RepeatBox (lập lịch, tạo bản nháp); yêu cầu hủy vận đơn; quản trị/lỗi API; webhook. Không đưa các phần này vào luồng bắt buộc P0.
- **RepeatBox:** Đến lịch chỉ tạo bản nháp; người gửi vẫn lấy báo giá mới và xác nhận trước khi tạo vận đơn.
- **Giới hạn MVP:** VietGo tích hợp GHN/GHTK, không trực tiếp giao hàng; không tự dự đoán ETA hoặc tự đối soát/chuyển tiền COD.

*Nguồn nội dung: bốn bảng Use Case Specifications đã hoàn thành tại tab Nhi trong Google Docs nhóm VietGo; được chuyển sang Markdown để nộp D02 trên GitHub.*

---

## UC-05 – Lập lịch gửi hàng định kỳ và tạo bản nháp RepeatBox (P1)

| Mục | Nội dung |
|---|---|
| **Use Case ID** | UC-05 · RepeatBox · P1 |
| **Use Case Name** | Lập lịch gửi hàng định kỳ và tạo bản nháp RepeatBox |
| **Actors** | Người gửi hàng (chính); Scheduler nội bộ của VietGo (thành phần kích hoạt theo lịch, không phải carrier). |
| **Pre-conditions** | Người gửi đã đăng nhập, có quyền quản lý mẫu/kế hoạch; dữ liệu mẫu và chu kỳ gửi hợp lệ. |
| **Trigger** | Người gửi tạo kế hoạch hoặc đến lịch xử lý kỳ gửi của một kế hoạch đang hoạt động. |
| **Post-conditions** | Có kế hoạch và tối đa một bản nháp nội bộ cho mỗi kỳ hợp lệ; chỉ tạo vận đơn hãng khi người gửi chủ động xác nhận UC-02. |
| **FR / AC liên quan** | P1: SI-13; BR-09; UR-15/UR-16; FR-19–FR-24; NFR-15/NFR-16; AC-10–AC-16 (đối chiếu tab CUSTOMIZE). |

### Main Flow
1. Người gửi nhập hoặc chọn mẫu gồm người gửi, người nhận, kiện hàng và COD (nếu có).
2. VietGo kiểm tra và lưu mẫu vận đơn.
3. Người gửi thiết lập chu kỳ tuần/tháng, ngày áp dụng rồi kích hoạt kế hoạch.
4. VietGo lưu kế hoạch ở trạng thái hoạt động; chưa gửi API tạo vận đơn.
5. Khi đến kỳ, Scheduler kiểm tra kế hoạch và ràng buộc tránh trùng `(repeat_plan_id, scheduled_for)`, rồi tạo đúng một bản nháp nội bộ.
6. Người gửi mở bản nháp, kiểm tra hoặc chỉnh sửa nội dung.
7. Khi muốn đặt đơn, người gửi yêu cầu báo giá **mới** qua UC-01, chọn dịch vụ rồi chủ động xác nhận tạo vận đơn qua UC-02.
8. VietGo liên kết vận đơn đã được xác nhận với kỳ gửi tương ứng.

### Alternate Flow
- **A1 – Tạm dừng:** Kế hoạch tạm dừng không phát sinh bản nháp cho kỳ chưa tới.
- **A2 – Tiếp tục:** Kích hoạt lại kế hoạch và chỉ xử lý các kỳ còn hiệu lực.
- **A3 – Bỏ qua kỳ:** Ghi nhận kỳ gửi là `SKIPPED`, không tạo đơn.
- **A4 – Sửa mẫu/lịch:** Thay đổi áp dụng cho kỳ tương lai; không tự sửa bản nháp đã tạo.
- **A5 – Chưa muốn gửi:** Giữ bản nháp `DRAFT_READY`, không gọi API carrier.
- **A6 – Báo giá mới không khả dụng/đã thay đổi:** Giữ bản nháp và cho chọn phương án khác; không sử dụng giá cũ để tự tạo đơn.

### Exception Flow
- **E1 – Dữ liệu/lịch sai:** Chỉ rõ lỗi và không kích hoạt kế hoạch.
- **E2 – Scheduler chạy đồng thời hoặc chạy lại:** Bảo đảm mỗi `(repeat_plan_id, scheduled_for)` chỉ được tạo một bản nháp.
- **E3 – Không lưu được bản nháp:** Ghi lỗi, không đánh dấu thành công; cho cơ chế thử lại an toàn.
- **E4 – Không có quyền truy cập:** Từ chối quản lý mẫu/kế hoạch của tài khoản khác.
- **E5 – Kế hoạch đã tạm dừng/hủy:** Không tạo nháp mới cho kỳ bị chặn.

**Business Rules / NFR:** RepeatBox chỉ lập lịch và tạo bản nháp P1; không tự đặt vận đơn GHN/GHTK, không giữ giá báo cũ, không tạo kỳ/bản nháp trùng. Scheduler là thành phần nội bộ, không phải actor ngoài hệ thống.
