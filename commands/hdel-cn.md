从存储在 的哈希中删除指定的字段`key`.
此哈希中不存在的指定字段将被忽略。
如果`key`不存在, 它被视为空哈希, 此命令返回
`0`.

@return

@integer回复：从哈希中删除的字段数, 而不是
包括指定但不存在的字段。

@examples

```cli
HSET myhash field1 "foo"
HDEL myhash field1
HDEL myhash field2
```
