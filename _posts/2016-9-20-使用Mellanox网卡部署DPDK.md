---
layout: post
title: "使用Mellanox网卡部署DPDK"
comments: true
description: "DPDK部署"
keywords: "DPDK"
---

实验室的服务器上使用的是Mellanox ConnetX-3系列的40Gbps 以太网网卡。由于网卡支持的带宽很高，一般的网络性能测试工具诸如netperf等由于发包速度不够快，往往跑不满带宽，无法测试链路拥塞的情况。

新出现的测试工具，诸如pktgen-dpdk、MoonGen等，都是依靠DPDK框架实现高速发包以测试高速链路的性能。因此，部署DPDK成为一个不可避免的选择。

DPDK由Intel开发，在用户态处理数据包，采用轮询方式取代传统的中断方式从网卡接收数据。DPDK与Open vSwitch相结合，可以完成全用户空间的数据处理，避免了在用户态和内核态传递数据的开销，性能不错。

根据DPDK的官方文档，Mellanox的网卡的Poll Mode Driver已经集成在DPDK中。需要更改设置文件使其开启；此外，还需要重新部署一些库文件以完成DPDK的编译。
具体流程可以参见DPDK的[文档](http://dpdk.org/doc/guides/linux_gsg/build_dpdk.html)。

## 开启DPDK的Mellanox Driver
DPDK的配置文件保存在config/目录中，配置项很多，需要根据自身的环境应用合适的配置文件。一般的服务器环境都是x86_64架构，所以通常的选择是config_x86_64-native-linuxapp-gcc。

在config/common_base文件中，寻找到CONFIG_RTE_LIBRTE_MLX4_PMD字段（如果使用mlx5驱动，应当使用包含MLX5的字段），并将其由n改为y，即可支持Mellanox的网卡。依照DPDK的文档安装流程开始编译。

## 解决DPDK编译及运行时报错
在CentOS 7.2环境下，修改完config中的设置后，编译mlx4模块时会出现error。

```bash
error:
 storage size of ‘params’ isn’t known
     struct ibv_exp_release_intf_params params;

 ……[省略n行]
```

根据DPDK的邮件列表中的[线索](http://dpdk.org/ml/archives/dev/2015-November/027797.html),系统自带的libibverbs软件包不包含一些拓展的数据结构，需要在Mellanox网站上下载Mellanox OFED的[软件包](http://www.mellanox.com/page/mlnx_ofed_matrix?mtag=linux_sw_drivers)。

根据我的实践，直接选择最新版本的镜像文件，不用考虑固件版本匹配的问题，因为OFED软件包中包含了固件升级器，可以自动将固件升级到更高版本。

采用下列下载iso镜像
```bash
wget http://www.mellanox.com/downloads/ofed/MLNX_OFED-3.3-1.0.4.0/MLNX_OFED_LINUX-3.3-1.0.4.0-rhel7.2-x86_64.iso
```

将其挂载并安装
```bash
mount -o ro,loop MLNX_OFED_LINUX-3.3-1.0.4.0-rhel7.2-x86_64.iso /mnt

/mnt/mlnxofedinstall 
```

安装完成后，需要重新启动。

重新启动后，加载相应的内核模块：
```bash
modprobe -a ib_uverbs mlx4_en mlx4_core mlx4_ib
```

检查是否可以连接到kernel verbs：
```bash
ls -d /sys/class/net/*/device/infiniband_verbs/uverbs* | cut -d / -f 5
```

典型的输出为：
```bash
ens5d1
ens5
```

挂载huge page：
```bash
mount -t hugetlbfs nodev /mnt/huge
```

再运行testpmd，确认Poll Mode Driver可用：
```bash
testpmd -c 0xff00 -n 4 -w 0000:8b:00.0 -- --rxq=2 --txq=2 -i
```

正常的输出为：
```bash
[...]
EAL: PCI device 0000:83:00.0 on NUMA socket 1
EAL:   probe driver: 15b3:1007 librte_pmd_mlx4
PMD: librte_pmd_mlx4: PCI information matches, using device "mlx4_0" (VF: false)
PMD: librte_pmd_mlx4: 2 port(s) detected
PMD: librte_pmd_mlx4: port 1 MAC address is 00:02:c9:b5:b7:50
PMD: librte_pmd_mlx4: port 2 MAC address is 00:02:c9:b5:b7:51
EAL: PCI device 0000:84:00.0 on NUMA socket 1
EAL:   probe driver: 15b3:1007 librte_pmd_mlx4
PMD: librte_pmd_mlx4: PCI information matches, using device "mlx4_1" (VF: false)
PMD: librte_pmd_mlx4: 2 port(s) detected
PMD: librte_pmd_mlx4: port 1 MAC address is 00:02:c9:b5:ba:b0
PMD: librte_pmd_mlx4: port 2 MAC address is 00:02:c9:b5:ba:b1
Interactive-mode selected
Configuring Port 0 (socket 0)
PMD: librte_pmd_mlx4: 0x867d60: TX queues number update: 0 -> 2
PMD: librte_pmd_mlx4: 0x867d60: RX queues number update: 0 -> 2
Port 0: 00:02:C9:B5:B7:50
Configuring Port 1 (socket 0)
PMD: librte_pmd_mlx4: 0x867da0: TX queues number update: 0 -> 2
PMD: librte_pmd_mlx4: 0x867da0: RX queues number update: 0 -> 2
Port 1: 00:02:C9:B5:B7:51
Configuring Port 2 (socket 0)
PMD: librte_pmd_mlx4: 0x867de0: TX queues number update: 0 -> 2
PMD: librte_pmd_mlx4: 0x867de0: RX queues number update: 0 -> 2
Port 2: 00:02:C9:B5:BA:B0
Configuring Port 3 (socket 0)
PMD: librte_pmd_mlx4: 0x867e20: TX queues number update: 0 -> 2
PMD: librte_pmd_mlx4: 0x867e20: RX queues number update: 0 -> 2
Port 3: 00:02:C9:B5:BA:B1
Checking link statuses...
Port 0 Link Up - speed 10000 Mbps - full-duplex
Port 1 Link Up - speed 40000 Mbps - full-duplex
Port 2 Link Up - speed 10000 Mbps - full-duplex
Port 3 Link Up - speed 40000 Mbps - full-duplex
Done
testpmd>
```

看到有“Done”字样，说明testpmd已经成功了，证明DPDK可以运行了。