---
layout: default
title: CPP String Implement
---

String

char*
COW
ADT

### COW

COW必然需要referecne count。
引用的内容是heap上的对象。

只有通过string object对象作为参数调用copy constructor或operator=才能触发COW。







- [标准C＋＋类std::string的内存共享和Copy-On-Write技术-陈皓](http://blog.csdn.net/haoel/article/details/24058)
- [STL 的string类怎么啦？](http://blog.csdn.net/haoel/article/details/1491219)
- [STL中string的源码解读](http://blog.csdn.net/abortexit/article/details/1638254)
