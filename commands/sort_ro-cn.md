的只读变体`SORT`命令。它与原版一模一样`SORT`但拒绝`STORE`选项, 并且可以安全地在只读副本中使用。

自原`SORT`具有`STORE`选项, 它在技术上被标记为 Redis 命令表中的写入命令。因此, 即使连接处于只读模式, Redis 集群中的只读副本也会将其重定向到主实例 (请参阅`READONLY`Redis Cluster 的命令) 。

这`SORT_RO`引入变体是为了允许`SORT`行为, 而不会破坏命令标志上的兼容性。

查看原文`SORT`了解更多详情。

@examples

    SORT_RO mylist BY weight_*->fieldname GET object_*->fieldname

@return

@array回复：已排序元素的列表。
