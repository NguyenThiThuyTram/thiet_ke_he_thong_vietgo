# Requirements — VietGo

Tài liệu tổng hợp yêu cầu nghiệp vụ, yêu cầu người dùng, yêu cầu chức năng và phi chức năng. Mỗi bảng sử dụng thống nhất ba cột: **ID**, **Mô tả**, **Liên quan**.

> Quy ước: Cột **Liên quan** của BR tham chiếu Scope In (SI) trong `scope.md` để truy vết phạm vi (không biểu thị SI có trước BR). UR → BR và NFR → FR là liên kết đề xuất dựa trên nội dung; BR → SI cũng là liên kết đề xuất cần nhóm xác nhận. FR → UR được giữ theo tài liệu gốc.

## 1. Business Requirements (BR)

| ID | Mô tả | Liên quan |
| --- | --- | --- |
| BR-01 | VietGo phải tích hợp GHN và GHTK để hỗ trợ người dùng sử dụng dịch vụ vận chuyển qua một nền tảng thống nhất. | SI-03 |
| BR-02 | VietGo phải hỗ trợ so sánh dịch vụ, cước phí, phụ phí và tổng phí dựa trên dữ liệu báo giá hợp lệ từ hãng. | SI-03, SI-04 |
| BR-03 | VietGo phải cho phép người dùng nhập thông tin đơn hàng một lần, lựa chọn dịch vụ và yêu cầu tạo vận đơn thông qua nền tảng. | SI-02, SI-06 |
| BR-04 | VietGo phải cung cấp giao diện thống nhất để người dùng quản lý và tra cứu vận đơn của mình. | SI-07, SI-09 |
| BR-05 | VietGo phải hiển thị trạng thái hiện tại và lịch sử vận chuyển dựa trên dữ liệu do hãng cung cấp. | SI-07, SI-12 |
| BR-06 | VietGo phải hiển thị ETA khi hãng cung cấp dữ liệu hợp lệ; nếu không, hệ thống phải thể hiện rõ rằng chưa có ETA. | SI-05 |
| BR-07 | VietGo phải lưu trữ và hiển thị số tiền COD cùng trạng thái COD khi có dữ liệu từ hãng; không bao gồm đối soát tài chính tự động. | SI-08 |
| BR-08 | VietGo phải hạn chế thông báo sai kết quả tạo vận đơn, xử lý lỗi tích hợp theo từng hãng và bảo vệ dữ liệu đơn hàng. | SI-01, SI-03, SI-06, SI-10 |
| BR-09 | VietGo phải hỗ trợ người gửi hàng thường xuyên tái sử dụng thông tin đơn hàng và lập kế hoạch chuẩn bị vận đơn theo chu kỳ tuần/tháng, nhằm giảm thao tác nhập liệu lặp lại và hỗ trợ quản lý từng đợt gửi. | SI-13 |

## 2. User Requirements (UR)

| ID | Mô tả | Liên quan (BR) |
| --- | --- | --- |
| UR-01 | Là người gửi hàng, tôi muốn đăng ký, đăng nhập và đăng xuất để sử dụng VietGo an toàn. | BR-01, BR-08 |
| UR-02 | Tôi muốn nhập thông tin người gửi, người nhận, kiện hàng, giá trị hàng hóa và COD để yêu cầu báo giá. | BR-03 |
| UR-03 | Tôi muốn yêu cầu báo giá GHN và GHTK qua một biểu mẫu duy nhất. | BR-01, BR-02 |
| UR-04 | Tôi muốn so sánh dịch vụ, cước phí, phụ phí và tổng phí để lựa chọn phương án phù hợp. | BR-02 |
| UR-05 | Tôi muốn xem ETA do hãng cung cấp để cân nhắc thời gian giao hàng. | BR-06 |
| UR-06 | Tôi muốn chọn dịch vụ vận chuyển từ các phương án báo giá hợp lệ. | BR-03 |
| UR-07 | Tôi muốn kiểm tra và xác nhận thông tin trước khi yêu cầu tạo vận đơn. | BR-03, BR-08 |
| UR-08 | Tôi muốn xem kết quả tạo vận đơn và mã vận đơn VietGo/hãng nếu có. | BR-03, BR-08 |
| UR-09 | Tôi muốn xem danh sách vận đơn và trạng thái của các đơn thuộc tài khoản của mình. | BR-04 |
| UR-10 | Tôi muốn lọc danh sách vận đơn theo hãng vận chuyển hoặc trạng thái. | BR-04 |
| UR-11 | Tôi muốn xem trạng thái hiện tại và lịch sử vận chuyển dựa trên dữ liệu từ hãng. | BR-05 |
| UR-12 | Tôi muốn xem số tiền COD và trạng thái COD khi hãng có cung cấp dữ liệu. | BR-07 |
| UR-13 | Tôi muốn được thông báo rõ khi yêu cầu báo giá hoặc tạo vận đơn thất bại, bao gồm trường hợp timeout hoặc kết quả chưa xác định. | BR-08 |
| UR-14 | Là quản trị viên, tôi muốn xem danh sách vận đơn, trạng thái tích hợp và lỗi API để giám sát hệ thống. | BR-08 |
| UR-15 | Là một người gửi hàng thường xuyên, tôi muốn lưu mẫu đơn và thiết lập lịch gửi theo tuần hoặc tháng để VietGo tự chuẩn bị bản nháp cho từng kỳ mà không cần nhập lại toàn bộ thông tin. | BR-09 |
| UR-16 | Là một người gửi hàng, tôi muốn xem, kiểm tra, xác nhận hoặc bỏ qua bản nháp của từng kỳ và tạm dừng kế hoạch khi cần, để không phát sinh vận đơn ngoài ý muốn. | BR-09 |

