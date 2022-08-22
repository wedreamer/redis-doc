列出当前*活动分片通道*.

活动分片通道是具有一个或多个订阅者的发布/订阅分片通道。

如果不是`pattern`, 则列出所有通道, 否则如果指定了模式, 则仅列出与指定的 glob 样式模式匹配的通道。

返回的有关活动分片通道的信息位于分片级别, 而不是集群级别。

@return

@array回复：活动通道的列表, 可选择与指定的模式匹配。

@examples

    > PUBSUB SHARDCHANNELS
    1) "orders"
    PUBSUB SHARDCHANNELS o*
    1) "orders"
