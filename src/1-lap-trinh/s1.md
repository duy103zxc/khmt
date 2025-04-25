# OOP - CS61A

**Lập trình hướng đối tượng (Object-Oriented Programming - OOP) | Góc nhìn cấp cao (Above the line view)**

Tài liệu này nên được đọc trước Phần 3.1 của sách. Một tài liệu thứ hai, "Lập trình hướng đối tượng | Góc nhìn cấp thấp (Below the line view)," nên được đọc sau Phần 3.1 và có thể sau Phần 3.2; ý tưởng là bạn trước tiên học cách sử dụng tính năng lập trình hướng đối tượng, sau đó bạn học cách nó được triển khai.

Lập trình hướng đối tượng là một phép ẩn dụ. Nó diễn đạt ý tưởng về nhiều tác nhân độc lập bên trong máy tính, thay vì một quá trình duy nhất thao tác các dữ liệu khác nhau. Ví dụ, dự án lập trình tiếp theo là một trò chơi phiêu lưu, trong đó nhiều người, địa điểm và đồ vật tương tác. Chúng ta muốn có thể nói những điều như "Yêu cầu Fred lấy đĩa bánh bao." (Fred là một đối tượng người, và đĩa bánh bao là một đối tượng đồ vật.)

Các lập trình viên sử dụng phép ẩn dụ đối tượng có một từ vựng đặc biệt để mô tả các thành phần của một hệ thống lập trình hướng đối tượng (OOP). Trong ví dụ trên, "Fred" được gọi là một *instance* (thể hiện) và danh mục chung "người" được gọi là một *class* (lớp). Các ngôn ngữ lập trình hỗ trợ OOP cho phép lập trình viên nói trực tiếp bằng từ vựng này; ví dụ, mọi ngôn ngữ OOP đều có lệnh "define class" dưới một hình thức nào đó. Đối với khóa học này, chúng tôi đã cung cấp một phần mở rộng cho Scheme hỗ trợ OOP theo phong cách của các ngôn ngữ OOP khác. Sau này chúng ta sẽ thấy cách các tính năng mới này được triển khai bằng cách sử dụng các khả năng Scheme mà bạn đã hiểu. OOP không phải là phép thuật; đó là một cách suy nghĩ và nói về cấu trúc của một chương trình.

Khi chúng ta nói về một "phép ẩn dụ," về mặt kỹ thuật, chúng ta có nghĩa là chúng ta đang cung cấp một sự trừu tượng. Góc nhìn cấp cao là góc nhìn về các tác nhân độc lập. Bên dưới dòng kẻ có ba ý tưởng kỹ thuật quan trọng: *message-passing* (truyền thông điệp) (phần 2.3), *local state* (trạng thái cục bộ) (phần 3.1) và *inheritance* (kế thừa) (được giải thích bên dưới). Tài liệu này sẽ giải thích cách các ý tưởng này xuất hiện đối với lập trình viên OOP; sau này chúng ta sẽ thấy cách chúng được triển khai.

Một phiên bản đơn giản hơn của hệ thống này và các ghi chú này đến từ MIT; phiên bản này được phát triển tại Berkeley bởi Matt Wright.

Để sử dụng hệ thống OOP, bạn phải tải tệp ~cs61a/lib/obj.scm vào Scheme.

**Truyền thông điệp (Message Passing)**

Cách để mọi thứ xảy ra trong một hệ thống hướng đối tượng là gửi thông điệp đến các đối tượng yêu cầu chúng làm điều gì đó. Bạn đã biết về truyền thông điệp; chúng ta đã sử dụng kỹ thuật này trong Phần 2.3 để triển khai các toán tử chung bằng cách sử dụng dữ liệu "thông minh". Ví dụ, trong Phần 3.1, phần lớn cuộc thảo luận sẽ là về các đối tượng tài khoản ngân hàng. Mỗi tài khoản có số dư (số tiền trong đó); bạn có thể gửi thông điệp đến một tài khoản cụ thể để gửi hoặc rút tiền. Phiên bản của sách cho thấy cách các đối tượng này có thể được tạo bằng ký hiệu Scheme thông thường, nhưng bây giờ chúng ta sẽ sử dụng từ vựng OOP để làm điều tương tự. Giả sử chúng ta có hai đối tượng Matt-Account và Brian-Account của lớp tài khoản ngân hàng. (Bạn chưa thể nhập điều này vào Scheme; ví dụ giả định rằng chúng ta đã tạo các đối tượng này.)

```scheme
> (ask Matt-Account 'balance)
1000
> (ask Brian-Account 'balance)
10000
> (ask Matt-Account 'deposit 100)
1100
> (ask Brian-Account 'withdraw 200)
9800
> (ask Matt-Account 'balance)
1100
> (ask Brian-Account 'withdraw 200)
9600
```

Chúng ta sử dụng thủ tục `ask` để gửi thông điệp đến một đối tượng. Trong ví dụ trên, chúng ta giả định rằng các đối tượng tài khoản ngân hàng biết về ba thông điệp: `balance`, `deposit` và `withdraw`. Lưu ý rằng một số thông điệp yêu cầu thông tin bổ sung; khi chúng ta yêu cầu số dư, điều đó là đủ, nhưng khi chúng ta yêu cầu một tài khoản rút hoặc gửi tiền, chúng ta cũng cần chỉ định số tiền.

Phép ẩn dụ là một đối tượng "biết cách" làm những việc nhất định. Những việc này được gọi là *methods* (phương thức). Bất cứ khi nào bạn gửi một thông điệp đến một đối tượng, đối tượng thực hiện phương thức mà nó liên kết với thông điệp đó.

**Trạng thái cục bộ (Local State)**

Lưu ý rằng trong ví dụ trên, chúng ta liên tục nói

