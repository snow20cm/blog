+++
categories = ["notes"]
date = "2015-07-20T20:27:50-06:00"
tags = ["cpp"]
title = "Efffective Modern C++"

+++


Chapter 1 Deducing Types
========================
```cpp
template<typename T>
void f(ParamType param);

f(expr);
```

It’s natural to expect that the type deduced for T is the same as the type of the argument passed to the function, i.e., that T is the type of expr. But it doesn’t always work that way. The type deduced for T is dependent not just on the type of expr(aurgument), but also on the form of ParamType(parameter). There are three cases:

- ParamType is a pointer or reference type, but not a universal reference.
- ParamType is a universal reference.
- ParamType is neither a pointer nor a reference.

### Case 1: ParamType is a Reference or Pointer, but not a Universal Reference

Type deduction works like this:

1. If expr’s type is a reference, ignore the reference part.
2. Then pattern-match expr’s type against ParamType to determine T.

For example, if this is our template,

```cpp
template<typename T>
void f(T& param); // param is a reference
```

and we have these variable declarations,

```cpp
int x = 27; // x is an int
const int cx = x; // cx is a const int
const int& rx = x; // rx is a reference to x as a const int
```

the deduced types for param and T in various calls are as follows:

```cpp
f(x); // T is int, param's type is int&
f(cx); // T is const int, param's type is const int&
f(rx); // T is const int, param's type is const int&
```

NOTE:
- It’s important for caller that a const object remain unmodifiable.
- Type deduction works exactly the same way for rvalue reference parameters.
- If param were a pointer (or a pointer to const) instead of a reference, things would work essentially the same way

If we change the type of f’s parameter from T& to const T&, things change a little, but not in any really surprising ways. The constness of cx and rx continues to be respected, but because we’re now assuming that param is a reference-to-const, there’s no longer a need for const to be deduced as part of T.

### Case 2: ParamType is a Universal Reference

Universal References are declared like right references(T&&).

- If expr is an lvalue, both T and ParamType are deduced to be lvalue references.
    * It’s the only situation in template type deduction where T is deduced to be a reference.
    * Although ParamType is declared using the syntax for an rvalue reference, its deduced type is an lvalue reference.
- If expr is an rvalue, the “normal” (i.e., Case 1) rules apply

```cpp
template<typename T>
void f(T&& param); // param is now a universal reference

int x = 27; // as before
const int cx = x; // as before
const int& rx = x; // as before

f(x); // x is lvalue, so T is int&, param's type is also int&
f(cx); // cx is lvalue, so T is const int&, param's type is also const int&
f(rx); // rx is lvalue, so T is const int&, param's type is also const int&
f(27); // 27 is rvalue, so T is int, param's type is therefore int&&
```

### Case 3: ParamType is Neither a Pointer nor a Reference

1. As before, if expr’s type is a reference, ignore the reference part.
2. If, after ignoring expr’s reference-ness, expr is const, ignore that, too. If it’s volatile, also ignore that.

```cpp
template<typename T>
void f(T param); // param is now passed by value

int x = 27; // as before
const int cx = x; // as before
const int& rx = x; // as before

f(x); // T's and param's types are both int
f(cx); // T's and param's types are again both int
f(rx); // T's and param's types are still both int
```

#### expr is a const pointer to a const object, and expr is passed to a byvalue param

```cpp
template<typename T>
void f(T param); // param is still passed by value
const char* const ptr = // ptr is const pointer to const object
"Fun with pointers";
f(ptr); // pass arg of type const char * const
```


### array arguments

`f(int a[3])` is interpreted as `f(int *a)`.
`f(int (&a)[3])` is interpreted as a reference to array

```cpp
const char name[] = "J. P. Briggs"; // name's type is const char[13]

template<typename T>
void f(T param); // template with by-value parameter
f(name); // name is array, but T deduced as const char*

template<typename T>
void f(T& param); // template with by-reference parameter
f(name); // T deduced as const char [13]
```

## Things to Remember
- During template type deduction, arguments that are references are treated as non-references, i.e., their reference-ness is ignored.
- When deducing types for universal reference parameters, lvalue arguments get special treatment.
- When deducing types for by-value parameters, const and/or volatile arguments are treated as non-const and non-volatile.
- During template type deduction, arguments that are array or function names decay to pointers, unless they’re used to initialize references.




Item 2: Understand auto type deduction.
=======================================

auto type deduction is template type deduction

When a variable is declared using auto, auto plays the role of T in the template, and
the type specifier for the variable acts as ParamType
```cpp
auto x1 = 27; // type is int, value is 27
auto x2(27); // ditto
auto x3 = { 27 }; // type is std::initializer_list<int>,
                  // value is { 27 }
auto x4{ 27 }; // ditto
```
When the initializer for an auto-declared variable is enclosed in braces, the deduced type is a std::initializer_list. If such a type can’t be deduced (e.g., because the values in the braced initializer are of different types), the code will be rejected

The treatment of braced initializers is the only way in which auto type deduction and template type deduction differ. When an auto–declared variable is initialized with a braced initializer, the deduced type is an instantiation of std::initializer_list. But if the corresponding template is passed the same initializer, type deduction fails, and the code is rejected 

C\++14 permits auto to indicate that a function’s return type should be deduced (see Item 3), and C++14 lambdas may use auto in parameter declarations. However, these uses of auto employ template type deduction, not auto type deduction. 

## Things to Remember
- auto type deduction is usually the same as template type deduction, but auto type deduction assumes that a braced initializer represents a std::initializer_list, and template type deduction doesn’t.
- auto in a function return type or a lambda parameter implies template type deduction, not auto type deduction.




Item 3: Understand decltype
===========================

`decltype` almost always yields the type of a variable or expression without any modifications.

In C++11, perhaps the primary use for decltype is `trailing return type`.
```cpp
template<typename Container, typename Index> // works, but
auto authAndAccess(Container& c, Index i) // requires
 -> decltype(c[i]) // refinement
{
 authenticateUser();
 return c[i];
}
```

C\++11 permits return types for single-statement lambdas to be deduced, and C++14 extends this to both all lambdas and all functions, including those with multiple statements.

```cpp
template<typename Container, typename Index> // C++14;
auto authAndAccess(Container& c, Index i) // not quite
{ // correct
 authenticateUser();
 return c[i]; // return type deduced from c[i]
}
```

Note: In this example, there is a conversion from reference type of `c[i]`(Container::Type &) to value type (Container::Type). See Item 1. This might not what we want. Here we need `decltype(auto)` specifier from C++14.

`auto` specifies that the type is to be deduced, and `decltype` says that decltype rules should be used during the deduction.

## Difference between auto deduction and decltype deduction
Given a name or an expression, decltype tells you the name’s or the expression’s type

```cpp
Widget w;
const Widget& cw = w;
auto myWidget1 = cw; // auto type deduction:
                     // myWidget1's type is Widget
decltype(auto) myWidget2 = cw; // decltype type deduction:
                               // myWidget2's type is
                               // const Widget&
```

```cpp
template<typename Container, typename Index> 
auto authAndAccess(Container& c, Index i)
// auto deduction : c[i] is T&, but return type is T
{ 
 return c[i];
}

////////////////////

template<typename Container, typename Index> 
decltype(auto) 
authAndAccess(Container& c, Index i) 
// decltype deduction : c[i] is T&, but return type is T&
{ 
 return c[i];
}
```

