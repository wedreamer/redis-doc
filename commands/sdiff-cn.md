返回由第一个
集和所有连续集。

例如：

    key1 = {a,b,c,d}
    key2 = {c}
    key3 = {a,c,e}
    SDIFF key1 key2 key3 = {b,d}

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
SDIFF key1 key2
```
