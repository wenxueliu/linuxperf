

这篇综述也是不错的
http://huoding.com/2016/08/18/531#comment-386419
http://www.brendangregg.com/blog/2015-07-08/choosing-a-linux-tracer.html

*
* sysdig
* ftrace
* LTTng
* perf_events
* systemtap
* eBPF




##Ktap

It has been abanoned. see the creater [announcement](https://github.com/ktap/ktap/issues/100)


##Java profile

1. System profilers: like Linux perf, which shows system code paths (eg, JVM GC, syscalls, TCP), but not Java methods.
2. JVM profilers: like hprof, LJP, and commercial profilers. These show Java methods, but usually not system code paths.


实现 Java 火焰图的两个问题:

1. The JVM compiles methods on the fly (just-in-time: JIT), and doesn't expose a traditional symbol table for system profilers to read.
2. The JVM also uses the frame pointer register (RBP on x86-64) as a general purpose register, breaking traditional stack walking.



解决上面两个问题的办法

1. A JVMTI agent, [perf-map-agent](https://github.com/jrudolph/perf-map-agent), which can provide a Java symbol table for perf to read (/tmp/perf-PID.map).
2. Patching JDK hotspot to reintroduce the frame pointer register, which allows full stack walking.




其他工具

###latencytop

ubuntu 下

$ cat /proc/latency_stats

    Latency Top version : v0.1

$ sudo apt-get install latencytop

Redhat 由于没有打开编译选项, 需要自己编译. 安装很简单.


##参考

http://dtrace.org/blogs/brendan/
