设置`key`以握住字符串`value`并设置`key`在给定值后超时
秒数。
此命令等效于执行以下命令：

    SET mykey value
    EXPIRE mykey seconds

`SETEX`是原子的, 可以使用前两个命令复制
在`MULTI`/`EXEC`块。
它是作为给定操作序列的更快替代方案提供的, 
因为当 Redis 用作缓存时, 此操作非常常见。

在以下情况下返回错误`seconds`无效。

@return

@simple字符串回复

@examples

```cli
SETEX mykey 10 "Hello"
TTL mykey
GET mykey
```
