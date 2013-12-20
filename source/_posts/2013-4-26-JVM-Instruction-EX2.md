layout: post
title: JVM Instruction Set Example 2
date: 2013-4-26
categories: JVM
---

这次看一下String和常量池相关的操作。

`ldc`指令，把运行时常量池(runtime constant pool)中的元素压入操作栈。
压入元素的类型由常量池中元素类型决定：

- 如果常量池中的元素是int/float类型，那么压入的类型是float/int
- 如果是String类型的reference，那么压入其String对象的reference
- 如果是class类型的symbolic reference，那么压入其class对象的reference
- 如果是method type/handler的symbolic reference，那么压入其MethodType/MethodHandle对象的reference

可以看到压入的共两种类型：数值类型或reference类型

### Example 2.1

又是一个经典笔试题：

```java
    void f1() {
	String s1 = "abc";
	String s2 = "abc";

	String s3 = new String("abc");
	String s4 = new String("abc");

	System.out.println(s1 == s2);
	System.out.println(s3 == s4);
    }
```

```
  void f1();
    Code:
       0: ldc           #7     // String abc;常量池中取出其ref
       2: astore_1             // (String s1 = "abc";)
	   
	   
       3: ldc           #7     // String abc;常量池中取出其ref
       5: astore_2             // (String s2 = "abc";)
	   
	   
       6: new           #8     // class java/lang/String;创建新的对象并返回ref
       9: dup           
      10: ldc           #7     // String abc
      12: invokespecial #9     // Method java/lang/String."<init>":(Ljava/lang/String;)V
      15: astore_3             // (String s3 = new String("abc");)
	  
	  
      16: new           #8     // class java/lang/String;创建新的对象并返回ref
      19: dup           
      20: ldc           #7     // String abc
      22: invokespecial #9     // Method java/lang/String."<init>":(Ljava/lang/String;)V
      25: astore        4      // (String s4 = new String("abc"));
	  
	  
      27: getstatic     #10    // Field java/lang/System.out:Ljava/io/PrintStream;
      30: aload_1       
      31: aload_2       
      32: if_acmpne     39     // (s1 == s2;)，比较了其引用值
      35: iconst_1      
      36: goto          40
      39: iconst_0      
      40: invokevirtual #11    // Method java/io/PrintStream.println:(Z)V
	  
	  
      43: getstatic     #10    // Field java/lang/System.out:Ljava/io/PrintStream;
      46: aload_3       
      47: aload         4
      49: if_acmpne     56     // (s3 == s4;)，比较了其引用值
      52: iconst_1      
      53: goto          57
      56: iconst_0      
      57: invokevirtual #11    // Method java/io/PrintStream.println:(Z)V
      60: return        
```

“123”和“456”作为当前类的常量池中成员，在class加载到JVM是已经有对象了。
因此s1,s2分别获得了它们的ref。

只要有`new`指令就会在heap中申请内存，从而有一个新的对象实例，新的ref。
因此，这段程序的输出是：

```
true
false
```

### Example 2.2


```java
    void f2() {
	String s1 = "12" + "3";
	String s2 = "456";
	String s3 = s1 + s2;
	String s4 = s1 + s2 + s3;	
    }
```

```
  void f2();
    Code:
       0: ldc           #12    // String 123
       2: astore_1             // (String s1 = "12" + "3";),可以看出编译阶段可以优化字符串
	   
	   
       3: ldc           #13    // String 456
       5: astore_2             // (String s1 = "456";)
	   
	   
       6: new           #14    // class java/lang/StringBuilder
       9: dup           
      10: invokespecial #15    // Method java/lang/StringBuilder."<init>":()V
      13: aload_1       
      14: invokevirtual #16    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      17: aload_2       
      18: invokevirtual #16    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      21: invokevirtual #17    // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      24: astore_3             // (String s3 = s1 + s2;)
	  
	  
      25: new           #14    // class java/lang/StringBuilder
      28: dup           
      29: invokespecial #15    // Method java/lang/StringBuilder."<init>":()V
      32: aload_1       
      33: invokevirtual #16    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      36: aload_2       
      37: invokevirtual #16    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      40: aload_3       
      41: invokevirtual #16    // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      44: invokevirtual #17    // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      47: astore        4      // (String s4 = s1 + s2 + s3;)
      49: return        
```

每个String对象保存的字符串是不会改变的。
改变String只会新建一个新的String对象。

上段代码可以看出，对于通过`+`来进行多个字符串的拼接操作，
Oracle JDK是通过StringBuilder来实现的，
这也是一种高效的做法。
相比于StringBuffer，StringBuilder也免去了sync的代价。
(StringBuffer只是给StringBuilder的绝大部份方法添加了synchronized)
但是这里并没有复用StringBuilder，
每个子符串连接的Java语句都要new一个新的StringBuilder对象，
这也是这种`+`操作耗时的地方。


### Example 2.3

这个例子看看String.intern()。

```java
    void f3() {
	String s1 = "123";
	String s2 = new String("123");

	System.out.println(s1 == s2);
	System.out.println(s1.intern() == s2.intern());
	System.out.println(s1 == s1.intern());
	System.out.println(s2 == s2.intern());
	System.out.println(s1 == s2.intern());
    }
```

指令码就不贴了，和上面没什么区别。

String.intern()的实现通过JNI由本地代码实现的，
其功能是在字符串常量池中查找与当前字符串字面值相等的字符串，
返回其在字符串常量池中的引用。
如果不存在，则加入后返回。
因此调用String.intern()可能会扩充字符串常量池。

```java
public naive String intern();
```

字面值相同的字符串调用intern()都会获得同一字符串对象的引用。


[这个页面](﻿http://stackoverflow.com/questions/10624232/performance-penalty-of-string-intern)里的测试说明intern()的调用是及其昂贵的，
大概是O(n)的算法复杂度，n是字符串常量池的大小。

这段代码的结果是：

```
false
true
true
false
true
```

s1开始就获得了字符串常量池中的对象，因此s1和s1.intern()都是同一对象的引用值，
结果相等。

### Example 2.4

最后比较下StringBuilder复用和·+·直接拼接的效率：

```java
public class TestStrBuilder {
	static String s1 = "1";
	static String s2 = "2";
	static String s3 = "3";
	static String s4 = "4";
	static String s5 = "5";
	static String s6 = "6";
	
	public static void main(String[] args) {
		f1(10000000);
		f2(10000000);
		f1(100000000);
		f2(100000000);
	}
	
	public static void f1(long time) {
		long cost = System.nanoTime();
		for (long i = 0; i < time; i++) {
			String s = s1 + s2 + s3 + s4 + s5 + s6 + s5 + s4 + s3 + s2 + s1;			
		}
		cost = System.nanoTime() - cost;
		System.out.println(cost / 1000000);
	}
	
	public static void f2(long time) {
		StringBuilder sb = new StringBuilder();
		long cost = System.nanoTime();
		for (long i = 0; i < time; i++) {
			sb.append(s1).append(s2).append(s3).append(s4).append(s5).append(s6);
			sb.append(s5).append(s4).append(s3).append(s2).append(s1);
			String s = sb.toString();
			sb.delete(0, sb.length());
		}
		cost = System.nanoTime() - cost;
		System.out.println(cost / 1000000);		
	}
}
````

在OpenJDK 1.7.0_21 64bit下结果是：

```
1465
1230
13044
10681
```

大概复用StringBuilder时效率提高了17%。
