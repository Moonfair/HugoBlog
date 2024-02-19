---
title: "Linux 信号详解"
date: 2024-01-25T15:03:43+08:00
draft: true
tags: 
    - 转载
---

> 原文地址: https://www.cnblogs.com/JohnABC/p/4530777.html

## Linux支持的所有信号

```shell
$ kill -l

 1) SIGHUP     2) SIGINT     3) SIGQUIT     4) SIGILL     5) SIGTRAP
 6) SIGABRT     7) SIGBUS     8) SIGFPE     9) SIGKILL    10) SIGUSR1
11) SIGSEGV    12) SIGUSR2    13) SIGPIPE    14) SIGALRM    15) SIGTERM
16) SIGSTKFLT    17) SIGCHLD    18) SIGCONT    19) SIGSTOP    20) SIGTSTP
21) SIGTTIN    22) SIGTTOU    23) SIGURG    24) SIGXCPU    25) SIGXFSZ
26) SIGVTALRM    27) SIGPROF    28) SIGWINCH    29) SIGIO    30) SIGPWR
31) SIGSYS    34) SIGRTMIN    35) SIGRTMIN+1    36) SIGRTMIN+2    37) SIGRTMIN+3
38) SIGRTMIN+4    39) SIGRTMIN+5    40) SIGRTMIN+6    41) SIGRTMIN+7    42) SIGRTMIN+8
43) SIGRTMIN+9    44) SIGRTMIN+10    45) SIGRTMIN+11    46) SIGRTMIN+12    47) SIGRTMIN+13
48) SIGRTMIN+14    49) SIGRTMIN+15    50) SIGRTMAX-14    51) SIGRTMAX-13    52) SIGRTMAX-12
53) SIGRTMAX-11    54) SIGRTMAX-10    55) SIGRTMAX-9    56) SIGRTMAX-8    57) SIGRTMAX-7
58) SIGRTMAX-6    59) SIGRTMAX-5    60) SIGRTMAX-4    61) SIGRTMAX-3    62) SIGRTMAX-2
63) SIGRTMAX-1    64) SIGRTMAX
```

## 信号描述及默认处理方式
|信号值|枚举值|默认处理动作|发出信号的原因|
|---|---|---|---|
|SIGHUP| 1| A| 终端挂起或者控制进程终止(hang up)|
|SIGINT| 2| A| 键盘中断（如break键被按下）(interrupt)|
|SIGQUIT| 3| C| 键盘的退出键被按下(quit)|
|SIGILL| 4| C| 非法指令(illegal)|
|SIGABRT| 6| C| 由abort(3)发出的退出指令(abort)|
|SIGFPE| 8| C| 浮点异常|
|SIGKILL| 9| AEF| Kill信号|
|SIGSEGV| 11| C| 无效的内存引用|
|SIGPIPE| 13| A| 管道破裂: 写一个没有读端口的管道|
|SIGALRM| 14| A| 由alarm(2)发出的信号|
|SIGTERM| 15| A| 终止信号|
|SIGUSR1| 30,10,16| A| 用户自定义信号1|
|SIGUSR2| 31,12,17| A| 用户自定义信号2|
|SIGCHLD| 20,17,18| B| 子进程结束信号|
|SIGCONT| 19,18,25| 进程继续|（曾被停止的进程）|
|SIGSTOP| 17,19,23| DEF| 终止进程|
|SIGTSTP| 18,20,24| D| 控制终端（tty）上按下停止键|
|SIGTTIN| 21,21,26| D| 后台进程企图从控制终端读|
|SIGTTOU| 22,22,27| D| 后台进程企图从控制终端写|

**处理动作一项中的字母含义如下:**
|||
|---|---|
|A| 缺省的动作是终止进程|
|B| 缺省的动作是忽略此信号，将该信号丢弃，不做处理|
|C| 缺省的动作是终止进程并进行内核映像转储（dump core），内核映像转储是指将进程数据在内存的映像和进程在内核结构中的部分内容以一定格式转储到文件系统，并且进程退出执行，这样做的好处是为程序员 提供了方便，使得他们可以得到进程当时执行时的数据值，允许他们确定转储的原因，并且可以调试他们的程序。|
|D| 缺省的动作是停止进程，进入停止状况以后还能重新进行下去，一般是在调试的过程中（例如ptrace系统调用）|
|E| 信号不能被捕获|
|F| 信号不能被忽略|

## 进程对信号的三种处理方式

1. 无视(ignore)信号，信号被清除，进程本身不采取任何特殊的操作

2. 默认(default)操作。每个信号对应有一定的默认操作。比如上面SIGCONT用于继续进程。

3. 自定义操作。也叫做获取 (catch) 信号。执行进程中预设的对应于该信号的操作。

## 信号小于SIGRTMIN的细致描述

#### 1) SIGHUP
本信号在用户终端连接(正常或非正常)结束时发出, 通常是在终端的控制进程结束时, 通知同一session内的各个作业, 这时它们与控制终端不再关联。

登录Linux时，系统会分配给登录用户一个终端(Session)。在这个终端运行的所有程序，包括前台进程组和后台进程组，一般都属于这个Session。当用户退出Linux登录时，前台进程组和后台有对终端输出的进程将会收到SIGHUP信号, 即会将SIGHUB信号传递给所有shell启动的进程。这个信号的默认操作为终止进程，因此前台进程组和后台有终端输出的进程就会中止。不过可以捕获这个信号，比如wget能捕获SIGHUP信号，并忽略它，这样就算退出了Linux登录，wget也能继续下载。

此外，对于与终端脱离关系的守护进程，这个信号用于通知它重新读取配置文件。