```scheme
(ask Brian-Account 'withdraw 200)
```

và nhận được một câu trả lời khác nhau mỗi lần. Điều đó có vẻ hoàn toàn tự nhiên, bởi vì đó là cách các tài khoản ngân hàng hoạt động trong đời thực. Tuy nhiên, cho đến bây giờ chúng ta đã sử dụng mô hình lập trình hàm, trong đó, theo định nghĩa, gọi cùng một hàm hai lần với cùng các đối số phải cho cùng một kết quả.

Trong mô hình OOP, các đối tượng có *state* (trạng thái). Nghĩa là, chúng có một số kiến thức về những gì đã xảy ra với chúng trong quá khứ. Trong ví dụ này, một tài khoản ngân hàng có số dư, thay đổi khi bạn gửi hoặc rút một số tiền. Hơn nữa, mỗi tài khoản có số dư riêng. Trong thuật ngữ OOP, chúng ta nói rằng `balance` là một biến trạng thái cục bộ.

Bạn đã biết biến cục bộ là gì: tham số hình thức của một thủ tục là một biến cục bộ. Khi bạn nói

```scheme
(define (square x) (* x x))
```

biến `x` là cục bộ đối với thủ tục `square`. Nếu bạn có một thủ tục khác `(cube x)`, biến `x` của nó sẽ hoàn toàn tách biệt với biến `x` của `square`. Tương tự, số dư của Matt-Account được giữ tách biệt với số dư của Brian-Account.

Mặt khác, mỗi khi bạn gọi `square`, bạn cung cấp một giá trị mới cho `x`; không có bộ nhớ về giá trị `x` đã có lần trước. Một biến trạng thái là một biến có giá trị tồn tại giữa các lần gọi. Sau khi bạn gửi một số tiền vào Matt-Account, giá trị mới của biến `balance` được ghi nhớ vào lần tiếp theo bạn truy cập tài khoản.

Để tạo các đối tượng trong hệ thống này, bạn tạo một thể hiện (instantiate) của một lớp. Ví dụ, Matt-Account và Brian-Account là các thể hiện của lớp `account`:

```scheme
> (define Matt-Account (instantiate account 1000))
Matt-Account
> (define Brian-Account (instantiate account 10000))
Brian-Account
```

Hàm `instantiate` lấy một lớp làm đối số đầu tiên và trả về một đối tượng mới của lớp đó. `Instantiate` có thể yêu cầu các đối số bổ sung tùy thuộc vào lớp cụ thể: trong ví dụ này, bạn chỉ định số dư ban đầu của tài khoản khi bạn tạo nó.

Phần lớn mã trong một chương trình hướng đối tượng bao gồm các định nghĩa của các lớp khác nhau. Đây là lớp `account`:

```scheme
(define-class (account balance)
  (method (deposit amount)
    (set! balance (+ amount balance))
    balance)
  (method (withdraw amount)
    (if (< balance amount)
        "Insufficient funds"
        (begin
          (set! balance (- balance amount))
          balance))))
```

Có rất nhiều điều để nói về mã này. Trước hết, có một dạng đặc biệt mới, `define-class`. Cú pháp của `define-class` tương tự như cú pháp của `define`. Nơi bạn mong đợi thấy tên của thủ tục bạn đang định nghĩa là tên của lớp bạn đang định nghĩa. Thay vì các tham số cho một thủ tục là các biến khởi tạo của lớp: đây là các biến trạng thái cục bộ có giá trị ban đầu phải được đưa ra dưới dạng các đối số bổ sung cho `instantiate`. Phần thân của một lớp bao gồm bất kỳ số lượng mệnh đề nào; trong ví dụ này chỉ có một loại mệnh đề, mệnh đề `method`, nhưng chúng ta sẽ tìm hiểu về những mệnh đề khác sau. Thứ tự mà các mệnh đề xuất hiện trong `define-class` không quan trọng.

Cú pháp để định nghĩa các phương thức cũng được chọn để giống với cú pháp để định nghĩa các thủ tục. "Tên" của phương thức thực sự là thông điệp được sử dụng để truy cập phương thức. Các tham số cho phương thức tương ứng với các đối số bổ sung cho thủ tục `ask`. Ví dụ, khi chúng ta nói

```scheme
(ask Matt-Account 'deposit 100)
```

chúng ta liên kết đối số 100 với tham số `amount`.

Bạn có thể đang tự hỏi chúng ta đã định nghĩa phương thức `balance` ở đâu. Đối với mỗi biến trạng thái cục bộ trong một lớp, một phương thức tương ứng có cùng tên được định nghĩa tự động. Các phương thức này không có đối số và chúng chỉ trả về giá trị hiện tại của biến có tên đó.

Ví dụ này cũng giới thiệu hai dạng đặc biệt mới không chỉ dành riêng cho hệ thống đối tượng. Dạng thứ nhất là `set!`, có nhiệm vụ thay đổi giá trị của một biến trạng thái. Đối số đầu tiên của nó không được đánh giá; đó là tên của biến mà bạn muốn thay đổi giá trị. Đối số thứ hai được đánh giá; giá trị của biểu thức này trở thành giá trị mới của biến. Giá trị trả về của `set!` không được định nghĩa.

Điều này trông giống như loại `define` không có dấu ngoặc đơn xung quanh đối số đầu tiên, nhưng ý nghĩa khác nhau. `Define` tạo một biến mới, trong khi `set!` thay đổi giá trị của một biến hiện có.

