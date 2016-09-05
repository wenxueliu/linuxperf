

##预备知识

* /lib/modules/`uname -r`/kernel/ 包含内核模块
* /boot/config-`uname -r` 包含内核编译参数, 可以通过 grep 获取
* /usr/src/debug/kernel-2.6.32-642.4.2.el6/linux-2.6.32-642.4.2.el6.x86_64/ 包含内核源码, 在 kernel-debuginfo-common 包中

###关于内核态和用户态跟踪

对内核态跟踪需要安装调试符号表

kprobe

对用户态跟踪需要对应程序的调试符号表

uprobe 或 utrace

yum install yum-utils

debuginfo-install


##基本概念

###跟踪事件(probe event)


###跟踪点(probe tracepoint)



##perf 工作流


##查看 perf 预定义的事件

$ perf list

$ perf list hw

$ perf list sw

$ perf list pmu

$ perf list "net:*"

$ perf list "block"

$ perf list subsys block:*

$ perf list "sched:*"

$ perf list "irq:*"

$ perf list "syscalls:sys_exit_a*"

更多参考 perf list --help

但 perf 只能显示 symbolic event types 对于更详细的模块函数需要借助 perf probe

$ perf probe -F

$ perf probe -F -m ext4

$ perf probe -F -m /lib/modules/2.6.32-642.4.2.el6.x86_64/kernel/fs/ext4/ext4.ko

$ perf probe -v -L vfs_read //ext4 中的 vfs_read

$ perf probe -v -V vfs_read

$ perf probe -v -V vfs_read --externs

$ perf probe -x /lib64/libc.so.6 -F

$ perf probe -x /lib64/libc.so.6 -L malloc

$ perf probe -x /lib64/libc.so.6 -V malloc

$ perf probe -x /bin/ls -F

$ perf probe -x /bin/ls -V acl_access_nontrivial

    The /bin/ls file has no debug information.
    Rebuild with -g, or install an appropriate debuginfo package.
      Error: Failed to show vars.

对于没有符号表的进程, 运行:

$ gdb ls

    GNU gdb (GDB) Red Hat Enterprise Linux (7.2-90.el6)
    Copyright (C) 2010 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>...
    Reading symbols from /bin/ls...(no debugging symbols found)...done.
    Missing separate debuginfos, use: debuginfo-install coreutils-8.4-43.el6.x86_64
    debuginfo-install coreutils-8.4-43.el6.x86_64

$ debuginfo-install coreutils-8.4-43.el6.x86_64

重新运行即可


所有通过 probe -F 显示的函数都可以通过 perf probe -v -L FUNCTION, 和
probd -v -V FUNCTION 看到更详细的信息

$ perf probe -F /lib/modules/`uname -r`/kernel/net/bridge/bridge.ko

如果你知道模块，但是不知道路径, 可以通过 modinfo MODULENAME 找到


$ modinfo openvswitch

    filename:       /lib/modules/2.6.32-642.4.2.el6.x86_64/kernel/net/openvswitch/openvswitch.ko
    license:        GPL
    description:    Open vSwitch switching datapath
    srcversion:     00938868C288DBF055E30F3
    depends:        libcrc32c,vxlan
    vermagic:       2.6.32-642.4.2.el6.x86_64 SMP mod_unload modversions

$ perf probe -F -m /lib/modules/2.6.32-642.4.2.el6.x86_64/kernel/net/openvswitch/openvswitch.ko

如果你不确定具体的模块名，但是知道大约的路径, 只能在 /lib/modules/`uname -r`/kernel/ 下查找了.




##跟踪事件

例子

$ perf probe --add 'vfs_read file:string'
$ perf record -e probe:vfs_read -aR sleep 10
$ perf script


##跟踪用户态

$ debuginfo-install glibc

$

##收集性能统计

经过前面的介绍，结合我们的需求, 我们就可以知道该具体该跟踪哪些事件, 因此,
下面就介绍如何进行性能统计.

###计数统计

统计一个程序的最直观的方式就是统计调用次数, 比如, 在原理的程序上对 cache 亲和性
更好, 那么, 如何验证呢, 没有 perf, 只能通过所用时间来从侧面验证，但是，现在有了
perf，跑同样的数据，看看 cache-miss 的比例就知道了，从正面验证是否真正提高了
cache 亲和性.

perf stat 要点分两部分

1. 事件: 通过 -e 参数指定
2. 命令: -p 到具体的进程, -t 到具体的线程, -C 具体的 CPU, 或 sleep N 全系统

perf state -e EVENT1 -e EVENT2 [-p PID] [-t TID] [-C CPUS] sleep 10
perf state -e EVENT1 -e EVENT2 [-p PID] [-t TID] [-a] sleep 10

更多参考 man perf-stat

####例子

$ perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -a sleep 10

$ perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -a ls

$ perf stat -e cycles,instructions,cache-references,cache-misses,bus-cycles -C 0,1 ls

$ perf stat -e dTLB-loads,dTLB-load-misses,dTLB-prefetch-misses -p PID

$ perf stat -e LLC-loads,LLC-load-misses,LLC-stores,LLC-prefetches COMMAND

$ perf stat -e L1-dcache-loads,L1-dcache-load-misses,L1-dcache-stores COMMAND

$ perf stat -e r003c -a sleep 5

$ perf stat -e cycles -e cpu/event=0x0e,umask=0x01,inv,cmask=0x01/ -a sleep 5

$ perf stat -e 'syscalls:sys_enter_*' -p PID

$ perf stat -e 'sched:*' -p PID sleep NSecs


###profiling

perf record -F 99 COMMAND

perf record -F 99 -p PID

perf record -F 99 -p PID sleep 10

perf record -F 99 -p PID -g -- sleep 10

perf record -F 99 -p PID -g dwarf sleep 10

perf record -F 99 -p PID -s -g dwarf sleep 10

perf record -F 99 -ag -- sleep 10

perf record -F 99 -e cpu-clock -ag -- sleep 10

perf record -F 99 -ag dwarf sleep 10

perf record -e L1-dcache-load-misses -c 10000 -ag -- sleep 5

perf record -e LLC-load-misses -c 100 -ag -- sleep 5

perf record -e cycles:k -a -- sleep 5

perf record -e cycles:u -a -- sleep 5

perf record -e cycles:p -a -- sleep 5

perf record -b -a sleep 1

perf top -F 49

perf top -F 49 -ns comm,dso

stdbuf -oL perf top -e net:net_dev_xmit -ns comm | strings















##常见问题

$ perf probe -x /lib/libc.so.6 malloc

    uprobe_events file does not exist - please rebuild kernel with CONFIG_UPROBE_EVENTS.
      Error: Failed to add events.


##参考

http://oliveryang.net/2016/07/linux-perf-tools-tips/
https://github.com/torvalds/linux/blob/master/Documentation/trace/kprobetrace.txt

