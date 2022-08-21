## 填写性能清单

Redis的开发非常注重性能。我们竭尽所能
每个版本，以确保您将体验到非常稳定和快速的产品。

尽管如此，如果你正在寻找提高Redis效率的空间，或者
正在进行性能回归调查，您将需要一个简洁的调查
有条不紊地监控和分析 Redis 性能。

要做到这一点，你可以依靠不同的方法（有些比其他方法更合适）
取决于我们打算进行的问题/分析的类别）。精选列表
的方法及其步骤由Brendan Greg在
[以下链接](http://www.brendangregg.com/methodology.html).

我们建议使用利用率饱和度和错误 （USE） 方法来回答
你的瓶颈是什么的问题。检查以下映射
系统资源、指标和工具，用于实际深入探讨：
[使用方法](http://www.brendangregg.com/USEmethod/use-rosetta.html).

### 确保 CPU 成为您的瓶颈

本指南假定您已按照上述方法之一执行
全面检查系统运行状况，并确定瓶颈是 CPU。
**如果您已确定大部分时间都花在 I/O、锁定上，则锁定，
计时器，分页/交换等，本指南不适合您**.

### 构建先决条件

为了进行正确的 CPU 上分析，Redis（以及任何动态加载的库，如
Redis 模块）要求堆栈跟踪可供跟踪程序使用，您可以将其用于跟踪程序
需要先修复。

默认情况下，Redis 使用`-O2`开关（我们打算保留
在分析期间）。这意味着启用了编译器优化。多
编译器省略帧指针作为运行时优化（保存寄存器），
从而打破基于指针的帧堆栈行走。这使得Redis
可执行速度更快，但同时它使Redis（像任何其他程序一样）
更难跟踪，可能会错误地将 CPU 上时间精确定位到最后一个
调用堆栈的可用帧指针，可以变得更深（但是
无法追踪）。

请务必确保：

*   调试信息存在：编译选项`-g`
*   帧指针寄存器存在：`-fno-omit-frame-pointer`
*   我们仍然运行优化，以获得生产运行时间的准确表示，这意味着我们将保持：`-O2`

您可以在 redis 主存储库中按如下方式执行此操作：

    $ make REDIS_CFLAGS="-g -fno-omit-frame-pointer"

## 一组用于识别性能倒退和/或潜在因素的工具**CPU 上性能**改进

本文档特别关注**在中央处理器上**资源瓶颈分析，
这意味着我们有兴趣了解线程在何处花费 CPU 周期
在 CPU 上运行时，同样重要的是，这些周期是否有效
用于计算或停止等待（未阻止！）内存 I/O，
和缓存未命中等。

为此，我们将依靠工具包（性能，密件抄送工具）和特定于硬件的PMC。
（性能监视计数器），继续执行：

*   热点分析（pref 或 bcc 工具）：分析代码执行情况，并确定哪些函数消耗的时间最多，因此是优化的目标。我们将提供两个选项，以使用性能或密件抄送/BPF 跟踪工具收集、报告和可视化热点。

*   调用计数分析：对事件（包括函数调用）进行计数，使我们能够依靠密件抄送/BPF 跟踪工具一次关联多个调用/组件。

*   硬件事件采样：对于了解 CPU 行为（包括内存 I/O、停滞周期和缓存未命中）至关重要。

### 工具预要求

以下步骤依赖于 Linux perf_events（又名[“性能”](https://man7.org/linux/man-pages/man1/perf.1.html)),[密件抄送/BPF 跟踪工具](https://github.com/iovisor/bcc)和布伦丹·格雷格的[火焰图回购](https://github.com/brendangregg/FlameGraph).

我们事先假设您有：

*   已在系统上安装性能工具。大多数Linux发行版可能会将其打包为与内核相关的软件包。有关 perf 工具的更多信息，请访问 perf[维基](https://perf.wiki.kernel.org/).
*   已遵循安装[密件抄送/场效应基金](https://github.com/iovisor/bcc/blob/master/INSTALL.md#installing-bcc)在计算机上安装密件抄送工具包的说明。
*   克隆布伦丹格雷格的[火焰图回购](https://github.com/brendangregg/FlameGraph)并使`difffolded.pl`和`flamegraph.pl`文件，以生成折叠的堆栈跟踪和火焰图。

## 使用性能或 eBPF 进行热点分析（堆栈迹线采样）

通过按定时间隔对堆栈跟踪进行采样来分析 CPU 使用率是一种快速且
识别性能关键型代码段（热点）的简单方法。

### 使用性能对堆栈跟踪进行采样

为特定的 redis 服务器的用户级和内核级堆栈进行分析
时间长度，例如 60 秒，采样频率为 999 个样本
每秒：

    $ perf record -g --pid $(pgrep redis-server) -F 999 -- sleep 60

#### 使用性能报告显示记录的配置文件信息

默认情况下，性能记录将在当前工作中生成一个性能数据文件
目录。

然后，您可以使用调用图输出（调用链，堆栈回溯）进行报告，
最小调用图包含阈值为 0.5%，具有：

    $ perf report -g "graph,0.5,caller"

查看[性能报告](https://man7.org/linux/man-pages/man1/perf-report.1.html)
高级筛选、排序和聚合功能的文档。

#### 使用火焰图可视化记录的配置文件信息

[火焰图](http://www.brendangregg.com/flamegraphs.html)允许快速
以及频繁代码路径的准确可视化。它们可以通过以下方式生成
Brendan Greg 的开源程序 on[github](https://github.com/brendangregg/FlameGraph),
从折叠的堆栈文件创建交互式 SVG。

具体来说，对于perf，我们需要将生成的perf.data转换为
捕获堆栈，并将每个堆栈折叠成单行。然后，您可以渲染
CPU 上的火焰图包含：

    $ perf script > redis.perf.stacks
    $ stackcollapse-perf.pl redis.perf.stacks > redis.folded.stacks
    $ flamegraph.pl redis.folded.stacks > redis.svg

默认情况下，perf 脚本将在当前工作中生成一个 perf.data 文件
目录。查看[性能脚本](https://linux.die.net/man/1/perf-script)
高级用法的文档。

看[火焰图使用选项](https://github.com/brendangregg/FlameGraph#options)
用于更高级的堆栈跟踪可视化（如差分可视化）。

#### 存档和共享记录的配置文件信息

因此，可以在其他计算机上分析perf.data内容
比收集发生的那个，你需要导出与
perf.data 文件所有在记录数据文件中找到的具有 build-id 的对象文件。
这可以在以下方面轻松完成
[perf-archive.sh](https://github.com/torvalds/linux/blob/master/tools/perf/perf-archive.sh)
脚本：

    $ perf-archive.sh perf.data

现在请运行：

    $ tar xvf perf.data.tar.bz2 -C ~/.debug

在需要运行的计算机上`perf report`.

### 使用密件抄送/密保积的配置文件对堆栈跟踪进行采样

与perf类似，从Linux内核4.9开始，BPF优化的分析现在完全
提供，并承诺降低 CPU 开销（因为堆栈跟踪
在内核上下文中计数的频率）和性能分析期间的磁盘 I/O 资源。

除此之外，仅依靠密件抄送/ BPF的配置文件工具，我们还
删除了perf.data和中间步骤，如果堆栈跟踪分析是我们的
主要目标。您可以使用密件抄送的配置文件工具直接输出折叠格式，用于
火焰图生成：

    $ /usr/share/bcc/tools/profile -F 999 -f --pid $(pgrep redis-server) --duration 60 > redis.folded.stacks

通过这种方式，我们已经删除了任何预处理，并且可以渲染CPU上的火焰。
使用单个命令的图形：

    $ flamegraph.pl redis.folded.stacks > redis.svg

### 使用火焰图可视化记录的配置文件信息

## 使用密件抄送/BPF 进行呼叫计数分析

函数可能会消耗大量 CPU 周期，因为其代码速度很慢
或者因为它经常被调用。回答函数的速率
调用，您可以依靠使用BCC的呼叫计数分析`funccount`工具：

    $ /usr/share/bcc/tools/funccount 'redis-server:(call*|*Read*|*Write*)' --pid $(pgrep redis-server) --duration 60
    Tracing 64 functions for "redis-server:(call*|*Read*|*Write*)"... Hit Ctrl-C to end.

    FUNC                                    COUNT
    call                                      334
    handleClientsWithPendingWrites            388
    clientInstallWriteHandler                 388
    postponeClientRead                        514
    handleClientsWithPendingReadsUsingThreads      735
    handleClientsWithPendingWritesUsingThreads      735
    prepareClientToWrite                     1442
    Detaching...

上面的输出显示，在跟踪时，Redis 的 call（） 函数是
调用 334 次，处理客户端与挂起重写（） 388 次，依此类推。

## 使用性能监视计数器 （PMC） 进行硬件事件计数

许多现代处理器都包含一个性能监控单元 （PMU） 公开
性能监视计数器 （PMC）。PMC 对于了解 CPU 至关重要
行为，包括内存 I/O、停顿周期和缓存未命中，并提供
在其他任何地方都不可用的低级 CPU 性能统计信息。

PMU 的设计和功能是特定于 CPU 的，您应该评估
通过使用 CPU 支持的计数器和功能`perf list`.

要计算每个周期的指令数，微操作数
执行，在此期间没有调度微操作的周期数，
内存上停滞周期数，包括每个内存类型的停顿周期数，用于
持续时间为 60 秒，特别是对于 redis 过程：

    $ perf stat -e "cpu-clock,cpu-cycles,instructions,uops_executed.core,uops_executed.stall_cycles,cache-references,cache-misses,cycle_activity.stalls_total,cycle_activity.stalls_mem_any,cycle_activity.stalls_l3_miss,cycle_activity.stalls_l2_miss,cycle_activity.stalls_l1d_miss" --pid $(pgrep redis-server) -- sleep 60

    Performance counter stats for process id '3038':

      60046.411437      cpu-clock (msec)          #    1.001 CPUs utilized          
      168991975443      cpu-cycles                #    2.814 GHz                      (36.40%)
      388248178431      instructions              #    2.30  insn per cycle           (45.50%)
      443134227322      uops_executed.core        # 7379.862 M/sec                    (45.51%)
       30317116399      uops_executed.stall_cycles #  504.895 M/sec                    (45.51%)
         670821512      cache-references          #   11.172 M/sec                    (45.52%)
          23727619      cache-misses              #    3.537 % of all cache refs      (45.43%)
       30278479141      cycle_activity.stalls_total #  504.251 M/sec                    (36.33%)
       19981138777      cycle_activity.stalls_mem_any #  332.762 M/sec                    (36.33%)
         725708324      cycle_activity.stalls_l3_miss #   12.086 M/sec                    (36.33%)
        8487905659      cycle_activity.stalls_l2_miss #  141.356 M/sec                    (36.32%)
       10011909368      cycle_activity.stalls_l1d_miss #  166.736 M/sec                    (36.31%)

      60.002765665 seconds time elapsed

重要的是要知道，PMC可以通过两种截然不同的方式
使用（计数和采样），我们只关注 PMC 计数
为了这个分析。Brendan Greg在下面清楚地解释了这一点
[链接](http://www.brendangregg.com/blog/2017-05-04/the-pmcs-of-ec2.html).
