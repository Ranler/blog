layout: date
title: JVM Instruction Set Example 1
date: 2013-4-25
categories: JVM
---

JVM Instruction Set官方手册：[jvms se7 chapter6](http://docs.oracle.com/javase/specs/jvms/se7/html/jvms-6.html)

JVM Instruction Wiki: [Java bytecode instruction listings](http://en.wikipedia.org/wiki/Java_bytecode_instruction_listings)

手册看看单个指令还是好理解的，下面看看一些例子，其中涉及的命令有：

```sh
$javac Test.java          // compiler
$javap -c Test.class      // disassembler
$java Test                // run
```

### Example 1.1

```java
    void f1() {
	int i = 0;
	int j = 0;
	int z = i + j;
	z++;
    }
```

```
  void f1();
    Code:                         // 以下局部变量数组是当前栈帧的局部变量数组
       0: iconst_0                // int型常量0入栈
       1: istore_1                // 栈顶int型值写入局部变量数组的第一个元素(i=0)
       2: iconst_0      
       3: istore_2                // 栈顶int型值写入局部变量数组的第二个元素(j=0)
       4: iload_1                 // 局部变量数组的第一个元素入栈，整型
       5: iload_2                 // 局部变量数组的第二个元素入栈，整型
       6: iadd                    // 取出栈顶两个int型元素，相加，结果入栈
       7: istore_3                // 栈顶int型值写入局部变量数组的第二个元素(z=i+j)
       8: iinc          3, 1      // 局部变量数组的第三个元素加1
      11: return
```

### Example 1.2

```java
    void f2() {
	Integer i = 100;
	Integer j = new Integer(100);
	Integer z = i + j;
	z++;
    }
```

```
void f2();
    Code:
       0: bipush        100        // byte类型100入栈顶，byte类型符号扩展成int
       2: invokestatic  #7         // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;自动装箱，结果在栈顶，是ref类型
       5: astore_1                 // 栈顶ref类型值写入局部变量数组的第1个元素(Integer i = 100;)
	   
	   
       6: new           #8         // class java/lang/Integer;创建instance,在heap中为此instance申请空间，其成员域初使化为默认值，向栈顶压入其ref
       9: dup                      // 复制栈顶元素，并压入栈顶
      10: bipush        100
      12: invokespecial #9         // Method java/lang/Integer."<init>":(I)V;调用Integer的初使化函数init(ref, 100)，将栈顶两个元素作为参数;dup拷贝的ref在这里使用掉了
      15: astore_2                 // 栈顶ref类型值写入局部变量数组的第2个元素(Integer j = new Integer(100););这个ref是dup前存在的ref
	  
	  
      16: aload_1                  // i的ref类型入栈
      17: invokevirtual #10        // Method java/lang/Integer.intValue:()I;自动拆箱，返回int类型值入栈
      20: aload_2       
      21: invokevirtual #10        // Method java/lang/Integer.intValue:()I
      24: iadd                     // 取出栈顶两个int元素，相加，入栈
      25: invokestatic  #7         // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;自动装箱
      28: astore_3                 // (Integer z = i + j;)   
	  
	  
      29: aload_3                  
      30: astore        4          // ? z的ref备份到局部变量数组的第4个位置，为了能够++先返回其原先值??
      32: aload_3                  
      33: invokevirtual #10        // Method java/lang/Integer.intValue:()I;z自动拆箱
      36: iconst_1      
      37: iadd          
      38: invokestatic  #7         // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;自动装箱，
      41: dup           
      42: astore_3                 // z指向了另一个instance,即更改了ref(z++;)
      43: astore        5          // ? 下面感觉没什么用了
      45: aload         4
      47: pop           
      48: return        
```

第一个指令`bipush`会做一个自动类型提升。
JVM定义如下自动类型提升：

- byte型,short型,char型会被提升到int型。
- 算数表达式的数据类型自动提升到与表达式中最高等级操作数同样的类型。

### Example 1.3

这个例子是一个经典笔试题了。

```java
    void f3() {
	Integer i1 = 100;
	Integer i2 = 100;
	Integer i3 = 0;
	Integer i4 = new Integer(100);
	Integer i5 = new Integer(100);
	Integer i6 = new Integer(0);
	
	System.out.println(i1 == i2);
	System.out.println(i1 == i2 + i3);
	System.out.println(i4 == i5);
	System.out.println(i4 == i5 + i6);

	Integer i7 = new Integer(200);
	Integer i8 = new Integer(200);
	System.out.println(i7 == i8);	
	System.out.println(i7 == i4 + i5);
    }
```

```
  void f3();
    Code:
       0: bipush        100
       2: invokestatic  #7    // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
       5: astore_1            // (i1 = 100;)
	   
	   
       6: bipush        100
       8: invokestatic  #7    // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      11: astore_2            // (i2 = 100;)
	  
	  
      12: iconst_0      
      13: invokestatic  #7    // Method java/lang/Integer.valueOf:(I)Ljava/lang/Integer;
      16: astore_3            // (i3 = 0;)
	  
	  
      17: new           #8    // class java/lang/Integer
      20: dup           
      21: bipush        100
      23: invokespecial #9    // Method java/lang/Integer."<init>":(I)V
      26: astore        4     // (i4 = new Integer(100);)
	  
	  
      28: new           #8    // class java/lang/Integer
      31: dup           
      32: bipush        100
      34: invokespecial #9    // Method java/lang/Integer."<init>":(I)V
      37: astore        5     // (i5 = new Integer(100);)
	  
	  
      39: new           #8    // class java/lang/Integer	
      42: dup           
      43: iconst_0      
      44: invokespecial #9    // Method java/lang/Integer."<init>":(I)V
      47: astore        6     // (i6 = new Integer(0);)
	  
	  
      49: getstatic     #11   // Field java/lang/System.out:Ljava/io/PrintStream;
      52: aload_1       
      53: aload_2       
      54: if_acmpne     61     // (i1 == i2)比较的ref值是否相等，即是否引用同一对象;不相等则话跳转至61
      57: iconst_1             
      58: goto          62
      61: iconst_0      
      62: invokevirtual #12    // Method java/io/PrintStream.println:(Z)V
	  
	  
      65: getstatic     #11    // Field java/lang/System.out:Ljava/io/PrintStream;
      68: aload_1       
      69: invokevirtual #10    // Method java/lang/Integer.intValue:()I
      72: aload_2       
      73: invokevirtual #10    // Method java/lang/Integer.intValue:()I
      76: aload_3       
      77: invokevirtual #10    // Method java/lang/Integer.intValue:()I
      80: iadd                 // 都拆箱，int相加
      81: if_icmpne     88     // (i1 == i2 + i3)，直接比较的int值
      84: iconst_1      
      85: goto          89
      88: iconst_0      
      89: invokevirtual #12    // Method java/io/PrintStream.println:(Z)V
	  
	  
      92: getstatic     #11    // Field java/lang/System.out:Ljava/io/PrintStream;
      95: aload         4
      97: aload         5
      99: if_acmpne     106    // (i4 == i5)，比较的ref值
     102: iconst_1      
     103: goto          107
     106: iconst_0      
     107: invokevirtual #12     // Method java/io/PrintStream.println:(Z)V
	 
	 
     110: getstatic     #11     // Field java/lang/System.out:Ljava/io/PrintStream;
     113: aload         4
     115: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     118: aload         5
     120: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     123: aload         6
     125: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     128: iadd          
     129: if_icmpne     136     // (i4 == i5 + i6),比较的int值
     132: iconst_1      
     133: goto          137
     136: iconst_0      
     137: invokevirtual #12     // Method java/io/PrintStream.println:(Z)V
	 
	 
     140: new           #8      // class java/lang/Integer
     143: dup           
     144: sipush        200
     147: invokespecial #9      // Method java/lang/Integer."<init>":(I)V
     150: astore        7       // (i7 = new Integer(200);)
	 
	 
     152: new           #8      // class java/lang/Integer
     155: dup           
     156: sipush        200
     159: invokespecial #9      // Method java/lang/Integer."<init>":(I)V
     162: astore        8       // (i8 = new Integer(200);)


     164: getstatic     #11     // Field java/lang/System.out:Ljava/io/PrintStream;
     167: aload         7
     169: aload         8
     171: if_acmpne     178     // (i7 == i8),比较的是ref值
     174: iconst_1      
     175: goto          179
     178: iconst_0      
     179: invokevirtual #12     // Method java/io/PrintStream.println:(Z)V
	 
	 
     182: getstatic     #11     // Field java/lang/System.out:Ljava/io/PrintStream;
     185: aload         7
     187: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     190: aload         4
     192: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     195: aload         5
     197: invokevirtual #10     // Method java/lang/Integer.intValue:()I
     200: iadd          
     201: if_icmpne     208     // (i7 == i4 + i5), 比较的是int值
     204: iconst_1      
     205: goto          209
     208: iconst_0      
     209: invokevirtual #12     // Method java/io/PrintStream.println:(Z)V
     212: return        

```

结果是：

```java
    void f3() {
	Integer i1 = 100;
	Integer i2 = 100;
	Integer i3 = 0;
	Integer i4 = new Integer(100);
	Integer i5 = new Integer(100);
	Integer i6 = new Integer(0);

	System.out.println(i1 == i2);           // true
	System.out.println(i1 == i2 + i3);      // true
	System.out.println(i4 == i5);           // false
	System.out.println(i4 == i5 + i6);      // true

	Integer i7 = new Integer(200);
	Integer i8 = new Integer(200);
	System.out.println(i7 == i8);           // false	
	System.out.println(i7 == i4 + i5);      // true
    }
```

这里还隐藏一个自动装箱时的优化。
自动装箱调用的是Integer.valueOf(int)，查看源码：

```java
﻿    public static Integer valueOf(int i) {
        if(i >= -128 && i <= IntegerCache.high)  // high = 127
            return IntegerCache.cache[i + 128];
        else
            return new Integer(i);
    }
```

这里会对值在-128~127的Integer做共享，相同的值共享同一对象。
因此其ref也相同。因此i1,i2指向同一对象，比较其ref时结果为true。

i3,i4,i7,i8直接创建了新的对象，指向了不同对象，ref不同。
因此i3==i4,i7==i8结果为false。

而其余几条都涉及代数计算，因此对象自动拆箱为基本类型进行计算，
结果也是比较其代数计算值的int类型结果。因此都为true。