Tên `set!` có dấu chấm than trong tên của nó vì quy ước Scheme cho các thủ tục sửa đổi một thứ gì đó. (Đây chỉ là một quy ước, giống như quy ước về dấu chấm hỏi trong tên của các hàm vị từ, không phải là một quy tắc cứng nhắc.) Lý do chúng ta chưa gặp quy ước này trước đây là vì lập trình hàm loại trừ toàn bộ ý tưởng sửa đổi mọi thứ; không có bộ nhớ về lịch sử quá khứ trong một chương trình hàm.

Dạng đặc biệt nguyên thủy Scheme khác trong ví dụ này là `begin`, đánh giá tất cả các biểu thức đối số của nó theo thứ tự và trả về giá trị của biểu thức cuối cùng. Cho đến bây giờ, trong mọi thủ tục, chúng ta chỉ đánh giá một biểu thức, để cung cấp giá trị trả về của thủ tục đó. Vẫn đúng là một thủ tục chỉ có thể trả về một giá trị. Tuy nhiên, bây giờ chúng ta đôi khi muốn đánh giá một biểu thức cho những gì nó làm thay vì những gì nó trả về, ví dụ như thay đổi giá trị của một biến. Lệnh gọi `begin` chỉ ra rằng `(set! amount (- amount balance))` và `balance` cùng nhau tạo thành một đối số duy nhất cho `if`. Bạn sẽ tìm hiểu thêm về `set!` và `begin` trong Chương 3.

**Kế thừa (Inheritance)**

Hãy tưởng tượng sử dụng OOP trong một chương trình phức tạp với nhiều loại đối tượng khác nhau. Rất thường xuyên, sẽ có một vài lớp gần giống nhau. Ví dụ, hãy nghĩ về một hệ thống cửa sổ. Có thể có nhiều loại cửa sổ khác nhau (cửa sổ văn bản, cửa sổ đồ họa, v.v.) nhưng tất cả chúng sẽ có một số phương thức chung, ví dụ như phương thức di chuyển cửa sổ đến một vị trí khác trên màn hình. Chúng ta không muốn phải lập trình lại cùng một phương thức trong nhiều lớp. Thay vào đó, chúng ta tạo một lớp tổng quát hơn (chẳng hạn như "cửa sổ") biết về các phương thức chung này; các lớp cụ thể (như "cửa sổ văn bản") kế thừa từ lớp tổng quát. Thực tế, định nghĩa của lớp tổng quát được bao gồm trong định nghĩa của lớp cụ thể hơn.

Giả sử chúng ta muốn tạo một lớp tài khoản séc. Tài khoản séc giống như tài khoản ngân hàng thông thường, ngoại trừ việc bạn có thể viết séc cũng như rút tiền trực tiếp. Nhưng bạn bị tính phí mười xu mỗi khi viết séc.

```scheme
> (define Hal-Account (instantiate checking-account 1000))
Hal-Account
> (ask Hal-Account 'balance)
1000
> (ask Hal-Account 'deposit 100)
1100
> (ask Hal-Account 'withdraw 50)
1050
> (ask Hal-Account 'write-check 30)
1019.9
```

Một cách để làm điều này là sao chép tất cả mã cho tài khoản thông thường trong định nghĩa của `checking-account`. Tuy nhiên, điều này không tuyệt vời lắm; nếu chúng ta muốn thêm một tính năng mới vào lớp `account`, chúng ta cần nhớ thêm nó vào lớp `checking-account`.

Rất phổ biến trong lập trình hướng đối tượng là một lớp sẽ là một chuyên môn hóa của một lớp khác: lớp mới sẽ có tất cả các phương thức của lớp cũ, cộng với một số phương thức bổ sung, giống như trong ví dụ tài khoản ngân hàng này. Để mô tả tình huống này, chúng ta sử dụng phép ẩn dụ về một họ các lớp đối tượng. Lớp ban đầu là lớp cha và phiên bản chuyên biệt là lớp con. Chúng ta nói rằng lớp con kế thừa các phương thức của lớp cha. (Các tên `subclass` cho lớp con và `superclass` cho lớp cha đôi khi cũng được sử dụng.)

Đây là cách chúng ta tạo một lớp con của lớp `account`:

```scheme
(define-class (checking-account init-balance)
  (parent (account init-balance))
  (method (write-check amount)
    (ask self 'withdraw (+ amount 0.10))))
```

Ví dụ này giới thiệu mệnh đề `parent` trong `define-class`. Trong trường hợp này, lớp cha là lớp `account`. Bất cứ khi nào chúng ta gửi một thông điệp đến một đối tượng `checking-account`, phương thức tương ứng đến từ đâu? Nếu một phương thức có tên đó được định nghĩa trong lớp `checking-account`, nó sẽ được sử dụng; nếu không, hệ thống OOP sẽ tìm kiếm một phương thức trong lớp cha `account`. (Nếu lớp đó cũng có một lớp cha, chúng ta có thể kế thừa một phương thức từ lớp đã bị xóa hai lần đó, v.v.)

Cũng lưu ý rằng phương thức `write-check` tham chiếu đến một biến có tên là `self`. Mỗi đối tượng có một biến trạng thái cục bộ `self` có giá trị là chính đối tượng đó. (Lưu ý rằng bạn có thể viết một phương thức trong định nghĩa của một lớp C nghĩ rằng `self` sẽ luôn là một thể hiện của C, nhưng trên thực tế, `self` có thể hóa ra là một thể hiện của một lớp khác có C là lớp cha của nó.)

Các phương thức được định nghĩa trong một lớp nhất định chỉ có quyền truy cập vào các biến trạng thái cục bộ được định nghĩa trong cùng một lớp. Ví dụ, một phương thức được định nghĩa trong lớp `checking-account` không thể tham chiếu đến biến `balance` được định nghĩa trong lớp `account`; tương tự, một phương thức trong lớp `account` không thể tham chiếu đến biến `init-balance`. Quy tắc này tương ứng với quy tắc Scheme thông thường về phạm vi của biến: mỗi biến chỉ khả dụng trong khối mà nó được định nghĩa. (Không phải mọi triển khai OOP đều hoạt động như thế này.)

