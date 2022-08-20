返回所有指定键的值。
对于每个不保存字符串值或不存在的键，特殊
价值`nil`返回。
因此，该操作永远不会失败。

@return

@array回复：指定键处的值列表。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
MGET key1 key2 nonexisting
```
