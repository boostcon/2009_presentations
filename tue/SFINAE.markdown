# Substitution Failure Is Not An Error

Last Change: 2009-05-01 07:34:23 by Doug Gregor

Substitution Failure Is Not An Error, or SFINAE for short, is a feature of C++ that protects against errors while trying to instantiate templates during overload resolution of partial ordering of class template partial specializations. SFINAE itself is usually invisible, eliminating overloaded function templates that don't work for a given call without triggering any compiler errors, but SFINAE has also become a powerful mechanism for template libraries. For example, the deceptively-simple [enable_if](http://www.boost.org/doc/libs/1_38_0/libs/utility/enable_if.html) facility makes it possible to enable or disable a given function template based on the result of a template metaprogram:

    template<typename T>
    typename boost::enable_if_c<boost::is_pod<T>::value>::type
    accepts_pod_types(const T&);

Here, the function template `accepts_pod_types` can only be instantiated for POD types (where `boost::is_pod<T>::value` is true). The actual trick behave `enable_if_c` is quite simple, and the entire facility looks like this:

    template<bool Cond, typename T = void>
    struct enable_if_c {
      typedef T type;
    };

    template<typename T>
    struct enable_if_c<false, T> { };

Note here that, when the condition is false, there is no `type` member within the `enable_if_c` structure. So, when `accepts_pod_type` receives a POD type, its return type is `void`; when `accepts_pod_type` receives a non-POD type, the lookup for `type` within the `enable_if_c` instantiation fails, but SFINAE makes that failure silent (no compiler diagnostic) and merely makes the `accepts_pod_type` function template disappear from the overload set.

There is a relatively long list in the current C++ standard that says what kinds of constructs fall under the SFINAE rule, and trying to find a type member when no such member exists is just one such rule. However, SFINAE is still rather simplistic, and for a large class of potential failures---such as an ambiguity in overload resolution or the inability to find any viable function when evaluation an expression---cause compiler errors rather than triggering the SFINAE rule. The addition of [Decltype] to the language made this issue all the more urgent:

    template<typename T, typename U>
    auto add(T&& x, U&& y) -> decltype(std::forward<T>(x) + std::forward<U>(y)) {
       return std::forward<T>(x) + std::forward<U>(y); 
   }

Imagine that we call `add(z, z)`, where `z` is of a type `Z` that does not have an `operator+`. However, the author of `Z` does introduce an `add` function so that a third party, which relies on `add`, can use it:

    class Z { ... };
    Z add(Z, Z);

    template<typename T> T double_me(T x) { return add(x, x); }

With the limited SFINAE rules provided by the current C++ standard, instantiating `double_me<Z>` would cause a problem: the simple, non-templated `add(Z, Z)` is clearly the best option, but the compiler will also try to instantiate the declaration for the template function `add<Z, Z>`. Unfortunately, that instantiation will fail inside the `decltype` expression, because one cannot add two objects of type `Z`!

The extended SFINAE rules of C++0x generalize SFINAE to work with arbitrary expressions. If, while instantiating the signature of a function template, type-checking failures for (almost!) any reason, then the SFINAE rule applies and the compiler will silently eliminate that function template from consideration. In the `add` example, that means that the compiler's inability to find a suitable operator `+` adding two objects of type `Z` means that the function template `add<Z, Z>` will not be considered by overload resolution at all. Then, the instantiation of `double_me<Z>` succeeds.

Extended SFINAE opens up many new opportunities for detecting the presence or absence of certain operations. Since it applies to *any* expression or type construction, one can imagine building traits that detect if various operators (or sets of operators) are available:

    typedef char yes_type;
    typedef char (&no_type)[4];

    template<typename> Capture;

    template<typename T, typename U>
    inline yes_type try_add(int, Capture<decltype(T() + U()>* = 0);

    template<typename T, typename U>
    inline no_type try_add(...);

    template<typename T, typename U>
    struct is_addable {
      static const bool value = (sizeof(try_add<T, U>(0)) == sizeof(yes_type));
    };

Here, the `yes_type` overload of `try_add` will be the better match for the argument '0', but it will only instantiate properly if `T` and `U` have a suitable `+` operator. (And if they both have default constructors, but that is an artifact of the poor implementation above). If that `try_add` fails, the second `try_add` will be used instead.

Previously, when we mentioned that SFINAE applied to any kind of failure, we said *almost* any kind of failure. There are two classes of errors for which SFINAE still does not apply:

  1. If the type-checking failure is due an access violation (e.g., `x + y` resolves to a private `operator+`), that error is a "hard" error that does not invoke SFINAE.

  2. If the type-checking failure occurs within the instantiation of a class template or in the body of a function template, that error is a "hard" error that does not invoke SFINAE.

The first of these rules is meant to fit into the spirit of access checking, which is that access checking only applies after all other type-checking is done. The second rule is meant to somewhat limit the impact that the extended SFINAE rule will have on the architecture of existing compilers, which will already need to stretch to meet these new requirements.

Overall, the extended SFINAE rules in C++0x make an already-powerful C++ feature into something that opens up new doors for template metaprogramming and yet more advanced template libraries. And it does so without changing a single bit of C++ syntax.
