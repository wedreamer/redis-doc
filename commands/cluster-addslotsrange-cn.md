这`CLUSTER ADDSLOTSRANGE`类似于`CLUSTER ADDSLOTS`命令，因为它们都为节点分配哈希槽。

这两个命令之间的区别在于`ADDSLOTS`获取要分配给节点的插槽列表，而`ADDSLOTSRANGE`获取要分配给节点的槽范围列表（由开始和结束槽指定）。

## 例

要将插槽 1 2 3 4 5 分配给节点，请`ADDSLOTS`命令是：

    > CLUSTER ADDSLOTS 1 2 3 4 5
    OK

可以通过以下操作完成相同的操作`ADDSLOTSRANGE`命令：

    > CLUSTER ADDSLOTSRANGE 1 5
    OK

## 在 Redis 集群中的用法

此命令仅在集群模式下有效，在以下 Redis 集群操作中很有用：

1.  要创建新的集群，请使用ADDLOTSRANGE，以便最初设置主节点在它们之间拆分可用的哈希槽。
2.  为了修复未分配某些插槽的损坏群集。

@return

@simple字符串回复：`OK`如果命令成功。否则，将返回错误。
