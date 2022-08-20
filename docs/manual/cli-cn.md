***

## 标题： “Redis CLI”&#xA;链接标题： “CLI”&#xA;体重： 1&#xA;描述： >&#xA;Redis-cli 概述，Redis 命令行界面&#xA;别名：&#xA;\- /docs/manual/cli

Redis 命令行界面 （`redis-cli`） 是一个终端程序，用于向 Redis 服务器发送命令并从 Redis 服务器读取回复。它有两种主要模式：交互式读取评估打印循环（REPL）模式，用户键入Redis命令并接收回复，以及命令模式`redis-cli`使用其他参数执行，并将回复打印到标准输出。

在交互模式下，`redis-cli`具有基本的行编辑功能，可提供熟悉的打字体验。

要以特殊模式启动程序，可以使用多个选项，包括：

*   模拟副本并打印它从主数据库接收的复制流。
*   检查 Redis 服务器的延迟并显示统计信息。
*   请求延迟样本和频率的 ASCII 艺术频谱图。

本主题涵盖`redis-cli`，从最简单的开始，以更高级的功能结束。

## 命令行用法

要运行 Redis 命令并在终端返回标准输出，请将要执行的命令作为`redis-cli`:

    $ redis-cli INCR mycounter
    (integer) 7

该命令的回复为“7”。由于 Redis 回复是键入的（字符串、数组、整数、nil、错误等），因此您会在括号之间看到回复的类型。当输出`redis-cli`必须用作另一个命令的输入或重定向到文件中。

`redis-cli`仅当检测到标准输出是 tty 或终端时，才显示其他信息以供用户阅读。对于所有其他输出，它将自动启用*原始输出模式*，如以下示例所示：

    $ redis-cli INCR mycounter > /tmp/output.txt
    $ cat /tmp/output.txt
    8

请注意，`(integer)`从输出中省略，因为`redis-cli`检测
输出不再写入终端。您可以强制原始输出
即使在终端上也带有`--raw`选择：

    $ redis-cli --raw INCR mycounter
    9

您可以在写入文件时强制人类可读输出
通过使用管道连接到其他命令`--no-raw`.

## 字符串引用和转义

什么时候`redis-cli`解析命令，空格字符自动分隔参数。
在交互模式下，换行符发送用于分析和执行的命令。
若要输入包含空格或不可打印字符的字符串值，可以使用带引号和转义的字符串。

带引号的字符串值括在双精度 （`"`） 或单个 （`'`） 引号。
转义序列用于将不可打印的字符放在字符和字符串文本中。

转义序列包含反斜杠 （`\`） 符号后跟一个转义序列字符。

双引号字符串支持以下转义序列：

*   `\"`- 双引号
*   `\n`- 换行符
*   `\r`- 回车
*   `\t`- 水平标签
*   `\b`- 退格
*   `\a`- 警报
*   `\\`- 反斜杠
*   `\xhh`- 由十六进制数 （*呵呵*)

单引号假定字符串是文字的，并且只允许以下转义序列：

*   `\'`- 单引号
*   `\\`- 反斜杠

例如，返回`Hello World`在两行上：

    127.0.0.1:6379> SET mykey "Hello\nWorld"
    OK
    127.0.0.1:6379> GET mykey
    Hello
    World

当您输入包含单引号或双引号的字符串时，例如在密码中，对字符串进行转义，如下所示：

    127.0.0.1:6379> AUTH some_admin_user ">^8T>6Na{u|jp>+v\"55\@_;OU(OR]7mbAYGqsfyu48(j'%hQH7;v*f1H${*gD(Se'"

## 主机、端口、密码和数据库

默认情况下，`redis-cli`连接到地址为 127.0.0.1 的服务器，端口为 6379。
您可以使用多个命令行选项更改端口。要指定其他主机名或 IP 地址，请使用`-h`选择。要设置其他端口，请使用`-p`.

    $ redis-cli -h redis15.localnet.org -p 6390 PING
    PONG

如果您的实例受密码保护，则`-a <password>`选项将
预制件身份验证，无需显式使用`AUTH`命令：

    $ redis-cli -a myUnguessablePazzzzzword123 PING
    PONG

