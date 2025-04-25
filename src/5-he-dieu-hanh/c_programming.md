# C Programming
## 1 Why C? And Why this Primitive Command-Line Stuff?

C is, by far, the most common language used for operating implmentation.
It is simple to the point of being primitive, with few unexpected behaviors.
System designers like it because it is predictable.

In addition, it is a least-common-denominator in that it should be a subset
or compatible with almost any other language (for example,
C++, Fortran, or Java).

Visual programming environments, with their automatic dependence analysis
and GUI's are quite pleasant and helpful.
However, they are almost never used in operating system programming.
First, the code tends to be so large and complex, that it is difficult to
manage in one of these environments strictly through the visual interface.
Even Microsoft used Makefiles and command-line operations in building and
testing Windows.

Second, debugging an operating system is still a black art. In many cases,
especially relating to the lowest levels of the system, you do not have
a debugger available.
In the cases where you do have a debugger (such as when using a virtual
machine partition, firmware debugger, or network debugger), it is often
quite primitive and uses simple command lines.

As my advisor (Mike Powell) likes to say, "Life is tough on the frontiers
of computer science."

## 2 Compiling

The simplest case (which you will **not** encounter in CS537, is when
the source program is entirely in a single file.
Of course, C source files all end with the .c suffix.
The simplest command to compile a single program ( `foo.c`)
into an executable file, you would type:

`    gcc foo.c
`

The result of a successful compiliation will be an executable file with the
name `a.out`.
You can specify the name of the file with the `-o` option:
`    gcc -o foo foo.c
`

To get more help from the compiler in finding questionable programming
practices that can lead to errors, you need to include options to turn
on warnings and strict type checking:

`    gcc -o foo -Wall -pedantic foo.c
`

## 3 Separate Compiliation 1: Compiling

Compiling your whole program in one source file is a truly bad idea.
You should divide your program into clear separate abstractions, based
on the types of data types (such as lists, search trees, or page tables)
and basic functional units (such as command-line options processing or
memory management).

Your program, therefore, will be divided into several `.c` files,
each of which will be compiled separately.
You will create a binary file, called an _object_ file for each
source file.
In the Linux world, these files have the suffix `.o`, while in
the Windows world the suffix tends to be `.obj`.
After you have compiled each source file to an object file, you will
_link_ them all together into a single exectuable file.

To compile file `bar.c` to an object file (called `bar.o`):

``

```
    gcc -c -Wall -pedantic bar.c

```

After you have compiled each of your source files,
`foo.c`,
`bar.c`, and
`glarch.c`
into object files, you can link them together into an executable with:

`    gcc -o foo foo.o bar.o glarch.o
`

## 4 Separate Compiliation 2: The C Code

### 4.1 Functions

Functions in one `.c` file need to be able to reference
functions in another file.
This means that they need to know the name and types of parameters of
the functions in another file.
C allows us to handle this situation by providing a couple of features.
First, we can declare _prototype_ declarations for a function.
These prototypes describe the name of the function and the type of its
parameters and return values.
Note that it does not name the parameters, just states their types.
So, if you had a function:

`    int
    PageNumber (int virtualAddress)
    {
        const int PAGESIZE = 1024;
        return (virtualAddress / PAGESIZE);
    }
`

It's prototype declaration would be:

`    int PageNumber (int);
`

For each source file, it is a good idea to create a separate header
( `.h`) file that contains the prototype declarations.
Then, we include the header file each time that we compile another
file that uses the functions.

Assume that function `PageNumber` is defined in a file called
`PageTable.c` and its prototype appears in `PageTable.h`.
If this function needed to be used from file `VirtualMem.c`, then
the code in `VirtualMem.c` would looking something like:

``

```
    #include "PageTable.h"

    . . .
    p = PageNumber (pn);
    . . .

```

If you are also defining data types, for example, a new structure
type definition for this PageTable.c file, and if the other code that will
use the PageTable.c will need these types, then you can (should) put
these declarations in the include file.

### 4.2 Variables

A global variable is one that is not local to any specific function,
i.e., it is statically allocated and accessible to all functions in all
files.

Note, however, that global variables should be used _exceedingly_
sparingly.
Because they can be accessed and modified from anywhere, it is easy
to have undiscipined and careless use of such a variable.
As a result, they are quite error prone.

As with a function, the global variable has one declaration, in one file,
that actually allocated the variable, and then has declarations that
reference it from other files.
So, in only one file, outside any function, we declare a character pointer:

``

```
    char *fileName;

```

And in each other file that wants to use this variable, we declare it as:

`    extern char *fileName;
`

## 5 Dynamic Memory Allocation

> _By indirections find directions out...__Polonius in_
>
> _Hamlet, Act 1_

_Systems programming heavily uses dynamically allocated data structures._
_Example are such data structures are lists or queues that grow as you_
_add elements._
_Typically, new elements are allocated when needed and linked into the_
_data structure by storing pointers to the newly allocated element._
_When elements are removed from the data structure, the memory must be freed._

_You can find more explanation in_
_[OSTEP Chapter 14](http://pages.cs.wisc.edu/~remzi/OSTEP/vm-api.pdf)_

_The library call `malloc` is used to allocate memory and `free`_
_is used to free it._
_In Java or C++, you would accomplish the same thing by using_
_the `new` and `delete` operations._

_A few observations about dynamically allocated memory:_

_1. You **must** master the use of `malloc`, `free`,_
_and pointers._
_There is no way to survive as a systems programmer without it._

_2. Pointers and dynamically allocated memory are extremely error prone._
_Common errors are forgetting to allocate an object (uninitialized pointer),_
_using a pointer after memory has been freed,_
_freeing memory more than once,_
_using a pointer to reference the wrong type of data structure,_
_and forgetting to free memory ("memory leak")._
_Careful and discipline use of pointers is important._

_3. When things go wrong with the use of pointers, debugging can be a_
_serioius challenge._
_You can scribble on a random memory location with a bad pointer and then_
_not encounter the problem until much later in the program's execution._
_Finding the source of such problems is not easy._
_Later in the semester, we will try out some debugging tools that help_
_with such problems._


_Here is a simple structure declaration for a node in a binary tree:_

_`struct node_
_{_
_char *nodeName;           /* Name of tree node */_
_int   nodeValue;          /* A value associated w/this node */_
_struct node *leftChild;_
_struct node *rightChild;_
_};_
_`_

_If we declare a pointer:_

_`    struct node *p;_
_`_

_We allocate a new element of type `node` with:_

_`    p = (struct node *)malloc(sizeof(struct node));_
_if (p == NULL) { handle the error case }_
_`_

_And, when we're done with this node, we can deallocate the memory with:_

_`    free(p);_
_`_

_* * *_

_Copyright © 2011, 2018, 2020 Barton P. Miller_

