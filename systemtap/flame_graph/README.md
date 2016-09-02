
##Preface

确保你已经安装了 systemtap. 具体参考 systamtap 文件夹中的 README

##Example

运行

$ sudo ./on_cpu_tool -p 21865 -t 50 -u -k > test.bt

    WARNING: missing unwind/symbol data for module 'stap_0bfcdd63ad3f579440e1276ddf12a591_7553'
    WARNING: missing unwind/symbol data for module 'uprobes'
    WARNING: Tracing 21865 (/usr/local/sbin/ovs-vswitchd) in both user-space and kernel-space...
    WARNING: Missing unwind data for module, rerun with 'stap -d /lib64/libc-2.12.so (deleted)'
    WARNING: no or bad debug frame hdr
    WARNING: No binary search table for eh frame, doing slow linear search for stap_b086928e3189235b7322242832c1081_32318
    WARNING: Missing unwind data for module, rerun with 'stap -d /lib64/libpthread-2.12.so.#prelink#.FHp24m (deleted)'
    WARNING: _stp_read_address failed to access memory location
    WARNING: Time's up. Quitting now...(it may take a while)


$ ./stackcollapse-stap.pl ovs.bt > test.cbt

$ ./flamegraph.pl ovs.cbt > test.svg

其中

* 21865 是进程号
* FLAME_NAME 是生成火焰图的名称

在当前路径下生成 FLAME_NAME.svg.

用浏览器打开即可. 具体问题参考附录


###读懂火焰图

* 每个框代表一个栈里的一个函数
* Y轴代表栈深度（栈桢数）。最顶端的框显示正在运行的函数，这之下的框都是调用者。在下面的函数是上面函数的父函数
* X轴代表采样总量。与常见的图片不同, 从左到右并不代表时间变化，从左到右也不具备顺序性
* 框的宽度代表函数 On-CPU 的总时间。越宽的框可能是每次调用消耗更多的 CPU 时间, 也可能被调用了更多次数。
* 如果是多线程同时采样，采样总数会超过总时间
* 框的颜色深浅也没有任何意义, 只是随机选的一些暖色.
* 鼠标在上面移动，点击可以看详细信息

##附录

在运行发现有许多调试符号没有, 需要安装对应库的调试符号

yum install -y yum-utils

$ gdb -p PID

    GNU gdb (GDB) Red Hat Enterprise Linux (7.2-90.el6)
    Copyright (C) 2010 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Attaching to process 21865
    Reading symbols from /usr/local/sbin/ovs-vswitchd...done.
    Reading symbols from /usr/lib64/libssl.so.10...(no debugging symbols
    found)...done.
    Loaded symbols for /usr/lib64/libssl.so.10
    Reading symbols from /usr/lib64/libcrypto.so.10...(no debugging symbols
    found)...done.
    Loaded symbols for /usr/lib64/libcrypto.so.10
    Reading symbols from /lib64/librt.so.1...(no debugging symbols found)...done.
    Loaded symbols for /lib64/librt.so.1
    Reading symbols from /lib64/libm.so.6...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libm.so.6
    Reading symbols from /lib64/libc.so.6...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libc.so.6
    Reading symbols from /lib64/libpthread.so.0...(no debugging symbols
    found)...done.
    [New LWP 21874]
    [New LWP 21873]
    [New LWP 21872]
    [New LWP 21871]
    [New LWP 21870]
    [Thread debugging using libthread_db enabled]
    Loaded symbols for /lib64/libpthread.so.0
    Reading symbols from /lib64/libgssapi_krb5.so.2...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libgssapi_krb5.so.2
    Reading symbols from /lib64/libkrb5.so.3...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libkrb5.so.3
    Reading symbols from /lib64/libcom_err.so.2...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libcom_err.so.2
    Reading symbols from /lib64/libk5crypto.so.3...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libk5crypto.so.3
    Reading symbols from /lib64/libdl.so.2...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libdl.so.2
    Reading symbols from /lib64/libz.so.1...(no debugging symbols found)...done.
    Loaded symbols for /lib64/libz.so.1
    Reading symbols from /lib64/ld-linux-x86-64.so.2...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/ld-linux-x86-64.so.2
    Reading symbols from /lib64/libkrb5support.so.0...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libkrb5support.so.0
    Reading symbols from /lib64/libkeyutils.so.1...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libkeyutils.so.1
    Reading symbols from /lib64/libresolv.so.2...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libresolv.so.2
    Reading symbols from /lib64/libselinux.so.1...(no debugging symbols
    found)...done.
    Loaded symbols for /lib64/libselinux.so.1
    0x00007fb5a87de283 in poll () from /lib64/libc.so.6
    Missing separate debuginfos, use: debuginfo-install glibc-2.12-1.192.el6.x86_64 keyutils-libs-1.4-5.el6.x86_64 krb5-libs-1.10.3-57.el6.x86_64 libcom_err-1.41.12-22.el6.x86_64 libselinux-2.0.94-7.el6.x86_64 openssl-1.0.1e-48.el6_8.1.x86_64 zlib-1.2.3-29.el6.x86_64

