# VietGo — Scope (Phạm vi dự án)

## 1. Scope In — Phạm vi thực hiện

| ID | Chức năng | Mô tả phạm vi | Ưu tiên |
| --- | --- | --- | --- |
| SI-01 | Quản lý tài khoản | Đăng ký, đăng nhập, đăng xuất; người dùng quản lý đơn của mình | P0 |
| SI-02 | Nhập thông tin đơn hàng | Nhập địa chỉ gửi/nhận, người liên hệ, hàng hóa, khối lượng, kích thước (nếu áp dụng), giá trị và COD | P0 |
| SI-03 | Tích hợp API đa hãng | Kết nối GHN và GHTK để yêu cầu báo giá và dịch vụ khả dụng | P0 |
| SI-04 | So sánh dịch vụ vận chuyển | Chuẩn hóa và hiển thị giá cước, các loại phí, hãng và dịch vụ trên cùng giao diện | P0 |
| SI-05 | Hiển thị ETA | Hiển thị thời gian giao dự kiến do carrier cung cấp, nếu API hỗ trợ | P0 |
| SI-06 | Lựa chọn dịch vụ và tạo vận đơn | Người dùng chọn phương án, xác nhận thông tin và tạo đơn qua API hãng vận chuyển | P0 |
| SI-07 | Quản lý và theo dõi vận đơn | Xem mã vận đơn, hãng, trạng thái, lịch sử cập nhật và chi tiết đơn hàng | P0 |
| SI-08 | Quản lý COD cơ bản | Lưu tiền COD, hiển thị thông tin COD và trạng thái thu hộ nếu carrier trả về | P0 |
| SI-09 | Quản lý đơn hàng cá nhân | Xem danh sách, lọc đơn theo hãng và trạng thái | P0 |
| SI-10 | Quản trị hệ thống cơ bản | Admin xem danh sách đơn, trạng thái tích hợp và lỗi API | P1 |
| SI-11 | Hủy vận đơn | Cho phép gửi yêu cầu hủy qua API nếu trạng thái và chính sách hãng cho phép | P1 |
| SI-12 | Đồng bộ trạng thái tự động | Tiếp nhận webhook và lưu lịch sử thay đổi trạng thái đơn | P1 |
| SI-13 | RepeatBox – chuẩn bị đơn định kỳ | Cho phép shop tạo và quản lý mẫu đơn, thiết lập lịch gửi theo tuần/tháng, tự tạo bản nháp theo kỳ và yêu cầu người gửi xác nhận trước khi tạo vận đơn thật. | P1 |
| SI-14 | Tiếp nhận và xử lý khiếu nại vận chuyển | Cho phép shop tạo ticket gắn với vận đơn thuộc quyền sở hữu, bổ sung chứng từ và theo dõi tiến độ. CSKH VietGo tiếp nhận, chuyển tiếp đến GHN qua Ticket API khi có quyền hoặc GHTK qua kênh hỗ trợ thủ công, cập nhật phản hồi và kết quả. | P1 |


## 2. Scope Out — Phạm vi không triển khai

