title: Data Structure Alignment
date: 2014-01-25 10:05:34
categories: C
---

- [WIKIPEDIA: Data structure alignment](http://en.wikipedia.org/wiki/Data_structure_alignment)
- [IBM: Data alignment: Straighten up and fly right](http://www.ibm.com/developerworks/library/pa-dalign/)


数据结构对齐包含两个问题：

- data alignment 数据对齐
- data structure padding 数据结构填充

现代CPU读写内存是以word为单位（4B for 32-bit, 8B for 64-bit），
并且访问地址是word的倍数。
一个数据对象即使是word大小，但是地址不是word的倍数，那么就需要两次访问才能取出。
对于更老的CPU，如果程序不按照对齐地址访问数据对象，则直接抛出异常。

为了增加程序性能，CPU从内存中取数据时要减少访问次数。
**数据对齐**保证了每个数据对象的起始地址是word的倍数，
**数据结构填充**保证每个数据对象的大小是word的倍数，以便下一个数据对象的起始地址是word的倍数。






