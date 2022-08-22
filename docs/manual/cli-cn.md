---
title: "Redis CLI"
linkTitle: "CLI"
weight: 1
description: >
    Overview of redis-cli, the Redis command line interface
aliases:
    - /docs/manual/cli
---

Redis 命令行界面  (`redis-cli`)  是一个终端程序, 用于向 Redis 服务器发送命令并从 Redis 服务器读取回复。它有两种主要模式：Read Eval Print Loop (REPL) 模式, 用户键入 Redis 命令并接收回复, 以及命令模式`redis-cli`使用其他参数执行, 并将回复打印到标准输出。

在交互模式下, `redis-cli` 具有基本的行编辑功能, 可提供熟悉的打字体验。

要以特殊模式启动程序, 可以使用多个选项, 包括：

*   模拟成一个副本并打印它从主数据库接收的复制流。
*   检查 Redis 服务器的延迟并显示统计信息。
*   请求延迟样本和频率的 ASCII-art 频谱图。

本主题涵盖`redis-cli`, 从最简单的开始, 以更高级的功能结束。

## 命令行用法

要运行 Redis 命令并在终端返回标准输出, 请将要执行的命令作为`redis-cli`:

    $ redis-cli INCR mycounter
    (integer) 7

该命令的回复为 "7"。由于 Redis 回复是键入的 (字符串、数组、整数、nil、错误等) , 因此您会在括号之间看到回复的类型。当输出`redis-cli`必须用作另一个命令的输入或重定向到文件中。

`redis-cli`仅当检测到标准输出是 tty 或终端时, 才显示其他信息以供用户阅读。对于所有其他输出, 它将自动启用*原始输出模式*, 如以下示例所示：

    $ redis-cli INCR mycounter > /tmp/output.txt
    $ cat /tmp/output.txt
    8

请注意, `(integer)`从输出中省略, 因为`redis-cli`检测输出不再写入终端。您可以强制原始输出即使在终端上也带有`--raw`选择：

    $ redis-cli --raw INCR mycounter
    9

您可以在写入文件时强制友好可读输出, 通过使用管道连接到其他命令`--no-raw`.

## 字符串引用和转义

什么时候`redis-cli`解析命令, 空格字符自动分隔参数。
在交互模式下, 换行符发送用于分析和执行的命令。
若要输入包含空格或不可打印字符的字符串值, 可以使用带引号和转义的字符串。

带引号的字符串值括在双精度  (`"`)  或单个  (`'`)  引号。
转义序列用于将不可打印的字符放在字符和字符串文本中。