| ID | Chức năng ngoài phạm vi | Lý do / giới hạn |
| --- | --- | --- |
| SO-01 | Quản lý tài xế, đội xe, hub và kho trung chuyển | VietGo không trực tiếp vận hành hoạt động vận chuyển. |
| SO-02 | Điều phối tài xế và tối ưu tuyến đường | Thuộc nghiệp vụ vận hành của hãng vận chuyển. |
| SO-03 | AI dự đoán ETA | ETA chỉ được hiển thị khi có dữ liệu hợp lệ từ hãng; không tự xây dựng mô hình dự đoán. |
| SO-04 | Tự xây dựng thuật toán tính giá cước | Sử dụng dữ liệu báo giá từ hãng; VietGo chuẩn hóa và trình bày, không tự đặt biểu giá thay hãng. |
| SO-05 | Tích hợp thêm hãng ngoài GHN và GHTK | MVP gốc giới hạn ở hai hãng này. |
| SO-06 | Sàn thương mại điện tử đa nhà bán | Không phát triển catalog, giỏ hàng hoặc checkout mua sản phẩm. |
| SO-07 | Thanh toán trực tuyến và ví điện tử | Không tích hợp cổng thanh toán trực tuyến hoặc ví điện tử trong phạm vi hiện tại. |
| SO-08 | Tự động thu hộ, chuyển tiền và đối soát COD | Chỉ quản lý thông tin COD do hãng cung cấp; không xây dựng nghiệp vụ tài chính tự động. |
| SO-09 | Báo cáo tài chính chuyên sâu | Không xây dựng hệ thống báo cáo tài chính toàn diện. |
| SO-10 | Bồi thường tự động, xử lý tài chính và hoàn hàng phức tạp | VietGo không tự quyết định bồi thường, xử lý tài chính, hoàn hàng phức tạp hoặc cam kết kết quả của hãng. Quy trình ticket khiếu nại cơ bản thuộc SI-14 vẫn nằm trong phạm vi P1. |
| SO-11 | Ứng dụng Android/iOS riêng | Ưu tiên website responsive. |
| SO-12 | Tích hợp trực tiếp với Shopee, TikTok Shop và WooCommerce | Không thuộc mục tiêu tích hợp của MVP hiện tại. |
| SO-13 | Quản lý nhiều kho và nhiều chi nhánh | Không triển khai hệ thống quản lý đa kho/chi nhánh trong phạm vi hiện tại. |
| SO-14 | CRM và tổng đài CSKH nâng cao | VietGo không xây dựng hệ thống CRM hoặc tổng đài CSKH nâng cao; chỉ cung cấp giao diện quản lý ticket và lịch sử xử lý khiếu nại cho nhân viên CSKH. |
| SO-15 | Tự động tạo vận đơn từ kế hoạch RepeatBox | RepeatBox chỉ tạo bản nháp nội bộ. Người dùng phải mở bản nháp, lấy báo giá mới và xác nhận theo quy trình hiện có trước khi tạo vận đơn. |
| SO-16 | Thu tiền thuê bao RepeatBox | Không triển khai thanh toán phí thuê bao hoặc mô hình thu phí định kỳ cho RepeatBox. |
| SO-17 | Quản lý sản phẩm và tồn kho trong RepeatBox | Mẫu đơn chỉ lưu thông tin cần thiết cho việc gửi hàng; không trở thành hệ thống quản lý catalog, sản phẩm hoặc tồn kho. |

## 3. Acceptance Criteria — Tiêu chí nghiệm thu

