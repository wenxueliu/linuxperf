

##预备知识

###符号表

perf 跟踪依赖与调试信息(symbols), 调试符号表的作用就是将内存的十六进制翻译为对应的函数即参数.

对于内核, 通过安装对应内核版本的调试包, 可以解决.  还可以自己手动编译内核源码增加调试相关信息.
对于用户态, 通过安装对应程序的调试符号包也可以解决. 还可以自己手动编译源码，不要 strip 调试符号.

检验你所用的内核是否支持调试符号, 运行

cat /boot/config-2.6.32-642.4.2.el6.x86_64 | grep CONFIG_KALLSYMS

    CONFIG_KALLSYMS=y
    CONFIG_KALLSYMS_ALL=y
    CONFIG_KALLSYMS_EXTRA_PASS=y

###栈帧

被优化的程序是忽略栈指针的, 如果没有栈帧, 有些调试符号就不能正确地显示.

自从 kernel 3.9, 对于应用户态的程序, perf_events 支持利用 dwarf(libunwind)
来绕过这个缺失栈帧的问题. 在编译的时候加上 -g dwarf 即可.

对应用户态, 编译时增加:

    -fno-omit-frame-pointer:

对应内核，编译时加参数

    CONFIG_FRAME_POINTER=y

CentOS 默认支持

    cat /boot/config-2.6.32-642.4.2.el6.x86_64 | grep CONFIG_FRAME_POINTER

注:

* /lib/modules/`uname -r`/kernel/ 包含内核模块
* /boot/config-`uname -r` 包含内核编译参数, 可以通过 grep 获取
* /usr/src/debug/kernel-2.6.32-642.4.2.el6/linux-2.6.32-642.4.2.el6.x86_64/ 包含内核源码, 在 kernel-debuginfo-common 包中

所有事件都可以在 /usr/src/debug/kernel-2.6.32-642.4.2.el6/linux-2.6.32-642.4.2.el6.x86_64/include/trace/events/
找到相应的定义. 对于 --filter 参数比较有用.


###关于内核态和用户态跟踪

对内核态跟踪需要安装调试符号表

kprobe



对用户态跟踪需要对应程序的调试符号表

uprobe 或 utrace

yum install yum-utils

debuginfo-install


##基本概念

##事件

###Hardware Events

These instrument low-level processor activity based on CPU performance counters.
For example, CPU cycles, instructions retired, memory stall cycles, level 2 cache
misses, etc. Some will be listed as Hardware Cache Events.

###Software Events

These are low level events based on kernel counters. For example, CPU migrations,
minor faults, major faults, etc.

###Tracepoint Events

This are kernel-level events based on the ftrace framework. These tracepoints are
placed in interesting and logical locations of the kernel, so that higher-level
behavior can be easily traced. For example, system calls, TCP events, file system
I/O, disk I/O, etc. These are grouped into libraries of tracepoints; eg, "sock:"
for socket events, "sched:" for CPU scheduler events.

###Dynamic Tracing

Software can be dynamically instrumented, creating events in any location. For
kernel software, this uses the kprobes framework. For user-level software, uprobes.

###Timed Profiling

Snapshots can be collected at an arbitrary frequency, using perf record -FHz. This
is commonly used for CPU usage profiling, and works by creating custom timed interrupt events.


###跟踪事件(probe event)


###跟踪点(probe tracepoint)


###perf 主要能力

* counting : 计数, 可以统计某个内核函数调用次数. 通过 perf stat 达成.
* sampling : 采样, 对应需要监控的事件, 内核会将事件信息写入内核 buffer, 然后异步地发送到用户态, 默认保持在 perf.data 文件. 通过 perf record 达成
* bpf : 可以在内核空间执行自定义的操作, 据此过滤,分析内核事件. 需要 4.4 以上内核.

bpf 应该是未来趋势, 它的优点是 TODO.

perf record : 如果采样事件过于密集, 会生成非常大的问题需要注意.

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

perf list | awk -F: '/Tracepoint event/ { lib[$1]++ } END { for (l in lib) { printf "  %-16s %d\n", l, lib[l] } }'

更多参考 perf list --help

但 perf 只能显示 symbolic event types 对于更详细的模块函数需要借助 perf probe

perf probe -F

perf probe -F -m ext4

perf probe -F -m /lib/modules/2.6.32-642.4.2.el6.x86_64/kernel/fs/ext4/ext4.ko

