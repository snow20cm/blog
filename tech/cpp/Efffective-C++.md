+++
categories = ["notes"]
date = "2015-07-20T20:32:33-06:00"
tags = ["cpp"]
title = "Efffective C++"

+++

05 - 自动构建的类函数
====================

    class Empty{};

等价于

    class Empty{
        public:
            Empty() { ... }                                   // default constructor
            Empty(const Empty& rhs) { ... }                   // copy constructor
            ~Empty() { ... }                                  // destructor — see below
            virtual Empty& operator=(const Empty& rhs) { ... } // copy assignment operator
    };

## Things to Remember

- Compilers may implicitly generate a class's default constructor, copy constructor, copy assignment operator, and destructor.


06 - Explicitly disallow the use of compiler-generated functions you do not want
====================

## Things to Remember

- To disallow functionality automatically provided by compilers, declare the corresponding member functions private and give no implementations. Using a base class like Uncopyable is one way to do this.

07 -  Declare destructors virtual in polymorphic base classes
====================

14 - 在资源管理类中小心copy行为
====================

当一个RAII对象被复制，会发生什么？

1. 禁止复制
2. 对底层资源引用计数
3. 复制底部资源
4. 移交控制权

15 - 在资源管理类中提供对原始资源的访问
====================

RAII 类对象提取资源对象有两种方式：

1. get()
2. -> *

16 - 成对使用new和delete时要采用相同形式
====================

17 - 以独立语句将newed对象置入智能指针
====================

函数参数的执行顺序是不确定的，要防止new和置入之间有异常抛出。

    f(shared_ptr<int>(new int(10)), g()); // new => g() => shared_ptr

g() 抛出异常，资源泄漏

    // 安装方案1
    shared_ptr<int> p(new int(10));
    f(p, g());
    // 安装方案2
    f(make_share(new int(10)), g());

18 - 让接口容易被正确使用，不易被误用
====================

23 - 宁以non-member，non-friend替换member函数
====================

类的封装性，可以简单的用访问它的public函数数量测量。 越少public函数当然越好。

如果一个函数可以用一系列public函数实现，那就没有必要让它成为一个public函数。 定义成non-friend, non-member 函数，通过public接口来实现功能。


24 - 若所有参数都需要类型转换，请为此采用non-member函数
====================

举个例子：

    class Rational；

有理数类定义乘法 operator *

member 函数 无法实现下面的计算

    Rational r；

    auto a = 2 * r


25 - 考虑写一个不抛出异常的swap函数
====================

how to support swap ?

1. if the default implementation ofswap offers acceptable efficiency for your class
or class template, you don't need to do anything. Anybody trying to swap objects of
your type will get the default version, and that will work fine.

2. if the default implementation of swap isn't efficient enough (which almost
always means that your class or template is using some variation of the pimpl idiom),
do the following:

    1. Offer a public swap member function that efficiently swaps the value of two
objects of your type. For reasons I'll explain in a moment, this function should
never throw an exception.

    2. Offer a non- member swap in the same namespace as your class or template.
Have it call your swap member function.

    3. If you're writing a class (not a class template), specialize std::swap for your
class. Have it also call your swap member function.

3. if you're calling swap, be sure to include a using declaration to make
std::swap visible in your function, then call swap without any namespace
qualification.


26 - 尽量延后变量定义的出现时间
====================

1. 变量提前定义，可能没有机会被使用。比如只在支路使用，在使用前退出函数。
2. 尽量在能获取变量初始化参数的地方定义变量。直接用参数初始化变量，而不是定一个空变量，然后再赋值。
3. 注意变量在循环中的使用。综合考虑，然后决定，变量定义在循环前还是循环中。


27 - 尽量少做转型动作
====================

