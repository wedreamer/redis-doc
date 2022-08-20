返回存储在`key`.

这与运行具有相同的效果`SINTER`带有一个参数`key`.

@return

@array回复：集合的所有元素。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SMEMBERS myset
```
