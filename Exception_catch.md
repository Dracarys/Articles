如果你很清楚操作系统的信号，那么跳过异常这一节。

## 异常的来源
异常控制流（Exceptional Control Flow)

- 系统层
- 应用层


## 系统层的异常
异常（Exception），就是控制流中的突变，用来响应处理器状态中的某些变化。

示例图

当处理器检测到有异常事件发生时，它就通过一个叫做异常表（exception table）的跳转表，进行一个间接过程调用（异常），到一个专门设计用来处理这类事件的操作系统子程序（异常处理程序（exception handler））。当异常处理程序完成处理后，根据引起异常的事件的类型，会发生一下 3 中情况中的一种：

- 处理程序将控制返回给当前指令 Icurr，
- 处理程序将控制返回给 Inext
- 处理程序终止被中断的程序。即“崩溃”

### 异常处理

### 异常的类别
异常可以分为四类：中断、陷阱、故障、终止

|类别|原因|异步/同步|返回行为|
|:--|:---|:------|:------|
|中断|来自 I/O 设备的信号|异步|总是返回到下一条指令|
|陷阱|有意的异常|同步|总是返回到下一条指令|
|故障|潜在可恢复到错误|同步|可能返回到当前指令|
|终止|不可回复的错误|同步|不会返回|

**终止**

终止是不可恢复的知名错误造成的结果，通常是一些硬件错误，但是某些严重疏忽导致的 Bug 也会终止。终止处理程序从不将控制返回给应用程序，而是讲控制返回给一个 abort 例程，该例程会终止这个应用程序——崩溃。

### 进程的终止

进程会因为三种原因终止：

1. 收到一个信号，该信号的默认行为是终止进程；
2. 从主程序返回；
3. 调用 exit 函数；

## 信号
一个信号就是一条小消息，它通知jinching系统中发生了一个某种类型的事件。每种信号类型都对应于某种系统事件。此层的硬件异常是由内核异常处理程序处理的，正常情况下，对用户进程是不可见的。信号提供了一种机制，通知用户进程发生了这些异常。

下图展示了系统定义的不同类型的信号：

```c
/* <sys/signla.h>*/
#define	SIGHUP	1	/* hangup */
#define	SIGINT	2	/* interrupt */
#define	SIGQUIT	3	/* quit */
#define	SIGILL	4	/* illegal instruction (not reset when caught) */
#define	SIGTRAP	5	/* trace trap (not reset when caught) */
#define	SIGABRT	6	/* abort() */
#if  (defined(_POSIX_C_SOURCE) && !defined(_DARWIN_C_SOURCE))
#define	SIGPOLL	7	/* pollable event ([XSR] generated, not supported) */
#else	/* (!_POSIX_C_SOURCE || _DARWIN_C_SOURCE) */
#define	SIGIOT	SIGABRT	/* compatibility */
#define	SIGEMT	7	/* EMT instruction */
#endif	/* (!_POSIX_C_SOURCE || _DARWIN_C_SOURCE) */
#define	SIGFPE	8	/* floating point exception */
#define	SIGKILL	9	/* kill (cannot be caught or ignored) */
#define	SIGBUS	10	/* bus error */
#define	SIGSEGV	11	/* segmentation violation */
#define	SIGSYS	12	/* bad argument to system call */
#define	SIGPIPE	13	/* write on a pipe with no one to read it */
#define	SIGALRM	14	/* alarm clock */
#define	SIGTERM	15	/* software termination signal from kill */
#define	SIGURG	16	/* urgent condition on IO channel */
#define	SIGSTOP	17	/* sendable stop signal not from tty */
#define	SIGTSTP	18	/* stop signal from tty */
#define	SIGCONT	19	/* continue a stopped process */
#define	SIGCHLD	20	/* to parent on child stop or exit */
#define	SIGTTIN	21	/* to readers pgrp upon background tty read */
#define	SIGTTOU	22	/* like TTIN for output if (tp->t_local&LTOSTOP) */
#if  (!defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE))
#define	SIGIO	23	/* input/output possible signal */
#endif
#define	SIGXCPU	24	/* exceeded CPU time limit */
#define	SIGXFSZ	25	/* exceeded file size limit */
#define	SIGVTALRM 26	/* virtual time alarm */
#define	SIGPROF	27	/* profiling time alarm */
#if  (!defined(_POSIX_C_SOURCE) || defined(_DARWIN_C_SOURCE))
#define SIGWINCH 28	/* window size changes */
#define SIGINFO	29	/* information request */
#endif
#define SIGUSR1 30	/* user defined signal 1 */
#define SIGUSR2 31	/* user defined signal 2 */
```

### 信号的传递
一个信号传递到目标进程有两个不同的步骤组成：

**发送信号**。内核通过更新目的进程上下文中的某个状态，发送一个信号给目的进程。发送信号可以有一下两种原因：

- 内核检测到一个系统事件，例如除零错误或者子进程终止；
- 一个进程调用了 Kill 函数，显示地要求内核发送一个信号给目的进程。

**接受信号**。当目的进程被内核强迫以某种方式对信号的发送做出反应时，它距接收了信号。进程可以忽略这个信号，终止或者执行一个称为信号处理程序（signal handler）的用户层函数捕获这个信号。

### 接收信号
当内核把进程 p 从内核模式切换到用户模式时，它会检查进程 p 的未被阻塞的待处理信号的集合。如果这个集合为空，那么内核讲控制传递到 p 的逻辑控制流中的下一条指令。然而，如果集合是非空的，那么内核选择集合中的某个信号 k（通常是最小的 k），并且强制 p 接收信号 k。收到这个信号会触发进程采取某种行为。一旦进程完成了这个行为，那么控制就传递回 p 的逻辑控制流中的下一条指令。每个信号类型都有一个与定义的默认行为：

- 进程终止
- 进程终止并转储内存
- 进程停止（挂起）直到被 SIGCONT 信号重启
- 进程忽略该信号

进程可以通过使用 signal 函数修改和信号相关联的默认行为。唯一的例外是 SIGSTOP 和 SIGKILL，它们的默认行为是不能修改的。

那么怎么修改呢？signal 函数可以通过下列三种方法之一来改变和信号 signum 相关联的行为：

- 如果 handler 是 SIG_IGN，那么忽略类型为 signum 的信号
- 如果 handler 是 SIG_DFL，那么类型为 signum 的信号行为恢复为默认行为
- 否则，handler 就是用户定义的函数的地址，这个函数被称为信号处理程序，只要进程接收到一个类型为 signum 的信号，就会调用这个程序。通过处理程序的地址传递到 signal 函数从而改变默认行为。这被称为信号处理程序（instlalling the handler）。调用信号处理程序被称为捕获信号。执行信号处理程序被称为处理信号。

当一个进程捕获来一个类型为 k 的信号时，会调用为信号 k 设置的处理程序，一个蒸熟参数被设置为 k。这个参数允许同一个处理函数捕获不同类型的信号。

至此，了解到的这些操作系统知识，已经足够支持我们对系统层的崩溃信号进行捕获了，接下就到了 Coding time。

### 捕获崩溃信号

```c

```