转义序列包含反斜杠  (`\`)  符号后跟一个转义序列字符。

双引号字符串支持以下转义序列：

*   `\"`- 双引号
*   `\n`- 换行符
*   `\r`- 回车
*   `\t`- 水平标签
*   `\b`- 退格
*   `\a`- 警报
*   `\\`- 反斜杠
*   `\xhh`- 由十六进制数  (*呵呵*)

单引号假定字符串是文字的, 并且只允许以下转义序列：

*   `\'`- 单引号
*   `\\`- 反斜杠

例如, 返回`Hello World`在两行上：

    127.0.0.1:6379> SET mykey "Hello\nWorld"
    OK
    127.0.0.1:6379> GET mykey
    Hello
    World

当您输入包含单引号或双引号的字符串时, 例如在密码中, 对字符串进行转义, 如下所示：

    127.0.0.1:6379> AUTH some_admin_user ">^8T>6Na{u|jp>+v\"55\@_;OU(OR]7mbAYGqsfyu48(j'%hQH7;v*f1H${*gD(Se'"

## 主机、端口、密码和数据库

默认情况下, `redis-cli`连接到地址为 127.0.0.1 的服务器, 端口为 6379。
您可以使用多个命令行选项更改端口。要指定其他主机名或 IP 地址, 请使用`-h`选择。要设置其他端口, 请使用`-p`.

    $ redis-cli -h redis15.localnet.org -p 6390 PING
    PONG

如果您的实例受密码保护, 则 `-a <password>` 选项将执行身份验证, 从而无需显式使用 `AUTH` 命令：

    $ redis-cli -a myUnguessablePazzzzzword123 PING
    PONG

**注意** 出于安全原因, 请通过 `REDISCLI_AUTH` 环境变量自动提供 `redis-cli` 的密码。

最后, 可以使用 `-n <dbnum>` 选项发送对默认数字零以外的数据库编号进行操作的命令：

    $ redis-cli FLUSHALL
    OK
    $ redis-cli -n 1 INCR a
    (integer) 1
    $ redis-cli -n 1 INCR a
    (integer) 2
    $ redis-cli -n 2 INCR a
    (integer) 1

也可以使用 `-u <uri>` 选项和 URI 模式 `redis://user:password@host:port/dbnum` 提供部分或全部信息:

    $ redis-cli -u redis://LJenkins:p%40ssw0rd@redis-16379.hosted.com:16379/0 PING
    PONG

## SSL/TLS

默认情况下, `redis-cli`使用纯 TCP 连接连接到 Redis。 您可以使用 `--tls`选项, 以及 `--cacert`或 `--cacertdir`以配置受信任的根证书捆绑包或目录。

如果目标服务器要求使用客户端证书进行身份验证, 您可以使用以下命令指定证书和相应的私钥 `--cert` 和 `--key`.

## 从其他程序获取输入

有两种方法可以使用 `redis-cli` 通过标准输入接收来自其他命令的输入。一种是使用目标有效负载作为 *stdin* 的最后一个参数。例如, 为了将 Redis 键 `net_services` 设置为来自本地文件系统的文件 `/etc/services` 的内容, 请使用 `-x` 选项：

    $ redis-cli -x SET net_services < /etc/services
    OK
    $ redis-cli GETRANGE net_services 0 50
    "#\n# Network services, Internet style\n#\n# Note that "

在上面会话的第一行中, `redis-cli`被执行与`-x`选项, 并且文件被重定向到 CLI 的标准输入作为值来满足`SET net_services`命令短语。这对于编写脚本非常有用。

另一种方法是向 `redis-cli` 提供写在文本文件中的一系列命令：

    $ cat /tmp/commands.txt
    SET item:3374 100
    INCR item:3374
    APPEND item:3374 xxx
    GET item:3374
    $ cat /tmp/commands.txt | redis-cli
    OK
    (integer) 101
    (integer) 6
    "101xxx"

中的所有命令`commands.txt`由 连续执行 `redis-cli`就好像它们是用户在交互模式下键入的一样。字符串可以是如果需要, 在文件中引用, 以便可以有单个带有空格、换行符或其他特殊字符的参数：

    $ cat /tmp/commands.txt
    SET arg_example "This is a single argument"
    STRLEN arg_example
    $ cat /tmp/commands.txt | redis-cli
    OK
    (integer) 25

## 连续运行相同的命令

可以在用户选择的执行之间暂停执行指定次数的单个命令。这在不同的上下文中很有用——例如当我们想要持续监控一些关键内容或 "INFO" 字段输出时, 或者当我们想要模拟一些重复的写入事件时, 例如每 5 秒将一个新项目推入列表中。

此功能由两个选项控制：`-r <count>`和`-i <delay>`.
`-r` 选项说明运行命令的次数, `-i` 设置不同命令调用之间的延迟 (以秒为单位)  (能够指定诸如 0.1 之类的值来表示 100 毫秒) 。

默认情况下, 间隔 (或延迟) 设置为 0, 因此命令会尽快执行：

    $ redis-cli -r 5 INCR counter_value
    (integer) 1
    (integer) 2
    (integer) 3
    (integer) 4
    (integer) 5

要无限期地运行相同的命令, 请使用`-1`作为计数值。
要监视 RSS 内存大小随时间的变化, 可以使用以下命令：

    $ redis-cli -r -1 -i 1 INFO | grep rss_human
    used_memory_rss_human:2.71M
    used_memory_rss_human:2.73M
    used_memory_rss_human:2.73M
    used_memory_rss_human:2.73M
    ... a new line will be printed each second ...

## 使用大量插入数据`redis-cli`

使用 `redis-cli` 的大量插入在单独的页面中进行了介绍, 因为它本身就是一个有价值的主题。请参阅我们的 [批量插入指南](/topics/mass-insert).

## CSV 输出

CSV (逗号分隔值) 输出功能存在于`redis-cli`将数据从 Redis 导出到外部程序。

    $ redis-cli LPUSH mylist a b c d
    (integer) 4
    $ redis-cli --csv LRANGE mylist 0 -1
    "d","c","b","a"

请注意, `--csv`flag 将仅适用于单个命令, 而不适用于整个数据库作为导出。

## 运行 Lua 脚本

`redis-cli` 广泛支持使用 Lua 脚本的调试工具, 可用于 Redis 3.2 及更高版本。关于这个特性, 请参考[Redis Lua debugger documentation](/topics/ldb).

即使不使用调试器, `redis-cli` 也可用于从文件运行脚本作为参数：

    $ cat /tmp/script.lua
    return redis.call('SET',KEYS[1],ARGV[1])
    $ redis-cli --eval /tmp/script.lua location:hastings:temp , 23
    OK

Redis `EVAL` 命令将脚本使用的键列表和其他非键参数作为不同的数组。调用 "EVAL" 时, 您将键的数量作为数字提供.

当使用上面的 `--eval` 选项调用 `redis-cli` 时, 不需要明确指定键的数量。相反, 它使用用逗号分隔键和参数的约定。这就是为什么在上面的调用中你会看到 `location:hastings:temp, 23` 作为参数。

所以`location:hastings:temp`将填充`KEYS`数组, 以及`23`这`ARGV`数组。

`--eval` 选项在编写简单脚本时很有用。对于更复杂的工作, 推荐使用 Lua 调试器。可以混合使用这两种方法, 因为调试器也可以从外部文件执行脚本。

# 交互模式

我们已经探索了如何将 Redis CLI 用作命令行程序。
这对于脚本和某些类型的测试很有用, 但是大多数人会在 `redis-cli` 中使用其交互模式花费大部分时间。

在交互模式下, 用户在提示符下键入 Redis 命令。命令被发送到服务器, 被处理, 回复被解析回来并呈现为更简单的形式来阅读.

在交互模式下运行 `redis-cli` 不需要什么特别的 - 只需在没有任何参数的情况下执行它.

    $ redis-cli
    127.0.0.1:6379> PING
    PONG

字符串`127.0.0.1:6379>`是提示符。它显示连接的 Redis 服务器实例的主机名和端口。

当连接的服务器发生更改时, 或者在对数据库编号为 0 不同的数据库进行操作时, 提示符会更新：

    127.0.0.1:6379> SELECT 2
    OK
    127.0.0.1:6379[2]> DBSIZE
    (integer) 1
    127.0.0.1:6379[2]> SELECT 0
    OK
    127.0.0.1:6379> DBSIZE
    (integer) 503

## 处理连接和重新连接

在交互模式下使用 `CONNECT` 命令可以通过指定要连接的 *hostname* 和 *port* 来连接到不同的实例：

    127.0.0.1:6379> CONNECT metal 6379
    metal:6379> PING
    PONG

如您所见, 当连接到不同的服务器实例时, 提示会相应更改。
如果尝试连接到无法访问的实例, 则 "redis-cli" 进入断开模式并尝试使用每个新命令重新连接：

    127.0.0.1:6379> CONNECT 127.0.0.1 9999
    Could not connect to Redis at 127.0.0.1:9999: Connection refused
    not connected> PING
    Could not connect to Redis at 127.0.0.1:9999: Connection refused
    not connected> PING
    Could not connect to Redis at 127.0.0.1:9999: Connection refused

一般在检测到断线后, `redis-cli` 总是尝试透明地重新连接；如果尝试失败, 则显示错误并进入断开连接状态。以下是断开和重新连接的示例：

    127.0.0.1:6379> INFO SERVER
    Could not connect to Redis at 127.0.0.1:6379: Connection refused
    not connected> PING
    PONG
    127.0.0.1:6379> 
    (now we are connected again)

执行重新连接时, `redis-cli` 会自动重新选择最后选择的数据库编号。但是, 有关连接的所有其他状态都会丢失, 例如在 MULTI/EXEC 事务中：

    $ redis-cli
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> PING
    QUEUED

    ( here the server is manually restarted )

    127.0.0.1:6379> EXEC
    (error) ERR EXEC without MULTI

在交互模式下使用 `redis-cli` 进行测试时, 这通常不是问题, 但应该知道这个限制。

## 编辑、历史记录、完成和提示

因为`redis-cli`使用了 [linenoise行编辑库](http://github.com/antirez/linenoise), 所以它始终具有行编辑能力, 无需依赖`libreadline`或其他可选库。

可以通过按箭头键 (向上和向下) 访问命令执行历史记录以避免重新键入命令。
历史记录在 CLI 重新启动之间保存在用户主目录中名为 .rediscli_history 的文件中, 由 `HOME` 环境变量指定。可以通过设置 `REDISCLI_HISTFILE` 环境变量来使用不同的历史文件名, 并通过将其设置为 `/dev/null` 来禁用它.

`redis-cli` 也可以通过按 TAB 键来完成命令名, 如下例所示：

    127.0.0.1:6379> Z<TAB>
    127.0.0.1:6379> ZADD<TAB>
    127.0.0.1:6379> ZCARD<TAB>

在提示符下输入 Redis 命令名称后, `redis-cli` 将显示语法提示。与命令历史一样, 此行为可以通过 `redis-cli` 首选项打开和关闭。

## 首选项

有两种方法可以自定义 `redis-cli` 行为。主目录中的文件 `.redisclirc` 在启动时由 CLI 加载。您可以通过将 `REDISCLI_RCFILE` 环境变量设置为替代路径来覆盖文件的默认位置。也可以在 CLI 会话期间设置首选项, 在这种情况下, 它们将仅持续会话的持续时间.

要设置首选项, 请使用特殊的 `:set` 命令。可以通过在 CLI 中键入命令或将其添加到 `.redisclirc` 文件来设置以下首选项：

*   `:set hints`- 启用语法提示
*   `:set nohints`- 禁用语法提示

## 运行相同的命令 N 次

通过在命令名称前加上一个数字, 可以在交互模式下多次运行相同的命令：

    127.0.0.1:6379> 5 INCR mycounter
    (integer) 1
    (integer) 2
    (integer) 3
    (integer) 4
    (integer) 5

## 显示有关 Redis 命令的帮助

`redis-cli` 使用`HELP` 命令为大多数 Redis [commands](/commands) 提供在线帮助。该命令可以以两种形式使用：

*   `HELP @<category>`显示有关给定类别的所有命令。这
    类别是：
    *   `@generic`
    *   `@string`
    *   `@list`
    *   `@set`
    *   `@sorted_set`
    *   `@hash`
    *   `@pubsub`
    *   `@transactions`
    *   `@connection`
    *   `@server`
    *   `@scripting`
    *   `@hyperloglog`
    *   `@cluster`
    *   `@geo`
    *   `@stream`
*   `HELP <commandname>`显示了作为参数给出的命令的特定帮助。

例如, 为了显示对`PFADD`命令, 使用：

    127.0.0.1:6379> HELP PFADD

    PFADD key element [element ...]
    summary: Adds the specified elements to the specified HyperLogLog.
    since: 2.8.9

请注意, `HELP`还支持 TAB 完成。

## 清除终端屏幕

使用`CLEAR`命令在交互模式下清除终端的屏幕。

# 特殊操作模式

到目前为止, 我们看到了两种主要模式`redis-cli`.

*   Redis 命令的命令行执行。
*   交互式 "REPL" 用法。

CLI 执行与 Redis 相关的其他辅助任务, 
将在以下各节中介绍：

*   监控工具, 用于显示有关 Redis 服务器的连续统计信息。
*   扫描 Redis 数据库中是否有非常大的 key。
*   键空间扫描匹配器。
*   充当[发布/订阅](/topics/pubsub)客户端以订阅频道。
*   监视在 Redis 实例中执行的命令。
*   检查[延迟](/topics/latency)以不同的方式管理 Redis 服务器。
*   检查本地计算机的计划程序延迟。
*   从本地远程 Redis 服务器传输 RDB 备份。
*   充当 Redis 副本, 用于显示副本接收的内容。
*   模拟[LRU](/topics/lru-cache)用于显示有关键命中的统计信息的工作负载。
*   Lua 调试器的客户端。

## 连续统计模式

连续统计模式可能是鲜为人知但非常有用的功能之一`redis-cli`以实时监控 Redis 实例。要启用此模式, 请将`--stat`使用选项。
输出非常清楚此模式下 CLI 的行为：

    $ redis-cli --stat
    ------- data ------ --------------------- load -------------------- - child -
    keys       mem      clients blocked requests            connections
    506        1015.00K 1       0       24 (+0)             7
    506        1015.00K 1       0       25 (+1)             7
    506        3.40M    51      0       60461 (+60436)      57
    506        3.40M    51      0       146425 (+85964)     107
    507        3.40M    51      0       233844 (+87419)     157
    507        3.40M    51      0       321715 (+87871)     207
    508        3.40M    51      0       408642 (+86927)     257
    508        3.40M    51      0       497038 (+88396)     257

在此模式下, 每秒打印一行新行, 其中包含有用信息和旧数据点之间请求值的差异。使用此辅助工具, 可以轻松理解有关所连接 Redis 数据库的内存使用情况、客户端连接计数以及各种其他统计信息`redis-cli`工具。

在这种情况下, `-i <interval>` 选项用作修饰符, 以更改发出新行的频率。默认为一秒。

## 扫描 big key

在这种特殊模式下, `redis-cli` 用作键空间分析器。它扫描数据集以查找大键, 但还提供有关数据集包含的数据类型的信息。使用 `--bigkeys` 选项启用此模式, 并产生详细输出：

    $ redis-cli --bigkeys

    # Scanning the entire keyspace to find biggest keys as well as
    # average sizes per key type.  You can use -i 0.01 to sleep 0.01 sec
    # per SCAN command (not usually needed).

    [00.00%] Biggest string found so far 'key-419' with 3 bytes
    [05.14%] Biggest list   found so far 'mylist' with 100004 items
    [35.77%] Biggest string found so far 'counter:__rand_int__' with 6 bytes
    [73.91%] Biggest hash   found so far 'myobject' with 3 fields

    -------- summary -------

    Sampled 506 keys in the keyspace!
    Total key length in bytes is 3452 (avg len 6.82)

    Biggest string found 'counter:__rand_int__' has 6 bytes
    Biggest   list found 'mylist' has 100004 items
    Biggest   hash found 'myobject' has 3 fields

    504 strings with 1403 bytes (99.60% of keys, avg size 2.78)
    1 lists with 100004 items (00.20% of keys, avg size 100004.00)
    0 sets with 0 members (00.00% of keys, avg size 0.00)
    1 hashs with 3 fields (00.20% of keys, avg size 3.00)
    0 zsets with 0 members (00.00% of keys, avg size 0.00)

在输出的第一部分, 报告每个遇到的比先前更大的键 (相同类型) 更大的新键。摘要部分提供了有关 Redis 实例内部数据的一般统计信息。

该程序使用 "SCAN" 命令, 因此它可以在繁忙的服务器上执行而不影响操作, 但是可以使用`-i` 选项来限制每个 "SCAN" 的指定秒数的扫描过程命令。

例如, `-i 0.01` 会大大减慢程序的执行速度, 但也会将服务器上的负载减少到可以忽略不计的程度。

请注意, 摘要还以更清晰的形式报告每次找到的最大键。如果针对非常大的数据集运行, 初始输出只是为了尽快提供一些有趣的信息。

## 获取 key 列表

也可以再次以不阻塞 Redis 服务器的方式扫描 key 空间 (当您使用类似 `KEYS *` 之类的命令时确实会发生这种情况) , 并打印所有密钥名称, 或针对特定模式过滤它们.这种模式, 像 `--bigkeys` 选项, 使用 `SCAN` 命令, 因此如果数据集发生变化, 可能会多次报告键, 但如果该键自开始时就存在, 则不会丢失任何键迭代。因为它使用这个选项的命令叫做`--scan`.

    $ redis-cli --scan | head -10
    key-419
    key-71
    key-236
    key-50
    key-38
    key-458
    key-453
    key-499
    key-446
    key-371

请注意, 使用 `head -10` 是为了只打印输出的第一行。

扫描能够使用带有 `--pattern` 选项的 `SCAN` 命令的底层模式匹配功能。

    $ redis-cli --scan --pattern '*-11*'
    key-114
    key-117
    key-118
    key-113
    key-115
    key-112
    key-119
    key-11
    key-111
    key-110
    key-116

通过 `wc` 命令管道输出可用于按键名计算特定类型的对象：

    $ redis-cli --scan --pattern 'user:*' | wc -l
    3829433

您可以使用`-i 0.01`以在调用之间添加延迟`SCAN`命令。
这将使命令速度变慢, 但会显著减少服务器上的负载。

## 发布/订阅模式

CLI 能够使用 "PUBLISH" 命令在 Redis Pub/Sub 通道中发布消息。订阅频道以接收消息是不同的 - 终端被阻塞并等待消息, 因此这是在 `redis-cli` 中作为特殊模式实现的。与其他特殊模式不同, 此模式不是通过使用特殊选项来启用的, 而是通过使用 `SUBSCRIBE` 或 `PSUBSCRIBE` 命令来启用的, 它们在交互或命令模式下可用：

    $ redis-cli PSUBSCRIBE '*'
    Reading messages... (press Ctrl-C to quit)
    1) "PSUBSCRIBE"
    2) "*"
    3) (integer) 1

