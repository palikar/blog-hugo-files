+++
title = "Compile time pipeline in C++"
author = ["Stanislav Arnaudov"]
description = "A simple but powerful idiom in C++ for pipelining functions"
date = 2019-04-10T00:00:00+02:00
keywords = ["c++", "templates", "idioms"]
lastmod = 2019-10-21T22:17:57+02:00
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

Recently I've been looking into the code of [ChaiScript](https://github.com/ChaiScript/ChaiScript/) -- a scripting language written by [Jason Turner](https://github.com/lefticus) that integrates seamlessly with C++. It's a great project and you can learn a ton on parsing, general scripting language design and overall C++ from it. In its code, I've found several gems and idioms that I didn't know before. Here I want to very briefly present a certain pattern that particularly caught my eye. The pattern addresses the problem of piping functions one after another. We cat assume that each function does its own thing independent of the others. The beauty, in my opinion, lies within the fact that there is almost no run time overhead and the whole thing is "configured" at compile time. The ability to extend or alter the pipeline is, however, still there and a change can be easily made (after recompilation, of course).


## Pipelining methods {#pipelining-methods}


### The "first thing in google" way {#the-first-thing-in-google-way}

I was interested in this pattern so I tried searching for it. With a quick google search, I found this [this](https://helloacm.com/how-to-pipeline-the-functions-in-c/). The blog post describes a way of generating a pipeline of functions that will be executed one after another. To note is, however, that in this case everything is done at run time. The post describes a function similar to:

```c++
template<class T>
std::function<T (void)> make_pipeline(const std::vector<std::function<T (void)>>& funcs)
{
    return [&funcs] (const T& arg)
           {
               T v = arg;
               for (const auto &f: funcs) { v = f(v); }
               return v;
           };
}
```

This is just a function that returns a lambda that will execute every function in the `funcs` vector. If you are not familiar with template programming, I believe this is the most intuitive way of achieving your pipeline-goal. `make_pipeline` can then be used as:

```c++
std::vector<std::function<int (void)>> functions;
functions.push_back([](int x){ return api.do_work_1(x); });
functions.push_back([](int x){ return api.do_work_2(x); });
functions.push_back([](int x){ return api.do_work_3(x); });

std::function<int (void)> func = makePipeline(functions);

func(10);
```

Now, again at run time, the process has to create a vector, add the three lambdas to it, then construct the final function `func` with the `makePipeline` method and finally call the function. The whole thing introduces a lot of run time overhead. Before performing the actual call to the pipeline, we must configure it. It doesn't necessarily need to be this way and in the next section we'll how we can address this problem.


### The ChaiScript way {#the-chaiscript-way}

Let's now look the ChaiScript's "compile-time" approach to pipeline. A little bit of background - we'll look at the optimizer of ChaiScript. This is class with a single method `optimize` that takes an object of type `AST_Node_Impl_Ptr<Tracer>`, does its magic and returns a new, optimized object. What exactly `AST_Node_Impl_Ptr<Tracer>` is not the point here. The main thing is - call `optimize`, perform a bunch of optimizations, return the object. The tricky thing is that we want a whole bunch of optimizers to be available. Then, when we call `optimize` on our main, container-like, optimizer, every inner is run.

<br />

The main Optimizer class looks like this:

```c++
template<typename ... T>
struct Optimizer : T...
{
    Optimizer() = default;
    explicit Optimizer(T ... t)
        : T(std::move(t))...
    {
    }

    template<typename Tracer>
        auto optimize(eval::AST_Node_Impl_Ptr<Tracer> p) {
        ((p=static_cast<T&>(*this).optimize(std::move(p))), ... );
        return p;
    }
};
```

A class templatized by a variadic template. As per the [The Curiously Recurring Template Pattern (CRTP)](https://www.fluentcpp.com/2017/05/12/curiously-recurring-template-pattern/), the class inherits every one of the passed template types. This means that the `Optimizer` class will have every public method of its template parameters. In the `optimized` method, the `...` operator is used. This is variadic template expansions and can be used to operate on every passed template parameter. In this case, cast `this` to a reference of the parameter (this is ok as we inherit from it), call the `optimize` method on this object and saves the result in `p`. All we have to ensure that each template parameter `T` has an `optimize` method with the same(ish?) signature. For example, we can have two optimizers like:

```c++
struct Optmizer1
{
    template<typename T>
    auto optimize(eval::AST_Node_Impl_Ptr<T> node)
    {
        //magic 1
        return node;
    }
};

struct Optmizer2
{
    template<typename T>
    auto optimize(eval::AST_Node_Impl_Ptr<T> node)
    {
        //magic 2
        return node;
    }
};

```

At the end, everything comes together in a `typedef` for the final optimizer type:

```c++
typedef Optimizer<Optimizer1,
                  Optimizer2> Optimizer_Default;
```

An we that we have it. Now we can just create a new object of type `Optimizer_Default` and call `optimize` on it. This will call the optimize method on `Optimize1` and `Optimize2`. The pipeline is constructed at compile time and the compiler can optimize everything optimally.

<br />

With this setup, we can change the executed optimizers only at compile time though. This means that every change will need a rebuild of the project. Although we cannot avoid the rebuilding, we can make out lives easier with a few macros. Say, if the project is compiled with defined `HEAVY_OTPIMIZATION` macro, we could construct our optimizer one way, if the macro is not defined, we construct it another way.

```c++
#ifdef HEAVY_OTPIMIZATION
typedef Optimizer<Optimizer1,
                  Optimizer2> DefaultOptimizer;
#else
typedef Optimizer<BigOptimizer1,
                  BigOptimizer2> DefaultOptimizer;
#endif
```


## Conclusion {#conclusion}

I don't have that much closing words. I can only say that variadic templates are one of the things that truly make C++ an interesting language. That one quote really stands true.

> Dot dot dot is where fun begins <br /> <br /> -- Someone, Somewhere