## universal reference as parameter

lvalue reference as parameter type cannot handle rvalue

```cpp
//// c++14 version
template<typename Container, typename Index>
decltype(auto)
authAndAccess(Container&& c, Index i)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
//// c++11 version
template<typename Container, typename Index>
auto
authAndAccess(Container&& c, Index i)
-> decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

## Exception about `decltype`

If an lvalue expression other than a name has type T, decltype reports that type as T&

```cpp
int x = 0;
decltype(x); // int
decltype((x)); // int&

decltype(auto) f1()
{
 int x = 0;
 return x; // decltype(x) is int, so f1 returns int
}
decltype(auto) f2()
{
 int x = 0;
 return (x); // decltype((x)) is int&, so f2 returns int&
}
```

**f2 returnref to a local variable**


## Things to Remember

- decltype almost always yields the type of a variable or expression without any modifications.
- For lvalue expressions of type T other than names, decltype always reports a type of T&.
- C++14 supports decltype(auto) , which, like auto, deduces a type from its initializer, but it performs the type deduction using the decltype rules.



Item 4: Know how to view deduced types.
=======================================

## Output type information while compiling

```cpp
template<typename T> // declaration only for TD;
class TD; // TD == "Type Displayer"

TD<decltype(x)> xType; 
```

## Output type information while running

The Boost TypeIndex library is perfectly for printfing type information.

```cpp
#include <boost/type_index.hpp>
using boost::typeindex::type_id_with_cvr;

