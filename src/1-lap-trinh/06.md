
- Data abstraction lets us write programs that work independently of the chosen representation for data objects. This helps control complexity.
- But it’s not powerful enough: sometimes we want not just an abstracted underlying representation, but multiple representations.
- In addition to horizontal abstraction barriers, separating high-level from low-level, we need vertical abstraction barriers, allowing multiple design choices to coexist.
- We’ll do this by constructing _generic procedures_, which can operate on data that may be represented in more than one way.

## 2.4.1 Representations for Complex Numbers⁠

- We can represent a complex number z=x+yi=reiθz=x+yi=re^{iθ} as a list in two ways: in rectangular form (x,y)(x,y) or in polar form (r,θ).(r,θ)\\htmlClass{math-punctuation}{\\text{.}}
- Rectangular form is convenient for addition and substraction, while polar form is convenient for multiplication and division.
- Our goal is to implement all the operations to work with either representation:

```
(define (add-complex z1 z2)
  (make-from-real-imag (+ (real-part z1) (real-part z2))
                       (+ (imag-part z1) (imag-part z2))))
(define (sub-complex z1 z2)
  (make-from-real-imag (- (real-part z1) (real-part z2))
                       (- (imag-part z1) (imag-part z2))))
(define (mul-complex z1 z2)
  (make-from-mag-ang (* (magnitude z1) (magnitude z2))
                     (+ (angle z1) (angle z2))))
(define (div-complex z1 z2)
  (make-from-mag-ang (/ (magnitude z1) (magnitude z2))
                     (- (angle z1) (angle z2))))
```

- We can implement the constructors and selectors to use rectangular form or polar form, but how do we allow both?

## 2.4.2 Tagged Data⁠

- One way to view data abstraction is as the “principle of least commitment.” We waited until the last minute to choose a concrete representation, retaining maximum flexibility.
- We can take it even further and avoid committing to a single representation at all.
- To do this, we need some way of distinguishing between representations. We will do this with a _type tag_ `'rectangular` or `'polar`.
- Each generic selector will strip off the type tag and use case analysis to pass the untyped data object to the appropriate specific selector.

```
(define (attach-tag type-tag contents)
  (cons type-tag contents))
(define (type-tag datum)
  (if (pair? datum)
      (car datum)
      (error "bad tagged datum" datum)))
(define (contents datum)
  (if (pair? datum)
      (cdr datum)
      (error "bad tagged datum" datum)))

(define (rectangular? z)
  (eq? (type-tag z) 'rectangular))
(define (polar? z)
  (eq? (type-tag z) 'polar))
```

- For example, here is how we implement the generic `real-part`:

```
(define (real-part z)
  (cond ((rectangular? z)
         (real-part-rectangular (contents z)))
        ((polar? z)
         (real-part-polar (contents z)))
        (else (error "unknown type" z))))
```

## 2.4.3 Data-Directed Programming and Additivity⁠

- The strategy of calling a procedure based on the type tag is called _dispatching on type_.
- The implementation in § 2.4.2 has two significant weaknesses:

1. The generic procedures must know about all the representations.
2. We must guarantee that no two procedures have the same name.
- The underlying issue is that the technique is not _additive_.
- The solution is to use a technique known as _data-directed programming_.
- Imagine a table with operations on one axis and types on the other axis:

PolarRectangularReal part`real-part-polar``real-part-rectangular`Imaginary part`imag-part-polar``imag-part-rectangular`Magnitude`magntiude-polar``magntiude-rectangular`Angle`angle-polar``angle-rectangular`

- Before, we implemented each generic procedure using case analysis. Now, we will write a single procedure that looks up the operation name and argument type in a table.
- Assume we have the procedure `(put op type item)` which installs `item` in the the table, and `(get op type)` which retrieves it. (We will implement them in § 3.3.3.)
- Then, we can define a collection of procedures, or _package_, to install the each representation of complex numbers. For example, the rectangular representation:

```
(define (install-rectangular-package)
  ;; Internal procedures
  (define (real-part z) (car z))
  (define (imag-part z) (cdr z))
  (define (make-from-real-imag x y) (cons x y))
  (define (magnitude z)
    (sqrt (+ (square (real-part z))
             (square (imag-part z)))))
  (define (angle z)
    (atan (imag-part z) (real-part z)))
  (define (make-from-mag-ang r a)
    (cons (* r (cos a)) (* r (sin a))))

  ;; Interface to the rest of the system
  (define (tag x) (attach-tag 'rectangular x))
  (put 'real-part '(rectangular) real-part)
  (put 'imag-part '(rectangular) imag-part)
  (put 'magnitude '(rectangular) magnitude)
  (put 'angle '(rectangular) angle)
  (put 'make-from-real-imag 'rectangular
       (lambda (x y) (tag (make-from-real-imag x y))))
  (put 'make-from-mag-ang 'rectangular
       (lambda (r a) (tag (make-from-mag-ang r a))))
  'done)
```

- Next, we need a way to look up a procedure in the table by operation and arguments:

```
(define (apply-generic op . args)
  (let ((type-tags (map type-tag args)))
    (let ((proc (get op type-tags)))
      (if proc
          (apply proc (map contents args))
          (error "no method for these types" (list op type-tags))))))
```

- Using `apply-generic`, we can define our generic selectors as follows:

```
(define (real-part z) (apply-generic 'real-part z))
(define (imag-part z) (apply-generic 'imag-part z))
(define (magnitude z) (apply-generic 'magnitude z))
(define (angle z) (apply-generic 'angle z))
```

### Message passing

- Data-directed programming deals explicitly with the operation–type table.
- In § 2.4.2, we made “smart operations” that dispatch based on type. This effectively decomposes the table into rows.
- Another approach is to make “smart data objects” that dispatch based on operation. This effectively decomposes the table into columns.
- This is called _message-passing_, and we can accomplish it in Scheme using closures.
- For example, here is how we would implement the rectangular representation:

```
(define (make-from-real-imag x y)
  (define (dispatch op)
    (cond ((eq? op 'real-part) x)
          ((eq? op 'imag-part) y)
          ((eq? op 'magnitude)
           (sqrt (+ (square x) (square y))))
          ((eq? op 'angle) (atan y x))
          (else (error "unknown op" op))))
  dispatch)
```

- Message passing can be a powerful tool for structuring simulation programs.





