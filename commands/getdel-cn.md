获取的价值`key`并删除密钥。
此命令类似于`GET`，除了它还会在成功时删除键（当且仅当键的值类型为字符串时）。

@return

@bulk字符串回复：的值`key`,`nil`什么时候`key`不存在，或者如果键的值类型不是字符串，则会出现错误。

@examples

```cli
SET mykey "Hello"
GETDEL mykey
GET mykey
```
