---
layout: post
title: "修改database schema为Open vSwitch添加新功能"
comments: true
description: "Open vSwitch优化"
keywords: "Open vSwitch"
---

Open vSwitch的组件主要包含datapath（内核空间）、ovs-vswitchd（用户空间）、ovsdb（用户空间）。其中datapath和ovs-vswitchd主要负责数据转发功能。ovsdb（Open vSwitch Data Base）作为数据库存储了Open vSwitch的配置信息。

通常情况下，可以使用Open vSwitch的API实现对OVS database的控制，在命令行下可以使用ovs-vsctl实现对ovs-vswitchd的控制。ovs-vsctl实际上连接到ovsdb-server进程，将要修改的内容发送到Open vSwitch的配置数据库，数据库再将改动传送到ovs-vswitchd进程，直到ovs-vswitchd进程完成改动，ovs-vsctl程序便可以确认改动完成并退出。

由于Open vSwitch的广泛应用，所以常常需要对其进行定制和优化。想要为Open vSwitch增加新功能时，首先要考虑的便是改进其配置数据库，使新功能可以用ovs-vsctl命令配置，该功能就具有了统一的控制接口。

配置数据库的表结构在vswitchd/vswitch.ovsschema中描述，当添加新功能时，要修改该文件。

## Open vSwitch database schema
Open vSwitch的schema指的就是vswitchd/vswitch.ovsschema，这个文件使用JSON语法定义了OVSDB数据库的内容。按照层次结构来分析，主要包括描述Bridge、Port、Interface的表（table）结构，在每个表中还包含栏（columns）结构，具体描述了诸如name、type、mac等属性信息。具体的描述可以参考Open vSwitch网站上的[ovs-vswitchd.conf.db](http://openvswitch.org/support/dist-docs-2.5/ovs-vswitchd.conf.db.5.html) Manual。

以vswitch.ovsschema中描述lacp协议的片段来分析，可以看出这是一个key-value的结构：key是3个枚举的字符串，对应的value为0或1。通过ovs-vswitchd通过分析这个key-value结构获取lacp的配置方式，将lacp设置成对应的模式。

```json
       "lacp": {
         "type": {"key": {"type": "string",
           "enum": ["set", ["active", "passive", "off"]]},
         "min": 0, "max": 1}},
```

当想要添加新功能时，按照JSON语法将新功能的配置项添加到vswitchd/vswitch.ovsschema文件中即可。

## schema文件修改后的编译

修改完vswitchd/vswitch.ovsschema文件后，需要重新编译Open vSwitch。有几处值得注意。首先是checksum的问题，其次是要更新系统内的数据库文件。

### checksum
Open vSwitch为保证软件的鲁棒性，**在编译时会校验vswitch.ovsschema的checksum。**

**修改vswitch.ovsschema文件后，在编译之前以下述命令生成新的校验和**。

```bash
$ cksum vswitchd/vswitch.ovsschema > vswitchd/vswitch.ovsschema.stamp
```

否则会编译错误。

### 更新系统的数据库文件
系统内的OVS database数据库文件是/usr/local/etc/openvswitch/conf.db，在安装时使用ovsdb-tool工具依照vswitch.ovsschema生成的。在修改vswitch.ovsschema文件后，这个数据库文件并不会自动更新，因此需要使用下述命令更新：

```bash
ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema
```

## 总结

概括来说，修改database schema为Open vSwitch添加新功能的步骤为：

- 修改vswitch.ovsschema
- 更新vswitch.ovsschema的校验和
- 重新编译Open vSwitch
- 替换系统内的OVS database数据库文件

根据我的实践，依照上述步骤便可以使用ovs-vsctl控制新功能了。