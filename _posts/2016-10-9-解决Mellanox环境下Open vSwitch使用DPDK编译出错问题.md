---
layout: post
title: "解决Mellanox环境下Open vSwitch结合DPDK编译出错问题"
comments: true
description: "Open vSwitch调试"
keywords: "Open vSwitch"
---

Open vSwitch使用DPDK作数据通路可以有效提升转发性能。然而DPDK与Open vSwitch分属不同的组织开发，新版本之间难免存在兼容性问题。例如，**Open vSwitch 2.5系列不支持DPDK 16.07**。为了解决这个问题，我只好使用最新发布的Open vSwitch 2.6.0版本，这个版本进一步完善了OVN的实现，可以看出Network Virtualization确实是一种趋势。

如果直接使用默认的编译方法去编译OVS：

```bash
./configure --with-dpdk=$DPDK_BUILD
make && make install
```

会出现下述连接错误：

```bash
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `rte_mlx4_pmd_init':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5939: undefined reference to `ibv_fork_init'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_mp2mr':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1316: undefined reference to `ibv_reg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `txq_mp2mr':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1384: undefined reference to `ibv_dereg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `priv_dev_link_status_handler':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5412: undefined reference to `ibv_ack_async_event'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5403: undefined reference to `ibv_get_async_event'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `txq_free_elts':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1074: undefined reference to `ibv_dereg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `txq_cleanup':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1134: undefined reference to `ibv_destroy_qp'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1136: undefined reference to `ibv_destroy_cq'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1152: undefined reference to `ibv_dereg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_pci_devinit':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5570: undefined reference to `ibv_get_device_list'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5605: undefined reference to `ibv_free_device_list'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5600: undefined reference to `ibv_open_device'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5620: undefined reference to `ibv_query_device'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5887: undefined reference to `ibv_close_device'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5889: undefined reference to `ibv_free_device_list'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5646: undefined reference to `ibv_open_device'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `___ibv_query_port':
/usr/include/infiniband/verbs.h:1160: undefined reference to `ibv_query_port'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_pci_devinit':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5664: undefined reference to `ibv_port_state_str'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5669: undefined reference to `ibv_alloc_pd'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5864: undefined reference to `ibv_dealloc_pd'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5866: undefined reference to `ibv_close_device'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5804: undefined reference to `ibv_get_device_name'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:5605: undefined reference to `ibv_free_device_list'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `txq_alloc_elts':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:1005: undefined reference to `ibv_reg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_rx_burst_sp':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:3095: undefined reference to `ibv_wc_status_str'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_rx_burst':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:3313: undefined reference to `ibv_wc_status_str'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `rxq_cleanup':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:2925: undefined reference to `ibv_destroy_qp'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:2928: undefined reference to `ibv_destroy_cq'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:2941: undefined reference to `ibv_dereg_mr'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `mlx4_dev_close':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:4321: undefined reference to `ibv_dealloc_pd'
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:4322: undefined reference to `ibv_close_device'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `ibv_exp_create_qp':
/usr/include/infiniband/verbs_exp.h:1942: undefined reference to `ibv_create_qp'
/root/code/dpdk-16.07/x86_64-native-linuxapp-gcc/lib/librte_pmd_mlx4.a(mlx4.o): In function `rxq_rehash':
/root/code/dpdk-16.07/drivers/net/mlx4/mlx4.c:3641: undefined reference to `ibv_resize_cq'
collect2: error: ld returned 1 exit status
```

根据输出的提示可以看出，这主要是链接库的问题，在链接生成ovs-vswitchd时找不到引用的一些变量和函数。

这也同时说明了，使用一款小众硬件需要更多的时间配置相关的环境。

## 解决编译错误

根据反复调试，我发现链接生成ovs-vswitchd文件需要使用两个库：

- libibverbs
- libnl

因此，修改```vswitchd/automake.mk```文件，将原来的链接设置：

```bash
vswitchd_ovs_vswitchd_LDADD = \
    ofproto/libofproto.la \
    lib/libsflow.la \
    lib/libopenvswitch.la 
```

修改为下述形式：

```bash
vswitchd_ovs_vswitchd_LDADD = \
    ofproto/libofproto.la \
    lib/libsflow.la \
    lib/libopenvswitch.la \
    /usr/lib64/libibverbs.a \
    /usr/lib64/libnl.so
```

在链接ovs-vswitchd时就会使用libibverbs、libnl库文件，可以编译成功。