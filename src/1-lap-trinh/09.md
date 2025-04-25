
- Recall the substitution model:

> To apply a compound procedure to arguments, evaluate the body of the procedure with each formal parameter replaced by the corresponding argument. (Section 3.2)

- This is no longer adequate once we allow assignment.
- Variables are no longer merely names for values; rather, a variable designates a “place” in which values can be stored.
- These places will be maintained in structures called _environments_.
- An environment is a sequence of _frames_. A frame is a table of _bindings_. A binding associates a variable name with one value.
- Each frame also has a pointer to its enclosing environment, unless it is considered to be global.
- The value of a variable with respect to an environment is the value given by the binding in the first frame of the environment that contains a binding for that variable.
- If no frame in the sequence specifies a binding for the variable, then the variable is _unbound_ in the environment.
- A binding _shadows_ another (of the same variable) if the other is in a frame that is further in the sequence.
- The environment determines the context in which an expression should be evaluated. An expression has no meaning otherwise.
- The global environment consists of a single frame, and it is implicitly used in interactions with the interpreter.

## 3.2.1 The Rules for Evaluation⁠

- To evaluate a combination:
1. Evaluate the subexpressions of the combination.
2. Apply the value of the operator subexpression to the values of the operand subexpressions.
- The environment model redefines the meaning of “apply.”
- A procedure is created by evaluating a λ-expression relative to a given environment.
- The resulting procedure object is a pair consisting of the text of the λ-expression and a pointer to the environment in which the procedure was created.
- To apply a procedure to arguments, create a new environment whose frame binds the parameters to the values of the arguments and whose enclosing environment is specified by the procedure.
- Then, within the new environment, evaluate the procedure body.
- `(lambda (x) (* x x))` evaluates a to pair: the parameters and the procedure body as one item, and a pointer to the global environment as the other.
- `(define square (lambda (x) (* x x)))` associates the symbol `square` with that procedure object in the global frame.
- Evaluating `(define var val)` creates a binding in the current environment frame to associate `var` with `val`.
- Evaluating `(set! var val)` locates the binding of `var` in the current environment (the first frame that has a binding for it) and changes the bound value to `val`.
- We use `define` for variables that are currently unbound, and `set!` for variables that are already bound.

## 3.2.2 Applying Simple Procedures⁠

- Let’s evaluate `(f 5)`, given the following procedures:

```
(define (square x) (* x x))
(define (sum-of-squares x y) (+ (square x) (square y)))
(define (f a) (sum-of-squares (+ a 1) (* a 2)))
```

- These definitions create bindings for `square`, `sum-of-squares`, and `f` in the global frame.
- To evaluate `(f 5)`, we create an environment E1E\_1 with a frame containing a single binding, associating `a` with `5`.
- In E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} we evaluate `(sum-of-squares (+ a 1) (* a 2))`.
- We must evaluate the subexpressions of this combination.
- We find the value associated with `sum-of-squares` not in E1E\_1 but in the global environment.
- Evaluating the operand subexpressions yields `6` and `10`.
- Now we create E2E\_2 with a frame containing two bindings: `x` is bound to `6`, and `y` is bound to `10`.
- In E2,E\_2\\htmlClass{math-punctuation}{\\text{,}} we evaluate `(+ (square x) (square y))`.
- The process continues recursively. We end up with `(+ 36 100)`, which evaluates to `136`.

## 3.2.3 Frames as the Repository of Local State⁠

- Now we can see how the environment model makes sense of assignment and local state.
- Consider the “withdrawal processor”:

```
(define (make-withdraw balance)
  (lambda (amount)
    (if (>= balance amount)
        (begin (set! balance (- balance amount))
               balance)
        "Insufficient funds")))
```

- This places a single binding in the global environment frame.
- Consider now `(define W1 (make-withdraw 100))`.

  - We set up E1E\_1 where `100` is bound to the formal parameter `balance`, and then we evaluate the body of `make-withdraw`.
  - This returns a lambda procedure whose environment is E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} and this is then bound to `W1` in the global frame.
- Now, we apply this procedure: `(W1 50)`.

  - We construct a frame in E2E\_2 that binds `amount` to `50`, and then we evaluate the body of `W1`.
  - The enclosing environment of E2E\_2 is E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} _not_ the global environment.
  - Evaluating the body results in the `set!` rebinding `balance` in E1E\_1 to the value `(- 100 50)`, which is `50`.
  - After calling `W1`, the environment E2E\_2 is irrelevant because nothing points to it.
  - Each call to `W1` creates a new environment to hold `amount`, but uses the same E1E\_1 (which holds `balance`).
- `(define W2 (make-withdraw 100))` creates another environment with a `balance` binding.

  - This is independent from E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} which is why the `W2` object and its local state is independent from `W1`.
  - On the other hand, `W1` and `W2` share the same code.

## 3.2.4 Internal Definitions⁠

- With block structure, we nested definitions using `define` to avoid exposing helper procedures.
- Internal definitions work according to the environmental model.
- When we apply a procedure that has internal definitions, there are `define` forms at the beginning of the body.
- We are in E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} so evaluating these adds bindings to the first frame of E1,E\_1\\htmlClass{math-punctuation}{\\text{,}} right after the arguments.
- When we apply the internal procedures, the formal parameter environment EnE\_n is created, and its enclosing environment is E1E\_1 because that was where the procedure was defined.
- This means each internal procedure has access to the arguments of the procedure they are defined within.
- The names of local procedures don’t interfere with names external to the enclosing procedure, due to E1.E\_1\\htmlClass{math-punctuation}{\\text{.}}





