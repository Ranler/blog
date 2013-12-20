layout: post
title: CPP Casting
date: 2013-7-24
categories: C&CPP
---

之前分析过[类型系统与C语言类型转换](158)。
C++是对C的继承，必定兼容C的类型系统。
但是C++新增的class及其后来的多态特性使得其有自己的类型转换方式。

### 1. C++ Cast

### 1.1 static_cast

`static_cast<new_type>(expression)`

`static_cast` is used for cases where you basically want to **reverse an implicit conversion**, with a few restrictions and additions. 
`static_cast` performs **no runtime checks**. 
This should be used if you know that you refer to an object of a specific type, and thus a check would be unnecessary.
`static_cast` does not allow casts between fundamentally different types, such as a cast between a BaseA and BaseB if they are not related. This will result in a compile time error.

`static_cast` 把隐式转换变为显示转换（例如，non-const 对象转型为 const 对象，int 转型为 double，，void* 指针转型为有类型指针，基类指针和派生类指针互转，等等）。
`static_cast` 不进行运行时检查。

`static_cast`它不能将一个 const 对象转型为 non-const 对象（只有 const_cast 能做到）。

如`(int)c`写成`static_cast<int>(c)`，`Wdiget(15)`可以写成`static_cast<Widget>(15)`。
转型可能会产生代码，如float转成long型，会产生代码对float数据格式转换和舍入。


结论：`static_cast`可用于基本类型转换，继承体系中的转换，不能跨类型转换。

### 1.2 dynamic_cast

`dynamic_cast<new_type>(expression)`

`dynamic_cast` is used for cases where you don't know what the dynamic type of the object is. 
You cannot use dynamic_cast if you downcast and the argument type is not polymorphic. 

`dynamic_cast`只用于多态对象指针(polymorphic objects: objects of classes which define at least one virtual function)中的转换，执行“安全的向下转型（safe downcasting）”
`dynamic_cast`进行运行时检查(有运行代价)，依赖RTTI把基类指针转为子类指针。
如果指定对象不是基类的子类，那么返回NULL。

结论：`dynamic_cast`只能用于多态对象的安全向下类型转换。

An "up-cast" is always valid with both `static_cast` and `dynamic_cast`, and also without any cast, 
as an "up-cast" is an implicit conversion.

```.cpp
class A{
    int a;
    virtual ~A(){}; // defined for dynamic_cast
};


class B{
    double b;
};

class C: public A{
    double c;  
};


int main(int argc, char *argv[])
{
    A *a = new A;
    // B *sb = static_cast(B*)(a); // error when compile
    C *sc = static_cast<C*>(a);    // no error when compile,  but invalid

    B *db = dynamic_cast<B*>(a);   // no compile error, db == 0
    C *dc = dynamic_cast<C*>(a);   // no compile error, dc == 0

    return 0;
}
```

### 1.3 const_cast

`const_cast<new_type>(expression)`

`const_cast`一般用于强制消除对象的常量性。
它是唯一能做到这一点的 C++ 风格的强制转型。 

### 1.4 reinterpret_cast

`reinterpret_cast<new_type>(expression)`

This is the ultimate cast, which disregards all kind of type safety, allowing you to cast anything to anything else.

reinterpret_cast 是特意用于底层的强制转型，导致实现依赖（implementation-dependent）
（就是说，不可移植）的结果，例如，将一个指针转型为一个整数。这样的强制转型在底层代码以外应该极为罕见。


### 2. C-style Cast / Regular Cast

`(T)expression`，如`(int)c`

`T(expression)`，如`Widget(15)`，等价于创建一个Widget对象并把15作为构造函数参数

A c-style cast is basically identical to trying out a range of sequences of C++ casts,
and taking the first c++ cast that works, without ever considering `dynamic_cast`. 
Needless to say that this is much more powerful as it combines all of `const_cast`(?), `static_cast` and `reinterpret_cast`,
but it's also unsafe because it does not use `dynamic_cast`. 


