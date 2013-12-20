layout: post
title: CPP Object Model Member Initialization List
date: 2013-4-3
categories: C&CPP
---

必需使用member initialization list的情况：

- 初使化reference member
- 初使化const member
- base class的constructor有参数
- member class的constructor有参数

在进入构造函数之前，member variable内存空间已经申请好。
在进入构造函数的user code之前，member class object就已经初使化(构造)完成。
这一步通过member initialization list(constructor有参数的object)和编译器(constructor无参的object)共同完成，顺序按照member class object的声明顺序。

这个阶段调用member function是合法的，但是不推荐。
这个阶段virtual function机制还没完成，vptr未初使化完成。