Nếu một phương thức trong lớp `checking-account` cần tham chiếu đến biến `balance` được định nghĩa trong lớp cha của nó, phương thức đó có thể nói

```scheme
(ask self 'balance)
```

Lệnh gọi `ask` này gửi một thông điệp đến đối tượng `checking-account`, nhưng vì không có phương thức `balance` nào được định nghĩa trong chính lớp `checking-account`, phương thức được kế thừa từ lớp `account` sẽ được sử dụng.

Chúng ta đã sử dụng tên `init-balance` cho biến khởi tạo của lớp mới, thay vì chỉ là `balance`, bởi vì chúng ta muốn tên đó có nghĩa là biến thuộc về lớp cha. Vì hệ thống OOP tự động tạo một phương thức được đặt tên theo mọi biến cục bộ trong lớp, nếu chúng ta gọi biến này là `balance` thì chúng ta không thể sử dụng thông điệp `balance` để truy cập vào biến trạng thái `balance` của lớp cha. (Rốt cuộc, đó là lớp cha mà số dư của tài khoản được thay đổi cho mỗi giao dịch.)

Bây giờ chúng ta đã mô tả ba phần quan trọng nhất của hệ thống OOP: truyền thông điệp, trạng thái cục bộ và kế thừa.

Trong phần còn lại của tài liệu này, chúng ta giới thiệu một số "chuông và còi" - các tính năng bổ sung giúp ký hiệu linh hoạt hơn, nhưng không thực sự liên quan đến các ý tưởng mới chính.

**Ba loại biến trạng thái cục bộ**

Cho đến nay, các biến trạng thái cục bộ duy nhất chúng ta đã thấy là các biến khởi tạo, có giá trị được đưa ra dưới dạng đối số khi một đối tượng được tạo. Đôi khi chúng ta muốn mỗi thể hiện có một biến trạng thái cục bộ, nhưng giá trị ban đầu giống nhau đối với mọi đối tượng trong lớp, vì vậy chúng ta không muốn phải đề cập đến nó ở Tất nhiên rồi, đây là phần dịch tiếp theo:

...mỗi lần tạo thể hiện. Để đạt được mục đích này, chúng ta sẽ sử dụng một loại mệnh đề `define-class` mới, được gọi là `instance-vars`:

```scheme
(define-class (checking-account init-balance)
  (parent (account init-balance))
  (instance-vars (check-fee 0.10))
  (method (write-check amount)
    (ask self 'withdraw (+ amount check-fee)))
  (method (set-fee! fee)
    (set! check-fee fee)))
```

Chúng ta đã thiết lập mọi thứ để mỗi tài khoản séc mới sẽ có một khoản phí mười xu cho mỗi séc. Có thể thay đổi phí cho bất kỳ tài khoản nào, nhưng chúng ta không cần phải nói gì nếu chúng ta muốn giữ nguyên giá trị mười xu.

Các biến khởi tạo cũng là các biến thể hiện; nghĩa là, mỗi thể hiện có giá trị riêng tư cho chúng. Sự khác biệt duy nhất là trong ký hiệu - đối với các biến khởi tạo, bạn đưa ra một giá trị khi bạn gọi `instantiate`, nhưng đối với các biến thể hiện khác, bạn đưa ra giá trị trong định nghĩa lớp.

Loại biến trạng thái cục bộ thứ ba là biến lớp. Không giống như trường hợp của các biến thể hiện, chỉ có một giá trị cho một biến lớp cho toàn bộ lớp. Mọi thể hiện của lớp đều chia sẻ giá trị này. Ví dụ, giả sử chúng ta muốn có một lớp công nhân đều đang làm việc trong cùng một dự án. Nghĩa là, bất cứ khi nào bất kỳ ai trong số họ làm việc, tổng lượng công việc đã hoàn thành sẽ tăng lên. Mặt khác, mỗi công nhân đều đói riêng khi họ làm việc. Do đó, có một biến `work-done` chung cho lớp và một biến `hunger` riêng biệt cho mỗi thể hiện.

```scheme
(define-class (worker)
  (instance-vars (hunger 0))
  (class-vars (work-done 0))
  (method (work)
    (set! hunger (1+ hunger))
    (set! work-done (1+ work-done))
    'whistle-while-you-work))
```

```scheme
> (define brian (instantiate worker))
BRIAN
> (define matt (instantiate worker))
MATT
> (ask matt 'work)
WHISTLE-WHILE-YOU-WORK
> (ask matt 'work)
WHISTLE-WHILE-YOU-WORK
> (ask matt 'hunger)
2
> (ask matt 'work-done)
2
> (ask brian 'work)
WHISTLE-WHILE-YOU-WORK
> (ask brian 'hunger)
1
> (ask brian 'work-done)
3
> (ask worker 'work-done)
3
```

Như bạn có thể thấy, yêu cầu bất kỳ đối tượng công nhân nào làm việc sẽ tăng biến `work-done`. Ngược lại, mỗi công nhân có biến thể hiện `hunger` riêng, do đó khi Brian làm việc, Matt không bị đói.

Bạn có thể yêu cầu bất kỳ thể hiện nào giá trị của một biến lớp hoặc bạn có thể yêu cầu chính lớp đó. Đây là một ngoại lệ cho quy tắc thông thường rằng các thông điệp phải được gửi đến các thể hiện, không phải đến các lớp.

**Khởi tạo (Initialization)**

