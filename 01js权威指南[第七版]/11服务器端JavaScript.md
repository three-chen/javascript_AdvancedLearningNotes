# 11服务器端JavaScript

## Rhino

### 介绍

Rhino是一种用Java编写的JavaScript解释器，，其设计目标是借助于强大的Java平台API实现轻松编写JavaScript程序。Rhino能自动完成JavaScript原生类型到Java原生类型之间的相互转换。

这里给出github地址：

[mozilla/rhino: Rhino is an open-source implementation of JavaScript written entirely in Java (github.com)](https://github.com/mozilla/rhino)

## Node

### 介绍

Node是基于C++的高速JavaScript解释器，绑定了用于进程、文件和网络套接字等底层Unix API，还绑定了HTTP客户端和服务器API。除了一些专门命名的同步方法外，Node的绑定都是异步的，且Node程序默认绝不阻塞，这意味着它们通常具备强大的可伸缩能力并能有效地处理高负荷。由于API是异步的，因此Node依赖事件处理程序，其通常使用嵌套函数和闭包来实现。
具体文档看http://nodejs.org/api/，中文文档看http://cnodejs.org/cman，下载地址：https://nodejs.org

