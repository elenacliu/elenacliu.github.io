---
title: C++ Metaprogramming - Template(1)
date: 2024-05-03
math: true
image:
  placement: 2
  caption: 'Image credit: [**John Moeses Bauan**](https://unsplash.com/photos/OGZtQF8iC0g)'
---

模板库为什么必须在头文件中实现：
因为模板的原理是：**在编译时，每发现一种模板参数的模板实例，就生成对应模板参数的代码**。
如果使用源代码分开编译，则编译模板库的源代码时，编译器并不知道这一模板库有哪些模板实例；而编译模板实例时，又没有模板库的源代码来作生成。
因此会产生链接错误，没有生成对应模板参数的源代码。

# Template parameters and template arguments

## **Template parameters**

Every [template](https://en.cppreference.com/w/cpp/language/templates) is parameterized by one or more template parameters, indicated in the *parameter-list* of the template declaration syntax:

```cpp
template < parameter-list > declaration		
```

Each parameter in *parameter-list* may be:

- a non-type template parameter:
- a type template parameter;
- a template template parameter.

## **Template arguments**

In order for a template to be instantiated, every template parameter (type, non-type, or template) must be replaced by a corresponding template argument. 

For [class templates](https://en.cppreference.com/w/cpp/language/class_template), the arguments are either explicitly provided, [deduced from the initializer](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction), (since C++17) or defaulted. For [function templates](https://en.cppreference.com/w/cpp/language/function_template), the arguments are explicitly provided, [deduced from the context](https://en.cppreference.com/w/cpp/language/template_argument_deduction), or defaulted.

可以暂时理解为 argument 是 parameter 例化的取值。

# SFINAE: Substitution Failure Is Not An Error

替换发生在编译期，用于在编译时选择合适的模板实例（注意，部分特化是通过重载实现的），因此替换失败并非错误是指，在模版实例的 candidate set 中不断尝试，直到找到一个成功的替换。而不是失败了就立马报错；如果全部失败才报错。

This rule applies during **overload resolution of function templates**: When [substituting](https://en.cppreference.com/w/cpp/language/function_template#Template_argument_substitution) the explicitly specified or [deduced type](https://en.cppreference.com/w/cpp/language/template_argument_deduction) for the template parameter fails, the specialization is discarded from the [overload set](https://en.cppreference.com/w/cpp/language/overload_resolution) instead of causing a compile error.

This feature is used in template metaprogramming.

## **Explanation**

Function template parameters are substituted (replaced by template arguments) twice:

- explicitly specified template arguments are substituted before template argument deduction
- deduced arguments and the arguments obtained from the defaults are substituted after template argument deduction

Substitution occurs in

- all types used in the function type (which includes return type and the types of all parameters)
- all types used in the template parameter declarations
- all types used in the template argument list of a partial specialization

# Constraints on Template

最近在实现 C++ 模版函数的时候思考了一个问题，如果替换的 class 并不满足其所有的实现怎么办呢？如何在写代码的时候就限制这个事情。比如 `lower_bound` 的声明：

```cpp
template< class ForwardIt, class T >
ForwardIt lower_bound( ForwardIt first, ForwardIt last,
                       const T& value );
template< class ForwardIt, class T, class Compare >
ForwardIt lower_bound( ForwardIt first, ForwardIt last,
                       const T& value, Compare comp );
```

Type requirements

- `ForwardIt` must meet the requirements of [LegacyForwardIterator](https://en.cppreference.com/w/cpp/named_req/ForwardIterator)
- `Compare` must meet the requirements of *[BinaryPredicate](https://en.cppreference.com/w/cpp/named_req/BinaryPredicate)*. It is not required to satisfy *[Compare](https://en.cppreference.com/w/cpp/named_req/Compare)*.

这里的 requirements 是如何保证的？通过查阅 cpp reference 的 named requirements 界面我们发现：

https://en.cppreference.com/w/cpp/named_req

> The *named requirements* listed on this page are the named requirements used in the normative text of the C++ standard to define the expectations of the standard library.
> 
> 
> **The burden to ensure that library templates are instantiated with template arguments that satisfy these requirements is on the programmer.** Failure to do so may result in very complex compiler diagnostics.
> 
> Some of these requirements are formalized in C++20 using the [concepts](https://en.cppreference.com/w/cpp/language/constraints) language feature.
> 

说明程序员要自己遵守好规定。但如果程序员不遵守规定，是编译时就报错，还是运行时才报错呢？

我们先做一个小测试：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

using namespace std;

int main() {
    int array[] = {0,1,2,5,6};
    cout << lower_bound(array[0], array + 5, 4) << endl;
    return 0;
}
```

使用 `g++ -o main main.cpp` 进行编译、链接：

```bash
> g++ --version
g++ (Ubuntu 9.4.0-1ubuntu1~20.04.2) 9.4.0
Copyright (C) 2019 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.

> g++ -dM -E -x c++  /dev/null | grep -F __cplusplus
#define __cplusplus 201402L
```

报错信息为：

```
main.cpp: In function ‘int main()’:
main.cpp:9:47: error: no matching function for call to ‘lower_bound(int&, int*, int)’
    9 |     cout << lower_bound(array[0], array + 5, 4) << endl;
      |                                               ^
In file included from /usr/include/c++/9/algorithm:61,
                 from main.cpp:1:
/usr/include/c++/9/bits/stl_algobase.h:1004:5: note: candidate: ‘template<class _ForwardIterator, class _Tp> _ForwardIterator std::lower_bound(_ForwardIterator, _ForwardIterator, const _Tp&)’
 1004 |     lower_bound(_ForwardIterator __first, _ForwardIterator __last,
      |     ^~~~~~~~~~~
/usr/include/c++/9/bits/stl_algobase.h:1004:5: note:   template argument deduction/substitution failed:
main.cpp:9:47: note:   deduced conflicting types for parameter ‘_ForwardIterator’ (‘int’ and ‘int*’)
    9 |     cout << lower_bound(array[0], array + 5, 4) << endl;
      |                                               ^
In file included from /usr/include/c++/9/algorithm:62,
                 from main.cpp:1:
/usr/include/c++/9/bits/stl_algo.h:2028:5: note: candidate: ‘template<class _FIter, class _Tp, class _Compare> _FIter std::lower_bound(_FIter, _FIter, const _Tp&, _Compare)’
 2028 |     lower_bound(_ForwardIterator __first, _ForwardIterator __last,
      |     ^~~~~~~~~~~
/usr/include/c++/9/bits/stl_algo.h:2028:5: note:   template argument deduction/substitution failed:
main.cpp:9:47: note:   deduced conflicting types for parameter ‘_FIter’ (‘int’ and ‘int*’)
    9 |     cout << lower_bound(array[0], array + 5, 4) << endl;
      |                                               ^
```

报错信息说的是两件事：

1. candidate 发生了 substitution failure。
2. 原因是 `_ForwardIterator` 第一个参数为 `int` 类型，第二个参数为 `int*` 类型，冲突了。没有任何一个 candidate 匹配，因此还没有例化 (instantiation) 就已经替换失败 (substitution failure) 导致编译错误 (error) 了。

<aside>

为什么这里的报错内容是 `_FIter`，但点进去看 `/usr/include/c++/9/bits/stl_algo.h:2028:5` 的内容是 `_ForwardIter` 呢？
stl_algo.h 文件开始的地方 include 了一个名为 bits/algorithmfwd.h 的头文件，里面是所有模板函数的声明。因为前面提到了，模板函数的定义不能在 .cpp 文件中，否则会造成链接错误。因此 stl_algo.h 是定义，bits/algorithmfwd.h 是声明。通过将 bits/algorithmfwd.h include 到 stl_algo.h 中实现模板函数定义在头文件中。

由此可见，g++ 在寻找 candidate 的时候，使用的是声明头文件中的信息，因此在 candidate 替换失败时，报错信息使用的 typename parameter 是声明时写的 `_FIter`。但是用户要查看具体错误定位，这时候就最好定位到定义中，所以定位到了定义所在的头文件，而这两个文件的 typename parameter 名字不一样。

</aside>

如果 main 写成下面这样：

```cpp
int main() {
    int array[] = {0,1,2,5,6};
    cout << lower_bound(array[0], array[4], 4) << endl;
    return 0;
}
```

可以预见到前两个参数都是 `int` 类型，肯定不会冲突。因此替换 (substitution) 成功了，进入到例化 (instantiation) 阶段。

```
In file included from /usr/include/c++/9/algorithm:61,
                 from main.cpp:1:
/usr/include/c++/9/bits/stl_algobase.h: In instantiation of ‘_ForwardIterator std::__lower_bound(_ForwardIterator, _ForwardIterator, const _Tp&, _Compare) [with _ForwardIterator = int; _Tp = int; _Compare = __gnu_cxx::__ops::_Iter_less_val]’:
/usr/include/c++/9/bits/stl_algobase.h:1013:32:   required from ‘_ForwardIterator std::lower_bound(_ForwardIterator, _ForwardIterator, const _Tp&) [with _ForwardIterator = int; _Tp = int]’
main.cpp:9:46:   required from here
/usr/include/c++/9/bits/stl_algobase.h:970:2: error: no type named ‘difference_type’ in ‘struct std::iterator_traits<int>’
  970 |  _DistanceType;
      |  ^~~~~~~~~~~~~
```

这次报错变成了在例化过程中发现 `struct std::iterator_traits<int>` 并没有 `difference_type` 这个“内嵌类型 (nested type)”。我们看下 `__lower_bound` 的具体实现：

```cpp
template<typename _ForwardIterator, typename _Tp, typename _Compare>
    _ForwardIterator
    __lower_bound(_ForwardIterator __first, _ForwardIterator __last,
		  const _Tp& __val, _Compare __comp)
    {
      typedef typename iterator_traits<_ForwardIterator>::difference_type
	_DistanceType;

      _DistanceType __len = std::distance(__first, __last);

      while (__len > 0)
	{
	  _DistanceType __half = __len >> 1;
	  _ForwardIterator __middle = __first;
	  std::advance(__middle, __half);
	  if (__comp(__middle, __val))
	    {
	      __first = __middle;
	      ++__first;
	      __len = __len - __half - 1;
	    }
	  else
	    __len = __half;
	}
      return __first;
    }
```

确实有一个 `typedef typename iterator_traits<_ForwardIterator>::difference_type _DistanceType;` 的语句，是为了能定义 `_first` 和 `_last` 之间的距离的。

看起来在 g++ 9.4.0 对应的默认 C++ 标准中（经查询，其默认 C++ 标准为 C++14）的解决办法是利用 type traits，如果对应的类型 `_ForwardIterator` 没有 `difference_type` 这个类型，那么就会出现例化失败，进而导致编译错误。那么 Type Traits 是如何实现的呢？

## C++20以前：Type Traits

参考内容：

https://leimao.github.io/blog/CPP-Traits/

Type traits 是一种用来实现 template metaprogramming 的方法。如果不使用 traits，将需要使用特化 (specialization) ，重复书写大量类似的代码。

比如想要实现一个函数，叫做 `greater_than_half_maximum`，返回各数值类型的值是否过半。

如果我们不使用 traits，实现起来将会是：

```cpp
#include <cfloat>

template <typename T>
bool greater_than_half_maximum(T value)
{
    return false;
}

template <>
bool greater_than_half_maximum<int32_t>(int32_t value)
{
    // 2^31 - 1
    if (value > 2147483647 / 2)
    {
        return true;
    }
    else
    {
        return false;
    }
}

template <>
bool greater_than_half_maximum<int64_t>(int64_t value)
{
    // 2^63 - 1
    if (value > 9223372036854775807 / 2)
    {
        return true;
    }
    else
    {
        return false;
    }
}

template <>
bool greater_than_half_maximum<float>(float value)
{
    // FLT_MAX defined in cfloat
    if (value > FLT_MAX / 2)
    {
        return true;
    }
    else
    {
        return false;
    }
}

template <>
bool greater_than_half_maximum<double>(double value)
{
    // DBL_MAX defined in cfloat
    if (value > DBL_MAX / 2)
    {
        return true;
    }
    else
    {
        return false;
    }
}

// Even more specializations for different types.
// ...
```

如果我们使用 `std::numeric_limits` 这个 type_trait，针对 `greater_than_half_maximum` 的实现就可以简化为下面这样。并且如果这个类型 `T` 不是数值类型，`std::numeric_limits<T>::max()` 一定会报错（类似上面迭代器例子中的 `std::iterator_traits<T>::difference_type`），类型限制依旧存在。

```cpp
#include <limits>

template <typename T>
bool greater_than_half_maximum(T value)
{
    if (value > std::numeric_limits<T>::max() / 2)
    {
        return true;
    }
    else
    {
        return false;
    }
}
```

在这种实现下，如果我们要添加新的数值类型，比如 NVIDIA `__half` ，为了让它支持 `greater_than_half_maximum` 函数，不需要进行 high-level specialization，而是实现 low-level type traits，即实现 `numeric_limits<__half>::max()` 。

```cpp
#include <cstdint>
#include <limits>

namespace std
{
template <>
class numeric_limits<__half>
{
public:
    constexpr static __half max()
    {
        constexpr uint16_t const half_max_bits{0x7BFF};
        __half const half_max{*reinterpret_cast<__half const*>(&half_max_bits)};
        return half_max;
    };
};
} // namespace std
```

当然，使用 type traits 无非是在 low-level 层次进行了特化，这些 low-level 实现还是要针对不同的类型进行特化。但是由于这些 type traits 的可复用性很高，对它们进行特化比在 high-level 进行特化的代价要小很多。

你可能会好奇，上文中的 `iterator_traits<T>` 和 `numeric_limits<T>` 究竟是什么？他们又怎么和类型 `T` 配合工作呢？

首先，他们都是一个 struct。具体实现和编译器支持的 C++ 标准有关。

```cpp
template< class Iter >
struct iterator_traits;
template< class T >
struct iterator_traits<T*>;

template<
    class Category,
    class T,
    class Distance = std::ptrdiff_t,
    class Pointer = T*,
    class Reference = T&
> struct iterator;

#if __cplusplus >= 201103L
  // _GLIBCXX_RESOLVE_LIB_DEFECTS
  // 2408. SFINAE-friendly common_type/iterator_traits is missing in C++14
  template<typename _Iterator, typename = __void_t<>>
    struct __iterator_traits { };

#if ! __cpp_lib_concepts

  template<typename _Iterator>
    struct __iterator_traits<_Iterator,
			     __void_t<typename _Iterator::iterator_category,
				      typename _Iterator::value_type,
				      typename _Iterator::difference_type,
				      typename _Iterator::pointer,
				      typename _Iterator::reference>>
    {
      typedef typename _Iterator::iterator_category iterator_category;
      typedef typename _Iterator::value_type        value_type;
      typedef typename _Iterator::difference_type   difference_type;
      typedef typename _Iterator::pointer           pointer;
      typedef typename _Iterator::reference         reference;
    };
#endif // ! concepts

/****
....
***/
    
#if __cplusplus > 201703L
  /// Partial specialization for object pointer types.
  template<typename _Tp>
#if __cpp_concepts >= 201907L
    requires is_object_v<_Tp>
#endif
    struct iterator_traits<_Tp*>
    {
      using iterator_concept  = contiguous_iterator_tag;
      using iterator_category = random_access_iterator_tag;
      using value_type	      = remove_cv_t<_Tp>;
      using difference_type   = ptrdiff_t;
      using pointer	      = _Tp*;
      using reference	      = _Tp&;
    };
#else
  /// Partial specialization for pointer types.
  /// 提供对原生指针的支持
  template<typename _Tp>
    struct iterator_traits<_Tp*>
    {
      typedef random_access_iterator_tag iterator_category;
      typedef _Tp                         value_type;
      typedef ptrdiff_t                   difference_type;
      typedef _Tp*                        pointer;
      typedef _Tp&                        reference;
    };

  /// Partial specialization for const pointer types.
  template<typename _Tp>
    struct iterator_traits<const _Tp*>
    {
      typedef random_access_iterator_tag iterator_category;
      typedef _Tp                         value_type;
      typedef ptrdiff_t                   difference_type;
      typedef const _Tp*                  pointer;
      typedef const _Tp&                  reference;
    };
#endif
```

看到这里你可能有疑问，为什么明明都已经有了 `_Iterator::xxxx` ，还需要包裹一层 `iterator_traits::xxxx` 呢？如果 C++ 里只有迭代器，没有裸指针，确实不需要这个 `iterator_traits`。但 C++ 为了在能使用迭代器的地方使用裸指针，而裸指针类型并没有 `Tp*::xxxx` ，所以才需要用 `iterator_traits` 进行包裹，并对裸指针类型进行特化。

同时，如果我们需要为某个现有的、不能修改的迭代器类提供额外的类型别名，可以通过对`std::iterator_traits` 进行特化来实现，而不需要改动原来的迭代器定义。`iterator_traits` 相当于一个 wrapper，让各个迭代器对外暴露的类型别名一致，而实际的逻辑隐藏在 `iterator_traits` 的特化逻辑中。*（核心思想：将 high level 的特化转移到 low level 的 traits 特化）*

```cpp

template<typename _Tp>
    struct numeric_limits : public __numeric_limits_base
    {
      /** The minimum finite value, or for floating types with
	  denormalization, the minimum positive normalized value.  */
      static _GLIBCXX_CONSTEXPR _Tp
      min() _GLIBCXX_USE_NOEXCEPT { return _Tp(); }

      /** The maximum finite value.  */
      static _GLIBCXX_CONSTEXPR _Tp
      max() _GLIBCXX_USE_NOEXCEPT { return _Tp(); }

/****

specialization for all the primitive types

*****/
/// numeric_limits<int> specialization.
  template<>
    struct numeric_limits<int>
    {
      static _GLIBCXX_USE_CONSTEXPR bool is_specialized = true;

      static _GLIBCXX_CONSTEXPR int
      min() _GLIBCXX_USE_NOEXCEPT { return -__INT_MAX__ - 1; }

      static _GLIBCXX_CONSTEXPR int
      max() _GLIBCXX_USE_NOEXCEPT { return __INT_MAX__; }

#if __cplusplus >= 201103L
      static constexpr int
      lowest() noexcept { return min(); }
#endif

      static _GLIBCXX_USE_CONSTEXPR int digits = __glibcxx_digits (int);
      static _GLIBCXX_USE_CONSTEXPR int digits10 = __glibcxx_digits10 (int);
#if __cplusplus >= 201103L
      static constexpr int max_digits10 = 0;
#endif
      static _GLIBCXX_USE_CONSTEXPR bool is_signed = true;
      static _GLIBCXX_USE_CONSTEXPR bool is_integer = true;
      static _GLIBCXX_USE_CONSTEXPR bool is_exact = true;
      static _GLIBCXX_USE_CONSTEXPR int radix = 2;

      static _GLIBCXX_CONSTEXPR int
      epsilon() _GLIBCXX_USE_NOEXCEPT { return 0; }

      static _GLIBCXX_CONSTEXPR int
      round_error() _GLIBCXX_USE_NOEXCEPT { return 0; }

      static _GLIBCXX_USE_CONSTEXPR int min_exponent = 0;
      static _GLIBCXX_USE_CONSTEXPR int min_exponent10 = 0;
      static _GLIBCXX_USE_CONSTEXPR int max_exponent = 0;
      static _GLIBCXX_USE_CONSTEXPR int max_exponent10 = 0;

      static _GLIBCXX_USE_CONSTEXPR bool has_infinity = false;
      static _GLIBCXX_USE_CONSTEXPR bool has_quiet_NaN = false;
      static _GLIBCXX_USE_CONSTEXPR bool has_signaling_NaN = false;
      static _GLIBCXX_USE_CONSTEXPR float_denorm_style has_denorm
       = denorm_absent;
      static _GLIBCXX_USE_CONSTEXPR bool has_denorm_loss = false;

      static _GLIBCXX_CONSTEXPR int
      infinity() _GLIBCXX_USE_NOEXCEPT { return static_cast<int>(0); }

      static _GLIBCXX_CONSTEXPR int
      quiet_NaN() _GLIBCXX_USE_NOEXCEPT { return static_cast<int>(0); }

      static _GLIBCXX_CONSTEXPR int
      signaling_NaN() _GLIBCXX_USE_NOEXCEPT { return static_cast<int>(0); }

      static _GLIBCXX_CONSTEXPR int
      denorm_min() _GLIBCXX_USE_NOEXCEPT { return static_cast<int>(0); }

      static _GLIBCXX_USE_CONSTEXPR bool is_iec559 = false;
      static _GLIBCXX_USE_CONSTEXPR bool is_bounded = true;
      static _GLIBCXX_USE_CONSTEXPR bool is_modulo = false;

      static _GLIBCXX_USE_CONSTEXPR bool traps = __glibcxx_integral_traps;
      static _GLIBCXX_USE_CONSTEXPR bool tinyness_before = false;
      static _GLIBCXX_USE_CONSTEXPR float_round_style round_style
       = round_toward_zero;
    };
```

使用 type traits 可以进行 template metaprogramming。两个常用的函数是 `std::enable_if` 和 `std::conditional` 。

```cpp
template< bool B, class T = void >
struct enable_if;   // 		(since C++11)

// If B is true, std::enable_if has a public member typedef type, equal to T; otherwise, there is no member typedef.
// If the program adds specializations for std::enable_if, the behavior is undefined.

template< bool B, class T = void >
using enable_if_t = typename enable_if<B,T>::type; // (since C++14)
```

可能的实现：

```cpp
template<bool B, class T = void>
struct enable_if {};
 
template<class T>
struct enable_if<true, T> { typedef T type; };
```

使用 enable_if 作为 template parameter 时需要注意重定义的问题：

```cpp
#include <type_traits>

struct T
{
    enum { int_t, float_t } type;
 
    template<typename Integer,
             typename = std::enable_if_t<std::is_integral<Integer>::value>>
    T(Integer) : type(int_t) {}
 
    template<typename Floating,
             typename = std::enable_if_t<std::is_floating_point<Floating>::value>>
    T(Floating) : type(float_t) {} // error: treated as redefinition
};
```

会出现重定义错误，并没有重载成功：

```bash
<source>:13:5: error: 'template<class Floating, class> T::T(Floating)' cannot be overloaded with 'template<class Integer, class> T::T(Integer)'
   13 |     T(Floating) : type(float_t) {} // error: treated as redefinition
      |     ^
<source>:9:5: note: previous declaration 'template<class Integer, class> T::T(Integer)'
    9 |     T(Integer) : type(int_t) {}
      |     ^
Compiler returned: 1
```

因为两个函数的 template 实际上完全一致，都是 `template<class T, class>` 

实际上，如果上面的代码写成

```cpp

struct T
{
    enum { int_t, float_t } type;
 
    template<typename Integer,
             typename = bool>
    T(Integer) : type(int_t) {}
 
    template<typename Floating,
             typename = int>
    T(Floating) : type(float_t) {} // error: treated as redefinition
};
```

会报一样的错。只是类的默认 argument 不一样而已，并不算做重载。

正确的写法应该是：

```cpp
struct T
{
    enum { int_t, float_t } type;
 
    template<typename Integer,
             std::enable_if_t<std::is_integral<Integer>::value, bool> = true>
    T(Integer) : type(int_t) {}
 
    template<typename Floating,
             std::enable_if_t<std::is_floating_point<Floating>::value, bool> = true>
    T(Floating) : type(float_t) {} // OK
};
```

虽然这两个参数在第二个构造函数模板中的类型 `bool` 是相同的，但由于它们各自的 `enable_if_t` 条件根据传入的类型是整数还是浮点数在编译时得到不同的结果（一个适用于整数，另一个适用于浮点数），因此允许重载。

这就允许构造函数根据传入参数的类型（是整数类型还是浮点类型）被正确选择和实例化，编译器不会混淆两个构造函数模板，避免了重定义的错误。`enable_if` 正体现了 SFINAE 原则。

`[std::conditional](https://en.cppreference.com/w/cpp/types/conditional)` 的使用是类似的，这里就不再讲述。

# 参考内容

- https://leimao.github.io/blog/CPP-Traits/
- https://leimao.github.io/blog/CPP-Enable-If/
- https://en.cppreference.com/w/cpp/types/conditional
- https://en.cppreference.com/w/cpp/types/enable_if
- https://en.cppreference.com/w/cpp/algorithm/lower_bound
- https://en.cppreference.com/w/cpp/language/template_parameters