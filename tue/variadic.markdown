# Variadic Templates

Start/HandsOnCXX0x/VariadicTemplates - Last Change: 2009-05-01 06:46:25 by Doug Gregor

Variadic templates allow a class or function template to accept some number (possibly zero) of "extra" template arguments. This behavior can be simulated for class templates in C++ via a long list of defaulted template parameters, e.g., a typical implementation for Boost's `tuple` looks like this: 

    struct unused; 

    template<typename T1 = unused, typename T2 = unused, 
             typename T3 = unused, typename T4 = unused, 
             /* up to */ typename TN = unused> 
      class tuple; 

This tuple type can be used with anywhere from zero to N template arguments. Assuming that N is "large enough", we could write: 

    typedef tuple<char, short, int, long, long long> integral_types; 

Of course, this tuple instantiation actually contains N - 5 unused parameters at the end, but presumably the implementation of tuple will somehow ignore these extra parameters. In practice, this means changing the representation of the argument list or providing a host of partial specializations: 

    template<> 
    class tuple<> 
    { /* handle zero-argument version. */ }; 

    template<typename T1> 
    class tuple<T1> 
    { /* handle one-argument version. */ }; 

    template<typename T1, typename T2> 
    class tuple<T1, T2> 
    { /* handle two-argument version. */ }; 

This technique is used by various Boost libraries, including Bind, Function, MPL, Phoenix, and Type Traits. With variadic templates, we can do better.

## Variadic Class Templates

Variadic templates let us explicitly state that tuple accepts one or more template type parameters. Using variadic templates, the `tuple` template above becomes: 

    template<typename... Elements> class tuple; 

The ellipsis to the left of the `Elements` identifier indicates that `Elements` is a *template type parameter pack.  A *parameter pack* is a new kind of entity introduced by variadic templates. Unlike a parameter, which can only bind to a single argument, multiple arguments can be "packed" into a single parameter pack. With the `integral_types` typedef in the introduction, the `Elements` parameter pack would get bound to a list of template arguments containing `char`, `short`, `int`, `long`, and `long long`. Aside from template parameter packs for template type arguments, variadic class templates can also have parameter packs for non-type and template template arguments. For instance, we could declare an array class that supports an arbitrary number of dimensions: 

    template<typename T, unsigned PrimaryDimension, unsigned... Dimensions> 
    class array 
    { /* implementation */ }; 

    array<double, 3, 3> rotation matrix; // 3x3 rotation matrix 

For rotation matrix, the template parameters for array are deduced as `T=double`, `PrimaryDimension=3`, and `Dimensions` contains 3. Note that the argument to `Dimensions` is actually a list of template non-type arguments, even though that list contains only a single element. 

## Packing and Unpacking Parameter Packs 

It is easy to declare a variadic class template, but so far we haven't done anything with the extra arguments that are passed to our template. To do anything with parameter packs, one must unpack (or expand) all of the arguments in the parameter pack into separate arguments, to be forwarded on to another template. To illustrate this process, we will define a template that counts the number of template type arguments it was 
given: 

    template<typename... Args> struct count; 

Our first step is to define a basis case. If count is given zero arguments, this full specialization will set the value member to zero: 

    template<> 
    struct count<> 
    { 
      static const int value = 0; 
    };

Next we want to define the recursive case. The idea is to peel off the first template type argument provided to count, then pack the rest of the arguments into a template parameter pack to be counted in the final sum: 

    template<typename T, typename... Args> 
    struct count<T, Args...> 
    { 
      static const int value = 1 + count<Args...>::value; 
    }; 

Whenever the ellipsis occurs to the right of a template argument, it is acting as a meta-operator that unpacks the parameter pack into separate arguments. In the expression `count<Args...>::value`, this means that all of the arguments that were packed into `Args` will become separate arguments to the `count` template. The use of the ellipsis for unpacking a parameter pack also occurs in the partial specialization, `count<T, Args...>`, where template argument deduction packs "extra" parameter passed to `count` into `Args`. For instance, the instantiation of `count<char, short, int>` would select this partial specialization, binding `T` to `char` and `Args` to a list containing `short` and `int`. 

