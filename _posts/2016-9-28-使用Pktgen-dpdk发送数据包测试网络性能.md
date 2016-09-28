---
layout: post
title: "使用Pktgen-dpdk发送数据包测试网络性能"
comments: true
description: "DPDK APP"
keywords: "DPDK"
---

Pktgen-dpdk是基于DPDK的高速包生成工具，用于测试高速链路连接性能。由于普通的网络测试工具netperf发包速率比较低，单个实例的netperf根本无法占满40GbE的带宽。对于高带宽以太网的测试，使用的工具大多基于DPDK框架，以达到高性能。


## Pktgen-dpdk的获取
Pktgen-dpdk在[DPDK网站](http://dpdk.org/download)上即可获得。它的git仓库地址是：

```
http://dpdk.org/git/apps/pktgen-dpdk
```

使用git clone获取到本地即可。

## Pktgen-dpdk的编译
Pktgen-dpdk依赖于libpcap软件包，编译之前先安装依赖包：

```bash
yum install libpcap-devel
```

然后将DPDK源代码目录设定为环境变量：

```bash
export RTE_SDK=<DPDKinstallDir>
export RTE_TARGET=x86_64-native-linuxapp-gcc
```

在上一篇post中，介绍了如何编译DPDK。假定已经成功编译DPDK之后，再编译Pktgen-dpdk:

```bash
make
```

编译如果没有错误，则会在app/文件夹下面生成多层目录，最深处有Pktgen-dpdk的可执行文件。

DPDK还需要Huge Page的支持，Pktgen-dpdk提供了setup.sh脚本，运行这个脚本即可。运行时可能报出很多错误，但是并不影响其功能。

```
./setup.sh
```

## Pktgen-dpdk的运行
在Pktgen-dpdk源代码的根目录下，有一个doit.sh的脚本，这个脚本需要修改后才可使用。需要掌握一定的shell的语法知识。

依照作者提供的doit.sh的样本，我自行添加的代码段如下：

```
if [ $name == "localhost.localdomain" ]; then
        dpdk_opts="-l 1-17 -n 4 --proc-type auto --log-level 8 --socket-mem 256,256 --file-prefix pg"
        pktgen_opts="-T -p"
        port_map="-m [2-5:6-9].0 -m [10-13:14-17].1"
        bl_common="-b 01:00.0 -b 01:00.1 -b 1:00.2 -b 1:00.3"
        black_list="${bl_common}"
        load_file="-f themes/white-black.theme"

        echo ${cmd} ${dpdk_opts} ${black_list} -- ${pktgen_opts} ${port_map} ${load_file}
        sudo ${cmd} ${dpdk_opts} ${black_list} -- ${pktgen_opts} ${port_map} ${load_file}

        # Restore the screen and keyboard to a sane state
        stty sane
fi
```

其中最重要的是获取到硬件相关的pci地址，将不使用的网卡添加到黑名单。使用下列命令查看以太网卡的pci地址：

```bash
lspci | grep Ethernet
```

其输出为：

```bash
01:00.0 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
01:00.1 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
01:00.2 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
01:00.3 Ethernet controller: Intel Corporation I350 Gigabit Network Connection (rev 01)
8b:00.0 Ethernet controller: Mellanox Technologies MT27500 Family [ConnectX-3]
```

若要仅使用Mellanox网卡，则需要将其他网卡添加到黑名单中。故而我的代码段中：

```bash
bl_common="-b 01:00.0 -b 01:00.1 -b 1:00.2 -b 1:00.3"
black_list="${bl_common}"
```

添加的均是其他网卡的pci地址。

此外，huge page的页数要充足，2M的huge page数量要足够多，否则运行会报出mbuf相关的错误。

修改完后，如果设置都正常，则会看到下述界面：
![界面](http://ww4.sinaimg.cn/mw690/6a964b69jw1f89b1gxdvuj20n20ke0tz.jpg)


