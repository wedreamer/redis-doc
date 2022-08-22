重 命名`key`自`newkey`.
当出现以下情况时, 它会返回错误`key`不存在。
如果`newkey`已经存在, 当这种情况发生时, 它被覆盖了`RENAME`执行隐式`DEL`操作, 因此, 如果已删除的密钥包含非常大的值, 则可能会导致高延迟, 即使`RENAME`本身通常是一个恒定时间操作。

在群集模式下, 两者`key`和`newkey`必须在同一处**哈希槽**, 这意味着在实践中, 只有具有相同哈希标记的键才能在群集中可靠地重命名。

@return

@simple字符串回复

@examples

```cli
SET mykey "Hello"
RENAME mykey myotherkey
GET myotherkey
```

## 行为更改历史记录

*   `>= 3.2.0`：当源名称和目标名称相同时, 该命令不再返回错误。