| ID | Tiêu chí nghiệm thu | Điều kiện đạt | Scope In liên quan |
| --- | --- | --- | --- |
| AC-01 | Nhập thông tin đơn hàng | Hệ thống kiểm tra các trường bắt buộc và thông báo rõ khi dữ liệu không hợp lệ. | SI-02 |
| AC-02 | Tích hợp đa hãng | Nhận được báo giá từ GHN/GHTK; nếu sử dụng dữ liệu mock, dữ liệu phải được gắn nhãn rõ ràng. | SI-03 |
| AC-03 | So sánh giá vận chuyển | Phí hiển thị có căn cứ từ dữ liệu hợp lệ; phí thiếu không được mặc định thành 0 và không bị cộng trùng. | SI-04 |
| AC-04 | Hiển thị ETA | Hiển thị ETA khi có dữ liệu hợp lệ; nếu không có, hệ thống thông báo “Chưa có ETA”. | SI-05 |
| AC-05 | Tạo vận đơn | Lưu và hiển thị kết quả cùng mã vận đơn nếu có; chỉ thông báo thành công khi nhận được xác nhận hợp lệ. | SI-06 |
| AC-06 | Theo dõi vận đơn (Tracking) | Hiển thị trạng thái và lịch sử vận chuyển khi có dữ liệu từ hãng vận chuyển. | SI-07, SI-12 |
| AC-07 | Thông tin COD | Hiển thị đúng số tiền COD và trạng thái nếu hãng vận chuyển cung cấp dữ liệu. | SI-08 |
| AC-08 | Xử lý lỗi tích hợp | Lỗi của một hãng không chặn kết quả hợp lệ của hãng còn lại; hệ thống hiển thị thông báo lỗi phù hợp. | SI-03 |
| AC-09 | Bảo mật cơ bản | Token API không bị lộ ở phía client; người dùng không thể truy cập vận đơn trái quyền. | SI-01, SI-03, SI-09 |
| AC-10 | Lưu và sử dụng mẫu đơn | Người dùng tạo được mẫu đơn và có thể sử dụng lại thông tin đã lưu để lập bản nháp. | SI-13 |
| AC-11 | Thiết lập lịch gửi định kỳ | Người dùng tạo được kế hoạch gửi hàng theo tuần hoặc tháng với ngày gửi dự kiến hợp lệ. | SI-13 |
| AC-12 | Tạo bản nháp đến hạn | Khi một kỳ đến hạn, hệ thống tạo đúng một bản nháp nội bộ và không gọi API tạo vận đơn của hãng vận chuyển. | SI-13 |
| AC-13 | Chống tạo bản nháp trùng | Khi Scheduler chạy nhiều lần cho cùng một kỳ, hệ thống không tạo thêm bản nháp trùng lặp. | SI-13 |
| AC-14 | Báo giá lại trước khi tạo vận đơn | Khi mở bản nháp để tiếp tục quy trình gửi hàng, hệ thống phải lấy báo giá mới trước khi người dùng xác nhận tạo vận đơn. | SI-13 |
| AC-15 | Tạm dừng kế hoạch và bỏ qua kỳ | Kế hoạch đã tạm dừng không phát sinh kỳ mới; kỳ đã bỏ qua không tự động xuất hiện lại như một kỳ cần xử lý. | SI-13 |
| AC-16 | Liên kết kỳ gửi với vận đơn | Chỉ liên kết kỳ gửi với vận đơn khi có kết quả tạo vận đơn hợp lệ. Trường hợp thất bại hoặc chưa xác định kết quả không được ghi nhận là thành công. | SI-13 |
| AC-17 | Phân quyền khiếu nại | Shop chỉ tạo và xem ticket đối với vận đơn thuộc quyền sở hữu. | SI-14 |
| AC-18 | Lưu ticket nội bộ | Ticket lưu mã VietGo, shipment_id, loại vấn đề, mô tả và lịch sử thao tác ngay cả khi chưa chuyển hãng. | SI-14 |
| AC-19 | Kiểm tra dữ liệu và chứng từ | Dữ liệu/file không hợp lệ bị từ chối có lý do; CSKH có thể yêu cầu bổ sung chứng từ. | SI-14 |
| AC-20 | Xử lý GHN Ticket API | Chỉ ghi nhận mã ticket GHN khi phản hồi hợp lệ; timeout hoặc kết quả chưa rõ không được tự ghi thành công hoặc tự gửi lại mù. | SI-14 |
| AC-21 | Chuyển tiếp GHTK thủ công | Lịch sử thể hiện chuyển tiếp và phản hồi thủ công, không gán nhãn đã sử dụng API GHTK. | SI-14 |
| AC-22 | Theo dõi và cập nhật ticket | Shop và CSKH xem được trạng thái, lịch sử; các thao tác cập nhật có phân quyền và lưu vết. | SI-14 |
| AC-23 | Chống trùng và độc lập trạng thái | Callback/phản hồi lặp không tạo lịch sử trùng; ticket không làm thay đổi trạng thái vận đơn hoặc đối soát COD. | SI-14 |


