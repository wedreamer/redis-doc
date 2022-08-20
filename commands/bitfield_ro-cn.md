的只读变体`BITFIELD`命令。
它就像原来的`BITFIELD`但只接受`!GET`子命令，可以安全地在只读副本中使用。

自原`BITFIELD`有`!SET`和`!INCRBY`选项，它在技术上被标记为 Redis 命令表中的写入命令。
因此，即使连接处于只读模式，Redis 集群中的只读副本也会将其重定向到主实例（请参阅`READONLY`Redis Cluster 的命令）。

自 Redis 6.2 以来，`BITFIELD_RO`引入变体是为了允许`BITFIELD`行为，而不会破坏命令标志上的兼容性。

查看原文`BITFIELD`了解更多详情。

@examples

    BITFIELD_RO hello GET i8 16

@return

@array回复：一个数组，其中每个条目都是在同一位置给出的子命令的相应结果。
