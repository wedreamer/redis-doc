`CLUSTER INFO`提供`INFO`有关 Redis 集群重要参数的样式信息。
回复中始终存在以下字段：

    cluster_state:ok
    cluster_slots_assigned:16384
    cluster_slots_ok:16384
    cluster_slots_pfail:0
    cluster_slots_fail:0
    cluster_known_nodes:6
    cluster_size:3
    cluster_current_epoch:6
    cluster_my_epoch:2
    cluster_stats_messages_sent:1483972
    cluster_stats_messages_received:1483968
    total_cluster_links_buffer_limit_exceeded:0

*   `cluster_state`：状态为`ok`如果节点能够接收查询。`fail`如果至少有一个哈希槽未绑定（没有关联的节点），处于错误状态（为其提供服务的节点被标记为 FAIL 标志），或者此节点无法访问大多数主节点。
*   `cluster_slots_assigned`：与某个节点（未未绑定）关联的插槽数。此数字应为 16384，节点才能正常工作，这意味着每个哈希槽都应映射到一个节点。
*   `cluster_slots_ok`：映射到不在 中的节点的哈希槽数`FAIL`或`PFAIL`州。
*   `cluster_slots_pfail`：映射到 中的节点的哈希槽数`PFAIL`州。请注意，这些哈希槽仍可正常工作，只要`PFAIL`状态未提升为`FAIL`通过故障检测算法。`PFAIL`仅表示我们当前无法与节点通信，但可能只是暂时性错误。
*   `cluster_slots_fail`：映射到 中的节点的哈希槽数`FAIL`州。如果此数字不为零，则节点无法为查询提供服务，除非`cluster-require-full-coverage`设置为`no`在配置中。
*   `cluster_known_nodes`：群集中已知节点的总数，包括`HANDSHAKE`状态，该状态当前可能不是群集的适当成员。
*   `cluster_size`：为集群中至少一个哈希槽提供服务的主节点数。
*   `cluster_current_epoch`： 本地`Current Epoch`变量。这用于在故障转移期间创建唯一递增的版本号。
*   `cluster_my_epoch`：`Config Epoch`的节点，我们正在与之交谈。这是分配给此节点的当前配置版本。
*   `cluster_stats_messages_sent`：通过群集节点到节点二进制总线发送的消息数。
*   `cluster_stats_messages_received`：通过群集节点到节点二进制总线接收的消息数。
*   `total_cluster_links_buffer_limit_exceeded`：由于超出`cluster-link-sendbuf-limit`配置。

如果值不为 0，则答复中可能包含以下与消息相关的字段：
每种消息类型都包含有关发送和接收的消息数的统计信息。
以下是这些字段的说明：

*   `cluster_stats_messages_ping_sent`和`cluster_stats_messages_ping_received`：群集总线 PING（不要与客户端命令混淆）`PING`).
*   `cluster_stats_messages_pong_sent`和`cluster_stats_messages_pong_received`：PONG（回复 PING）。
*   `cluster_stats_messages_meet_sent`和`cluster_stats_messages_meet_received`：通过八卦或向新节点发送的握手消息`CLUSTER MEET`.
*   `cluster_stats_messages_fail_sent`和`cluster_stats_messages_fail_received`：将节点 xxx 标记为失败。
*   `cluster_stats_messages_publish_sent`和`cluster_stats_messages_publish_received`：发布/订阅发布传播，请参阅[酒吧酒吧](/topics/pubsub#pubsub).
*   `cluster_stats_messages_auth-req_sent`和`cluster_stats_messages_auth-req_received`：复制副本启动的领导者选举以替换其主节点。
*   `cluster_stats_messages_auth-ack_sent`和`cluster_stats_messages_auth-ack_received`：指示在领导人选举期间投票的消息。
*   `cluster_stats_messages_update_sent`和`cluster_stats_messages_update_received`：另一个节点插槽配置。
*   `cluster_stats_messages_mfstart_sent`和`cluster_stats_messages_mfstart_received`：暂停客户端以进行手动故障转移。
*   `cluster_stats_messages_module_sent`和`cluster_stats_messages_module_received`：模块集群 API 消息。
*   `cluster_stats_messages_publishshard_sent`和`cluster_stats_messages_publishshard_received`：发布/订阅发布分片传播，请参阅[分片酒吧](/topics/pubsub#sharded-pubsub).

有关当前纪元和配置纪元变量的更多信息，请参见[Redis 集群规范文档](/topics/cluster-spec#cluster-current-epoch).

@return

@bulk字符串回复：命名字段和值之间的映射，格式为`<field>:<value>`由两个字节组成的换行符分隔的行`CRLF`.