这*阅读消息*消息显示我们进入了发布/订阅模式。
当另一个客户端在某个通道中发布某些消息时, 例如使用命令`redis-cli PUBLISH mychannel mymessage`, 则发布/订阅模式下 的 CLI 将显示如下内容：

    1) "pmessage"
    2) "*"
    3) "mychannel"
    4) "mymessage"

这对于调试发布/订阅问题非常有用。
要退出发布/订阅模式, 只需处理`CTRL-C`.

## 监视在 Redis 中执行的命令

与 Pub/Sub 模式类似, 一旦您使用 "MONITOR" 命令, 就会自动进入监控模式。活动 Redis 实例接收到的所有命令都将打印到标准输出：

    $ redis-cli MONITOR
    OK
    1460100081.165665 [0 127.0.0.1:51706] "set" "shipment:8000736522714:status" "sorting"
    1460100083.053365 [0 127.0.0.1:51707] "get" "shipment:8000736522714:status"

请注意, 可以使用管道输出, 因此您可以使用 `grep` 等工具监视特定模式.

## 监控 Redis 实例的延迟

Redis 通常用于延迟非常关键的环境中。延迟涉及应用程序中的多个移动部分, 从客户端库到网络堆栈, 再到 Redis 实例本身。

`redis-cli` 有多种工具可用于研究 Redis 实例的延迟并了解延迟的最大值、平均值和分布。

