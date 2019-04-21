+++
title = "C++ Template Specialization"
author = ["Stanislav Arnaudov"]
description = "On template specialization of a member function in C++."
date = 2018-07-06T00:00:00+02:00
keywords = ["c++", "cpp", "templates", "programming"]
lastmod = 2019-04-21T23:00:57+02:00
categories = ["c++"]
draft = false
weight = 100
+++

## Abstract {#abstract}

C++ is awesome! I am starting strong here but it really is. When experienced (and crazy) enough one can implement everything and the at the same can do it incredibly efficient at run time (cuz who cares how much work does the compiler do for us). I can rant about how c++ is the best and all of its great features but for this post I will constraint myself to one particular part of C++ - template meta-programming! Even more concretely, I would like to show you a trick that I discovered myself that helped me solve one nasty problem. Yes, I admit, I am ~~probably~~ certainty not the first one who discovers this trick **BUT** I promise you, I discovered it by myself and the whole process of discovering it was immensely satisfying. I mean... templates **are** magic and I can only imagine how the people who understand them feel.


## The problem {#the-problem}

Say you have a templated class that implements some kind of functionality and uses the type given by it's template argument as some soft of data. This probably sound a little bit vague. A more concrete example of what I am talking about - imagine you have some class that sends some data to a server _somewhere_. The data can be `std::string` or `int` or `double` or some other random structure of yours. Of course the natural thing to come with that is a method `send(data)` that takes some data object of the templates type and does the _internal magic_ (we don't care about that though). In code everything will look something like:

```C++
templete<DataType>
class Sender
{
  Sender(){}

  void send(DataType data){...}

};
```

Pretty simple, pretty straight forward. To expand our hypothetical situation, imagine now that this is not your class but someone else's and other people are using it in it's current form. So you don't think about it all that much and go on to use the class yourself. <br /> <br /> After some while you find out that internally the `send` function uses a field named _header_ of the type you've passed to the template of the class. For example you have something like this:

```c++
template<DataType>
void Sender<DataType>::send(DataType data)
{
...
data.header = currentTime();
...
}
```

This automatically locks you to sending only structures that have a field named header (probably also with time `unsigned int`). Not the biggest problem but again, let's make the situation complicated and say you have a project where you **have** to send a plain `std::string` to the server through this class (don't think about the situation too much, just roll with it and accept that I had very similar problem which I just couldn't avoid). <br /> <br /> So what do we have so far:

-   someone else's templates class - we are not the designers
-   people are using it - no change of API is preferable
-   the template argument of the class expects a field _header_
-   we **have** to send something without this _header_ field

<br /> <br /> And this is pretty much where I found myself that one time. The following explanations are the way how I solved the problem.


## Tackling it probably in the most terrible way {#tackling-it-probably-in-the-most-terrible-way}

So, the way I tried to do it was to add another template argument to the class - a `bool` - that specifies whether a _header_ is used or not. Then I hoped that I could just specialize the `send` function with this new template argument set to _false_ with an implementation without the use of _header_. Of course the default value of the new template argument would be true for preserving backward compatibility. The way I though it's gonna work as something like:

```c++
template<DataType, bool header = true>
class Sender{...}

template<DataType>
void Sender<DataType, false>::send(DataType data)
{
...
//data.header = currentTime();
...
}

template<DataType bool header>
void Sender<DataType, header>::send(DataType data)
{
//default implementaion here
...
data.header = currentTime();
...
}
```

Seemed logical at the time but couple of problems with that. By the implementation of the function the _template<...>_ is for the _Sender_ class not for the function. This means that that all of the template arguments in the original declarations of the class have to be present there. Also when you think about it, there is no template function there to specialize. There is a templates class and the function is part of this class. After some more googling I found out that indeed, one cannot specialize member function of a templated class on some values of the template arguments. "Okay, so I have to specialize the entire class" I told myself. But then I would have to essentially have the class twice with one little difference in this one function - one version with using _header_ and one without. That's kind of dumb. Having almost the same code twice is dumb. This is why we have abstraction after all. Abstraction! Inheritance! <br /> <br /> So, how about that. I create a new class with two templated arguments, I inherit the old class and pass the _DataType_ argument to it and based on the second template I implement `send`. Again, make the _bool_ default to _true_ and name the new class the _Sender_ so that the existing code base doesn't get broken. I would have something like this:

