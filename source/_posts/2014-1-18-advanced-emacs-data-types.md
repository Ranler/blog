title: Advanced Emacs (1): ELisp Data Types
date: 2014-01-18 13:48:47
categories: emacs
---

Emacs用了也有三四年了，但是感觉自己还没有到达中级水平。
这是因为我只是用了Emacs这个工具，只是掌握了一些trick，添加和修改了一些插件，
但没有去理解Emacs的运行机制，也没有很系统的学习ELisp，
所以渐渐感到驾驭不了各种各样不同风格的插件，也不能按照自己的想法去扩展。
因此目前需要进阶一下，就先从ELisp入手。


### 测试环境

在Emacs中测试ELisp的环境是切换到`*scratch*`缓冲里，使用`lisp-interaction-mode`模式。
输入下面语句，使用`C-j`或`C-x C-e`执行：

``` lisp
(message "hello world")
```


### 1. 原子(atom)数据类型 ###
------

在Lisp中,除了序对(cons cells，下节介绍)组成的类型之外的其它所有类型称为**原子(atom)**类型。
以下是常见的原子类型：

### 1.1 数字 integer & floating

数字包括整数(Integer, 30bits)，浮点数(Floating, double)。
可以从`most-positive-fixnum`和`most-negative-fixnum`两个变量得到整数的围。
`NaN`也表示特殊值。

相关函数：

``` lisp
;; 测试函数
(integerp OBJECT)
(floatp OBJECT)
(numberp OBJECT)
(zerop NUMBER)
(wholenump OBJECT)
;; 比较函数
(> NUM1 NUM2)
(< NUM1 NUM2)
(>= NUM1 NUM2)
(<= NUM1 NUM2)
(= NUM1 NUM2)
(eql OBJ1 OBJ2)
(/= NUM1 NUM2)
;; 转换函数
(float ARG)
(truncate ARG &optional DIVISOR)
(floor ARG &optional DIVISOR)
(ceiling ARG &optional DIVISOR)
(round ARG &optional DIVISOR)
;; 运算
(+ &rest NUMBERS-OR-MARKERS)
(- &optional NUMBER-OR-MARKER &rest MORE-NUMBERS-OR-MARKERS)
(* &rest NUMBERS-OR-MARKERS)
(/ DIVIDEND DIVISOR &rest DIVISORS)
(1+ NUMBER)
(1- NUMBER)
(abs ARG)
(% X Y)
(mod X Y)
(sin ARG)
(cos ARG)
(tan ARG)
(asin ARG)
(acos ARG)
(atan Y &optional X)
(sqrt ARG)
(exp ARG)
(expt ARG1 ARG2)
(log ARG &optional BASE)
(log10 ARG)
(logb ARG)
;; 随机数
(random &optional N)
```


### 1.2 字符 character ###

字符(Character, 包括ASCII和宽字符，22bits)。字符也是数字。

字符读入语法是在字符前加上问号。比如`?A`代表字符`A`。
和C相同，一些控制字符需要加上转义符，如制表符`?\t`。

`\M-`表示Meta/Alt键，`\C-`表示Ctrl键。

相关函数：

``` lisp
;; 测试函数
(stringp OBJECT)
(string-or-null-p OBJECT)
(char-or-string-p OBJECT)
;; 构造函数
(make-string LENGTH INIT)
(string &rest CHARACTERS)
(substring STRING FROM &optional TO)
(concat &rest SEQUENCES)
;; 比较函数
(char-equal C1 C2)
(string= S1 S2)
(string-equal S1 S2)
(string< S1 S2)
;; 转换函数
(char-to-string CHAR)
(string-to-char STRING)
(number-to-string NUMBER)
(string-to-number STRING &optional BASE)
(downcase OBJ)
(upcase OBJ)
(capitalize OBJ)
(upcase-initials OBJ)
(format STRING &rest OBJECTS)
;; 查找与替换
(string-match REGEXP STRING &optional START)
(replace-match NEWTEXT &optional FIXEDCASE LITERAL STRING SUBEXP)
(replace-regexp-in-string REGEXP REP STRING &optional FIXEDCASE LITERAL SUBEXP START)
(subst-char-in-string FROMCHAR TOCHAR STRING &optional INPLACE)
```