基本的延迟检查工具是 `--latency` 选项。使用此选项, CLI 会运行一个循环, 将 "PING"命令发送到 Redis 实例并测量接收回复的时间。这种情况每秒发生 100 次, 并且统计信息会在控制台中实时更新：

    $ redis-cli --latency
    min: 0, max: 1, avg: 0.19 (427 samples)

统计信息以毫秒为单位提供。通常, 一个非常快的实例的平均延迟往往会被高估一点, 因为系统的内核调度程序本身运行 `redis-cli` 会导致延迟, 因此上述 0.19 的平均延迟可能很容易达到 0.01 或更小.

然而, 这通常不是一个大问题, 因为大多数开发人员对几毫秒或更长时间的事件感兴趣.

有时研究最大和平均延迟如何随时间变化很有用。 `--latency-history` 选项用于此目的：它的工作方式与 `--latency` 完全相同, 但每 15 秒 (默认情况下) 从头开始一个新的采样会话：

    $ redis-cli --latency-history
    min: 0, max: 1, avg: 0.14 (1314 samples) -- 15.01 seconds range
    min: 0, max: 1, avg: 0.18 (1299 samples) -- 15.00 seconds range
    min: 0, max: 1, avg: 0.20 (113 samples)^C

采样会话的长度可以通过`-i <interval>`选择。

