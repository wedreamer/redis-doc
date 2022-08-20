返回存储在`key`.

@return

@integer回复：集合的基数（元素数），或`0`如果`key`
不存在。

@examples

```cli
SADD myset "Hello"
SADD myset "World"
SCARD myset
```
