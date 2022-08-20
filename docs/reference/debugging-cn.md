***

## 标题：“调试”&#xA;链接标题：“调试”&#xA;体重： 1&#xA;描述： >&#xA;调试 Redis 服务器进程的指南&#xA;别名：&#xA;\- /主题/调试

Redis的开发重点是稳定性。我们竭尽所能
每个版本，以确保您将体验到稳定的产品，没有
崩溃。但是，如果您需要调试 Redis 进程本身，请继续阅读。

当 Redis 崩溃时，它会生成有关所发生情况的详细报告。然而
有时查看崩溃报告是不够的，也不可能
Redis 核心团队独立重现问题。在这种情况下，我们
需要可以重现问题的用户的帮助。

本指南介绍如何使用 GDB 提供
Redis开发人员需要更轻松地跟踪错误。

## 什么是GDB？

GDB是Gnu调试器：一个能够检查内部状态的程序
另一个程序。通常，跟踪和修复错误是一项练习
收集有关当前程序状态的更多信息
错误发生了，所以GDB是一个非常有用的工具。

GDB 可以通过两种方式使用：

*   它可以附加到正在运行的程序，并在运行时检查其状态。
*   它可以检查已使用所谓的程序终止的程序的状态*核心文件*，即程序运行时的内存映像。

从调查 Redis 错误的角度来看，我们需要同时使用这两个
GDB 模式。能够重现 bug 的用户会将 GDB 附加到其正在运行的 Redis 上
实例，当崩溃发生时，它们会创建`core`文件，依次
开发人员将用于在崩溃时检查 Redis 内部。

这样，开发人员就可以在他或她的计算机中执行所有检查
无需用户帮助，用户可以自由地重新启动 Redis
生产环境。

## 编译 Redis 而不进行优化

默认情况下，Redis 使用`-O2`开关，这意味着编译器
优化已启用。这使得 Redis 可执行速度更快，但
同时，它使Redis（像任何其他程序一样）更难使用GDB进行检查。

最好将 GDB 附加到 Redis 编译，而不使用
`make noopt`命令（而不是仅使用普通命令`make`命令）。然而
如果您在生产环境中已经运行了 Redis，则无需重新编译
并重新启动它，如果这会给您带来问题。GDB仍然有效
针对使用优化编译的可执行文件。

您不应该过分担心编译 Redis 会降低性能
没有优化。这不太可能导致您的问题
环境，因为 Redis 不受 CPU 限制。

## 将 GDB 附加到正在运行的进程

如果您有一个已经在运行的 Redis 服务器，则可以将 GDB 附加到它，以便
如果 Redis 崩溃，则可以检查内部并生成
一个`core dump`文件。

将 GDB 附加到 Redis 进程后，它将继续照常运行，而无需
任何性能损失，因此这不是一个危险的过程。

为了附加GDB，您需要的第一件事是*进程标识*的运行
Redis 实例（*皮德*的过程）。您可以使用以下命令轻松获取它
`redis-cli`:

    $ redis-cli info | grep process_id
    process_id:58414

在上面的示例中，进程 ID 为**58414**.

登录到您的 Redis 服务器。

（可选，但建议使用）开始**屏幕**或**断续器**或任何其他程序，将确保您的GDB会话不会关闭，如果您的ssh连接超时。您可以在[本文](http://www.linuxjournal.com/article/6340).

通过键入以下内容将 GDB 附加到正在运行的 Redis 服务器：

    $ gdb <path-to-redis-executable> <pid>

例如：

    $ gdb /usr/local/bin/redis-server 58414

GDB 将启动并附加到正在运行的服务器，打印如下内容：

    Reading symbols for shared libraries + done
    0x00007fff8d4797e6 in epoll_wait ()
    (gdb)

此时，GDB已附加，但**您的 Redis 实例被 GDB 阻止**.在
为了让 Redis 实例继续执行，只需键入**继续**在
GDB 提示符，然后按回车键。

    (gdb) continue
    Continuing.

做！现在，您的 Redis 实例已附加 GDB。现在，您可以等待下一次崩溃。:)