**注意：**出于安全原因，请将密码提供给`redis-cli`自动通过
`REDISCLI_AUTH`环境变量。

最后，可以发送对数据库编号进行操作的命令。
除了默认数字 0 之外，请使用`-n <dbnum>`选择：

    $ redis-cli FLUSHALL
    OK
    $ redis-cli -n 1 INCR a
    (integer) 1
    $ redis-cli -n 1 INCR a
    (integer) 2
    $ redis-cli -n 2 INCR a
    (integer) 1

也可以使用`-u <uri>`
选项和 URI 模式`redis://user:password@host:port/dbnum`:

    $ redis-cli -u redis://LJenkins:p%40ssw0rd@redis-16379.hosted.com:16379/0 PING
    PONG

## SSL/TLS

默认情况下，`redis-cli`使用纯 TCP 连接连接到 Redis。
您可以使用`--tls`选项，以及`--cacert`或
`--cacertdir`以配置受信任的根证书捆绑包或目录。

如果目标服务器要求使用客户端证书进行身份验证，
您可以使用以下命令指定证书和相应的私钥`--cert`和
`--key`.

## 从其他程序获取输入

有两种方法可以使用`redis-cli`为了接收来自其他的输入
通过标准输入的命令。一种是使用目标有效负载作为最后一个参数
从*标准丁*.例如，为了设置 Redis 键`net_services`
到文件的内容`/etc/services`从本地文件系统，使用`-x`
选择：

    $ redis-cli -x SET net_services < /etc/services
    OK
    $ redis-cli GETRANGE net_services 0 50
    "#\n# Network services, Internet style\n#\n# Note that "

在上面会话的第一行中，`redis-cli`被执行与`-x`选项，并且文件被重定向到 CLI 的
标准输入作为值来满足`SET net_services`命令短语。这对于编写脚本非常有用。

另一种方法是喂养`redis-cli`一系列命令写在
文本文件：

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

中的所有命令`commands.txt`由 连续执行
`redis-cli`就好像它们是用户在交互模式下键入的一样。字符串可以是
如果需要，在文件中引用，以便可以有单个
带有空格、换行符或其他特殊字符的参数：

    $ cat /tmp/commands.txt
    SET arg_example "This is a single argument"
    STRLEN arg_example
    $ cat /tmp/commands.txt | redis-cli
    OK
    (integer) 25

## 连续运行相同的命令

可以执行单个命令指定次数
在执行之间有用户选择的暂停。这在以下方面很有用：
不同的上下文 - 例如，当我们想要持续监控某些环境时
关键内容或`INFO`场输出，或者当我们想模拟一些
重复写入事件，例如每 5 秒将新项推送到列表中。

此功能由两个选项控制：`-r <count>`和`-i <delay>`.
这`-r`选项声明运行命令的次数，以及`-i`集
不同命令调用之间的延迟（以秒为单位）（具有
以指定值（如 0.1）以表示 100 毫秒）。

默认情况下，间隔（或延迟）设置为 0，因此只需执行命令
尽快：

    $ redis-cli -r 5 INCR counter_value
    (integer) 1
    (integer) 2
    (integer) 3
    (integer) 4
    (integer) 5

要无限期地运行相同的命令，请使用`-1`作为计数值。
要监视 RSS 内存大小随时间的变化，可以使用以下命令：

    $ redis-cli -r -1 -i 1 INFO | grep rss_human
    used_memory_rss_human:2.71M
    used_memory_rss_human:2.73M
    used_memory_rss_human:2.73M
    used_memory_rss_human:2.73M
    ... a new line will be printed each second ...

## 使用大量插入数据`redis-cli`

批量插入使用`redis-cli`在单独的页面中涵盖，因为它是
值得一提的话题本身。请参考我们的[批量插入指南](/topics/mass-insert).

## 断续器输出

CSV（逗号分隔值）输出功能存在于`redis-cli`将数据从 Redis 导出到外部程序。

    $ redis-cli LPUSH mylist a b c d
    (integer) 4
    $ redis-cli --csv LRANGE mylist 0 -1
    "d","c","b","a"

