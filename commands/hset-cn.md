集`field`存储在`key`自`value`.
如果`key`不存在, 则会创建一个保存哈希值的新密钥。
如果`field`哈希中已存在, 它将被覆盖。

@return

@integer回复：已添加的字段数。

@examples

```cli
HSET myhash field1 "Hello"
HGET myhash field1
```
