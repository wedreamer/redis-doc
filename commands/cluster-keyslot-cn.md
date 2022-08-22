返回一个整数, 该整数标识指定键哈希到的哈希槽。
此命令主要用于调试和测试, 因为它公开了
通过 API 实现哈希算法的底层 Redis。
此命令的示例用例：

1.  客户端库可以使用Redis来测试自己的哈希算法, 生成随机密钥并使用本地实现和使用Redis对其进行哈希处理`CLUSTER KEYSLOT`命令, 然后检查结果是否相同。
2.  人类可以使用此命令来检查哈希槽是什么, 然后检查负责给定密钥的关联 Redis 集群节点。

## 例

    > CLUSTER KEYSLOT somekey
    (integer) 11058
    > CLUSTER KEYSLOT foo{hash_tag}
    (integer) 2515
    > CLUSTER KEYSLOT bar{hash_tag}
    (integer) 2515

请注意, 该命令实现了完整的哈希算法, 包括对**哈希标签**, 这是 Redis Cluster 键哈希算法的特殊属性, 它只对两者之间的内容进行哈希处理`{`和`}`如果在键名内找到这样的模式, 则强制多个键由同一节点处理。

@return

@integer回复：哈希槽号。
