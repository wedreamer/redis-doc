返回与 关联的值`field`存储在`key`.

@return

@bulk字符串回复：与 关联的值`field`或`nil`什么时候`field`莫
存在于哈希或`key`不存在。

@examples

```cli
HSET myhash field1 "foo"
HGET myhash field1
HGET myhash field2
```
