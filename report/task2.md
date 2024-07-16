
[the R4L out of tree template](https://github.com/Rust-for-Linux/rust-out-of-tree-module)

# 问题
Makefile 文件中的内容很简单，如下所示：

```sh
KDIR ?= ../linux

default:
	$(MAKE) -C $(KDIR) M=$$PWD
```

其中，选项 `C` 指定在读取 makefile 和执行构建命令之前，先切换到由 $(KDIR) 指定的目录；`M` 变量由内核构建系统用来指定模块源代码所在的目录

# 记录
```
~ # insmod r4l_e1000_demo.ko
[   32.907654] r4l_e1000_demo: loading out-of-tree module taints kernel.
[   32.913370] r4l_e1000_demo: Rust for linux e1000 driver demo (init)
[   32.913918] r4l_e1000_demo: Rust for linux e1000 driver demo (probe): None
[   33.066234] ACPI: \_SB_.LNKC: Enabled at IRQ 11
[   33.087657] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[   33.089484] insmod (81) used greatest stack depth: 11144 bytes left
```

```
~ # ifconfig
[  120.002654] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64) 
```

```
~ # ip link set eth0 up
[  179.259495] r4l_e1000_demo: pending_irqs: 3
[  179.259790] r4l_e1000_demo: Rust for linux e1000 driver demo (napi poll)
[  196.154281] r4l_e1000_demo: Rust for linux e1000 driver demo (net device start_xmit) tdt=0, tdh=0, rdt=7, rdh=0
[  196.154662] r4l_e1000_demo: Rust for linux e1000 driver demo (handle_irq)
```

```
~ # ifconfig
[  281.693779] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
eth0      Link encap:Ethernet  HWaddr 52:54:00:12:34:56
          inet6 addr: fe80::5054:ff:fe12:3456/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

```
~ # ip addr add broadcast 10.0.2.255 dev eth0
[  524.304685] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[  524.306324] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
ip: RTNETLINK answers: Invalid argument
```

```
~ # ip addr add 10.0.2.15/255.255.255.0 dev eth0
[  698.392017] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
[  698.392418] r4l_e1000_demo: Rust for linux e1000 driver demo (net device get_stats64)
```

```
~ # ip route add default via 10.0.2.1
```

可以 ping 10.0.2.2

