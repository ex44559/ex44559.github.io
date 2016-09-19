---
layout: post
title: "解决Mellanox网卡部署DPDK出错问题"
comments: true
description: "DPDK部署"
keywords: "DPDK"
---

实验室的服务器上使用的是Mellanox ConnetX-3系列的40Gbps 以太网网卡。由于网卡支持的带宽很高，一般的网络性能测试工具诸如netperf等由于发包速度不够快，往往跑不满带宽，无法测试链路拥塞的情况。

新出现的测试工具，诸如pktgen-dpdk、MoonGen等，都是依靠DPDK框架实现高速发包以测试高速链路的性能。因此，部署DPDK成为一个不可避免的选择。

DPDK由Intel开发，在用户态处理数据包，采用轮询方式取代传统的中断方式从网卡接收数据。DPDK与Open vSwitch相结合，可以完成全用户空间的数据处理，避免了在用户态和内核态传递数据的开销，性能不错。

根据DPDK的官方文档，Mellanox的网卡的Poll Mode Driver已经集成在DPDK中。需要更改设置文件使其开启；此外，还需要重新部署一些库文件以完成DPDK的编译。
具体流程可以参见DPDK的[文档](http://dpdk.org/doc/guides/linux_gsg/build_dpdk.html)。

##开启DPDK的Mellanox Driver

DPDK的配置文件保存在config/目录中，配置项很多，需要根据自身的环境应用合适的配置文件。一般的服务器环境都是x86_64架构，所以通常的选择是config_x86_64-native-linuxapp-gcc。

在config/common_base文件中，寻找到CONFIG_RTE_LIBRTE_MLX4_PMD字段（如果使用mlx5驱动，应当使用包含MLX5的字段），并将其由n改为y，即可支持Mellanox的网卡。依照DPDK的文档安装流程开始编译。

##解决DPDK编译时报错

在CentOS 7.2环境下，修改完config中的设置后，编译mlx4模块时会出现error。

```bash
error:
 storage size of ‘params’ isn’t known
     struct ibv_exp_release_intf_params params;

 ……[省略n行]
```

根据DPDK的邮件列表中的[线索](http://dpdk.org/ml/archives/dev/2015-November/027797.html),系统自带的libibverbs软件包不包含一些拓展的数据结构，需要在Mellanox网站上下载Mellanox OFED的[软件包](http://www.mellanox.com/page/mlnx_ofed_matrix?mtag=linux_sw_drivers)。根据我的实践，应当下载较新的版本，而不是完全跟固件版本匹配的版本。

然后卸载系统自带的包，并安装下载的软件包。这个下载的压缩包中包含许多RPM，只需要安装带有libibverbs的rpm，不需要全部安装。如果已经装有CentOS版本的libibverbs，在用yum卸载这个包时，由于依赖关系会直接卸载掉**qemu-kvm**。所以卸载完libibverbs后，不仅需要用rpm命令安装下载的软件包，还需要重新编译安装qemu-kvm。注意，还需要保证除了下载的软件包安装的infiniband/verbs_exp.h文件外，在系统的引用目录中不应当存在其他版本的infiniband/verbs_exp.h文件。

消除编译错误之后，就可以愉快地make和make install啦。