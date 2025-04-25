
- So far our compound data has been made up of numbers.
- Now, we will start working with arbitrary symbols.

## 2.3.1 Quotation⁠

- Lists with symbols look just like Lisp code, except we _quote_ them.
- To quote in Lisp, we use the special form `(quote exp)`, or the shorthand `'exp`.
- For example, `'x` evaluates to the symbol “x” instead of the variable `x`’s value.
- This is like just natural language. If I say, “Say your name,” you will say your name. If I instead say, “Say ‘your name’,” you will literally say the words “your name.”

```
(define a 1)
(define b 2)

(list a b)
→ (1 2)

(list 'a 'b)
→ (a b)

(list 'a b)
→ (a 2)
```

- Now that we have quotation, we will use `'()` for the empty list instead of `nil`.
- We need another primitive now: `eq?`. This checks if two symbols are the same.

```
(eq? 'a 'b)
→ #f

(eq? 'a 'a)
→ #t
```

## 2.3.2 Example: Symbolic Differentiation⁠

- Let’s design a procedure that computes the derivative of an algebraic expression.
- This will demonstrate both symbol manipulation and data abstraction.

> Symbolic differentiation is of special historical significance in Lisp. It was one of the motivating examples behind the development of a computer language for symbol manipulation. (Section 2.3.2)

### The differentiation program with abstract data

- To start, we will only consider addition and multiplication.
- We need three differentiation rules: constant, sum, and product.
- Let’s assume we already have these procedures:

ProcedureResult`(variable? e)`Is `e` a variable?`(same-variable? v1 v2)`Are `v1` and `v2` the same variable?`(sum? e)`Is `e` a sum?`(addend e)`Addend of the sum `e``(augend e)`Augend of the sum `e``(make-sum a1 a2)`Construct the sum of `a1` and `a2``(product? e)`Is `e` a product?`(multiplier e)`Multiplier of the product `e``(multiplicand e)`Multiplicand of the product `e``(make-product m1 m2)`Construct the product of `m1` and `m2`

- Then, we can express the differentiation rules like so:

```
(define (deriv exp var)
  (cond ((number? exp) 0)
        ((variable? exp)
         (if (same-variable? exp var) 1 0))
        ((sum? exp)
         (make-sum (deriv (addend exp) var)
                   (deriv (augend exp) var)))
        ((product? exp)
         (make-sum
          (make-product (multiplier exp)
                        (deriv (multiplicand exp) var))
          (make-product (deriv (multiplier exp) var)
                        (multiplicand exp))))
        (else
         (error "unknown expression type" exp))))
```

### Representing algebraic expressions

- There are many ways we could represent algebraic expressions.
- The most straightforward way is parenthesized Polish notation: Lisp syntax.
- We can simplify answers in the constructor just like in the rational number example.
- Simplifying the same way a human would is hard, partly because the most “simplified” form is sometimes subjective.

## 2.3.3 Example: Representing Sets⁠

There are a number of possible ways we could represent sets. A set is a collection of distinct objects. Our sets need to work with the following operations:

ProcedureResult`(element-of-set? x s)`Is `x` a member of the set `s`?`(adjoin-set x s)`Set containing `x` and the elements of set `s``(union-set s t)`Set of elements that occur in either `s` or `t``(intersection-set s t)`Set of elements that occur in both `s` and `t`

### Sets as unordered lists

- One method is to just use lists. Each time we add a new element, we have to check to make sure it isn’t already in the set.
- This isn’t very efficient. `element-of-set?` and `adjoin-set` are Θ(n),Θ(n)\\htmlClass{math-punctuation}{\\text{,}} while `union-set` and `intersection-set` are Θ(n2).Θ(n^2)\\htmlClass{math-punctuation}{\\text{.}}
- We can make `adjoin-set` Θ(1)Θ(1) if we allow duplicates, but then the list can grow rapidly, which may cause an overall decrease in performance.

### Sets as ordered lists

- A more efficient representation is a sorted list.
- To keep things simple, we will only consider sets of numbers.
- Now, scanning the entire list in `element-of-set?` is a worst-case scenario. On average we should expect to scan half of the list.
- This is still Θ(n),Θ(n)\\htmlClass{math-punctuation}{\\text{,}} but now we can make `union-set` and `intersection-set` Θ(n)Θ(n) too.

### Sets as binary trees

- An even more efficient representation is a binary tree where left branches contain smaller numbers and right branches contain larger numbers.
- The same set can be represented by such a tree in many ways.
- If the tree is _balanced_, each subtree is about half the size of the original tree. This allows us to implement `element-of-set?` in Θ(log⁡(n))Θ(\\log(n)) time.
- We’ll represent each node by `(list number left-subtree right-subtree)`.
- Efficiency hinges on the tree being balanced. We can write a procedure to balance trees, or we could use a difference data structure (B-trees or red–black trees).

### Sets and information retrieval

- The techniques discussed for sets show up again and again in information retrieval.
- In a data management system, each record is identified by a unique key.
- The simplest, least efficient method is to use an unordered list with Θ(n)Θ(n) access.
- For fast “random access,” trees are usually used.
- Using data abstraction, we can start with unordered lists and then later change the constructors and selectors to use a tree representation.

## 2.3.4 Example: Huffman Encoding Trees⁠

- A _code_ is a system for converting information (such as text) into another form.
- ASCII is a _fixed-length_ code: each symbol uses the same number of bits.
- Morse code is _variable-length_: since “e” is the most common letter, it uses a single dot.
- The problem with variable-length codes is that you must have a way of knowing when you’ve reached the end of a symbol. Morse code uses a temporal pause.
- Prefix codes, like Huffman encoding, ensure that no code for a symbol is a prefix of the code for another symbol. This eliminates any ambiguity.
- A Huffman code can be represented as a binary tree where each leaf contains a symbol and its weight (relative frequency), and each internal node stores the set of symbols and the sum of weights below it. Zero means “go left,” one means “go right.”

### Generating Huffman trees

Huffman gave an algorithm for constructing the best code for a given set of symbols and their relative frequencies:

1. Begin with all symbols and their weights as isolated leaves.
2. Form an internal node branching off to the two least frequent symbols.
3. Repeat until all nodes are connected.

### Representing Huffman trees

- Leaves are represented by `(list 'leaf symbol weight)`.
- Internal nodes are represented by `(list left right symbols weight)`, where `left` and `right` are subtrees, `symbols` is a list of the symbols underneath the node, and `weight` is the sum of the weights of all the leaves beneath the node.

### The decoding procedure

The following procedure decodes a list of bits using a Huffman tree:

```
(define (decode bits tree)
  (define (decode-1 bits current-branch)
    (if (null? bits)
        '()
        (let ((next-branch
               (choose-branch (car bits) current-branch)))
          (if (leaf? next-branch)
              (cons (symbol-leaf next-branch)
                    (decode-1 (cdr bits) tree))
              (decode-1 (cdr bits) next-branch)))))
  (decode-1 bits tree))

(define (choose-branch bit branch)
  (cond ((= bit 0) (left-branch branch))
        ((= bit 1) (right-branch branch))
        (else (error "bad bit" bit))))
```

### Sets of weighted elements

- Since the tree-generating algorithm requires repeatedly finding the smallest item in a set, we should represent a set of leaves and trees as a list ordered by weight.
- The implementation of `adjoin-set` is similar to Exercise 2.61, except that we compare items by weight, and the element being added is never already in the set:

```
(define (adjoin-set x set)
  (cond ((null? set) (list x))
        ((< (weight x) (weight (car set))) (cons x set))
        (else (cons (car set)
                    (adjoin-set x (cdr set))))))
```