### 3. C++ 隐式类型转换

C++允许编译器在不同类型之间执行隐式转换(implicit conversions)。
基本的就是继承了C的基本类型转换，包括安全的转换(如char->double)和不安全的转换(double->char)。

只有当对象以by value或以reference-to-const方式传递参数时，才会发生隐式类型转换。
以reference-to-non-const参数传递时，不会发生此类转换。(MoreEffectiveC++ P100)

C++有两种函数也支持隐式转换：

### 3.1 单自变量constructors

这种情况也可以是多个参数，并且除了第一个参数之外都有默认值。

``` cpp
class Name {
public:
	Name(const string& s);
	...
};
void printName(Name& name);

printName("Joke");
```

这种情况下将隐式地把字符串类型数组(char*)转换为string类型对象，再把string类型对象隐式转换为Name类型对象，
最后传给printName函数。

防止这种隐式类型转换的方法是在Name前加上`explicit`关键字。

### 3.2 隐式类型转换操作符

在operator成员函数关键字之后加上一个类型名，表示该类型能够隐式转换成目标类型。

``` cpp
class Rational {
public:
   ...
   operator double() const;
   ....
};

Rational r(1,2);
double d = 0.5 * r;
```

表示Rational类型能够隐式转换成double类型。


### 4. boost cast

### 4.1  polymorphic_cast 

The C++ built-in `static_cast` can be used for efficiently downcasting pointers to polymorphic objects, 
but provides no error detection for the case where the pointer being cast actually points to the wrong derived class.
The `polymorphic_downcast` template retains the efficiency of `static_cast` for non-debug compilations, 
but for debug compilations adds safety via an assert() that a dynamic_cast succeeds.

The C++ built-in `dynamic_cast` can be used for downcasts and crosscasts of pointers to polymorphic objects, 
but error notification in the form of a returned value of 0 is inconvenient to test, or worse yet, easy to forget to test.
The throwing form of `dynamic_cast`, which works on references, can be used on pointers through the ugly expression `&dynamic_cast<T&>(*p)`, which causes undefined behavior if p is 0. 
The `polymorphic_cast` template performs a `dynamic_cast` on a pointer, and throws an exception if the `dynamic_cast` returns 0.

### 4.2 polymorphic_downcast

A `polymorphic_downcast` should be used for downcasts that you are certain should succeed. Error checking is only performed in translation units where NDEBUG is not defined, via

```
  assert( dynamic_cast<Derived>(x) == x )
```

where x is the source pointer. This approach ensures that not only is a non-zero pointer returned, but also that it is correct in the presence of multiple inheritance. Attempts to crosscast using `polymorphic_downcast` will fail to compile. Warning: Because `polymorphic_downcast` uses assert(), it violates the One Definition Rule (ODR) if NDEBUG is inconsistently defined across translation units.

For crosscasts, or when the success of a cast can only be known at runtime, or when efficiency is not important, polymorphic_cast is preferred. 

The C++ built-in dynamic_cast must be used to cast references rather than pointers. It is also the only cast that can be used to check whether a given interface is supported; in that case a return of 0 isn't an error condition.


## 参考

- [http://www.cnblogs.com/welfare/articles/336091.html](http://www.cnblogs.com/welfare/articles/336091.html)
- [http://msdn.microsoft.com/zh-cn/library/vstudio/hh279663.aspx](http://msdn.microsoft.com/zh-cn/library/vstudio/hh279663.aspx)
- [http://stackoverflow.com/questions/28002/regular-cast-vs-static-cast-vs-dynamic-cast](http://stackoverflow.com/questions/28002/regular-cast-vs-static-cast-vs-dynamic-cast)
- [http://www.boost.org/doc/libs/1_47_0/libs/conversion/cast.htm#Polymorphic_cast](http://www.boost.org/doc/libs/1_47_0/libs/conversion/cast.htm#Polymorphic_cast)