perf probe -v -L vfs_read //ext4 中的 vfs_read

perf probe -v -V vfs_read

perf probe -v -V vfs_read --externs

perf probe -x /lib64/libc.so.6 -F

perf probe -x /lib64/libc.so.6 -L malloc

perf probe -x /lib64/libc.so.6 -V malloc

perf probe -x /lib64/libc.so.6 --add malloc

    uprobe_events file does not exist - please rebuild kernel with CONFIG_UPROBE_EVENTS.
      Error: Failed to add events.

perf probe -x /bin/ls -F

perf probe -x /bin/ls -V acl_access_nontrivial

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

注: 更高的采样频率意味着更高的负载. 而一般用 99 而不用 100

The choice of 99 Hertz, instead of 100 Hertz, is to avoid accidentally sampling
in lockstep with some periodic activity, which would produce skewed results.

对应不完整的用户态栈, 可以在 linux kernel 3.9 版本之后可以在 perf record 后加上 -g dwarf

###静态跟踪

perf record -e sched:sched_process_fork -a

perf record -e context-switches -a

perf record -e context-switches -ag

perf record -e context-switches -ag -- sleep 10

perf record -e migrations -a -- sleep 10

perf record -e syscalls:sys_enter_connect -ag

perf record -e syscalls:sys_enter_accept* -ag

perf record -e block:block_rq_insert -ag

perf record -e block:block_rq_issue -e block:block_rq_complete -a

perf record -e block:block_rq_complete --filter 'nr_sector > 200'

perf record -e block:block_rq_complete --filter 'rwbs == "WS"'

perf record -e block:block_rq_complete --filter 'rwbs ~ "*W*"'

perf record -e minor-faults -ag

perf record -e page-faults -ag

perf record -e 'ext4:*' -o /tmp/perf.data -a

perf record -e vmscan:mm_vmscan_wakeup_kswapd -ag

perf record -e block:block_rq_issue -e block:block_rq_complete -a sleep 120 //磁盘延迟跟踪

其中 --fliter 具体可以传递的参数参见 include/trace/events 中相关文件

###动态跟踪

如果添加一个动态跟踪点的步骤

0. 找到你想跟踪的事件, 并定位内核对于的函数;
1. 查找现有的跟踪点是否满足需求; 如 perf probe -l 以及 perf list
2. 列出 per 跟踪的内核源码; 如 perf probe -L KERNEL_FUNC
3. 列出内核函数对应的参数; 如 perf probe -V KERNEL_FUNC
4. 添加跟踪事件; 如 perf probe 'KERNEL_FUNC PARAM'
5. 跟踪事件; 如 perf record 'KERNEL_FUNC' -a sleep 10
6. 查看事件; 如 perf script
7. 如果不需要删除该跟踪点; 如 perf probe -d 'KERNEL_FUNC'

详细一个例子

0. perf probe -l
1. perf probe -L do_sys_open
2. perf probe -V do_sys_open
3. perf probe -V do_sys_open:8
4. perf probe 'do_sys_open filename:string'
5. perf record -e probe:do_sys_open -aR sleep 10
6. perf script
7. perf probe -d do_sys_open



perf probe --add tcp_sendmsg

perf probe -d tcp_sendmsg

perf probe -V tcp_sendmsg

perf probe -V tcp_sendmsg --externs

perf probe -L tcp_sendmsg

perf probe -V tcp_sendmsg:16

perf probe -V tcp_sendmsg:26

perf probe -V tcp_sendmsg:48

perf probe 'tcp_sendmsg %ax %dx %cx'

perf probe -f 'tcp_sendmsg bytes=%cx'

perf record -e probe:tcp_sendmsg --filter 'bytes > 100'

perf probe -f 'tcp_sendmsg%return $retval'

perf record -e probe:tcp_sendmsg --filter 'bytes > 100'

perf probe -f 'tcp_sendmsg%return $retval'

perf probe 'tcp_sendmsg size'

perf probe -f 'tcp_sendmsg size sock->sk->__sk_common.skc_state'

perf probe -f 'tcp_sendmsg seglen sk'


需要注意的 probe 增加跟踪事件与内核相关, 尤其是参数方面, 具体要以相关版本的
内核代码为准.

##报告

perf report

perf report -n

perf report --stdio

perf script

perf script -F time,event

perf script -D

