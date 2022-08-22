递增存储在`field`存储在`key`由
`increment`.
如果`key`不存在, 则会创建一个保存哈希值的新密钥。
如果`field`不存在, 该值设置为`0`在操作之前
执行。

支持的值范围`HINCRBY`限制为 64 位有符号整数。

@return

@integer回复：值`field`在增量操作之后。

@examples

由于`increment`参数是有符号的, 递增和递减
可以执行的操作：

```cli
HSET myhash field 5
HINCRBY myhash field 1
HINCRBY myhash field -1
HINCRBY myhash field -10
```
