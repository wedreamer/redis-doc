从存储在 的集合中删除指定的成员`key`.
不属于此集成员的指定成员将被忽略。
如果`key`不存在，它被视为空集，此命令返回
`0`.

当值存储在`key`不是集合。

@return

@integer回复：从集合中删除的成员数，而不是
包括非现有成员。

@examples

```cli
SADD myset "one"
SADD myset "two"
SADD myset "three"
SREM myset "one"
SREM myset "four"
SMEMBERS myset
```
