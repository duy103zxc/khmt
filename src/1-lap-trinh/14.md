## 4.2 Variations on a Scheme—Lazy Evaluation⁠

- We can experiment with different language design just by modifying the evaluator.
- This is often how new languages are invented. It’s easy to iterate on a high level evaluator, and it also allows stealing features from the underlying language.

## 4.2.1 Normal Order and Applicative Order⁠

- In § 1.1.5, we noted that Scheme is an _applicative-order_ language.
- _Normal-order_ languages use _lazy evaluation_ to delay evaluation as long as possible.
- Consider this procedure:

```
(define (try a b) (if = a 0) 1 b)
```

- In Scheme, `(try 0 (/ 1 0))` causes a division-by-zero error. With lazy evaluation, it does not because the value of `b` is never needed.
- In a lazy language, we can implement `if` as an ordinary procedure.

## 4.2.2 An Interpreter with Lazy Evaluation⁠

- In this section, we will modify the interpreter to support lazy evaluation.
- The lazy evaluator _delays_ certain arguments, transforming them into _thunks_.
- The expression in a thunk does not get evaluated until the thunk is _forced_.
- A thunk gets forced when its value is needed:
    - when passed to a primitive procedure;
    - when it is the predicate of conditional;
    - when it is an operator about to be applied as a procedure.
- This is similar to streams, but uniform and automatic throughout the language.
- For efficiency, we’ll make our interpreter _memoize_ thunks.
    - This is called _call-by-need_, as opposed to non-memoized _call-by-name_.
    - It raises subtle and confusing issues in the presence of assignments.

Highlights

> The word _thunk_ was invented by an informal working group that was discussing the implementation of call-by-name in Algol 60. They observed that most of the analysis of (“thinking about”) the expression could be done at compile time; thus, at run time, the expression would already have been “thunk” about. (Footnote 4.34)

### Modifying the evaluator

- The main change required is the procedure application logic in `eval` and `apply`.
- The `application?` clause of `eval` becomes:

```
((application? exp)
 (apply (actual-value (operator exp) env)
        (operands exp)
        env))
```

- Whenever we need the actual value of an expression, we force in addition to evaluating:

```
(define (actual-value exp env) (force-it (eval exp env)))
```

- We change `apply` to take `env`, and use `list-of-arg-values` and `list-of-delayed-args`:

```
(define (apply procedure arguments env)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure
          procedure
          (list-of-arg-values arguments env)))  ; changed
        ((compound-procedure? procedure)
         (eval-sequence
          (procedure-body procedure)
          (extend-environment
           (procedure-parameters procedure)
           (list-of-delayed-args arguments env) ; changed
           (procedure-environment procedure))))
        (else (error "Unknown procedure type: APPLY" procedure))))
```

- We also need to change `eval-if` to use `actual-value` on the predicate:

```
(define (eval-if exp env)
  (if (true? (actual-value (if-predicate exp) env))
      (eval (if-consequent exp) env)
      (eval (if-alternative exp) env)))
```

### Representing thunks

- To force a thunk, we evaluate it in its environment. We use `actual-value` instead of `eval` so that it recursively forces if the result is another thunk:

```
(define (force-it obj)
  (if (thunk? obj)
      (actual-value (thunk-exp obj) (thunk-env obj))
      obj))
```

- We can represent thunks simply by a list containing the expression and environment:

```
(define (delay-it exp env) (list 'thunk exp env))
(define (thunk? obj) (tagged-list? obj 'thunk))
(define (thunk-exp thunk) (cadr thunk))
(define (thunk-env thunk) (caddr thunk))
```

- To memoize thunks, `force-it` becomes a bit more complicated.

## 4.2.3 Streams as Lazy Lists⁠

- Before introducing lazy evaluation, we implemented streams as delayed lists.
- We can instead formulate streams as _lazy_ lists. There are two advantages:
    1. No need for special forms `delay` and `cons-stream`.
    2. No need for separate list and stream operations.
- All we need to do is make `cons` lazy, either by introducing non-strict primitives or by defining `cons`, `car`, and `cdr` as compound procedures.
- Now we can write code without distinguishing normal lists from infinite ones:

```
(define ones (cons 1 ones))
(define (scale-list items factor) (map (lambda (x) (* x factor)) items))
```

- These lazy lists are even lazier than our original streams, since the `car` is delayed too.
- This also eliminates the problems we had earlier around having to explicitly delay and force some arguments when computing integrals.