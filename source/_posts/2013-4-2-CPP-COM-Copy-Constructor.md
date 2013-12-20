layout: post
title: CPP Object Model Copy Constructor
date: 2013-4-2
categories: C&CPP
---

和default constructor一样，copy constructor也是在必要的时候才由编译器产生出来。

### bitwise copy semantics

当用户不显示提供copy constructor的时候，
对于copy constructor和copy assignment operator操作，
编程器最直接的方法是施行default memberwise initialization:

- 对于基本类型的data member，内存拷贝
- 对于member class object，递归实行memberwise initialization

这也称为bitwise copy semantics(位逐次拷贝)。
这种机制并不需要编译器隐式合成copy constructor。

### 合成copy constructor

在以下情况下class不实行bitwise copy semantics，即需要编译器隐式合成copy constructor:

- class内含一个member object, 而后者的class声明有一个copy constructor(无论是用户显示声明或编译器隐式合成)
- class继承一个base class, 而后者存在一个copy constructor(无论是用户显示声明或编译器隐式合成)
- class声明了一个或多个virtual functions
- class继承链中有一个或多个virtual base classes

对于隐式合成的copy constructor，其操作是：

- 对于基本类型的data member，内存拷贝
- 对于没有声明copy constructor的member class object/base class，递归实行bitwise copy semantics
- 对于声明了copy constructor的member class object/base class，调用其copy constructor
- 如果class声明了一个或多个virtual functions，调整vptr（不能直接内存拷贝，因为被拷贝的const class reference可能是一个派生类对象，vptr指向不同的vtbl）
- 如果class继承链中有一个或多个virtual base classes，调整virtual base class共享机制

除了virtual机制, 需要编译器隐式合成copy constructor的根源是member class object及其递归member class object有一个显示声明的copy constructor。

### NRV(Named Return Value)/RVO(Return Value Optimization)

NRV的触发？需要用户定义copy constructor?
如果是，那么用户定义一般情况下用户定义copy constructor就有必要。
