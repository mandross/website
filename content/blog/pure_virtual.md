+++
title = "Pure virtual function call hadler"
date = "2015-11-28" 
+++

# Pure virtual function call hadler

*Handling pure virtual function call is compiler specific. Analysis is done for g++ 4.9.2.*
## How to call a pure virtual fucntion?
```c++
struct Dad
{
    virtual ~Dad(){make();}
    void make() {outsource();}
    virtual void outsource() = 0;
};

struct Kid : public Dad
{
    void outsource() {}
};

int main()
{
    Kid k;
}
```
Output:
```
pure virtual method called
terminate called without an active exception
Aborted (core dumped)
```

## What the heck just happened?
After leaving the scope object k of type Kido is destructed. Firstly, derived's class destructor is called. Secondly, the base class destructor is called, during the destruction method make() calls method outsource(). The derived class is already destroyed, thus we call the pure virtual method.
## Why to do that?
The answer is: you shouldn't, but 'shit happens'. If you try to call pure virtual functions directly from destructor, g++ will warn you, and the linking will fail. Wrapping the call fulls the compiler.
## How is pure virtual function call handled?
Once such call happens static function *static int __cxa_pure_virtual()* is fired, you see nice print and the application is terminated.
## What to do about it?
If your are coding for PC: nothing more than finding the spot and fixing the bug. In embedded software universum you may want to implement your own handler.
## Why?

* he handler is not implemented at all, and you start getting strange, hard to hunt down bugs
* you would like to implement your own assertion
* leaving the default handler brings a lot of unnecessary symbols into you object e.g.: exceptions

## How?

#include <cassert>
a pure virtual fucntion?
```c++
extern "C"
{
static int __cxa_pure_virtual() __attribute__((noinline, used));
static int __cxa_pure_virtual()
{
    assert(0);
    return 0;
}
}a pure virtual fucntion?
```