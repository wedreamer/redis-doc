此命令类似于`SINTER`，但它不返回结果集，而是仅返回结果的基数。
返回集合的基数，该基数将由所有给定集合的交集产生。

不存在的键被视为空集。
当其中一个键是空集时，结果集也是空的（因为集与空集的交集总是导致空集）。

默认情况下，该命令计算所有给定集的交集的基数。
当提供可选`LIMIT`参数（默认为 0，表示无限制），如果交集基数在计算中途达到极限，则算法将退出并生成极限作为基数。
这种实现可确保在限制低于实际交集基数的情况下显著加快查询速度。

@return

@integer回复：生成的交集中的元素数。

@examples

```cli
SADD key1 "a"
SADD key1 "b"
SADD key1 "c"
SADD key1 "d"
SADD key2 "c"
SADD key2 "d"
SADD key2 "e"
SINTER key1 key2
SINTERCARD 2 key1 key2
SINTERCARD 2 key1 key2 LIMIT 1
```
