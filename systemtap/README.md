



##systemtap 可以做什么

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
*

###重用模块

stap -p4 -m MODULE_NAME SCRIPT.stp

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



##Reference

https://github.com/majek/dump/tree/master/system-tap
