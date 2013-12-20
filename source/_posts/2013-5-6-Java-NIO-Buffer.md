layout: post
title: Java NIO Buffer
date: 2013-5-6
categories: JVM
---

classic Java I/O(`java.io`) backwards:

- do not scale well when moving large amounts of data
- lack of some common I/O features such as
  - file locking
  - nonblocking I/O
  - readiness selection
  - memory mapping


Java NIO(`java.nio`), JSR 51, JDK1.4 2002

- Base: Buffers
- Channels
- Selectors

### Buffer

抽象类`java.nio.Buffer`代表了一段数据，主要5个数据成员：

- capacity:  元素最大容量，一但初使化就不能改变
- limit:     有效元素尾端
- position:  下一个待读写的位置索引，`get()`和`set()`自动修改
- mark:      标记的索引，`mark()`使得mark=position，`reset()`使得position=mark

0 <= mark <= position <= limit <= capacity

Buffer不是线程安全的类。

几个常用操作：

- `get()`, `put(E)`: position++;
- `flip()`: limit=position,position=0,mark=-1;
- `compact()`: 将[position, limit)区间内的元素拷贝到起始位置[0, limit-position),经常和`flip()`组合使用
- `rewind()`: position=0,mark=-1;
- `remaining()`: 返回limit-position;
- `clear()`: limit=capacity, position=0,mark=-1;


clear() -> channel读入 -> flip() 
flip() -> channel写出 -> clear()

### eight buffer classes

These are all abstract classes:

- IntBuffer
- DoubleBuffer
- ShortBuffer
- LongBuffer
- FloatBuffer
- ByteBuffer
- CharBuffer
- MappedByteBuffer: like mmap()

But these are contain factory method(`allocate(int capacity )`) to create new instance. Or use `wrap(T[])` to create new instance with **backing array**.

Nondirect buffers have backing arrays.
It can be used by `hasArray()` and `array()`.

### Duplicating Buffers

- `duplicate()`: 创建Buffer，共享(0, capacity)部份数组
- `slice()`: 创建Buffer，共享(position, limit)部份数组

### Byte Buffers

ByteBuffer byte order is BIG_ENDIAN.
Only ByteBuffer can be Direct Buffers.
Direct Buffer创建昂贵(jni)，但是对于I/O操作十分高效。

使用`ByteBuffer.allocateDirect()`创建Direct Buffer。


### Tips

- 在Java中，字符串使用Unicode表示，因此每个Unicode字符占16bits。
所以Java中char类型是2个字节:
	- byte   1B
	- char   2B
	- short  2B
	- int    4B
	- float  4B
	- long   8B
	- double 8B
	- boolean Java规范未规定，JVM规范规定boolean映射成int，boolean数组映射成byte数组。
	


### Reference

- Java NIO book

