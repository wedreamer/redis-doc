当群集客户机收到`-ASK`重定向，`ASKING`命令被发送到目标节点，后跟重定向的命令。
这通常由群集客户端自动完成。

如果`-ASK`在事务期间接收重定向，在将完整的事务发送到目标节点之前，只需要向目标节点发送一个 ASK 命令。

看[Redis 集群规范中的 ASK 重定向](/topics/cluster-spec#ask-redirection)了解详情。

@return

@simple字符串回复：`OK`.
