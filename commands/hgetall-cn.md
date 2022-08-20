返回存储在`key`.
在返回值中，每个字段名称后跟其值，因此长度
的回复大小是哈希值的两倍。

@return

@array回复：存储在哈希中的字段及其值的列表，或
空列表在`key`不存在。

@examples

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HGETALL myhash
```
