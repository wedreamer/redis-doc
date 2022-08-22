此命令类似于`ZRANGE`, 但将结果存储在`<dst>`目标键。

@return

@integer回复：生成的排序集中的元素数。

@examples

```cli
ZADD srczset 1 "one" 2 "two" 3 "three" 4 "four"
ZRANGESTORE dstzset srczset 2 -1
ZRANGE dstzset 0 -1
```