### 1.3 符号 symbol ###

在Lisp中使用变量的必须通过**符号(Symbol)**类型来实现。
符号类型由4个部分组成：。：

- 符号名(Name)：只能绑定string对象。`symbol-name`访问。
- 符号值(Value)：可以绑定任意类型。`symbol-value`访问，`defvar`和`defconst`赋值。
- 函数(Function)：应该绑定function对象或macro对象。`symbol-function`访问，`defun`和`defmacro`赋值。
- 属性列表(Property list)：应该绑定prperty list对象。`symbol-plist`访问。

其中符号名必须绑定一个string对象，其它三个部分可以独立地绑定任何类型的对象。
其中符号值和函数部分可以为空。
因此符号类型是一种符号名到其它三个绑定对象的映射关系。

符号名是由字符串组成，有命名规则。
符号名具有唯一性。
全局变量obarray记录着emacs中所有全局符号名到符号映射。
也可以说这些符号使用相同的命名空间。
`setq`可以设置全局变量，并绑定符号值或函数部分。
也可以创建新的obarray，由`set`加入符号。
obarray是自动增长的vector类型，通过hash符号名找到符号在vector中的位置。
vector中每个元素是一个bucket，在bucket中符号由链表连接。
在ELisp中只能通过`make-vector`创建obarray，`intern`插入符号。

符号在Lisp求值时会根据符号名取出符号对象，然后返回符号值或函数并求值。
而其它原子类型(数字，字符)都是**自求值**：简单返回自身。

下面两种特殊的符号的符号值不能修改（也可以看成自求值的）：

1. 关键字(keywork)符号：符号名以`:`开始的符号。
2. `t`和`nil`表示的真假特殊符号。

在Lisp中，符号`nil`有3三种不同的意义：

- 逻辑上代表事实中的“假”
- 一个名为nil的符号类型
- 一个空的表类型，等价`()`

``` lisp
(symbolp OBJECT)
(intern-soft NAME &optional OBARRAY)
(intern STRING &optional OBARRAY)
(unintern NAME &optional OBARRAY)
(mapatoms FUNCTION &optional OBARRAY)
(symbol-name SYMBOL)
(symbol-value SYMBOL)
(boundp SYMBOL)
(set SYMBOL NEWVAL)
(setq SYM VAL SYM VAL ...)
(symbol-function SYMBOL)
(fset SYMBOL DEFINITION)
(fboundp SYMBOL)
(symbol-plist SYMBOL)
(get SYMBOL PROPNAME)
(put SYMBOL PROPNAME VALUE)
```



### 2. 复合数据类型 ###
------



### 2.1 序对 Cons Cells ###

Lisp提供一种称为**序对(Cons Cells, construction of cells,或称Conses)**的复合结构。

![](http://cs.gmu.edu/~sean/lisp/cons/cons.GIF)

序对包含两个对象（或称槽slot)，第一个对象叫CAR(Contents of Address part of Register)，
第二个对象叫CDR(Contents of the Decrement part of Register)。
每个对象可以是任何类型，这是cons的**闭包性质**。
`(cons 1 2)`表示用盒子指针表示：

