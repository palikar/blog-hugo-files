+++
title = "C++ things to think about while programming"
author = ["Stanislav Arnaudov"]
description = "Several tips that I've explained to myself after I read Effective Modern C++"
date = 2018-07-27T00:00:00+02:00
keywords = ["c++", "programming", "code", "type-deduction", "rvalue", "lvalue", "template"]
lastmod = 2019-10-21T22:20:24+02:00
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

C++ is hard! C++ is even harder when you want to write good code. There are a lot of things to think about when you write code and if you don't think about them, you are probably going to mess things up. Recently I've found that one good book (see references) that gives 42 (hehe!) concrete tips on how to be a better c++ programmer. This is my summary of sorts about the contents of said book.


## The cool new things {#the-cool-new-things}


### Rvalue, lvalue and move semantics {#rvalue-lvalue-and-move-semantics}

<a id="org1edb870"></a> "Move semantics" is a thing that we have in C++ now (since C++11 actually). It makes a ton of sense once you get it but it may be a little hard to wrap your head around it. At least in the beginning, of course. First off - why do we need move-semantics? For code efficiency! If you know what you are doing, your code can run faster, with fewer object copies and fewer objects created. Second, and perhaps more importantly, what are move semantics? The way I like to think about it - the ability to distinguish between an object that won't be needed after the evaluation of an expression and the ability to do a different thing, if you are dealing with such object.<br /> Take a look at the following code:

```c++
MyObject obj1, obj2;
MyObject new_obj = obj1 + obj2;
```

Let's say that the part `obj1 + obj2` creates a new object. This new object then gets copied through the copy assignment operator of the class `MyObject`. This creates another object that is stored in the `new_object` variable. All in all, two objects were created in this situation. The object that was originally created by `obj1 + obj2` is... well, gone. We just copied it and threw it away. Shame on us! Wouldn't it have been nice, if we could have just used that object and just "moved" it in `new_obj`. Yes, yes it would! And yes, it is possible. Through move semantics. That kind of object that won't continue to live after the expression evaluation is called **rvalue** object. All other objects are **lvalue**. In the class `MyObject` we can define a special kind of constructor (move constructor) that "creates" object from rvalues, i. e. "moves" the data from the given object in **\*this**. So,

-   Lvalue - an object that can be used on the left-hand side of expression; an object with identifiable address;
-   Rvalue - an object that won't persist after the evaluation of a given expression; no identifiable address

_Note:_ Every value is either an lvalue or an rvalue!


### Auto {#auto}

Type deduction for variables. Almost the same rules as the template argument type deduction with one small caveat. Also, in a lot of cases, it allows you to write more efficient code. Example to get you going:

```c++
std::unordered_map<std::string, std::string> map;
for(auto it : map){...} //just look how simple that is!!!
```


### Smart pointers {#smart-pointers}

<a id="orgdc1a8fb"></a> For an year now, I only hear how the C++ gurus scream how raw pointers are dangerous and will probably cause memory leaks when used so... smart pointers! For the price of a tiny bit of overhead, you will make harder (but not impossible!) for yourself to do something stupid with your code. Your new two best friends are `shared_ptr<>` and `unique_ptrt<>`:

```c++
std::shared_ptr<int> int_ptr = std::maked_shared<int>(2);
std::unique_ptr<float> float_ptr = std::make_unique<float>(3.3);
```

When it comes to `unique_ptr`, it's important to know how does one transfer the ownership of an object.

```c++
void take_ownership(std::unique_ptr<float> ptr)
{
  ... // now the pointer is mine!
}
std::unique_ptr<int[]> int_arr_ptr = std::make_unique<int[]>(10);
take_ownership(std::move(int_arr_ptr));
```

`shared_ptr`, on the other hand, can share their ownership. Again, it's good to know how is this done because you can mess it up.

```c++
class PtrHolder {
public:
  void save_ptr(std::shared_ptr<int> ptr)
    {
      this->ptr_ = ptr;
    }
private:
  std::shared_ptr<int> ptr_;
}
int main(int argc, char *argv[])
{
  Ptrholder obj;
  std::shared_ptr<int> int_ptr = std::maked_shared<int>(2);
  obj.save_ptr(int_ptr); // know there are two references to the int of int_ptr
}
```

