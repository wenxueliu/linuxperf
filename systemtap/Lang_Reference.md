
##名词约定

* probe : 表示一个探测点, 即一个 stp 脚本中的 probe
* hook : 表示 probe 后的 probe 函数

其他如果感觉英文能够更好地表达意思，就会用英文。

systemtap 退出的方式

* The user interrupts the script with a CTRL-C.
* The script executes the exit() function.
* The script encounters a sufficient number of soft errors.
* The monitored command started with the stap program’s -c option exits.

需要注意的是最后一种, 比如你在监听一个进程或命令执行过程中系统情况, 非常有用

systemtap 执行过程

* Translates the script
* Generates and compiles a kernel module
* Inserts the module; output to stap’s stdout
* CTRL-C unloads the module and terminates stap

##资源限制

###MAXNESTING

The maximum number of recursive function call levels. The default is 10

###MAXSTRINGLEN

The maximum length of strings. The default is 256 bytes for 32 bit machines and 512 bytes for all other machines.

###MAXTRYLOCK

The maximum number of iterations to wait for locks on global variables before declaring
possible deadlock and skipping the probe. The default is 1000.

###MAXERRORS

The maximum number of soft errors before an exit is triggered. The default is 0.

###MAXSKIPPED

The maximum number of skipped reentrant probes before an exit is triggered. The
default is 100.

###MINSTACKSPACE

The minimum number of free kernel stack bytes required in order to run a probe
handler. This number should be large enough for the probe handler’s own needs, plus a safety margin. The
default is 1024.

一旦触发上述值的边界，就会杀掉用户进程，并删除正在运行的内核模块, 捕获的信息也会丢失

###guru mode

it can constructs such as embedded C can violate these constraints, leading to a kernel crash or
data corruption.

SystemTap supports a guru mode where script safety features such as code and data memory reference
protection are removed. Guru mode is set by passing the -g option to the stap command. When in guru
mode, the translator accepts C code enclosed between “%{” and “%}” markers in the top level of the script
file. The embedded C code is transcribed verbatim, without analysis, in sequence, into the top level of the
generated C code. Thus, guru mode may be useful for adding #include instructions at the top level of the
generated module, or providing auxiliary definitions for use by other embedded code.

##语法

Global variables are shared among all probes and remain instantiated
as long as the SystemTap session. There is one namespace for all global variables, regardless of the script
file in which they are found. Because of possible concurrency limits, such as multiple probe handlers, each
global variable used by a probe is automatically read- or write-locked while the handler is running. A global
declaration may be written at the outermost level anywhere in a script file, not just within a block of code.
Global variables which are written but never read will be displayed automatically at session shutdown.

all SystemTap functions and probes run with interrupts disabled, thus you cannot call functions
that might sleep within the embedded C.

###嵌入 C

用 "%{" 和 "%}" 包围

    STAP_RETURN();
    STAP_RETURN("parameter should be three-two-");
    STAP_RETVALUE == 3
    STAP_ARG_val
    STAP_PRINTF("%d\n", STAP_ARG_val)
    STAP_ERROR("wrong guess: %d", (int) STAP_RETVALUE);

##脚本

###前缀

kernel
module
timer

###后缀

空 : 表示调用函数前时执行该 hook, 该 hook 可以获取该进入函数体之前的所有堆栈信息
return : 表示调用函数后执行该 hook, 该 hook 可以获取该函数体执行完的所有堆栈信息

###通配符

kernel.function("sys_*")
kernel.syscall.*

###可选

该探测点是可选的, 如果失败, 不会产生错误

kernel.function("no_such_function") ?

###内置探测点(有 DWARF 调试符)类型

依赖编译时加 -g 的程序(带调试符号) 或调试符号在单独的一个包中

Points in a kernel are identified by module, source file, line number, function name or some combination of
these.

    kernel.function(PATTERN)
    kernel.function(PATTERN).call
    kernel.function(PATTERN).return
    kernel.function(PATTERN).return.maxactive(VALUE)
    kernel.function(PATTERN).inline
    kernel.function(PATTERN).label(LPATTERN)
    module(MPATTERN).function(PATTERN)
    module(MPATTERN).function(PATTERN).call
    module(MPATTERN).function(PATTERN).return.maxactive(VALUE)
    module(MPATTERN).function(PATTERN).inline
    kernel.statement(PATTERN)
    kernel.statement(ADDRESS).absolute
    module(MPATTERN).statement(PATTERN)

.function  : hook 在函数体开始执行前调用, 函数参数可以在 hook 中使用
.return    : hook 在函数体执行完，返回之前调用, 返回值可以在 hook 中使用, 以 $return;
函数参数, 局部和全局变量也可以使用, 但是值可能已经被改变. 不支持 inline 函数
.maxactive : 同时可以 probe 数量, 默认为 KRETACTIVE
.inline    : 对 .function 的过滤，只包含函数被声明为 inline 的
.call      : 对 .function 的过滤，只包含函数没有被声明为 inline 的, 与 inline 相对.
.exported  : 对 .function 的过滤，只匹配被 exported 的函数
.statement : 对一个特定位置插入 hook, 局部变量在 hook 中可用