请注意，`--csv`flag 将仅适用于单个命令，而不适用于整个数据库作为导出。

## 运行 Lua 脚本

这`redis-cli`对使用调试工具具有广泛的支持
的 Lua 脚本，可在 Redis 3.2 及更高版本中使用。有关此功能，请参阅[Redis Lua 调试器文档](/topics/ldb).

即使不使用调试器，`redis-cli`可用于
从文件运行脚本作为参数：

    $ cat /tmp/script.lua
    return redis.call('SET',KEYS[1],ARGV[1])
    $ redis-cli --eval /tmp/script.lua location:hastings:temp , 23
    OK

雷迪斯酒店`EVAL`命令获取脚本使用的键列表，并且
其他非键参数，作为不同的数组。呼叫时`EVAL`你
以数字形式提供密钥数。

呼叫时`redis-cli`与`--eval`上面的选项，无需指定键数
明确地。相反，它使用分离键和参数的约定。
带逗号。这就是为什么在上面的电话中你会看到`location:hastings:temp , 23`作为参数。

所以`location:hastings:temp`将填充`KEYS`数组，以及`23`这`ARGV`数组。

这`--eval`选项在编写简单脚本时很有用。更多
工作复杂，建议使用Lua调试器。可以混合使用这两种方法，因为调试器还可以从外部文件执行脚本。

# 交互模式

我们已经探索了如何使用 Redis CLI 作为命令行程序。
这对于脚本和某些类型的测试很有用，但大多数
人们将花费大部分时间在`redis-cli`使用其交互式
模式。

在交互模式下，用户在提示符下键入 Redis 命令。命令
发送到服务器，进行处理，并解析回并呈现回复
以更简单的形式阅读。

运行`redis-cli`在交互模式下 -
只需执行它而没有任何参数

    $ redis-cli
    127.0.0.1:6379> PING
    PONG

字符串`127.0.0.1:6379>`是提示符。它显示连接的 Redis 服务器实例的主机名和端口。

当连接的服务器发生更改时，或者在对数据库编号为 0 不同的数据库进行操作时，提示符会更新：

    127.0.0.1:6379> SELECT 2
    OK
    127.0.0.1:6379[2]> DBSIZE
    (integer) 1
    127.0.0.1:6379[2]> SELECT 0
    OK
    127.0.0.1:6379> DBSIZE
    (integer) 503

## 处理连接和重新连接

使用`CONNECT`命令在交互模式下，可以进行连接
到其他实例，通过指定*主机名*和*港口*我们想要
以连接到：

    127.0.0.1:6379> CONNECT metal 6379
    metal:6379> PING
    PONG

如您所见，在连接到其他服务器实例时，提示会相应地更改。
如果尝试连接到无法访问的实例，则`redis-cli`断开连接
模式并尝试与每个新命令重新连接：

    127.0.0.1:6379> CONNECT 127.0.0.1 9999
    Could not connect to Redis at 127.0.0.1:9999: Connection refused
    not connected> PING
    Could not connect to Redis at 127.0.0.1:9999: Connection refused
    not connected> PING
    Could not connect to Redis at 127.0.0.1:9999: Connection refused

通常在检测到断开连接后，`redis-cli`总是尝试
以透明方式重新连接;如果尝试失败，它将显示错误和
进入断开连接状态。以下是断开连接的示例
和重新连接：

    127.0.0.1:6379> INFO SERVER
    Could not connect to Redis at 127.0.0.1:6379: Connection refused
    not connected> PING
    PONG
    127.0.0.1:6379> 
    (now we are connected again)

执行重新连接时，`redis-cli`自动重新选择
上次选择的数据库编号。但是，所有其他状态关于
连接丢失，例如在 MULTI/EXEC 事务中：

    $ redis-cli
    127.0.0.1:6379> MULTI
    OK
    127.0.0.1:6379> PING
    QUEUED

    ( here the server is manually restarted )

    127.0.0.1:6379> EXEC
    (error) ERR EXEC without MULTI

