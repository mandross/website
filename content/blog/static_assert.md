+++
title = "static_assert"
date = "2015-12-02" 
+++

# static_assert

The nice feature [static_assert](https://en.cppreference.com/w/cpp/language/static_assert) is available since C++11, what to do if your compiler is lacking this feature?
You can choose between two options:

* use the [one from BOOST](https://www.boost.org/doc/libs/1_59_0/doc/html/boost_staticassert.html)
* write your own

In some of my previous projects BOOST was not available for production code. I ended up digging into BOOST, understanding the implementation and extracting it. Let's go through this process again.

```c++
namespace static_assert
{
    template <bool> struct STATIC_ASSERTION_FAILURE;
    template <> struct STATIC_ASSERTION_FAILURE<true> { enum { value = 1 }; };
    template <int> struct static_assert_test{};
}

#define JOIN(A, B)  DO_JOIN(A, B)
#define DO_JOIN(A, B)  A ## B

#define STATIC_ASSERT(...) \
    typedef static_assert::static_assert_test< \
        sizeof(static_assert::STATIC_ASSERTION_FAILURE<static_cast<bool>(__VA_ARGS__) >)> \
            JOIN(t_staticAssert, __LINE__)
```

## How it works?
The main idea is explained by boost members [here](https://www.boost.org/doc/libs/1_59_0/doc/html/boost_staticassert/how.html).
> The key feature is that the error message triggered by the undefined expression sizeof(STATIC_ASSERTION_FAILURE<0>), tends to be consistent across a wide variety of compilers. The rest of the machinery of BOOST_STATIC_ASSERT is just a way to feed the sizeof expression into a typedef.

The result of sizeof (let's call it X) is fed into following expression:
`typedef static_assert::static_assert_test<X> JOIN(t_staticAssert, __LINE__)`
If the condition passed to macro STATIC_ASSERT is false the compiler will not be able to evaluate sizeof operator. The attempt to make a typedef will fail. The JOIN macro is used to enable usage of static_assert many times in the same file. If the condition is true, every created typedef will have unique name. The macro has two layers to correctly evaluate \_\_LINE\_\_ macro. The typedef is used, because it will not impact memory map.

## Alternatives

Different compilers behave a bit differently, thus I have two others ways to use sizeof.

```c++
#define ENUM_STATIC_ASSERT(...) \
    enum  \
    { \
        JOIN(static_assert_enum, __LINE__) = sizeof(static_assert::STATIC_ASSERTION_FAILURE< \
            static_cast<bool>(__VA_ARGS__) >), \
    };
```
```c++
typedef for char array
#define ARRAY_STATIC_ASSERT(...) \
    typedef char JOIN(t_staticAssert, __LINE__)[static_assert::STATIC_ASSERTION_FAILURE< \
        static_cast<bool>(__VA_ARGS__) >::value]
```

Try all, choose the one for which yours compiler gives the most descriptive error.