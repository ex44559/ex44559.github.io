---
layout: post
title: "从windows下ps/2键盘失灵到windows的driver filter机制"
comments: true
description: "Open vSwitch"
keywords: "Open vSwitch DPDK"
---

就在刚刚，我在实验室使用的电脑突然键盘不能用了，由于实验室给我配的都是前人留下的遗产，所以出问题也是很正常的。

这个键盘还在使用古老的紫色ps/2接口，我反复插拔，都不能使它正常工作。但幸运的是，这台电脑有两个系统，开机引导时加载grub2界面时，键盘是可以正常工作的。那么就可以确定，硬件都没问题，是Windows不能识别键盘了。

于是只好开启屏幕键盘暂时替代了，我使用Google没有搜索到合理解决方法。最后倒是一篇百度经验解决了问题。链接[在此](http://jingyan.baidu.com/article/e5c39bf5dca8fc39d660335e.html)。

## 解决方法

- 定位到HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E96B-E325-11CE-BFC1-08002BE10318}，删除UpperFilters项
- 卸载键盘设备，重新启动。
- 定位到HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\Class\{4D36E96B-E325-11CE-BFC1-08002BE10318}，新建添加字符串UpperFilters项，修改内容为kbdclass
- 再次卸载键盘设备，重新启动,系统提示发现并成功安装了ps/2键盘驱动

## Driver Filter

根据Windows设备管理器显示的PS/2设备驱动界面，看到可用的驱动有两个。如下图所示。

![](http://ww2.sinaimg.cn/mw1024/6a964b69jw1f9713pt81wj20bc0fu0sw.jpg)

故障时，事件信息如下图所示。

![](http://ww4.sinaimg.cn/mw1024/6a964b69jw1f97155cjt6j20bu03pglm.jpg)

这说明Windows不能识别键盘，是因为i8042prt不能正确启动这个设备导致。

而上面的解决办法正是让windows使用kbdclass驱动键盘。

这种在两个可用驱动中设置选择策略的组件，就是Driver Filter。正如注册表中的名字所示，令上层组件选择kbdclass驱动键盘，而不是i8042prt。

Driver Filter机制在科研中的体现，就是这篇FAST'16的文章(sRoute: Treating the Storage Stack Like a Network)，基于Windows的Driver Filter特性为不同的I/O选择各自的路径。由于文章中没有给出实现方法，我大胆推测可能就是设置了不同的注册表键值来控制各种I/O使用的路径。:)