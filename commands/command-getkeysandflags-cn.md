返回来自完整 Redis 命令及其用法标志的密钥的@array回复。

`COMMAND GETKEYSANDFLAGS`是一个帮助程序命令, 用于从完整的 Redis 命令中查找键, 以及指示每个键的用途的标志。

`COMMAND`提供了有关如何查找每个命令的键名的信息 (请参阅`firstkey`,[主要规格](/topics/key-specs#logical-operation-flags)和`movablekeys`),
但在某些情况下, 不可能找到某些命令的键, 然后必须解析整个命令以发现某些/所有键名称。
您可以使用`COMMAND GETKEYS`或`COMMAND GETKEYSANDFLAGS`直接从 Redis 解析命令的方式中发现键名。

指[主要规格](/topics/key-specs#logical-operation-flags), 了解有关键标志含义的信息。

@return

@array回复：命令中的键列表。
数组的每个元素都是一个数组, 第一个条目中包含键名, 第二个条目中包含标志。

@examples

```cli
COMMAND GETKEYS MSET a b c d e f
COMMAND GETKEYS EVAL "not consulted" 3 key1 key2 key3 arg1 arg2 arg3 argN
COMMAND GETKEYSANDFLAGS LMOVE mylist1 mylist2 left left
```