## 3. Functional Requirements (FR)

| ID | Mô tả | Liên quan (UR) |
| --- | --- | --- |
| FR-01 | Hệ thống phải cho phép người gửi đăng ký, đăng nhập và đăng xuất để truy cập các chức năng quản lý vận đơn. | UR-01 |
| FR-02 | Hệ thống phải cho phép người gửi nhập thông tin đơn hàng gồm địa chỉ gửi, địa chỉ nhận, thông tin liên hệ, khối lượng, kích thước khi áp dụng, giá trị hàng hóa và số tiền COD. | UR-02 |
| FR-03 | Hệ thống phải kiểm tra tính đầy đủ và hợp lệ của thông tin đơn hàng trước khi gửi yêu cầu báo giá hoặc tạo vận đơn. | UR-02, UR-07 |
| FR-04 | Hệ thống phải gửi yêu cầu báo giá đến GHN và GHTK thông qua API tương ứng. | UR-03 |
| FR-05 | Hệ thống phải chuẩn hóa kết quả báo giá từ GHN và GHTK để hiển thị thống nhất trên cùng giao diện. | UR-03, UR-04 |
| FR-06 | Hệ thống phải hiển thị hãng vận chuyển, tên dịch vụ, cước phí, phụ phí và tổng phí của từng phương án khi có dữ liệu. | UR-04 |
| FR-07 | Hệ thống phải phân biệt các khoản phí có dữ liệu với các khoản phí chưa có dữ liệu, không tự xem trường bị thiếu là bằng 0. | UR-04 |
| FR-08 | Hệ thống phải hiển thị ETA khi có dữ liệu hợp lệ từ hãng vận chuyển; nếu không có thì thông báo chưa có dữ liệu ETA. | UR-05 |
| FR-09 | Hệ thống phải cho phép người dùng lựa chọn dịch vụ vận chuyển từ kết quả báo giá hợp lệ. | UR-06 |
| FR-10 | Hệ thống phải hiển thị thông tin đơn hàng và dịch vụ đã chọn để người dùng xác nhận trước khi tạo vận đơn. | UR-07 |
| FR-11 | Hệ thống phải gửi yêu cầu tạo vận đơn và lưu mã vận đơn VietGo, mã vận đơn của hãng nếu có, cùng kết quả xử lý. | UR-07, UR-08 |
| FR-12 | Hệ thống phải hiển thị rõ kết quả tạo vận đơn và không thông báo thành công khi chưa có xác nhận hợp lệ. | UR-08, UR-13 |
| FR-13 | Hệ thống phải cho phép người dùng xem danh sách vận đơn của mình, bao gồm hãng vận chuyển, mã vận đơn và trạng thái hiện tại khi có dữ liệu. | UR-09 |
| FR-14 | Hệ thống phải cho phép người dùng lọc danh sách vận đơn theo hãng vận chuyển và trạng thái. | UR-10 |
| FR-15 | Hệ thống phải cho phép người dùng xem trạng thái hiện tại và lịch sử vận chuyển khi có dữ liệu từ hãng vận chuyển. | UR-11 |
| FR-16 | Hệ thống phải lưu trữ và hiển thị số tiền COD cùng trạng thái thu hộ khi hãng vận chuyển cung cấp dữ liệu. | UR-12 |
| FR-17 | Hệ thống phải thông báo rõ ràng khi yêu cầu báo giá hoặc tạo vận đơn thất bại. | UR-13 |
| FR-18 | Hệ thống phải cho phép quản trị viên xem danh sách đơn hàng, trạng thái tích hợp và lỗi API. | UR-14 |
| FR-19 | Hệ thống cho phép người dùng tạo, xem và cập nhật mẫu đơn gồm thông tin gửi/nhận, kiện hàng và COD khi có. | UR-15 |
| FR-20 | Hệ thống cho phép người dùng tạo, xem, tạm dừng và tiếp tục kế hoạch gửi theo tuần hoặc tháng. | UR-15, UR-16 |
| FR-21 | Hệ thống kiểm tra các kế hoạch đến hạn và tự động sinh tối đa một bản nháp nội bộ cho mỗi kỳ. | UR-15 |
| FR-22 | Hệ thống hiển thị danh sách kỳ gửi, ngày dự kiến và trạng thái; cho phép bỏ qua kỳ chưa xác nhận. | UR-16 |
| FR-23 | Hệ thống cho phép người dùng mở bản nháp và tiếp tục quy trình tra cứu cước, chọn dịch vụ, xác nhận tạo vận đơn hiện có. | UR-16 |
| FR-24 | Sau khi tạo vận đơn thành công, hệ thống liên kết vận đơn với kỳ gửi tương ứng và cập nhật lịch sử RepeatBox. | UR-16 |

