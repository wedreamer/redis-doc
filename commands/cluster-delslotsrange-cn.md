这`CLUSTER DELSLOTSRANGE`命令类似于`CLUSTER DELSLOTS`命令, 因为它们都从节点中删除哈希槽。
不同之处在于`CLUSTER DELSLOTS`获取要从节点中删除的哈希槽列表, 而`CLUSTER DELSLOTSRANGE`获取要从节点中删除的插槽范围 (由开始和结束插槽指定) 的列表。

## 例

要从节点中删除插槽 1 2 3 4 5, 请将`CLUSTER DELSLOTS`命令是：

    > CLUSTER DELSLOTS 1 2 3 4 5
    OK

可以通过以下操作完成相同的操作`CLUSTER DELSLOTSRANGE`命令：

    > CLUSTER DELSLOTSRANGE 1 5
    OK

但是, 请注意：

1.  仅当所有指定的插槽都已与节点关联时, 该命令才有效。
2.  如果多次指定同一插槽, 则该命令将失败。
3.  作为命令执行的副作用, 节点可能会进入*下*状态, 因为并非所有哈希槽都已覆盖。

## 在 Redis 集群中的用法

此命令仅在群集模式下有效, 可能适用于
调试 和 为了手动编排群集配置
创建新群集时。它目前未由`redis-cli`,
并且主要是为了API完整性而存在的。

@return

@simple字符串回复：`OK`如果命令成功。否则
返回错误。