最先进的延迟研究工具, 但对于没有经验的用户来说也是最复杂的解释, 是使用彩色终端显示一系列延迟的能力。您将看到一个彩色输出, 指示不同百分比的样本, 以及指示不同延迟数字的不同 ASCII 字符.
使用 `--latency-dist` 选项启用此模式：

    $ redis-cli --latency-dist
    (output not displayed, requires a color terminal, try it!)

内部还实现了另一个非常不寻常的延迟工具`redis-cli`.
它不检查 Redis 实例的延迟, 而是检查运行 `redis-cli` 的计算机的延迟。这种延迟是内核调度程序、虚拟化实例情况下的管理程序等所固有的。

Redis称之为*固有延迟*因为它对程序员来说大多是不透明的。
如果 Redis 实例具有高延迟, 不管所有明显的事情可能是源头原因, 那么值得通过直接在运行 Redis 的系统中以这种特殊模式运行 `redis-cli` 来检查您的系统可以做的最好的事情服务器在。

通过测量内在延迟, 您知道这是基线, Redis 无法超越您的系统。为了在这种模式下运行 CLI, 请使用 `--intrinsic-latency <test-time>`。请注意, 测试时间以秒为单位, 并指示测试应运行多长时间。

    $ ./redis-cli --intrinsic-latency 5
    Max latency so far: 1 microseconds.
    Max latency so far: 7 microseconds.
    Max latency so far: 9 microseconds.
    Max latency so far: 11 microseconds.
    Max latency so far: 13 microseconds.
    Max latency so far: 15 microseconds.
    Max latency so far: 34 microseconds.
    Max latency so far: 82 microseconds.
    Max latency so far: 586 microseconds.
    Max latency so far: 739 microseconds.

    65433042 total runs (avg latency: 0.0764 microseconds / 764.14 nanoseconds per run).
    Worst run took 9671x longer than the average latency.

