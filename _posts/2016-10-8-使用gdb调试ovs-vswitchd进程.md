---
layout: post
title: "使用gdb调试ovs-vswitchd进程"
comments: true
description: "Open vSwitch调试"
keywords: "Open vSwitch"
---

在修改完Open vSwitch的源代码后，有必要进行相关的测试以验证其实现是否正确。测试时经常会发现修改完的代码会存在bug，导致程序行为不正常，甚至直接出错退出。然而源程序规模巨大，为定位程序出错位置，可以使用gdb调试工具跟踪程序执行情况。

gdb既可以调试可执行程序，也可以调试以后台进程形式存在的服务程序。ovs-vswitchd显然属于后者。下面分享一下我调试这个程序的过程。

首先要获得进程的pid，这一步可以使用下述命令实现：

```bash
ps -ef | grep ovs
```

这个命令会打印出所有进程名中包含ovs字段的进程信息。从中获取到ovs-vswitchd的pid号。

然后启动gdb：

```bash
gdb ovs-vswitchd <pid>
```

即可进入gdb调试界面。

首先令该服务程序继续运行：

```bash
(gdb) r
```

会继续执行ovs-vswitchd程序。直到程序出错。

程序出错时，gdb会输出一些简单的信息，如下图所示：
![错误信息](http://ww4.sinaimg.cn/mw1024/6a964b69jw1f8l47dbjzkj20oc0290sw.jpg)

这些信息过于模糊，仍然不能判断程序出错的位置。

此时需要显示程序的调用栈，判断具体位置。显示调用栈的命令是：

```bash
(gdb) bt
```

输出结果如下图所示：
![调用堆栈](http://ww2.sinaimg.cn/mw1024/6a964b69jw1f8l48rajbfj21120ct42f.jpg)

从中即可准确定位代码出错的行数，从而了解具体的bug情况。