将指定成员添加到存储在 的集合中`key`.
已是此集合成员的指定成员将被忽略。
如果`key`不存在, 则在添加指定的集之前创建一个新集
成员。

当值存储在`key`不是集合。

@return

@integer回复：添加到集合中的元素数, 不包括
集合中已存在的所有元素。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SADD myset "World"
SMEMBERS myset
```
