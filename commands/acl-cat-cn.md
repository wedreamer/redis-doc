该命令显示可用的 ACL 类别 (如果不带参数) 。
如果给定了类别名称, 则该命令将显示
指定的类别。

ACL 类别对于创建包含或
一次排除大量命令, 而不指定每个命令
命令。例如, 以下规则将允许用户`karin`执行
除了可能影响服务器的最危险的操作之外的所有内容
稳定性：

    ACL SETUSER karin on +@all -@dangerous

我们首先将所有命令添加到命令集中, `karin`是能够
执行, 但随后我们删除所有危险的命令。

检查所有可用类别非常简单：

    > ACL CAT
     1) "keyspace"
     2) "read"
     3) "write"
     4) "set"
     5) "sortedset"
     6) "list"
     7) "hash"
     8) "string"
     9) "bitmap"
    10) "hyperloglog"
    11) "geo"
    12) "stream"
    13) "pubsub"
    14) "admin"
    15) "fast"
    16) "slow"
    17) "blocking"
    18) "dangerous"
    19) "connection"
    20) "transaction"
    21) "scripting"

然后, 我们可能想知道哪些命令是给定类别的一部分：

    > ACL CAT dangerous
     1) "flushdb"
     2) "acl"
     3) "slowlog"
     4) "debug"
     5) "role"
     6) "keys"
     7) "pfselftest"
     8) "client"
     9) "bgrewriteaof"
    10) "replicaof"
    11) "monitor"
    12) "restore-asking"
    13) "latency"
    14) "replconf"
    15) "pfdebug"
    16) "bgsave"
    17) "sync"
    18) "config"
    19) "flushall"
    20) "cluster"
    21) "info"
    22) "lastsave"
    23) "slaveof"
    24) "swapdb"
    25) "module"
    26) "restore"
    27) "migrate"
    28) "save"
    29) "shutdown"
    30) "psync"
    31) "sort"

@return

@array回复：ACL 类别列表或给定类别中的命令列表。如果将无效的类别名称作为参数给出, 则该命令可能会返回错误。