使用`redis-cli`在交互模式下
测试，但这种限制应该是已知的。

## 编辑、历史记录、完成和提示

因为`redis-cli`使用
[线噪声行编辑库](http://github.com/antirez/linenoise)它
始终具有行编辑功能，无需依赖`libreadline`或
其他可选库。

可以访问命令执行历史记录，以避免通过按箭头键（向上和向下）重新键入命令。
历史记录在 CLI 重新启动之间保留在名为
`.rediscli_history`在用户主目录内，如指定
由`HOME`环境变量。可以使用不同的
历史记录文件名，通过设置`REDISCLI_HISTFILE`环境变量
并通过将其设置为`/dev/null`.

这`redis-cli`还能够通过按 Tab 执行命令名称完成
键，如以下示例所示：

    127.0.0.1:6379> Z<TAB>
    127.0.0.1:6379> ZADD<TAB>
    127.0.0.1:6379> ZCARD<TAB>

在提示符下输入 Redis 命令名称后，`redis-cli`将显示
语法提示。与命令历史记录一样，此行为可以通过`redis-cli`偏好。

## 偏好

有两种自定义方法`redis-cli`行为。文件`.redisclirc`
中的主目录由 CLI 在启动时加载。您可以覆盖
文件的默认位置，方法是将`REDISCLI_RCFILE`环境变量
另一条路径。还可以在 CLI 会话期间设置首选项，其中
如果它们将仅持续会话的持续时间。

要设置首选项，请使用`:set`命令。以下首选项
可以通过在 CLI 中键入命令或将其添加到
`.redisclirc`文件：

*   `:set hints`- 启用语法提示
*   `:set nohints`- 禁用语法提示

## 运行相同的命令 N 次

通过在命令前面加上前缀，可以在交互模式下多次运行同一命令
以数字命名：

    127.0.0.1:6379> 5 INCR mycounter
    (integer) 1
    (integer) 2
    (integer) 3
    (integer) 4
    (integer) 5

## 显示有关 Redis 命令的帮助

`redis-cli`为大多数 Redis 提供在线帮助[命令](/commands)，使用`HELP`命令。可以使用该命令
有两种形式：

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

例如，为了显示对`PFADD`命令，使用：

    127.0.0.1:6379> HELP PFADD

    PFADD key element [element ...]
    summary: Adds the specified elements to the specified HyperLogLog.
    since: 2.8.9

请注意，`HELP`还支持 TAB 完成。

## 清除终端屏幕

使用`CLEAR`命令在交互模式下清除终端的屏幕。

# 特殊操作模式

到目前为止，我们看到了两种主要模式`redis-cli`.

*   Redis 命令的命令行执行。
*   交互式“REPL”用法。

CLI 执行与 Redis 相关的其他辅助任务，
将在以下各节中介绍：

*   监控工具，用于显示有关 Redis 服务器的连续统计信息。
*   扫描 Redis 数据库中是否有非常大的密钥。
*   具有图案匹配的键空间扫描仪。
*   充当[发布/订阅](/topics/pubsub)客户端以订阅频道。
*   监视在 Redis 实例中执行的命令。
*   检查[延迟](/topics/latency)以不同的方式管理 Redis 服务器。
*   检查本地计算机的计划程序延迟。
*   从本地远程 Redis 服务器传输 RDB 备份。
*   充当 Redis 副本，用于显示副本接收的内容。
*   模拟[断续器](/topics/lru-cache)用于显示有关键命中的统计信息的工作负载。
*   Lua 调试器的客户端。

## 连续统计模式

连续统计模式可能是鲜为人知但非常有用的功能之一`redis-cli`以实时监控 Redis 实例。要启用此模式，请将`--stat`使用选项。
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

在此模式下，每秒打印一行新行，其中包含有用信息和旧数据点之间请求值的差异。使用此辅助工具，可以轻松理解有关所连接 Redis 数据库的内存使用情况、客户端连接计数以及各种其他统计信息`redis-cli`工具。

这`-i <interval>`在这种情况下，选项用作修饰符，以便
更改发出新行的频率。默认值为 1
第二。

## 扫描大键

在这种特殊模式下，`redis-cli`用作关键空间分析仪。它扫描
数据集用于大键，但也提供有关数据类型的信息
数据集由其组成。启用此模式时`--bigkeys`选择
并生成详细输出：

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

在输出的第一部分中，每个新键都大于前一个较大
报告遇到的键（相同类型）。摘要部分
提供有关 Redis 实例内数据的一般统计信息。

该程序使用`SCAN`命令，因此可以对繁忙执行
服务器而不影响操作，但是`-i`选项可以是
用于限制指定馏分的扫描过程
秒数为每个`SCAN`命令。

例如`-i 0.01`将大大减慢程序执行速度，但也会减少服务器上的负载
可以忽略不计。

请注意，摘要还以更清晰的形式报告找到的最大键
每次。初始输出只是为了提供一些有趣的信息
如果针对非常大的数据集运行，则尽快。

## 获取密钥列表

也可以扫描密钥空间，再次以不扫描的方式
阻止 Redis 服务器（当您使用命令时确实会发生这种情况）
喜欢`KEYS *`），然后打印所有键名称，或针对特定名称进行筛选
模式。此模式，如`--bigkeys`选项，使用`SCAN`命令
因此，如果数据集正在更改，则可能会多次报告键，但不会
key 将永远丢失，如果该密钥自
迭 代。由于它使用的命令，此选项被调用`--scan`.

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

请注意，`head -10`用于仅打印
输出。

扫描能够使用底层模式匹配功能
这`SCAN`命令与`--pattern`选择。

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

通过管道将输出通过`wc`命令可用于计算特定
对象的种类，按键名：

    $ redis-cli --scan --pattern 'user:*' | wc -l
    3829433

您可以使用`-i 0.01`以在调用之间添加延迟`SCAN`命令。
这将使命令速度变慢，但会显著减少服务器上的负载。

## 发布/订阅模式

CLI 能够在 Redis 发布/订阅通道中发布消息，使用
这`PUBLISH`命令。订阅频道以接收
消息不同 - 终端被阻止并等待
消息，因此这在`redis-cli`.与
其他特殊模式 此模式未通过使用特殊选项启用，
但只需使用`SUBSCRIBE`或`PSUBSCRIBE`命令，这些命令在
交互或命令模式：

    $ redis-cli PSUBSCRIBE '*'
    Reading messages... (press Ctrl-C to quit)
    1) "PSUBSCRIBE"
    2) "*"
    3) (integer) 1

