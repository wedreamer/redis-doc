重 命名`key`自`newkey`如果`newkey`尚不存在。
当出现以下情况时, 它会返回错误`key`不存在。

在群集模式下, 两者`key`和`newkey`必须在同一处**哈希槽**, 这意味着在实践中, 只有具有相同哈希标记的键才能在群集中可靠地重命名。

@return

@integer回复, 具体而言：

*   `1`如果`key`已重命名为`newkey`.
*   `0`如果`newkey`已存在。

@examples

```cli
SET mykey "Hello"
SET myotherkey "World"
RENAMENX mykey myotherkey
GET myotherkey
```
