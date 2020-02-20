+++
title = "Generic Execution Around Pointer"
author = ["Stanislav Arnaudov"]
description = "A set of generic classes for constructing the execution-around-pointer idiom"
keywords = ["c++", "templates", "idioms", "execution-around-pointer"]
lastmod = 2019-12-05T00:12:08+01:00
tags = ["c++", "templates"]
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

I recently found out what the [Execute-Around\_Pointer](https://en.wikibooks.org/wiki/More%5FC%252B%252B%5FIdioms/Execute-Around%5FPointer) idiom in C++ is. What it does is track access to a specific object. For example, when you want to observe how certain properties of an object change on each method call, you would employ the use of this idiom. Think of it like wrapping each method call for an object with additional function calls. This, for empale, is ugly code:

```c++
std::vector<int> vec{1,2,3,4};

std::cout << std::size(vec)
vec.push_back(5);
std::cout << std::size(vec)

std::cout << std::size(vec)
vec.push_back(6);
std::cout << std::size(vec)
```

With the `Execute-Around_Pointer` you can "prettify" this pattern by constructing a special "tracer" object that internally keeps a reference to the vector. The underlying object is then accessed through the `->` operator. The code then can become:

```c++
std::vector<int> vec{1,2,3,4};
VecotSizeTracer vec_tracer{vec};
vec_tracer->push_back(5);
vec_tracer->push_back(6);
```

On the wiki [page](https://en.wikibooks.org/wiki/More%5FC%252B%252B%5FIdioms/Execute-Around%5FPointer) there is a similar example. The implementation there is, however, non-genric and does not use any templated code. I decided to write several classes that realize the logic of the idiom. Those can be then used to quickly construct different tracers for different types of types.


## Making it generic {#making-it-generic}


### Iterating over tuple {#iterating-over-tuple}

First off, we'll need one utility method. Namely, `for_each` for iterating over a tuple. Weirdly enough, there is no such method in the standard library. It's not that hard to implement something ourselves though:

```c++
template<std::size_t I = 0, typename FuncT, typename... Tp>
inline std::enable_if_t<I == sizeof...(Tp), void>
for_each(std::tuple<Tp...> &, FuncT)
{ }

template<std::size_t I = 0, typename FuncT, typename... Tp>
inline std::enable_if_t<I < sizeof...(Tp), void>
for_each(std::tuple<Tp...>& t, FuncT && f)
{
    f(std::get<I>(t));
    for_each<I + 1, FuncT, Tp...>(t, std::forward<FuncT>(f));
};

```

Special thanks to [this](https://stackoverflow.com/a/6894436) StackOverflow answer. The function just recursively loops over the tuple by keeping track of the current index in the `I` template parameter. The recursion is terminated when `I` is equal to the size of the tuple. If you've programmed in Haskel, this should not be too hard for you.


### The proxy class {#the-proxy-class}

Next, we need that class that can keep a pointer to an object as well as several other objects that will be notified when the pointer is accessed.

```c++
template<typename T, typename...Tracker>
struct proxy
{
public:

    proxy(T& obj, Tracker && ... track) :
        m_obj(&obj),
        m_track(std::forward<decltype(track)>(track)...)
    {
        (track.before(obj), ...);
    }

    ~proxy()
    {
        detail::for_each(m_track, [&](auto& tracker){tracker.after(*m_obj);});
    }

    T* operator ->() { return m_obj;}
private:
    T* m_obj;
    std::tuple<Tracker ...> m_track;
};
```

The pointer is stored in the `m_obj` member. The notified objects are stored in the `m_track` tuple. Those can be of different types but must implement a `before` and a `after` method. The `before` methods of each object will be called before any operation on the object pointed by `m_obj` is performed, and the `after` method -- after. We call the `after` method in the destructor by using our `for_each` function that we previously implemented. In the constructor, I've simply used [pack expansion](https://arne-mertz.de/2016/11/more-variadic-templates/).


### The Tracer {#the-tracer}

Now we have every that we need to write the final tracer class:

```c++
template<typename T, typename ...Tracker>
class Tracked : public Tracker ...
{
  public :
    explicit Tracked(T& obj) : m_obj(obj)
    {}

    detail::proxy<T, Tracker...> operator ->() {
        return detail::proxy<T, Tracker...>(m_obj, static_cast<Tracker>(*this)...);
    }

  private :
    T& m_obj;

};
```

On construction, the `Tracked` class takes the object that needs to be traced as well as several types -- the `Tracker` variadic template argument. The `Tracker` class will be derived from each of the passed in types. This means that it will "have" all of the methods defined by the `Tracker` types. In the overload of the `->` operator, we create a proxy object with the underlying object and statically cast version of the `this` object with each of the given `Tracker` types. This is possible because we've derived the `Tracked` from each of those types.


### Usage {#usage}

With our defined classes, we can realize the logic of the Execute-Around-Pointer idiom in an easy manner. For example, if we want to track the size of a vector we can write something like:

```c++
// class with before and after methods
class SizeTracker
{
    public:
    static void before(std::vector<int>& vec) {
        std::cout << "Size Before:" << std::size(vec) << "\n";
    }

    static void after(std::vector<int>& vec) {
        std::cout << "Size After:" << std::size(vec) << "\n";
    }
};

// tracker type for vector of ints.
using VectorSizeTracker = Tracked<std::vector<int>, SizeTracker>;
```

With this, we can rewrite the example that I gave at the beginning of the post.

```c++
int main()
{
    std::vector<int> vec{1,2,3,4};
    VectorSizeTracker vec_track{vec};

    vec_track->push_back(5);
    vec_track->push_back(6);

    return 0;
}
```