The `count` template is a recursive algorithm. The partial specialization pulls off one argument (`T`) from the template argument list, placing the remaining arguments in `Args`. It then computes the length of `Args` by instantiating `count<Args...>` and adding one to that value. When `Args` is eventually empty, the instantiation 
`count<Args...>` selects the full specialization `count<>` (which acts as the basis case), and the recursion terminates. The use of the ellipsis operator for unpacking allows us to implement `tuple` with variadic templates: 

    template<typename... Elements> class tuple; 

    template<typename Head, typename... Tail> 
    class tuple<Head, Tail...> : private tuple<Tail...> 
    { 
      Head head; 
      public: 
      / * implementation */ 
    }; 

    template<> 
    class tuple<> 
    { 
      /* zero-tuple implementation */ 
    }; 

Here, we use recursive inheritance to store a member for each element of the parameter pack. Each instance of `tuple` peels off the first template argument, creates a member `head` of that type, and then derives from a `tuple` containing the remaining template arguments. 

## Variadic Function Templates


C's variable-length function parameter lists allow an arbitrary number of "extra" arguments to be passed to the function (sometimes called a "varargs function" or a "variadic function"). This feature is rarely used in C++ code (except for compatibility with C libraries), because passing non-POD types through an ellipsis ("...") invokes undefined behavior. For instance, this simple code will likely crash at run-time: 

    const char *msg = "The value of %s is about %g (unless you live in %s)"; 
    printf(msg, std::string("pi"), 3.14159, "Indiana");2 

Variadic templates also allow an arbitrary number of "extra" arguments to be passed to a function, but retain complete type information. For instance, the following function accepts any number of arguments, each of which may have a different type: 

    template<typename... Args> void eat(Args... args); 

The `eat()` function is a variadic template with template parameter pack `Args`. However, the function parameter list is more interesting: the ellipsis to the left of the identifier `args` indicates that `args` is a function parameter pack. Function parameter packs can accept any number of function arguments, the types of which are in the template type parameter pack `Args`. Consider, for instance, a call 

    eat(17, 3.14159, string("Hello!")): 

the three function call arguments will be packed, so `args` contains `17`, `3.14159` and `string("Hello!")` while `Args` contains the types of those arguments: `int`, `double`, and `string`. The effect is precisely the same as if we had just written a three-argument version of `eat()` like the following, except without the limitation to three arguments: 

    template<typename Arg1, typename Arg2, typename Arg3> 
    void eat(Arg1 arg1, Arg2 arg2, Arg3 arg3); 

Variadic function templates allow us to express a type-safe `printf()` that works for non-POD types. The following code uses variadic templates to implement a C++-friendly, type-safe `printf()`: 

    void printf(const char * s) { 
      while ( *s) { 
        if ( *s == '%' && *++s != '%') 
          throw std::runtime error("invalid format string: missing arguments");
        std::cout << *s++; 
      } 
    } 

    template<typename T, typename... Args> 
    void printf(const char * s, T value, Args... args) { 
      while ( *s) { 
        if ( *s == '%' && *++s != '%') { 
          std::cout << value; 
          return printf(++s, args...); 
        } 
        std::cout << *s++; 
      } 
      throw std::runtime error("extra arguments provided to printf"); 
    } 

There is only one innovation in this implementation of `printf()`, but we've seen it before: in the recursive call to `printf()`, we unpack the function parameter pack args with the syntax `args...`. Just like unpacking a template parameter pack (e.g., for `tuple` or `count`), the use of the ellipsis meta-operator on the right unpacks the parameter pack into separate arguments. We've again used recursion, peeling an argument off the front of the list, processing it, then recursing to handle the rest of the arguments. 

## Packing and Unpacking, Revisited 

When using the ellipsis meta-operator on the right to unpack a template parameter pack, we can employ arbitrary unpacking patterns containing the parameter pack. These patterns will be expanded for each argument in the parameter pack, or used to direct how template argument deduction proceeds. For example, our `printf()` function above should really have taken its arguments be `const` reference, because copies are unnecessary and could be expensive. So, we change the declaration of `printf()` to: 

    template<typename T, typename... Args> 
    void printf(const char * s, T value, const Args&... args); 