```c++
templete<DataType>
class SenderBase
{
  Sender(){}

  void send(DataType data){//no implementaion here}
}
}
templete<DataType, bool header = true>
class Sender : public SenderBase<DataType>{}

templete<DataType>
class Sender<DataType, true> : public SenderBase<DataType>
{
  void send(DataType data)
    {
      ...
      //default implementaion
      data.header = currentTime();
      ...
    }
}

templete<DataType>
class Sender<DataType, false> : public SenderBase<DataType>
{
  void send(DataType data)
    {
      ...
      //no header version implementaion
      //data.header = currentTime();
      ...
    }
}
```

There, that _should_ do it. Think about what is happening when a new object of the class gets created. If we have

```c++
Sender<SomeDataType> obj;
```

the second templated arguments defaults to true and the second versions gets instantiated. If, on the otter hand, someone calls:

```c++
Sender<std::string, false> obj;
```

the compiler generates the version without the use of the _header_ field and does not complaint because there is no such field in `std::string`. Perfect! <br /> <br /> There was, however, something wrong with this approach. Think about what would happen if the class uses internally `send` for something. What happens if the client code is not the one who calls the `send` directly but the function call is made by other method that just passes the data to be `send`. This will be of course implemented in the base class `SenderBase` and this class sees only the empty implementation of the `send` function. The fix is very simple if you know your language. We just make the `send` function virtual and allow the program to dispatch the right function call at runtime based of the type of the calling object. <br /> <br /> The "final" solution becomes:

```c++
templete<DataType>
   class SenderBase
   {
     Sender(){}

     virtual void send(DataType data) = 0;
   }
   }
   templete<DataType, bool header = true>
   class Sender : public SenderBase<DataType>{}

   templete<DataType>
   class Sender<DataType, true> : public SenderBase<DataType>
   {
     virtual void send(DataType data){...}
   }

   templete<DataType>
   class Sender<DataType, false> : public SenderBase<DataType>
   {
      virtual void send(DataType data){...}
   }

```

There! Now we have it. Now we've (kinda) specialized a function of a templated class based on a value of a template argument without writing the whole class two times **AND** no previous user will notice this new feature of the class. It's not exactly a straight forward solution but I found it very elegant workaround.


## Tackling it in a better way {#tackling-it-in-a-better-way}

But of course, a simpler, more straight forward solution exists. Enter `enable_if` and class templates with added caveat. I mentioned - there is no template function so no use of [SFINAE](https://en.wikipedia.org/wiki/Substitution%5Ffailure%5Fis%5Fnot%5Fan%5Ferror) on deciding which function to pick. The thing is, template classes can have template functions as members. So, there is no reason we can't do something like

```c++
template<class DataType, bool Header = true>
class Sender
{
  template<bool Q = Header>
  typename std::enable_if<Q, void>::type send()
    {
      //use header here!
    };
  template<bool Q = Header>
  typename std::enable_if<!Q, void>::type send()
    {
      //do not use header here!
    };
};
```

We again add the extra _header_ template and based on it we pick one of two template functions `send`. It's important that they are in fact template functions and not plain member functions of a templated class. Now when we instantiate the class one of those two template functions is picked and compiler sees only it. Here you can even drop the _Header_ template parameter and work only with `enable_if` and some other template programming voodoo to detect whether DataType has field _header_ or not. I'll leave that to you though. _Note:_ The check for the _header_ field has to be done though the _Q_ template parameter of the functions because _SFINAE_ applies only on it.
