***

## 标题：“Redis 数据类型”&#xA;链接标题： “数据类型”&#xA;说明：Redis 支持的数据类型概述&#xA;体重： 2&#xA;别名：&#xA;\- /docs/manual/data-types&#xA;\- /主题/数据类型

Redis 是一个数据结构服务器。
Redis 的核心是提供一系列本机数据类型，可帮助您解决各种问题，从[缓存](/docs/manual/client-side-caching/)自[排队](/docs/data-types/lists/)自[事件处理](/docs/data-types/streams/).
以下是每种数据类型的简短说明，以及指向更广泛概述和命令参考的链接。

如果您想尝试全面的教程，请参阅[Redis 数据类型教程](/docs/data-types/tutorial/).

## 核心

### 字符串

[Redis 字符串](/docs/data-types/strings)是最基本的 Redis 数据类型，表示一个字节序列。
有关详细信息，请参阅：

*   [Redis 字符串概述](/docs/data-types/strings/)
*   [Redis 字符串命令引用](/commands/?group=string)

### 列表

[Redis 列表](/docs/data-types/lists)是按广告订单排序的字符串列表。
有关详细信息，请参阅：

*   [Redis 列表概述](/docs/data-types/lists/)
*   [Redis list 命令引用](/commands/?group=list)

### 集

[Redis sets](/docs/data-types/sets)是唯一字符串的无序集合，其作用类似于您喜欢的编程语言中的集合（例如，[Java HashSets](https://docs.oracle.com/javase/7/docs/api/java/util/HashSet.html),[Python sets](https://docs.python.org/3.10/library/stdtypes.html#set-types-set-frozenset)，依此类推）。
使用 Redis 集， 您可以添加、 删除和测试 O（1） 时间的存在 （换句话说， 不管集合元素的数量）。
有关详细信息，请参阅：

*   [Redis 集概述](/docs/data-types/sets/)
*   [Redis set 命令引用](/commands/?group=set)

### 散 列

[Redis 哈希](/docs/data-types/hashes)是建模为字段值对集合的记录类型。
因此，Redis哈希类似于[Python字典](https://docs.python.org/3/tutorial/datastructures.html#dictionaries),[Java HashMaps](https://docs.oracle.com/javase/8/docs/api/java/util/HashMap.html)和[红宝石哈希](https://ruby-doc.org/core-3.1.2/Hash.html).
有关详细信息，请参阅：

*   [Redis 哈希概述](/docs/data-types/hashes/)
*   [Redis 哈希命令引用](/commands/?group=hash)

### 排序集

[Redis 排序集](/docs/data-types/sorted-sets)是按每个字符串的关联分数保持顺序的唯一字符串的集合。
有关详细信息，请参阅：

*   [Redis 排序集概述](/docs/data-types/sorted-sets)
*   [Redis 排序集命令引用](/commands/?group=sorted-set)

### 流

一个[Redis stream](/docs/data-types/streams)是一种数据结构，其作用类似于仅追加日志。
流有助于按事件发生的顺序记录事件，然后将其联合起来进行处理。
有关详细信息，请参阅：

*   [Redis Streams 概述](/docs/data-types/streams)
*   [Redis Streams 命令引用](/commands/?group=streams)
*   [Redis Streams 教程](/docs/data-types/streams-tutorial)

### 地理空间索引

[Redis 地理空间索引](/docs/data-types/geospatial)对于查找给定地理半径或边界框内的位置非常有用。
有关详细信息，请参阅：

*   [Redis 地理空间索引概述](/docs/data-types/geospatial/)
*   [Redis 地理空间索引命令参考](/commands/?group=geo)

### 位图

[雷迪斯位图](/docs/data-types/bitmaps/)允许您对字符串执行按位运算。
有关详细信息，请参阅：

*   [Redis 位图概述](/docs/data-types/bitmaps/)
*   [Redis 位图命令引用](/commands/?group=bitmap)

### 比特菲尔德

[雷迪斯比特菲尔德](/docs/data-types/bitfields/)在字符串值中有效地对多个计数器进行编码。
Bitfields 提供原子获取、设置和递增操作，并支持不同的溢出策略。
有关详细信息，请参阅：

*   [Redis 位字段概述](/docs/data-types/bitfields/)
*   这`BITFIELD`命令。

### 超级日志

这[Redis HyperLogLog](/docs/data-types/hyperloglogs)数据结构提供对大型集合的基数（即元素数）的概率估计。有关详细信息，请参阅：

*   [Redis HyperLogLog 概述](/docs/data-types/hyperloglogs)
*   [Redis HyperLogLog 命令参考](/commands/?group=hyperloglog)

## 扩展

要扩展所包含的数据类型提供的功能，请使用以下选项之一：

1.  编写自己的自定义[Lua 中的服务器端函数](/docs/manual/programmability/).
2.  使用[模块接口](/docs/reference/modules/)或查看[社区支持的模块](/docs/modules/).
3.  用[断续器](/docs/stack/json/),[查询](/docs/stack/search/),[时间序列](/docs/stack/timeseries/)和提供的其他功能[Redis Stack](/docs/stack/).