Đôi khi chúng ta muốn mọi thể hiện mới của một lớp nào đó thực hiện một số hoạt động ban đầu ngay sau khi nó được tạo. Ví dụ, giả sử chúng ta muốn duy trì một danh sách tất cả các đối tượng công nhân. Chúng ta sẽ tạo một biến lớp có tên là `all-workers` để giữ danh sách, nhưng chúng ta cũng phải đảm bảo rằng mỗi thể hiện mới được tạo sẽ tự thêm vào danh sách. Chúng ta làm điều này với một mệnh đề `initialize`:

```scheme
(define-class (worker)
  (instance-vars (hunger 0))
  (class-vars (all-workers '())
              (work-done 0))
  (initialize (set! all-workers (cons self all-workers)))
  (method (work)
    (set! hunger (1+ hunger))
    (set! work-done (1+ work-done))
    'whistle-while-you-work))
```

Phần thân của mệnh đề `initialize` được đánh giá khi đối tượng được tạo thể hiện. (Nhân tiện, đừng nhầm lẫn về hai từ dài đó đều bắt đầu bằng "I". Tạo thể hiện là quá trình tạo một thể hiện (nghĩa là một đối tượng cụ thể) của một lớp. Khởi tạo là một số hoạt động tùy chọn, dành riêng cho lớp mà đối tượng mới được tạo thể hiện có thể thực hiện.)

Nếu một lớp và lớp cha của nó đều có mệnh đề `initialize`, mệnh đề của lớp cha sẽ được đánh giá trước. Điều này có thể quan trọng nếu quá trình khởi tạo của lớp con tham chiếu đến trạng thái cục bộ được duy trì bởi các phương thức trong lớp cha.

**Các lớp nhận ra bất kỳ thông điệp nào**

Giả sử chúng ta muốn tạo một lớp đối tượng trả về giá trị của thông điệp trước đó mà chúng nhận được bất cứ khi nào bạn gửi cho chúng một thông điệp mới. Rõ ràng, mỗi đối tượng như vậy cần một biến thể hiện trong đó nó sẽ ghi nhớ thông điệp trước đó. Phần khó khăn là chúng ta muốn các đối tượng của lớp này chấp nhận bất kỳ thông điệp nào, không chỉ một vài thông điệp cụ thể. Đây là cách thực hiện:

```scheme
(define-class (echo-previous)
  (instance-vars (previous-message 'first-time))
  (default-method
    (let ((result previous-message))
      (set! previous-message message)
      result)))
```

Chúng ta đã sử dụng một mệnh đề `default-method`; phần thân của một mệnh đề `default-method` được đánh giá nếu một đối tượng nhận được một thông điệp mà nó không có phương thức. (Trong trường hợp này, đối tượng `echo-previous` không có bất kỳ phương thức thông thường nào, vì vậy mã `default-method` được thực thi cho bất kỳ thông điệp nào.)

Bên trong phần thân của mệnh đề `default-method`, biến `message` được liên kết với thông điệp đã nhận và biến `args` được liên kết với một danh sách các đối số bổ sung cho `ask`.

**Sử dụng phương thức của lớp cha một cách rõ ràng**

Trong ví dụ về tài khoản séc trước đó, chúng ta đã nói

```scheme
(define-class (checking-account init-balance)
  (parent (account init-balance))
  (method (write-check amount)
    (ask self 'withdraw (+ amount 0.10))))
```

Đừng quên cách hoạt động của điều này: Vì lớp `checking-account` có một lớp cha, bất kỳ thông điệp nào nó không hiểu đều được xử lý theo cùng cách mà lớp cha (`account`) sẽ xử lý chúng. Cụ thể, các đối tượng `account` có các phương thức `deposit` và `withdraw`.

Mặc dù một đối tượng `checking-account` yêu cầu chính nó rút một số tiền, chúng ta thực sự dự định thông điệp này được xử lý bởi một phương thức được định nghĩa trong lớp cha `account`. Không có vấn đề gì ở đây vì chính lớp `checking-account` không có phương thức `withdraw`.

Hãy tưởng tượng chúng ta muốn định nghĩa một lớp có một phương thức có cùng tên với một phương thức trong lớp cha của nó. Ngoài ra, chúng ta muốn phương thức của lớp con gọi phương thức của lớp cha có cùng tên. Ví dụ, chúng ta sẽ định nghĩa một lớp `TA` là một chuyên môn hóa của lớp `worker`. Sự khác biệt duy nhất là khi bạn yêu cầu một `TA` làm việc, anh ấy hoặc cô ấy sẽ trả về câu "Hãy để tôi giúp bạn với sơ đồ hộp và con trỏ đó" sau khi gọi phương thức `work` được định nghĩa trong lớp `worker`.

Chúng ta không thể chỉ nói `(ask self 'work)`, vì điều đó sẽ tham chiếu đến phương thức được định nghĩa trong lớp con. Nghĩa là, giả sử chúng ta nói:

```scheme
(define-class (TA)
  (parent (worker))
  (method (work)
    (ask self 'work)
    ;; SAI!
    '(Let me help you with that box and pointer diagram))
  (method (grade-exam) 'A+))
```

Khi chúng ta yêu cầu một `TA` làm việc, chúng ta hy vọng nhận được kết quả của việc yêu cầu một `worker` làm việc (tăng sự đói, tăng công việc đã hoàn thành) nhưng trả về một câu khác. Nhưng điều thực sự xảy ra là một đệ quy vô hạn. Vì `self` tham chiếu đến `TA` và `TA` có phương thức `work` riêng, đó là những gì được sử dụng.

(Trong ví dụ trước với tài khoản séc, `ask self` hoạt động vì tài khoản séc không có phương thức `withdraw` riêng.)

Thay vào đó, chúng ta cần một cách để truy Chắc chắn rồi, đây là phần dịch tiếp theo:

...cập phương thức được định nghĩa trong lớp cha (`worker`). Chúng ta có thể thực hiện điều này với `usual`:

