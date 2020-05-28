+++
title = "Alisp [PART 1]"
author = ["Stanislav Arnaudov"]
description = "Interpreted Lisp-language written in C++."
date = 2020-02-20T00:00:00+01:00
keywords = ["c++", "language", "lisp"]
lastmod = 2020-05-27T22:55:07+02:00
categories = ["c++"]
draft = false
weight = 100
+++

{{< figure src="/ox-hugo/alisp.png" >}}


## Abstract {#abstract}

It's has been a long standing idea of mine to just sit down and write a language. Not a complicated one, but also not completely useless one. Alisp is the crystallization of this idea into something concrete. It is a interpreted language based on [Emacs-Lisp](https://www.gnu.org/software/emacs/manual/html%5Fnode/eintr/) (elisp). I've borrowed a lot concepts from there and syntax-wise, Alisp closely follows elisp. By now the core runtime of the language is pretty feature complete and it is possible to run simple to moderately complicated programs. Being a lisp dialect, the overall feeling of the language is functional. The syntax is "minimal" and once the basic concepts of s-expression evaluation are understood, it is easy to become proficient in the language.

<br />

The project is fully open source and any one can study the code of the interpreter. When it comes to the design, I've tried to keep it as simple as possible while still implementing non-trivial features that make the working with Alisp more pleasant.


## Design Goals {#design-goals}

At the beginning, I didn't have concrete plan but as the project grew in scope, I had to come up with several general guidelines. I now have several considerations while continuing with the development. These are:

-   Easy to add functionality -- it should be relatively easy to extend the language through modules that can either be written in C++ or in Alisp itself.
-   Access to low level memory -- even though this is an interpreted language, the user should have the ability to operate on some raw memory, read and write to it, and use OS functions to manipulate it.
-   Automatic memory management -- the user should not be worrying about when are objects destroyed and deallocated from memory.
-   Simple syntax -- adding things to the syntax is not absolutely unwanted but I prefer it to work around the current syntax and do cool stuff with it, instead of trying to bodge new things.


## Features so far {#features-so-far}

By now, I've put quite the work into the language and the utilities around it, so there are lots of things that ALisp can offer. Those include but are not limited to:

-   Language constructs - control flow, loops, variable and function declarations, local variables, lambdas, macros, math functions, etc.
-   Importing -- files are considered modules that can be imported in other files.
-   Referring symbols in other modules
-   REPL -- Read–eval–print loop, much like the python console.
-   Stack tracing -- Printing the current stack on error.
-   Working with lists
-   Signal raising and handling -- "Signals" are kind of like the exceptions in Alisp.
-   Properties -- each object has it's own set of properties that can store arbitrary information.
-   Controlling certain interpreter behavior through environment variables.
-   Several simple utility algorithms for working with lists.
-   Files -- opening, reading from, writing to and closing file handles.
-   Memory buffers -- ability to allocate arbitrary sized buffer of bytes in memory.
-   Streams -- a layer of abstraction that enables unified reading and writing form "stuff"
-   Dynamically linked modules -- modules can be written in C++ and linked at runtime.
-   Prelude script -- each time the interpreter is started, a script that defines several useful macros and functions is executed


## Documentation {#documentation}

The documentation is not at all complete but there is enough to get you started [here](https://alisp.readthedocs.io/en/latest/). Also, check out the [Github repository](https://github.com/palikar/alisp). The code has its own documentation.


## Examples {#examples}

Here I'll present several simple programs. The point is just to illustrate the syntax and prove that that there is, in fact, something that can be done with Alisp.


### Hello world {#hello-world}

```lisp
(println "Hello World form Alisp!")
```


### Defining variables and functions {#defining-variables-and-functions}

```lisp
(defvar global-var-1 "string")
(defvar global-var-2 42)

(defun fun (x)
  (let ((local-var-1 "val-1")
      (local-var-2  43))
  (println "x: " x)
  (println global-var-1 " -> " local-var-1)
  (println (+ global-var-2 local-var-2))))

(fun "x")
(fun 42)

```


### Flow control {#flow-control}

```lisp
(defvar cnt 1)

(while (< cnt 100)
  (println "i: " cnt)
  (setq cnt (+ cnt 1)))

(if (== cnt 100)
    (println "Count is 100")
  (println "Count is not 100!"))

```
