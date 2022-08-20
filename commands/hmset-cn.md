将指定的字段设置为存储在
`key`.
此命令将覆盖哈希中已存在的任何指定字段。
如果`key`不存在，则会创建一个保存哈希值的新密钥。

@return

@simple字符串回复

@examples

```cli
HMSET myhash field1 "Hello" field2 "World"
HGET myhash field1
HGET myhash field2
```
