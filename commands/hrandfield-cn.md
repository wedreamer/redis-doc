当仅调用`key`参数, 从存储在`key`.

如果提供`count`参数为正数, 返回数组**不同领域**.
数组的长度为`count`或哈希值的字段数  (`HLEN`,  , 以较低者为准。

如果调用时带有否定值`count`, 则行为发生更改, 并且允许命令返回**同一字段多次**.
在这种情况下, 返回字段的数量是指定字段的绝对值`count`.

可选`WITHVALUES`修饰符更改回复, 使其包含随机选择的哈希字段的相应值。

@return

@bulk字符串回复：无需附加`count`参数, 则该命令返回包含随机选择的字段的批量回复, 或者`nil`什么时候`key`不存在。

@array回复：当附加`count`参数被传递, 命令返回字段数组, 或空数组`key`不存在。
如果`WITHVALUES`使用修饰符, 回复是列表字段及其值来自哈希。

@examples

```cli
HMSET coin heads obverse tails reverse edge null
HRANDFIELD coin
HRANDFIELD coin
HRANDFIELD coin -5 WITHVALUES
```

## 指定传递计数时的行为

当`count`参数是正值, 此命令的行为如下：

*   不返回重复字段。
*   如果`count`大于哈希中的字段数, 该命令将仅返回整个哈希, 而不返回其他字段。
*   回复中的字段顺序并不是真正随机的, 因此如果需要, 由客户端来洗牌它们。

当`count`为负值, 行为更改如下：

*   重复字段是可能的。
*   完全`count`字段或空数组 (如果哈希为空 (不存在) ) ) ) 始终返回。
*   回复中的字段顺序确实是随机的。