perf annotate --stdio


##BPF

TODO

##CPU 火焰图

git clone https://github.com/brendangregg/FlameGraph
cd FlameGraph

perf record -F 99 -ag -- sleep 60

perf script | ./stackcollapse-perf.pl > out.perf-folded
cat out.perf-folded | ./flamegraph.pl > perf-kernel.svg

perf script | ./stackcollapse-perf.pl | ./flamegraph.pl > perf-kernel.svg

perf script | ./stackcollapse-perf.pl > out.perf-folded

grep -v cpu_idle out.perf-folded | ./flamegraph.pl > nonidle.svg

grep ext4 out.perf-folded | ./flamegraph.pl > ext4internals.svg

egrep 'system_call.*sys_(read|write)' out.perf-folded | ./flamegraph.pl > rw.svg

##延迟火焰图

perf record -e block:block_rq_issue -e block:block_rq_complete -a sleep 120

perf script | awk '{ gsub(/:/, "") } $5 ~ /issue/ { ts[$6, $10] = $4 }
    $5 ~ /complete/ { if (l = ts[$6, $9]) { printf "%.f %.f\n", $4 * 1000000,
    ($4 - l) * 1000000; ts[$6, $10] = 0 } }' > out.lat_us




##常见问题

$ perf probe -x /lib/libc.so.6 malloc

    uprobe_events file does not exist - please rebuild kernel with CONFIG_UPROBE_EVENTS.
      Error: Failed to add events.



##参考

http://oliveryang.net/2016/07/linux-perf-tools-tips/
https://github.com/torvalds/linux/blob/master/Documentation/trace/kprobetrace.txt

##附录

###缩略词

PMU : performance monitoring unit
PMC : performance monitoring counters
PIC : performance instrumentation counters
IPC : instructions per cycle

###调试基于 VM 的语言

对于所有 GC 的语言, 都自带 VM, 每个 VM 本身运行是可以通过 perf 直接跟踪的(只要安装对应的调试符号)
由于具体的语言最终都被翻译为本地代码, 因此单从 VM 的符号地址是无法还原到语言具体的函数,
因此, 需要 VM 提供语言函数与地址的映射.

###静态跟踪点和动态跟踪点的区别

静态跟踪只能跟踪一些内核最顶层的 hook, 而动态跟踪可以跟踪整个系统的任何地方. 对应内核, 利用 kprobe,
对应用户态程序利用 uprobe



###内核参数

for perf_events:

    CONFIG_PERF_EVENTS=y

for stack traces:

    CONFIG_FRAME_POINTER=y

kernel symbols:

    CONFIG_KALLSYMS=y

tracepoints:

    CONFIG_TRACEPOINTS=y

kernel function trace:

    CONFIG_FTRACE=y

kernel-level dynamic tracing:

    CONFIG_KPROBES=y
    CONFIG_KPROBE_EVENTS=y

user-level dynamic tracing:

    CONFIG_UPROBES=y
    CONFIG_UPROBE_EVENTS=y

full kernel debug info:

    CONFIG_DEBUG_INFO=y

kernel lock tracing:

    CONFIG_LOCKDEP=y

kernel lock tracing:

    CONFIG_LOCK_STAT=y

kernel dynamic tracepoint variables:

    CONFIG_DEBUG_INFO=y


###perf vs strace

strace 只能跟踪系统调用, 此外, 负载要比 perf 大. 原因是 perf 数据在内核的
buffer. 此外, perf 功能要更为强大

当然, 在一些场合 strace 仍然有用武之地, 因为 strace 安装比 perf 要简单得多.
如果只是跟踪一个进程读写哪些文件, strace 是比 perf 更为直观简单.

感受下区别:

dd if=/dev/zero of=/dev/null bs=512 count=1000k

    1024000+0 records in
    1024000+0 records out
    524288000 bytes (524 MB) copied, 0.34933 s, 1.5 GB/s

perf stat -e 'syscalls:sys_enter_*' dd if=/dev/zero of=/dev/null bs=512 count=1000k

    1024000+0 records in
    1024000+0 records out
    524288000 bytes (524 MB) copied, 0.830807 s, 631 MB/s

strace -c dd if=/dev/zero of=/dev/null bs=512 count=1000k

    1024000+0 records in
    1024000+0 records out
    524288000 bytes (524 MB) copied, 30.328 s, 17.3 MB/s
