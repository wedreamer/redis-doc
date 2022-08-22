返回与指定的值关联的值`fields`存储在
`key`.

对于每个`field`哈希中不存在的, 一个`nil`返回值。
由于不存在的键被视为空哈希, 因此正在运行`HMGET`对
不存在的`key`将返回列表`nil`值。

@return

@array回复：与给定字段关联的值列表, 在同一个字段中
按要求订购。

```cli
HSET myhash field1 "Hello"
HSET myhash field2 "World"
HMGET myhash field1 field2 nofield
```
