更改密钥的上次访问时间。
如果密钥不存在, 则忽略该密钥。

@return

@integer回复：触摸的键数。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
TOUCH key1 key2
```
