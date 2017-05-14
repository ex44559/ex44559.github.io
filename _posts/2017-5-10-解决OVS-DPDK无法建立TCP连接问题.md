---
layout: post
title: "解决OVS-DPDK无法建立TCP连接的问题"
comments: true
description: "Open vSwitch"
keywords: "Open vSwitch"
---

# 问题描述

当Open vSwitch使用DPDK数据通路时，从虚拟机发送数据包到同一局域网的物理机时，接收方在收到TCP连接的SYN包时，发现checksum error，SYN包被丢弃。TCP连接的握手过程无法完成。

# 问题定位

当虚拟机使用virtio_net作为前端驱动时，如果开启tx checksum offload功能，就会出现该问题。

此bug亦见于[launchpad bugs 页面](https://bugs.launchpad.net/ubuntu/+source/openvswitch/+bug/1629053)。

# 解决方式

Open vSwitch LTS 版本尚未见修复此问题。

## workaround

在虚拟机中关闭tx checksum offload 功能。

使用下述命令即可：

```bash
ethtool --offload eth0  tx off
```