## 4. Non-Functional Requirements (NFR)

| ID | Mô tả | Liên quan (FR) |
| --- | --- | --- |
| NFR-01 | Đăng nhập, xem và lọc vận đơn phải phản hồi trong vòng 3 giây ở điều kiện tải bình thường. | FR-01, FR-13, FR-14 |
| NFR-02 | Mỗi yêu cầu API báo giá hoặc tạo vận đơn có thời gian chờ tối đa 10 giây. Khi hết thời gian chờ, hệ thống phải thông báo trạng thái phù hợp. | FR-04, FR-11, FR-17 |
| NFR-03 | Lỗi API của một hãng vận chuyển không được làm gián đoạn việc hiển thị kết quả hợp lệ từ hãng còn lại. | FR-04, FR-17 |
| NFR-04 | Mọi yêu cầu truy cập dữ liệu vận đơn phải được kiểm tra quyền. Người dùng chỉ được xem vận đơn thuộc tài khoản của mình. | FR-13, FR-14, FR-15 |
| NFR-05 | Token API của GHN/GHTK chỉ được lưu trữ và sử dụng ở phía máy chủ, không được để lộ trên trình duyệt. | FR-04, FR-11 |
| NFR-06 | Thông tin đơn hàng, mã vận đơn, phí vận chuyển và COD phải nhất quán với dữ liệu hợp lệ nhận được từ hệ thống liên quan. | FR-05, FR-06, FR-11, FR-16 |
| NFR-07 | Phí vận chuyển bị thiếu hoặc không hợp lệ không được tự động mặc định thành 0. | FR-07 |
| NFR-08 | ETA chỉ được hiển thị khi có dữ liệu hợp lệ từ hãng vận chuyển; hệ thống không tự dự đoán ETA. | FR-08 |
| NFR-09 | Hệ thống không được thông báo tạo vận đơn thành công khi chưa có xác nhận hợp lệ. Khi kết quả chưa xác định, hệ thống phải hạn chế gửi lại yêu cầu gây trùng vận đơn. | FR-11, FR-12 |
| NFR-10 | Dữ liệu phản hồi từ GHN/GHTK phải được ánh xạ về cấu trúc dữ liệu chung để phục vụ việc hiển thị và xử lý thống nhất. | FR-05 |
| NFR-11 | Lỗi tích hợp phải được ghi nhận kèm thông tin hãng vận chuyển và thời điểm xảy ra để hỗ trợ tra cứu. | FR-17, FR-18 |
| NFR-12 | Giao diện website phải hỗ trợ hiển thị trên máy tính, máy tính bảng và điện thoại theo định hướng responsive. | FR-01, FR-02, FR-13 |
| NFR-13 | Thành phần tích hợp GHN và GHTK phải được tách biệt nhằm hạn chế ảnh hưởng chéo khi một thành phần gặp lỗi. | FR-04, FR-05 |
| NFR-14 | Dữ liệu mock sử dụng trong bản demo phải được nhận diện rõ ràng, không được trình bày như giao dịch thực tế. | FR-04, FR-05 |
| NFR-15 | Hệ thống phải bảo đảm mỗi kế hoạch RepeatBox chỉ tạo tối đa một bản nháp cho mỗi kỳ dự kiến, kể cả khi tác vụ Scheduler chạy lặp lại. | FR-21 |
| NFR-16 | Chỉ người dùng sở hữu kế hoạch RepeatBox mới được xem hoặc chỉnh sửa kế hoạch và các bản nháp liên quan. Việc tạo bản nháp, bỏ qua kỳ hoặc tạm dừng kế hoạch không được tự ý thay đổi vận đơn đã được hãng vận chuyển xác nhận. | FR-19, FR-20, FR-21, FR-22, FR-23, FR-24 |
