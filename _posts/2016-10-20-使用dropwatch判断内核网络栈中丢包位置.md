---
layout: post
title: "使用dropwatch判断内核网络栈中丢包位置"
comments: true
description: "网络栈"
keywords: "网络栈"
---

通常情况下，服务器之间使用基于以太网的TCP/IP协议进行互联。在这样的环境下，可能出现丢包、无法连通等等问题。对于丢包情况，首先要确定丢包发生的位置，是发生在源端的TCP/IP协议栈中，还是发生在交换机中；对于无法连通的情况，要确定故障发生在哪一层。

为了解决上述问题，使用dropwatch是一种可能的解决方法。

dropwatch使用Kernel Tracepoint API，具体来说，就是各协议释放socket缓冲区时（kfree_skb函数）收集信息。可应用于IP、ARP、ICMP、IGMP、UDP等等各种协议的数据包。

在CentOS 7.2中，仅需使用下述yum命令安装：

```bash
yum install dropwatch
```

在bash中，使用dropwatch命令启动程序，最好加上"-l kas"参数，这样可以显示信息时包含函数名：

```bash
dropwatch -l kas
```

启动后，需要输入start指令开始监测：

```bash
dropwatch> start
```

启动的界面如图：
![](http://ww3.sinaimg.cn/mw1024/6a964b69jw1f8yubhu4mdj20a702yjrd.jpg)

当链路无法连通的情况出现时，dropwatch的典型输出为：
![](http://ww1.sinaimg.cn/large/6a964b69jw1f8yubwubfcj20ec05nq5l.jpg)

可以推测出，是ARP表中没有目标端的MAC地址，导致无法连通，可以将问题定位到L2层中。此时就应当检查网络线缆的连接，排查故障。