重要说明：此命令必须在运行 Redis 服务器实例的计算机上执行, 而不是在其他主机上执行。它不会连接到 Redis 实例, 而是在本地执行测试。

在上述情况下, 系统的最坏情况延迟不能超过 739 微秒, 因此可以预期某些查询偶尔会运行不到 1 毫秒。

## 远程备份 RDB 文件

在 Redis 复制的第一次同步期间, 主副本和副本以 RDB 文件的形式交换整个数据集。 `redis-cli` 利用此功能提供远程备份工具, 允许将 RDB 文件从任何 Redis 实例传输到运行 `redis-cli` 的本地计算机。要使用此模式, 请使用 `--rdb <dest-filename>` 选项调用 CLI：

    $ redis-cli --rdb /tmp/dump.rdb
    SYNC sent to master, writing 13256 bytes to '/tmp/dump.rdb'
    Transfer finished with success.

这是确保 Redis 实例的灾难恢复 RDB 备份存在的简单但有效的方法。在脚本或 `cron` 作业中使用此选项时, 请务必检查命令的返回值.
如果它不是零, 则发生错误, 如以下示例所示：

    $ redis-cli --rdb /tmp/dump.rdb
    SYNC with master failed: -ERR Can't SYNC while not connected with my master
    $ echo $?
    1