```scheme
(define-class (TA)
  (parent (worker))
  (method (work)
    (usual 'work)
    '(Let me help you with that box and pointer diagram))
  (method (grade-exam) 'A+))
```

`Usual` lấy một hoặc nhiều đối số. Đối số đầu tiên là một thông điệp và các đối số còn lại là bất kỳ đối số bổ sung nào cần thiết. Gọi `usual` giống như nói `(ask self ...)` với cùng các đối số, ngoại trừ việc chỉ các phương thức được định nghĩa trong một lớp tổ tiên (cha, ông bà, v.v.) mới đủ điều kiện được sử dụng. Việc gọi `usual` từ một lớp không có lớp cha sẽ gây ra lỗi.

Bạn có thể nghĩ rằng `usual` là một cái tên buồn cười cho hàm này. Đây là ý tưởng đằng sau cái tên: Chúng ta đang nghĩ về các lớp con như là các chuyên môn hóa. Nghĩa là, lớp cha đại diện cho một số loại đối tượng rộng lớn và lớp con là một phiên bản chuyên biệt. (Hãy nghĩ về mối quan hệ của tài khoản séc với tài khoản nói chung.) Đối tượng lớp con làm hầu hết mọi thứ giống như lớp cha của nó. Lớp con có một số cách đặc biệt để xử lý một vài thông điệp, khác với cách thông thường (như lớp cha làm). Nhưng lớp con có thể quyết định rõ ràng để làm điều gì đó theo cách thông thường (giống lớp cha), thay vì theo cách chuyên biệt của riêng nó.

**Nhiều siêu lớp (Multiple Superclasses)**

Chúng ta có thể có các loại đối tượng kế thừa các phương thức từ nhiều hơn một loại. Chúng ta sẽ phát minh ra một lớp `singer` và sau đó tạo ra `singer-TAs` và `TA-singers`.

```scheme
(define-class (singer)
  (parent (worker))
  (method (sing) '(tra-la-la)))
```

```scheme
(define-class (singer-TA)
  (parent (singer) (TA)))
```

```scheme
(define-class (TA-singer)
  (parent (TA) (singer)))
```

```scheme
> (define Matt (instantiate singer-TA))
> (define Chris (instantiate TA-singer))
> (ask Matt 'grade-exam)
A+
> (ask Matt 'sing)
(TRA-LA-LA)
> (ask Matt 'work)
WHISTLE-WHILE-YOU-WORK
> (ask Chris 'work)
(LET ME HELP YOU WITH THAT BOX AND POINTER DIAGRAM)
```

Cả Matt và Chris đều có thể làm bất cứ điều gì một `TA` có thể làm, chẳng hạn như chấm điểm bài kiểm tra, và bất cứ điều gì một `singer` có thể làm, chẳng hạn như hát. Sự khác biệt duy nhất giữa chúng là cách chúng xử lý các thông điệp mà `TA` và `singer` xử lý khác nhau. Matt chủ yếu là một `singer`, vì vậy anh ấy phản hồi thông điệp `work` như một `singer` sẽ làm. Tuy nhiên, Chris chủ yếu là một `TA` và sử dụng phương thức `work` từ lớp `TA`.

Trong ví dụ trên, Matt đã sử dụng phương thức `work` từ lớp `worker`, được kế thừa thông qua hai cấp độ quan hệ cha con. (Lớp `worker` là lớp cha của `singer`, là lớp cha của `singer-TA`.) Trong một số tình huống, có thể tốt hơn nếu chọn một phương thức được kế thừa trực tiếp từ một lớp cha lựa chọn thứ hai (lớp `TA`) hơn là một phương thức được kế thừa từ một ông bà lựa chọn đầu tiên. Phần lớn sự phức tạp của các ngôn ngữ lập trình hướng đối tượng đương đại liên quan đến việc chỉ định các cách kiểm soát thứ tự kế thừa trong các tình huống như thế này.

**CS 61A**

**Phần A&S 3.2**

**Lập trình hướng đối tượng | Góc nhìn cấp thấp (Below the line view)**

Tài liệu này ghi lại hệ thống Lập trình hướng đối tượng cho CS 61A về mặt triển khai của nó trong Scheme. Nó giả định rằng bạn đã biết hệ thống này làm gì, tức là bạn đã đọc "Lập trình hướng đối tượng | Góc nhìn cấp cao". Ngoài ra, tài liệu này sẽ giả định kiến thức về cách triển khai truyền thông điệp và các biến trạng thái cục bộ trong Scheme, từ chương 2.3 và 3.1 của A&S. (Chương 3.2 từ A&S cũng sẽ hữu ích.)

Hầu hết công việc của hệ thống đối tượng được xử lý bởi dạng đặc biệt `define-class`. Khi bạn nhập một danh sách bắt đầu bằng ký hiệu `define-class`, Scheme sẽ dịch định nghĩa lớp của bạn thành mã Scheme để triển khai lớp đó. Phiên bản đã dịch này của định nghĩa lớp của bạn được viết hoàn toàn bằng `define`, `let`, `lambda`, `set!` và các hàm Scheme khác mà bạn đã biết.

Chúng ta sẽ tập trung vào việc triển khai ba ý tưởng kỹ thuật chính trong OOP: truyền thông điệp, trạng thái cục bộ và kế thừa.

**Truyền thông điệp**

Văn bản giới thiệu truyền thông điệp với ví dụ này từ Phần 2.3.3 (trang 141):

```scheme
(define (make-rectangular x y)
  (define (dispatch m)
    (cond ((eq? m 'real-part) x)
          ((eq? m 'imag-part) y)
          ((eq? m 'magnitude)
           (sqrt (+ (square x) (square y))))
          ((eq? m 'angle) (atan y x))
          (else
           (error "Unknown op -- MAKE-RECTANGULAR" m))))
  dispatch)
```

