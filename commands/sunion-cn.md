返回由所有给定集合的并集生成的集合的成员。

例如：

    key1 = {a,b,c,d}
    key2 = {c}
    key3 = {a,c,e}
    SUNION key1 key2 key3 = {a,b,c,d,e}

不存在的键被视为空集。

@return

@array回复：列出结果集的成员。

@examples

```cli
SADD key1 "a"
SADD key1 "b"
SADD key1 "c"
SADD key2 "c"
SADD key2 "d"
SADD key2 "e"
SUNION key1 key2
```
