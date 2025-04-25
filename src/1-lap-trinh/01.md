# Lập trình hướng chương trình
Bài viết sẽ tập trung vào liệt kê các yếu tố cơ bản cho một ngôn ngữ.
### Những yếu tố cơ bản cho một ngôn ngữ lập trình

Có ba cơ chế kết hợp các ý tưởng đơn giản để tạo thành những ý tưởng phức tạp trong mọi ngôn ngữ lập trình mạnh mẽ:

- Biểu thức nguyên thủy (primitive expressions)
- Phương thức kết hợp (means of combination)
- Phương thức abstraction (means of abstraction)

Lập trình xử lý các thủ tục (procedures) và dữ liệu (data). Thủ tục thao tác dữ liệu.

### Biểu thức (Expressions)

Về high-level, low-level, interpreter và compiler: Scheme là một trình phiên dịch (Interpreter) cũng tương tự như Python, là một ngôn ngữ lập trình bậc cao (high-level language). 

Lưu ý:
- High-level khác với low-level ở chỗ tương tác với hệ thống, low-level xử lý các phần sâu hơn trong hệ thống.
- Trình phiên dịch (Interpreter) khác với trình biên dịch (Compiler):
    - Nhiệm vụ chính của Compiler là thực hiện việc chuyển đổi toàn bộ mã nguồn trong một lần hay thậm chí nhiều lần như vậy, để trả về cho người dùng những mã được biên dịch sẵn sàng và phục vụ cho việc thực thi chương trình.
    - Không giống với Compiler, trước khi chuyển source code sang mã máy Interpreter cần phải chuyển mã nguồn sang một dạng gọi là Intermediate code. Tiếp đó, mỗi phần trong đoạn code sẽ được thông dịch và đưa vào thực thi một cách riêng biệt theo trình tự. 

Chúng ta sẽ có Interactive Shell hay REPL:

> In contrast to running a Python file, you can input commands in the REPL and see the results displayed immediately.

**REPL** hay *Read, Evaluate, Print, and Loop* thực hiện các thao tác sau: reads an expression (đọc biểu thức), evaluates it (đánh giá), prints the result (in kết quả), and repeats (lặp lại).

**Primitive expression là gì?** 

**Combinations là gì?** A combination denotes procedure application by a list of expressions inside parentheses. The first element is the operator ; the rest are the operands. Combinations can be nested: an operator or operand can itself be another combination.

**Môi trường**: Là nơi lưu giữ các cặp name–value (được định nghĩa thông qua `define` trong Scheme)

### Evaluating Combinations⁠

To evaluate a combination:

1.  Evaluate the subexpressions of the combination.
2.  Apply the procedure (value of leftmost subexpression, the operator) to the arguments (values of other subexpressions, the operands).

Before evaluating a combination, we must first evaluate each element inside it. Evaluation is recursive in nature---one of its steps is invoking itself.

(Cái này còn được gọi là _applicative order_, sẽ được nhắc tới trong các phần tiếp theo)

Để kết thúc đệ quy:

1.  Numbers evaluate to themselves.
2.  Built-in operators evaluate to machine instruction sequences.
3.  Names evaluate to the values associated with them in the environment.

Một số lưu ý khác:

- `(define x 3)` does not apply `define` to two arguments; this is not a combination.
-   Exceptions such as these are *special forms*. Each one has its own evaluation rule (Mình cũng chưa hiểu quá rõ về *dạng thức đặc biệt* trong Scheme)

### Compound Procedures
Thủ tục hay *Procedures* là một phương thức thực hiện abstraction mạnh mẽ. Về cơ bản nó sẽ trông thế này:

```scheme
(define (name formal-parameters) body)
;;; Ví dụ
(define (square x) (* x x))
```

### The Substitution Model for Procedure Application⁠

Với các Procedure Application⁠, chúng ta sẽ thực hiện _đánh giá_ theo hai trình tự khác nhau: _Applicative order_ và _normal order_.

Từ đầu bài đến giờ là sử dụng _Applicative order_, evaluate all the subexpressions first, then apply the procedure to the arguments.

Sử dụng _Normal order_ là khi _apply a compound procedure to arguments, evaluate the body of the procedure with each formal parameter replaced by the corresponding argument._ Only when it reaches primitive operators -> do combinations reduce to values.

