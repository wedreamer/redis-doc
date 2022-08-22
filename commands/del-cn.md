删除指定的键。
如果密钥不存在, 则忽略该密钥。

@return

@integer回复：已删除的密钥数。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
DEL key1 key2 key3
```
