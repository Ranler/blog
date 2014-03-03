title: Lua String Analysis
date: 2014-01-25 11:00:00
categories: Lua
---

在Lua中String类型是以引用方式保存，
每个string变量保存着`TString`类型的指针，并且`TString`类型也是`GCObject`类型，受GC管理。
`TString`类型定义如下：

``` c
// lobject.h
typedef union TString {
     L_Umaxalign dummy;
     struct {
          CommonHeader;
          lu_byte extra;  /* reserved words for short strings; "has hash" for longs */
          unsigned int hash;
          size_t len;      /* number of characters in string */
     } tsv;
}TString;

// llimits.h
typedef unsigned char lu_byte;
```

从Lua 5.2.1开始，字符串类型在实现时又被细分，分为：

- 短字符串: 长度不大于LUAI_MAXSHORTLEN（默认40）的字符串，包括所有元方法名和关键字字符串。相同字符串数据在Lua State中只保存一份(intern)。
- 长字符串：长度大于LUAI_MAXSHORTLEN，每个字符串数据独立保存。

它们的类型标签是；

``` c
// lobject.h 56
/* Variant tags for strings */
#define LUA_TSHRSTR     (LUA_TSTRING | (0 << 4))  /* short strings */
#define LUA_TLNGSTR     (LUA_TSTRING | (1 << 4))  /* long strings */

// lua.h 82
#define LUA_TSTRING             4
```

`GCObject`头部信息中使用1B大小的tt字段做为类型标签。
目前Lua只有9种类型，保留1B的低4bit完全足够表示类型标签，
那么高4bit用于每种类型特殊的mask，这里`TString`用以区别短字符串和长字符串。

Lua区分长字符串和短字符串的原因是防止Hash Dos。
并且长字符串数据也很难相同，而短字符串数据相同的概率比较高。

### CommonHeader

`TString`和其它`GCObject` 一样，拥有`CommonHeader`。
但需要注意，`CommonHeader` 中的`next` 域却和其它类型的单向链表意义不同。

它被挂接在 stringtable 这个 hash 表中。

### dummy字段

`dummy`字段用来使`GCObject`字节对齐，使得`GCObject`至少占一个word大小。

``` c
// llimits.h
/* type to ensure maximum alignment */
#if !defined(LUAI_USER_ALIGNMENT_T)
#define LUAI_USER_ALIGNMENT_T   union { double u; void *s; long l; }
#endif
   
typedef LUAI_USER_ALIGNMENT_T L_Umaxalign;
```

### extra字段

`extra`为0表示字符串没有被hash，大于0表示已经被hash。
并且在大于0时，如果字符串是短字符串，则保存着保留字或元方法名在stringtable中的下标。

### hash字段

Lua使用hash表(stringtable)保持string类型的引用。
这样的结果是string不可修改，改变string的操作只是新建了一个string而删除了原来的。
并且这也保证了系统中不会有值相同的 string 被创建两份。
每个string的hash值通过一个函数计算并保存在TSting的hash成员中。
这样保证了string能够快速索引和string之间能快速比较。
hash值计算函数并不读取整个string，而只读一部分，这样减少了操作长string时的性能损失。

从外部压入一个长字符串时，简单复制一遍字符串，并不立刻计算其hash值，
而是标记一下`extra`字段。直到需要对字符串做键匹配时，才惰性计算`hash`值，加快以后的键比较过程。

Lua并没有单独使用内存分配器分配字符串真正的数据，
而是把字符串数据存储在第一次出现的时的`TString`结构之后。

``` c
// lobject.h 422
/* get the actual string (array of bytes) from a TString */
#define getstr(ts)      cast(const char *, (ts) + 1)
```



### len字段

`len`字段保存着字符串的长度。
由于Lua并不以`\n`结尾来识别字符串的长度，所以需要记录长度，并且在与C语言交互时，自动添加`\0`兼容C库。


### stringtable

所有的短字符串保存在`global_State`的`strt`字段内。
`strt`字段是`stringtable`类型：

``` c
typedef struct stringtable {
   GCObject **hash;
   lu_int32 nuse;  /* number of elements */
   int size;
} stringtable;
```

### 字符串操作

#### TString创建

``` c
// lstring.c 98
/*
 ** new string (with explicit length)
 */
TString *luaS_newlstr (lua_State *L, const char *str, size_t l) {
   if (l <= LUAI_MAXSHORTLEN)  /* short string? */
     return internshrstr(L, str, l);                                     // 短字符串
   else {
     if (l + 1 > (MAX_SIZET - sizeof(TString))/sizeof(char))
       luaM_toobig(L);
     return createstrobj(L, str, l, LUA_TLNGSTR, G(L)->seed, NULL);      // 长字符串
  }
}
```

长字符串的创建：

``` c
/*
 ** creates a new string object
 */
static TString *createstrobj (lua_State *L, const char *str, size_t l,
                              int tag, unsigned int h, GCObject **list) {
   TString *ts;
   size_t totalsize;  /* total size of TString object */
   totalsize = sizeof(TString) + ((l + 1) * sizeof(char));  // TString后紧跟str字符串
   ts = &luaC_newobj(L, tag, totalsize, list, 0)->ts;   
   ts->tsv.len = l;
   ts->tsv.hash = h;                                        
   ts->tsv.extra = 0;                                       // 0表示hash尚未计算，以后再通过luaS_hash计算
   memcpy(ts+1, str, l*sizeof(char));
   ((char *)(ts+1))[l] = '\0';  /* ending 0 */              // Lua中字符串都以\0结束，兼容C库
   return ts;
}
```










参考 ：

- 风云 Lua源码欣赏