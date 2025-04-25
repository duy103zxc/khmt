
## 4.1 The Metacircular Evaluator⁠

- We will implement a Lisp evaluator as a Lisp program.
- The metacircular evaluator implements the environment model of evaluation:
    1. To evaluate a combination, evaluate subexpressions and then apply the operator subexpression to the operand subexpressions.
    2. To apply a procedure to arguments, evaluate the body of the procedure in a new environment. To construct the new environment, extend the environment part of the procedure object by a frame in which the formal parameters of the procedure are bound to the arguments to which the procedure is applied.
- This embodies the interplay between two critical procedures, `eval` and `apply`.

## 4.1.1 The Core of the Evaluator⁠

### `Eval`

- `eval` classifies an expression and directs its evaluation in an environment.
- We use _abstract syntax_ to avoid committing to a particular syntax in the evaluator.

```
(define (eval exp env)
  (cond ((self-evaluating? exp) exp)
        ((variable? exp) (lookup-variable-value exp env))
        ((quoted? exp) (text-of-quotation exp))
        ((assignment? exp) (eval-assignment exp env))
        ((definition? exp) (eval-definition exp env))
        ((if? exp) (eval-if exp env))
        ((lambda? exp) (make-procedure (lambda-parameters exp)
                                       (lambda-body exp)
                                       env))
        ((begin? exp) (eval-sequence (begin-actions exp) env))
        ((cond? exp) (eval (cond->if exp) env))
        ((application? exp)
         (apply (eval (operator exp) env)
                (list-of-values (operands exp) env)))
        (else (error "Unknown expression type: EVAL" exp))))
```

### `Apply`

- `apply` classifies a procedure and directs its application to a list of arguments.
- If compound, it evaluates the procedure body in an extended environment.

```
(define (apply procedure arguments)
  (cond ((primitive-procedure? procedure)
         (apply-primitive-procedure procedure arguments))
        ((compound-procedure? procedure)
         (eval-sequence
          (procedure-body procedure)
          (extend-environment
           (procedure-parameters procedure)
           arguments
           (procedure-environment procedure))))
        (else (error "Unknown procedure type: APPLY" procedure))))
```

### Procedure arguments

### Conditionals

### Sequences

### Assignments and definitions

## 4.1.2 Representing Expressions⁠

- The evaluator is reminiscent of the symbolic differentiator: both make recursive computations on compound expressions, and both use data abstraction.
- The syntax of the language is determined solely by procedures that classify and extract pieces of expressions. For example:

```
(define (quoted? exp) (tagged-list? exp 'quote))
(define (text-of-quotation exp) (cadr exp))
(define (tagged-list? exp tag)
  (if (pair? exp)
      (eq? (car exp) tag)
      false))
```

### Derived expressions

- Some special forms can be defined in terms of others.
- For example, we can reduce `cond` to an `if` expression:

```
(define (cond? exp) (tagged-list? exp 'cond))
(define (cond-clauses exp) (cdr exp))
(define (cond-else-clause? clause) (eq? (cond-predicate clause) 'else))
(define (cond-predicate clause) (car clause))
(define (cond-actions clause) (cdr clause))
(define (cond->if exp) (expand-clauses (cond-clauses exp)))

(define (expand-clauses clauses)
  (if (null? clauses)
      'false ; no else clause
      (let ((first (car clauses))
            (rest (cdr clauses)))
        (if (cond-else-clause? first)
            (if (null? rest)
                (sequence->exp (cond-actions first))
                (error "ELSE clause isn't last: COND->IF" clauses))
            (make-if (cond-predicate first)
                     (sequence->exp (cond-actions first))
                     (expand-clauses rest))))))
```

- Practical Lisp systems allow the user to define new derived expressions by syntactic transformation. These are called _macros_.
- There is much research on avoiding name-conflict problems in macro definition languages.

## 4.1.3 Evaluator Data Structures⁠

### Testing of predicates

- Anything other than `false` is considered “truthy.”

```
(define (true? x) (not (eq? x false)))
(define (false? x) (eq? x false))
```

### Representing procedures