cout << type_id_with_cvr<decltype(param)>().pretty_name();
```

## Things to Remember
- Deduced types can often be seen using IDE editors, compiler error messages, and the Boost TypeIndex library.
- The results of some tools may be neither helpful nor accurate, so an under‐ standing of C++’s type deduction rules remains essential.


Item 5: Prefer `auto` to explicit type declarations.
==================================================

## Things to Remember
- `auto` variables must be initialized, are generally immune to type mismatches that can lead to portability or efficiency problems, can ease the process of refactoring, and typically require less typing than variables with explicitly specified types.
- auto-typed variables are subject to the pitfalls described in Items 2 and 6.

Why should we prefer `auto`
1. `auto` variables have their type deduced from their initializer, so they must be initialized. So there is no uninitialized variable.
2. `auto` will be the original type which may be faster and more effective. See compare `auto` and `std::function`
3. We don't know the exact type sometimes. For example lambda.
4. `auto` avoid _type shortcuts_

## Compare `auto` and `std::function`

Using `std::function` is not the same as using `auto`.`std::function` object typically uses more memory than the auto-declared object. Invoking a closure via a std::function object is almost certain to be slower than calling it via an auto-declared object.

An auto-declared variable holding a closure has the same type as the closure, and as such it uses only as much memory as the closure requires. The type of a `std::function` declared variable holding a closure is an instantiation of the `std::function` template, and that has a fixed size for any given signature. This size may not be adequate for the closure it’s asked to store, and when that’s the case, the `std::function` constructor will allocate heap memory to store the closure.


Item 6: Use the explicitly typed initializer idiom when auto deduces undesired types.
============

Sometimes, auto doesn't work as your expection. This item shows one case.

## Things to Remember
- “Invisible” proxy types can cause auto to deduce the “wrong” type for an initializing expression.
- The explicitly typed initializer idiom forces auto to deduce the type you want it to have

`proxy class`: a class that exists for the purpose of emulating and augmenting the behavior of some other type.

Category of `proxy class`:
* Some `proxy classes` are designed to be _apparent_ to clients. That’s the case for `std::shared_ptr` and `std::unique_ptr`, for example.
* Other `proxy classes` are designed to act more or less _invisibly_. `std::vector<bool>::reference` is an example of such “invisible” proxies, as is its `std::bitset` compatriot, `std::bitset::reference`.

As a general rule, “invisible” proxy classes don’t play well with `auto`. Objects of such classes are often not designed to live longer than a single statement, so creating variables of those types tends to violate fundamental library design assumptions.

You therefore want to avoid code of this form:
    auto someVar = expression of "invisible" proxy class type;

Example:

```cpp
std::vector<bool> v;
auto b = v[i]; //b is std::vector<bool>::reference, not bool
```
## How to find out proxy classes

Paying careful attention to the interfaces you’re using can often reveal the existence of proxy classes.

## How to make `auto` work with proxy classes

**explicitly typed initializer idiom**

    auto highPriority = static_cast<bool>(features(w)[5] );

It can also be useful to emphasize that you are deliberately creating a variable of a type that is different from that generated by the initializing expression. 

    double calcEpsilon();
    auto ep = static_cast<float>(calcEpsilon());  // ep is float


Item 7: Distinguish between ()  and {} when creating objects.
=============================================================

    int x(0);
    int y = 0;
    int z{0};

    int z = {0}; //For the remainder of this Item, I’ll generally ignore 
                 //the equals-sign-plus-braces syntax, because C++ usually 
                 //treats it the same as the braces-only version 


For user defined types, it’s important to distinguish initialization from assignment, because different function calls are involved

    Widget w1; // call default constructor
    Widget w2 = w1; // not an assignment; calls copy ctor
    w1 = w2; // an assignment; calls copy operator=

Braces can also be used to specify default initialization values for non-static data members. This capability—new to C++11—is shared with the “=” initialization syntax, but not with parentheses:

    class Widget {
    private:
     int x{ 0 }; // fine, x's default value is 0
     int y = 0; // also fine
     int z(0); // error!
    };

On the other hand, uncopyable objects may be initialized using braces or parentheses, but not using “=”

A novel feature of braced initialization is that it prohibits implicit narrowing conversions among built-in types. If the value of an expression in a braced initializer isn’t guaranteed to be expressible by the type of the object being initialized, the code won’t compile. Initialization using parentheses and “=” doesn’t check for narrowing conversions.

    double x, y, z;
    int sum1{ x + y + z }; // error! sum of doubles may
                           // not be expressible as int

Another noteworthy characteristic of braced initialization is its immunity to C++’s most vexing parse.

    Widget w2(); // most vexing parse! declares a function
     // named w2 that returns a Widget!

    Widget w3{}; // calls Widget ctor with no args

In constructor calls, parentheses and braces have the same meaning as long as `std::initializer_list` parameters are not involved

If there’s any way for compilers to construe a call using a braced initializer to be to a constructor taking a `std::initializer_list`, compilers will employ that interpretation.

    class Widget {
    public:
     Widget(int i, bool b); // as before
     Widget(int i, double d); // as befor

     Widget(std::initializer_list<long double> il);  // added
    };

    Widget w1(10, true); // uses parens and, as before,
                         // calls first ctor
    Widget w2{10, true}; // uses braces, but now calls
                         // std::initializer_list ctor
                         // (10 and true convert to long double)
    Widget w3(10, 5.0);  // uses parens and, as before,
                         // calls second ctor
    Widget w4{10, 5.0};  // uses braces, but now calls
                         // std::initializer_list ctor
                         // (10 and 5.0 convert to long double)


Even what would normally be copy and move construction can be hijacked by `std::initializer_list` constructors

    class Widget {
    public:
     Widget(int i, bool b);                         // as before
     Widget(int i, double d);                       // as before
     Widget(std::initializer_list<long double> il); // as before
     operator float() const;                        // convert
                                                    // to float
    };
    Widget w5(w4); // uses parens, calls copy ctor
    Widget w6{w4}; // uses braces, calls
                   // std::initializer_list ctor
                   // (w4 converts to float, and float
                   // converts to long double)

It prevails even if the best-match `std::initializer_list` constructor can’t be called. For example:

    class Widget {
    public:
     Widget(int i, bool b);                  // as before
     Widget(int i, double d);                // as before
     Widget(std::initializer_list<bool> il); // element type is
                                             // now bool
                                             // no implicit
    };                                       // conversion funcs
    Widget w{10, 5.0};                       // error! requires narrowing conversions

Only if there’s no way to convert the types of the arguments in a braced initializer to the type in a `std::initializer_list` do compilers fall back on normal overload resolution.


Empty braces mean no arguments, not an empty `std::initializer_list`:
    class Widget {
    public:
     Widget();                              // default ctor
     Widget(std::initializer_list<int> il); // std::initializer_list ctor
                                            // no implicit
    };                                      // conversion funcs
    Widget w1;                              // calls default ctor
    Widget w2{};                            // also calls default ctor
    Widget w3();                            // most vexing parse! declares a function!

If you want to call a `std::initializer_list` constructor with an empty `std::initializer_list`, you do it by making the empty braces a constructor argument—by putting the empty braces inside the parentheses or braces demarcating what you’re passing

    Widget w4({}); // calls std::initializer_list ctor
                   // with empty list
    Widget w5{{}}; // ditto

So add `std::initializer_list` overloads only with great deliberation.

## Things to Remember
- Braced initialization is the most widely usable initialization syntax, it prevents narrowing conversions, and it’s immune to C++’s most vexing parse.
- During constructor overload resolution, braced initializers are matched to `std::initializer_list` parameters if at all possible, even if other constructors offer seemingly better matches.
- An example of where the choice between parentheses and braces can make a significant difference is creating a `std::vector<numeric type>` with two arguments.
- Choosing between parentheses and braces for object creation inside templates can be challenging



Item 8: Prefer `nullptr` to 0 and NULL.
=====================================

    void f(int); // three overloads of f
    void f(bool);
    void f(void*);
    f(0);    // calls f(int), not f(void*)
    f(NULL); // might not compile, but typically calls
             // f(int). Never calls f(void*)

The uncertainty regarding the behavior of `f(NULL)` is a reflection of the leeway granted to implementations regarding the type of `NULL`. If `NULL` is defined to be, say, 0L (i.e., 0 as a long), the call is ambiguous, because conversion from long to int, long to bool, and 0L to `void*` are considered equally good.

This counterintuitive behavior is what led to the guideline for C++98 programmers to avoid overloading on pointer and integral types. That guideline remains valid in C++11, because, the advice of this Item notwithstanding, it’s likely that some developers will continue to use 0 and `NULL`, even though `nullptr` is a better choice.


The fact that template type deduction deduces the “wrong” types for 0 and `NULL` (i.e., their true types, rather than their fallback meaning as a representation for a null pointer) is the most compelling reason to use `nullptr` instead of 0 or `NULL` when you want to refer to a null pointer. 

## Things to Remember
- Prefer `nullptr` to 0 and `NULL`.
- Avoid overloading on integral and pointer types.


Item 9: Prefer alias declarations to typedefs.
==============================================

    typedef
     std::unique_ptr<std::unordered_map<std::string, std::string>>
     UPtrMapSS;

    using UPtrMapSS =
     std::unique_ptr<std::unordered_map<std::string, std::string>>;

    // FP is a synonym for a pointer to a function taking an int and
    // a const std::string& and returning nothing
    typedef void (*FP)(int, const std::string&);  // typedef
                                                  // same meaning as above
    using FP = void (*)(int, const std::string&); // alias
                                                  // declaration

## A Compelling Reason for **alias templates**

A compelling reason does exist: templates. In particular, alias declarations may be templatized (in which case they’re called alias templates), while typedefs cannot. This gives C++11 programmers a straightforward mechanism for expressing things that in C++98 had to be hacked together with **typedefs nested inside templatized structs**

A linked list that uses a custom allocator, `MyAlloc`.

    template<typename T> // MyAllocList<T>
    using MyAllocList = std::list<T, MyAlloc<T>>; // is synonym for
     // std::list<T,
     // MyAlloc<T>>
    MyAllocList<Widget> lw; // client code

    //// Hack together with typedefs nested inside templatized structs
    template<typename T>                    // MyAllocList<T>::type
    struct MyAllocList {                    // is synonym for
     typedef std::list<T, MyAlloc<T>> type; // std::list<T,
    };                                      // MyAlloc<T>>
    MyAllocList<Widget>::type lw;           // client code

    // use the typedef inside a template for the purpose of creating
    // a linked list holding objects of a type specified by a template
    // parameter, you have to precede the typedef name with typename
    template<typename T>
    class Widget {                          // Widget<T> contains
    private:                                // a MyAllocList<T>
        typename MyAllocList<T>::type list; // as a data member
    };

    template<typename T>
    class Widget {
    private:
     MyAllocList<T> list; // no "typename",
                          // no "::type"
    };

## c++11 std library still uses `typedef`, c++14 uses **alias templates**

    std::remove_const<T>::type // C++11: const T → T
    std::remove_const_t<T> // C++14 equivalent

    template <class T>
    using remove_const_t = typename remove_const<T>::type;

## Things to Remember
- typedefs don’t support templatization, but alias declarations do.
- Alias templates avoid the “::type” suffix and, in templates, the “typename” prefix often required to refer to typedefs.
- C++14 offers alias templates for all the C++11 type traits transformations.


Item 10: Prefer scoped enums to unscoped enums.
===============================================

## The reduction in namespace pollution 

    enum Color { black, white, red }; // black, white, red are
                                      // in same scope as Color
    auto white = false;               // error! white already
                                      // declared in this scope


Their new C++11 counterparts, scoped enums, don’t leak names in this way:

    enum class Color { black, white, red }; // black, white, red
                                            // are scoped to Color
    auto white = false;                     // fine, no other
                                            // "white" in scope
    Color c = white;                        // error! no enumerator named
                                            // "white" is in this scope
    Color c = Color::white;                 // fine
    auto c = Color::white;                  // also fine (and in accord
                                            // with Item 5's advice)

## strongly typed

    enum class Color { black, white, red }; // enum is now scoped
    Color c = Color:: red;                  // as before, but
                                            // with scope qualifier
    if (c < 14.5) {                         // error! can't compare
                                            // Color and double

## enums may be forward-declared

    enum class Status: std::uint32_t; // underlying type for
                                      // Status is std::uint32_t
                                      // (from <cstdint>)
    enum Color: std::uint8_t;         // fwd decl for unscoped enum;
                                      // underlying type is
                                      // std::uint8_t
    enum class Status: std::uint32_t { good = 0,
        failed = 1,
        incomplete = 100,
        corrupt = 200,
        audited = 500,
        indeterminate = 0xFFFFFFFF
    };

## Things to Remember
- C++98-style enums are now known as unscoped enums.
- Enumerators of scoped enums are visible only within the enum. They convert to other types only with a cast.
- Both scoped and unscoped enums support specification of the underlying type. The default underlying type for scoped enums is int. Unscoped enums have no default underlying type.
- Scoped enums may always be forward-declared. Unscoped enums may be forward-declared only if their declaration specifies an underlying type


Item 11: Prefer deleted functions to private undefined ones.
============================================================

    template <class charT, class traits = char_traits<charT> >
    class basic_ios : public ios_base {
        private:
            basic_ios(const basic_ios& ); // not defined
            basic_ios& operator=(const basic_ios&); // not defined
    };
    template <class charT, class traits = char_traits<charT> >
    class basic_ios : public ios_base {
        public:
            basic_ios(const basic_ios& ) = delete;
            basic_ios& operator=(const basic_ios&) = delete;
    };

Making the new functions public will generally result in better error messages.When client code tries to use a member function, C++ checks accessibility before deleted status. When client code tries to use a deleted private function, some compilers complain only about the function being private, even though the function’s accessibility doesn’t really affect whether it can be used.

## non-member functions can be deleted
    bool isLucky(int number); // original function, is, lucky number
    bool isLucky(char) = delete; // reject chars
    bool isLucky(bool) = delete; // reject bools
    bool isLucky(double) = delete; // reject doubles and floats

‘a’ is not a number and `islucky(‘a’)` doesn’t make sense. Now don’t need to worry about `islucky(‘a’)`.
In c++98, we can declare the functions, but do not define them. But we saw error messages until link-time. 

## template specializations can be deleted

template specification must have same access level with the template

    // c++98 , cannot compile
    class Widget {
        public:
            template<typename T>
                void processPointer(T* ptr)
                { }
        private:
            template<> // error!
                void processPointer<void>(void*);
    };

    // c++11 
    class Widget {
        public:
            template<typename T>
                void processPointer(T* ptr)
                { }
    };
    template<>                                         // still
    void Widget::processPointer<void>(void*) = delete; // public,
                                                       // but
                                                       // deleted

## Things to Remember
- Prefer deleted functions to private undefined ones.
- Any function may be deleted, including non-member functions and template specializations.


Item 12: Declare overriding functions override.
===============================================


## Things to Remember
- Declare overriding functions override.
- Member function reference qualifiers make it possible to treat lvalue and rvalue objects `(*this)` differently.

Item 13: Prefer const_iterators to iterators.
=============================================

Item 14: Declare functions `noexcept` if they won’t emit exceptions.
==================================================================



In C++11, unconditional `noexcept` is for functions that guarantee they won’t emit exceptions


Whether a function is `noexcept` is as important a piece of information as whether a member function is `const.` Failure to declare a function `noexcept` when you know that it won’t emit an exception is simply poor interface specification.

But there’s an additional incentive to apply `noexcept` to functions that won’t produce exceptions: it permits compilers to generate better object code.

`std::vector::push_back` takes advantage of this “move if you can, but copy if you must” strategy, and it’s not the only function in the Standard Library that does. Other functions sporting the strong exception safety guarantee in C++98 (e.g., `std::vector::reserve`, `std::deque::insert`, etc.) behave the same way. All these functions replace calls to copy operations in C++98 with calls to move operations in C++11 only if the move operations are known to not emit exceptions. 


Interestingly, whether swaps in the Standard Library are `noexcept` is sometimes dependent on whether user defined swaps are `noexcept`.

    template <class T1, class T2>
    struct pair {
    // noexcept(bool)
     void swap(pair& p) noexcept(noexcept(swap(first, p.first)) &&
             noexcept(swap(second, p.second)));
    };

If you declare a function `noexcept` and later regret that decision, your options are bleak.

Exception-neutral functions are never `noexcept`, because they may emit such “just passing through” exceptions. Most functions, therefore, quite properly lack the `noexcept` designation.

By default, all memory deallocation functions and all destructors—both user-defined and compilergenerated—are implicitly `noexcept`. There’s thus no need to declare them `noexcept`.

Compilers typically offer no help in identifying inconsistencies between function implementations and their exception specifications.

## Things to Remember
- `noexcept` is part of a function’s interface, and that means that callers may depend on it.
- `noexcept` functions are more optimizable than non-noexcept functions.
- `noexcept` is particularly valuable for the move operations, swap, memory deallocation functions, and destructors.
- Most functions are exception-neutral rather than `noexcept`.

Item 15: Use `constexpr` whenever possible.
=========================================

Conceptually, `constexpr` indicates a value that’s not only constant, it’s known during compilation.

All `constexpr` objects are `const`, but not all `const` objects are `constexpr`

* embedded system. `constexpr` values can be put into read-only memory.
* an integral constant expression, like array size.

`constexpr` functions produce compile-time constants when they are called with compile-time constants. 

Item 16: Make const member functions thread safe.
=================================================

`std::mutex` and `std::atomic` are move-only types (i.e., a type that can be
moved, but not copied), a side effect of adding a mutex variable to a class  is that the class loses the ability to be copied. It can still be moved, however.

There’s a lesson here. For a single variable or memory location requiring synchronization, use of a std::atomic is adequate, but once you get to two or more variables or memory locations that require manipulation as a unit, you should reach for a mutex. 

## Things to Remember
- Make const member functions thread safe unless you’re certain they’ll never be used in a concurrent context.
- Use of std::atomic variables may offer better performance than a mutex, but
they’re suited for manipulation of only a single variable or memory location.


Item 17: Understand special member function generation.
=======================================================

The **special member functions** are the ones that C++ is willing to generate on its own.

C++98 has four such functions:
- the default constructor
- the destructor
- the copy constructor
- and the copy assignment operator

C++11 has two more:
- the move constructor 
- the move assignment operator. 

    class Widget {
    public:
        Widget(Widget&& rhs);            // move constructor
        Widget& operator=(Widget&& rhs); // move assignment operator
    };

These functions are generated only if they’re needed. 
They’re nonvirtual unless the function in question is a destructor in a derived class inheriting from a base class with a virtual destructor. In that case, the compiler-generated destructor for the derived class is also virtual.

The move operations are generated only if they’re needed, and if they are generated, they perform “memberwise moves” on the non-static data members of the class.

The move constructor also move-constructs its base class parts (if there are any), and the move assignment operator move-assigns its base class parts.

Types that aren’t move-enabled (i.e., that offer no special support for move operations, e.g., most C++98 legacy classes) will be “moved” via their copy operations.

- The two copy operations are independent: declaring one doesn’t prevent compilers from generating the other.
- The two move operations are not independent. If you declare either, that prevents compilers from generating the other.
- Furthermore, move operations won’t be generated for any class that explicitly declares a copy operation.
- Declaring a move operation (construction or assignment) in a class causes compilers to disable the copy operations. 

So move operations are generated for classes (when needed) only if these three things are true:
- No copy operations are declared in the class.
- No move operations are declared in the class.
- No destructor is declared in the class.

Item 20: Use `std::weak_ptr` for `std::shared_ptrlike` pointers that can dangle.
============================================================================

Item 21: Prefer `std::make_unique` and `std::make_shared` to direct use of new.
===========================================================================


## Reason :
1. less typing

    auto upw1(std::make_unique<Widget>()); // with make func
    std::unique_ptr<Widget> upw2(new Widget); // without make func
    auto spw1(std::make_shared<Widget>()); // with make func
    std::shared_ptr<Widget> spw2(new Widget); // without make func

2. The second reason to prefer make functions has to do with exception safety.

    processWidget(std::shared_ptr<Widget>(new Widget), // potential
            computePriority());                        // resource
                                                       // leak!

That is, compilers may emit code to execute the operations in this order:
    1. Perform “new Widget”.
    2. Execute computePriority. // and throw exception. Widget object will never be released.
    3. Run `std::shared_ptr` constructor


    processWidget(std::make_shared<Widget>(), // no potential
            computePriority());               // resource leak

A special feature of `std::make_shared` (compared to direct use of new) is improved efficiency. Using `std::make_shared` allows compilers to generate smaller, faster code that employs leaner data structures.

3. That’s because `std::make_shared` allocates a single chunk of memory to hold both the Widget object and the control block. 

## Cannot use make functions

1. None of the make functions permit the specification of custom deleters (see Items 18 and 19), but both `std::unique_ptr` and `std::shared_ptr` have constructors that do.
2. That means that within the make functions, the perfect forwarding code uses parentheses, not braces. If you want to construct your pointed-to object using a braced initializer, you must use new directly. Although, there is workaround: 

    // create std::initializer_list
    auto initList = { 10, 20 };
    // create std::vector using std::initializer_list ctor
    auto spv = std::make_shared<std::vector<int>>(initList);

## Things to Remember
- Compared to direct use of new, make functions eliminate source code duplication, improve exception safety, and, for `std::make_shared` and `std::allo` cate_shared, generate code that’s smaller and faster.
- Situations where use of make functions is inappropriate include the need to specify custom deleters and a desire to pass braced initializers.
- For `std::shared_ptrs`, additional situations where make functions may be ill-advised include (1) classes with custom memory management and (2) systems with memory concerns, very large objects, and `std::weak_ptrs` that outlive the corresponding `std::shared_ptrs`.

Item 22: When using the Pimpl Idiom, define special member functions in the implementation file.
===========================================================================


Pimpl (“pointer to implementation”) Idiom. That’s the technique whereby you replace the data members of a class with a pointer to an implementation class (or struct), put the data members that used to be in the primary class into the implementation class, and access those data members indirectly through the pointer. 

    class Widget { // still in header "widget.h"
        public:
            Widget();
        private:
            std::unique_ptr<Impl> pImpl; // use smart pointer
    };                                   // instead of raw point

A type that has been declared, but not defined, is known as an incomplete type. `Widget::Impl` is such a type. There are very few things you can do with an incomplete type, but declaring a pointer to it is one of them. The Pimpl Idiom takes advantage of that.

## Destructor created by compiler does not work for an incomplete type.

    #include "widget.h"
    Widget w; // error!

All compiler-generated special functions are inline. So destructor is created at line where Widget w defined. The destructor calls std::unique_ptr’s destructor which needs to see the complete type of Widget.


Solution: 

Define destructor in widget.cpp after definition of Widget::Impl. Then destructor will be created in that file and after defination of Widget::Impl. 

    // widget.cpp
    ... 
    Widget::~Widget() = default; //after definition of Widget::Impl

The declaration of a destructor in Widget prevents compilers from generating the move operations, so if you want move support, you must declare the functions yourself. 

    Widget::Widget(Widget&& rhs) = default;  // definitions
    Widget& Widget::operator=(Widget&& rhs) = default;

For same reason, move definitions to cpp file.

It’s interesting to note that if we were to use std::shared_ptr instead of std::unique_ptr for pImpl, we’d find that the advice of this Item no longer applied.

There’d be no need to declare a destructor in Widget, and without a user-declared destructor, compilers would happily generate the move operations, which would do exactly what we’d want them to.

The difference in behavior between std::unique_ptr and std::shared_ptr for pImpl pointers stems from the differing ways these smart pointers support custom deleters. For std::unique_ptr, the type of the deleter is part of the type of the smart pointer, and this makes it possible for compilers to generate smaller runtime data structures and faster runtime code. A consequence of this greater efficiency is that pointed-to types must be complete when compiler-generated special functions (e.g., destructors or move operations) are used. For std::shared_ptr, the type of the deleter is not part of the type of the smart pointer. This necessitates larger runtime data structures and somewhat slower code, but pointed-to types need not be complete when compiler-generated special functions are employed.

## Things to Remember
- The Pimpl Idiom decreases build times by reducing compilation dependencies between class clients and class implementations.
- For std::unique_ptr pImpl pointers, declare special member functions in the class header, but implement them in the implementation file. Do this even if the default function implementations are acceptable.
- The above advice applies to std::unique_ptr, but not to std::shared_ptr.

Item 23: Understand std::move and std::forward.
===============================================

- std::move unconditionally casts its argument to an rvalue.
- std::forward performs this cast only if a particular condition is fulfilled. 

## std::move

    template<typename T> // in namespace std
    typename remove_reference<T>::type&&
    move(T&& param)
    {
        using ReturnType = typename remove_reference<T>::type&&;
        return static_cast<ReturnType>(param);
    }

std::move takes a reference to an object (a universal reference. 
If the type T happens to be an lvalue reference, T&& would become an lvalue reference. To prevent this from happening, the type trait (see Item 9) std::remove_reference is applied to T, thus ensuring that “&&” is applied to a type that isn’t a reference. 


rvalue doesn’t mean it must be moved. 


    class string {                     // std::string is actually a
        public:                        // typedef for std::basic_string<char>
            string(const string& rhs); // copy ctor
            string(string&& rhs);      // move ctor
    };

    class Annotation {
        public:
            explicit Annotation(const std::string text)
                : value(std::move(text)) // "move" text into value; this code
            {}                           // doesn't do what it seems to!
        private:
            std::string value;
    };

Before the cast, text is an lvalue const std::string, and the result of the cast is an rvalue const std::string. const rvalue cannot match non-const rvalue reference.

Moving a value out of an object generally modifies the object, so the language should not permit const objects to be passed to functions (such as move constructors) that could modify them


- don’t declare objects const if you want to be able to move from them. Move requests on const objects are silently transformed into copy operations.
- std::move not only doesn’t actually move anything, it doesn’t even guarantee that the object it’s casting will be eligible to be moved. 

## std::forward

    void process(const Widget& lvalArg); // process lvalues
    void process(Widget&& rvalArg); // process rvalues
    template<typename T> // template that passes
        void logAndProcess(T&& param) // param to process
        {
            auto now = // get current time
                std::chrono::system_clock::now();
            makeLogEntry("Calling 'process'", now);
            process(std::forward<T>(param));
        }

    template<typename T> 
    T&& forward(typename 
            remove_reference<T>::type& param) 
    {
        return static_cast<T&&>(param);
    }

That’s why std::forward is a conditional cast: it casts to an rvalue only if its argument was initialized with an rvalue.

Here T applied to `std::forward` is not deduced from its argument. Information is encoded in logAndProcess’s template parameter T. So `std::forward` knows if its argument is lvalue.

## std::forward simulates std::move

std::forward<remove_reference<decltype(param)>::type>(param) 
<==>
std::move(param)

So don’t use std::forward simulates std::move


## Things to Remember
- std::move performs an unconditional cast to an rvalue. In and of itself, it doesn’t move anything.
- std::forward casts its argument to an rvalue only if that argument is bound to an rvalue.
- Neither std::move nor std::forward do anything at runtime.


Item 24: Distinguish universal references from rvalue references.
=================================================================

    void f(Widget&& param);   // rvalue reference
    Widget&& var1 = Widget(); // rvalue reference
    auto&& var2 = var1;       // not rvalue reference
    template<typename T>
    void f(std::vector<T>&& param); // rvalue reference
    template<typename T>
    void f(T&& param); // not rvalue reference

In fact, “T&&” has two different meanings. One is rvalue reference, the other meaning is universal reference.

**Universal Reference**:  bind to rvalues (like rvalue references) as well as lvalues (like lvalue references). Furthermore, they can bind to const or non-const objects, to volatile or non-volatile objects, even to objects that are both const and volatile. They can bind to virtually anything.

Universal references arise in two contexts.
1. The most common is function template parameters,
2. the second context is auto declarations. 

    template<typename T> void f(T&& param);
    auto&& var2 = var1;

What these contexts have in common is the presence of type deduction. If you see “T&&” without type deduction, you’re looking at an rvalue reference:

    void f(Widget&& param);   // no type deduction;
                              // param is an rvalue reference
    Widget&& var1 = Widget(); // no type deduction;
                              // var1 is an rvalue referenc

The initializer for a universal reference determines whether it represents an rvalue reference or an lvalue reference. 

For a reference to be universal, type deduction is necessary, but it’s not sufficient. The form of the reference declaration must also be correct, and that form is quite con‐ strained. It must be precisely “T&&”. 

    template<typename T>
    void f(std::vector<T>&& param); // param is an rvalue reference

Even the simple presence of a const qualifier is enough to disqualify a reference from being universal:

    template<class T, class Allocator = allocator<T>> 
    class vector { 
        public:
            void push_back(T&& x); // not universal reference ,
                                   // since T is already known. No type deduce here.
    };

Bear in mind that this entire Item—the foundation of universal references—is a lie...er, an “abstraction.” The underlying truth is known as reference collapsing, a topic to which Item 28 is dedicated.

## Things to Remember
- If a function template parameter has type T&& for a deduced type T, or if an object is declared using auto&&, the parameter or object is a universal reference.
- If the form of the type declaration isn’t precisely type&&, or if type deduction does not occur, type&& denotes an rvalue reference.
- Universal references correspond to rvalue references if they’re initialized with rvalues. They correspond to lvalue references if they’re initialized with lvalues.

Item 25: Use std::move on rvalue references, std::forward on universal references.
==================================================================================

rvalue references should be unconditionally cast to rvalues (via std::move) when forwarding them to other functions, because they’re always bound to rvalues, and universal references should be conditionally cast to rvalues (via std::forward) when forwarding them, because they’re only sometimes bound to rvalues.

## Common overload functions cannot simulates universal reference:

    void setName(const std::string& newName) // set from
    { name = newName; }                      // const lvalue
    void setName(std::string&& newName)      // set from
    { name = std::move(newName); }           // rvalue

overload has problems:
1. more to write and maintain
2. less efficent
3. poor scalability of design

In some cases, you’ll want to use the object bound to an rvalue reference or a universal reference more than once in a single function, and you’ll want to make sure that it’s not moved from until you’re otherwise done with it. In that case, you’ll want to apply std::move (for rvalue references) or std::forward (for universal references) to only the final use of the reference. For example:

    template<typename T>       // text is
    void setSignText(T&& text) // univ. reference
    {
        sign.setText(text); // use text, but
                            // don't modify it
        auto now =          // get current time
            std::chrono::system_clock::now();
        signHistory.add(now,
                std::forward<T>(text)); // conditionally cast
    }                                   // text to rvalu


**Things that are declared as rvalue reference can be lvalues or rvalues. The distinguishing criterion is: if it has a name, then it is an lvalue. Otherwise, it is an rvalue.**

If a function returns its parameter by value, you should move for forward it. 
If you’re in a function that returns by value, and you’re returning an object bound to an rvalue reference or a universal reference, you’ll want to apply std::move or std::forward when you return the reference. 

    Widget makeWidget() // Moving version of makeWidget
    {
        Widget w;
        return std::move(w); // move w into return value
    }                        // (don't do this!)

return value optimization (RVO):
    compilers may elide the copying (or moving) of a local object in a function that returns by value if (1) the type of the local object is the same as that returned by the function and (2) the local object is what’s being returned

In effect, the Standard requires that when the RVO is permitted, either copy elision takes place or std::move is implicitly applied to local
objects being returned. 

## Things to Remember
- Apply std::move to rvalue references and std::forward to universal references the last time each is used.
- Do the same thing for rvalue references and universal references being returned from functions that return by value.
- Never apply std::move or std::forward to local objects if they would other‐ wise be eligible for the return value optimization.

Item 26: Avoid overloading on universal references.
===================================================

Functions taking universal references are the greediest functions in C++. They instantiate to create exact matches for almost any type of argument. This is why combining overloading and universal references is almost always a bad idea: the universal reference overload vacuums up far more argument types than the developer doing the overloading generally expects.

    void logAndAdd(const std::string& name)
    {
        names.emplace(name);
    }
    // Because name is an lvalue, it is copied into names. 
    logAndAdd(petName);
    // pass rvalue std::string. name itself is an lvalue, so it’s copied into names, 
    logAndAdd(std::string("Persephone"));
    // pass string literal. emplace would have used the string literal
    // to create the std::string object directly. we’re paying to copy a std::string, 
    logAndAdd("Patty Dog");

perfect forward can fix this problem.

    template<typename T>
    void logAndAdd(T&& name)
    {
        names.emplace(std::forward<T>(name));
    }

But sometimes you want to overload logAndAdd. 

    std::string nameFromIdx(int idx); // return name
                                      // corresponding to idx
    void logAndAdd(int idx)           // new overload
    {
        names.emplace(nameFromIdx(idx));
    }


Overloading on universal reference causes problem.

    short nameIdx;
    logAndAdd(nameIdx); // error!

The one taking a universal reference can deduce T to be short, thus yielding an exact match. 

An easy way to topple into this pit is to write a perfect forwarding constructor. A small modification to the logAndAdd example demonstrates the problem. 


## overloading of constructor on universal reference is worse

    class Person {
        public:
            template<typename T>
                explicit Person(T&& n)    // perfect forwarding ctor;
            : name(std::forward<T>(n)) {} // initializes data member
            explicit Person(int idx)      // int ctor
                : name(nameFromIdx(idx)) {}
        private:
                std::string name;
    };

There are compiler-generated copy constructor for Person.

    class Person {
        public:
            template<typename T> // perfect forwarding ctor
                explicit Person(T&& n)
                : name(std::forward<T>(n)) {}
            explicit Person(int idx);  // int ctor
            Person(const Person& rhs); // copy ctor
                                       // (compiler-generated)
            Person(Person&& rhs);      // move ctor
                                       // (compiler-generated)
    };

You will get error if you call:

    Person p("Nancy");
    auto cloneOfP(p); // create new Person from p;
                      // this won't compile!

This code won’t call the copy constructor. It will call the perfect forwarding constructor.

    explicit Person(Person& n)           // instantiated from
     : name(std::forward<Person&>(n)) {} // perfect-forwarding
                                         // template

The interaction among perfect-forwarding constructors and compiler-generated copy and move operations develops even more wrinkles when inheritance enters the picture.

    class SpecialPerson: public Person {
        public:
            SpecialPerson(const SpecialPerson& rhs) // copy ctor; calls
                : Person(rhs)                       // base class
            { }                                     // forwarding ctor!
            SpecialPerson(SpecialPerson&& rhs)      // move ctor; calls
                : Person(std::move(rhs))            // base class
            { }                                     // forwarding ctor!
    }

Note that the derived class functions are using arguments of type SpecialPerson to pass to their base class, then work through the template instantiation and overload-resolution consequences for the constructors in class Person. Ultimately, the code won’t compile, because there’s no std::string constructor taking a SpecialPerson.

## Things to Remember
- Overloading on universal references almost always leads to the universal reference overload being called more frequently than expected.
- Perfect-forwarding constructors are especially problematic, because they’re typically better matches than copy constructors for non-const lvalues, and they can hijack derived class calls to base class copy and move constructors

Item 27: Familiarize yourself with alternatives to overloading on universal references.
=======================================================================================

If the universal reference is part of a parameter list containing other parameters that are not universal references, sufficiently poor matches on the non-universal reference parameters can knock an overload with a universal reference out of the running. That’s the basis behind the tag dispatch approach, 

## tag dispatch: 

    template<typename T>
    void logAndAdd(T&& name)
    {
        // if name is lvalue, T willd be int& which is not integral.
        // That’s why use std::remove_reference here
        logAndAddImpl(std::forward<T>(name),
                std::is_integral<typename std::remove_reference<T>::type>());
    }

    template<typename T>                             // non-integral
    void logAndAddImpl(T&& name, std::false_type)    // argument:
    {                                                // add it to
        auto now = std::chrono::system_clock::now(); // global data
        log(now, "logAndAdd");                       // structure
        names.emplace(std::forward<T>(name));
    }

    std::string nameFromIdx(int idx);
    void logAndAddImpl(int idx, std::true_type) // integral
    {                                           // argument: look
        logAndAdd(nameFromIdx(idx));            // up name and
    }                                           // call logAndAdd with it

In this design, the types std::true_type and std::false_type are “tags” whose only purpose is to force overload resolution to go the way we want.

## Constraining templates that take universal references

Example in Item 26. class Person’s constructors.

For situations like these, where an overloaded function taking a universal reference is greedier than you want[take Person lvalue and derived class. we expect copy constructor takes these type], yet not greedy enough to act as a single dispatch function[cannot take const lvalue of Person], tag dispatch is not the droid you’re looking for.

In our case, we’d like to enable the Person perfect-forwarding constructor only if the type being passed isn’t Person. If the type being passed is Person, we want to disable the perfect-forwarding constructor (i.e., cause compilers to ignore it), because that will cause the class’s copy or move constructor to handle the call, which is what we want when a Person object is initialized with another Person.

###  The solution:

class Person {
    public:
        template<
            typename T,
                     typename = std::enable_if_t<
                         !std::is_base_of<Person, std::decay_t<T>>::value
                         &&
                         !std::is_integral<std::remove_reference_t<T>>::value
                         >
                         >
                         explicit Person(T&& n) // ctor for std::strings and
                         : name(std::forward<T>(n)) // args convertible to
                         {  } // std::strings
        explicit Person(int idx) // ctor for integral args
            : name(nameFromIdx(idx))
        {  }
        // copy and move ctors, etc.
    private:
        std::string name;
};

1. std::decay_t C++14, remove reference and cv from type. 

## Trade-offs

As a rule, perfect forwarding is more efficient, because it avoids the creation of temporary objects solely for the purpose of conforming to the type of a parameter declaration.

But perfect forwarding has drawbacks.
1. One is that some kinds of arguments can’t be perfect-forwarded, even though they can be passed to functions taking specific types. 
2. A second issue is the comprehensibility of error messages when clients pass invalid arguments. Add static_assert to get more useful error messages.

    explicit Person(T&& n)
    : name(std::forward<T>(n))
    {
        // assert that a std::string can be created from a T object
        static_assert(
                std::is_constructible<std::string, T>::value,
                "Parameter n can't be used to construct a std::string"
                );
        // the usual ctor work goes here
    }

## Things to Remember
- Alternatives to the combination of universal references and overloading include the use of distinct function names, passing parameters by lvaluereference-to-const, passing parameters by value, and using tag dispatch.
- Constraining templates via std::enable_if permits the use of universal references and overloading together, but it controls the conditions under which compilers may use the universal reference overloads.
- Universal reference parameters often have efficiency advantages, but they typically have usability disadvantages.



Item 28: Understand reference collapsing.
=========================================

    template<typename T>
    void func(T&& param);  // universal reference deduction

The encoding mechanism is simple. When an lvalue is passed as an argument, T is deduced to be an lvalue reference. When an rvalue is passed, T is deduced to be a non-reference.

## reference collapsing.

You are forbidden from declaring references to references, but compilers may produce them in 4 particular contexts:
1. template instantiation.
2. auto variables. 
3. typedef
4. decltype

If either reference is an lvalue reference, the result is an lvalue reference. Otherwise (i.e., if both are rvalue references) the result is an rvalue reference.


* A& & becomes A&
* A& && becomes A&
* A&& & becomes A&
* A&& && becomes A&& // 助记：4个以下都是左值

A universal reference isn’t a new kind of reference, it’s actually an rvalue reference in a context where two conditions are satisfied:
- Type deduction distinguishes lvalues from rvalues. Lvalues of type T are deduced to have type T&, while rvalues of type T yield T as their deduced type.
- Reference collapsing occurs

## Things to Remember
- Reference collapsing occurs in four contexts: template instantiation, auto type generation, creation and use of typedefs and alias declarations, and decltype.
- When compilers generate a reference to a reference in a reference collapsing context, the result becomes a single reference. If either of the original references is an lvalue reference, the result is an lvalue reference. Otherwise it’s an rvalue reference.
- Universal references are rvalue references in contexts where type deduction distinguishes lvalues from rvalues and where reference collapsing occurs.



Item 29: Assume that move operations are not present, not cheap, and not used.
==============================================================================

There are thus several scenarios in which C++11’s move semantics do you no good:
- No move operations: The object to be moved from fails to offer move operations. The move request therefore becomes a copy request.
- Move not faster: The object to be moved from has move operations that are no faster than its copy operations.
- Move not usable: The context in which the moving would take place requires a move operation that emits no exceptions, but that operation isn’t declared noexcept.

## Things to Remember
- Assume that move operations are not present, not cheap, and not used.
- In code with known types or support for move semantics, there is no need for assumptions.

Item 31: Avoid default capture modes.
=====================================

## reason 1
A by-reference capture causes a closure to contain a reference to a local variable or to a parameter that’s available in the scope where the lambda is defined. If the lifetime of a closure created from that lambda exceeds the lifetime of the local variable or parameter, the reference in the closure will dangle.

Long-term, it’s simply better software engineering to explicitly list the local variables and parameters that a lambda depends on

In general, default by-value capture isn’t the antidangling elixir you might imagine. The problem is that if you capture a pointer by value, you copy the pointer into the closures arising from the lambda, but you don’t prevent code outside the lambda from deleteing the pointer and causing your copies to dangle.


Captures apply only to **non-static local variables** (including parameters) visible in the scope where the lambda is created. 

    class Widget {
        public:
            auto addFilter() const; // add an entry to filters
            int divisor; // used in Widget's filter
    };

    auto Widget::addFilter() const
    {
        // what’s being captured here is the Widget’s this pointer,
        // not divisor. this->divisor
        return [=](int value) { return value % divisor == 0; } ;
    }


    {
        Widget w;
        auto f = w.addFilter();
        return f; // f contains copy of this pointer of w.
    }             // f contains dangling pointer here


Solve this problem:

    auto Widget::addFilter() const
    {
        auto divisorCopy = this->divisor;
        return [divisorCopy](int value) { return value % divisorCopy == 0; } ;
    }

## reason 2

An additional drawback to default by-value captures is that they can suggest that the corresponding closures are self-contained and insulated from changes to data outside the closures. In general, that’s not true, because lambdas may be dependent not just on local variables and parameters (which may be captured), but also on objects with static storage duration. Such objects are defined at global or namespace scope or are declared static inside classes, functions, or files. These objects can be used inside lambdas, but they can’t be captured. Yet specification of a default by-value capture mode can lend the impression that they are. 


    auto addDivisorFilter()
    {
        static auto divisor = 1;
        auto l = [=]() { std::cout <<  divisor << std::endl;} ;
        ++divisor; // modify divisor
        return l;
    }


    auto l = addDivisorFilter();
    l();  // output 2
    auto ll = addDivisorFilter();
    ll(); // output 3


## Things to Remember
- Default by-reference capture can lead to dangling references.
- Default by-value capture is susceptible to dangling pointers (especially this), and it misleadingly suggests that lambdas are self-contained.


Item 32: Use init capture to move objects into closures.
========================================================


## init capture

If you have a move-only object (e.g., a std::unique_ptr or a std::future) that you want to get into a closure, C++11 offers no way to do it. But that’s C++11. C++14 is a different story. It offers direct support for moving objects into closures. 


Using an init capture makes it possible for you to specify
1. **the name of a data member** in the closure class generated from the lambda and
2. **an expression** initializing that data member.

Here’s how you can use init capture to move a std::unique_ptr into a closure:

    class Widget { // some useful type
        public:
            bool isValidated() const;
    };

    auto func = [pw = std::make_unique<Widget>()]
    { return pw->isValidated() && pw->isArchived(); };

`[pw = std::move(pw)]` is init capture. To the left of the “=” is the name of the data member in the closure class you’re specifying, and to the right is the initializing expression. The scope on the left of the “=” is different from the scope on the right. The scope on the left is that of the closure class. The scope on the right is the same as where the lambda is being defined. 

This should make clear that the C++14 notion of “capture” is considerably generalized from C++11, because in C++11, it’s not possible to capture the result of an expression. As a result, another name for init capture is **generalized lambda capture**.


## do same thing in C++11

1. moving the object to be captured into a function object produced by std::bind and
2. giving the lambda a reference to the “captured” object


    std::vector<double> data;

    // C++14
    auto func = [data = std::move(data)] // C++14 init capture
         { /* uses of data */ };

    // C++11
    auto func =
         std::bind(                          // C++11 emulation
         [](const std::vector<double>& data) // of init capture
         { /* uses of data */ },
         std::move(data)
         );

`bind` return an object which contains a lambda and a member variable. `data` has been moved into that member variable. When calling binded object, lambda receives lvalue reference to that member variable.


By default, the operator() member function inside the closure class generated from a lambda is const. That has the effect of rendering all data members in the closure const within the body of the lambda. The move-constructed copy of data inside the bind object is not const, however, so to prevent that copy of data from being modified inside the lambda, the lambda’s parameter is declared reference-to-const. 


- It’s not possible to move-construct an object into a C++11 closure, but it is possible to move-construct an object into a C++11 bind object.
- Emulating move-capture in C++11 consists of move-constructing an object into a bind object, then passing the move-constructed object to the lambda by reference.
- Because the lifetime of the bind object is the same as that of the closure, it’s possible to treat objects in the bind object as if they were in the closure.


## Things to Remember
- Use C++14’s init capture to move objects into closures.
- In C++11, emulate init capture via hand-written classes or std::bind

Item 33: Use decltype on auto&& parameters to std::forward them.
================================================================

One of the most exciting features of C++14 is generic lambdas—lambdas that use auto in their parameter specifications. The implementation of this feature is straightforward: operator()  in the lambda’s closure class is a template.

    auto f = [](auto x){ return func(normalize(x)); };

the closure class’s function call operator looks like this:

    class SomeCompilerGeneratedClassName {
        public:
            template<typename T>
                auto operator()(T x) const 
                { return func(normalize(x)); }
    }; // functionality

We need to forward `x` to `normalize`.

    auto f = [](auto&& param)
    {
        return func(normalize(std::forward<decltype(param)>(param)));
    };

`decltype` is perfect way to declare the type: 

    template<typename T> // in namespace
    T&& forward(remove_reference_t<T>& param) // std
    {
         return static_cast<T&&>(param);
    }

If `param` is lvalue, decltype(param)` is `T&`, `forward` return `T& &&`.
If `param` is rvalue, decltype(param)` is `T`, `forward` return `T&&`.
If `param` is rvalue reference,  decltype(param)` is `T&&`, `forward` return `T&& &&`.

