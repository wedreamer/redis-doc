从设置值存储区中删除并返回一个或多个随机成员，位于`key`.

此操作类似于`SRANDMEMBER`，从集合中返回一个或多个随机元素，但不删除它。

默认情况下，该命令从集合中弹出单个成员。当提供
可选`count`参数，回复将包含最多`count`成员
取决于集合的基数。

@return

在没有`count`论点：

@bulk字符串回复：已删除的成员，或`nil`什么时候`key`不存在。

当调用`count`论点：

@array回复：删除的成员或空数组`key`不存在。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myset "three"
SPOP myset
SMEMBERS myset
SADD myset "four"
SADD myset "five"
SPOP myset 3
SMEMBERS myset
```

## 返回元素的分布

请注意，当您需要保证返回元素的均匀分布时，此命令不适用。有关用于`SPOP`，查找高德纳采样和弗洛伊德采样算法。
