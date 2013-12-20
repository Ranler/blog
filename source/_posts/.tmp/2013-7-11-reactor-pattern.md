---
layout: default
title: Reactor Pattern 反应堆模式
---

- Handle: 一个handle管理一个资源，如socket handle, file handle, timer handle。
- Event Handler: 包含一个回调的方法处理事件，与具体应用相关。
- Dispatcher 分发器/适配器: 在。。。上注册、删除、派发event handler。
- Demultiplexer 分拣器: 在一个handle集合上等待event发生(可使用select实现)，待事件发生后通过Dispatcher回调event handler。



- Notifer 通知器: 




Reactor模式难于调试。

stream模式
JavaNIO，Selector模式的Server和Comsumer模式的网络编程。

### Reference

[^WIKI]: http://en.wikipedia.org/wiki/Reactor_pattern

