# Bài 1

## Algorithm là gì?
- Định nghĩa: Một thuật toán là phương pháp giải quyết một bài toán tính toán.
- Cách thực hiện: 
  - Thu thập đầu vào (input) và xử lý để tạo ra đầu ra (output).
  - Trong Python, một thuật toán thường được cài đặt dưới dạng một hàm, với input là tham số và output là giá trị trả về.
- Ví dụ: Đếm số lượng số chẵn trong danh sách
  - Danh sách mẫu: `[5, 4, 1, 7, 9, 6]`
  - Kết quả mong đợi: `2` (vì có 2 số chẵn: 4 và 6)

### Ví dụ cài đặt ban đầu:

```python
def count_even(numbers):
    result = 0
    for x in numbers:
        if x % 2 == 0:
            result += 1
    return result

print(count_even([1, 2, 3]))      # 1
print(count_even([2, 2, 2, 2, 2]))# 5
print(count_even([5, 4, 1, 7, 9, 6])) # 2
```

### Cách cài đặt rút gọn với generator expression:
```python
def count_even(numbers):
    return sum(x % 2 == 0 for x in numbers)
```
- Ở đây, hàm `sum` tính tổng của các giá trị Boolean (True được tính là 1, False là 0).

## Data Structure - Cấu trúc dữ liệu
- Định nghĩa: Cách lưu trữ dữ liệu trong chương trình.
- List là cấu trúc dữ liệu cơ bản (Trong Python, nó kiểu `[1, 2, 3, 4, 5]` á), tuy nhiên có rất nhiều cấu trúc khác.
- Lựa chọn cấu trúc dữ liệu phù hợp ảnh hưởng lớn đến hiệu quả thuật toán.

## "Hiện thực hóa" cái thuật toán

Bạn có thể sử dụng các thành phần cơ bản này (Đây là trong Python, thường cũng sẽ đúng với đa phần các ngôn ngữ khác). Cá nhân tui đọc thì thấy cũng tạm hiểu dù không lập trình Python:

- Variables
- Operators (như `+`, `=`, v.v.)
- Conditionals (`if`)
- Loops (`for`, `while`)
- Lists
- Functions
- Classes

## Efficiency of algorithms (Hiệu quả của Thuật toán)
- Là khả năng giải quyết bài toán một cách nhanh chóng, đặc biệt với đầu vào lớn.
- Ví dụ: Tìm hiệu số lớn nhất giữa hai số trong danh sách
  - Danh sách mẫu: `[3, 2, 6, 5, 8, 5]`
  - Kết quả mong đợi: `6` (hiệu số giữa 2 và 8)

### Xem thử các chương trình dưới đây:

#### Thuật toán 1 (sử dụng 2 vòng lặp lồng nhau):
```python
def max_diff(numbers):
    result = 0
    for x in numbers:
        for y in numbers:
            result = max(result, abs(x - y))
    return result
```

#### Thuật toán 2 (sử dụng sắp xếp):
```python
def max_diff(numbers):
    numbers = sorted(numbers)
    return numbers[-1] - numbers[0]
```

#### Thuật toán 3 (sử dụng hàm min và max):
```python
def max_diff(numbers):
    return max(numbers) - min(numbers)
```

### Thực nghiệm đo hiệu suất:

Bảng so sánh thời gian thực thi:

| Độ dài danh sách n | Thuật toán 1 | Thuật toán 2 | Thuật toán 3 |
| ------------------ | ------------ | ------------ | ------------ |
| 1000               | 0.17 s       | 0.00 s       | 0.00 s       |
| 10000              | 15.93 s      | 0.00 s       | 0.00 s       |
| 100000             | –            | 0.01 s       | 0.00 s       |
| 1000000            | –            | 0.27 s       | 0.02 s       |

Nói chung là đọc cái bài này xong gất chi là khai sáng với một con người đần code như tui.

## Measuring efficiency (Đo độ hiệu quả)
Thử đọc cái quả code này:

```python
1 def count_even(numbers):
2    result = 0 
3    for x in numbers:
4        if x % 2 == 0:
5            result += 1
6    return result
```

- Gọi n là độ dài danh sách:
	- Các dòng ngoài vòng lặp (khởi tạo và return): thực hiện 1 lần (Không trong loop mà chỉ là một dòng "statement" nên được tính là 1).
	- Các dòng bên trong vòng lặp: thực hiện n lần (Dòng số 3 và 4, cái này thì trong vòng lặp, tính cả dòng `for x in numbers:` á :<).
	- Dòng `result += 1` kia (Dòng số 5) thì thực hiện từ 0 đến n lần (Do nếu if thỏa mãn điều kiện thì dòng đó mới được chạy)

Tổng số bước thực hiện: Từ `2n + 2` đến `3n + 2` bước (Tại sao lại ra được cái số này?). Giải thích cho các bồ về cái `2n + 2` đến `3n + 2`:

- `2n + 2`: Là trường hợp "tốt nhất", tức là cái `if` nào cũng không thỏa mãn nên không phải chạy dòng dưới.
- `3n + 2`: Ngược lại cái trên, là cái if nào cũng đúng nên dòng 5 được chạy `n` lần. 

## Độ phức tạp về thời gian (Time Complexity)

Là biểu thị số bước thực hiện của thuật toán theo kích thước đầu vào (n), thường dưới dạng $O(...)$).

### Quy tắc đơn giản hóa:

Bạn chỉ cần đấy cái n có số mũ lớn nhất thôi á, bỏ qua các hằng số hay số có số mũ không phải lớn nhất. Kiểu ví dụ trong $f(n) = n^2 - 2n - 1$ thì $O(f(n)) = O(n^2)$. Ít nhất thì tui hiểu là vậy. Một ví dụ khác là hàm `count_even` có số bước tối đa là `3n+2` nên độ phức tạp được coi là O(n).


### Các lớp độ phức tạp thường thấy:
- O(1): Thời gian hằng số (không có vòng lặp).
- O(log n): Thời gian logarit.
- O(n): Thời gian tuyến tính (vòng lặp đơn).
- O(n log n): Phổ biến với các thuật toán sắp xếp.
- O(n²): Vòng lặp lồng nhau.
- O(n³): Ba vòng lặp lồng nhau.

### Phân tích dựa trên cấu trúc vòng lặp:
- Không có vòng lặp: O(1)
- Một vòng lặp đơn (1 loop) : O(n)
- Vòng lặp lồng nhau (nested loop): O(n²) (với k vòng lặp lồng nhau thì là O(n^k))
- Các đoạn mã thực thi tuần tự: Độ phức tạp tổng bằng độ phức tạp của đoạn có độ phức tạp lớn nhất.

