<!-- 
**Topic:** Lazy Evaluator

**Reading:** Abelson, Sussman, Section 4.2, 4.3

#### Lazy Evaluator

To load the lazy metacircular evaluator, use the following command:

```scheme
(load "~cs61a/lib/lazy.scm")
```

**Streams and Pairs**

Streams require careful handling. The textbook defines the `pairs` procedure as follows:

```scheme
(define (pairs s t)
  (cons-stream
    (list (stream-car s) (stream-car t))
    (interleave
      (stream-map (lambda (x) (list (stream-car s) x))
                  (stream-cdr t))
      (pairs (stream-cdr s) (stream-cdr t)))))
```

In Exercise 3.68, Louis Reasoner suggests a simpler version:

```scheme
(define (pairs s t)
  (interleave
    (stream-map (lambda (x) (list (stream-car s) x)) t)
    (pairs (stream-cdr s) (stream-cdr t))))
```

However, this version fails because `interleave` evaluates its arguments immediately, leading to infinite recursion. The textbook's version uses `cons-stream`, which defers evaluation of its second argument until it is needed.

#### Explicit Delaying

To make Louis's version work, we can use explicit delaying:

```scheme
(define (interleave-delayed s1 delayed-s2)
  (if (stream-null? s1)
      (force delayed-s2)
      (cons-stream
        (stream-car s1)
        (interleave-delayed (force delayed-s2)
                            (delay (stream-cdr s1))))))

(define (pairs s t)
  (interleave-delayed
    (stream-map (lambda (x) (list (stream-car s) x)) t)
    (delay (pairs (stream-cdr s) (stream-cdr t)))))
```

While this works, it requires careful handling of delays, which defeats the purpose of abstraction.

#### Automatic Delay

Lazy evaluation delays all arguments automatically. This means Louis's `pairs` procedure would work without explicit delays if every argument were automatically delayed.

To implement this, we modify the evaluator:

1. Use `actual-value` instead of `eval` to find the procedure.
2. Use `list-of-delayed-values` instead of `list-of-values` to delay arguments.

In `apply`, we force arguments when calling a primitive procedure:

```scheme
(define (apply procedure arguments)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure procedure (map force arguments)))
        ...))
```

#### Delay and Force Implementation

Instead of using Scheme's built-in `delay`, we define `delay-it` and `force-it` for the metacircular evaluator:

```scheme
(define (delay-it exp env)
  (list 'thunk exp env))

(define (force-it obj)
  (if (thunk? obj)
      (let ((exp (thunk-exp obj))
            (env (thunk-env obj)))
        (set-car! obj (actual-value exp env))
        (set-car! (cdr obj) 'evaluated)
        (car obj))
      obj))
```

#### Memoization

Memoization avoids redundant computations by storing results of previously evaluated thunks:

```scheme
(define (delay-it exp env)
  (let ((thunk (list 'thunk exp env)))
    (lambda ()
      (if (eq? (car thunk) 'thunk)
          (let ((result (actual-value (cadr thunk) (caddr thunk))))
            (set-car! thunk 'evaluated)
            (set-car! (cdr thunk) result)
            result)
          (cadr thunk)))))
``` -->

#### Nondeterministic Evaluator

To load the nondeterministic metacircular evaluator, use:

```scheme
(load "~cs61a/lib/ambeval.scm")
```

**Solution Spaces and Backtracking**

Many problems involve finding solutions that satisfy certain conditions. The solution space can be represented as a stream, and `stream-filter` can be used to select valid solutions.

**Amb and Require**

The `amb` special form represents multiple possible values. The `require` procedure causes a failure if a condition is not met:

```scheme
(define (require condition)
  (if (not condition) (amb)))
```

**Recursive Amb**

`amb` can be used recursively to generate infinite solution spaces:

```scheme
(define (an-integer-between from to)
  (if (> from to)
      (amb)
      (amb from (an-integer-between (+ from 1) to))))
```

**Continuations**

Continuations allow computations to be paused and resumed. In the nondeterministic evaluator, `eval` is given two continuations: one for success and one for failure.

**Implementing Continuations**

In `eval-if`, we handle continuations as follows:

```scheme
(define (eval-if exp env succeed fail)
  (eval (if-predicate exp)
        env
        (lambda (pred-value fail2)
          (if (true? pred-value)
              (eval (if-consequent exp) env succeed fail2)
              (eval (if-alternative exp) env succeed fail2)))
        fail))
```

For `eval-amb`, we pass a new failure continuation:

```scheme
(define (eval-amb exp env succeed fail)
  (if (null? (cdr exp))
      (fail)
      (eval (cadr exp)
            env
            succeed
            (lambda ()
              (eval-amb (cons 'amb (cddr exp))
                        env
                        succeed
                        fail)))))
```

**Mutation and Continuations**

Mutation (e.g., `set!`) must handle continuations to ensure correct behavior when computations fail.

**Note:** The second part of Programming Project 4 is due this week.

---

This revised note should be clearer and more concise, making it easier to understand the concepts of lazy evaluation and the nondeterministic evaluator.