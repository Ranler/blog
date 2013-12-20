---
layout: default
title: JVM Quick Reference
---

### class文件格式

class文件格式

```
﻿ClassFile {
    u4             magic;                                // 0xCAFEBABE
    u2             minor_version;                        // 小版本号
    u2             major_version;                        // 大版本号
    u2             constant_pool_count;                  // 常量池大小
    cp_info        constant_pool[constant_pool_count-1]; // 常量池数组
    u2             access_flags;                         // 访问控制标记
    u2             this_class;                           // 当前类信息
    u2             super_class;                          // 父类信息
    u2             interfaces_count;                     // 实现接口个数
    u2             interfaces[interfaces_count];         // 实现接口信息数组
    u2             fields_count;                         // 域个数
    field_info     fields[fields_count];                 // 域信息数组
    u2             methods_count;                        // 方法个数
    method_info    methods[methods_count];               // 方法信息数组
    u2             attributes_count;                     // 属性个数
    attribute_info attributes[attributes_count];         // 属性信息数组
}
```

属性可以看做杂项。


