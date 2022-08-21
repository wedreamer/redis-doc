***

标题： Redis 简介
链接标题： "关于"
体重： 1
描述： 了解 Redis 开源项目
别名：

*   /主题/简介
*   /嗡嗡声

***

Redis 是一个开源（BSD 许可），内存中 **数据结构存储** 用作数据库、缓存、消息代理和流式处理引擎。
Redis 提供[数据结构](/docs/data-types/)
如
[字符串](/docs/data-types/strings/),
[散 列](/docs/data-types/hashes/),
[列表](/docs/data-types/lists/),
[集合](/docs/data-types/lists/),
[已排序集](/docs/data-types/sorted-sets/)使用范围查询，
[位图](/docs/data-types/bitmaps/),
[超日志](/docs/data-types/hyperloglogs/),
[地理空间索引](/docs/data-types/geospatial/),
[流](/docs/data-types/streams/).

Redis内置了
[复制](/topics/replication),
[Lua 脚本](/commands/eval),
[LRU cache](/topics/lru-cache),
[事务](/topics/transactions)，
不同级别的[磁盘持久性](/topics/persistence)，
并通过以下方式提供高可用性 [Redis Sentinel](/topics/sentinel)和自动分区[Redis Cluster](/topics/cluster-tutorial).

您可以运行**原子操作**在这些类型上，例如
[追加到字符串](/commands/append);
[递增哈希中的值](/commands/hincrby);
[将元素推送到列表](/commands/lpush);
[计算集交集](/commands/sinter),
[union](/commands/sunion)和[diff](/commands/sdiff);
[获取有序集中排名最高的成员](/commands/zrange).

为了实现最佳性能，Redis 工作在**内存中数据集**.
根据您的使用场景，Redis 可以保留您的数据通过定期[将数据集转储到磁盘](/topics/persistence#snapshotting)或由[将每个命令追加到基于磁盘的日志](/topics/persistence#append-only-file).
如果只需要功能丰富的网络内存中缓存，也可以禁用持久性。

Redis 支持[异步复制](/topics/replication)，具有快速无阻塞同步和自动重新连接，并在网络拆分时进行部分重新同步。

Redis 还包括：

*   [事务](/topics/transactions)
*   [发布/订阅](/topics/pubsub)
*   [Lua 脚本](/commands/eval)
*   [具有有限生存时间的 keys](/commands/expire)
*   [LRU key 淘汰](/topics/lru-cache)
*   [自动故障转移](/topics/sentinel)

您可以从以下位置使用 Redis：[大多数编程语言](/clients).

Redis 写在**ANSI C**并适用于大多数POSIX系统，如Linux，\*BSD 和 Mac OS X，没有外部依赖关系。
Linux 和 OS X 是 Redis 开发和测试最多的两个操作系统，我们**建议使用 Linux 进行部署**.Redis可能适用于Solaris派生的系统，如SmartOS，只保证 *尽最大努力*.
没有对 Windows 版本的官方支持。