## Things to Remember
- Use decltype on auto&& parameters to std::forward them.

Item 34: Prefer lambdas to std::bind.
=====================================

The most important reason to prefer lambdas over std::bind is that lambdas are more readable. 

    using namespace std::chrono;
    using namespace std::literals;  // for C++14 suffixes
    void setAlarm(Time t, Sound s, Duration d);

    // lambda
    auto setSoundL =
     [](Sound s)
     {
     setAlarm(steady_clock::now() + 1h, // C++14,
     s, 
     30s); 
     };

    // bind
    using namespace std::placeholders;
    auto setSoundB =
     std::bind(setAlarm,
     std::bind(std::plus<>(), steady_clock::now(), 1h),
     _1,
     30s);


    auto betweenL =
     [lowVal, highVal]
     (const auto& val) // C++14
     { return lowVal <= val && val <= highVal; };

    ////////////////////////////////////////

    using namespace std::placeholders; // as above
    auto betweenB =
     std::bind(std::logical_and<>(), // C++14
     std::bind(std::less_equal<>(), lowVal, _1),
     std::bind(std::less_equal<>(), _1, highVal));

It’s thus possible that using lambdas generates faster code than using std::bind. Lambda will be treated as inline function, bind not.

You can know if variables are captured by lambda by value or by reference.

     [lowVal, highVal] // by value

You cannot know from std::bind signature. You have to remember it.

## In C++14, there are no reasonable use cases for std::bind. C++11 needs bind when
- Move capture. C++11 lambdas don’t offer move capture, but it can be emulated through a combination of a lambda and std::bind. For details, consult Item 32, which also explains that in C++14, lambdas’ support for init capture eliminates the need for the emulation.
- Polymorphic function objects. Because the function call operator on a bind object uses perfect forwarding, it can accept arguments of any type (modulo the restrictions on perfect forwarding described in Item 30). This can be useful when you want to bind an object with a templatized function call operator.



## Things to Remember
- Lambdas are more readable, more expressive, and may be more efficient than using std::bind.
- In C++11 only, std::bind may be useful for implementing move capture or for binding objects with templatized function call operators.
