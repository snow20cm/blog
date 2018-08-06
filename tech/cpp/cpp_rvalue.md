+++
Categories = ["Notes"]
Description = "Learn what C++ rvalue is"
Tags = ["cpp"]
date = "2015-03-09T10:41:29-06:00"
title = "C++ Rvalue"

+++

## [source](http://thbecker.net/articles/rvalue_references/section_01.html)

```
template <typename T>
typename remove_reference<T>::type&& move(T&& arg)
{
    return static_cast<typename remove_reference<T>::type&&>(arg);
}
```

* `std::move` turns its argument into an rvalue without doing anything else.
* `a = std::move(b)`  a只有copy constructor 或者 只有 move constructor , 都可以
* `move` 版的`swap（）` 对象只有copy constructor 或者 只有 move constructor , 都可以

有了`std::move`， 

* 大量标准算法将获益。例如`inplace sort`
* STL container 将只要求对象`movable`


How to implement a move assignment operator :


``` 
X& X::operator=(X&& rhs)
{
    // Perform a cleanup that takes care of at least those parts of the
    // destructor that have side effects. Be sure to leave the object
    // in a destructible and assignable state.
    // Move semantics: exchange content between this and rhs

    return *this;
}
```


one might expect that anything that is declared as an rvalue reference is itself an rvalue. But the designers of rvalue references have chosen a solution that is a bit more subtle than that:

**Things that are declared as rvalue reference can be lvalues or rvalues. The distinguishing criterion is: if it has a name, then it is an lvalue. Otherwise, it is an rvalue.**

`std::move` "turns its argument into an rvalue even if it isn't," and it achieves that by "hiding the name."

example:

```
// 声明成右值，但是是左值
void foo(X&& x)
{
  X anotherX = x; // calls X(X const & rhs)
}

// 声明成右值，就是右值
X&& goo();
X x = goo(); // calls X(X&& rhs) because the thing on
             // the right hand side has no name

//另一个例子
int main(void)
{
    A a(1), b(2);

    a = b;  //copy assignment operator
    a = std::move(b); //move assignment operator

    A&& c = A(3);
    a = c; //copy assignment operator
    a = std::move(c); //move assignment operator

    std::cout << "exit" << std::endl;
    return 0;
}

// 派生类的move constructor. 陷阱
Derived(Derived&& rhs) 
  : Base(rhs) // wrong: rhs is an lvalue 
  : Base(std::move(rhs)) // correct
{}
```

Any modern compiler will apply return value optimization to the original function definition

## perfect forwarding

**First**, C++11, by contrast, introduces the **following reference collapsing rules**:

* A& & becomes A&
* A& && becomes A&
* A&& & becomes A&
* A&& && becomes A&& // 助记：4个以下都是左值

**Second**, there is a special template argument deduction rule for function templates that take an argument by rvalue reference to a template argument:

    template<typename T>
    void foo(T&&);

Here, the following apply:

1. When foo is called on an lvalue of type A, then T resolves to A& and hence, by the reference collapsing rules above, the argument type effectively becomes A&.
2. When foo is called on an rvalue of type A, then T resolves to A, and hence the argument type becomes A&&.

Here's what the solution looks like:

``` 
template<typename T, typename Arg> 
shared_ptr<T> factory(Arg&& arg)
{ 
  return shared_ptr<T>(new T(std::forward<Arg>(arg)));
} 
```

where `std::forward` is defined as follows:

```
template<class S>
S&& forward(typename remove_reference<S>::type& a) noexcept
{
  return static_cast<S&&>(a);
} 
```






