



##systemtap 可以做什么

    SystemTap's goal is to provide full system observability on production systems,
    which is safe, non-intrusive, (near) zero-overhead and which allows ubiquitous
    data collection across the whole system for any interesting event that could
    happen. To achieve this goal, SystemTap defines the stap language, in which the
    user defines probes, actions, and data acquisition. The SystemTap translator and
    runtime guarantees that probe points are only placed on safe locations and that
    probe functions cannot generate too much overhead when collecting data. For
    dynamic probes on addresses inside the kernel, SystemTap uses kprobes; for dynamic
    probes in user space programs, instead, SystemTap uses its cousin uprobes.
    This provides a unified way of probing and then collecting data for observing
    the whole system. To dynamically find locations for probe points, arguments of
    the probed functions and the variables in scope at the probe point, SystemTap
    uses the debuginfo (Dwarf) standard debugging information that the compiler generates.

    Depending on the type of probe, one can access specifics of the probe point. For the
    debuginfo based probes these are $var for in-scope variables or function arguments,
    $var->field for accessing structure fields, $var[N] for array elements, $return for
    the return value of a function in a return probe, and meta variables like $$vars to
    get a string representation of all the in-scope variables at a particular probe point.
    All access to such constructs are safeguarded by the SystemTap runtime to make sure
    no illegal accesses can occur.

   更多参考 http://lwn.net/Articles/315022/


* 可以接受外边传递参数
* hook 内核函数
* 访问内核函数的参数, 返回值
* 修改内核函数的返回值

##systemtap 的原理

##systemtap 安装

参考 install

##systemtap 运行

stap -v {SCRIPT}.stp

man stap

##Document

/boot/System.map-`uname -r` : map starting addresses for each function to function name

[RedHat Systemtap Guide](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/pdf/SystemTap_Beginners_Guide/Red_Hat_Enterprise_Linux-7-SystemTap_Beginners_Guide-en-US.pdf)
man stapprobes
man stp

##Example

yum install systemtap-testsuite.x86_64

* /usr/share/systemtap/tapset/
* /usr/share/doc/systemtap-client-2.9/
* /usr/share/systemtap/testsuite

##基本命令

###stap

stap -L 'kernel.function("sys_read")'

stap -l 'kernel.function("sys_read")'




###staprun

###重用模块

stap -p4 -m MODULE_NAME SCRIPT.stp

stap -k SCRIPT.stp

##系统调优

##锁



##CPU 调优

1. 找到销毁 cpu 最严重的进程

    htop

2. 找到用户态和内核态的比例

    stp -v thread_times.stp

3. 找到消耗 CPU 的具体函数


##IO

1. 找到 IO 消耗严重的进程

    stp -v traceio.stp

2. 找到该进程读写销毁的时间, 读写文件内容大小及比例, 读写次数的比例.

    stp -x PID -v rw_ratio.stp

3. 找到目标文件每次读写的大小及消耗时间


##网络

1. 从网卡到内核的所有队列大小及是否已经满
2. 找到目标进程各个网络操作销毁的时间比率
3. 定位到与具体哪个地址, 协议耗时较高


##Tips

stap -t -DDEBUG_SYMBOLS

当遇到某些符号无法解析, 可以通 -d 或 --all-modules 解决之, -d 优先与 --all-modules
因为 --all-modules 会增加生成的模块大小


##问题

Q: ERROR: probe overhead exceeded threshold
A: 增加 --suppress-time-limits 或　-DSTP_NO_OVERLOAD 参数



##Reference

https://github.com/majek/dump/tree/master/system-tap
http://oliveryang.net/2016/07/linux-perf-tools-tips/


