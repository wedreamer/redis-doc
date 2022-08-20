返回存储在`key`.

@return

@integer回复：哈希中的字段数，或`0`什么时候`key`不存在。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HLEN myhash
```
