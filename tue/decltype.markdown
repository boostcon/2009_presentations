# Declared Type of an Expression (decltype)

Decltype - Last Change: 2009-05-01 06:33:07 by Doug Gregor 

The C++0x `decltype`feature allows one to compute the declared type of an expression. Computing the type of an expression is a common need in the lower-level layers of template libraries, and is currently handled via conventions (e.g., `result_of`) or the Boost Typeof library.

`decltype` is a simple facility, which takes an expression and computes its type. Specifically, `decltype(e)` determines the type of an expression e using the following rules:

  + If `e` is an id-expression, `decltype(e)` is the type of the entity named, An id-expression is, simply, an expression that directly refers to some kind of declaration, such as a variable, function, or data member.
  + If `e` is a function call, `decltype(e)` is the return type of the function called,
  + Otherwise, `T` is the type of `e`, and
    + if `e` is an lvalue, `decltype(e)` is `T&`,
    + if `e` is an rvalue, `decltype(e)` is `T`

In most template libraries, the second and third bullets are most important, because `decltype` allows us to answer the following question: if I evaluate some expression (such as `x + y`, or `f(x)`), what return type should I use to return that result from my function? For example, if we were to build a forwarding function `add` to add values of types `T` and `U`, we might write it as:

    template<typename T, typename U>
    decltype(T() + U()) add(const T& x, const U& y) { return x + y; }

Here, the result type of `add()` depends entirely on the types `T` and `U`, and of course what `+` operators are available for these types. If `T` and `U` are `short` and `int`, for example, the type of `add()` will be `int` because that is the type of the expression. On the other hand, if both `T` and `U` are of type `X`, which has an overloaded `operator+` such as

    X operator+(X, X);

then the return type of `add()` will be `X`. 

While the rules behind `decltype`may seem somewhat obscure, they were (carefully!) crafted to ensure that one can "perfectly" forward the return type of some computation, making it easier to build forwarding and adaptor functions. This forwarding of the return type complements the "perfect" forwarding of argument types provided by rvalue references. Together, these features make it possible to build true forwarding functions. `add`, for example, could look like this:

    template<typename T> struct get_a { static T object; };

    template<typename T, typename U>
    decltype(std::forward<T>(get_a<T&&>::object) + std::forward<U>(get_a<U&&>::object))
    add(T&& x, U&& y) { return std::forward<T>(x) + std::forward<U>(y); }

This formulation is clumsy, but effective: it can accept lvalues or rvalues, and forwards them on to the underlying `+` operation exactly the way they came in, so that the syntax `add(x, y)` is almost entirely indistinguishable from the syntax `x + y`. 

The clumsiness of the syntax above is somewhat ameliorated by the "new"  function declarator syntax in C++0x, which places the return type *after* the function parameter list, so that you can refer to function parameters in the return type. This simplifies the formulation of `add` by eliminating the rather unsightly `get_a` template:

    template<typename T, typename U>
    auto add(T&& x, U&& y) -> decltype(std::forward<T>(x) + std::forward<U>(y)) {
       return std::forward<T>(x) + std::forward<U>(y); 
    }

One still has to duplicate the expression from the body, but it's easier to get right.

`decltype` is a simple but powerful facility. Use it to compute the type of an expression when you need to return that expression from a generic function or to query some other property of the type of that expression.
