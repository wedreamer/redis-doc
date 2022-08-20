返回存储在`key`.
在以下情况下返回错误`key`保存非字符串值。

@return

@integer回复：字符串的长度`key`或`0`什么时候`key`不
存在。

@examples

```cli
SET mykey "Hello world"
STRLEN mykey
STRLEN nonexisting
```
