# Bài 2: Mô hình Internet 4 lớp (4-Layer Internet Model)

Mô hình Internet 4 lớp được thiết kế để giúp các ứng dụng có thể tái sử dụng các khối xây dựng chung, thay vì phải tạo mới cho mỗi ứng dụng. Mỗi lớp có một chức năng riêng biệt và cung cấp dịch vụ cho lớp trên nó.

## 1. Lớp Liên kết (Link Layer)

Chức năng: Truyền dữ liệu qua một liên kết (link) giữa hai thiết bị mạng, chẳng hạn như giữa máy tính và bộ định tuyến (router), hoặc giữa các bộ định tuyến.

Ví dụ: Ethernet và Wi-Fi là các giao thức thuộc lớp này.

Ghi chú: Lớp Liên kết hoạt động theo phương thức "hop-by-hop", nghĩa là dữ liệu được chuyển từ một nút mạng đến nút mạng kế tiếp trên đường đi đến đích.

## 2. Lớp Mạng (Network Layer)

Chức năng: Định tuyến (routing) và chuyển tiếp các gói tin (packet) từ nguồn đến đích qua nhiều liên kết và bộ định tuyến.

Giao thức chính: Giao thức Internet (Internet Protocol - IP).

Ghi chú: Gói tin ở lớp này được gọi là datagram. Lớp Mạng cung cấp dịch vụ "best-effort", tức là không đảm bảo gói tin sẽ đến đích, có thể bị mất, trùng lặp hoặc đến không theo thứ tự.

## 3. Lớp Vận chuyển (Transport Layer)

Chức năng: Cung cấp giao tiếp dữ liệu đáng tin cậy giữa các ứng dụng trên các thiết bị khác nhau.

Giao thức chính:

- TCP (Transmission Control Protocol): Đảm bảo dữ liệu được truyền tải đầy đủ và theo đúng thứ tự. Nếu gói tin bị mất hoặc lỗi, TCP sẽ tự động gửi lại.

- UDP (User Datagram Protocol): Không đảm bảo độ tin cậy hoặc thứ tự của dữ liệu, thích hợp cho các ứng dụng yêu cầu tốc độ cao và chấp nhận mất mát dữ liệu, như truyền video trực tuyến.

Ghi chú: Lớp Vận chuyển giúp kiểm soát tắc nghẽn (congestion control) và cung cấp cơ chế phát hiện lỗi.

## 4. Lớp Ứng dụng (Application Layer)

Chức năng: Cung cấp các giao thức và giao diện cho các ứng dụng mạng để giao tiếp với nhau.

Ví dụ: HTTP (HyperText Transfer Protocol) cho web, FTP (File Transfer Protocol) cho truyền tệp, SMTP (Simple Mail Transfer Protocol) cho email.

Ghi chú: Lớp này định nghĩa cú pháp và ngữ nghĩa của dữ liệu được trao đổi giữa các ứng dụng.

## Tóm tắt

Mô hình 4 lớp của Internet giúp phân tách các chức năng mạng thành các tầng riêng biệt, cho phép sự linh hoạt và khả năng mở rộng. Mỗi lớp cung cấp dịch vụ cho lớp trên nó và sử dụng dịch vụ của lớp dưới nó, tạo nên một hệ thống mạng hoạt động hiệu quả và đáng tin cậy. 

## Bổ sung: xàm l thêm về mô hình 7 lớp
Có lẽ mn đã/sẽ được học về mô hình 7 lớp OSI thì đây là bảng so sánh:

Mô hình OSI (Open Systems Interconnection) gồm 7 lớp, trong khi mô hình Internet (còn gọi là mô hình TCP/IP) có 4 lớp. Dưới đây là bảng so sánh các lớp tương ứng giữa hai mô hình này:

| Mô hình OSI (7 lớp) | Mô hình Internet (4 lớp) |
| --- | --- |
| Lớp 7: Application | Lớp 4: Application |
| Lớp 6: Presentation |  |
| Lớp 5: Session | |
| Lớp 4: Transport | Lớp 3: Transport |
| Lớp 3: Network | Lớp 2: Internet |
| Lớp 2: Data Link | Lớp 1: Network Access |
| Lớp 1: Physical | |

Giải thích chi tiết:

-   Lớp 7 (Application), Lớp 6 (Presentation),  của mô hình OSI: Ba lớp này được kết hợp thành Lớp 4 (Application) trong mô hình Internet 4 lớp. Lớp Application trong mô hình Internet đảm nhận các chức năng liên quan đến giao tiếp trực tiếp với ứng dụng người dùng, bao gồm hiển thị dữ liệu và quản lý phiên làm việc.
-   Lớp 4 (Transport) và Lớp 5 (Session) của mô hình OSI: Tương ứng với Lớp 3 (Transport) trong mô hình Internet. Cả hai lớp này đều chịu trách nhiệm về truyền dữ liệu end-to-end và kiểm soát lỗi. ​

-   Lớp 3 (Network) của mô hình OSI: Tương ứng với Lớp 2 (Internet) trong mô hình Internet. Chúng đảm nhiệm việc định tuyến và chuyển tiếp gói tin qua các mạng khác nhau. ​

-   Lớp 2 (Data Link) và Lớp 1 (Physical) của mô hình OSI: Hai lớp này được kết hợp thành Lớp 1 (Network Access) trong mô hình Internet. Lớp Network Access xử lý các khía cạnh liên quan đến truyền dữ liệu qua các phương tiện vật lý và kiểm soát truy cập vào phương tiện đó.
