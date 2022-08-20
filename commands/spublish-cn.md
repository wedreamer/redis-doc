将消息发布到给定的分片通道。

在 Redis 集群中，分片通道通过用于将密钥分配给插槽的相同算法分配给插槽。
分片消息必须发送到拥有分片通道哈希到的插槽的节点。
集群确保将已发布的分片消息转发到分片中的所有节点，以便客户端可以通过连接到分片中的任何一个节点来订阅分片通道。

有关分片 pubsub 的更多信息，请参阅[分片酒吧](/topics/pubsub#sharded-pubsub).

@return

@integer回复：收到消息的客户端数。

@examples

例如，以下命令发布到通道`orders`订阅者已在等待消息。

    > spublish orders hello
    (integer) 1
