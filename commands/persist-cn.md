删除 上的现有超时`key`，转动钥匙从*挥发性的*（一把钥匙
设置了过期）*持续*（由于没有超时而永不过期的密钥
是关联的）。

@return

@integer回复，具体而言：

*   `1`如果超时已被删除。
*   `0`如果`key`不存在或没有关联的超时。

@examples

```cli
SET mykey "Hello"
EXPIRE mykey 10
TTL mykey
PERSIST mykey
TTL mykey
```
