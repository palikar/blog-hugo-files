+++
title = "Alisp [PART 2]"
author = ["Stanislav Arnaudov"]
description = "Building Alisp from source"
date = 2020-05-27T00:00:00+02:00
keywords = ["c++", "language", "lisp", "interpreter"]
lastmod = 2020-05-29T01:52:50+02:00
categories = ["c++", "alisp"]
draft = false
weight = 100
+++

{{< figure src="/ox-hugo/alisp.png" >}}


## Abstract {#abstract}

In this post, I want to give a little more details on how [Alisp](https://github.com/palikar/alisp) can build from sources. ALisp is an interpreted language I've developed in C++. Over the last half a year, the project has become quite big (currently around 24K LOC) and thus the build process is so trivial as in the beginning. In the process of development, I've added numerous options for how the code can be compiled and all of those can be confusing at times. Depending on the purpose of the build and what you want to do with it, different options have to enabled and here I want to give a brief overview of the options and the reasoning behind them.


## Prerequisites {#prerequisites}

Alisp is written in C++ (17) and uses [CMake](https://cmake.org/) as its build generator. The project is tested and it compiles with

-   gcc 8,9
-   clang 9

Currently, only Linux builds are tested and I can't guarantee that everything will work on Windows (hopefully supports comes soon). The code requires features of C++17 so it may not be compatible with older compilers.

<br />

On Debian based systems, for minimal builds the package [build-essential](https://packages.debian.org/sid/build-essential) should be enough:

```sh
sudo apt-get install build-essential
```

<br />

For more advanced builds, you'll need more things:

```sh
sudo apt-get install valgrind gcov doxygen
```

[Doxygen](https://www.doxygen.nl/) is used to generate the documentation for the source, [valgrind](https://www.valgrind.org/) performs some run-time checks and [gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html).

<br />

Optionally, the build setup can take advantage of [ccache](https://ccache.dev/) if you have on your system. This is a compiler cache that can reduce compilation times. For Debian systems, to install ccach, you can run:

```sh
sudo apt-get install ccache
```

<br />

The code itself should not have any external dependencies. The project's repository contains everything needed for building all executables and libraries of the language form source. Alisp does, however, uses third party C++ libraries and those are:

-   [Catch2](https://github.com/ChaiScript/ChaiScript/) - used for testing
-   [Clipp](https://github.com/muellan/clipp) - used for command line arguments parsing
-   [Rang](https://www.google.com/search?q=c%2B%2B+rang&ie=utf-8&oe=utf-8&client=firefox-b-e) - used for printing in colors on the terminal
-   [TinyXML2](https://github.com/leethomason/tinyxml2) - used for parsing xml documents
-   [Subprocess](https://github.com/arun11299/cpp-subprocess) - used for spawning child processes
-   [SimpleJSON](https://github.com/nbsdx/SimpleJSON) - used for parsing json documents.
-   [Fmt](//fmt.dev/latest/index.html) - used for formatting strings


## Building {#building}

Once you have everything you need, you can proceed to the actual build process. The way I usually do this is to create a build folder in the source directory and build the project there. The build commands are standard for any CMake project.

```sh
git clone https://github.com/palikar/alisp
cd alisp
mkdir build && cd build
cmake .. <options>
make <targets>
```

Optionally, after cloning you can checkout the project to any of the [current stable version](https://github.com/palikar/alisp/releases).

`<options>` and `<targets>` are, of course, placeholder and will be discussed in the next section.


## Options {#options}

As shown, options are given when executing the CMake command. The format is `-D<OPTION>=<VALUE>`. For example, `DOPTION_1=TRUE` sets the option `OPTION_1` to `TRUE`.

<br />

First, we'll go over the project-specific options. Those have been declared in the [CMakeList.txt](https://github.com/palikar/alisp/blob/master/CMakeLists.txt) file with the [option](https://cmake.org/cmake/help/v3.13/command/option.html) function of CMake. Next, I'll also mention a couple of other general CMake options that are commonly used.


### Project-specific options {#project-specific-options}

-   `LIBCPP` - when compiling with clang, this option allows you to use llvm's implementation of the C++ standard library [libc++](https://libcxx.llvm.org/%20). In certain situations, libc++ can improve the performance of a program and reduce the compilation time. For the majority of the development of Alisp, however, I've used GNU's standard library [libstdc++](https://gcc.gnu.org/onlinedocs/libstdc++/) and I haven't got any problems with it. If the `LIBCPP` option is set to `TRUE` (default `FALSE`), libc++ will be used.

-   `ENABLE_COVERAGE` - if set to `TRUE` (default `FALSE`), the project will be compiled with coverage flags (`--coverage -O0 -g`) and special make targets will be created that allows you to run coverage of the code and generate nice HTML pages with the results.

-   `ENABLE_ASAN` - if set to `TRUE` (default `FALSE`), the project will be compiled with address sanitizer flags (`-fsanitize=address`). When an executable is built with address sanitizer, running it will execute the program while checking for any memory issues (buffer overflows, invalid reads/ writes, etc.) More information on address sanitizers [here](https://github.com/google/sanitizers/wiki/AddressSanitizer).

<!--listend-->

-   `BUILD_SHARED_LIBS` - if set to `TRUE` (default `FALSE`), all of the libraries will be built as shared libraries. This usually reduces the recompilation time but may cause some overhead on startup. By now, I use Alisp almost exclusively in shared libraries mode.

-   `ENABLE_TESTING` - if set to `TRUE` (default `FALSE`), all of the tests for the project will be compiled and build. Also, several make targets will be created for running the added tests.

-   `ENABLE_CLANG_TIDY` - if set to `TRUE` (default `FALSE`), [clang-tidy](https://clang.llvm.org/extra/clang-tidy/) will be run over the source code during compilation. Clang-tidy is a code linter and static analyzer and can report on some common problems with the code.

-   `ENABLE_CPPCHECK` - if set to `TRUE` (default `FALSE`), [cppcheck](http://cppcheck.sourceforge.net/) will be run over the source code during compilation. Cppcheck is a static analysis tool for C/C++ code that detects bugs and focuses on detecting undefined behavior and dangerous coding constructs.

-   `ENABLE_LTO` - if set to `TRUE` (default `FALSE`), the project is compiled with [link time optimizations](https://gcc.gnu.org/wiki/LinkTimeOptimization) enabled.

-   `BUILD_DOC` - if set to `TRUE` (default `FALSE`), a special make target is created that will generate documentation of the source code with Doxygen.

-   `FORCE_COLORED_OUTPUT` - if set to `TRUE` (default `TRUE`), the output of the compilation will use coloring. This is handy when compiling and looking at the output on the screen. It could be annoying when the output of the compilation is dumped into a file.

-   `ELF_HASHING` - if set to `TRUE` (default `FALSE`), the default hashing algorithm that Alisp uses internally will be the [PJW hash function](https://en.wikipedia.org/wiki/PJW%5Fhash%5Ffunction). By default, Alisp uses the [Fowler–Noll–Vo function](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo%5Fhash%5Ffunction) for hashing strings.

-   `JENKINS_HASHING` - if set to `TRUE` (default `FALSE`), the default hashing algorithm that Alisp uses internally will be the [Jenkins hash function](https://en.wikipedia.org/wiki/Jenkins%5Fhash%5Ffunction). If this option is `TRUE`, `ELF_HASHING` should be set to `FALSE`.

-   `DEBUG_LOGGING` - if set to `TRUE` (default `FALSE`), the project will be compiled with support for enabling lots of logging. The interpreter will have an extra CLI flag (`-DL`) that will print lots of debugging information during execution.

-   `BUILD_EXAMPLES` - if set to `TRUE` (default `FALSE`), the example scripts in the `scripts` folder will be copied in the build directory. Also, an extra make target will be created (`test_examples`) that will execute all of the examples and verify they execute without errors.

-   `VALGRIND_CHECKS` - if set to `TRUE` (default `FALSE`), an extra make target (`valgrind_checks`) will be created that executes the language tests while checking the execution with various valgrind tools. This includes checking for memory leaks, running the tests with [massif](https://valgrind.org/docs/manual/ms-manual.html), [memcheck](https://valgrind.org/docs/manual/mc-manual.html) and [callgrind](https://valgrind.org/docs/manual/cl-manual.html).

-   `RUN_PERFORMANCE_TESTS` - if set to `TRUE` (default `FALSE`), the scripts in the `performance_tests` will be copied in the build directory and an extra make target will be created (`timing_check`). The target will execute the copied scripts and will create a file `timing_checks.txt` with their execution time. The scripts are designed in such a way to load the interpreter and test its performance.

-   `DISABLE_DYN_MODULES` - if set to `TRUE` (default `FALSE`), the dynamic Alisp modules will not be compiled. Those are modules that are not loaded in each interpreter executable but are loaded dynamically at run time. Disabling the compilation of those can be handy when you don't want to compile the entirety of the project but just the core interpreter.

-   `DISABLE_DEFAULT_MODULES` - if set to `TRUE` (default `FALSE`), the builtin Alisp modules will not be compiled and linked to the interpreter. The point of this is again to reduce the compilation time if you don't need these modules.

-   `ENABLE_STACK_TRACE` - if set to `TRUE` (default `TRUE`), the interpreter will keep track of each function call and will print out a stacktrace in case of an error in the execution. Disabling this option can potentially increase performance.

-   `ENABLE_LINE_TRACE` - if set to `TRUE` (default `TRUE`), the interpreter will keep track of the line number where a list-form object was defined in a file. This makes the stracktrace more understandable but also requires a tiny bit more memory. The line numbers will be saved as object properties (`--line--`) of list-forms that appear in the source code of an Alisp script.

-   `ENABLE_OBJECT_DOC` - if set to `TRUE` (default `TRUE`), the interpreter will keep the documentation for symbols. If this option is disabled, the given in the source code documentation for the symbols will be left out and won't be part of the symbols. The documentation for a symbol is saved in the `--doc--` property.

-   `DISABLE_RUN_TIME_CHECKS` - if set to `TRUE` (default `FALSE`), a lot of run-time checks (among which arity checks by the function calling and type checks) will be removed from the code. This can potentially increase performance but will make the interpreter extremely unstable. Segmentation faults are to be expected if this option is enabled. It is, however, guarantee that a valid program in Alisp will run correctly if these checks are disabled.

-   `TIME_CHECK_SAMPLES` - this is an integer value (default 10) that signifies how many times the performance test scripts should be run to get a stable average value.

-   `ENABLE_MTEL` - if set to `TRUE` (default `FALSE`), the project will be built with a multi-threaded event loop. Usually, the event loops run on a single thread that does all of the asynchronous dispatching. With this option, multiple threads will be used to handle asynchronous events and callbacks.


### General Options To Keep in Mind {#general-options-to-keep-in-mind}

-   `CMAKE_INSTALL_PREFIX` - this is a path indicating where the compiled libraries and executable should be installed (copied to when everything is built) through `make install`. On Linux systems, this is usually `/usr` but I often change this to some local `usr` directory. Executable will be put in `CMAKE_INSTALL_PREFIX/bin`, libraries in `CMAKE_INSTALL_PREFIX/lib` and header files in `CMAKE_INSTALL_PREFIX/include/alisp`.

-   `CMAKE_BUILD_TYPE` - string variable indicating the type of build that is to be built. It can be "Debug", "Release", "RelWithDebInfo" or "MinSizeRel". In most cases, I use either "Debug" (debug information is compiled in) or "Release" (the compilation is optimized fully and there is no debug information).


## Make Targets overview {#make-targets-overview}

With the options being enplaned now, here I'll give a brief overview of all possible make targets. Those can be run after Cmake has executed, build files are ready and we can proceed to compilation and testing.

-   `all` - compile the project

-   `test` - run all of the tests. The project has to be built with `-DENABLE_TESTING=TRUE`

-   `alisp_coverage_html` - execute the tests, gather coverage information, and generate a web site with the results. The project has to be built with `-DENABLE_COVERAGE=TRUE`

-   `alisp_coverage_xml` - execute the tests, gather coverage information, and generate a web site with the results. The project has to be built with `-DENABLE_COVERAGE=TRUE`

-   `doc` - build the documentation of the source code

-   `install` - copy the compiled executable, libraries, and public headers at the appropriate places.

-   `package` - package the project files with [CPack](https://cmake.org/cmake/help/latest/module/CPack.html). This will generate [.deb](https://en.wikipedia.org/wiki/Deb%5F(file%5Fformat)) and .tar.gz files.

-   `test_examples` - execute all of the examples in the `scripts` directory. The project has to be built with `-DBUILD_EXAMPLES=TRUE`

-   `valgrind_checks` - execute `./bin/alisp_language_test` with valgrind for all of the defined valgrind checks. The project has to be built with `-DVALGRIND_CHECKS=TRUE`

-   `timing_check` - execute the scripts that test the performance of the interpreter. The project has to be built with `-DRUN_PERFORMANCE_TESTS=TRUE`
