# 2. PHÂN TÍCH YÊU CẦU (REQUIREMENT ANALYSIS)

## 2.0. Các yêu cầu kinh doanh (Business Requirements)

| ID | Requirement | Mô tả (Description) |
|---|---|---|
| BR-01 | Tập trung hóa quản lý vận chuyển | VIETGO cần giúp người gửi và chủ shop quản lý vận đơn từ GHN và GHTK trên một nền tảng thống nhất. |
| BR-02 | Nâng cao khả năng lựa chọn dịch vụ | VIETGO cần hỗ trợ người gửi so sánh chi phí và thông tin dịch vụ vận chuyển để lựa chọn phương án phù hợp. |
| BR-03 | Đơn giản hóa quy trình gửi hàng | VIETGO cần giảm việc chuyển đổi giữa các nền tảng khi tra cứu cước và tạo vận đơn. |
| BR-04 | Tăng khả năng kiểm soát giao hàng | VIETGO cần giúp chủ shop theo dõi vận đơn và tiến trình giao hàng từ nhiều hãng tập trung. |
| BR-05 | Hỗ trợ kiểm soát thông tin COD | VIETGO cần hỗ trợ theo dõi thông tin thu hộ COD do các hãng vận chuyển cung cấp, không thực hiện đối soát tiền. |
| BR-06 | Duy trì hoạt động tra cứu đa hãng | VIETGO cần hạn chế gián đoạn tra cứu khi một hãng vận chuyển không phản hồi. |
| BR-07 | Hỗ trợ giám sát vận hành | VIETGO cần giúp quản trị viên nhận biết và theo dõi lỗi tích hợp với các hãng vận chuyển. |
| BR-08 | Giảm thao tác lặp lại khi gửi hàng định kỳ | VIETGO cần hỗ trợ shop chuẩn bị đơn gửi theo tuần hoặc tháng thông qua bản nháp RepeatBox trước khi xác nhận tạo vận đơn. |

## 2.1. Các yêu cầu người dùng (User Requirements)

| ID | Requirement | Mô tả (Description) |
|---|---|---|
| UR-01 | Đăng ký và đăng nhập tài khoản | Người gửi cần đăng ký, đăng nhập và đăng xuất để quản lý hoạt động gửi hàng của mình. |
| UR-02 | Nhập thông tin gửi hàng | Người gửi cần nhập thông tin người gửi, người nhận, kiện hàng và COD nếu có. |
| UR-03 | Tra cứu báo giá đa hãng | Người gửi cần xem các dịch vụ và báo giá khả dụng của GHN và GHTK tại một nơi. |
| UR-04 | So sánh cước phí và ETA | Người gửi cần so sánh cước phí, phụ phí, tổng phí và ETA nếu hãng cung cấp. |
| UR-05 | Lựa chọn dịch vụ và tạo vận đơn | Người gửi cần chọn dịch vụ và xác nhận thông tin trước khi tạo vận đơn. |
| UR-06 | Xem và lọc vận đơn | Người gửi cần xem và lọc các vận đơn thuộc tài khoản của mình. |
| UR-07 | Theo dõi vận chuyển | Người gửi cần xem trạng thái hiện tại và lịch sử vận chuyển. |
| UR-08 | Theo dõi COD | Chủ shop cần xem số tiền COD và trạng thái thu hộ do hãng cung cấp. |
| UR-09 | Nhận thông báo lỗi | Người gửi cần biết khi báo giá hoặc tạo vận đơn thất bại hay chưa rõ kết quả. |
| UR-10 | Giám sát lỗi tích hợp | Quản trị viên cần xem danh sách vận đơn, trạng thái kết nối và lỗi API. |
| UR-11 | Lưu mẫu gửi hàng định kỳ | Chủ shop cần lưu mẫu thông tin kiện hàng và người nhận để tái sử dụng. |
| UR-12 | Quản lý lịch gửi định kỳ | Chủ shop cần tạo lịch theo tuần/tháng, xem, tạm dừng và tiếp tục kế hoạch. |
| UR-13 | Kiểm tra bản nháp định kỳ | Chủ shop cần kiểm tra, điều chỉnh hoặc bỏ qua bản nháp của kỳ gửi. |
| UR-14 | Xác nhận vận đơn từ bản nháp | Chủ shop cần nhận báo giá mới và xác nhận trước khi tạo vận đơn thật từ bản nháp. |

## 2.2. Các yêu cầu chức năng (Functional Requirements)

