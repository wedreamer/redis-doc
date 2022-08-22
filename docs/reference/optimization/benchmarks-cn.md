
Redis 包括`redis-benchmark`模拟正在运行的命令完成的实用程序
通过 N 个客户端同时发送 M 个查询。该实用程序提供
可以提供一组默认测试, 也可以提供一组自定义测试。

支持以下选项：

    Usage: redis-benchmark [-h <host>] [-p <port>] [-c <clients>] [-n <requests]> [-k <boolean>]

     -h <hostname>      Server hostname (default 127.0.0.1)
     -p <port>          Server port (default 6379)
     -s <socket>        Server socket (overrides host and port)
     -a <password>      Password for Redis Auth
     -c <clients>       Number of parallel connections (default 50)
     -n <requests>      Total number of requests (default 100000)
     -d <size>          Data size of SET/GET value in bytes (default 2)
     --dbnum <db>       SELECT the specified db number (default 0)
     -k <boolean>       1=keep alive 0=reconnect (default 1)
     -r <keyspacelen>   Use random keys for SET/GET/INCR, random values for SADD
      Using this option the benchmark will expand the string __rand_int__
      inside an argument with a 12 digits number in the specified range
      from 0 to keyspacelen-1. The substitution changes every time a command
      is executed. Default tests use this to hit random keys in the
      specified range.
     -P <numreq>        Pipeline <numreq> requests. Default 1 (no pipeline).
     -q                 Quiet. Just show query/sec values
     --csv              Output in CSV format
     -l                 Loop. Run the tests forever
     -t <tests>         Only run the comma separated list of tests. The test
                        names are the same as the ones produced as output.
     -I                 Idle mode. Just open N idle connections and wait.

在启动基准测试之前, 您需要有一个正在运行的 Redis 实例。
您可以像这样运行基准测试实用程序：

    redis-benchmark -q -n 100000

### 仅运行测试的子集

您不需要在每次执行时都运行所有默认测试`redis-benchmark`.
例如, 若要仅选择测试的子集, 请使用`-t`选择
如以下示例所示：

    $ redis-benchmark -t set,lpush -n 100000 -q
    SET: 74239.05 requests per second
    LPUSH: 79239.30 requests per second

此示例运行`SET`和`LPUSH`命令并使用安静模式 (请参阅`-q`开关) 。

您甚至可以对特定命令进行基准测试：

    $ redis-benchmark -n 100000 -q script load "redis.call('set','foo','bar')"
    script load redis.call('set','foo','bar'): 69881.20 requests per second

### 选择密钥空间的大小

默认情况下, 基准测试针对单个键运行。在Redis中, 区别
在这样的综合基准和真实的基准之间并不大, 因为它是一个
内存中系统, 但是可以强调缓存未命中, 并且通常
通过使用较大的密钥空间来模拟更真实的工作负载。

这是通过使用`-r`开关。例如, 如果我想运行
一百万个 SET 操作, 对出的每个操作使用一个随机键
100k可能的键, 我将使用以下命令行：

    $ redis-cli flushall
    OK

    $ redis-benchmark -t set -r 100000 -n 1000000
    ====== SET ======
      1000000 requests completed in 13.86 seconds
      50 parallel clients
      3 bytes payload
      keep alive: 1

    99.76% `<=` 1 milliseconds
    99.98% `<=` 2 milliseconds
    100.00% `<=` 3 milliseconds
    100.00% `<=` 3 milliseconds
    72144.87 requests per second

    $ redis-cli dbsize
    (integer) 99993

### 使用流水线

默认情况下, 每个客户端 (如果不是其他情况, 则基准测试模拟 50 个客户端) 
指定为`-c`)  仅当上一个命令的回复时才发送下一个命令
命令被接收, 这意味着服务器可能需要读取调用
以便从每个客户端读取每个命令。RTT也是付费的。

Redis 支持[流水线](/topics/pipelining), 因此可以发送
一次多个命令, 这是现实世界应用程序经常利用的功能。
Redis 流水线能够显著提高每个操作的操作数
其次, 服务器能够交付。

