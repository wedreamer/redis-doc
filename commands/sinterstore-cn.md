此命令等于`SINTER`, 但不是返回结果集, 
它存储在`destination`.

如果`destination`已经存在, 它被覆盖了。

@return

@integer回复：结果集中的元素数。

@examples

```cli
SADD key1 "a"
SADD key1 "b"
SADD key1 "c"
SADD key2 "c"
SADD key2 "d"
SADD key2 "e"
SINTERSTORE key key1 key2
SMEMBERS key
```
