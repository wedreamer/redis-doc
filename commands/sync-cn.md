从主服务器启动复制流。

这`SYNC`命令由 Redis 副本调用以启动复制
来自主节点的流。在较新版本的 Redis 中, 它已被替换为
`PSYNC`.

有关 Redis 中复制的更多信息, 请查看
[复制页][tr].

[tr]: /topics/replication

@return

**非标准返回值**, 批量传输数据, 然后`PING`并写入来自主服务器的请求。
