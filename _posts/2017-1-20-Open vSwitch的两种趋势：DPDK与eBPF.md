---
layout: post
title: "Open vSwitch的两种趋势：DPDK与eBPF"
comments: true
description: "Open vSwitch"
keywords: "Open vSwitch"
---

### 内核态还是用户态？

为了改善虚拟化环境的I/O性能，减少I/O栈的处理开销势在必行。可是虚拟化环境的I/O路径太长，既有存在于用户态的部分，也有存在于内核态的部分。为了减少用户态与内核态之间的切换，自然有人主张向用户态迁移更多组件，一个体现就是DPDK；自然也有人主张优化原有的内核路径，一个体现就是eBPF。

eBPF全称是Extended Berkeley Packet Filter，这个组件在1997年第一引入到2.1.75版本的内核中。在3.15版本之后的内核中，eBPF功能更加完善。源程序可用C语言书写，使用LLVM编译生成JIT模式的机器码，被内核执行；而且可以被注入到内核路径的socket、TC等部分，扩展I/O路径的功能。

![](http://wx2.sinaimg.cn/mw690/6a964b69ly1fd0oxcxj3bj20c80csdgk.jpg)

在使用Open vSwitch的环境中，既可以使用DPDK代替Open vSwitch的内核组件，让数据包处理在用户态进行；也可以使用eBPF优化（甚至取代）Open vSwitch内核组件，令内核态数据路径变得更加灵活可扩展。

# eBPF:可拓展的内核路径

在Open vSwitch 2016 Conference上，就有一个Presentation是提出了eBPF取代OVS内核路径的想法。如下图所示：

![](http://wx3.sinaimg.cn/mw690/6a964b69ly1fd0oxy41hdj20zk0jujt6.jpg)

如果使用eBPF实现OVS的内核路径，则拓展性更好；由于eBPF的JIT特性，使得这个组件与内核版本解耦；当然，可能存在着性能损失。

# DPDK:向用户态迁移的网络栈

DPDK提供了一组库文件和Poll Mode的网卡驱动；使用DPDK替代原有的OVS内核数据路径，则可以令数据包的处理更多地处于用户态之中。

![](http://wx1.sinaimg.cn/mw690/6a964b69ly1fd0p11y3j1j20uj0i1n7k.jpg)

# 总结

 eBPF和DPDK却走在两条不同的道路之上，未来的发展难以轻易预测。合理的策略就是风险对冲，二者都了解一下，不至于日后束手无措。