| ID | Requirement | Mô tả (Description) |
|---|---|---|
| FR-01 | Quản lý tài khoản và nhập thông tin gửi hàng | Hệ thống phải cho phép người dùng đăng ký, đăng nhập, đăng xuất và nhập thông tin người gửi, người nhận, kiện hàng để yêu cầu dịch vụ vận chuyển. |
| FR-02 | Tra cứu báo giá từ nhiều hãng vận chuyển | Hệ thống phải gửi yêu cầu báo giá qua API của GHN và GHTK để lấy các dịch vụ vận chuyển khả dụng theo thông tin kiện hàng và địa chỉ giao nhận. |
| FR-03 | Chuẩn hóa và so sánh cước vận chuyển | Hệ thống phải chuẩn hóa và hiển thị hãng vận chuyển, dịch vụ, cước phí, phụ phí, tổng phí và thời gian giao dự kiến (ETA) khi hãng cung cấp dữ liệu hợp lệ; nếu không có ETA, hệ thống phải thể hiện rõ là chưa có dữ liệu. |
| FR-04 | Xác nhận dịch vụ và tạo vận đơn | Hệ thống phải cho phép người dùng lựa chọn dịch vụ, kiểm tra và xác nhận thông tin trước khi gửi yêu cầu tạo vận đơn qua API của hãng vận chuyển đã chọn; chỉ thông báo thành công khi nhận được kết quả xác nhận hợp lệ. |
| FR-05 | Xem và lọc danh sách vận đơn | Hệ thống phải cho phép người dùng xem danh sách, chi tiết và lọc các vận đơn thuộc tài khoản của mình theo hãng vận chuyển và trạng thái. |
| FR-06 | Theo dõi trạng thái vận chuyển | Hệ thống phải hiển thị trạng thái hiện tại và lịch sử vận chuyển của từng vận đơn dựa trên dữ liệu do hãng vận chuyển cung cấp. |
| FR-07 | Lưu và hiển thị thông tin COD | Hệ thống phải lưu trữ và hiển thị số tiền thu hộ (COD) và trạng thái thu hộ khi hãng vận chuyển cung cấp dữ liệu; không thực hiện đối soát hoặc chuyển tiền COD. |
| FR-08 | Thông báo lỗi và giám sát tích hợp | Hệ thống phải thông báo khi tra cứu báo giá hoặc tạo vận đơn thất bại; đồng thời cho phép quản trị viên xem danh sách vận đơn, trạng thái tích hợp và lỗi API. |
| FR-09 | Quản lý lịch gửi hàng định kỳ RepeatBox | Hệ thống phải cho phép người dùng lưu mẫu thông tin gửi hàng, tạo kế hoạch gửi theo tuần hoặc tháng, xem lịch, tạm dừng và tiếp tục kế hoạch. |
| FR-10 | Tạo và xử lý bản nháp RepeatBox | Khi một kỳ gửi đến hạn, hệ thống phải tạo tối đa một bản nháp tương ứng; cho phép người dùng kiểm tra, bỏ qua, yêu cầu báo giá mới và xác nhận tạo vận đơn. Hệ thống chỉ liên kết vận đơn với kỳ gửi sau khi hãng xác nhận tạo thành công; không tự động tạo vận đơn thật khi chưa có xác nhận của người dùng. |

## 2.3. Các yêu cầu phi chức năng (Non-functional Requirements)

| ID | Requirement | Mô tả (Description) |
|---|---|---|
| NFR-01 | Đảm bảo thời gian phản hồi | Trong điều kiện tải bình thường, hệ thống phải phản hồi thao tác đăng nhập, xem và lọc vận đơn trong tối đa 3 giây; mỗi yêu cầu API báo giá hoặc tạo vận đơn phải có thời gian chờ tối đa 10 giây trước khi chuyển sang xử lý lỗi hoặc chưa xác định kết quả. |
| NFR-02 | Bảo vệ dữ liệu và phân quyền truy cập | Hệ thống phải xác thực và kiểm tra quyền truy cập đối với dữ liệu người dùng, vận đơn, chức năng quản trị và RepeatBox; khóa và thông tin xác thực API GHN/GHTK phải được lưu và sử dụng ở phía máy chủ, không để lộ cho trình duyệt. |
| NFR-03 | Duy trì khả năng chịu lỗi và hạn chế trùng vận đơn | Hệ thống phải tiếp tục hiển thị các báo giá hợp lệ nếu một hãng vận chuyển gặp lỗi; không được thông báo tạo vận đơn thành công khi kết quả API chưa xác định và phải áp dụng biện pháp hạn chế gửi yêu cầu tạo vận đơn trùng lặp. |
| NFR-04 | Bảo đảm tính toàn vẹn và nhất quán dữ liệu | Hệ thống phải duy trì tính nhất quán của báo giá, cước phí, ETA, trạng thái vận đơn và COD; không tự suy diễn giá trị thiếu; không được tạo nhiều hơn một bản nháp RepeatBox cho cùng một kế hoạch và kỳ gửi. |
| NFR-05 | Hỗ trợ bảo trì và tương thích thiết bị | Hệ thống phải tổ chức phần tích hợp GHN và GHTK thành các thành phần độc lập, có cơ chế chuẩn hóa dữ liệu và ghi nhận lỗi tích hợp; giao diện website phải sử dụng được trên máy tính, máy tính bảng và điện thoại. |
