layout: post
title: CPP Object Model Constructor
date: 2013-3-26
categories: C&CPP
---

如果没有定义任何constructor, 编译器将生成一个implicit default constructor.

但是这个implicit default constructor只会去做编译器需要做的事,不会去做多余的事情(trivial).
比如如果class的member variables都是基本类型, 那么这个default constructor将什么都不干,
**不会去初始化基本类型**.

implicit default constructor只会在以下几种情况下会隐式调用(nontrivial):

##### 1. class有 "带有default constructor的member class object"

编译器为class合成inline default constructor(每个文件module可见)
或explicit non-inline static default constructor(每个文件module可见)。
这个合成在真正被调用时发生.

如果class A有一个或以上的member class objects, 
那么class A的每一个constructor必需调用每一个member classes的default constructor.
编译器会**扩张**已存在的constructors, 在其中安插一些代码, 
使得user code被执行**之前**, 先调用必要的default constructor.

CPP要求member object在class中以**声明顺序**调用各个constructor,
和初始化列表的顺序无关.

**总结:任何类型constructor,在进入user code之前必需调用member class object
的constructor进行初始化,或在初始化列表里显式调用(可以调用任何constructor), 或由编译器隐式调用(此时只能调用default constructor).调用的顺序是member class object的声明顺序.
基本类型的member variable无需初始化.**

##### 2. class有 "带有default constructor的base class"

**任何类型constructor,在进入user code之前必需调用base class的constructor进行初始化,
或在初始化列表里显式调用(可以调用任何constructor), 或由编译器隐式调用(此时只能调用default constructor).调用的顺序是base class的声明顺序.**

base class的初始化在member class object**之前**.

##### 3. class有 virtual function

class声明或继承一个virtual function后, 
需要编译器在constructor中调整object的vptr指向class的vtbl.

因此, **对于含有virtual function的class的任何类型constructor,
必需对vptr进行初始化, 这是由编译器隐式完成.
**

virtual function在constructors中被静态决议，virtual机制不起作用。
vptr是在base constructor之后，在member init list之前进行初使化的。

##### 4. class继承了 virtual base class

如果class派生自一个继承串链, 其中有一个或多个的virtual base class,
则需要编译器为virtual base class的共享做些操作。

因此，**对于含有virtual base class的class的任何类型constructor,
必需对virtual base class的共享进行初始化，这是由编译器隐式完成。**



上面所述的4条编译器的职责, 如果用户定义了constructor, 则扩展用户定义的constructor去完成,
否则在第一次调用时隐式合成一个default constructor去完成。
如果没有上述4种情况又没有声明任何constructor的class，它的default constructor实际上并不会合成出来。