- `(apply-primitive-procedure proc args)` applies a primitive procedure to `args`.
- `(primitive-procedure? proc)` tests whether `proc` is a primitive procedure.
- Compound procedures are represented by the following data structure:

```
(define (make-procedure parameters body env)
  (list 'procedure parameters body env))
(define (compound-procedure? p) (tagged-list? p 'procedure))
(define (procedure-parameters p) (cadr p)) (define (procedure-body p) (caddr p))
(define (procedure-environment p) (cadddr p))
```

### Operations on environments

- `(lookup-variable-value var env)` returns the value bound to a variable.
- `(extend-environment variables values base-env)` returns a new environment extended with a frame containing the given bindings.
- `(define-variable! var value env)` adds a binding to the first frame of `env`.
- `(set-variable-value! var value env)` changes a binding in `env`.
- Here is a partial implementation:

```
(define (enclosing-environment env) (cdr env))
(define (first-frame env) (car env))
(define the-empty-environment '())

(define (make-frame variables values) (cons variables values))
(define (frame-variables frame) (car frame))
(define (frame-values frame) (cdr frame))

(define (extend-environment vars vals base-env)
  (if (= (length vars) (length vals))
      (cons (make-frame vars vals) base-env)
      (if (< (length vars) (length vals))
          (error "Too many arguments supplied" vars vals)
          (error "Too few arguments supplied" vars vals))))

(define (lookup-variable-value var env)
  (define (env-loop env)
    (define (scan vars vals)
      (cond ((null? vars) (env-loop (enclosing-environment env)))
            ((eq? var (car vars)) (car vals))
            (else (scan (cdr vars) (cdr vals)))))
    (if (eq? env the-empty-environment)
        (error "Unbound variable" var)
        (let ((frame (first-frame env)))
          (scan (frame-variables frame) (frame-values frame)))))
  (env-loop env))
```

- This representation is simple, but inefficient, since the evaluator may have to search through many frames to find a binding. This approach is called _deep binding_.

## 4.1.4 Running the Evaluator as a Program⁠

- We can run our evaluator as a program: Lisp within Lisp.
- The evaluator ultimately reduces expressions to applications of primitive procedures, so we need the evaluator to map these to the underlying Lisp’s primitive procedures.
- We set up a global environment mapping primitive procedures, `true`, and `false`:

```
(define (setup-environment)
  (let ((initial-env
         (extend-environment (primitive-procedure-names)
                             (primitive-procedure-objects)
                             the-empty-environment)))
    (define-variable! 'true true initial-env)
    (define-variable! 'false false initial-env)
    initial-env))
(define the-global-environment (setup-environment))
```

- We define a list of primitive procedures `car`, `cdr`, `cons`, and `null?`:

```
(define (primitive-procedure? proc) (tagged-list? proc 'primitive))
(define (primitive-implementation proc) (cadr proc))

(define primitive-procedures
  (list (list 'car car)
        (list 'cdr cdr)
        (list 'cons cons)
        (list 'null? null?)
        more-primitives))
(define (primitive-procedure-names)
  (map car primitive-procedures))
(define (primitive-procedure-objects)
  (map (lambda (proc) (list 'primitive (cadr proc)))
       primitive-procedures))
```

- Here, `apply-in-underlying-scheme` refers to the built-in `apply`, not the one we defined:

```
(define (apply-primitive-procedure proc args)
  (apply-in-underlying-scheme (primitive-implementation proc) args))
```

- Finally, we make a simple _driver loop_, or REPL:

```
(define input-prompt ";;; M-Eval input:")
(define output-prompt ";;; M-Eval value:")
(define (driver-loop)
  (prompt-for-input input-prompt)
  (let ((input (read)))
    (let ((output (eval input the-global-environment)))
      (announce-output output-prompt)
      (user-print output)))
  (driver-loop))

(define (prompt-for-input string)
  (newline) (newline) (display string) (newline))
(define (announce-output string)
  (newline) (display string) (newline))
(define (user-print object)
  (if (compound-procedure? object)
      (display (list 'compound-procedure
                     (procedure-parameters object)
                     (procedure-body object)
                     'procedure-env))
      (display object)))
```

## 4.1.5 Data as Programs⁠