这是在MacBook Air 11“中使用
16 个命令的流水线：

    $ redis-benchmark -n 1000000 -t set,get -P 16 -q
    SET: 403063.28 requests per second
    GET: 508388.41 requests per second

使用流水线可显著提高性能。

### 陷阱和误解

第一点是显而易见的：一个有用的基准的黄金法则是
只比较苹果和苹果。可以比较不同版本的 Redis
例如, 在相同的工作负载上。或者相同版本的Redis, 但带有
不同的选项。如果您打算将Redis与其他产品进行比较, 那么它是
重要的是评估功能和技术差异, 并采取它们
在帐户中。

*   Redis是一个服务器：所有命令都涉及网络或IPC往返。将其与嵌入式数据存储进行比较是没有意义的, 因为大多数操作的成本主要在于网络/协议管理。
*   Redis 命令返回所有常用命令的确认。将 Redis 与涉及单向查询的存储进行比较只是稍微有用。
*   天真地迭代同步 Redis 命令不会对 Redis 本身进行基准测试, 而是测量网络 (或 IPC) 延迟和客户端库固有延迟。要真正测试 Redi, , 您需要多个连接 (如 redis-benchmark) 和/或使用流水线来聚合多个命令和/或多个线程或进程。
*   Redis 是一个内存中数据存储, 具有一些可选的持久性选项。如果您计划将其与事务服务器 (MySQL, PostgreSQL等) 进行比, , 那么您应该考虑激活AOF并确定合适的fsync策略。
*   Redis主要是来自命令执行POV的单线程服务器 (实际上, Redis的现代版本将线程用于不同的事情) 。它不是为了从多个 CPU 内核中受益而设计的。如果需, , 人们应该启动几个Redis实, , 以便在多个内核上进行横向扩展。将单个 Redis 实例与多线程数据存储进行比较是不公平的。

这`redis-benchmark`程序是获取一些数字和
评估给定硬件上 Redis 实例的性能。然而
默认情况下, 它不表示 Redis 实例可以达到的最大吞吐量
维持。实际上, 通过使用流水线和快速客户端 (hiredis), , 它是相当公平的
易于编写一个程序, 生成比 redis 基准测试更多的吞吐量。这
redis 基准测试的默认行为是通过利用
仅并发 (即它创建与服务器的多个连接) 。
它根本不使用流水线或任何并行性 (每个
最多连接, 并且没有多线程), , 如果未通过
这`-P`参数。所以以某种方式使用`redis-benchmark`并且, 触发, 
例如, `BGSAVE`在后台操作的同时, 将提供
数字更靠近的用户*最坏情况*而不是最好的情况。

要使用流水线模式运行基准测试 (并实现更高的吞吐量), , 
您需要显式使用 -P 选项。请注意, 它仍然是一个
逼真的行为, 因为许多基于Redis的应用程序都在积极使用
流水线以提高性能。但是, 应使用管道大小
或多或少是您可以在您的
应用程序以获得现实的数字。

基准测试应应用相同的操作, 并以相同的方式工作
与要比较的多个数据存储。这绝对毫无意义
将 redis 基准测试的结果与另一个基准测试的结果进行比较
程序和外推。

例如, Redis 和 memcached 在单线程模式下可以进行比较
获取/设置操作。两者都是内存数据存储, 主要在同一环境中工作
在协议级别的方式。前提是他们各自的基准测试应用程序是
以相同的方式聚合查询 (流水线), , 并使用相似数量的
连接, 比较其实是有意义的。

当您对像 Redis 这样的高性能内存中数据库进行基准测试时, 
可能难以饱和
服务器。有时, 性能瓶颈位于客户端, 
而不是服务器端。在这种情况下, 客户端 (即基准测试程序本身) 
必须固定或横向扩展, 以达到最大吞吐量。

### 影响 Redis 性能的因素

有多种因素会对 Redis 性能产生直接影响。
我们在这里提到它们, 因为它们可以改变任何基准测试的结果。
但请注意, 在低端运行的典型 Redis 实例, 
未调谐的 Box 通常可为大多数应用程序提供足够好的性能。