MPATTERN : probe 感兴趣的内核模块(已加载), 可以包含 "*", "[]", "?"
LPATTERN : 源代码的 label, 可以包含 "*", "[]", "?"
PATTERN  :

第一部分 : 函数名, 以 "*", "?" 匹配多个
第二部分 : 以 @ 开头, 之后是包含函数的源代码路径，该路径也可以包含 "*"(如 mm/slab*), 一般是相对路径(内核源路径)，当然绝对路径也可以; 该部分可选
第三部分 : 指定源代码的行号, ":" 表示绝对行号,  "+" 表示相对匹配函数入口的行号, ":*" 表示全部行, ":x-y" 表示 x 行到 y 行之间.

此外, PATTERN 也可以用一个数字常量表示一个绝对的地址，表绝对的内核地址或相对模块的地址

* 访问在作用域的变量 : $var
* 可以访问 char* 的变量 : kernel_string() 或 user_string()
* 引用全局变量 : @var("varname"),  @var("varname@src/file.c")
* 访问一个结构体的变量 : $var->field 或 @var("var@file.c")->field
* 访问数组 : $var[N] or @var("var@file.c")[N]
* 访问变量的地址 : &$var 或 &@var("var@file.c"), &var->field, &@var("var@file.c")[N]
* 一个结构体中的基本类型 : $var$, 比如结构体中包含 int, char, long 等基本类型
* 一个结构体中的非基本类型 : @var("var")$$, 比如结构体中的一个成员是结构, 可以通过后缀 "$$" 访问.

$$vars : sprintf("parm1=%x ...  ... varN=%x", $parm1, ..., $parmN, $var1, ..., $varN)
$$locals : sprintf("var1=%x ...  varN=%x", $var1, ..., $varN)
$$parms : ("parm1=%x ... parmN=%x", $parm1, ..., $parmN)


###例子

kernel.function("func[@file]")
module("modname").function("func[@file]")

kernel.function("*init*"), kernel.function("*exit*")
kernel.function("*@kernel/time.c:240")
module("ext3").function("*")

kernel.statement("func@file:linenumber")
module("modname").statement("func@file:linenumber")

kernel.statement("*@kernel/time.c:296")
kernel.statement("bio_init@fs/bio.c+3")

###无 DWARF Probe

没有调试符号并不意味着 systemtap 已经无用武之地,

比如

    asmlinkage ssize_t sys_read(unsigned int fd, char __user * buf, size_t count)

通过 uint arg(1), pointer arg(2), ulong arg(3) 分别获取 fd, buf, count

需要注意的 hook 必须首先调用 asmlinkage()

对于返回值, 可以通过 returnval() 或 returnstr()

register("regname") : 获取寄存器的值
u_register("regname") : 获取寄存器的无符号整数值

kprobe.function(FUNCTION)
kprobe.function(FUNCTION).return
kprobe.module(NAME).function(FUNCTION)
kprobe.module(NAME).function(FUNCTION).return
kprobe.statement(ADDRESS).absolute

在这种模式下， FUNCTION 和 NAME 不要用通配符. statement 只在 grun 模式下可用

例子

sudo stap -v -e 'probe kprobe.function("sys_read") { printf("read\n"); exit() }'

sudo stap -v -e 'probe kprobe.function("sys_read") { printf("read fd=%d buf=%d count=%d\n", uint_arg(1), pointer_arg(2), ulong_arg(3)); exit() }'

其实 tapset 已经内置了很多不需要调试符号的 stp 脚本, 用下面的命令查找

    grep -r "probe nd_" /usr/share/systemtap/tapset/

###用户态 probe

    process.begin
    process("PATH").begin
    process(PID).begin
    process.thread.begin
    process("PATH").thread.begin
    process(PID).thread.begin
    process.end
    process("PATH").end
    process(PID).end
    process.thread.end
    process("PATH").thread.end
    process(PID).thread.end

下面的 probe 不需要 DWARF 调试符

    process.syscall
    process("PATH").syscall
    process(PID).syscall
    process.syscall.return
    process("PATH").syscall.return
    process(PID).syscall.return

    process("PATH").function("NAME")
    process("PATH").statement("*@FILE.c:123")
    process("PATH").function("*").return
    process("PATH").function("myfun").label("foo")

    probe process("...").plt { ... }
    probe process("...").plt process("...").library("...").plt { ... }
    probe process("...").library("...").function("....") { ... }

    以下对性能影响较大
    process("PATH").insn
    process(PID).insn
    process("PATH").insn.block
    process(PID).insn.block

    process("PATH").mark("LABEL")

STAP_PROBE1

