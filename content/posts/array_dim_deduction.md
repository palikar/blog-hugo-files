+++
title = "Multidimensional Array size deduction in C++"
author = ["Stanislav Arnaudov"]
description = "A workaround for deducing the dimensions of multidimensional array in C++."
date = 2019-10-31T00:00:00+01:00
keywords = ["c++", "templates", "array", "templete-type-deduction", "c++-17"]
lastmod = 2019-10-31T12:08:47+01:00
tags = ["c++", "programming"]
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

Recently I encountered a small particularity of the C++ standard that I've known about previously but didn't really care about up this point. The whole thing has to do with template type parameters. It turns out that C++ can't deduce the type of a braced initializer. This has some funny consequences when you are designing a function that can take "everything" including arrays and you want to be able to call the function as `function({1,3,4,5})`. The desired outcome is that `{1,3,4,5}` is deduced to a one-dimensional array but at the same time `{{1,3,4,5}}` is two-dim array, `{{{1,3,4,5}}}` is a three-dim array and so forth. Turns out, this is a bit tricky.


## Problem {#problem}

The exact details of my situation are not important. The gist of it is as follows.

<br />

We have a templated function `value` that can take "everything" and "pipes" it's inputs to other helper functions. The `value` function is simply defined as:

```c++
template<typename... Args>
auto value(Args&&... args)
{
    return APIHelper::get(std::forward<Args>(args)...);
}
```

`APIHelper` is just a struct with a whole bunch of static functions, all named `get` and each function is responsible for some kind of input that can be passed to `value`. The `get` function creates some object and returns it. In this sense, the `value` function can be used to create an object in arbitrary ways. In my case, I wanted one of those cases to be through a multidimensional array. As mentioned, however, the following calls can't be used as the template arguments can't be deduced.

```c++
//calling value with one-dimensional array
value({1,3,4,5}); // cant't be deduced

//calling value with two-dimensional array
value({{1,3,4,5},
       {1,3,4,5},
       {1,3,4,5}}); // cant't be deduced
```

As far as I understand, this is because of some weird specification of the C++11 standard. `§ 14.8.2.5/5` says:

> A function parameter for which the associated argument is an initializer list (8.5.4) but the parameter does not have std::initializer\_list or reference to possibly cv-qualified std::initializer\_list type. [Example:
>
> template<class T> void g(T); <br /> g({1,2,3}); // error: no argument deduced for T
>
> — end example ]

This essentially boils down to the fact that we just can't call a function as `value({...})` and expect reasonable type deduction.

<br />

A possible workaround for my problem is to define an overload of `value` that can accept a one-dimensional array. This can be done like:

```c++
template<typename T, size_t N>
auto value(const (&values)[N])
{
    return API::get(std::forward<Args>(values)...);
}
```

This definition even deduces the size of the passed array. This is great and all but it doesn't work with two-dimensional arrays.

```c++
value({1,3,4,5}); // deduces OK
value({{1,3,4,5},
       {1,3,4,5},
       {1,3,4,5}}); // cant't be deduced
```

For two-dimensional arrays, we have to define another function. And another one for a three-dimensional array. And then another one and so forth.


## Solution {#solution}

I researched the whole topic of multidimensional array dimensions deduction for a good couple of hours but I couldn't find anything. My conclusion, for now, is that this cannot currently be done, even with C++17. For this reason, I had to come up with something on my own.

<br />

My solutions is the following. In the `APIHelper` struct I have a function defined as:

```c++
struct APIHelper
{
template<typename T, size_t...N>
inline static auto get(const T* values)
{
    constexpr size_t SIZE = (N * ... );
    for (size_t i = 0; i < SIZE; ++i) {
        values[i]; // doing something with values
    }
    return 0;
}
};
```

This function should handle all object constructions involving an array. The template pack `N` should contain the dimensions, `T` is the type of the array and `values` is a pointer to the first element of the array. `SIZE` is constant that tells us how many elements there are in the array. If everything is used correctly, `values[SIZE-1]` should be safe to access.

<br />

We now have to look at the `value` function. It should be able to handle all sorts of multidimensional arrays and forward everything to the `APIHelper::get` in the right format. The function should have some the general form of:

```c++
template<typename T, size_t N>
inline static auto value(const T(&values)[N])
{
    const T* s = reinterpret_cast<const T*>(&values);
    return API::get<T, N>(s);
}
```

This works just fine for one-dimensional arrays. For higher dimensions, we have to define more functions. Defining lots of things per hand is tedious. Thankfully, C++ provides as with mechanism for saving typing -- the [C preprocessor](https://en.wikipedia.org/wiki/C%5Fpreprocessor). Using macros is generally considers a bad programming practice but in some situations, I say it is justified. This seems like one of those.

<br />

I spend around 15 minutes on [Compiler Explorer](https://godbolt.org/) (you can test only macro expansions with the -E flag for GCC) and I came up a this set of definitions.

```c++
#define SIZE_T_S_0 size_t N
#define SIZE_T_S_1 SIZE_T_S_0 , size_t M
#define SIZE_T_S_2 SIZE_T_S_1 , size_t L
#define SIZE_T_S_3 SIZE_T_S_2 , size_t K
#define SIZE_T_S_4 SIZE_T_S_3 , size_t J
#define SIZE_T_S_5 SIZE_T_S_4 , size_t I
#define SIZE_T_S_6 SIZE_T_S_5 , size_t H
#define SIZE_T_S_7 SIZE_T_S_6 , size_t F

#define BRACKETS_S_0 [N]
#define BRACKETS_S_1 BRACKETS_S_0[M]
#define BRACKETS_S_2 BRACKETS_S_1[L]
#define BRACKETS_S_3 BRACKETS_S_2[K]
#define BRACKETS_S_4 BRACKETS_S_3[J]
#define BRACKETS_S_5 BRACKETS_S_4[I]
#define BRACKETS_S_6 BRACKETS_S_5[H]
#define BRACKETS_S_7 BRACKETS_S_6[F]

#define LETTERS_S_0 N
#define LETTERS_S_1 LETTERS_S_0, M
#define LETTERS_S_2 LETTERS_S_1, L
#define LETTERS_S_3 LETTERS_S_2, K
#define LETTERS_S_4 LETTERS_S_3, J
#define LETTERS_S_5 LETTERS_S_4, I
#define LETTERS_S_6 LETTERS_S_4, H
#define LETTERS_S_7 LETTERS_S_5, F

#define VALUE_FUN(dim) template<typename T, SIZE_T_S_##dim>        \
    inline static auto value(const T(&values) BRACKETS_S_##dim)     \
    {                                                               \
        const T* s = reinterpret_cast<const T*>(&values);       \
        return API::get<T, LETTERS_S_##dim>(s);                 \
    }

```

The macro can then be used several times to generate the functions for arrays of different dimensionality.

```c++
VALUE_FUN(0) // one-dim array
VALUE_FUN(1) // two-dim array
VALUE_FUN(2) // three-dim array
VALUE_FUN(3) // four-dim array
VALUE_FUN(4) // five-dim array
VALUE_FUN(5) // six-dim array
VALUE_FUN(6) // seven-dim array
VALUE_FUN(7) // eight-dim array
```

There. Now we can handle up to 8-dimensional arrays. This should be enough for know. If more dimensions are need, we just have to add more definitions of `SIZE_T_S_*`, `BRACKETS_S_*` and `LETTERS_S_*`
