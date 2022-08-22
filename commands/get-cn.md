获取的价值`key`.
如果该键不存在, 则特殊值`nil`返回。
如果值存储在`key`不是字符串, 因为`GET`
仅处理字符串值。

@return

@bulk字符串回复：的值`key`或`nil`什么时候`key`不存在。

@examples

```cli
GET nonexisting
SET mykey "Hello"
GET mykey
```
