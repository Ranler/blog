layout: post
title: Java IncompatibleClassChangeError 异常一则
date: 2013-3-27
categories: JVM
---

涉及两个模块：

- A模块, 提供了A.jar和源码
- B模块, 提供了B.jar和dummy源码

以上提供的jar文件不知什么环境下打的什么版本的包。

首先只部署A.jar文件到Linux服务器上，测试运行，发现抛出`NoClassDefFoundError`异常。
查看A模块源码，发现A模块依赖B模块用于license认证，缺少的class指向B模块下的类。

于是部署B.jar文件到Linux服务器上，测试运行，发现B.jar抛出dll文件未找到的异常。
经过反编译B.jar，发现B.jar是Windows下的真实license认证程序，需要通过JNI去调用dll，
在Linux平台下肯定执行不正确。

下一步把A模块的源码和B模块的dummy源码合并，打成AB.jar包。
部署AB.jar到Linux服务器上，测试正常。

然后接下来还是准备把A模块和B模块单独打包。
于是把dummy源码打成newB.jar，并和A.jar一起部署到服务器上。
测试运行，抛出`IncompatibleClassChangeError`。

寻找原因，发现反编译的B.jar中，之前`NoClassDefFoundError`的class License
在B.jar中是Interface类型，而在dummy源码中是Class类型。
而之前估计A模块的源码是通过依赖B.jar来打成包的，
在A.jar中理所当然认为License是Interface类型。
而部署的newB.jar中License是Class类型，因此抛出了`IncompatibleClassChangeError`异常。
随后调整A模块依赖newB.jar来打成newA.jar包，部署测试，运行成功。

因此此次`IncompatibleClassChangeError`异常是同一个符号名类型不一致的原因。
有个疑问是什么情况下是compatibleClassChange？



