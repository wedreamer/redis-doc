返回由所有给定的交集生成的集合的成员
集。

例如：

    key1 = {a,b,c,d}
    key2 = {c}
    key3 = {a,c,e}
    SINTER key1 key2 key3 = {c}

不存在的键被视为空集。
当其中一个键为空集时，生成的集也是空的（因为
集合与空集的交集始终导致空集）。

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
SINTER key1 key2
```