这*阅读消息*消息显示我们进入了发布/订阅模式。
当另一个客户端在某个通道中发布某些消息时，例如使用命令`redis-cli PUBLISH mychannel mymessage`，则发布/订阅模式下的 CLI 将显示如下内容：

    1) "pmessage"
    2) "*"
    3) "mychannel"
    4) "mymessage"

这对于调试发布/订阅问题非常有用。
要退出发布/订阅模式，只需处理`CTRL-C`.

## 监视在 Redis 中执行的命令

与发布/订阅模式类似，监视模式会自动进入
一旦您使用`MONITOR`命令。活动 Redis 实例接收的所有命令都将打印到标准输出中：

    $ redis-cli MONITOR
    OK
    1460100081.165665 [0 127.0.0.1:51706] "set" "shipment:8000736522714:status" "sorting"
    1460100083.053365 [0 127.0.0.1:51707] "get" "shipment:8000736522714:status"

请注意，可以使用 它来管道输出，以便您可以监视
对于使用工具的特定模式，例如`grep`.

## 监控 Redis 实例的延迟

Redis 通常用于延迟非常关键的上下文中。延迟
涉及应用程序中的多个移动部件，来自客户端库
到网络堆栈，到 Redis 实例本身。

这`redis-cli`具有多个用于研究 Redis 延迟的设施
实例并了解延迟的最大、平均值和分布。

基本的延迟检查工具是`--latency`选择。使用这个
选项 CLI 运行一个循环，其中`PING`命令被发送到 Redis
实例和接收回复的时间是度量的。发生这种情况 100
每秒的次数和统计信息在控制台中实时更新：

    $ redis-cli --latency
    min: 0, max: 1, avg: 0.19 (427 samples)

