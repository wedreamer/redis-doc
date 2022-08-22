返回与指定值关联的分数`members`在排序集中存储在`key`.

对于每个`member`在排序集中不存在, 则`nil`返回值。

@return

@array回复：分数列表或`nil`与指定的`member`值 (双精度浮点数,  , 
表示为字符串。

@examples

```cli
ZADD myzset 1 "one"
ZADD myzset 2 "two"
ZMSCORE myzset "one" "two" "nofield"
```