*   网络带宽和延迟通常对性能有直接影响。
    使用ping程序快速检查延迟是一种很好的做法
    在启动基准测试之前, 客户端和服务器主机之间是正常的。
    关于带宽, 估计通常很有用
    吞吐量 (以 Gbit/s 为单位), , 并将其与理论带宽进行比较
    的网络。例如, 基准设置 4 KB 字符串
    在 Redis 中, 在 100000 q/s 时, 实际上会消耗 3.2 Gbit/s 的带宽
    并且可能适合 10 Gbit/s 链路, 但不适合 1 Gbit/s 链路。在许多真实
    世界场景, Redis吞吐量在被网络限制之前
    受 CPU 的限制。整合多个高吞吐量 Redis 实例
    在单个服务器上, 值得考虑放置一个 10 Gbit/s 的 NIC
    或多个具有 TCP/IP 绑定的 1 Gbit/s 网卡。
*   CPU是另一个非常重要的因素。作为单线程, Redis青睐
    具有大型缓存且内核不多的快速 CPU。在本场比赛中, 英特尔CPU是
    目前是获奖者。性能只有一半的情况并不少见
    AMD皓龙CPU与类似的Nehalem EP /Westmere EP / Sandy Bridge的比较
    带有 Redis 的英特尔 CPU。当客户端和服务器在同一机箱上运行时, CPU 是
    具有重新定位基准的限制因素。
*   RAM 速度和内存带宽对于全局性能似乎不那么重要
    特别是对于小物体。对于大型对象  (>10 KB), , 它可能会变为
    不过很明显。通常, 购买昂贵的价格并不划算
    快速内存模块, 用于优化 Redis。
*   与不使用虚拟化运行相比, Redis 在 VM 上运行的速度更慢
    相同的硬件。如果您有机会在物理机器上运行 Redis
    这是首选。然而, 这并不意味着Redis在
    虚拟化环境, 交付的性能仍然非常好
    以及您在虚拟化中可能遇到的大多数严重性能问题
    环境是由于过度配置、具有高延迟的非本地磁盘, 
    或速度慢的旧虚拟机管理程序软件`fork`系统调用实现。
*   当服务器和客户端基准测试程序在同一个盒子上运行时, 两者
    可以使用 TCP/IP 环回和 unix 域套接字。取决于
    平台, unix域套接字可以实现比 50% 多出约 50% 的吞吐量
    TCP /IP环回 (例如在Linux上) 。的默认行为
    redis-benchmark 是使用 TCP/IP 环回。
*   与 TCP/IP 环回相比, unix 域套接字的性能优势
    当大量使用流水线 (即长管道) , , 往往会减少。
*   当使用以太网网络访问 Redis 时, 使用
    当数据的大小保持在
    以太网数据包大小 (约 1500 字节) 。实际, , 处理 10 个字, , 
    100 字节或 1000 字节查询几乎会产生相同的吞吐量。
    请参阅下图。