- A program can be viewed as a description of an abstract machine.
- The evaluator is a machine that emulates another machine given its description. In other words, the evaluator is a _universal machine_.
- Deep idea: any evaluator can emulate any other. This gets to the heart of _computability_.
- Just as the evaluator can emulate any Lisp-described machine, a _universal Turing machine_ can emulate any other Turing machine.
- The existence of a universal machine is a deep and wonderful property of computation.
- The user’s programs are the evaluator’s data. Lisp takes advantage of this and provides a primitive `eval` procedure for evaluating data as programs.

Highlights

> We can regard the evaluator as a very special machine that takes as input a description of a machine. Given this input, the evaluator configures itself to emulate the machine described. …
> 
> From this perspective, our evaluator is seen to be a _universal machine_. It mimics other machines when these are described as Lisp programs. This is striking. Try to imagine an analogous evaluator for electrical circuits. This would be a circuit that takes as input a signal encoding the plans for some other circuit, such as a filter. Given this input, the circuit evaluator would then behave like a filter with the same description. Such a universal electrical circuit is almost unimaginably complex. It is remarkable that the program evaluator is a rather simple program. (Section 4.1.5)
> 
> Some people find it counterintuitive that an evaluator, which is implemented by a relatively simple procedure, can emulate programs that are more complex than the evaluator itself. The existence of a universal evaluator machine is a deep and wonderful property of computation. _Recursion theory_, a branch of mathematical logic, is concerned with logical limits of computation. Douglas Hofstadter’s beautiful book Gödel, Escher, Bach (1979) explores some of these ideas. (Footnote 4.20)

## 4.1.6 Internal Definitions⁠

- Global definitions have _sequential scoping_: they are defined one at a time.
- Internal definitions should have _simultaneous scoping_, as if defined all at once.
- The Scheme standard requires internal definitions to come first in the body and not use each other during evaluation. Although this restriction makes sequential and simultaneous scoping equivalent, simultaneous scoping makes compiler optimization easier.
- To achieve simultaneous scoping, we “scan out” internal definitions:

```
(lambda vars
  (define u e1)
  (define v e2)
  e3)
```

- Transforming them into a `let` with assignments:

```
(lambda vars
  (let ((u '*unassigned*)
        (v '*unassigned*))
    (set! u e1)
    (set! v e2) e3))
```

- Here, `'*unassigned*` is a special symbol causing an error upon variable lookup.

## 4.1.7 Separating Syntactic Analysis from Execution⁠

- Our evaluator is inefficient because it interleaves syntactic analysis with execution.
- For example, given a recursive procedure:

```
(define (factorial n)
  (if (= n 1) 1 (* (factorial (- n 1)) n)))
```

- When evaluating `(factorial 4)`, on all four recursive calls the evaluator must determine anew that the body is an `if` expression by reaching the `if?` test.
- We can arrange the evaluator to analyze syntax only once by splitting `eval` into two parts:
    - `(analyze exp)` performs syntactic analysis and returns an _execution procedure_.
    - `((analyze exp) env)` completes the evaluation.
- `analyze` is similar to the original `eval`, except it only performs analysis, not full evaluation:

```
(define (analyze exp)
  (cond ((self-evaluating? exp) (analyze-self-evaluating exp))
        ((quoted? exp) (analyze-quoted exp))
        ((variable? exp) (analyze-variable exp))
        ((assignment? exp) (analyze-assignment exp))
        ((definition? exp) (analyze-definition exp))
        ((if? exp) (analyze-if exp))
        ((lambda? exp) (analyze-lambda exp))
        ((begin? exp) (analyze-sequence (begin-actions exp)))
        ((cond? exp) (analyze (cond->if exp)))
        ((application? exp) (analyze-application exp))
        (else (error "Unknown expression type: ANALYZE" exp))))
```

- Here is one of the helper procedures, `analyze-lambda`. It provides a major gain in efficiency because we only analyze the lambda body once, no matter how many times the procedure is called.

```
(define (analyze-lambda exp)
  (let ((vars (lambda-parameters exp))
        (bproc (analyze-sequence (lambda-body exp))))
    (lambda (env) (make-procedure vars bproc env))))
```