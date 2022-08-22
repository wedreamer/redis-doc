修剪现有列表, 使其仅包含指定的范围
元素。
双`start`和`stop`是从零开始的索引, 其中`0`是第一个元素
的列表 (头部,  , `1`下一个元素依此类推。

例如：`LTRIM foobar 0 2`将修改存储在`foobar`因此
将仅保留列表的前三个元素。

`start`和`end`也可以是负数, 表示从末端偏移
, 其中`-1`是列表的最后一个元素, `-2`倒数第二
元素等。

超出范围的索引不会产生错误：如果`start`大于
列表的末尾, 或`start > end`, 结果将是一个空列表 (其中
原因`key`要删除) 。
如果`end`大于列表的末尾, Redis 会像对待最后一个一样对待它
元素。

常见用途`LTRIM`与`LPUSH`/`RPUSH`.
例如：

    LPUSH mylist someelement
    LTRIM mylist 0 99

这对命令将在列表中推送一个新元素, 同时确保
列表不会增长到大于 100 个元素。
例如, 当使用 Redis 存储日志时, 这非常有用。
重要的是要注意, 当以这种方式使用时`LTRIM`是 O () )  操作
因为在一般情况下, 只有一个元素从尾部移除
列表。

@return

@simple字符串回复

@examples

```cli
RPUSH mylist "one"
RPUSH mylist "two"
RPUSH mylist "three"
LTRIM mylist 1 -1
LRANGE mylist 0 -1
```