Trong ví dụ này, một đối tượng số phức được biểu diễn bằng một thủ tục phân phối. Thủ tục lấy một thông điệp làm đối số và trả về một số làm kết quả. Sau đó, trong Phần 3.1.1 (trang 173), văn bản sử dụng một tinh chỉnh của biểu diễn này, trong đó thủ tục phân phối trả về một thủ tục thay vì một số. Lý do chúng thực hiện thay đổi này là để cho phép các đối số bổ sung cho những gì chúng ta đang gọi là phương thức phản hồi một thông điệp. Người dùng nói

```scheme
((acc 'withdraw) 100)
```

Đánh giá biểu thức này yêu cầu một quá trình hai bước: Đầu tiên, thủ tục phân phối (có tên là `acc`) được gọi với thông điệp `withdraw` làm đối số. Thủ tục phân phối trả về thủ tục phương thức `withdraw` và thủ tục thứ hai đó được gọi với 100 làm đối số để thực hiện công việc thực tế. Tất cả hoạt động của một đối tượng đến từ việc gọi các thủ tục phương thức của nó; công việc duy nhất của chính đối tượng là trả về đúng thủ tục khi nó nhận được một thông điệp.

Bất kỳ hệ thống OOP nào sử dụng mô hình truyền thông điệp phải có một cơ chế cấp thấp để liên kết các phương thức với các thông điệp. Trong Scheme, với các thủ tục hạng nhất của nó, rất tự nhiên khi sử dụng một thủ tục phân phối làm cơ chế liên kết. Trong một số ngôn ngữ khác, đối tượng có thể được biểu diễn dưới dạng một mảng các cặp thông điệp-phương thức.

Nếu chúng ta đang xử lý các đối tượng như một kiểu dữ liệu trừu tượng, các chương trình sử dụng đối tượng không cần phải biết rằng chúng ta đang biểu diễn các đối tượng dưới dạng các thủ tục. Ký hiệu hai bước để gọi một phương thức vi phạm rào cản trừu tượng này. Để khắc phục điều này, chúng ta phát minh ra thủ tục `ask`:

```scheme
(define (ask object message . args)
  (let ((method (object message)))
    ; Bước 1: gọi thủ tục phân phối
    (if (method? method)
        (apply method args)
        ; Bước 2: gọi phương thức
        (error "No method" message (cadr method)))))
```

`Ask` thực hiện về cơ bản các bước tương tự như ký hiệu rõ ràng được sử dụng trong văn bản. Đầu tiên, nó gọi thủ tục phân phối (tức là chính đối tượng) với thông điệp làm đối số. Điều này sẽ trả về một phương thức (một thủ tục khác). Bước thứ hai là gọi thủ tục phương thức đó với bất kỳ đối số bổ sung nào đã được cung cấp cho `ask`.

Phần thân của `ask` trông phức tạp hơn phiên bản trước, nhưng hầu hết điều đó liên quan đến việc kiểm tra lỗi: Điều gì xảy ra nếu đối tượng không nhận ra thông điệp chúng ta gửi cho nó? Những chi tiết này không quan trọng lắm. `Ask` sử dụng hai tính năng của Scheme mà chúng ta chưa thảo luận trước đây:

Ký hiệu dấu chấm được sử dụng trong danh sách tham số hình thức của `ask` có nghĩa là nó chấp nhận bất kỳ số lượng đối số nào.

Hai phần đầu tiên được liên kết với các tham số hình thức `object` và `message`; tất cả các đối số còn lại (không hoặc nhiều hơn) được đưa vào một danh sách và được liên kết với tham số hình thức `args`.

Thủ tục `apply` lấy một thủ tục và một danh sách các đối số và áp dụng thủ tục cho các đối số. Lý do chúng ta cần nó ở đây là vì chúng ta không biết trước có bao nhiêu đối số sẽ được đưa ra cho phương thức; nếu chúng ta nói `(method args)`, chúng ta sẽ đưa cho phương thức một đối số, cụ thể là một danh sách.

Trong hệ thống OOP của chúng ta, bạn thường gửi thông điệp đến các thể hiện, nhưng bạn cũng có thể gửi một số thông điệp đến các lớp, cụ thể là các thông điệp để kiểm tra các biến lớp. Khi bạn gửi một thông điệp đến một lớp, giống như khi bạn gửi một thông điệp đến một thể hiện, bạn nhận lại một phương thức. Đó là lý do tại sao chúng ta có thể sử dụng `ask` với cả thể hiện và lớp. (Bản thân hệ thống OOP cũng gửi cho lớp một thông điệp `instantiate` khi bạn yêu cầu nó tạo một thể hiện mới.) Do đó, cả lớp và mỗi thể hiện đều được biểu diễn bằng một thủ tục phân phối. Cấu trúc tổng thể của một định nghĩa lớp trông giống như thế này:

```scheme
(define (class-dispatch-procedure class-message)
  (cond ((eq? class-message 'some-var-name) (lambda () (get-the-value)))
        (...)
        ((eq? class-message 'instantiate)
         (lambda (instantiation-var ...)
           (define (instance-dispatch-procedure instance-message)
             (cond ((eq? instance-message 'foo) (lambda ...))
                   (...)
                   (else (error "No method in instance"))))
           instance-dispatch-procedure))
        (else (error "No method in class"))))
```