统计数据以毫秒为单位提供。通常，平均延迟
一个非常快的实例往往被高估了一点，因为
由于正在运行的系统的内核调度程序而导致的延迟`redis-cli`
本身，所以0.19以上的平均延迟可能很容易是0.01或更低。
然而，这通常不是一个大问题，因为大多数开发人员都对
几毫秒或更长时间的事件。

有时研究最大和平均延迟是有用的
随着时间的推移而发展。这`--latency-history`选项用于
目的：它的工作原理与`--latency`，但每 15 秒（通过
default） 从头开始启动新的采样会话：

    $ redis-cli --latency-history
    min: 0, max: 1, avg: 0.14 (1314 samples) -- 15.01 seconds range
    min: 0, max: 1, avg: 0.18 (1299 samples) -- 15.00 seconds range
    min: 0, max: 1, avg: 0.20 (113 samples)^C

采样会话的长度可以通过`-i <interval>`选择。

最先进的潜伏期研究工具，也是最复杂的
对于没有经验的用户来说，就是能够使用彩色终端的用户
以显示延迟范围。您将看到一个彩色输出，指示
不同百分比的样本，以及指示的不同 ASCII 字符
不同的延迟数字。此模式是使用`--latency-dist`
选择：

    $ redis-cli --latency-dist
    (output not displayed, requires a color terminal, try it!)

内部还实现了另一个非常不寻常的延迟工具`redis-cli`.
它不检查 Redis 实例的延迟，而是检查
计算机正在运行`redis-cli`.这种延迟是内核调度程序固有的，
虚拟化实例情况下的虚拟机管理程序，依此类推。

Redis称之为*固有延迟*因为它对程序员来说大多是不透明的。
如果 Redis 实例具有高延迟，而不考虑所有明显的事情
这可能是源原因，值得检查一下什么是最好的系统
可以通过运行来完成`redis-cli`在这个特殊模式下，直接在系统中
正在运行 Redis 服务器。

通过测量固有延迟，您知道这是基线，
并且 Redis 无法超越您的系统。为了运行 CLI
在此模式下，使用`--intrinsic-latency <test-time>`.请注意，测试时间以秒为单位，指示测试应运行多长时间。

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

重要说明：此命令必须在运行 Redis 服务器实例的计算机上执行，而不是在其他主机上执行。它不会连接到 Redis 实例，而是在本地执行测试。

在上述情况下，系统不能做得比最差的739微秒更好
案例延迟，因此可以预期某些查询偶尔运行不到 1 毫秒。

## 远程备份 RDB 文件

在 Redis 复制的首次同步期间，主副本和副本
以RDB文件的形式交换整个数据集。此功能被利用
由`redis-cli`为了提供远程备份设施，允许
将 RDB 文件从任何 Redis 实例传输到正在运行的本地计算机
`redis-cli`.要使用此模式，请使用`--rdb <dest-filename>`
选择：

    $ redis-cli --rdb /tmp/dump.rdb
    SYNC sent to master, writing 13256 bytes to '/tmp/dump.rdb'
    Transfer finished with success.

这是确保灾难恢复的简单但有效的方法
Redis 实例存在 RDB 备份。在 中使用此选项时
脚本或`cron`作业，请确保检查命令的返回值。
如果它不是零，则发生错误，如以下示例所示：

    $ redis-cli --rdb /tmp/dump.rdb
    SYNC with master failed: -ERR Can't SYNC while not connected with my master
    $ echo $?
    1

## 复制副本模式

CLI 的副本模式是一项高级功能，可用于
Redis 开发人员和用于调试操作。
它允许检查主服务器在复制中发送到其副本的内容
流，以便将写入内容传播到其副本。选项
名称很简单`--replica`.下面是一个工作示例：

    $ redis-cli --replica
    SYNC with master, discarding 13256 bytes of bulk transfer...
    SYNC done. Logging commands from master.
    "PING"
    "SELECT","0"
    "SET","last_name","Enigk"
    "PING"
    "INCR","mycounter"