* .mark   : 程序中 STAP_PROBE1(handle,LABEL,arg1) 被调用，
* .plt    : program linkage table
* .insn   : 每次程序单步执行的时候调用
* .insn.block : 当 block 的指令执行时调用

STAP_PROBE1 只能传一个参数, STAP PROBE2 可以传两个参数, 以此类推, 需要头文件 std.h

其中 STAP_ERROR

handle : the application handle
LABEL  : corresponds to the .mark argument
arg1   : the argument

例子

    sudo stap -v -e 'probe process(9961).syscall { printf("call syscall %d %d %d %d %d %d\n", $arg1, $arg2, $arg3, $arg4, $arg5, $arg6); exit()}'
    sudo stap -v -e 'probe process(9961).syscall.return { printf("call syscall return %d", $return); exit()}'


$ cat test.stp

    probe process(9961).syscall {
        if ($arg1 == 7) {
            printf("call syscall %d %d %d %d %d %d\n", $arg1, $arg2, $arg3, $arg4, $arg5, $arg6)
        }
    }

    probe process(9961).syscall.return {
        if ($return > 0) {
            printf("call syscall return %d\n", $return)
        }
    }

    probe timer.s(3) {
        exit()
    }

执行上面脚本通过下面命令

sudo stap -v test.stp


$cat docker.stp

    probe process("/usr/bin/docker").function("*").call {
        log (probefunc()." ".$$parms)
    }

    probe timer.s(10) {
        exit()
    }

如果你已经安装, 当然，你可以替换为任何你想 probe 的进程, 执行上面脚本通过下面命令

sudo stap -v docker.stp -c 'sudo docker ps'

如果你不知道哪个进程通过什么命令启动, 可以通过 file /proc/PID/exe 获取启动进程的
程序名


###Java 程序

利用 byteman 作为工具进行指令探测

限制 :

1. 一次只能探测一个 java 进程
2. 只能探测本用户的 java 进程, (即使是 root 用户也只能探测本用户的 java 进程)


    java("PNAME").class("CLASSNAME").method("PATTERN")
    java("PNAME").class("CLASSNAME").method("PATTERN").return
    java(PID).class("CLASSNAME").method("PATTERN")
    java(PID).class("CLASSNAME").method("PATTERN").return

其中 PATTERN 不支持统配符


###ProceFS

    procfs("PATH").read
    procfs("PATH").write
    procfs.read
    procfs.write

###Marker probes

如果 marker 能够满足需求, 应该优先用 marker, 因为比基于 DWARF 调试符的方式更快，更稳定
当然, 不需要调试符号

kernel.mark("MARK")
kernel.mark("MARK").format("FORMAT")

* MARK 可以包含通配符
* FORMAT 可选

hook 中可以包含 $arg1, $arg2 以此类推, 其中  $format 和 $name 分别用于获取 FORMAT 和名称

更多参考 http://sourceware.org/systemtap/wiki/UsingMarkers.

###Tracepoints

This family of probe points hooks to static probing tracepoints inserted into the kernel or kernel modules.
As with marker probes, these tracepoints are special macro calls inserted by kernel developers to make probing
faster and more reliable than with DWARF-based probes.  DWARF debugging information is not required
to probe tracepoints. Tracepoints have more strongly-typed parameters than marker probes.


    kernel.trace("NAME")

NAME 可以包含通配符

参数的访问, 要根据情况决定.

kernel.trace("sched switch") provides the parameters $rq, $prev, and $next.

Tracepoint parameters cannot be modified; however, in guru mode a script can modify fields of parameters.

$$name : Tracepoints 的名称
$$vars : Tracepoints 的变量
$$parms : Tracepoints 参数

###系统调用

    syscall.NAME
    syscall.NAME.return


* argstr: A pretty-printed form of the entire argument list, without parentheses.
* name: The name of the system call.
* retstr: For return probes, a pretty-printed form of the system call result.


###Timer

A jiffy is a kernel-defined unit of time typically between 1 and 60 msec

    timer.jiffies(N)
    timer.jiffies(N).randomize(M)
    timer.ms(N)
    timer.ms(N).randomize(M)
    timer.profile.tick

例子

    timer.jiffies(1000)
    timer.sec(5)
    timer.jiffies(1000).randomize(200)


There are no target variables provided in either context. Probes can be run concurrently on multiple processors

It is recommended to use the tapset probe timer.profile rather than timer.profile.tick. This probe
point behaves identically to timer.profile.tick when the underlying functionality is available, and falls
back to using perf.sw.cpu_clock on some recent kernels which lack the corresponding profile timer facility.

For kernels prior to 2.6.17, timers are limited to jiffies resolution, so intervals are
rounded up to the nearest jiffies interval. After 2.6.17, the implementation uses
hrtimers for greater precision, though the resulting resolution will be dependent upon architecture.


##四段式

probe begin {
    预处理
}

probe 实际探测

probe end {
    清理工作
}

probe error {
    保存现场
    清理工作
}


##参考

