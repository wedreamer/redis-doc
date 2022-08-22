将多个 HyperLogLog 值合并为一个唯一值, 该值将近似
源 HyperLogLog 的观测集合的并集的基数
结构。

计算出的合并 HyperLogLog 设置为目标变量, 即
如果不存在, 则创建 (默认为空的 HyperLogLog) 。

如果目标变量存在, 则将其视为源集之一
其基数将包含在计算的基数中
超级日志。

@return

@simple字符串回复：命令仅返回`OK`.

@examples

```cli
PFADD hll1 foo bar zap a
PFADD hll2 a b c foo
PFMERGE hll3 hll1 hll2
PFCOUNT hll3
```
