# Bài 2

**Danh sách trong bộ nhớ**

Bộ nhớ máy tính bao gồm một dãy các vị trí có thể lưu trữ dữ liệu, mỗi vị trí có một địa chỉ cụ thể. Khi chương trình được thực thi, dữ liệu được lưu trữ trong bộ nhớ. Ví dụ, xem xét đoạn mã Python sau:


```python
a = 7
b = -3
c = [1, 2, 3, 1, 2]
d = 99
```


Giả sử các biến và danh sách được lưu trữ trong bộ nhớ bắt đầu từ địa chỉ 100. Biến `a` được lưu tại địa chỉ 100, `b` tại 101, và danh sách `c` chiếm các địa chỉ từ 102 đến 109, với 5 phần tử đầu tiên được sử dụng. Biến `d` được lưu tại địa chỉ 110. Các phần tử của danh sách được lưu trữ liên tiếp trong bộ nhớ, giúp việc xác định vị trí của một phần tử cụ thể trở nên dễ dàng. Địa chỉ của một phần tử được tính bằng cách cộng chỉ số của phần tử đó vào địa chỉ của phần tử đầu tiên. Ví dụ, phần tử `c[2]` nằm tại địa chỉ 102 + 2 = 104.

**Các thao tác trên danh sách**

Python cung cấp nhiều thao tác tích hợp để quản lý danh sách. Hiểu rõ độ phức tạp thời gian của các thao tác này là quan trọng để thiết kế thuật toán hiệu quả. Hầu hết các thao tác trên danh sách có độ phức tạp thời gian là O(1) hoặc O(n):

- **Truy cập và thay đổi phần tử**: Sử dụng toán tử chỉ số `[]` để truy cập và thay đổi phần tử. Thao tác này có độ phức tạp O(1) vì các phần tử được lưu trữ liên tiếp trong bộ nhớ.

  
```python
  numbers = [4, 3, 7, 3, 2]
  print(numbers[2])  # 7
  numbers[2] = 5
  print(numbers[2])  # 5
  ```


- **Kích thước danh sách**: Hàm `len` trả về số lượng phần tử trong danh sách với độ phức tạp O(1).

  
```python
  numbers = [4, 3, 7, 3, 2]
  print(len(numbers))  # 5
  ```


- **Tìm kiếm**: Toán tử `in` kiểm tra xem một phần tử có trong danh sách hay không. Phương thức `index` trả về chỉ số của lần xuất hiện đầu tiên của phần tử. Phương thức `count` đếm số lần xuất hiện của phần tử. Các thao tác này có độ phức tạp O(n) vì cần duyệt qua danh sách.

  
```python
  numbers = [4, 3, 7, 3, 2]
  print(3 in numbers)       # True
  print(numbers.index(3))   # 1
  print(numbers.count(3))   # 2
  ```


- **Thêm phần tử**: Phương thức `append` thêm một phần tử vào cuối danh sách với độ phức tạp O(1). Phương thức `insert` thêm một phần tử vào vị trí cụ thể với độ phức tạp O(n) vì cần di chuyển các phần tử khác để tạo chỗ trống.

  
```python
  numbers = [1, 2, 3, 4]
  numbers.append(5)
  print(numbers)  # [1, 2, 3, 4, 5]
  numbers.insert(1, 6)
  print(numbers)  # [1, 6, 2, 3, 4, 5]
  ```


- **Xóa phần tử**: Phương thức `pop` loại bỏ một phần tử khỏi danh sách. Nếu không có tham số, nó loại bỏ phần tử cuối cùng với độ phức tạp O(1). Nếu có tham số, nó loại bỏ phần tử tại chỉ số cụ thể với độ phức tạp O(n) vì cần di chuyển các phần tử khác để lấp đầy khoảng trống. Phương thức `remove` loại bỏ lần xuất hiện đầu tiên của một phần tử cụ thể với độ phức tạp O(n).

  
```python
  numbers = [1, 2, 3, 4, 5, 6]
  numbers.pop()
  print(numbers)  # [1, 2, 3, 4, 5]
  numbers.pop(1)
  print(numbers)  # [1, 3, 4, 5]
  numbers.remove(3)
  print(numbers)  # [1, 4, 5]
  ```


**Tóm tắt**

Bảng sau tóm tắt độ phức tạp thời gian của các thao tác trên danh sách:

| Thao tác                        | Độ phức tạp thời gian |
|---------------------------------|-----------------------|
| Truy cập phần tử (`[]`)         | O(1)                  |
| Lấy kích thước (`len`)          | O(1)                  |
| Kiểm tra phần tử (`in`)         | O(n)                  |
| Tìm kiếm (`index`)              | O(n)                  |
| Đếm (`count`)                   | O(n)                  |
| Thêm vào cuối (`append`)        | O(1)                  |
| Thêm vào giữa (`insert`)        | O(n)                  |
| Xóa từ cuối (`pop`)             | O( 