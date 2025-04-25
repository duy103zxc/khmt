# Bài 1: Giới thiệu về Computer Networking

### Mô hình bốn lớp của Internet:
- Lớp Ứng dụng (Application Layer): Giao tiếp trực tiếp với phần mềm ứng dụng và xác định các giao thức như HTTP, FTP.
- Lớp Giao vận (Transport Layer): Đảm bảo truyền dữ liệu đáng tin cậy giữa các thiết bị, ví dụ như sử dụng giao thức TCP.
- Lớp Mạng (Network Layer): Xử lý định tuyến và địa chỉ IP, đảm bảo các gói dữ liệu đến đúng đích.
- Lớp Liên kết Dữ liệu (Data Link Layer): Quản lý truyền dữ liệu giữa hai nút mạng liền kề và xử lý các lỗi truyền dẫn.

### Chuyển mạch gói (Packet Switching)

- Giao thức IP (Internet Protocol):
  - Là "keo dán" của Internet, cho phép các thiết bị kết nối và giao tiếp với nhau.
  - Địa chỉ IP xác định duy nhất mỗi thiết bị trên mạng và hỗ trợ định tuyến gói tin đến đúng đích.
- Công cụ kiểm tra kết nối mạng:
  - Sử dụng các công cụ phần mềm để kiểm tra và phân tích cách máy tính sử dụng Internet, giúp hiểu rõ hơn về hoạt động mạng khi duyệt web.
- Ứng dụng mạng và kết nối:
  - Web: Sử dụng mô hình máy khách-máy chủ (client-server) để truy cập và hiển thị nội dung từ các máy chủ web.
  - BitTorrent: Cho phép chia sẻ tệp tin lớn giữa nhiều người dùng bằng cách chia nhỏ tệp thành các phần và tải xuống từ nhiều nguồn khác nhau.
  - Skype: Sử dụng kết nối ngang hàng (peer-to-peer) để truyền tải âm thanh và video, với sự hỗ trợ của các máy chủ trung gian khi cần thiết để vượt qua các hạn chế mạng như NAT.

### Về NAT trong quá trình truyền và nhận dữ liệu (Network Address Translation):
- NAT cho phép nhiều thiết bị trong mạng nội bộ sử dụng một địa chỉ IP công cộng duy nhất, nhưng gây khó khăn cho việc thiết lập kết nối từ bên ngoài vào.
- Các ứng dụng như Skype phải sử dụng các máy chủ trung gian (rendezvous servers) hoặc máy chủ chuyển tiếp (relay servers) để thiết lập kết nối giữa các thiết bị nằm sau NAT.

Bài giảng này cung cấp cái nhìn tổng quan về các khái niệm cơ bản và nguyên tắc hoạt động của mạng máy tính, đặt nền tảng cho việc tìm hiểu sâu hơn về các giao thức và ứng dụng mạng trong các bài giảng tiếp theo. 