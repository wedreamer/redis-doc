为客户端订阅指定的分片通道。

在 Redis 集群中，分片通道通过用于将密钥分配给插槽的相同算法分配给插槽。
客户端可以订阅覆盖插槽（主/副本）的节点，以接收已发布的消息。
所有指定的分片通道都需要属于单个插槽才能在给定的插槽中订阅`SSUBSCRIBE`叫
客户端可以通过单独的插槽订阅跨不同插槽的通道`SSUBSCRIBE`叫。

有关分片发布/订阅的更多信息，请参阅[分片酒吧/子](/topics/pubsub#sharded-pubsub).

@examples

    > ssubscribe orders
    Reading messages... (press Ctrl-C to quit)
    1) "ssubscribe"
    2) "orders"
    3) (integer) 1
    1) "smessage"
    2) "orders"
    3) "hello"
