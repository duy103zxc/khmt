
## 3.5 Streams⁠

- We’ve used assignment as a powerful tool and dealt with some of the complex problems it raises.
- Now we will consider another approach to modeling state, using data structures called _streams_.
- We want to avoid identifying time in the computer with time in the modeled world.
- If x is a function of time x(t), we can think of the identity x as a history of values (and these don’t change).
- With time measured in discrete steps, we can model a time function as a sequence of values.
- Stream processing allows us to model state without assignments.

## 3.5.1 Streams Are Delayed Lists⁠

- If we represent streams as lists, we get elegance at the price of severe inefficiency (time _and_ space).
- Consider adding all the primes in an interval. Using `filter` and `reduce`, we waste a lot of space storing lists.
- Streams are lazy lists. They are the clever idea of using sequence manipulations without incurring the cost.
- We only construct an item of the stream when it is needed.
- We have `cons-stream`, `stream-car`, `stream-cdr`, `the-empty-stream`, and `stream-null?`.
- The `cons-stream` procedure must not evaluate its second argument until it is accessed by `stream-cdr`.
- To implement streams, we will use _promises_. `(delay exp)` does not evaluate the argument but returns a promise. `(force promise)` evaluates a promise and returns the value.
- `(cons-stream a b)` is a special form equivalent to `(cons a (delay b))`.

### The stream implementation in action

> In general, we can think of delayed evaluation as “demand-driven” programming, we herby each stage in the stream process is activated only enough to satisfy the next stage. (Section 3.5.1)

### Implementing `delay` and `force`

- Promises are quite straightforward to implement.
- `(delay exp)` is syntactic sugar for `(lambda () exp)`.
- `force` simply calls the procedure. We can optimize it by saving the result and not calling the procedure a second time.
- The promise stored in the `cdr` of the stream is also known as a _thunk_.

## 3.5.2 Infinite Streams⁠

- With lazy sequences, we can manipulate infinitely long streams!
- We can define Fibonacci sequence explicitly with a generator:

```
(define (fibgen a b) (cons-stream a (fibgen b (+ a b))))
(define fibs (fibgen 0 1))
```

- As long as we don’t try to display the whole sequence, we will never get stuck in an infinite loop.
- We can also create an infinite stream of primes.

### Defining streams implicitly

- Instead of using a generator procedure, we can define infinite streams implicitly, taking advantage of the laziness.

```
(define fibs
  (cons-stream
   0
   (cons-stream 1 (stream-map + fibs (stream-cdr fibs)))))
(define pot
  (cons-stream 1 (stream-map (lambda (x) (* x 2)) pot)))
```

- This implicit technique is known as _corecursion_. Recursion works backward towards a base case, but corecursion works from the base and creates more data in terms of itself.

## 3.5.3 Exploiting the Stream Paradigm⁠

- Streams can provide many of the benefits of local state and assignment while avoiding some of the theoretical tangles.
- Using streams allows us to have different module boundaries.
- We can focus on the whole stream/series/signal rather than on individual values.

### Formulating iterations as stream processes

- Before, we made iterative processes by updating state variables in recursive calls.
- To compute the square root, we improved a guess until the values didn’t change very much.
- We can make a stream that converges on the square root of `x`, and a stream to approximate π.
- One neat thing we can do with these streams is use sequence accelerators, such as Euler’s transform.

### Infinite streams of pairs

- Previously, we handled traditional nested loops as processes defined on sequences of pairs.
- We can find all pairs (i,j) with i≤j such that i+j is prime like this:

```
(stream-filter
 (lambda (pair) (prime? (+ (car pair) (cadr pair))))
 int-pairs)
```

- We need some way of producing a stream of all integer pairs.
- More generally, we can combined two streams to get a two-dimensional grid of pairs, and we want to reduce this to a one-dimensional stream.
- One way to do this is to use `interleave` in the recursive definition, in order to handle infinite streams.

### Streams as signals

- We can use streams to model signal-processing systems.
- For example, taking the integral of a signal:

```
(define (integral integrand initial-value dt)
  (define int
    (cons-stream initial-value
                 (add-streams (scale-stream integrand dt)
                              int)))
  int)
```

## 3.5.4 Streams and Delayed Evaluation⁠

- The use of `delay` in `cons-stream` is crucial to defining streams with feedback loops.
- However, there are cases where we need further, explicit uses of `delay`.
- For example, solving the differential equation dy/dt=f(y) where f is a given function.
- The problem here is that `y` and `dy` will depend on each other.
- To solve this, we need to change `integral` to take a delayed integrand.

### Normal-order evaluation

- Explicit use of `delay` and `force` is powerful, but makes our programs more complex.
- It creates two classes of procedures: normal, and ones that take delayed arguments.
- (This is similar to the sync vs. async divide in modern programming languages.)
- This forces us to define separate classes of higher-order procedures, unless we make everything delayed, equivalent to normal-order evaluation (like Haskell).
- Mutability and delayed evaluation do not mix well.

## 3.5.5 Modularity of Functional Programs and Modularity of Objects⁠

- Modularity through encapsulation is a major benefit of introducing assignment.
- Stream models can provide equivalent modularity without assignment.
- For example, we can reimplement the Monte Carlo simulation with streams.

### A functional-programming view of time

- Streams represent time explicitly, decoupling the simulated world from evaluation.
- They produce stateful-seeming behavior but avoid all the thorny issues of state.
- However, the issues come back when we need to merge streams together.
- Immutability is a key pillar of _functional programming languages_.

Highlights

> We can model the world as a collection of separate, time-bound, interacting objects with state, or we can model the world as a single, timeless, stateless unity. Each view has powerful advantages, but neither view alone is completely satisfactory. A grand unification has yet to emerge. (Section 3.5.5)
> 
> The object model approximates the world by dividing it into separate pieces. The functional model does not modularize along object boundaries. The object model is useful when the unshared state of the “objects” is much larger than the state that they share. An example of a place where the object viewpoint fails is quantum mechanics, where thinking of things as individual particles leads to paradoxes and confusions. Unifying the object view with the functional view may have little to do with programming, but rather with fundamental epistemological issues. (Footnote 3.76)