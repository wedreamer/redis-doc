返回指定 Redis 集群哈希槽中的密钥数。这
命令仅查询本地数据集, 因此联系节点
未提供指定哈希槽的将始终导致计数
返回零。

    > CLUSTER COUNTKEYSINSLOT 7000
    (integer) 50341

@return

@integer回复：指定哈希槽中的键数, 如果哈希槽无效, 则为错误。
