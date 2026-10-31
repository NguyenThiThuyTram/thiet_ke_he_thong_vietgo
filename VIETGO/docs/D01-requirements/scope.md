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
| SI-13 | Quản lý kế hoạch gửi hàng định kỳ – RepeatBox | Cho phép lưu mẫu đơn, thiết lập chu kỳ gửi theo tuần/tháng, tự tạo bản nháp nội bộ khi đến hạn, xem danh sách và lịch sử các kỳ gửi, bỏ qua kỳ, tạm dừng/tiếp tục kế hoạch và xác nhận gửi thông qua quy trình hiện có. | P1 |


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
| SO-10 | Quản lý khiếu nại, bồi thường và hoàn hàng phức tạp | Phụ thuộc quy trình và chính sách của hãng. |
| SO-11 | Ứng dụng Android/iOS riêng | Ưu tiên website responsive. |
| SO-12 | Tích hợp trực tiếp với Shopee, TikTok Shop và WooCommerce | Không thuộc mục tiêu tích hợp của MVP hiện tại. |
| SO-13 | Quản lý nhiều kho và nhiều chi nhánh | Không triển khai hệ thống quản lý đa kho/chi nhánh trong phạm vi hiện tại. |
| SO-14 | CRM nâng cao | Không triển khai hệ thống chăm sóc khách hàng nâng cao. |
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
