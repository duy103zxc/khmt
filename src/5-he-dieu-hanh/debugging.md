# Debugging
## What Is A Debugger?

A debugger is a program that can control other programs. A programmer
uses it to:

- Find bugs!

- Start up a program

- Stop the program at a desired point (example: "when I enter
  function foo")

- Examine (and change) the state of the program

- Wait for the program to do something bad.


A debugger helps to answer the following questions:

- What statement made my program crash?

- If a function returns an error, who called it (and what were the
  arguments?)

- What is the value of a variable at a point in execution?

- What is the result of a particular expression in the program?


**Knowledge and familiarity with a debugger is _critical_ to**
**your success as a programmer.**

## What Is GDB?

GDB is the GNU Project's debugger, and is the most commonly used UNIX
debugger. Google for "gdb tutorial" and you'll see what we mean. In
these notes we will briefly describe how GDB works; the tutorials
available on-line go into much more detail.

GDB can:

- Start a program running and stop it at any point in execution.

- _Single-step_ through a program.

- Print source or disassembly surrounding the current instruction.

- Print out the contents of registers and memory, or change them.

- Execute functions within the program.

- Stop at a particular function or line in a function.

- Stop the program when the value of a variable changes.

- Print the _call chain_ to the current location in the
  program.


We'll go over these here.

First, you should compile your program with full debugging
information; this means that GDB can return more useful information to
you. This is done by putting "-g" on the command line:

```
gcc -g -o simulation simulation.c

```

When using multiple source files, each file gets the "-g", as well as
the link line:

```
gcc -g -c process.c
gcc -g -c unix.c
gcc -g -c binary.c
gcc -g -o program process.o unix.o binary.o

```

In general, there is no reason not to always use "-g".

Once you have a compiled program, you run it under GDB:

```
gdb program

```

This will get you a screen like so:

```
GNU gdb 6.3
Copyright 2004 Free Software Foundation, Inc.
GDB is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
Type "show copying" to see the conditions.
There is absolutely no warranty for GDB.  Type "show warranty" for details.
This GDB was configured as "i686-pc-linux-gnu"...
Using host libthread_db library "/lib/tls/libthread_db.so.1".

(gdb)

```

The (gdb) is the prompt, much like a command prompt. Here are some
commands:

- **help**

   Gives help about GDB.

- **run** or **run arg1 arg2 arg3 ...**

  This starts the program running. If you just type "run" it will run
  the program with no arguments; any command line arguments are
  specified after the run command.

- **break** _location_

  Creates a _breakpoint_: the program will halt when the point is
  reached and GDB will ask you what to do next. The most common is to
  break at the entry of a _function_:



  ```
  (gdb) break main
  Breakpoint 1 at 0x80483a0: file foo.c, line 3.
  (gdb)

  ```


  but you can also break at a particular line (or address):


  ```
  (gdb)break foo.c:4
  Breakpoint 2 at 0x80483ac: file foo.c, line 4.
  (gdb)

  ```



  When you run the program, it will automatically stop when a breakpoint
  is reached, and report that to you:



  ```
  (gdb) run
  Starting program: /afs/cs.wisc.edu/u/b/e/bernat/foo

  Breakpoint 1, main () at foo.c:3
  3         printf("42");

  ```



  Note that GDB tells you which breakpoint was reached (1), the file and
  line (foo.c, line 3), and then prints out the statement that was going
  to be executed (printf).

- **delete**

  Deletes a breakpoint. For example, delete 2.

- **step**, **next** and **finish**

   Step: executes a single statement and then stops again. Next: as
   with step, but if the next statement is a function call, it executes
   the entire call. These are very useful for fine-grained control over
   the program, and are equivalent to setting breakpoints at each
   statement. Finish runs until the end of the current function, then stops.

- **continue**

   Continues a process that was stopped via a breakpoint or
   single-stepping. It will run until you interrupt it or it hits a
   breakpoint.

- **where** or **backtrace**

   Print the _call chain_ \- the list of called functions - that
   brought the program to its current location, and print their
   arguments as well.

- **print** _E_

   Print the value of expression _E_ from the point of view of the
   program, where _E_ is some C statement. For example, take the
   following simple program:




  ```
  brie(62)% cat foo.c
  int main() {
      int i = 32;
      printf("42");
      i = 42;
      return i;
  }

  ```



  which we compile and start under GDB:


  ```
  (gdb) break main
  (gdb) break foo.c:3
  (gdb) break foo.c:5
  (gdb) run
  ...
  Breakpoint 1, main() at foo.c:2
  (gdb) print i
  $1 = 9989280    <--- Note - i isn't initialized yet, and so is garbage
  (gdb) continue
  Breakpoint 2, main () at foo.c:3
  (gdb) print i
  $2 = 32
  (gdb) continue
  Breakpoint 3, main () at foo.c:5
  (gdb) print i
  $3 = 42
  (gdb) continue
  Program exited with code 052. <--- the return value of the program

  ```



  We can use breakpoints and print to track the value of a variable
  through the execution of the program.

- **quit**

Copyright © 2008 Andrew R. Bernat