```scheme
> (applic (f (+ 2 3) (- 15 6))) ; show applicative-order evaluation
(f (+ 2 3) (- 15 6)) ;; applic thì sẽ đi kiểu expression -> invocation

> (normal (f (+ 2 3) (- 15 6))) ; show normal-order evaluation
(f (+ 2 3) (- 15 6)) ----> ;; normal thì sẽ đi kiểu invocation cho đến khi đạt được đến primitive operators (+, -, *, /) thì mới bắt đầu tính
(+ (g (+ 2 3)) (- 15 6))
(g (+ 2 3)) ---->

; Same result, different process.
```

-> The rule is that if you’re doing functional programming, you get **the same answer regardless of order of evaluation**.

### Conditional Expressions and Predicates

**Case analysis**: testing each predicate (A predicate is an expression that evaluates to true or false). Chạy qua từng predicate một và thực hiện _evaluates to true or false_. Nếu _true_ thì sẽ bỏ qua các trường hợp còn lại.

Logical values can be combined with `and`, `or`, and `not`. The first two are special forms, not procedures, because they have short-circuiting behavior.

```scheme
(and (fun? learning) (boring? working))
(or (fun? learning) (boring? working))
(not lazy)
```

### Procedures as Black-Box Abstractions⁠

Each procedure in a program should accomplish an identifiable task that can be used as a module in defining other procedures.

(Module là _a section of code that is added in as a whole or is designed for easy reusability_)

When we use a procedure as a "black box," we are concerned with *what* it is doing but not *how* it is doing it. This is called procedural abstraction. Its purpose is to suppress detail. Bạn không cần phải biết cách người ta viết ra *thủ tục* đấy như thế nào và vẫn có thể sử dụng được. Tập trung vào *nó để làm gì* chứ không phải *cách nó vận hành*.

> A user should not need to know how the procedure is implemented in order to use it.

```scheme
(define (hello language) (say-hello-in language))
(hello Vietnamese)
```
Bạn không cần biết `say-hello-in` được viết như thế nào để có thể sử dụng

### Local names, Internal definitions và block structure

The choice of names for the procedure’s formal parameters should not matter to the user of the procedure. Consequentially, the parameter names must be local to the body of the procedure. The name of a formal parameter doesn't matter; it is a *bound variable*. The procedure *binds* its formal parameters (Hiểu cơ bản thì formal params được *bind* ở trong *thủ tục* mà nó được sử dụng => If a variable is not bound, it is free)

```scheme
;; Bạn không cần biết đến formal param ở đây là `language` để có thể sử dụng, nó sẽ chỉ ở trong *procedure* đấy thôi
(define (hello language) (say-hello-in language))
(hello Vietnamese)
```

-   The expression in which a binding exists is called the *scope* of the name. For parameters of a procedure, this is the body.
-   Using the same name for a bound variable and an existing free variable is called *capturing* the variable.
-   The names of the free variables *do* matter for the meaning of the procedure.
-   Putting a definition in the body of a procedure makes it local to that procedure. This nesting is called *block structure*.
-   Now we have two kinds of name isolation: formal parameters and internal definitions.
-   By internalizing auxiliary procedures, we can often eliminate bindings by allowing variables to remain free.
-   Scheme uses *lexical scoping*, meaning free variables in a procedure refer to bindings in enclosing procedure definitions.

Đọc thêm về [[02. Higher-order procedures & Recursion]]

### Về Function (Hàm)
```scheme
(define (square x) (* x x)) ;; defining a function
(square 5) ;; invoking the function
(square (+ 2 3)) ;; composition with defined functions
```
Thuật ngữ: 
- The *formal parameter* is the name of the argument (x); (Khi mà viết Function thì x chính là *formal parameter*)
- The *actual argument expression* is the expression used in the invocation ((+ 2 3)); 
- The *actual argument value* is the value of the argument in the invocation (5) (Ở đây cần phân biệt giữ expression và value, expression (là một argument của Function ấy) sẽ được thực hiện trước khi chạy Function)
-> The argument’s name comes from the function’s definition; the argument’s value comes from the invocation.

A function can have any number of arguments, including zero, but must have exactly one return value. -> It’s not a function unless you always get the same answer for the same arguments.