![Data size impact](https://github.com/dspezia/redis-doc/raw/client_command/topics/Data_size.png)

*   在多 CPU 插槽服务器上, Redis 性能取决于
    NUMA 配置和进程位置。最明显的效果是
    redis 基准测试结果似乎不是确定性的, 因为客户端和服务器
    进程随机分布在内核上。要获得确定性结果, 
    它需要使用进程放置工具 (在Linux上：taskset或numactl) 。
    最有效的组合始终是将客户端和服务器放在两个上
    同一 CPU 的不同内核可从 L3 高速缓存中受益。
    以下是 3 个服务器 CPU 的 4 KB SET 基准测试的一些结果 (AMD 伊斯坦布尔, 
    Intel Nehalem EX 和 Intel Westmere) 具有不同的相对位置。
    请注意, 此基准测试并非旨在比较它们之间的CPU型号
     (因此, CPU的确切型号和频率未公开) 。

![NUMA chart](https://github.com/dspezia/redis-doc/raw/6374a07f93e867353e5e946c1e39a573dfc83f6c/topics/NUMA_chart.gif)

*   对于高端配置, 客户端连接数也是
    重要因素。基于 epoll/kqueue, Redis 事件循环非常丰富
    可 伸缩。Redis 已经被基准测试为超过 60000 个连接, 
    并且在这些条件下仍然能够维持50000 q / s。根据经验, 
    具有 30000 个连接的实例只能处理一半的吞吐量
    可通过 100 个连接实现。下面是一个示例, 显示了
    每个连接数的 Redis 实例：

![connections chart](https://github.com/dspezia/redis-doc/raw/system_info/topics/Connections_chart.png)

*   通过高端配置, 可以通过以下方式实现更高的吞吐量
    调整网卡配置和相关中断。最佳吞吐量
    通过在 Rx/Tx NIC 队列和 CPU 内核之间设置关联来实现, 
    并激活 RPS (接收数据包控制) 支持。更多信息请见此
    [线](https://groups.google.com/forum/#!msg/redis-db/gUhc19gnYgc/BruTPCOroiMJ).
    当使用大型对象时, 巨型帧还可以提供性能提升。
*   根据平台的不同, Redis可以针对不同的内存进行编译
    分配器 (libc malloc, jemalloc, tcmalloc), , 它们可能具有不同的行为
    在原始速度, 内部和外部碎片化方面。
    如果您没有自己编译 Redis, 则可以使用 INFO 命令检查
    这`mem_allocator`田。请注意, 大多数基准测试的运行时间不够长
    产生显著的外部碎片 (与生产 Redis 相反) 
    实例) 。

### 其他需要考虑的事项

任何基准测试的一个重要目标都是获得可重复的结果, 因此它们
可以与其他测试的结果进行比较。

*   一个好的做法是尝试尽可能多地在隔离的硬件上运行测试。
    如果不可能, 则必须监视系统以检查基准测试
    不受某些外部活动的影响。
*   某些配置 (当然是台式机和笔记本电脑, 也有一些服务器) 
    具有可变 CPU 内核频率机制。控制此情况的策略
    可以在操作系统级别设置机制。某些 CPU 型号比
    其他人则根据工作负载调整CPU内核的频率。要获得
    结果可重复, 最好设置尽可能高的固定频率
    适用于基准测试中涉及的所有 CPU 内核。
*   重要的一点是要根据基准测试来调整系统的大小。
    系统必须有足够的 RAM, 并且不能换用。在 Linux 上, 不要忘记
    以设置`overcommit_memory`参数正确。请注意 32 位和 64 位
    Redis 实例不具有相同的内存占用量。
*   如果您计划使用 RDB 或 AOF 作为基准测试, 请检查没有其他
    系统中的 I/O 活动。避免将 RDB 或 AOF 文件放在 NAS 或 NFS 共享上, 
    或在影响网络带宽和/或延迟的任何其他设备上
     (例如, Amazon EC2 上的 EBS) 。
*   将 Redis 日志记录级别 (日志级别参数) 设置为警告或通知。避免放置
    远程文件系统上生成的日志文件。
*   避免使用可能改变基准测试结果的监控工具。为
    实例定期使用 INFO 来收集统计信息可能很好, 
    但 MONITOR 将显著影响测量的性能。

### 其他 Redis 基准测试工具

有几个第三方工具可用于对 Redis 进行基准测试。请参阅每个工具的
文档, 了解有关其目标和功能的更多信息。

*   [memtier_benchmark](https://github.com/redislabs/memtier_benchmark)从[瑞迪斯有限公司](https://twitter.com/RedisInc)是一个NoSQL Redis和Memcache流量生成和基准测试工具。
*   [rpc-perf](https://github.com/twitter/rpc-perf)从[唽](https://twitter.com/twitter)是用于对支持 Redis 和 Memcache 的 RPC 服务进行基准测试的工具。
*   [中国上海交响乐团](https://github.com/brianfrankcooper/YCSB)从[雅虎@Yahoo](https://twitter.com/Yahoo)是一个基准框架, 具有许多数据库 (包括Redis) 的客户端。
