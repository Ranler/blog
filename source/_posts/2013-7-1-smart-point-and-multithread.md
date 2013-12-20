layout: post
title: Smart Pointer and MultiThread in CPP
date: 2013-7-1
categories: C&CPP
---

### Smart Pointer

智能指针用于管理heap上对象的生命周期。

- `std::auto_ptr`: ptr:对象=1:1, ptr引用的对象可以转移（通过拷贝或赋值）(C++11已被废弃，使用`std::unique_ptr`代替)。
- `boost::scoped_ptr`: ptr:对象=1:1, ptr引用的对象不可转移。
- `boost::shared_ptr`: ptr:对象=N:1, 内置引用计数器。无法解决循环引用。
- `boost::weak_ptr`: ptr:对象=N:1, 可以手工解决循环引用。

后三个在`std::tr1`中也提供。

`weak_ptr`只能通过`shared_ptr`或`weak_ptr`创建，因此其必须对应一个`shared_ptr`，但是不会增加`shared_ptr`的引用计数，因此是一个弱引用。`weak_ptr`通过**提升/lock()**获得对应的`shared_ptr`。如果对象已释放，则返回空的`shared_ptr`。

`shared_ptr`避免循环引用通常的做法是：如果是

```
        ---(shared_ptr)---->
owner                          child
        <--(shared_ptr)-----
```

的情况，则设计成：

```
        ---(shared_ptr)---->
owner                           child
        <---(weak_ptr)-----
```



### MultiThread

##### shared_ptr

```.cpp
template<class T> class shared_ptr
{
...
private:
    T * px;                     // contained pointer
    detail::shared_count pn;    // reference counter
}
```

`shared_ptr`有两个数据成员，pn是引用计数器。
`shared_ptr`的计数操作是安全且无锁的原子操作。
因此在多线程中通过拷贝或赋值操作创建新的`shared_ptr`对象是安全的，能够正确记录引用的值。

```.cpp
// thread A
shared_ptr<int> p2(p); // reads p

// thread B
shared_ptr<int> p3(p); // read p, OK, multiple reads are safe
```

上面代码读取了同一个`shared_ptr`对象，结果是正确的。
但是同一个`shared_ptr`被多个线程写入时，结果就未定义了（因为有两个成员，pn并不是执行计数操作，而是被赋值），这时就需要加锁。
`shared_ptr`对象的析构也算写入操作。


### Reference

- Linux多线程服务端编程
- http://www.boost.org/doc/libs/1_53_0/libs/smart_ptr/shared_ptr.htm#ThreadSafety
