返回与 关联的值的字符串长度`field`存储在`key`.如果`key`或`field`不存在，则返回 0。

@return

@integer回复：与 关联的值的字符串长度`field`，或零，当`field`不存在于哈希或`key`根本不存在。

@examples

```cli
HMSET myhash f1 HelloWorld f2 99 f3 -256
HSTRLEN myhash f1
HSTRLEN myhash f2
HSTRLEN myhash f3
```