#### 2) SIGINT
程序终止(interrupt)信号, 在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。

#### 3) SIGQUIT
和SIGINT类似, 但由QUIT字符(通常是Ctrl-/)来控制. 进程在因收到SIGQUIT退出时会产生core文件, 在这个意义上类似于一个程序错误信号。

#### 4) SIGILL
执行了非法指令. 通常是因为可执行文件本身出现错误, 或者试图执行数据段. 堆栈溢出时也有可能产生这个信号。

#### 5) SIGTRAP
由断点指令或其它trap指令产生. 由debugger使用。

#### 6) SIGABRT
调用abort函数生成的信号。

#### 7) SIGBUS
非法地址, 包括内存地址对齐(alignment)出错。比如访问一个四个字长的整数, 但其地址不是4的倍数。它与SIGSEGV的区别在于后者是由于对合法存储地址的非法访问触发的(如访问不属于自己存储空间或只读存储空间)。

#### 8) SIGFPE
在发生致命的算术运算错误时发出. 不仅包括浮点运算错误, 还包括溢出及除数为0等其它所有的算术的错误。

#### 9) SIGKILL
用来立即结束程序的运行. 本信号不能被阻塞、处理和忽略。如果管理员发现某个进程终止不了，可尝试发送这个信号。

#### 10) SIGUSR1
留给用户使用

#### 11) SIGSEGV
试图访问未分配给自己的内存, 或试图往没有写权限的内存地址写数据.

#### 12) SIGUSR2
留给用户使用

#### 13) SIGPIPE
管道破裂。这个信号通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止就往管道写，写进程会收到SIGPIPE信号。此外用Socket通信的两个进程，写进程在写Socket的时候，读进程已经终止。

#### 14) SIGALRM
时钟定时信号, 计算的是实际的时间或时钟时间. alarm函数使用该信号.

#### 15) SIGTERM
程序结束(terminate)信号, 与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。

#### 17) SIGCHLD
子进程结束时, 父进程会收到这个信号。

如果父进程没有处理这个信号，也没有等待(wait)子进程，子进程虽然终止，但是还会在内核进程表中占有表项，这时的子进程称为僵尸进程。这种情况我们应该避免(父进程或者忽略SIGCHILD信号，或者捕捉它，或者wait它派生的子进程，或者父进程先终止，这时子进程的终止自动由init进程来接管)。

#### 18) SIGCONT
让一个停止(stopped)的进程继续执行. 本信号不能被阻塞. 可以用一个handler来让程序在由stopped状态变为继续执行时完成特定的工作. 例如, 重新显示提示符

#### 19) SIGSTOP
停止(stopped)进程的执行. 注意它和terminate以及interrupt的区别:该进程还未结束, 只是暂停执行. 本信号不能被阻塞, 处理或忽略.

#### 20) SIGTSTP
停止进程的运行, 但该信号可以被处理和忽略. 用户键入SUSP字符时(通常是Ctrl-Z)发出这个信号

#### 21) SIGTTIN
当后台作业要从用户终端读数据时, 该作业中的所有进程会收到SIGTTIN信号. 缺省时这些进程会停止执行.

#### 22) SIGTTOU
类似于SIGTTIN, 但在写终端(或修改终端模式)时收到.

#### 23) SIGURG
有"紧急"数据或out-of-band数据到达socket时产生.

#### 24) SIGXCPU
超过CPU时间资源限制. 这个限制可以由getrlimit/setrlimit来读取/改变。

#### 25) SIGXFSZ
当进程企图扩大文件以至于超过文件大小资源限制。

#### 26) SIGVTALRM
虚拟时钟信号. 类似于SIGALRM, 但是计算的是该进程占用的CPU时间.

#### 27) SIGPROF
类似于SIGALRM/SIGVTALRM, 但包括该进程用的CPU时间以及系统调用的时间.

#### 28) SIGWINCH
窗口大小改变时发出.

#### 29) SIGIO
文件描述符准备就绪, 可以开始进行输入/输出操作.

#### 30) SIGPWR
Power failure

#### 31) SIGSYS
非法的系统调用。

> 在以上列出的信号中，**程序不可捕获、阻塞或忽略**的信号有：SIGKILL,SIGSTOP
>
> **不能恢复至默认动作**的信号有:
> SIGILL, SIGTRAP
>
> **默认会导致进程流产**的信号有:
> SIGABRT, SIGBUS, SIGFPE, SIGILL, SIGIOT, SIGQUIT, SIGSEGV, SIGTRAP, SIGXCPU, SIGXFSZ
>
> **默认会导致进程退出**的信号有: 
> SIGALRM, SIGHUP, SIGINT, SIGKILL, SIGPIPE, SIGPOLL, SIGPROF, SIGSYS, SIGTERM, SIGUSR1, SIGUSR2, SIGVTALRM
>
> **默认会导致进程停止**的信号有:
> SIGSTOP, SIGTSTP, SIGTTIN, SIGTTOU
>
> **默认进程忽略**的信号有:
> SIGCHLD, SIGPWR, SIGURG, SIGWINCH

## 在SHELL中使用信号 实例

```shell
$ ping localhost
# 此时我们可以通过CTRL+Z来将SIGTSTP传递给该进程。shell中显示
[1]+  Stopped                 ping localhost

# 我们使用$ps来查询ping进程的PID (PID是ping进程的房间号), 在我的机器中为27397
# 我们可以在shell中通过$kill命令来向某个进程发出信号:
$ kill -SIGCONT  27397
# SIGCONT 表示让此暂停的进程继续执行
```