
###Perf 工作原理

the perf profiling tool has a user space part and a
kernel space part. The collection of stack traces is done by the kernel.
When a user-specified event (or series of events) occur, the process
being profiled is interrupted and the sampled information (which can
optionally include a full stack trace) is made available to the user space
perf tool to be saved to a file for future post-profiling processing.

During the profiling phase, the perf tool collects information about the
profiled process's memory mappings, which allows for this address-to-symbol.
resolution, It's in the post-profiling phase where the sampled instruction,
along with its associated stack trace, are resolved to the appropriate symbol
(i.e., function/method) in a specific binary file (e.g., library, exectuable).

And if the VM creates a /tmp/perf-<PID>.map file to save information about
JITed methods, the perf's post-profiling tool will find it and use it to
correlate sampled addresses it collected from the VM's executable anonymous
memory mappings to the method names.

##安装

$ sudo yum install perf


##例子

$ perf record -p 8187 -F 99 -g -o ovs_kerner_4_4.data -- sleep 90
$ perf script -i ovs_kerner_4_4.data | ./stackcollapse-perf.pl > ovs_perf_kernel_4_4.out
$ cat ovs_perf_kernel_4_4.out | ./flamegraph.pl > ovs_perf_kernel_4_4.svg
$ egrep 'mem*' ovs_perf_kernel_4_4.out | ./flamegraph.pl > mem.svg


$ perf list
$ perf list sys

##参考

http://www.brendangregg.com/perf.html
http://mail.openjdk.java.net/pipermail/hotspot-compiler-dev/2014-December/016552.html
http://www.brendangregg.com/perf.html
