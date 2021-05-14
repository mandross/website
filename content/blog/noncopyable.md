+++
title = "Noncopyable"
date = "2015-11-26" 
+++

Quite often you do not want your objects to be copy-constructed or assigned to others. As there is more than one way to skin a cat, let's go through possible solutions.

## Classic way

The one I have seen the most often. Leave copy constructor and assignment operator declared but not defined:

### Code

```c++
class Foo
{
public:
    Foo(){};

private:
    Foo(const Foo&);
    Foo &operator = (const Foo&);
};

int main()
{
    Foo A, B;
    A = B;
}
```

### Compiler output

```
noncopyable.cpp: In function ‘int main()’:
noncopyable.cpp:25:10: error: ‘Foo& Foo::operator=(const Foo&)’ is private
     Foo &operator = (const Foo&);
          ^
noncopyable.cpp:43:7: error: within this context
     A = B;
       ^
```
So far, so good.

### Glitches  
The first glitch with this code will be visible, once we try to make a copy in member method. Lets add a method and change our main function:

```c++
class Foo
{
public:
    void copy()
    {
        Foo X;
        *this = X;
    }
[...]

int main()
{
    Foo A;
    A.copy;
}
```

```
/tmp/ccnMGbs7.o: In function `Foo::copy()':
noncopyable.cpp:(.text._ZN3Foo4copyEv[_ZN3Foo4copyEv]+0x31): undefined reference to `Foo::operator=(Foo const&)'
collect2: error: ld returned 1 exit status 
```

This time we are getting linker error. This is the glitch I dislike the most. IMHO you should get the error as soon as possible to save precious time. What is more, linker errors are not as clear to read as compiler errors. Let's analyze what happened. 

In the first example the copying was done from outside of the class. The copy constructor and assignment operator are private, thus the outside world is not aware of their existence. Compiler throws an error immediately. In the second example the coping was done from inside. The copy constructor and assignment operator are visible. Once the linking is done, the linker can't find their definition and linking fails.  

The second glitch is self-descriptiveness. The declaration without definition is a bit tricky, and may be confusing. Some people argue, that this way is the best, because it is the oldest and most common, which means: well known by engineers. Following this way of thinking, we should stick to fireplaces as the heat sources in our homes. The technology is well known and proven to work, right?

The third glitch is the amount of code you have to write. You need two lines of code per noncopyable class plus some comment to lower the risk of confusion. Not too bad, but in large project cutting it down is worth consideration.

## C++ 11 way

### Code

```c++
class Foo
{
public:
    Foo(){};

private:
    Foo(const Foo &) = delete;
    Foo &operator = (const Foo&) = delete;
};
```

### Compiler output

This time in both cases we are getting the same compiler error:

```
noncopyable.cpp:43:7: error: use of deleted function ‘Foo& Foo::operator=(const Foo&)’
noncopyable.cpp:25:10: note: declared here
     Foo &operator = (const Foo&) = delete
```

It is way nicer now, isn't it? The code is self descriptive, and we are getting the error early. Two birds hit with one stone. Unfortunately, we still have to deal with the glitch no. 3.

## BOOST way

If you haven't heard about boost library, you should definitely check it out.

### Code

```c++
#include <boost/utility.hpp>

class Foo : private boost::noncopyable {};

int main()
{
    Foo A, B;
    A = B;
}
```

Short, nice and sweet. You have to include one header, inherit from *boost::noncopyable* and that's it. Let's check compiler output:

### Compiler output

```
noncopyable.cpp: In function ‘int main()’:
noncopyable.cpp:8:7: error: use of deleted function ‘Foo& Foo::operator=(const Foo&)’
     A = B;
       ^
noncopyable.cpp:3:7: note: ‘Foo& Foo::operator=(const Foo&)’ is implicitly deleted because the default definition would be ill-formed:
 class Foo : private boost::noncopyable {};
       ^
noncopyable.cpp:3:7: error: use of deleted function ‘boost::noncopyable_::noncopyable& boost::noncopyable_::noncopyable::operator=(const boost::noncopyable_::noncopyable&)’
In file included from /usr/include/boost/utility.hpp:19:0,
                 from noncopyable.cpp:1:
/usr/include/boost/noncopyable.hpp:35:22: note: declared here
         noncopyable& operator=( const noncopyable& ) = delete;
```

Note that you will get a different error when compiling with c++98. Boost detects the standard version and provides different implementation. No matter which standard we use or from where we try to copy, there will be a nice, descriptive compiler error (good). Additionally, there is no need to type too much (lovely!).

### Glitches  

We deal with inheritance here: what will happen if we have a class with two parents, both inheriting from boost::noncopyable?

```c++
#include <boost/utility.hpp>
#include <iostream>

class Mom : private boost::noncopyable {};
class Dad : private boost::noncopyable {};
class Junior: Mom, Dad {};

int main()
{
    Mom m;
    Dad d;
    Junior j;
    std::cout<<"Mom/Dad/Junior "<<sizeof(m)<<"/"<<sizeof(d)<<"/"<<sizeof(j)<<std::endl;
}
```
Console: `Mom/Dad/Junior 1/1/2`
Why the Junior is so fatty? I will start answering by explaining why the parents are so lean. They both inherit from boost::noncopyable, if you check the implementation of the base you will notice that the class has no members.  C++ standards requires that the most derived object has not zero size, but the base can be optimized. This is called Empty Base Optimization (EBO) which is out of scope of this post. Ok, but why the size of the Junior is 2? Both Mom and Dad have common ancestor. The standard requires that both have distinct addresses to avoid ambiguity. As a result the EBO is prohibited.

The second issue is that we are bringing the implementation with the inheritance. Inheritance is a very strong coupling mechanism in C++. Usually, I try to use composition instead, but noncopyable is kind of special here (yes, I am too lazy to type when I don't have to). There are some people who threat the rule of bringing in the implementation only by composition as a dogma. If you have to cooperate with them you can:

* give up on noncopyable
* start religious war
* unfriend them (rage quit)

## Custom noncopyable

If, for some reason, you can't use boost (e.g.: hard to be configured for your exotic compiler, you are not allowed to) or you are dealing with small memory software and EBO glitch is too much, you may use your own idiom in Curiously Recurring Template (CRTP) version:

```c++
template <class T>
class noncopyable
{
protected:
    noncopyable() {};
    ~noncopyable() {};
private:
    noncopyable(const noncopyable &) = delete;
    noncopyable& operator= (const noncopyable&) = delete;
};

class Foo : private noncopyable<Foo> {};
```

If we compare the sizes as previously: `Mom/Dad/Junior 1/1/1`, the EBO glitch is gone, we do not have the famous Diamond of Death any longer. Sadly, we still have to deal with inheritance, but life (at least for C++ programmers) is not perfect.

## Summary
Personally I am using either the boost or custom option. Every single one will work. I hope, that after reading this pretty long post, you will be aware of pros and cons of every presented method. Choose the one which suits you best. Happy coding!

## Links
1. [More C++ Idioms](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Non-copyable_Mixin) (Wikibook), 
1. [Empty Base Optimization] (http://en.cppreference.com/w/cpp/language/ebo)
1. [BOOST noncopyable](http://www.boost.org/doc/libs/1_59_0/libs/core/doc/html/core/noncopyable.html)