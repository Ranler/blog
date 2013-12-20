layout: post
title: Linux Search Commands
date: 2013-8-21
categories: Shell
---

- which
- whereis
- locate
- find

### which

which在$PATH路径中查找可执行文件。

### whereis

whereis从以下路径中查找指定类型文件：

- $PATH或-B指定路径
- /{bin,sbin,etc}
- /usr/{lib,bin,old,new,local,games,include,etc,src,man,sbin,X386,TeX,g++-include}
- /usr/local/{X386,TeX,X11,include,lib,man,etc,bin,games,emacs}

查找类型可以是binaries, manual sections, source and unusual entries.

### locate

locate在数据库中查找文件。
数据库文件路径在`/var/lib/mlocate/mlocate.db`。
因此locate可以快速查找文件，但是不保证文件在文件系统中是否还存在。
数据中只是记录了历史信息。
可以使用`updatedb`更新数据库文件。

### find

find从指定路径中进行遍历查找，包含丰富的查找条件。