**Important:** <span class="underline">Do not</span> return or take smart pointers by reference! <br /> Creating factory methods is also relatively straightforward. Just return them by value! There is however one things to remember about `unique_ptr` - if the return type does not match the thing you are returning, you ought to use `std::move`. This holds true even, if the return of the function type is a base class and you are returning derived class (explained [here](https://stackoverflow.com/questions/39478956/how-does-returning-stdmake-uniquesubclass-work/39479117)).


### Some random abbreviations {#some-random-abbreviations}

It is a well-known fact that C++ programmers love their abbreviations. And you know what, it actually makes a lot of sense to know those. The abbreviations in the C++ world reveal some cool, useful and right out elegant concepts that everyone should know about. Also, just to be prepared, C++ programmers are really bad at naming things.

| Abbreviations | Expansion                               |
|---------------|-----------------------------------------|
| RAII          | Resource acquisition is initialization  |
| SFINAE        | Substitution failure is not an error    |
| Pimple        | Pointer to implementation               |
| CRTP          | Curiously recurring template pattern    |
| IIFE          | Immediately invoked function expression |
| RVO           | Return value optimization               |


## Tips {#tips}


### Knowing your template type deduction is a blessing. {#knowing-your-template-type-deduction-is-a-blessing-dot}

So every C++ programmer knows how you can define generic "templated" functions and then you can call them with different types of arguments. The calling itself will cause the compiler to instantiate the function by replacing the generic type with the deduced type. The thing you should know in this process - how the type is being deduced when it's not given explicitly (i. e.the function is called without the <>-brackets). <br /> So, based on `T`, `ParamType` and `expr` you have to know what type would be deduced in

```c++
template<T>
void f(ParamType arg);

f(expr);
```

There are several cases to look at.


#### ParamType is a reference or a pointer {#paramtype-is-a-reference-or-a-pointer}

```c++
template<T>
void f(T& arg);
```

In this case, if `expr` is a reference type, the reference part is ignored and the rest of the type is taken verbatim.

```c++
int x = 42;
const int cx = x;
const int& rx = cx;

f(x); // T is is int

f(cx); // T is const int

f(rx); // T is again const int
```

_Note:_ This means that depending on how `f` is called, this may not compile

```c++
template<T>
T f(T& arg)
{
  T new_var = arg;
  new_var += 5; // if T is const int, this line here will fail
  return new_var;
}
```

<br /> If we now change the definition of `f` to

```c++
template<T>
void f(const T& arg);
```

the things to be "ignored" during the deduction of T are the _reference_ part <span class="underline">and</span> the _const_ part. This means that in the above examples, T will be deduced to _int_ every time.


#### ParamType is a Universal Reference {#paramtype-is-a-universal-reference}

Here we are getting a little bit fancier with come cool c++11 features. We define **f** like:

```c++
template<T>
void f(T&& arg);
```

In this case

-   if _expr_ is a lvalue, **T** and **ParamType** are deduced to be lvalue reference. This is the only case where **T** is deduced top be a reference.

```c++
f(x); // T is int&

f(cx); // T is cont int &

f(rx); // T is cont int &
```

-   if _expr_ is a rvalue - the rules for reference and pointer apply.

```c++
f(27); // T is int
```


#### ParamType is neither a reference, nor a pointer {#paramtype-is-neither-a-reference-nor-a-pointer}

This is just pass by value.

```c++
template<T>
void f(T arg);
```

By instantiation we ignore everything except the "pure type" (i.e. _reference_, _const_ and/or _volatile_ part are ignored)


#### Bonus {#bonus}

Array arguments decay to pointers so when passing arrays to a template function, the `T` will be deduced with the rules for pointers.<br /> One can, however, define e reference to an array so with this "trick" you can force your `T` to be deduced to an array type.

```c++
template<T>
void f(T& arg);

int arr[13];
f(arr) // T is int[13]
       // and ParamType is int(&)[13]
```

"int(&)[13]" is a reference type to an array with 13 elements... myeah! With such references to arrays you can write this extraordinary function that will deduce the number of elements in an array at compile time

```c++
template<typename T, std::size_t N>
constexpr std::size_t arraySize(T (&)[N]) noexcept
{
  return N;
}
```

And... with that knowledge, you can now punish people who claim they "know C++".


### auto type deduction is also something to thinks about {#auto-type-deduction-is-also-something-to-thinks-about}

The deduction of auto while used as a "type" of a local variable behaves almost exactly as the deduction of template types. This means you already know most of the rules.

```c++
int x = 5;
const int cx = x;
const int& rx = cx;

//case 1

auto& xx = x; //the auto is int
auto& xcx = cx; //the auto is const int
auto& xry = rx; //the auto is const int

const auto& xx = x; //the auto is int
const auto& xcx = cx; //the auto is int
const auto& xry = rx; //the auto is int


//case 2

auto&& xx = x; //the auto is int&
auto&& xcx = cx; //the auto is const int&
auto&& xry = rx; //the auto is const int&
auto&& x_rvalue = 27; // the auto is int

.
.
.
```

So yeah, not much new things here. Just think about what is becoming with the _auto_ in the declaration of the variable. The final type may not be the same as the thing deduced for _auto_. For that, you'll have to replace the deduced type for the _auto_ in the declaration. <br /> There is however one caveat with _auto_ type deduction. Consider the snippet:

```c++
auto x1(27);
auto x2{27};
```

This compiles but the types of the variables are probably not what you expect. Both statements look the same and while the **x1** is "well behaved" and to be expected (it is an int...surprise!), the type of x2 is `std::initializer_list<int>`. Yes, _auto_ treats bracketed expressions differently and the general deduced type for them is `std::initializer_list<T>`. This means that the following code won't compile

```c++
auto x{12, "random string"};
```

If you try using the bracketed expression with a templated function on the other side, it just won't compile even if the objects in the brackets are all of the same types. Template type deduction just cannot handle bracketed expressions.


### decltype is cool little thing {#decltype-is-cool-little-thing}

At its core `decltype` has a simple concept. It takes a single argument and it "returns" its type. The quotation marks are there because the thing returned thing can be used further as a part of the code. For example, you can declare a new variable with given deduced from `decltype` type. This is possible:

```c++
int x = 5;
decltype(x) xx = 23; // xx now has the type of x.... just plain int
```

This means, however, that `decltype` introduces a whole new set of rules for deducing a type from an expression. Thankfully, those rules are pretty simple and are what you expect...mostly as I understand it, `decltype` gives you exactly the type that was used when declaring the argument. It returns all the "reference-y" and "const-y" parts. Everything! <br /> The primary use of `decltype` is for specifying a return type of a function that depends on the type of the incoming arguments. Imagine we want to write a generic function that accesses a given index of a given container and returns the object at the index while before that...it does "something". In this situation, you can't know the type of the object at the index(its constness, its referenceness...). `decltype` to the rescue. Check this out:

```c++
template <typename Container, typename Index>
auto doSomethingAndAcess(Container& c, Index i) -> decltype(c[i])
{
  doSomething();
  return c[i];
}
```

The **auto** at the beginning has nothing to do with auto type deduction. It just delegates the specification of the return type of the function to the end where we have access to the parameters. I believe the whole concept is called _trailing return type_. <br /> This is, however, the C++11 way of doing it. C++14 offers a sprinkle of syntactic sugar to "turn on" `decltype`-deduction rules for auto. Namely:

```c++
template <typename Container, typename Index>
decltype(auto) doSomethingAndAcess(Container& c, Index i)
{
  doSomething();
  return c[i];
}
```

This way the type of the return statement will be used as a return type and it will be done in a `decltype`-y kind of way.


### Prefer auto to explicit type declarations {#prefer-auto-to-explicit-type-declarations}


#### General advantages of `auto` {#general-advantages-of-auto}

-   The first and obvious advantage of `auto` is that it spares a lot of typing. This, by extension, allows you to even not remember the types of certain things. Like... at this point who even knows what are the complete types of certain iterators! So, things like

    ```c++
    template<typename It>
    void dwim(It b, It e)
    {
      while (b != e)
      {
        typename std::iterator_traits<It>::value_type var = b*;
        b++;
      }
    }
    ```

    become

    ```c++
    template<typename It>
    void dwim(It b, It e)
    {
      while (b != e)
      {
        auto var = b*;
        b++;
      }
    }
    ```

    Awesome!
-   With `auto` you can define types that are known only to the compiler. Lambdas! Also, since C++14 you can also use `auto` with lambada's parameters.

    ```c++
    auto derefLess = [](const auto& p1, const auto& p2){return *p1 < *p2};
    ```
-   `auto` forces you to initialize your variables which is almost always something you should do.

```c++
int x1; //uninitialized..bad!
auto x2; // wont't compile!
auto x3 = 0; //good!
```

-   `auto` prevents you from using the wrong type. The official return type of `std::vector<T>::size()` is `std::vector<int>::size_type` but a lot of time it is approximated with _unsigned_. This may cause problems in certain situations. The problems go away if you just use `auto` for the type of the variable holding the return value of `std::vector<>::size()`.
-   _foreach_-loops become easier to type <span class="underline">and</span> you cannot mess up the type of the iterated elements. **Always** use something like:

```c++
std::unordered_map<std::string, int> m;
...
for (const auto& e : m )
{
  // e has the "most possible correct" type here
}
```


#### Explicitly typed initializer idiom {#explicitly-typed-initializer-idiom}

Sometimes `auto` won't give you the type you desire. In those situations, it's preferable to cast the thing you are assigning to a variable to the desired type and continue with the use of auto.

```c++
auto ep = static_cast<float>(calcEpsilon()); // explicitly reducing double to float... good!
```


### nullptr is a pointer to nothing, 0 and NULL are not that {#nullptr-is-a-pointer-to-nothing-0-and-null-are-not-that}

`O` and `NULL` sound so logical but they are not what you probably think they are. `0` is an _int_. If the compiler sees `0` in the context of a pointer it will be interpreted as the null-pointer. There are, however, many situations where the context is not that clear. In a lot of cases, `0` will be treated as a simple int. <br /> The same story with `NULL`. Depending on the implementation it is usually some integral type and it will be treated as a number in situations where you don't expect it to behave like this. <br /> All problems can be solved, if you just forget about the existence of `NULL` and `0` and start using `nullptr`. `nullptr` is designed to be a pointer to nothing and pointer is the only way it can be interpreted. The following snippet demonstrates everything:

```c++
void f(void*);
void f(int);
void f(bool);

f(NULL); // calls f(int)

f(0); // calls f(int)

f(nullptr); // correctly calls f(void*)
```

Also, another added bonus to the usage of `nullptr` is that it is the only thing that gets properly deduced with template functions.

```c++
void f(void*);
template<typename Func,
         typename Mux,
         typename Ptr>
decltype(auto) loackAndCall(Func func, Mux& mutex, Ptr ptr)
{
  MuxGuard g(mutex);
  return func(ptr);
}

lockAndCall(f1,f1m, 0);       // error!
lockAndCall(f1,f1m, NULL);    // error!
lockAndCall(f1,f1m, nullptr); // finex
```


### Alias declarations are better than typedef {#alias-declarations-are-better-than-typedef}


#### What even I am talking about {#what-even-i-am-talking-about}

Brief refresher. This is `typedef`:

```c++
typedef std::unique_ptr<std::vector<int>> VecPtr;
```

And this is the new cool way of doing the same thing using `using` (alias declaration)

```c++
using VecPtr std::unique_ptr<std::vector<int>>;
```


#### Advantages of `using` {#advantages-of-using}

-   When dealing with types involving function pointers, `using` is much easier to swallow.

```c++
// which one of those look like the easier to type out and remember

typedef void (*FP)(int, const std::string&);

using FP = void (*) (int, const std::string&);
```

-   Alias declarations can be templetized while `typedef` cannot. If you want to write template type with `typedef`, you'll have to define some underling _struct_. `using` does not suffer from the same problem.

```c++
template<typename T>
using MyAllocList = std::list<T, MyAlloc<T>>

// VS.
template<typename T>
struct MyAllocList {
  typedef std::list<T, MyAlloc<T>> type;
}
```

It gets even worse when you want to use the type defined with `typedef`. Then you'll have to use the trailing `::type` after the type.

```c++
template<typename T>
class Widget
{
private:
  typename MyAllocList<T>::type list;
}
```

Do yourself a favor, use `using`!


### Type transformations that come in handy {#type-transformations-that-come-in-handy}

Sometimes you want to "cook" yourself some type from some already given template types. For those purposes the standard library offers some predefined type transformations:

```c++
std::remove_const<T>::type;
std::remove_const_t<T>;

std::remove_reference<T>::type;
std::remove_reference_t<T>;

std::add_lvalue_reference<T>::type;
std::add_lvalue_reference_t<T>;

...
```


### Deleted functions are to be used instead of private ones {#deleted-functions-are-to-be-used-instead-of-private-ones}

In some cases what you want is to prevent certain functions from being called from user code. In the good old day programmers just defined such functions private. The smarter way to do the same thing nowadays is to delete the function. This can even be done from a derived class that wants to "hide" some of the functions from its base class.

```c++
class basic_ios: public ios_base
{
public:
  basic_ios(const basic_ios&) = delete;
  basic_ios& operator=(const basic_ios&) = delete;
}
//this deletes the copy constrctor and the assgiment operator but only for the derived class
```

By convention deleted functions are to be declared _public_ and not _private_. <br /> Also <span class="underline">any</span> function could be deleted, not only member functions of a given class. With deletion you could "turn off" certain overloads of functions.

```c++
bool isLucky(int number);
bool isLucky(char) = delete;
bool isLucky(bool) = delete;
bool isLucky(double) = delete;

if(isLucky(2)){...} // fine
if(isLucky('2')){...} // error!
if(isLucky(true)){...} // error!
if(isLucky(2.5)){...} // error!
```

Without the deletions the function calls will compile and may not behave the way you expect them to because of the implicit conversations to _int_.


### Use override {#use-override}


#### Virtual functions {#virtual-functions}

Refresher again. Which functions are to be considered virtual.

-   the base class function must be declared virtual.
-   The base and derived function names must be <span class="underline">identical</span>.
-   Parameter types of the baser and derived function must be <span class="underline">identical</span>.
-   The _constness_ of the base and derived functions must be <span class="underline">identical</span>.
-   The return types and exception specifications of the base and derived functions must be <span class="underline">compatible</span>

Inevitably you will forget those rules and you will think that you are overriding something in a "virtual" way but you really won't be doing that. So, the advice goes, use `override` to be explicit. Then you'll have compiler tell you've done something wrong.

```c++
class Base
{
private:
  virtual void mf1() const;
  virtual void mf2(int x);
  virtual void mf3() &;
  void mf4() const;
}

class Derived : public Base
{
private:
  virtual void mf1() override;               // won't compile
  virtual void mf2(unsigned int x) override; // won't compile
  virtual void mf3() && override;            // won't compile
  virtual void mf4() const override;         // won't compile
}
```


#### Overloading on rvalue and lvalue {#overloading-on-rvalue-and-lvalue}

You can also overload a function based on weather the `*this` object is a r- or lvalue:

```c++
class Widget
{
public:
  ...
  std::vector<double>& data() & // for lvalue Widgets
    {
      return values; // return lvalue
    }

  std::vector<double> data() && // for rvalue Widgets
    {
      return std::move(values); // return rvalue
    }
};

```


### Think when you need const\_iterator and when iterator {#think-when-you-need-const-iterator-and-when-iterator}


#### Const iterators {#const-iterators}

Not that much to say. C++ can optimize the code much better if it deals with _constness_. Remember to put `const` before `auto` when you don't need to change the objects that you iterate.

```c++
for(const auto& e : container){}
```

Also, when you use a function that takes iterators to perform something that does not modify the container, use `std::cbegin()` and `std::cend()` which are the const versions of `std::begin()` and `std::end()`

```c++
auto it = std::find(values.cbegin(), values.cend(), 1986);
auto it = std::find(std::cbegin(values), std::cend(values), 1986);
```

This brings the next point.


#### Want generic code, use std::begin(), std::end(),... etc {#want-generic-code-use-std-begin-std-end-dot-dot-dot-etc}

Sometimes you know that your incoming object is a container but you don't have the guarantee that the `container::begin()` and `container::end()` methods are provided. This is the reasons to get in the habit of using `std::begin()` and `std::end()`. This makes the things a lot more generic. With them you can do this:

```c++
template<typename C, typename V>
void findAndInsert(C& container, const V& target, const V& insert)
{
  auto it = std::find(std::cbegin(container), values::cend(container), target);
  container.insert(insert, it);
}
```


### noexcept is good and it is to be used carefully {#noexcept-is-good-and-it-is-to-be-used-carefully}

Today the C++ compilers are quite the smart little things. Much smarter than before. Therefore they can optimize a lot of things and produce a more efficient binary. `noexcept` is one of the conditions that must be met for a function to be "most optimizable". It means that the function <span class="underline">cannot</span> and <span class="underline">won't</span> emit exception at runtime,

```c++
void fun(int x) noexcept;
```

<br /> Right off the start, we need to say that `noexcept` is a part of a function's interface. Callers may examine if a function is `noexcept` and their behavior may depend on it. `noexcept` is almost as important as `const`. Think about `noexcept` in the definition of each function. To not declare a function `noexcept` is a missed opportunity to tell the compiler and everyone else that your function meets certain conditions. However, be careful, as said before `noexcept` is part of the interface. Whatever your choice might be, it must not be changed in the long term. Removing `noexcept` (or adding for that matter) to a function definition may break binary compatibility with other parts of the program that use said function. <br /> If an exception is emitted from a `noexcept` function at the runtime, the program will simply terminate. So again, be careful with defining functions `noexcept`. If your function is `noexcept` but in the body usage of no-`noexcept` function is present, you might be in trouble. <br /> A function may conditionally be `noexcept`.

```c++
template<class T, size_t N>
void swap(T (&a)[N], T (&b)[N]) noexcept(noexcept(swap_el(*a,*b)));
```

This function is noexcept only if the condition in the `noexcept` block is true. In this case, only if the function swap\_el applied on two elements of the arrays a and b is `noexcept`.


### constexpr is the new hot thing! {#constexpr-is-the-new-hot-thing}

`constexpr` indicates a value(when used for variable definition) that is known during compilation. This is quite different from simply being _const_. Function parameters can be _const_ but are not known during compilation. This:

```c++
constexpr int x = 5;
```

is on the other hand known during compilation. In this example _x_ is a _compile-time constant_. It can be used in interesting ways. For example:

```c++
std::array<int, x> arr; // defines array with 5 int elements
```

At this point, I'll have to mention that, of course, all `constexpr` objects are _const_ by extension. Not all _const_ objects are `constexpr` however. <br /> Things become a lot more interesting once `constexpr` functions are involved. Those create what is called a `constexpr` _context_. This means that those functions could be parsed and executed during compilation given that the provided arguments are known during compilation(i. e. they are defined as `constexpr`). When the conditions for compile-time executions are not met, the function can also act like a normal function. In C++11 those `constexpr` functions were limited to a single statement - the return statement. In C++14 however, this limitation is drooped. Simple example:

```c++
constexpr int pow(int base, int exp) noexcept
{
  auto res = 1;
  for(int i = 0; i < exp; i++) res *= base;
  return res;
}
constexpr size = 3;
std::array<double, pow(3,2)> arr; // array of size 9
```

When I first saw this, it blew my mind! `constexpr` functions may be executed in `constexpr` contents only if take literal type. Those include all base types except void but user defined types may also be literal if the define `constexpr` constructor and may be used in `constexpr` context if some of their methods are `constexpr`. This is absolutely valid:

```c++
class Point
{
public:
  constexpr Point(int xval = 0, int yval = 0) noexcept
    :x(xval), y(yval)
    {}

  constexpr int getX() const noexcept{return this->x;}
  constexpr int getY() const noexcept{return this->y;}
private:
  int x, y;
}
int main(int argc, char *argv[])
{

  constexpr Point point(5,3);
}
```

`point` can be further used in `constexpr` functions and those could be executed during compilation. To note is that in C++11 setters for the _Point_-Class can't be `constexpr` as they modify the object and therefore they are not even _const_. Further more, the return types of those are _void_ which is not literal type. C++14 lifts both of these restrictions. So..

```c++
class Point
{
public:
  constexpr Point(int xval = 0, int yval = 0) noexcept
    :x(xval), y(yval)
    {}

  constexpr int getX() const noexcept{return this->x;}
  constexpr int getY() const noexcept{return this->y;}

  constexpr void setX(int val) noexcept{this->x = val;}
  constexpr void setY(int val) noexcept{this->y = val;}
private:
  int x, y;
}
```

This gives us the ability to write something like this:

```c++
constexpr Point reflection(const Point& p) noexcept
{
  Point result;
  result.setX(-p.getX());
  result.setY(-p.getY());
  return result; // returning copy of the object
}
```

If invoked with a `constexpr` variable of type _Point_ the function will be evaluated at compile time. <br /> It's important to keep in mind that `constexpr` is a part of a function's interface. Again, as `noexcept`, users may rely on this interface. Also, if `constexpr` is used with `constexpr` variables in `constexpr` context <span class="underline">**and**</span> it has some side effects(as I/O or simply logging something to the standard output) it will cause a compile-time error. So yeah, be careful. `constexpr` is pretty close to the new `const` but not quite! <br /> One final thing. Please do yourself a favor and check out [this](https://www.youtube.com/watch?v=PJwd4JLYJJY)! A talk with [Jason Turnen](https://articles.emptycrate.com/about.html) and [Ben Deane](https://github.com/elbeno) that shows exactly what you can do with `constexpr`.


### The mutable keyword exists and you should know about it! {#the-mutable-keyword-exists-and-you-should-know-about-it}

So let's say you have the following class that is used not only by you but by someone that is not you and over whom you have to direct control.

```c++
class ResourceProvider
{
public:
  ResourceProvider(...){}

  void changeState(int x, int y)
    {
      this->x = x;
      this->y = y;
    }

  double getResource() const
    {
      return this->expensivecomputation();
    }
private:
  double expensiveComputation(int x, int y) const {...}
  double x,y;
}
```

Everything is perfect. The _getResource_ function is `const` and it has no side effect. This is as good as it gets with C++ functions. It's optimizable AF. <br /> One they, however, one of the clients of the class writes you an email with the complaint that the `getResources` function is too slow and `changeState` gets called relatively seldom so they end up caching the result of `getResource`. "Wouldn't it be convenient if the class did that on its own automatically," they say. And yes! It certainly would be nice. So you change your class to:

```c++
class ResourceProvider
{
public:
  ResourceProvider(...){}

  void changeState(int x, int y)
    {
      this->x = x;
      this->y = y;
      this->state_changed = true;
    }

  double getResource() const
    {
      if (state_changed)
      {
        cached_value = this->expensivecomputation();
        state_changed = false;
      }

      return cached_value;
    }
private:
  double expensiveComputation(int x, int y) const {...}
  double x,y;
  bool state_changed = true;
  double cached_value;
}
```

And....it doesn't compile. Sad! `getResource` has side effects now. It isn't `const`! It can't `const`! BAD! You have a few options now:

-   remove the `const` from the function declaration and the hell breaks loose
-   use something like `const_cast` and feel... like a horrible human being after the fact (plus it's really ugly to do that). I mean... you are lying with your function interface if you do that
-   use `mutable`!!

Yes `mutable` is completely different beast. If you define variable as `mutable` it can be changed from _const_ functions. Myeah! You rewrite your class.

```c++
class ResourceProvider
{
public:
  ResourceProvider(...){}

  void changeState(int x, int y)
    {
      this->x = x;
      this->y = y;
      this->state_changed = true;
    }

  double getResource() const
    {
      if (state_changed)
      {
        cached_value = this->expensivecomputation();
        state_changed = false;
      }

      return cached_value;
    }
private:
  double expensiveComputation(int x, int y) const {...}
  double x,y;
  mutable bool state_changed = true;
  mutable double cached_value;
}
```

You ship it. Everyone is happy. The code is clean. You can live in peace with the new knowledge now! Also, watch [this](https://www.youtube.com/watch?v=tTexD26jIN4) talk by [Kate Gregory](http://www.gregcons.com/kateblog/). She goes over the problem presented here in much more engaging fashion.


### std::unique\_ptr is for exclusive ownership! {#std-unique-ptr-is-for-exclusive-ownership}

Generally, when you want to use a pointer in the new modern C++ world, your first thought should be "Can I use `unique_ptr` here?". And yes, this is the preferred way of using "pointers" these days. `unique_ptr` can be viewed as small as a raw pointer and for most operations, they behave exactly the same way as raw pointers. <br /> There are few things to keep in mind while using `unique_ptr`.

-   `unique_ptr` embodies exclusive ownership. Every `unique_ptr` that is not empty "owns" the resource it's holding and you have a guarantee (generally) that this is the only object holding a pointer to the underlying object. The `unique_ptr` cannot be copied as this would create another holder of the resource so `unique_ptr`-s are only movable
-   You can know <span class="underline">exactly</span> when an `unique_ptr` object would be destroyed and with that the resource released. It is said that one of the greatest features of C++ is the closing brackets `}`. In order words, the fact that you know the exact moment at which an object will be destroyed and the destructor of the class will be executed for this object. The ramifications for `unique_ptr` is that the object will be destroyed when the enclosing scope comes to an end. Of course, you could move the `unique_ptr` before that and transfer its ownership to some other part of the program. The moving was talked about in the [beginning](#orgdc1a8fb).

<br /> A typical use of `unique_ptr` are the factory methods. The factory function could even return different type (from some hierarchy, of course) of an object depending on the inputs input parameters.

```c++
class Base {}
class D1 : public Base {}
class D2 : public Base {}

std::unique_ptr<Base> createObj(int param1, double param2)
{
  if (...)
    return std::make_unique<D1>();
  else
    return std::make_unique<D2>();
}
```

<br /> Another thing to know about `unique_ptr` is its ability to define custom deleter for the object. Normally `unique_ptr` uses simple `delete` to destroy the resource it's holding to but this behavior could be overridden.

```c++
auto del_base = [](Base *object){
                 makeLog(object);
                 ...
                 delete object;
               };
std::unique_ptr<Base, decltype(del_base)> ptr(new Base(), del_base);
```


### std::shared\_ptr is for shared ownership! {#std-shared-ptr-is-for-shared-ownership}

Not having garbage collection in C++ is a curse and a blessing at the same time. The knowledge of when exactly your objects are being destroyed and memory released gives you quite the freedom to write high performing code. On the other hand, this freedom comes with a lot of pitfalls and potential problems that you can introduce into your program. `shared_ptr` aims to provide you with "garbage collection"-like solution while staying true to the C++ "optimize everything" principles. <br /> `shared_ptr` implements a reference counting system. Several different `shared_ptr` objects can hold a pointer to the same underlying resource. The resource will be destroyed only then when all `shared_ptr` have exited their respective scopes (i.e. are destroyed). There is a little bit of overhead once you bring the whole _reference_-spiel. The important things to keep in mind:

-   `shared_ptr` are twice the size of a raw pointer
-   memory for the reference count must be dynamically allocated
-   Increments and decrements of the reference count must be atomic
-   Constructing a `shared_ptr` usually increments the reference count. Why usually? Because move-construction is also construction but it doesn't actually produce "new object". The pointer of the old object (the one we are moving from) is just set to _null_. and the new object takes its place in the world so no new reference is actually created.
-   Each resource that is managed through `shared_ptr` has what is called a _control block_. There the `shared_ptr` meta-information about he pointer it holds(reference count, custom delete... ). The _control block_ is allocated on the heap
    -   `std::make_shared` always creates new _control block_
    -   A _control block_ is also created when the `shared_ptr` object is constructed from a unique-ownership pointer (`unique_ptr`).
    -   A _control block_ is created even when you pass a raw pointer to the constructor of the `shared_ptr`. If you want a new `shared_ptr` that manages the same resource, you should copy-construct it from another `shared_ptr`

The last point is kinda important. You have to think about the control blocks that are created or else you could end up with several control blocks for the same resource which is a perfect recipe for undefined behavior. Look the following code

```c++
std::vector<std::shared_ptr<Widget>> processed;
...
class Widget
{
public:
  void process()
    {
      processed.emplace_back(this);// bad!!!
    }
}
```

The part that is wrong is the passing of a raw pointer to a container of `shared_ptr`-s. This will create a new control block which <span class="underline">could</span> be not bad but if there are other `sahred_ptr`-s in some other part of the program that already have control block for the `*this` object... **undefined behavior**!! This is, however, a problem that was thought about and a solution exists. Introducing `std::enable_shared_from_this<T>`. Weird name but it does the trick. If a class inherits from this thing, then it is safe to create `shared_ptr` from the `this` pointer. The safe code looks like:

```c++
class Widget : public std::enable_shared_fro_this<Widget>
{
public:
  void process()
    {
      processed.emplace_back(shared_from_this());// good!!!
    }
}
```


### Pimpl and the proper way to use it. {#pimpl-and-the-proper-way-to-use-it-dot}

Ever heard of [forward declaration](http://www.learncpp.com/cpp-tutorial/17-forward-declarations/). If not go check it out. The Pimpl idiom kinda solves the same problem. If you don't want to clutter your header files with the header files of the classes that you use, you can "deffer" this "implementation detail" to the `.cpp` file. Check this out:

```c++
//Wifget.hpp
class Widget
{
  Widget();
  ~Widget();

private:
  struct Impl;
  Impl impl*;
}
//Wifget.cpp

#include <iostream>
#include <string>

#include "Gadget.hpp"
#include "Widget.hpp"

struct Widget::Impl
{
  std::sting name;
  Gadget g1, g2, g3;
  std::vecotr<double> data;
}

Widget::Widget()
  :impl(new Impl);

Widget::~Widget()
{
  delete impl;
}
```

Notice how in `Widget.hpp` we didn't have to include a single header. There is no mention of the headers for the fields in the implementation of the class. The implementation appears only in the `.cpp` file. This can potentially speed up compilation times as it reduces the compile-dependencies between the classes. <br /> Now to utilize C++14 we can rewrite the class to use `uniqe_ptr` for the implementation and write our necessary constructors.

```c++
//Wifget.hpp
class Widget
{
  Widget();
  Widget(const Widget& rhs);
  Widget& operator=(const Widget& rhs);
  Widget(const Widget&& rhs);
  Widget& operator=(Widget&& rhs);
  ~Widget();

private:
  struct Impl;
  std::uniqe_ptr<Impl> impl;
}
//Wifget.cpp

#include <iostream>
#include <string>

#include "Gadget.hpp"
#include "Widget.hpp"

  struct Widget::Impl{..}

    Widget::Widget()
      :impl(std::make_unique<Widget::Impl>()){};

Widget::Widget(const Widget&& rhs) = default;
Widget::operator=(Widget&& rhs) = default;
Widget::~Widget() = default;

Widget::Widget(const Widget& rhs) :
  impl(std::make_unique<Widget::Impl>(*rhs.impl)){}

Widget& Widget::operator=(const Widget& rhs)
{
  *impl = *rhs.impl;
  return *this;
}
```

_Note:_ Use your header only for a declaration when possible. Also, `= defualt` is <span class="underline">implementation</span> so put in the `.cpp` file, not in the header. If in this case, you did that in the header file, the compiler would not how to generate the move constructor and the move assignment operator as an implementation of the calls is <span class="underline">not</span> in the header. In the header the `struct Impl` is incomplete and `*impl` is a pointer to incomplete type so the compiler could not deal with that. <br /> The compiler happily generates the move constructors for us as the default implementation is exactly the thing we want. Just perform _move_ on the implementation. The default copy constructors, however, would only perform a shallow copy of the object so we have to write them ourselves. <br /> The detractor is also the default one because we have no code to put in it. The `unique_ptr` automatically deletes its contents once it is destroyed.


### std::forward and std::move are quite interesting. {#std-forward-and-std-move-are-quite-interesting-dot}

<a id="org2e5602b"></a> Ok, hopefully by now you at least have heard of **move semantics**. Inevitably you've also probably seen `std::move()` and `std::forward<T>()` used in some weird way and wondered "What the hell is happening here?". The first thing to understand about hose functions - they are functions that don't do much in run-time. They don't generate code. They don't "move". At their core, those functions are casts. They cast _rvalue_ object to _lvalue_ ones. Refer to one of the [intro sections](#org1edb870) for more information about what are those. The difference between `std::move()` and `std::forward<T>()` is that they perform the cast under different conditions. `std::move()` performs it always with no conditions. `std::forward<T>()` casts to revalue only if a certain condition is met - if the argument is bind to rvalue. <br /> Yes, it is well known and by this point accepted fact that `std::move()` has confusing AF name but... just roll with it and it will grow on you. `std::move()` doesn't actually move anything. `std::move()` merely makes an object eligible for moving. This is no guarantee however that an object will be moved from. Consider this:

```c++
class Anotation
{
private:
  std::string text;
public:
  Anotation(const std::string name): text(std::move(name)){}
  //this doesn't do what it seems to do
}
Anotation("ano");
```

Think about what is happening here. We are passing a string to the constructor which takes it by value and "moves" its content inside the `text` field. But there is also `const`. This means that the result of the `std::move(name)` is _rvalue const std::string_. The constness is still there. This means that the move constructor of `std::string` <span class="underline">cannot</span> be used as it will change the object that it takes which is _const_. For that reason, the ordinary constructor is called and the value is just copied and not moved. The lesson to be learned here - rvalue objects will not be moved sorely on the fact that they are _rvalue/s. They also must be /const_. <br /> As already said `std::forward<T>()` only casts to rvalue if the argument is bound to rvalue object. `std::forward<T>()` is typically used in cases where _perfect forwarding_ is required. That is, a function takes some arguments and those are automatically _lvalue_ in the function body even if the fiction is invoked with _rvalue/s. In the body, however, you may want to "forward" the argument to other functions that may need to differentiate between /r-_ and _lvalues_. In this case, one would use `std::forward<T>()`. Example:

```c++
template<typename T>
void logAndProcess(T&& param)
{
  auto now = std::chrono::system_clock::now();
  makeEntry("Calling process", now);
  /* Here we dont't know whether the function was invoed
   with rvalue or lvalue. Therefore we forward the
   argument perfectly */
  process(std::forward<T>(param));
}
```


### Universal references and rvalue references {#universal-references-and-rvalue-references}

First things first - "universal reference" is a lie. There is no such thing in the official C++ specification. There there is "forwarding reference". "Forwarding reference" is absolutely the same thing as what Scott Meyers means with "universal reference" in his book. Here I will also use universal reference. OK? OK. Good that now this is out of the way. <br /> We know that we can define functions that take rvalue references like:

```c++
void f(Widget&& param);
```

Here `param` is _rvalue_ reference and it binds only to _rvalue_ objects. Consider on the other hand this function:

```c++
template<typename T>
void g(T&& param);
```

Here param can bind to _rvalues_ <span class="underline">and</span> to _lvalues_. This is what makes it _universal reference_. <br /> For a reference to be universal one certain condition must be met

1.  Type deduction must occur
2.  The reference must have exactly the form "T&&". No `const`, no nothing. Just "T&&".

If the `const` qualifier is used for the function parameter, it(the parameter) will "collapse" _rvalue const reference_ which pretty useless on itself as we say in the [previous tip](#org2e5602b). <br /> As stated if universal reference is bound to "rvalue", it is an "rvalue"-reference and "lvalue"-reference if it's bound to an "lvalue". This means that universal references are exactly the things to be used with `std::forward<T>()`.


### _Pass by value_ is not what your first C++ book would have you believe {#pass-by-value-is-not-what-your-first-c-book-would-have-you-believe}

Maybe this is exclusive to me but when I was learning C++ I was left with the impression that passing things by value is kinda dumb if you can pass it by reference. I mean, who needs the extra copy, right?! Not quite. Consider the case of a simple setter

```c++
class Widget
{
  void setName(const std::string& name)
    {
      this->name = name;
    }
}
```

What you actually are doing is:

-   take a string by reference
-   copy it in the field `name`

The copy is still there! It's just that <span class="underline">you</span> performed the copying and not the compiler. Now, this is kinda sub-optimal. The compiler <span class="underline">is</span> smarter than you. After the compiler has copied the value from the caller in the function's parameter we can - enter modern C++ - _move_ it in the corresponding field:

```c++
class Widget
{
  void setName(std::string name)
    {
      this->name = std::move(name);
    }
}
```

This looks like good modern c++. Copying... is not that bad as might you think. Compilers are pretty good these days. Do not be mislead. A copy of a single string has virtually no overhead. <br /> Still, there still exists a time and place for passing by _const references_. If you actually don't need a copy of the passed argument, then there is no reason for pass-by-value

```c++
class Widget
{
  void process(const Gadget& gadget)
    {
      // now we can only use const functions of Gadget
      if(gadget.getNumberOfTicks){....}
    }
}
```

Passing pure references also makes sense in some situations:

```c++
std::vector<std::string> names;
void populate(std::string& name)
{
  name.pusb_back("Stanislv");
  name.pusb_back("Marina");
}
populete(names);
```

To summarize (taken from an answer from [this](https://stackoverflow.com/questions/7592630/is-pass-by-value-a-reasonable-default-in-c11) StackOverflow question)

| Signature          | Use                      |
|--------------------|--------------------------|
| bar(foo f);        | want to obtain a copy of |
| bar(const foo& f); | want to read f           |
| bar(foo& f);       | want to modify f         |


### Return value optimization(RVO) - don't return std:::move of local variable {#return-value-optimization--rvo--don-t-return-std-move-of-local-variable}

With your awesome new knowledge about move-semantics you can now write highly optimized, highly performant code that will go \\\*whoosh\\\* past those pesky languages running on virtual machines. You will, however, be tempted to make some "optimizations" at places where you really shouldn't mess with the compiler. Please always repeat to yourself "The compiler is smarter than me!". <br /> Consider the following snippet:

```c++
Widget makeWIdget()
{
  Widget w;
  ...;
  return w;
}
```

We are creating a local variable in a function and we are returning it by value. We think about what is happening. The object is created - construction, the object is returned by value - we <span class="underline">copy</span> our new object and return the copy because the local object will be destroyed once the scope of the function ends. Copy! We know what _rvlaues_ are. We don't need any copy. We can move! We rewrite the code like:

```c++
Widget makeWIdget()
{
  Widget w;
  ...;
  return std::move(w);
}
```

Awesome, we saved ourselves one copy! **Wrong!!!** <br /> Introducing _Return Value Optimization_. In some situations(most of them) the compiler will notice that you are returning a local variable by value. If certain conditions are met, the compiler will construct this local object in the exact place in memory where the return value of the function will reside after the function is invoked. In such cases, the return statement won't produce copy <span class="underline">nor</span> will move construction would be necessary. This is good! The conditions that should be met for RVO to occur:

1.  The type of the local object must be the same as the return value.
2.  The local object is being returned.
3.  The type is move-constructible.

What happens when we `std::move` the local object - we crate reference to it and RVO can't be performed. So, **don't** return `std::move`! <br /> Ok, but still. RVO is just an optimization. Maybe the compiler won't be able to figure out which local object we want to return and will punish us with a copy while returning. No! Again - "The compiler is smarter than me!". Return values are <span class="underline">always</span> treated as _rvalues_. So when your function is:

```c++
Widget makeWIdget()
{
  Widget w;
  ...;
  return w;
}
```

<span class="underline">and</span> the compiler can't do RVO, what it effectively sees is :

```c++
Widget makeWIdget()
{
  Widget w;
  ...;
  return std::move(w);
}
```

So no. You would never need to return with `std::move` of local object.


### std::async is something that exists and it's generally to be preferred over std::thread. {#std-async-is-something-that-exists-and-it-s-generally-to-be-preferred-over-std-thread-dot}

A lot of times you will want to run something asynchronously in your program. C++ and the standard library make this relatively easy and give you two approaches.

-   `std::thread` - for _thread-based_ strategy
-   `std::async` - for _task-based_ strategy

In general, you should prefer `std::async`. It works on a higher level of abstraction than `std::thread` and it hides some of the details that you can mess up and delegates them to the implementation. The basic asynchronous call goes like:

```c++
int doAsyncWork();

auto fut = std::async(doAsyncWork); // fut is a future
```

A further reason why `std::async` is better is because with it we can get the result from the function that we are calling asynchronously through the future's _get_ method. There isn't really a straight forward way of extracting a result from a separate thread constructed with `std::thread`. <br /> If you opt to use thread-based programming you'll have to deal with thread exhaustion, over-subscription, load balancing, and adaptation to new platforms. And you know...all of these are super fun to deal with. You may have to do it sometimes thou. As stated, threads are lower level concurrency API. Threads may allow you to write more efficient code suited to your specific needs. Cases where threads may be necessary

-   You want to mess directly with the concurrency API provided by the OS. Given this would be pretty hard, it could potentially lead to tremendous speed when done right. Essentially you would have to define the way how your program communicates with the OS as "real" threads are managed by it.
-   You know exactly what your needs are and you want to deal with threads and build the whole multi-threading and asynchronous code/system yourself. This is pretty doable and could be a nice learning experience. You probably will implement something that already exists but hey, that's how you learn new things and I would actually encourage this.
-   If you are implementing some concurrency technology that is not offered by C++, yeah, you bet you'll need to use threads and deal with them.


## Conclusion {#conclusion}

I just want to say one thing. Never, never say that you "know C++" and be deeply skeptical of people who use the said expression. C++ is complex and you can do everything in a lot of different ways and probably things are going to work no matter what. However, some solutions in C++ are very clearly better than others and a lot of times it is not clear what is the "optimal" way of achieving your goal. Do not be mislead by your knowledge! C++ offers you an ever-expanding world of good\bad solutions and knowing everything that there is to know is most likely infeasible. And you know what, that's actually OK. Because let's face it, software engineering is all about learning something new every day in the process of horribly breaking your code.


## References {#references}

I'm very thankful to these sources:

-   [Effective modern C++](https://www.amazon.de/Effective-Modern-Specific-Ways-Improve/dp/1491903996) - the book am talking about in this post
-   [Smart pointers guidelines](https://www.chromium.org/developers/smart-pointer-guidelines) - a very useful guide on how to be smart with your smart pointers
-   [Understanding the meaning of lvalue or rvalues](https://www.internalpointers.com/post/understanding-meaning-lvalues-and-rvalues-c) - a good blog post that will walk you through the usages and the meaning of move semantics and how are they implemented through _rvalue_ and _lvalue_.

Check them out if you want to be a better C++ programmer.
