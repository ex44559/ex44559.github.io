---
layout: post
title: "解决OVS-DPDK启动时出现Insufficient memory to create memory pool for netdev dpdk0问题"
comments: true
description: "Open vSwitch"
keywords: "Open vSwitch DPDK"
---

在部署OVS-DPDK的过程中，出现```"ovs|00014|timeval|WARN|Unreasonably long 6567ms poll interval (232msuser, 5958ms system)"```错误提示。于是我在OVS的Mail List中讨论了这个问题。通过进一步配置，我发现是内存分配的问题。

这次讨论的链接：http://openvswitch.org/pipermail/discuss/2016-October/023026.html

## 问题描述

在使用Intel 82599ES万兆网卡的环境中，部署DPDK 16.07和OVS 2.6.0，出现"Unreasonably long 6567ms poll interval (232ms user,
5958ms system)"字样，具体log如下：

```bash
Oct 26 05:27:41 localhost.localdomain ovsdb-server[13795]:
ovs|00001|ovsdb_server|INFO|ovsdb-server (Open vSwitch) 2.6.0
Oct 26 05:27:48 localhost.localdomain ovs-vsctl[13796]:
ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait set Open_vSwitch .
other_config:dpdk-init=true
Oct 26 05:27:51 localhost.localdomain ovsdb-server[13795]:
ovs|00002|memory|INFO|2300 kB peak resident set size after 10.0 seconds
Oct 26 05:27:51 localhost.localdomain ovsdb-server[13795]:
ovs|00003|memory|INFO|cells:16 monitors:0
Oct 26 05:27:52 localhost.localdomain ovs-vsctl[13797]:
ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait set Open_vSwitch .
other_config:dpdk-socket-mem=256,256,256,256
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13798]:
ovs|00001|socket_util|ERR|localhost: port must be specified
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13798]:
ovs|00002|vlog|INFO|opened log file /usr/local/var/log/
openvswitch/ovs-vsctl.log
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00003|ovs_numa|INFO|Discovered 20 CPU cores on NUMA node 0
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00004|ovs_numa|INFO|Discovered 20 CPU cores on NUMA node 1
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00005|ovs_numa|INFO|Discovered 20 CPU cores on NUMA node 2
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00006|ovs_numa|INFO|Discovered 20 CPU cores on NUMA node 3
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00007|ovs_numa|INFO|Discovered 4 NUMA nodes and 80 CPU cores
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00008|reconnect|INFO|unix:/usr/local/var/run/openvswitch/db.sock:
connecting...
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00009|reconnect|INFO|unix:/usr/local/var/run/openvswitch/db.sock:
connected
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00010|dpdk|INFO|DPDK Enabled, initializing
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00011|dpdk|INFO|No vhost-sock-dir provided - defaulting to
/usr/local/var/run/openvswitch
Oct 26 05:28:00 localhost.localdomain ovs-vswitchd[13799]:
ovs|00012|dpdk|INFO|EAL ARGS: ovs-vswitchd --socket-mem 256,256,256,256 -c
0x00000001
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: PMD:
bnxt_rte_pmd_init() called for (null)
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:01:00.0 on NUMA socket 0
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:1521 rte_igb_pmd
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:01:00.1 on NUMA socket 0
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:1521 rte_igb_pmd
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:01:00.2 on NUMA socket 0
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:1521 rte_igb_pmd
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:01:00.3 on NUMA socket 0
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:1521 rte_igb_pmd
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:8b:00.0 on NUMA socket 2
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:10fb rte_ixgbe_pmd
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL:   using
IOMMU type 1 (Type 1)
Oct 26 05:28:06 localhost.localdomain ovs-vswitchd[13799]: EAL: Ignore
mapping IO port bar(2) addr: ffc1
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]: EAL: PCI device
0000:8b:00.1 on NUMA socket 2
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]: EAL:   probe
driver: 8086:10fb rte_ixgbe_pmd
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00013|bridge|INFO|ovs-vswitchd (Open vSwitch) 2.6.0
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00014|timeval|WARN|Unreasonably long 6567ms poll interval (232ms user,
5958ms system)
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00015|timeval|WARN|faults: 9659 minor, 10 major
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00016|timeval|WARN|disk: 2384 reads, 0 writes
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00017|timeval|WARN|context switches: 19 voluntary, 62 involuntary
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00018|coverage|INFO|Event coverage, avg rate over last: 5 seconds, last
minute, last hour,  hash=ec4c58ab:
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00019|coverage|INFO|bridge_reconfigure         0.0/sec     0.000/sec
     0.0000/sec   total: 1
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00020|coverage|INFO|cmap_expand                0.0/sec     0.000/sec
     0.0000/sec   total: 9
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00021|coverage|INFO|miniflow_malloc            0.0/sec     0.000/sec
     0.0000/sec   total: 11
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00022|coverage|INFO|hmap_pathological          0.0/sec     0.000/sec
     0.0000/sec   total: 3
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00023|coverage|INFO|hmap_expand                0.0/sec     0.000/sec
     0.0000/sec   total: 646
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00024|coverage|INFO|txn_unchanged              0.0/sec     0.000/sec
     0.0000/sec   total: 3
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00025|coverage|INFO|poll_create_node           0.0/sec     0.000/sec
     0.0000/sec   total: 42
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00026|coverage|INFO|seq_change                 0.0/sec     0.000/sec
     0.0000/sec   total: 49
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00027|coverage|INFO|pstream_open               0.0/sec     0.000/sec
     0.0000/sec   total: 1
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00028|coverage|INFO|stream_open                0.0/sec     0.000/sec
     0.0000/sec   total: 1
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00029|coverage|INFO|util_xalloc                0.0/sec     0.000/sec
     0.0000/sec   total: 11218
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00030|coverage|INFO|netdev_get_hwaddr          0.0/sec     0.000/sec
     0.0000/sec   total: 2
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00031|coverage|INFO|netlink_received           0.0/sec     0.000/sec
     0.0000/sec   total: 3
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00032|coverage|INFO|netlink_sent               0.0/sec     0.000/sec
     0.0000/sec   total: 1
Oct 26 05:28:07 localhost.localdomain ovs-vswitchd[13799]:
ovs|00033|coverage|INFO|89 events never hit
```

添加net-dev类型的bridge和dpdk类型的dpdk0端口后，得到了新的错误信息：

```bash
2016-10-26T13:50:29Z|00022|dpdk|ERR|Insufficient memory to create memory pool for netdev dpdk0, with MTU 1500 on socket 2
2016-10-26T13:50:29Z|00023|bridge|WARN|could not open network device dpdk0 (Cannot allocate memory)
```

可以发现这是内存分配导致的问题。然而我在4个NUMA节点中都分配了1024个2M huge page，不应当出现这种内存不足的问题。

于是我搜索了这个错误，信息，发现它来自于2016年8月的一个patch，为Jumbo Frames提供了支持。可能这个功能开发得尚不完善。

## workaround 

切换到Open vSwitch 2.5.1和dpdk-2.2.0，则不会出现这个问题。

stable version 出现的问题确实会少一些。