## 复制副本模式

CLI 的复制模式是一项高级功能, 可用于 Redis 开发人员和用于调试操作.
它允许检查主节点在复制流中发送到其副本的内容, 以便将写入传播到其副本。选项名称只是 `--replica`。以下是一个工作示例：

    $ redis-cli --replica
    SYNC with master, discarding 13256 bytes of bulk transfer...
    SYNC done. Logging commands from master.
    "PING"
    "SELECT","0"
    "SET","last_name","Enigk"
    "PING"
    "INCR","mycounter"

该命令首先放弃第一次同步的 RDB 文件, 然后以 CSV 格式记录收到的每个命令。

如果您认为某些命令未在副本中正确复制, 这是检查正在发生的事情的好方法, 也是改进错误报告的有用信息.

## 执行 LRU 模拟

Redis 通常用作缓存[LRU 淘汰](/topics/lru-cache).
根据键的数量和为缓存分配的内存量 (通过 `maxmemory` 指令指定) , 缓存命中和未命中的数量会发生变化。有时, 模拟命中率对于正确配置缓存非常有用。

`redis-cli` 有一个特殊的模式, 它执行 GET 和 SET 操作的模拟, 在请求模式中使用 80-20% 的幂律分布。
这意味着 20% 的 key 将被请求 80% 的次数, 这是缓存场景中的常见分布。