(Xin lưu ý rằng đây không chính xác là những gì một lớp thực sự trông như thế nào. Trong phiên bản đơn giản hóa này, chúng ta đã bỏ qua nhiều chi tiết. Điểm quan trọng duy nhất ở đây là có hai thủ tục phân phối, một thủ tục bên trong thủ tục kia.) Trong mỗi thủ tục phân phối, có một `cond` với một mệnh đề cho mỗi thông điệp được phép. Biểu thức hệ quả của mỗi mệnh đề là một biểu thức `lambda` định nghĩa phương thức tương ứng. (Trong văn bản, các ví dụ thường sử dụng các thủ tục phương thức được đặt tên và các biểu thức hệ quả là tên thay vì `lambda`. Chúng tôi thấy cách này thuận tiện hơn, nhưng điều đó không thực sự quan trọng.)

**Trạng thái cục bộ**

Bạn đã học trong phần 3.1 rằng cách để cung cấp cho một thủ tục một biến trạng thái cục bộ là định nghĩa thủ tục đó bên trong một thủ tục khác thiết lập biến đó. Thủ tục bên ngoài đó có thể là thủ tục ngầm trong dạng đặc biệt `let`, như trong ví dụ này từ trang 171:

```scheme
(define new-withdraw
  (let ((balance 100))
    (lambda (amount)
      (if (>= balance amount)
          (begin (set! balance (- balance amount))
                 balance)
          "Insufficient funds"))))
```

Trong hệ thống OOP, có ba loại biến trạng thái cục bộ: biến lớp, biến thể hiện và biến khởi tạo. Mặc dù các biến khởi tạo chỉ là một loại biến thể hiện đặc biệt ở cấp cao, chúng được triển khai khác nhau. Đây là một cái nhìn đơn giản hóa khác về một định nghĩa lớp, lần này bỏ qua tất cả các phần truyền thông điệp và tập trung vào các biến:

```scheme
(define class-dispatch-procedure
  (LET ((CLASS-VAR1 VAL1)
        (CLASS-VAR2 VAL2) ...)
    (lambda (class-message)
      (cond ((eq? class-message 'class-var1) (lambda () class-var1))
            ...
            ((eq? class-message 'instantiate)
             (lambda (INSTANTIATION-VARIABLE1 ...)
               (LET ((INSTANCE-VAR1 VAL1)
                     (INSTANCE-VAR2 VAL2) ...)
                 (define (instance-dispatch-procedure instance-message)
                   (cond ((eq? instance-message 'instance-var1) (lambda () instance-var1))
                         ...
                         (else ...)))
                 instance-dispatch-procedure)))
            (else ...)))))
```

(Đây cũng là một phiên bản đơn giản hóa, nhưng nó có ý nghĩa hơn phiên bản trước.)

Hãy xem xét định nghĩa lớp `worker` từ phần trước:

```scheme
(define-class (worker)
  (instance-vars (hunger 0))
  (class-vars (work-done 0))
  (method (work)
    (set! hunger (1+ hunger))
    (set! work-done (1+ work-done))
    'whistle-while-you-work))
```

Khi Scheme dịch định nghĩa lớp này, nó sẽ tạo ra mã tương tự như sau:

```scheme
(define worker
  (let ((work-done 0))
    (lambda (class-message)
      (cond ((eq? class-message 'work-done) (lambda () work-done))
            ((eq? class-message 'instantiate)
             (lambda ()
               (let ((hunger 0))
                 (lambda (instance-message)
                   (cond ((eq? instance-message 'hunger) (lambda () hunger))
                         ((eq? instance-message 'work)
                          (lambda ()
                            (set! hunger (1+ hunger))
                            (set! work-done (1+ work-done))
                            'whistle-while-you-work))
                         (else (error "No method")))))))))))
```

(Lưu ý rằng chúng ta đã bỏ qua một số chi tiết như kiểm tra lỗi.)

Như bạn có thể thấy, các biến lớp được thiết lập trong `let` bên ngoài nhất. Các biến thể hiện được thiết lập trong `let` bên trong nhất. Các biến khởi tạo sẽ được thiết lập trong `let` bên trong, cùng với các biến thể hiện.

**Kế thừa**

Khi một lớp có một lớp cha, điều đó có nghĩa là lớp con kế thừa các phương thức của lớp cha. Về mặt triển khai, điều này có nghĩa là thủ tục phân phối của lớp con nên tham khảo thủ tục phân phối của lớp cha. Trong ví dụ về `singer-TA` từ phần trước, chúng ta có thể hình dung một cái gì đó như sau:

```scheme
(define singer-TA
  (lambda (class-message)
    (cond ((eq? class-message 'instantiate)
           (lambda ()
             (let ((singer-instance (singer 'instantiate))
                   (TA-instance (TA 'instantiate)))
               (lambda (instance-message)
                 (cond ((eq? instance-message 'sing) (singer-instance instance-message))
                       ((eq? instance-message 'grade-exam) (TA-instance instance-message))
                       ((eq? instance-message 'work) (singer-instance instance-message))
                       (else (error "No method")))))))))
```

(Đây là một phiên bản rất đơn giản hóa; phiên bản thực tế phức tạp hơn nhiều.)

Như bạn có thể thấy, thủ tục phân phối của `singer-TA` tạo ra hai thể hiện, một thể hiện của `singer` và một thể hiện của `TA`. Khi một thể hiện của `singer-TA` nhận được một thông điệp, nó sẽ chuyển thông điệp đó đến một trong hai thể hiện được tạo ra, tùy thuộc vào thông điệp.

**Kết luận**

Chúng ta đã thấy cách hệ thống OOP được triển khai trong Scheme. Điều quan trọng là hiểu rằng OOP không phải là một tính năng đặc biệt của Scheme; đó chỉ là một cách tổ chức mã. Chúng ta có thể đạt được kết quả tương tự bằng bất kỳ ngôn ngữ lập trình nào, miễn là ngôn ngữ đó có các thủ tục hạng nhất và trạng thái cục bộ.