- const_cast : const_cast is typically used to cast away the constness of objects. It is the only C++-style cast that can do this.
- dynamic_cast: dynamic_cast is primarily used to perform "safe downcasting," i.e., to determine whether an object is of a particular type in an inheritance hierarchy. It is the only cast that cannot be performed using the old-style syntax. It is also the only cast that may have a significant runtime cost. (I'll provide details on this a bit later.)
- reinterpret_cast: reinterpret_cast is intended for low-level casts that yield implementation-dependent (i.e., unportable) results, e.g., casting a pointer to an int. Such casts should be rare outside low- level code. I use it only once in this book, and that's only when discussing how you might write a debugging
allocator for raw memory (see Item 50(See 16.2)).
- static_cast can be used to force implicit conversions (e.g., non-const object to const object (as in Item 3(See 9.3)), int to double, etc.). It can also be used to perform the reverse of many such conversions  (e.g., `void*` pointers to typed pointers, pointer-to-base to pointer-to-derived), though it cannot cast from const to non-const objects. (Only const_cast can do that.)

cast 会有运行时消耗。 static_cast int to float, 一定要产生代码。

基类指针和继承类指针指向同一个对象，但是它们的值可能是不同的。这在c, c#, java中是不存在的特性。

28 - 避免返回指向内部成分的handles
====================

返回一个对private变量的non-const引用，相当于把private变量改成了public变量

而返回一个对private变量的const引用， 可能面对dangling reference的错误。


29 - 为“异常安全”而努力是值得的
====================

异常安全要保证2点：

1. leak no resources
2. Don't allow data structures to become corrupted.

1. 保证资源不泄漏 ： RAII
2. 保证数据结构不破坏 ：　让函数达到一定的异常安全等级。

异常安全分成3个级别：

1. the basic guarantee：  失败后，对象处于合法状态，但不能确定是什么合法状态。 
2. the strong guarantee： 失败后， 对象数据完全恢复原状
3. the nothrow guarantee： 不抛出异常

综合考虑效率，开销，选择一个级别。但至少保证basic guarantee。

如果函数g调用函数f，f不是异常安全的，那么g肯定也不是异常安全的。

swap和pimpl是常用的技术手段。

30 - 透彻了解inlining的里里外外
====================

复杂inline函数会增大代码体积，减低cache命中率。

template和inline都在头文件里，但没有内在联系

编译器有时会为inline函数生成函数体。比如要用函数指针调用它。 这时，普通调用 inline， 指针调用，用函数体。


构造函数和析构函数不是inline的好候选人。它们都很复杂，会负责一堆事情，生成member variables，生成基类部分等等。

inline函数无法随库升级。

调试器不能调试inline函数

31 - 将文件间的编译依赖关系降至最低
====================

1. handle classes =>pimpl idiom (pointer to implementation)
2. interface classes

用“声明依赖”替换“定义依赖”，这是编译依赖最小化的本质。

1. 如果使用object references 或 object pointers 可以完成的任务，不使用objects
2. 尽量用class声明，而不是定义. 函数声明里用到的类，也只需要类的声明。

    class Date;
    Date today();    // OK
    void f(Date d);  // OK

3. 声明和定义，分放在不同头文件

如 <iosfwd> 和 <sstream>,<streambuf>,<fstream>


32 - 确定你的public继承塑模出is-a关系
====================


34 - 区分接口继承和实现继承
====================

public 继承由两部分组成： function interfaces 和 function implementations

- 声明一个pure virtual function 目的是继承 function interface
- non-pure virtual function 目的是继承 function interface和function implementation

有时，大部分继承类的函数f使用相同的实现f1，少部分使用其他实现fn。如果将f1作为基类virtual 函数f的实现，有如下危险：

本应实现fn的继承类，忘记实现fn，而继承了基类的f1。会产生非法结果。 但这一问题，不会在编译时发现。

解决方法是：　基类的f声明为pure virtual， 但基类同时提供一个默认实现 f1. 继承类必须定义自己的f，但可以在f中调用基类的默认实现f1.

基类中定义f1的方法 2 种： 使用新名字， 如defaultF； 使用f， 但继承类调用它要指定类名 Base：：f（）

- non-virtual 函数 目的是继承接口和强制的实现。 继承类不能修改或掩盖它。


35 - 考虑virtual函数以外的其他选择
====================

1. strategy 方案
2. NVI non-virtual interface

virtual函数声明为私有函数，然后在一个public的函数中调用这个private 的virtual函数。 这个template pattern的一个实现样例。

这里一个有意思的地方是： virtual 函数在base class里是private，但是可以在继承类中，override它。 

但其实，没看出方案2，怎么是virtual的替代方案了。还是调用了virtual函数啊？。


36 - 绝不重新定义继承而来的non-virtual 函数
====================

37 - 绝不重新定义继承而来的缺省参数值
====================

virtual 函数是动态绑定， 省缺参数是静态绑定

所以， 如果基类和继承类对virtual函数 f给与不同缺省参数。 以base class引用调用virtual函数f，编译时，默认参数就绑死到了基类f的缺省参数。

    B::f(int i = 10);
    D::f(int i = 20);

    B* bp; D d;
    bp = &d;
    bp->f();  // i == 10

如果想基类和继承类对virtual函数 f给与相同缺省参数， 怎么定义呢？

    B::f(int i = 10);
    D::f(int i = 10);   // 这样定义，一旦B::f更改缺省参数，D::f也要改，如果忘了，就出问题。

解决方案： NVI

non-virtual函数定义缺省参数，而virtual函数不必有缺省值。


38 - 通过复合塑模出has-a或“is-implemented-in-terms-of”
====================

要注意区分is-a和is-implemented-in-terms-of之间的区别。

前者可以public继承，后者用复合来实现。

例如，用list实现自己版本的set，这是is-implemented-in-terms-of，用复合。

如果public继承list，语义是奇怪的，应为一个list不是一个set，它有很多set没有的功能


39 - 明智而审慎地使用private继承
====================

private继承，就是一种实现is-implemented-in-terms-of的技术手段。 基类B和继承类D之间，没有语义联系。

private继承和复合都是实现is-implemented-in-terms-of的手段。 尽量用复合。 用private继承，是因为涉及protected函数或virtual函数的访问。


使用private继承的另一个情况：EBO empty base optimization

复合一个空类，体积会至少加1；单一private继承一个空基类，体积不变



40 - 明智而审慎地使用多重继承
====================



41 - 了解隐式接口和编译期多态
====================

隐式接口，就是template中所有的操作形成的对T的约束。

    template<typename T>
    void f(T val) {
        val.f();
        val.ff();
    }

f和ff是T的隐式接口

## Things to Remember

- Both classes and templates support interfaces and polymorphism.
- For classes, interfaces are explicit and centered on function signatures. Polymorphism occurs at runtime through virtual functions.
- For template parameters, interfaces are implicit and based on valid expressions. Polymorphism occurs during compilation through template instantiation and function overloading resolution.


42 - 了解typename的双重意义
====================

- non-dependent name: template内的名字，不依赖template参数。
- dependent name : template 内的名字，依赖template参数。
- nested dependent name: 嵌套的 dependent name
- nested dependent type name: nested dependent name 是类型


    template<typename IterT>
    void workWithIterator(IterT iter)
    {
        typename std::iterator_traits<IterT>::value_type temp(*iter);
    }

nested dependent name 的解析存在困难，无法断定它是什么，可能是类型，或者static成员。 C++默认它不是类型。所以使用时，要用typename明确指定。

The exception to the "typename must precede nested dependent type names" rule is that typename must not precede nested dependent type names in a list of base classes or as a base class identifier in a member initializatio n list.

    template<typename T>
    class Derived: public Base<T>::Nested { // base class list: typename not
        public:                             // allowed
            explicit Derived(int x)
                : Base<T>::Nested(x)            // base class identifier in mem
            {                                   // init. list: typename not allowed
                typename Base<T>::Nested temp;  // use of nested dependent type
                                                // name not in a base class list or
            }                                   // as a base class identifier in a
                                                // mem. init. list: typename required
    };

43 - 学习处理模板化基类内的名称
====================

    class CompanyA {
        public:
            void sendCleartext(const std::string& msg);
            void sendEncrypted(const std::string& msg);
    };
    class CompanyB {
        public:
            void sendCleartext(const std::string& msg);
            void sendEncrypted(const std::string& msg);
    };

    template<typename Company>
    class MsgSender {
        public:
            void sendClear(const MsgInfo& info) {
                std::string msg;
                Company c;
                c.sendCleartext(msg);
            }
    }

    template<typename Company>
    class LoggingMsgSender: public MsgSender<Company> {
        public:
            void sendClearMsg(const MsgInfo& info)
            {
                sendClear(info); // call base class function;
                                 // this code will not compile!
            }
    };

上面的代码LoggingMsgSender无法编译，因为编译器假设可能存在total template specialization。例如下面的total template specialization

class CompanyZ { // this class offers no
    public:      // sendCleartext function
        void sendEncrypted(const std::string& msg);
};

template<>                  // a total specialization of
class MsgSender<CompanyZ> { // MsgSender; the same as the
    public:                 // general template, except
                            // sendCleartext is omitted
        void sendSecret(const MsgInfo& info) { }
};

`Msgsender<CompanyZ>`没有`sendclear`函数，所以`loggingmsgsender`如果继承自它，调用sendClear就是非法。

it recognizes that base class templates may be specialized and that such specializations may not offer the same interface as the general template. As a result, it generally refuses to look in templatized base classes for inherited names.

In some sense, when we cross from Object-oriented C++ to Template C++ , inheritance stops working.

解决这个问题，要强制编译器去基类里寻找函数。有3种方法

1. `this->`
2. `using`
3. 加base class qualification

第三种方法最不推荐，因为和virtual函数冲突。

## Things to Remember

- In derived class templates, refer to names in base class templates via a "this->" prefix, via using declarations, or via an explicit base class qualification.


49 - 了解new-handler的行为
====================

Operator new 抛出异常前，调用new-handler，即用户制定的错误处理函数。

    namespace std {
        typedef void (*new_handler)();
        new_handler set_new_handler(new_handler p) throw();
    }

new 分配内存失败，会反复调用new_handler,直到成功。

In new-handler:

1. 让更多内容可用。例如提前分配一块内存，这时释放。
2. 安装另一个new_handler. 当前的new_handler 不能获取足够内存，安装可以获取足够内存的new_handler.
3. 卸载new_handler. set NULL. new会抛出异常。
4. 抛出异常
5. 退出程序

创建类对象的时候，可以通过继承下面的模版，在创建对象的时候，指定new_handler.

    template<typename T>     // "mixin-style" base class for
    class NewHandlerSupport{ // class -specific
        set_new_handler
        public: // support
            static std::new_handler set_new_handler(std::new_handler p)
                throw();
            static void * operator new(std::size_t size)
                throw(std::bad_alloc);
        private:
                static std::new_handler currentHandler;
    };

    class Widget: public NewHandlerSupport<Widget>;

Widget inheriting from a templatized base class that takes Widget as a type parameter, it's called the curiously recurring template pattern (CRTP). 

`Newhandlersupport` 没有直接使用`T`. 这里使用模版，为的是不同模版实例，有不同static member currenthandler.

## Things to Remember

- set_new_handler allows you to specify a function to be called when memory allocation requests cannot be satisfied.
- Nothrow new is of limited utility, because it applies only to memory allocation; subsequent constructor calls may still throw exceptions.

50 - 了解new和delete的合理替换时机
====================

- 为了检测运行错误
- 收集内存分配统计信息
- 增加分配回收速度
- 降低额外开销
- 达到最佳对齐
- 簇集
- 自定义行为

51 - 自定义new和delete，要遵守的规则
====================

`new(0)` 也要返回合法地址。通常返回1字节的空间。
`sizeof(class)`不能为0

52 - 写了placement new也要写placement delete
====================

## Things to Remember

- When you write a placement version ofoperator new, be sure to write the corresponding placement version of operator delete. If you don't, your program may experience subtle, intermittent memory leaks.
- When you declare placement versions ofnew and delete, be sure not to unintentionally hide the normal versions of those functions.


53 - 不要忽视编译器警告
====================

Things to Remember

- Take compiler warnings seriously, and strive to compile warning- free at the maximum warning level supported by your compilers.
- Don't become dependent on compiler warnings, because different compilers warn about different things. Porting to a new compiler may eliminate warning messages you've come to rely on.
