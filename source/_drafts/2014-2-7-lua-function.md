title: Lua Function
date: 2014-02-07 20:25:12
categories: Lua
---

- lua_State 表示Lua每个线程的执行状态机，拥有独立的数据栈和函数调用链，以及独立的调试钩子和错误处理设施。
- global_State 完整的Lua虚拟机，被所有线程的lua_State共享。

C用户持有lua_State对象，通过lua_State对象访问Lua的每个状态机，或者说每个线程。



### 数据栈与调用栈

[在分析虚拟机指令时](http://www.findfunaax.com/notes/file/61)讲过，Lua虚拟机为函数调用准备了两个平行的栈：

- 数据栈：TValue数组实现。每个数组保存着所有函数调用时的数据（常量、变量、匿名变量）。
- 调用栈：CallInfo双向链表实现。每个CallInfo保存着一个函数调用时的状态。

数据栈和调用栈这种划分和C语言不同。
C语言把函数调用所需的静态数据保存在数据段，把变量、匿名变量和函数状态保存在栈段。
而Lua把常量、变量、匿名变量保存在数据栈，把函数调用状态保存在调用栈。
由此，每个lua_State包含一个TValue数组组成的数据栈和一个CallInfo双向链表组成的调用栈。

### 数据栈

前面说过，数据栈由TValue数组组成。
lua_State通过以下几个字段和数据栈相关：

``` c
// 
struct lua_State {
   ...
  StkId top;         /* first free slot in the stack */
  StkId base;        /* base of current function */
  StkId stack_last;  /* last free slot in the stack */
  StkId stack;       /* stack base */
  int stacksize;
  ...
};
```

下面函数分别初始化了数据栈的TValue数组和调用栈链表的第一个CallInfo节点（TODO: 第二个参数L的作用？）：

``` c
static void stack_init (lua_State *L1, lua_State *L) {
  int i; CallInfo *ci;

  /* initialize stack array */
  L1->stack = luaM_newvector(L, BASIC_STACK_SIZE, TValue);
  L1->stacksize = BASIC_STACK_SIZE;
  for (i = 0; i < BASIC_STACK_SIZE; i++)
    setnilvalue(L1->stack + i);  /* erase new stack */
  L1->top = L1->stack;
  L1->stack_last = L1->stack + L1->stacksize - EXTRA_STACK;

  /* initialize first ci */  // CallInfo下一节再分析
  ci = &L1->base_ci;
  ci->next = ci->previous = NULL;
  ci->callstatus = 0;
  ci->func = L1->top;
  setnilvalue(L1->top++);  /* 'function' entry for this 'ci' */
  ci->top = L1->top + LUA_MINSTACK;
  L1->ci = ci;
}

// lstate.h
#define BASIC_STACK_SIZE (2* LUA_MINSTACK)
/* extra stack space to handle TM calls and some other extras */
#define EXTRA_STACK   5

// lua.h
/* minimum Lua stack available to a C function */
#define LUA_MINSTACK    20
```

`BASIC_STACK_SIZE`表示初始数据栈大小，任意一次在lua_State上执行的C函数只保证`LUA_MINSTACK`大小空间可用
(TODO: WHY NOT？BASIC_STACK_SIZE == LUA_MINSTACK + EXTRA_STACK)。
当数据栈空间不足时，使用下面函数扩充：

``` c
// ldo.c
void luaD_reallocstack (lua_State *L, int newsize) {
  TValue *oldstack = L->stack;
  int lim = L->stacksize;
  lua_assert(newsize <= LUAI_MAXSTACK || newsize == ERRORSTACKSIZE);
  lua_assert(L->stack_last - L->stack == L->stacksize - EXTRA_STACK);
  luaM_reallocvector(L, L->stack, L->stacksize, newsize, TValue);
  for (; lim < newsize; lim++)
    setnilvalue(L->stack + lim); /* erase new segment */
  L->stacksize = newsize;
  L->stack_last = L->stack + newsize - EXTRA_STACK;
  correctstack(L, oldstack);
}

// lua.conf
#define LUAI_MAXSTACK           1000000
```










### 调用栈




