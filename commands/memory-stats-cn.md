这`MEMORY STATS`命令返回有关 内存使用情况的 @array回复
服务器。

有关内存使用情况的信息以指标及其各自的指标形式提供
值。报告了以下指标：

*   `peak.allocated`：Redis 消耗的峰值内存 (以字节为单位)  (请参阅`INFO`的
    `used_memory_peak`)
*   `total.allocated`：Redis 使用其分配的字节总数
    分配器 (请参见`INFO`的`used_memory`)
*   `startup.allocated`：启动时 Redis 消耗的初始内存量
    以字节为单位 (请参阅`INFO`的`used_memory_startup`)
*   `replication.backlog`：复制积压工作的大小 (以字节为单) )  (请参阅
    `INFO`的`repl_backlog_active`)
*   `clients.slaves`：所有副本开销的总大小 (以字节为单) )  (输出
    和查询缓冲区、连接上下文) 
*   `clients.normal`：所有客户端开销的总大小 (以字节为单) )  (输出
    和查询缓冲区、连接上下文) 
*   `cluster.links`：群集链接的内存使用情况 (在 Redis 7.0 中添加, 请参阅`INFO`的`mem_cluster_links`).
*   `aof.buffer`：AOF 相关缓冲区的总和大小 (以字节为单) ) 。
*   `lua.caches`：Lua 脚本开销的总和 (以字节为单) ) 
    缓存
*   `dbXXX`：对于每个服务器的数据库, 主数据库和数据库的开销
    到期词典  (`overhead.hashtable.main`和
    `overhead.hashtable.expires`, 分别) 以字节为单位报告
*   `overhead.total`：所有间接费用的总和, 即`startup.allocated`,
    `replication.backlog`,`clients.slaves`,`clients.normal`,`aof.buffer`和
    用于管理的内部数据结构的那些
    Redis 键空间 (请参见`INFO`的`used_memory_overhead`)
*   `keys.count`：存储在
    服务器
*   `keys.bytes-per-key`：比例之间**净内存使用情况**(`total.allocated`
    减去`startup.allocated`)  和`keys.count`
*   `dataset.bytes`：数据集的大小 (以字节为单), ) , 即`overhead.total`
    减去`total.allocated` (请参见`INFO`的`used_memory_dataset`)
*   `dataset.percentage`： 百分比`dataset.bytes`出网
    内存使用情况
*   `peak.percentage`： 百分比`peak.allocated`出
    `total.allocated`
*   `fragmentation`： 请参阅`INFO`的`mem_fragmentation_ratio`

@return

@array回复：内存使用情况指标及其值的嵌套列表

**关于本手册页中使用的“奴隶”一词的说明**：从 Redis 5 开始, 如果不是为了向后兼容, Redis 项目不再使用“从属”一词。不幸的是, 在此命令中, 单词 slave 是协议的一部分, 因此只有当此 API 自然弃用时, 我们才能删除此类事件。