该命令首先放弃第一次同步的 RDB 文件
，然后以 CSV 格式记录收到的每个命令。

如果您认为某些命令未在副本中正确复制
这是检查正在发生的事情的好方法，也是有用的信息
为了改进错误报告。

## 执行 LRU 仿真

Redis 通常用作缓存[LRU 驱逐](/topics/lru-cache).
根据键数和为
缓存（通过`maxmemory`指令），缓存命中量
错过会改变。有时，模拟命中率是非常
对于正确配置缓存非常有用。

这`redis-cli`具有特殊模式，可执行 GET 和 SET 的模拟
操作，在请求模式中使用 80-20% 的幂律分布。
这意味着 20% 的密钥将在 80% 的次数中被请求，这是一个
缓存方案中的常见分布。

从理论上讲，给定请求和 Redis 内存的分布
开销，应该可以通过分析计算命中率
用数学公式。但是，Redis 可以配置
不同的 LRU 设置（样本数）和 LRU 的实现，其中
在Redis中是近似的，不同版本之间变化很大。同样地
每个密钥的内存量可能会在版本之间发生变化。这就是为什么这个
工具被构建：它的主要动机是测试Redis的LRU的质量
实现，但现在对于测试给定版本的方式也很有用
与最初用于部署的设置一起运行。

要使用此模式，请在测试中指定密钥的数量，并配置合理的`maxmemory`设置为第一次尝试。

重要说明：配置`maxmemory`Redis 配置中的设置
至关重要：如果没有最大内存使用量上限，则命中将
最终是100%，因为所有密钥都可以存储在内存中。如果指定了具有最大内存的键过多，则最终将使用所有计算机 RAM。还需要配置适当的
*最大内存策略*;大多数时候`allkeys-lru`处于选中状态。

在以下示例中，配置了 100MB 的内存限制和 LRU
使用1000万个密钥进行模拟。

警告：测试使用流水线，会给服务器带来压力，不要使用它
使用生产实例。

    $ ./redis-cli --lru-test 10000000
    156000 Gets/sec | Hits: 4552 (2.92%) | Misses: 151448 (97.08%)
    153750 Gets/sec | Hits: 12906 (8.39%) | Misses: 140844 (91.61%)
    159250 Gets/sec | Hits: 21811 (13.70%) | Misses: 137439 (86.30%)
    151000 Gets/sec | Hits: 27615 (18.29%) | Misses: 123385 (81.71%)
    145000 Gets/sec | Hits: 32791 (22.61%) | Misses: 112209 (77.39%)
    157750 Gets/sec | Hits: 42178 (26.74%) | Misses: 115572 (73.26%)
    154500 Gets/sec | Hits: 47418 (30.69%) | Misses: 107082 (69.31%)
    151250 Gets/sec | Hits: 51636 (34.14%) | Misses: 99614 (65.86%)

该程序每秒显示一次统计信息。在最初的几秒钟内，缓存开始填充。未命中率后来稳定在可以预期的实际数字中：

    120750 Gets/sec | Hits: 48774 (40.39%) | Misses: 71976 (59.61%)
    122500 Gets/sec | Hits: 49052 (40.04%) | Misses: 73448 (59.96%)
    127000 Gets/sec | Hits: 50870 (40.06%) | Misses: 76130 (59.94%)
    124250 Gets/sec | Hits: 50147 (40.36%) | Misses: 74103 (59.64%)

对于某些用例，59%的未命中率可能不被接受
100MB 的内存是不够的。观察使用半 GB 内存的示例。经过几次
分钟输出稳定到以下数字：

    140000 Gets/sec | Hits: 135376 (96.70%) | Misses: 4624 (3.30%)
    141250 Gets/sec | Hits: 136523 (96.65%) | Misses: 4727 (3.35%)
    140250 Gets/sec | Hits: 135457 (96.58%) | Misses: 4793 (3.42%)
    140500 Gets/sec | Hits: 135947 (96.76%) | Misses: 4553 (3.24%)

500MB有足够的空间用于密钥数量（1000万）和分发（80-20样式）。
