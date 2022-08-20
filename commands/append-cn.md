如果`key`已存在并且是一个字符串，此命令将追加`value`在
字符串的末尾。
如果`key`不存在它被创建并设置为空字符串，因此`APPEND`
将类似于`SET`在这种特殊情况下。

@return

@integer回复：追加操作后字符串的长度。

@examples

```cli
EXISTS mykey
APPEND mykey "Hello"
APPEND mykey " World"
GET mykey
```

## 模式：时间序列

这`APPEND`命令可用于创建一个非常紧凑的表示形式
固定大小的样本列表，通常称为*时间序列*.
每次有新样本到达时，我们都可以使用以下命令存储它

    APPEND timeseries "fixed-size sample"

访问时间序列中的单个元素并不难：

*   `STRLEN`可用于获取样品数量。
*   `GETRANGE`允许随机访问元素。
    如果我们的时间序列具有相关的时间信息，我们可以轻松实现
    二进制搜索以获得范围组合`GETRANGE`使用 Lua 脚本
    引擎在 Redis 2.6 中可用。
*   `SETRANGE`可用于覆盖现有时间序列。

这种模式的局限性在于，我们被迫进入仅追加模式。
的操作，没有办法轻松地将时间序列切割到给定的大小
因为 Redis 当前缺少能够修剪字符串对象的命令。
然而，以这种方式存储的时间序列的空间效率是显着的。

提示：可以根据当前的Unix切换到不同的密钥
时间，通过这种方式，可以只拥有相对少量的
每个键的样本，以避免处理非常大的键，并制作此模式
更友好地分布在许多 Redis 实例中。

使用固定大小的字符串对传感器温度进行采样的示例（使用
二进制格式在实际实现中更好）。

```cli
APPEND ts "0043"
APPEND ts "0035"
GETRANGE ts 0 3
GETRANGE ts 4 7
```
