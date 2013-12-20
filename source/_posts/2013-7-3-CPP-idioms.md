layout: post
title: CPP Idioms
date: 2013-7-3
categories: C&CPP
---

### 1. The Rule of Three

The Rule of Three is a rule of thumb for C++, basically saying, if your class needs either:

- a copy constructor,
- an assignment operator,
- or a destructor,

then it is likely to need all three of them.

The default semantics for these three member functions are:

- **Destructor**: Call the destructors of all the object's class-type members 
- **Copy constructor**: Construct all the object's members from the corresponding members of the copy constructor's argument, calling the copy constructors of the object's class-type members, and doing a plain assignment of all non-class type (e.g., int or pointer) data members
- **Copy assignment operator**: Assign all the object's members from the corresponding members of the assignment operator's argument, calling the copy assignment operators of the object's class-type members, and doing a plain assignment of all non-class type (e.g., int or pointer) data members.


Reference:

- [http://en.wikipedia.org/wiki/Rule_of_three_%28C%2B%2B_programming%29](http://en.wikipedia.org/wiki/Rule_of_three_%28C%2B%2B_programming%29)
- [http://stackoverflow.com/questions/4172722/what-is-the-rule-of-three?rq=1](http://stackoverflow.com/questions/4172722/what-is-the-rule-of-three?rq=1)


### 2. Copy-and-Swap

The **copy-and-swap** idiom is the solution, and elegantly assists the assignment operator in achieving two things: 

1. avoiding code duplication with copy constructor;
2. providing a strong exception guarantee;

```.cpp
// traditional
T& operator=(const T& rhs)
{
	T temp(rhs);       // create temporary local data of the data, may throw exception
	swap(*this, temp); // swap the old data with the local new data
	return *this;   
	                   // temporay local data destructs with old data destruct
}
```

Copy-and-swap idiom need three things: 

- a working copy-constructor
- a working destructor
- a non-throwing swap function.(should not use `std::swap()`), more about this see Effective CPP Q25

So as to The Rule of Three, the assignment operator can be write in form automatically by adding a swap function.


If you're going to make a copy of something in a function, let the compiler do it in the parameter list.

```.cpp
// better
T& operator(T rhs)     // don't need enter the function if construction of the copy fails
{
	swap(*this, rhs);
	return *this;
}
```


Reference:

- http://stackoverflow.com/questions/3279543/what-is-the-copy-and-swap-idiom

### 3. Pointer to implementation (Pimpl)

The pimpl idiom is a modern C++ technique to hide implementation, to minimize coupling(耦合), and to separate interfaces.
This technique is described in Design Patterns as the **Bridge pattern**.
It is sometimes referred to as:

- "handle classes", 
- the "Pimpl idiom" (for "pointer to implementation idiom"),
- "Compiler firewall idiom"
- or "Cheshire Cat", 

especially among the C++ community.

Here's how the pimpl idiom can improve the software development lifecycle:

- Minimization of compilation dependencies.
- Separation of interface and implementation.
- Portability.

```.cpp
// .h
class Handler {
public:
	Handler();
	void foo1();
	int foo2();

	Handler(const Handler& other);
	Handle& Handle::operator=(const Handle &other);
	
private:
	class HandlerImpl;          // 私有，对用户隐藏此类
	std::tr1::unique_ptr<HandlerImpl> pimpl;   // 用于深拷贝，shared_prt用于浅拷贝
};
```

```.cpp
// .c
class Handler::HandlerImpl {
public:
	...
	HandlerImpl() {...}         // 修改函数实现，不需要重新编译用户代码
	HandlerImpl(const HandlerImpl&) {...}
	void foo1() {...}
	int foo2() {...}
	...
};

Handler::Handler() 
	: pimpl(new HandlerImpl)
{
	...
}

Handler::Handler(const Handler& other)
	: pimple(new HandlerImpl(*(other.pimpl)))   // 深拷贝
{
	...
}

Handler& Handler::operator=(Handler other)
{
	swap(*this, other);
	return *this;
}

void Handler::foo1() {
	pimpl->foo1();
}

int Handler::foo2() {
	return pimpl->foo2();
}
```


Reference:

- http://en.wikipedia.org/wiki/Opaque_pointer
- http://c2.com/cgi/wiki?PimplIdiom‎
- http://msdn.microsoft.com/en-us/library/vstudio/hh438477.aspx


### Copy on Write (COW)

On `std::string`

### RAII

### NRV(Named Return Value)/RVO(Return Value Optimization)


### One Definition Rule (ODR)

- http://en.wikipedia.org/wiki/One_Definition_Rule


### Compiler and Name

C/CPP**编译期间**不一定需要查看函数的**声明**。对于未声明的函数，编译器可以采用“隐式函数声明(implicit declaration of function)”：编译器认为未声明的函数都返回int，并且能接受任意个数的int型参数。如果声明了函数，则需要进行参数检查。而**链接期间**一定会做参数检查。

CPP从C那里继承了单遍编译的特点，编译器只能根据目前看到的代码做出决策，后面读到的代码也不会影响前面做出的决定。这也影响了**名字查找(name lookup)**和**函数重载决议**。对于一个名字/符号，CPP只能通过解析之前源码来了解名字的含义，而不是像Java那样读取类的元数据获得信息。对于函数重载决议，当CPP编译器读到一个函数调用语句时，它必需（也只能）从目前已看到的同名函数中选出最佳函数，即使后面还有更适合的函数也不影响当前决定（对于class成员函数来说，全体同名函数同会参与重载决议）。

对于函数来说，它的前向声明(forward declaration)就是函数的接口/原型声明。
对于class来说，它的前向声明就是class名的声明`class foo;`。
有时候class也需看到完整定义，比如需要访问class成员时或需要知道class大小时。


### data abstract vs object-oriented

Data Abstract: Abstract Data Type(ADT), 值语义。拷贝时是内存拷贝。很像Object-based.
Object-oriented: 对象/引用语义。拷贝是引用拷贝。三大特征：封装，继承，多态。
Object-based: 对象/引用语义。特征：封装，没有继承和多态。即只有具体类，没有抽象接口。