现在是时候分离您的屏幕/ tmux会话了，如果您正在使用它运行GDB，则通过
紧迫**Ctrl-a a**组合键。

## 坠机后

Redis 有一个命令来模拟分段错误（换句话说，一个糟糕的崩溃），使用
这`DEBUG SEGFAULT`命令（当然，不要对实际生产实例使用它！
因此，我将使用此命令使我的实例崩溃，以显示GDB端发生的情况：

    (gdb) continue
    Continuing.

    Program received signal EXC_BAD_ACCESS, Could not access memory.
    Reason: KERN_INVALID_ADDRESS at address: 0xffffffffffffffff
    debugCommand (c=0x7ffc32005000) at debug.c:220
    220         *((char*)-1) = 'x';

如您所见，GDB检测到Redis崩溃了，甚至能够向我展示
导致崩溃的文件名和行号。这已经好多了
比 Redis 崩溃报告回溯跟踪（仅包含函数名称和
二进制偏移量）。

## 获取堆栈跟踪

首先要做的是使用GDB获取完整的堆栈跟踪。这是作为
使用简单**断续器**命令：

    (gdb) bt
    #0  debugCommand (c=0x7ffc32005000) at debug.c:220
    #1  0x000000010d246d63 in call (c=0x7ffc32005000) at redis.c:1163
    #2  0x000000010d247290 in processCommand (c=0x7ffc32005000) at redis.c:1305
    #3  0x000000010d251660 in processInputBuffer (c=0x7ffc32005000) at networking.c:959
    #4  0x000000010d251872 in readQueryFromClient (el=0x0, fd=5, privdata=0x7fff76f1c0b0, mask=220924512) at networking.c:1021
    #5  0x000000010d243523 in aeProcessEvents (eventLoop=0x7fff6ce408d0, flags=220829559) at ae.c:352
    #6  0x000000010d24373b in aeMain (eventLoop=0x10d429ef0) at ae.c:397
    #7  0x000000010d2494ff in main (argc=1, argv=0x10d2b2900) at redis.c:2046

这显示了回溯，但我们也希望使用**信息寄存器**命令：

    (gdb) info registers
    rax            0x0  0
    rbx            0x7ffc32005000   140721147367424
    rcx            0x10d2b0a60  4515891808
    rdx            0x7fff76f1c0b0   140735188943024
    rsi            0x10d299777  4515796855
    rdi            0x0  0
    rbp            0x7fff6ce40730   0x7fff6ce40730
    rsp            0x7fff6ce40650   0x7fff6ce40650
    r8             0x4f26b3f7   1327936503
    r9             0x7fff6ce40718   140735020271384
    r10            0x81 129
    r11            0x10d430398  4517462936
    r12            0x4b7c04f8babc0  1327936503000000
    r13            0x10d3350a0  4516434080
    r14            0x10d42d9f0  4517452272
    r15            0x10d430398  4517462936
    rip            0x10d26cfd4  0x10d26cfd4 <debugCommand+68>
    eflags         0x10246  66118
    cs             0x2b 43
    ss             0x0  0
    ds             0x0  0
    es             0x0  0
    fs             0x0  0
    gs             0x0  0

请**确保包括**这两个输出都在错误报告中。

## 获取核心文件

下一步是生成核心转储，即正在运行的 Redis 进程的内存映像。这是使用`gcore`命令：

    (gdb) gcore
    Saved corefile core.58414

现在，您有核心转储可以发送给 Redis 开发人员，但是**这很重要
了解**这恰好包含
崩溃时的 Redis 实例;Redis开发人员将确保不这样做
与其他人共享内容，并在文件没有时立即删除该文件
用于调试目的的时间更长，但通过发送内核，您会被警告
文件，您正在发送数据。

## 发送给开发人员的内容

最后，您可以将所有内容发送给 Redis 核心团队：

*   您正在使用的 Redis 可执行文件。
*   生成的堆栈跟踪**断续器**命令，然后寄存器转储。
*   使用 gdb 生成的核心文件。
*   有关操作系统和 GCC 版本以及您正在使用的 Redis 版本的信息。

## 谢谢

您的帮助非常重要！许多问题只能通过这种方式进行跟踪。所以
谢谢！
