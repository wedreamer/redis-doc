此命令与`GEORADIUS`唯一的区别是
以经度和纬度值作为要查询的区域的中心, 它采用已存在于由排序集表示的地理空间索引中的成员的名称。

指定成员的位置用作查询的中心。

请检查下面的示例和`GEORADIUS`文档, 了解有关命令及其选项的更多信息。

请注意, `GEORADIUSBYMEMBER_RO`自 Redis 3.2.10 和 Redis 4.0.0 起也可用, 以便提供可在副本中使用的只读命令。查看`GEORADIUS`页面以获取更多信息。

@examples

```cli
GEOADD Sicily 13.583333 37.316667 "Agrigento"
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEORADIUSBYMEMBER Sicily Agrigento 100 km
```
