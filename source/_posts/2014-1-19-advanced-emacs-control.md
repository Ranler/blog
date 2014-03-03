title: Advanced Emacs (2): ELisp 控制语句
date: 2014-01-19 11:00:20
categories: emacs
---

### 1. 数据类型

primitive types:

- 整数
- 浮点数
- symbol 符号：包括关键字和特殊符号

- cons 序对
- 序列：list，以及模拟的stack，set，tree，association list
- 数组：包括向量(vector)，字符串(string)，字符表(char-table)，布尔向量(bool-vector)
- hash-table 散列表

- 内建函数：比如cons, if, and之类
- byte-code function, primitive function原子函数
- 其它特殊类型：如缓冲区(buffer)

### 求值规则

可以求值的Lisp对象被称为表达式(expression/form)。
表达式有以下几种类型：

- 自求值表达式：数字、字符、关键字、特殊符号(t,nil)、数组(vector, string)。
- 符号：返回符号值。
- 列表list：list的第一个元素可以是function，macro或special form。如果这个元素是符号，那么返回符号function部分对象。
  - function: list中参数元素求值，作为实参使用`apply`调用函数。
  - macro: 根据宏定义扩展，然后函数调用。
  - special form:  list中参数元素可能并不会全求值。 每个特殊表达式都有对应的求值规则。

符号的函数值可以是一个 lisp 函数 （ lambda 表达式） 、 byte-code函数、 原子函数 （ primitive function） 、 宏、 特殊表达式或 autoload 对象。

`'`或`quote`可以阻止S-表达式求值。


### 2. 变量


变量无需声明，可以直接定义（赋值）。

``` lisp
(setq variable-name value)
(defvar variable-name value
  "document string")
```

`defvar`可以定义未定义的变量，并可以为已定义的变量添加document string。
使用`C-h v`查看变量的document string。

ELisp中函数是全局的， 变量也很容易成为全局变量，
因为局部变量和局部变量的赋值都是使用setq函数。

##### 局部变量

使用`let`或`let*`，后者可以使用已定义的变量。

``` lisp
(let (bindings)
  body)
```

##### buffer-local 变量

使用 `make-local-variable` 创建buffer-local变量。

##### 函数的形式参数

##### 作用域(scope)与生存期(extent)

[以前讨论过](http://findfunaax.com/notes/file/126)，ELisp是一种动态调用域。
当它遇到变量的符号名时，会逐层检查运行时的调用栈寻找这个符号，直至最外层的全局变量。
如果都没有定义，则报出Symbol's value as variable is void错误。
也因此Emacs在编译时并不能检查变量存不存在，只有运行时才知道。

ELisp不支持闭包。

生存期指对象在内存中的创建和释放。

- 全局变量：创建:第一次定义时；释放：Emacs关闭时或`unintern`
- buffer-local变量：创建：第一次定义时；释放：buffer关闭或`kill-local-variable`
- 局部变量：创建：`let`执行时；释放：离开`let`表达式
- 函数的形式参数：创建：函数调用时。释放：离开函数


命名习惯后缀：

- hook 一个在特定的况下调用的函数列表， 比如关闭缓冲时， 进入某个模式时。
- function 值为一个函数
- functions 值为一个函数列表
- flag 值为 nil 或 non-nil
- predicatee 值是一个判断函数，返回nil或non-nil
- program 或-command 一个程序或shell命令
- form 一个表达式
- forms 一个表达式列表。
- map 一个按键映射(keymap)



### 3. 函数

``` lisp
(defun function-name (arguments-list)
  "document string"
  body)
```

函数都有返回值，返回值是body中最后一个表达式的值。

使用`C-h f`可以在`*help*`缓冲中显示函数的document string和源码链接。

##### 匿名函数 lambda

lambda表达式返回匿名函数，可以通过`setq`赋予变量。

``` lisp
(lambda (arguments-list)
  "documentation string"
  body)
```

``` lisp
(setq foo (lambda (name)
(message "Hello, %s!" name)))
  (funcall foo "Emacser")
```

### 4. 控制流

##### 顺序流

``` lisp
(progn A B C ...)
```

##### 条件判断

``` lisp
;; 两类
(if condition
  then
  else)

;; 多类
(cond (case1 do-when-case1)
      (case2 do-when-case2)
      ...
      (t do-when-none-meet))
```

- `when` 省去if里的progn结构
- `unless` 省去条件为真子句需要的nil表达式

##### 循环

``` lisp
(while condition
  body)
```

### 运算

##### 逻辑运算

- `and`: 可用于代替`when`和参数检查，短路性质
- `or`: 可用于代替`unless`和设置函数参数缺省值，短路性质
- `not`: 


### 库

- 模拟的 Common Lisp 的库 cl, RMS不推荐使用
- emms



### 参考

- Elisp 入门 叶文彬 
- GNU Emacs Lisp Reference Manual