理论上, 给定请求的分布和 Redis 内存开销, 应该可以用数学公式分析计算命中率。但是, Redis 可以配置不同的 LRU 设置 (样本数) , 并且在 Redis 中近似的 LRU 的实现在不同版本之间变化很大。同样, 每个密钥的内存量可能会在版本之间发生变化。这就是构建此工具的原因：它的主要动机是测试 Redis 的 LRU 实现的质量, 但现在也可用于测试给定版本在最初用于部署的设置下的行为方式。

要使用此模式, 请在测试中指定密钥的数量, 并配置合理的`maxmemory`设置为第一次尝试。

重要提示：在 Redis 配置中配置 `maxmemory` 设置至关重要：如果最大内存使用没有上限, 则命中最终将是 100%, 因为所有键都可以存储在内存中。如果为最大内存指定了太多键, 最终将使用所有计算机 RAM。还需要配置一个合适的*maxmemory policy*; 大多数时候选择了`allkeys-lru`。

在以下示例中, 配置了 100MB 的内存限制和使用 1000 万个 key 的 LRU 模拟。

警告：测试使用流水线, 会给服务器带来压力, 不要将它与生产实例一起使用。

    $ ./redis-cli --lru-test 10000000
    156000 Gets/sec | Hits: 4552 (2.92%) | Misses: 151448 (97.08%)
    153750 Gets/sec | Hits: 12906 (8.39%) | Misses: 140844 (91.61%)
    159250 Gets/sec | Hits: 21811 (13.70%) | Misses: 137439 (86.30%)
    151000 Gets/sec | Hits: 27615 (18.29%) | Misses: 123385 (81.71%)
    145000 Gets/sec | Hits: 32791 (22.61%) | Misses: 112209 (77.39%)
    157750 Gets/sec | Hits: 42178 (26.74%) | Misses: 115572 (73.26%)
    154500 Gets/sec | Hits: 47418 (30.69%) | Misses: 107082 (69.31%)
    151250 Gets/sec | Hits: 51636 (34.14%) | Misses: 99614 (65.86%)

该程序每秒显示一次统计信息。在最初的几秒钟内, 缓存开始填充。未命中率后来稳定在可以预期的实际数字中：

    120750 Gets/sec | Hits: 48774 (40.39%) | Misses: 71976 (59.61%)
    122500 Gets/sec | Hits: 49052 (40.04%) | Misses: 73448 (59.96%)
    127000 Gets/sec | Hits: 50870 (40.06%) | Misses: 76130 (59.94%)
    124250 Gets/sec | Hits: 50147 (40.36%) | Misses: 74103 (59.64%)

对于某些用例, 59% 的未命中率可能是不可接受的, 因为 100MB 的内存是不够的。观察一个使用半 GB 内存的示例。几分钟后输出稳定到以下数字：

    140000 Gets/sec | Hits: 135376 (96.70%) | Misses: 4624 (3.30%)
    141250 Gets/sec | Hits: 136523 (96.65%) | Misses: 4727 (3.35%)
    140250 Gets/sec | Hits: 135457 (96.58%) | Misses: 4793 (3.42%)
    140500 Gets/sec | Hits: 135947 (96.76%) | Misses: 4553 (3.24%)

500MB有足够的空间用于密钥数量 (1000万) 和分发 (80-20样式) 。
