从提供的键名列表中的第一个非空排序集弹出一个或多个元素，这些元素是成员分数对。

`ZMPOP`和`BZMPOP`类似于以下更受限制的命令：

*   `ZPOPMIN`或`ZPOPMAX`它只需要一个键，并且可以返回多个元素。
*   `BZPOPMIN`或`BZPOPMAX`它接受多个键，但只从一个键返回一个元素。

看`BZMPOP`用于此命令的阻塞变体。

当`MIN`使用修饰符，弹出的元素是第一个非空排序集中得分最低的元素。这`MAX`修饰符会导致具有最高分数的元素被弹出。
可选`COUNT`可用于指定要弹出的元素数，并且默认情况下设置为 1。

弹出元素的数量是排序集的基数中的最小值，并且`COUNT`的值。

@return

@array回复：具体而言：

*   一个`nil`当没有元素可以弹出时。
*   一个双元素数组，其中第一个元素是从中弹出元素的键的名称，第二个元素是弹出元素的数组。元素数组中的每个条目也是一个包含成员及其分数的数组。

@examples

```cli
ZMPOP 1 notsuchkey MIN
ZADD myzset 1 "one" 2 "two" 3 "three"
ZMPOP 1 myzset MIN
ZRANGE myzset 0 -1 WITHSCORES
ZMPOP 1 myzset MAX COUNT 10
ZADD myzset2 4 "four" 5 "five" 6 "six"
ZMPOP 2 myzset myzset2 MIN COUNT 10
ZRANGE myzset 0 -1 WITHSCORES
ZMPOP 2 myzset myzset2 MAX COUNT 10
ZRANGE myzset2 0 -1 WITHSCORES
EXISTS myzset myzset2
```
