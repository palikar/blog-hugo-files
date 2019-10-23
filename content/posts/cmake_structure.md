+++
title = "The optimal CMake project structure."
author = ["Stanislav Arnaudov"]
description = "Description of the project structure I intend to use in the future for my c++ projects."
date = 2019-04-10T00:00:00+02:00
keywords = ["c++", "cmake", "projects", "structure"]
lastmod = 2019-10-23T09:26:27+02:00
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

I am sure that every C++ programmer has at one point struggled with [CMake](https://cmake.org/). There have been multiple times where when I have to start work on some C++ project, I've to spend a good couple of hours in thinking how should my project structure look like. It's just such a huge hassle to think about all of your CMakeList.txt files and possible libraries and different modules and... the things that may go wrong with your build system. I've been meaning to find a good template of a CMake project for a long time and now I think I've found\created my long wanted gem. In this post, I first want to give a brief overview of my experience with CMake and the present the template project that I have finally settled with.


## The horrible past experience {#the-horrible-past-experience}

I have bad memories in my early days of dealing with CMake. At the start, I had the impression that a build system should make everything as easy as possible but it seems that often this is not the case. Don't get me wrong. I know build systems do not solve an easy problem, but still. I quickly noticed how there are a lot of people on the internet talking about their way of doing CMake projects. It didn't seem that there wasn't **the** way of doing it. This is generally a good thing. Freedom and doing things _your_ way! But everything comes with a cost. The cost of CMake -- you have no idea what you are supposed to do at the beginning.

<br />

My first experience with a big CMake project was while dealing with legacy code. The structure was similar to the one of [OpenCV](https://github.com/opencv/openc). A big CMakeList.txt file at the top lever with general project settings, a separate file with all of the libraries needed for the project and then a source directory with different modules. Each module is in a folder of its own. The structure of a module: source, include, test, and data directory; a CMakeList.txt file a the root of the module to define all executables and libraries. In all fairness, I still like the structure of the individual modules. The strange thing with the module-system was the way it did dependency resolution. Everything was done "manually". It essentially was a big CMake framework that first collected info about every module, looked at the dependencies between them and the needed libraries and then build targets manually for a module by linking explicitly everything that module needed. The whole thing was implemented in a collection of complicated macros. There maps and lists and algorithms and directory traversals and everything you can imagine. The whole thing is written in CMake of course. Legacy code for the win!

<br />

At one point I was considering building a similar system myself for my personal projects. Needless to say, that didn't take off. I wanted to clear up the structure and tightened it around the edges but I didn't have any form of success.

<br />

I browsed some more CMake projects, I watched some talks (see [Effective CMake](https://www.youtube.com/watch?v=bsXLMQ6WgIk) Talk by [Daniel Pfeifer](https://github.com/purpleKarrot)) and I also tried setting up some thins myself. Nothing adequately clicked with me. CMake was still getting in the way of my C++ programming and it was making it even more painful than it already is. This went on for some while.


## The bright new world {#the-bright-new-world}

Finally, a very fortunate thing happened. I watched [Applied Best Practices](https://www.youtube.com/watch?v=DHOlsEd0eDE) and decided to check out the [repository](https://github.com/lefticus/cpp%5Fbox). The project is created with the idea of being a demonstration of "doing things properly" in C++. I can agree with this sentiment 100%! I still did some adjustments but I am still grateful that I found this project. Here I will go over the elements of the structure of the template project that I created.


### Conan {#conan}

[Conan](https://conan.io/) is one of the numerous attempts to bring the "package management world" to C++. Package management is one of those things that modern languages just "have by default" but somehow C++ hasn't exactly caught up. For by CMake structure I wanted at least some attempt to make external dependency management a little bit easier. For this reason, I figured that Conan may be the utility to start my experimentation with "C++ package management"

<br />

From Conan's website:

> The open-source, decentralized and multi-platform package manager to create and share all your native binaries.

I won't give (because I can't) a comprehensive guide on Conan here. I believe this here is just enough to set you up and give you some basics of how Conan does its magic.

<br /> Conan can be installed through [pip](https://pypi.org/project/conan/) and there is an official [get started](https://docs.conan.io/en/latest/introduction.html) guide. Once everything is set up properly, you can write your `conanfile.txt` file in the root directory of your project. This is where all of your dependencies are defined. An example file is:

```conf
[requires]
        catch2/2.4.0@bincrafters/stable
        spdlog/1.3.1@bincrafters/stable
        fmt/5.3.0@bincrafters/stable
        rang/3.1.0@rang/stable
        clara/1.1.4@bincrafters/stable


[generators]
        cmake
        cmake_find_package
```

It is a _conf_ file with several sections. The dependencies go in the `requires` node. Usually what to put there is defined on the page of each Conan package. The `generators` section defines exactly what Conan does and how it builds the dependencies configuration files. Different generators have different effects and use cases. As we are dealing with a CMake project, the `cmake` and `cmake_find_package` are enough. With this configuration setup, you can later include several lines of CMake code in your top-level CMakeLists.txt and just call `find_package(...)` for your dependencies. In theory at least. Sometimes things are a little bit tricky to get them running. If everything is as it is supposed to be, you can execute `conan install <path_to_source_directory>` in the build directory of the project and all of the dependencies will be fetched and saved in some cache on your machine.

<br />

In my top-level CMakeLists.txt, I have the following lines:

```cmake
# First a conan.cmake is downloaded, if not present. This file handles all of the "Conan"-y things with cmake... I guess.
if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
    message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD  "https://raw.githubusercontent.com/conan-io/cmake-conan/master/conan.cmake"
        "${CMAKE_BINARY_DIR}/conan.cmake")
endif()
include(${CMAKE_BINARY_DIR}/conan.cmake)

# This is supposedly the minimal setup in CMake for Conan. After these lines, everything should be ready to use
conan_check(REQUIRED)
conan_cmake_run(
    CONANFILE
    conanfile.txt
    BASIC_SETUP
    CONAN_COMMAND
    ${CONAN_CMD}
    CMAKE_TARGETS
    BUILD
    missing)
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake) # conanbuildinfo.cmake is file generated by the cmake generator.
conan_basic_setup()
```

This sets up everything and at some later point I can just use something like:

```cmake
find_package(clara)
```

This includes all of the CMake targets defined for [Clara](https://github.com/catchorg/Clara). We can later effortlessly link agings them when defining our executable for example.


### Top level CMakeList.txt {#top-level-cmakelist-dot-txt}

With Conan out of the way, I can now proceed to the pure CMake part of my project structure. At the start, I want to layout my key core design goals.

-   Follow [modern CMake guidelines](https://www.youtube.com/watch?v=bsXLMQ6WgIk)
-   Ease of use
-   Modular setup
-   Not worrying about dependencies between modules
-   Support for testing
-   Support for automatics documentation generation


#### Folder structure {#folder-structure}

A high-level overview of the my project structure is as follows:

```sh
├── cmake
│   ├── FindGit.cmake
│   ├── git_revision.cmake
│   └── safeguards.cmake
├── doc
│   └── Doxyfile.in
├── libs
│   └── CMakeLists.txt
├── src
│   ├── module_1
│   ├── module_2
│   └── CMakeLists.txt
├── templates
│   ├── template_app_module
│   └── template_lib_module
├── CMakeLists.txt
├── conanfile.txt
├── create_new_app_module.sh
├── create_new_lib_module.sh
├── Doxyfile.in
├── LICENSE
├── Makefile
└── README.md
```

Everything comes together at the top-level CMakeLists.txt. This is the main entry point when running `cmake <path_to_source_directory>`. In the `cmake` several utility CMake scripts solve several small problems like finding the git version of the host machine and preventing me to build the project inside the source file tree. The `Doxyfile.in` file contains the basic configuration setup for [Doxygen](http://www.doxygen.nl/) which I use for generating documentation. My idea for the `libs` folder is to encapsulate the external dependencies that are not available through Conan. Of course, for some big dependencies (e.g. OpenCV) this is not really viable but it works like a charm for header-only libraries. The `libs/CMakeLists.txt` is responsible for loading the libraries in the `libs` folder. The `src` directory houses all of the individual sub-modules in separate sub-folders that are included through the `src/CMakeLists.txx` script. More on the individual modules in a minute. In `templates/` I have template visions of the two modules that the project can have -- a library or an executable. I want to add a new module to the project, I can use of the scrips at top-level -- `create_new_app_module.sh` or `create_new_lib_module.sh` -- to generate the module easily. The `create_new_app_module.sh` looks like this:

```sh
# root directory of the project
DIR=$(dirname "$(readlink -f "$0")")

MODULE_NAME=$1
# get the name of the project crom the CMakeLists.txt file
PROJECT_NAME=$(grep "project(\w*" CMakeLists.txt -o | grep "(.*" -o | cut -c 2-)

if [ -d $DIR/${PROJECT_NAME}/${MODULE_NAME} ]; then
    echo "The module already exists"
    exit 1
fi

# copy the template module and rename its parts
cp -r $DIR/templates/template_app_module/ $DIR/src/
mv $DIR/src/template_app_module/ $DIR/src/${MODULE_NAME}
mv $DIR/src/${MODULE_NAME}/include/PROJECT/MODULE_NAME $DIR/src/${MODULE_NAME}/include/PROJECT/${MODULE_NAME}
mv $DIR/src/${MODULE_NAME}/include/PROJECT $DIR/src/${MODULE_NAME}/include/${PROJECT_NAME}

# expand 'MODULE_NAME' to the name of the module in every file of the module's folder
find $DIR/src/${MODULE_NAME} -type f -exec sed -i "s/MODULE_NAME/${MODULE_NAME}/g" {} \;
find $DIR/src/${MODULE_NAME} -type f -exec sed -i "s/PROJECT_NAME/${PROJECT_NAME}/g" {} \;

# add the module to the 'src/CMakeLists.txt' script
LINE="add_subdirectory(${MODULE_NAME})"
if [ ! $(grep $LINE $DIR/src/CMakeLists.txt) ]; then
    echo ${LINE} >> $DIR/src/CMakeLists.txt
fi
```

With this setup, I can simply execute `./create_new_app_module.sh executable_1` to add a new sub-module with the name "executable\_1".

<br />

The `Makefile` is there just so that I can automate some of the things I do regularly in the project folder. Things like rebuilding, creating debug or release builds or cleaning all build folders.


#### Options {#options}

With the general folder structure, we can now go through several parts of the top-level CMakeLists.txt script.

<br />

Near the top of the script, I have the options with which the project can be built. Those are just variables that can be true or false and enable certain conditions for the later parts of the scripts. The options are:

```cmake
option(ONLY_COVERAGE "Build only tests necessary for coverage" FALSE)
option(LIBCPP "Build with libc++" FALSE)
option(ENABLE_COVERAGE "Enable coverage reporting for gcc/clang" FALSE)
option(ENABLE_ASAN "Enable address sanitizer" FALSE)
option(BUILD_SHARED_LIBS "Enable compilation of shared libraries" FALSE)
option(ENABLE_TESTING "Enable the building of the test" FALSE)
option(ENABLE_CLANG_TIDY "Enable testing with clang-tidy" FALSE)
option(ENABLE_CPPCHECK "Enable testing with cppcheck" FALSE)
option(SIMPLE_BUILD "Build the project as minimally as possible" FALSE)
option(BUILD_DOC "Build the project's documentation" ON)
option(FORCE_COLORED_OUTPUT "Always produce ANSI-colored output (GNU/Clang only)." TRUE)
option(DEBUG_LOGGING "Enabling debug logging" FALSE)
```

I think the help strings are pretty self-explanatory so I won't go over each option individually. If you see any of these variables in later snippets, just know that it can be adjusted through the way the cmake is called when building the project. The options are passed as `-D` arguments to the cmake command. For example, to build with `DEBUG_LOGGING` enabled, we must call cmake like:

```sh
cmake .. -DDEBUG_LOGGING=TRUE
```


#### Targets {#targets}

Modern CMake is all about targets! The general rule of thumb is not to touch any variable in CMake (like `CMAKE_CXX_FLAGS`) directly but rather impose some requirements on a certain target. For the most part, the top-level CMakeLists.txt follows this paradigm. Targets can be defines as INTERFACE. This means that they don't produce any build output (neither library nor executable) but rather exist purely to be dependencies of other targets. Interface targets can be used, for example, to "contain" compile options. When an executable target is defined and it links against one such interface target, all of the compiler options imposed on the interface will also be imposed on the executable. This is my general idea that the CMakeLists.txt is structured around.

<br />

At the start, there are two INTERFACE targets defined -- `project_warnings` and `project_options`

```cmake
add_library(project_warnings INTERFACE)
add_library(project_options INTERFACE)
```

`project_warnings` is meant to keep track of the flags that instruct the compiler on what warning to report on. `project_options` is for every other flag that may be passed to the compiler.


#### Flags {#flags}

After the definition of the targets, several checks decide on the compiler flags that are to be used. The significant parts of the "building" of both targets are given in the following snippets.

<br /> For starter, we make sure that we are programming in C++17. C++17 is as good as it gets and it's the current year so, of course, we are going to use it for every personal project.

```cmake
target_compile_features(project_options INTERFACE cxx_std_17)
```

If coverage is enabled for the build, we set the appropriate flags for the compiler. Notice how we are defining the options only on the targets and not in "global scope" through the [CMAKE\_CXX\_FLAGS](https://cmake.org/cmake/help/latest/variable/CMAKE%5FLANG%5FFLAGS.html#variable:CMAKE%5F%253CLANG%253E%5FFLAGS) flag.

```cmake
if(ONLY_COVERAGE OR ENABLE_COVERAGE)
    target_compile_options(project_options INTERFACE --coverage -O0 -g)
    target_link_libraries(project_options INTERFACE --coverage)
endif()
```

We do something similar for the address sanitizers of the compiler:

```cmake
if(ENABLE_ASAN)
    target_compile_options(project_options INTERFACE -fsanitize=address)
    target_link_libraries(project_options INTERFACE -fsanitize=address)
endif()
```

While developing in C++, warnings are your friend. The more the better! Warnings can expose lots of tiny mistakes that you can make while writing C++ and in this sense, the compiler is your friend. As long as you tell it to report on the proper warning, of course.

```cmake
target_compile_options(project_warnings
    INTERFACE
    -Wall
    -Wextra
    -Wshadow
    -Wnon-virtual-dtor
    -Wold-style-cast
    -Wcast-align
    -Wunused
    -Woverloaded-virtual
    -Wpedantic
    -Wconversion
    -Wsign-conversion
    -Wnull-dereference
    -Wdouble-promotion
    -Wformat=2)

# some GCC specific warnings. These flags are added only if the used compiler is GCC.
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    target_compile_options(project_warnings
        INTERFACE
        -Wmisleading-indentation
        -Wduplicated-cond
        -Wlogical-op
        -Wuseless-cast
        )
    target_link_libraries(project_options INTERFACE stdc++fs)
endif()

```

A colorful output on the terminal is always useful.

```cmake
if (${FORCE_COLORED_OUTPUT})
    if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
        target_compile_options (project_options INTERFACE -fdiagnostics-color=always)
    elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
        target_compile_options (project_options INTERFACE -fcolor-diagnostics)
    endif ()
endif ()
```


#### Extra tools {#extra-tools}

Other than the compiler flags, several other external tools can help you in your C++ development. Many of them can be integrated with CMake. In my project template, I have three of them.

<br />

[CCache](https://ccache.dev/) is a compiler cache that speeds up recompilation. It is a separate program and you have to have it installed on your system. If this is the case, the following snippet will set up ccache in our build.

```cmake
find_program(CCACHE ccache)
if(CCACHE)
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE})
endif()
```

<br />

[Cppcheck](http://cppcheck.sourceforge.net/) is a static analysis tool for C++ code. I can help you catch some common mistakes while programming and it even doesn't require you to compile your code. If you have Cppcheck on your system and you've enabled it in your build, you can use it with CMake like this:

```cmake
if(ENABLE_CPPCHECK)
    find_program(CPPCHECK cppcheck)
    if(CPPCHECK)
    set(CMAKE_CXX_CPPCHECK
        ${CPPCHECK}
        --suppress=syntaxError
        --enable=all
        --inconclusive)
    else()
    message(SEND_ERROR "cppcheck requested but executable not found")
    endif()
endif()
```

<br />

[Clang-Tidy](https://clang.llvm.org/extra/clang-tidy/) is yet another tool for static analysis of C++ code. It can catch different set ot errors than cppcheck. As with the warnings, the more things the tools can tell us about our code, the better. The CMake integration is, again, possible and trivial:

```cmake
if(ENABLE_CLANG_TIDY)
    find_program(CLANGTIDY clang-tidy)
    if(CLANGTIDY)
    set(CMAKE_CXX_CLANG_TIDY ${CLANGTIDY})
    else()
    message(SEND_ERROR "clang-tidy requested but executable not found")
    endif()
endif()
```


#### Configurable header {#configurable-header}

In some situations, in your C++ code, you'll need information that is available only in the CMake build. This may include build information, version of the project, [endianness](https://en.wikipedia.org/wiki/Endianness) of the host system, a compile-time configuration of some sort, etc. Because of this, I figured out that I may need some sort of a "global" header file where such things are saved by CMake and accessible in the C++ code. My approach is to have the file `src/include/<project_name>/config.hpp.in` that will be configured by CMake and every file in the project would be able to include it. On the CMake side of the things, the code looks like this:

```cmake
set(PROJECT_VERSION_MAJOR 1)
set(PROJECT_VERSION_MINOR 0)
set(PROJECT_VERSION_PATCH 0)
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH})
...
configure_file (
    "${PROJECT_SOURCE_DIR}/src/include/${PROJECT_NAME}/config.hpp.in"
    "${PROJECT_BINARY_DIR}/src/include/${PROJECT_NAME}/config.hpp")

include_directories("${PROJECT_BINARY_DIR}/src/include")
```

Using a CMake function like `include_directories` is generally a bad practice but in this case, just for once, in a "global context", I am okay with it. In the `config.hpp.in` the file we can use all of the variables in the CMake environment to define whatever we want in the C++ code. To access a CMake variable, we use the `@<VARIABLE>@` syntax. For example, the file can look like something like:

```c++
#pragma once

#define PROJECT_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define PROJECT_VERSION_PATCH @PROJECT_VERSION_PATCH@

#define FILE_LOGGING 0
#define CONSOLE_LOGGING 1
#define DEBUG_LOGGING 1
```


#### Documentation building {#documentation-building}

One of my design goals for the project structure was to be able to handle building documentation. I achieve that through a custom target. In CMake we can specify a command to be executed (in the sense on a command on the terminal) and "bind" is to a [make target](https://stackoverflow.com/questions/2270643/what-is-a-make-target) in the final Makefile in the build directory. This means that at the end of the build process, I can execute something like `make doc` in the build directory to build the documentation for the project. Setting up Doxygen in CMake is not complicated. There is a package that can be included through `find_package`. This sets several CMake variables that are relevant to Doxygen. The whole setup can be done like:

```cmake
if(BUILD_DOC)
    find_package(Doxygen)
    if (DOXYGEN_FOUND)
                SET(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/doc/Doxyfile.in)
                SET(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

                configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)

                add_custom_target(doc ALL
                        COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
                        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
                        COMMENT "Buidling Doxygen documentation"
                        VERBATIM )
    else (DOXYGEN_FOUND)
                message("No doxygen binary found on the system.")
                SET(${BUILD_DOC} OFF)
    endif ()
endif()
```


#### Summery {#summery}

At the end of CMakeLists.txt, I've created a bunch of messages that show exactly hot the project is currently being build. It just prints out the options of the build and whether or not they are active or not.

```cmake
message("#########################################")
message("\t \t Summary")
message("#########################################")
message("Build type:       \t ${CMAKE_BUILD_TYPE}")
message("Install prefix:   \t ${CMAKE_INSTALL_PREFIX}")
message("Testing enabled:  \t ${ENABLE_TESTING}")
message("Clang-tidy:       \t ${ENABLE_CLANG_TIDY}")
message("Cppcheck:         \t ${ENABLE_CPPCHECK}")
message("Compiler:         \t ${CMAKE_CXX_COMPILER_ID}")
message("Sanizizers:       \t ${ENABLE_ASAN}")
message("Shared libs:      \t ${BUILD_SHARED_LIBS}")
message("Build libcpp:     \t ${LIBCPP}")
message("CCache executable:\t ${CCACHE}")
message("Building doc:     \t ${BUILD_DOC}")
message("------------------------------------------")
message("Version:          \t ${PROJECT_VERSION}")
message("########################################")
```


### Modules {#modules}

Let's now look at the individual submodules and how are those organized. As said, the `src/CMakeLists.txt` script includes them all through several calls of the `add_subdirectory` function.

```cmake
add_subdirectory(submodule_1)
add_subdirectory(submodule_1)
```

<br />

The folder structure of each module is the following:

```sh
├── CMakeLists.txt
├── data
├── include
│   └── project_name
│       └── module_name
│           └── main.hpp
├── src
│   └── main.cpp
└── tests
    ├── CMakeLists.txt
    └── test_main.cpp
```

The `src/` directory is meant for the source files as well as the private headers of the module. "Private headers" means that those won't be visible outside of the project. The public headers are meant to go in the `include/` directory. There is, however, a small caveat to its subfolder structure. In order to keep everything in my project organize, so that I can keep my sanity, I prefer to include the headers files in the form of:

```c++
#include "<project_name>/<submodule_name>/<header_file>"
```

In my mind, this is the most "logical" way to include something that is somewhere in the project as there is a clear hierarchy. This can be achieved by having several subfolders in the `include/` directory. The structure at the end is:

```sh

```

<br />

The CMakeLists.txt file for the modules is relatively simple. It just has to create a target (a library or executable), set up the include directories and then link it against the necessary other targets. In the top-level CMakeLists.txt, we've created the two INTERFACE targets `project_options` and `project_warnings`. Those are the "mandatory" ones to link against and every other target can be a one from the external library and\or a different submodule of the project. The whole CMakeLists.txt file looks like:

```cmake
# setting up sources
add_library(submodule_1_lib
    src/src_file_1.cpp
    src/src_file_2.cpp)

# setting up include directories
target_include_directories(submodule_1_lib
    PUBLIC include
    PRIVATE src)

# linking against the desired targets
target_link_libraries(submodule_1_lib
    PRIVATE
    project_options
    project_warnings
        PUBLIC
        clara::calra)

install(TARGETS submodule_1_lib
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/
    RUNTIME DESTINATION       ${CMAKE_INSTALL_BINDIR}
    LIBRARY DESTINATION       ${CMAKE_INSTALL_LIBDIR}
    INCLUDES DESTINATION      ${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/
    ARCHIVE DESTINATION       ${CMAKE_INSTALL_SHAREDSTATEDIR})
```

The last call to the `install` function instructs CMake on how to construct the _install_ target of the final Makefile. The function call basically makes all artifacts "go to the right place". I think that the most relevant part is that the header files will be copied to the global include directory (e.g. `/usr/include/`) but the will be put inside a folder with the project's name. In this case, another project can include headers from this project like:

```c++
#include <<project_name>/<submodule_name>/<header_file>>
```

which again helps to keep things organized.


### Testing {#testing}

The final aspect we have to look at is how to enable support for running C++ tests. Thankfully, CMake makes the integration incredibly easy. CMake uses [ctest](https://cmake.org/cmake/help/v3.15/manual/ctest.1.html) to discover and run tests I write the tests themselves with [Catch](https://github.com/catchorg/Catch2) -- a header-only, test framework for unit-tests. There is a nice startup guide for Catch [here](https://github.com/catchorg/Catch2/blob/master/docs/tutorial.md). The general idea is to create test executables with the help of Catch and then let ctest run them. To enable ctest support in a project its enough to call the `enable_testing` function in the top CMakeLists.txt file. In there I have:

```cmake
if(ENABLE_TESTING)
    enable_testing()
endif()
```

In the CMakeLists.txt files, I conditionally add the `test/` directory of each submodule. In there, there is a different CMakaLists.txt file that sets up the testing for the corresponding module. The script is nothing special and it is very similar to the upper-level one.

```cmake

add_executable(submodule_1_testThe final aspect we have to look at, is how to eable support for running C++ tests. Thankfully, CMake makes the integration incredibly easy. CMake uses [[https://cmake.org/cmake/help/v3.15/manual/ctest.1.html][ctest]] to discrover and run tests  I write the tests themselves with [[https://github.com/catchorg/Catch2][Catch]] -- a header-only, test framework for unit-tests. There is a nice start up guide for Catch [[https://github.com/catchorg/Catch2/blob/master/docs/tutorial.md][here]]. The genral idea is to create test executables with the help of Catch and then let ctest run them. To enable ctest support in a project its enough to call the ~enable_testing~ function in the top CMakeLists.txt file. In there I have:
#+BEGIN_SRC cmake
if(ENABLE_TESTING)
    enable_testing()
endif()
```

In the CMakeLists.txt files I conditionally add the `test/` directrory of each submodule. In therem, there is a different CMakaLists.txt file that sets up the testing for the corresponding module. The script is nothing special and it is very similar to the upper level one.

```cmake

add_executable(submodule_1_test
        submodule_1_test.cpp
    ../src/src_file_1.cpp
    ../src/src_file_2.cpp)

target_include_directories(${PROJECT_NAME}_alisp_test
    PUBLIC ../include
    PRIVATE ../src)

target_link_libraries(${PROJECT_NAME}_alisp_test
    PRIVATE
    project_options
    project_warnings
    PUBLIC
    Catch2::Catch2)

include(CTest)
include(Catch)
catch_discover_tests(submodule_1_test)
```

The last three lines turn this into a "test target". The build will thus produce a binary that can be ran and all the tests defined in it will be executed. As ctest was enabled, the whole process boils down to executing `make test` in the build directrory of the project. submodule\_1\_test.cpp ../src/src\_file\_1.cpp ../src/src\_file\_2.cpp)

target\_include\_directories(${PROJECT\_NAME}\_alisp\_test PUBLIC ../include PRIVATE ../src)

target\_link\_libraries(${PROJECT\_NAME}\_alisp\_test PRIVATE project\_options project\_warnings PUBLIC Catch2::Catch2)

include(CTest) include(Catch) catch\_discover\_tests(submodule\_1\_test) \#+END\_SRC

The last three lines turn this into a "test target". The build will thus produce a binary that can be ran and all the tests defined in it will be executed. As ctest was enabled, the whole process boils down to executing `make test` in the build directory of the project.


## Conclusion {#conclusion}

So those are my two cents about CMake and project structure. I don't claim to have a lot of experience but I've done a lot of research in the past year and a half. I've looked into different projects, read the best practices, read a lot of vague tutorials on the internet and watched the relevant talks. I have thought this several times, but <span class="underline">this</span> time I really think I've nailed it. I hope that I've created (mostly stolen) something scalable that will serve me well in my future small to mid-size projects. Whether of not scalability should be of my concern is a completely separate matter 🙂.


## References {#references}

-   [1] [CPP\_BOX](https://github.com/lefticus/cpp%5Fbox) is a project by [Jason Turner](https://github.com/lefticus).
-   [2] [Victoria Rudakova's Post](https://vicrucann.github.io/tutorials/quick-cmake-doxygen/)
-   [3] [Effective CMake](https://www.youtube.com/watch?v=bsXLMQ6WgIk) Talk by Daniel Pfeifer
-   [4] [Applied Best Practices](https://www.youtube.com/watch?v=DHOlsEd0eDE) Talk by [Jason Turner](https://github.com/lefticus)
-   [5] [CMake's documentation](https://cmake.org/cmake/help/v3.15/manual/ctest.1.html)
