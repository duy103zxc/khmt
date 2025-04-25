
### Mở đầu

We already know about procedural abstraction, which suppresses implementation details by *treating procedures as black boxes* -> Data abstractions allows us to isolate how a compound data object is used from the details of its actual representation.

That is, our programs should use data in such a way as to make no assumptions about the data that are not strictly necessary for performing the task at hand. (Section 2.1)

There is a concrete data representation behind the abstraction. The interface is made up of procedures called **constructors** and **selectors**.

### Data abstraction

If we are dealing with some particular type of data, we want to talk about it in terms of **its meaning**, **not** in terms of **how it happens to be represented** in the computer.

We want to add, subtract, multiply, divide, and test equality with our rational numbers.
Assume we have `(make-rat n d)`, `(number x)`, and `(denom x)`available as the constructor and selectors. This is **wishful thinking**, and it is a good technique.

Here is the implementation for addition:

```scheme
(define (add-rat x y)
(make-rat (+ (* (numer x) (denom y))
		   (* (numer y) (denom x)))
		(* (denom x) (denom y))))
```


```scheme
(define (make-rat n d) (cons n d))
(define (numer x) (car x))
(define (denom x) (cdr x))
```
The auxiliary functions like *numer* and *denom* are called **selectors** because they select one component of a multi-part *datum* (a single piece of information).


> Using data abstraction we can **change the implementation of the data type without affecting the programs** that use that data type.

You can write programs using *sentences* (Ở đây chỉ kiểu dữ liệu được cung cấp bởi CS61A) without knowing how they’re implemented. But it turns out that because of the way they are implemented, `first` and `butfirst` take $Θ(1)$ time, while `last` and `butlast` take $Θ(N)$ time. 
-> If you know that, your programs will be faster (Biết cách implementation giúp bạn hiểu bạn đang làm cái gì =D)

### Abstraction barrier

In general, the underlying idea of data abstraction is to identify for each type of data object a basic set of operations in terms of which all manipulations of data objects of that type will be expressed, and then to use only those operations in manipulating the data. (Section 2.1.2)

Details on the other side of an abstraction barrier are irrelevant to the code on this side.
This makes programs easier to maintain and modify.

Constraining the dependence on the representation to a few interface procedures helps us design programs as well as modify them, because it allows us to maintain the flexibility to consider alternate implementations. (Section 2.1.2)

### Pairs

To represent data types that have component parts, some way to *aggregate information*. In Lisp the most basic aggregation unit is the **pair** — two things combined to form a bigger thing. A pair is a concrete structure that we create with cons. 

We extract the parts of the pair with car and cdr. The constructor for pairs is CONS; the selectors are CAR and CDR.

![[Pasted image 20250129000952.png]]


```scheme
(define x (cons 1 2))

(car x)
→ 1

(cdr x)
→ 2
```

This is all the glue we need to *implement all sorts of complex data structures*. Data objects constructed from pairs are called *list-structured data*.

### Data aggregation doesn’t have to be primitive.

Về cơ bản là, Data aggregation thường có sẵn (*primitive*) trong các ngôn ngữ khác.

The point is that we can satisfy ourselves that this version of cons, car, and cdr works in the sense that if we construct a pair with this cons we can extract its two components with this car and cdr.

### Data là cái quái gì?⁠

Data is defined by some collection of *selectors* and *constructors*, together with specified conditions that these procedures must fulfill in order to be a valid representation.

For rationals, we have the following definition:

- We can construct a rational x with (make-rat n d).
- We can select the numerator with (numer x).
- We can select the denominator with (denom x).
- For all values of x, (/ (numer x) (denom x)) must equal n/d.

For pairs, it is even simpler: we need three operations, which we will call cons, car, and cdr, such that if z is (cons x y), then (car z) is x and (cdr z) is y.

### Message passing

The ability to manipulate procedures as objects automatically provides the ability to represent compound data. (Section 2.1.3)

This style of programming is often called message passing.