优先进程没有权限结果如下

$ gdb -p 21865

    GNU gdb (GDB) Red Hat Enterprise Linux (7.2-90.el6)
    Copyright (C) 2010 Free Software Foundation, Inc.
    License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
    This is free software: you are free to change and redistribute it.
    There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
    and "show warranty" for details.
    This GDB was configured as "x86_64-redhat-linux-gnu".
    For bug reporting instructions, please see:
    <http://www.gnu.org/software/gdb/bugs/>.
    Attaching to process 21865
    ptrace: Operation not permitted.



$ sudo debuginfo-install -y glibc-2.12-1.192.el6.x86_64 keyutils-libs-1.4-5.el6.x86_64 krb5-libs-1.10.3-57.el6.x86_64 libcom_err-1.41.12-22.el6.x86_64 libselinux-2.0.94-7.el6.x86_64 openssl-1.0.1e-48.el6_8.1.x86_64 zlib-1.2.3-29.el6.x86_64

    Loaded plugins: fastestmirror
    enabling epel-debuginfo
    Loading mirror speeds from cached hostfile
    http://download.fedoraproject.org/pub/epel/6/x86_64/debug/repodata/75f2b3b35708d8ec9b2b6c2b26285746a68c1c992dfaa109ac794cab5c05cd2d-primary.sqlite.bz2: [Errno 14] PYCURL ERROR 22 - "The requested URL returned err
    or: 404 Not Found"
    Trying other mirror.
    To address this issue please refer to the below knowledge base article 

    https://access.redhat.com/articles/1320623

    If above article doesn't help to resolve this issue please open a ticket with Red Hat Support.

    failure: repodata/75f2b3b35708d8ec9b2b6c2b26285746a68c1c992dfaa109ac794cab5c05cd2d-primary.sqlite.bz2 from epel-debuginfo: [Errno 256] No more mirrors to try.

对于如上错误:

打开浏览器到

http://download.fedoraproject.org/pub/epel/6/x86_64/debug/repodata/

下载. 拷贝到 /var/cache/yum/x86_64/6/epel-debuginfo/

之后重新运行

$ sudo debuginfo-install -y glibc-2.12-1.192.el6.x86_64 keyutils-libs-1.4-5.el6.x86_64 krb5-libs-1.10.3-57.el6.x86_64 libcom_err-1.41.12-22.el6.x86_64 libselinux-2.0.94-7.el6.x86_64 openssl-1.0.1e-48.el6_8.1.x86_64 zlib-1.2.3-29.el6.x86_64

如果部分包安装失败, 重试即可. 实在不行记录每个包, 用 wget -c 下载对应的包.
如
    $ wget -c http://debuginfo.centos.org/6/x86_64/krb5-debuginfo-1.10.3-57.el6.x86_64.rpm
    $ sudo rpm -ivh krb5-libs-1.10.3-57.el6.x86_64.rpm


$ sudo ./on_cpu_tool -p 21865 -t 50 -u -k > test.bt

    WARNING: Tracing 21865 (/usr/local/sbin/ovs-vswitchd) in both user-space and kernel-space...
    WARNING: Missing unwind data for module, rerun with 'stap -d /lib64/libc-2.12.so (deleted)'
    WARNING: no or bad debug frame hdr
    WARNING: No binary search table for eh frame, doing slow linear search for stap_b086928e3189235b7322242832c1081_32715
    WARNING: Missing unwind data for module, rerun with 'stap -d /lib64/libpthread-2.12.so.#prelink#.FHp24m (deleted)'
    WARNING: Time's up. Quitting now...(it may take a while)

对比发现比之前的警告少了.


##无法跟踪用户态程序

对于内核大于 3.5，运行

    $ grep CONFIG_UPROBES /boot/config-`uname -r`

对于内核小于 3.5, 运行

    $ grep CONFIG_UTRACE /boot/config-`uname -r`

    CONFIG_UTRACE=y

表面可以跟踪用户态程序

https://sourceware.org/systemtap/wiki/utrace

##

SystemTap 需要 Dwarf 标准的调试信息


##参考

http://huoding.com/2016/08/18/531#comment-386419
http://www.brendangregg.com/FlameGraphs/cpuflamegraphs.html
