---
layout: post
title: "为Open vSwitch编译添加依赖库的三种方式"
comments: true
description: "Open vSwitch"
keywords: "Open vSwitch DPDK"
---

对于实验室服务器使用的Mellanox Connect-X3 Pro网卡来说，不仅在编译DPDK时需要修改响应的编译设置，而且在编译OVS-DPDK时，也同样需要指定特殊的依赖库。此前的博客中已经介绍了修改automake.mk的方式，经过一段时间的实践，还有两种方式可以添加依赖库。

## 为什么要添加依赖库

根据我对ovs-discuss邮件列表的研究，大家认为，DPDK在使用Mellanox网卡与Intel网卡行为有差异。Mellanox网卡不需要绑定到VFIO或uio_generic或igb_uio内核模块上，也就不需要运行dpdk-devbind.py脚本。DPDK直接通过VERBS接口poll Mellanox网卡，因此，这个libibverbs的库文件对于编译OVS-DPDK而言是必要的。不修改直接编译是会报错的。

## 第一种方法：export LIBS="-libverbs"

这种方法最为简单，在执行configure脚本之前先执行这个命令，不需要修改源文件。

编译流程为：

```bash
export LIBS="-libverbs"

./configure --with-dpdk=$DPDK_BUILD

make

make install
```


## 第二种方法：修改acinclude.m4

acinclude.m4是自动编译工具aclocal的配置文件，它可以定义宏来指定编译时需要的库文件。

首先在OVS 2.6.0源文件目录下使用grep搜索DPDK库文件的定义位置:

```bash
grep -nr DPDK_LIB ./
```

结果为：

```bash
./configure:20774:        DPDK_LIB_DIR="$with_dpdk/lib"
./configure:20778:    DPDK_LIB="-ldpdk"
./configure:20785:      LDFLAGS="$LDFLAGS -L${DPDK_LIB_DIR}"
./configure:20994:        LIBS="$DPDK_LIB $extras $save_LIBS $DPDK_EXTRA_LIB"
./configure:21023:        as_fn_error $? "Could not find DPDK libraries in $DPDK_LIB_DIR" "$LINENO" 5
./configure:21029:      OVS_LDFLAGS="$OVS_LDFLAGS -L$DPDK_LIB_DIR"
./configure:21077:    DPDK_vswitchd_LDFLAGS=-Wl,--whole-archive,$DPDK_LIB,--no-whole-archive
./acinclude.m4:186:        DPDK_LIB_DIR="$with_dpdk/lib"
./acinclude.m4:190:    DPDK_LIB="-ldpdk"
./acinclude.m4:197:      LDFLAGS="$LDFLAGS -L${DPDK_LIB_DIR}"
./acinclude.m4:246:        LIBS="$DPDK_LIB $extras $save_LIBS $DPDK_EXTRA_LIB"
./acinclude.m4:263:        AC_MSG_ERROR([Could not find DPDK libraries in $DPDK_LIB_DIR])
./acinclude.m4:269:      OVS_LDFLAGS="$OVS_LDFLAGS -L$DPDK_LIB_DIR"
./acinclude.m4:282:    DPDK_vswitchd_LDFLAGS=-Wl,--whole-archive,$DPDK_LIB,--no-whole-archive
```

其中，configure脚本是由aclocal和autoconf生成的，因此只需要修改acinclude.m4文件即可。

在第191行的DPDK_EXTRA_LIB配置项中添加"-libverbs":

```bash
DPDK_EXTRA_LIB="-libverbs"
```

在编译之前首先运行自动配置工具更新编译配置文件：

```bash
aclocal

autoconf

automake
```

然后按照标准流程编译即可：

```bash
./configure --with-dpdk=$DPDK_BUILD

make

make install
```

## 第三种方法：修改automake.mk

第三种方法是修改vswitchd/automake.mk文件，修改```vswitchd/automake.mk```文件，将原来的链接设置：

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