![](http://mitpress.mit.edu/sicp/full-text/book/ch2-Z-G-11.gif)

<pre>
对于C程序员来说：Lisp中并不区分保存一个值和指向一个值，因为指针在Lisp中是隐式的。
</pre>

如前面所示，序对可以用函数`cons`构造出来`(cons CAR CDR)`：

<pre>
(defvar x (cons 1 (cons 2 3)))    ; => x
(car x)                           ; => 1
(cdr x)                           ; => (2 . 3)
(car (cdr x))                     ; => 2
(cdr (cdr x))                     ; => 3
</pre>

`(CAR . CDR)`也是一种构造序对的方法：

<pre>
(cons 1 (cons 2 3))               ; => (1 2 . 3)
'(1 . (2 . 3))                    ; => (1 2 . 3)
</pre>

序对可以用作构造任意种类的复杂数据结构的通用的基本构件，也是Lisp的核心数据结构类型。
（这和哲学中任何多元论都可转变为二元论的思想一致。）
正因为序对如此重要，我们把Lisp中不是序对的对象称为**原子(atom)**。

### 2.2 序列 Sequence ###

序列表示的是有序元素的集合。
在Emacs Lisp中，序列有两种类型：**列表(list)**和**数组(array)**。

一些叫做序列函数的函数，可以接受任何类型的序列。

### 2.3 列表 list ###

采用序对表示序列的方式很多，一种最直接的表示方式是把序对串成单链表一样，
每个序对的car部分对应每个链中的条目，cdr部分则是链中下一个序对，最后一个序对的cdr用`nil`表示。
这种嵌套序对形成的序列称为**表(list)**。

![](http://mitpress.mit.edu/sicp/full-text/book/ch2-Z-G-13.gif)

`()`或`nil`也可以当作一个不包含任何元素的序列，称为空表。
因此`(A ())`等价于`(A nil)`。

![](https://public.bay.livefilestore.com/y1pgqtdo8MaVzUCe2V7FFJjKmpksx4hOKWfDzrF_tSrYDnJBKVwPpvfag-Rc2qOt7I4BwpDZeva3QOGTIEI0zcXFA/emacs-a-nil.png?psid=1)

除了用`cons`和`.`嵌套地定义表之外，lisp方便地提供基本操作`list`来构造表。

<pre>
(cons a1 (cons a2 (cons ... (cons an nil) ... )))
(a1 . (a2 ... (an-1 . (an)) ... ))
(list a1 a2 ... an)
</pre>

显然，`cons`可以用于在已有的表的最前面添加一个元素。
`append`用于合并两个表。
`length`用于返回表的长度。

<pre>
(defvar l (list 1 2 3))        ; => l
(cons 0 l)                     ; => (0 1 2 3)
(append l l)                   ; => (1 2 3 1 2 3)
(length l)                     ; => 3
</pre>

`mapc`,`mapcar`,`dolist`用于遍历list。

取出list元素的操作有`nth`，获得列表一个区间的函数有`nthcdr`、`last`和`butlast`。

#### 2.3.1 列表模拟其它数据结构

使用`push`,`pop`操作操纵list可以模拟**栈**的行为。

使用`append`,`delete-dups`,`memq`,`member`,`delq`,`remq`操作操纵list可以模拟**集合**的行为。

把序对的看成指向左右孩子节点的指针，可以把这种结构看成二叉树。进一步推广，可以看成**树**。

![](http://mitpress.mit.edu/sicp/full-text/book/ch2-Z-G-15.gif")


也可以实现association list关联列表（和2.6节hash table有区别）。

list相关函数：

``` lisp
;; 测试
(consp OBJECT)
(listp OBJECT)
(null OBJECT)
;; 构造
(cons CAR CDR)
(list &rest OBJECTS)
(append &rest SEQUENCES)
;; 访问list元素
(car LIST)
(cdr LIST)
(cadr X)
(caar X)
(cddr X)
(cdar X)
(nth N LIST)
(nthcdr N LIST)
(last LIST &optional N)
(butlast LIST &optional N)
;; 修改 cons cell
(setcar CELL NEWCAR)
30 1ÊÙ Äy-sequence ARG)
(copy-tree TREE &optional VECP)
;; 数组
(vector &rest OBJECTS)
(make-vector LENGTH INIT)
(aref ARRAY IDX)
(aset ARRAY IDX NEWELT)
(vconcat &rest SEQUENCES)
(append &rest SEQUENCES)
```

### 2.5 函数 function ###

在Lisp中，函数是Lisp对象，是第一类型。
函数有以下几类：

- lisp函数
- primitive function 原子函数
- lambda function
- 特殊表达式
- macro
- command: 包含`interactive`

`eval`,`funcall`,`apply`。


Lisp中允许使用`lambda`定义匿名函数(anonymous function)。
一个命名函数(named function)只是一个符号，其函数单元(function cell)指向一个有效函数。

### 2.6 哈希表 hash table ###

哈希表是一个快速索引表。



### 参考 ###
- [SICP](http://mitpress.mit.edu/sicp/full-text/book/book.html)
- [GNU Emacs Lisp Reference Manual](http://www.gnu.org/software/emacs/manual/html_node/elisp/index.html)





