返回存储在`key`.
可以返回的不同类型包括：`string`,`list`,`set`,`zset`,
`hash`和`stream`.

@return

@simple字符串回复：类型`key`或`none`什么时候`key`不存在。

@examples

```cli
SET key1 "value"
LPUSH key2 "value"
SADD key3 "value"
TYPE key1
TYPE key2
TYPE key3
```