In fact, when the ellipsis operator is used on the right, the argument can be arbitrarily complex and can even include multiple parameter packs. In conjunction with [rvalue references](https://www.boostpro.com/trac/wiki/BoostCon09/RValue101), this allows us to implement perfect forwarding for any number of arguments, for instance allowing an allocator's `construct()` operation to work even when constructing a noncopyable type: 

    template<typename T> 
    struct allocator 
    { 
      template<typename... Args> 
      void construct(T * ptr, Args&&... args) { 
        new (ptr)(std::forward<Args>(args)...); 
      } 
    };

In this example, we use placement new and call the constructor by forwarding each argument in `args` with the corresponding type in `Args`. That pattern will be expanded for all arguments, with both `Args` and `args` expanding concurrently. When used with two arguments, it essentially produces the same code as the following two-argument version of construct(): 

    template<typename T> 
    struct allocator 
    { 
      template<typename Arg1, typename Arg2> 
      void construct(T * ptr, Arg1&& arg1, Arg2&& arg2) { 
        new (ptr)(std::forward<Arg1>(arg1), std::forward<Arg2&&>(arg2)); 
      } 
    }; 

This same approach can be used to mimic inheritance of constructors, e.g., 

    template<typename Base> 
    struct adaptor : public Base 
    { 
      template<typename... Args> 
      adaptor(Args&&... args) : Base(std::forward<Args>(args)...) 
      { } 
    }; 

An unpacking pattern can occur at the end of any template argument list. This also means that multiple parameter packs can occur in the same template. For instance, one can declare a `tuple` equality operator than can compare tuples that store different sets of elements: 

    template<typename... Elements1, typename... Elements2> 
    bool operator==(const tuple<Elements1...>& t1, const tuple<Elements2...>& t2); 

This function is a variadic template (it contains template parameter packs), but the length of the function parameter list is fixed (it contains no function parameter packs). It uses two template parameter packs, one for the arguments to each `tuple`. If we wanted our equality comparison to only consider tuples with precisely the same element types, we would have written: 

    template<typename... Elements> 
    bool operator==(const tuple<Elements...>& t1, const tuple<Elements...>& t2); 

Similarly, one can use a template type parameter pack at the end of a function parameter list in a function type, e.g., 

    template<typename Signature> struct result type; 

    // Matches any function pointer
    template<typename R, typename Args...> 
    struct result type<R( *)(Args...)> { 
      typedef R type; 
    }; 

    // Matches a member function pointer with no cv-qualifiers 
    template<typename R, typename Class, typename Args...> 
    struct result type<R(Class:: *)(Args...)> { 
      typedef R type; 
    };

We've mentioned several places where parameter packs can be unpacked, such as template arguments lists, function call arguments, and function parameter types. In fact, there are several more contexts where one can unpack a parameter pack:

  + In an argument to a template: 

      tuple<Args...>

  + In an argument to a function: 

      print(args...)

  + In a function type's parameter list: 

      R (*)(Args...)

  + In a special sizeof expression that determines the number of arguments in a parameter pack:

      sizeof...(Args)

  + In a base class list:

      class \MyClass : public Mixins...

  + In a base-class initializer list:

      my_class(Args... args) : Mixins(args)...

  + In an exception specification:

      throw(Exceptions...)

  + In an initializer list:

      boost::any array[] = { args... };

## Wrap-Up

Use variadic templates when you need to eliminate repeated template parameters in a class or function template. The three keys to successful use of variadic templates are:

  + An ellipsis to the left of a parameter name is a parameter pack, which acceptes zero or more template arguments and bundles them into a single pack.

  + An ellipsis to the right of an argument is a pack expansion, which splits a pattern (containing one or more parameter packs) into separate arguments to be processed by another template.

  + Recursion allows a parameter pack to be unwrapped piece-by-piece, so that each argument within the parameter pack can be processed individually by a template metaprogram.

For more information about variadic templates, we suggest looking at the examples in the [variadic templates proposal](http://www.generic-programming.org/~dgregor/cpp/variadic-templates.pdf), from which this tutorial was taken. The proposal contains a set of extended examples that model the use of variadic templates in various Boost libraries (or their TR1 counterparts).

