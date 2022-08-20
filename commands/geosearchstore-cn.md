此命令类似于`GEOSEARCH`，但将结果存储在目标键中。

此命令将替换现已弃用的`GEORADIUS`和`GEORADIUSBYMEMBER`.

默认情况下，它将结果存储在`destination`已使用其地理空间信息排序集。

使用`STOREDIST`选项中，该命令将项目存储在已排序的集中，该集合填充了它们与圆或框中心的距离，作为浮点数，以为该形状指定的相同单位。

@return

@integer回复：结果集中的元素数。

@examples

```cli
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEOADD Sicily 12.758489 38.788135 "edge1"   17.241510 38.788135 "edge2" 
GEOSEARCHSTORE key1 Sicily FROMLONLAT 15 37 BYBOX 400 400 km ASC COUNT 3
GEOSEARCH key1 FROMLONLAT 15 37 BYBOX 400 400 km ASC WITHCOORD WITHDIST WITHHASH
GEOSEARCHSTORE key2 Sicily FROMLONLAT 15 37 BYBOX 400 400 km ASC COUNT 3 STOREDIST
ZRANGE key2 0 -1 WITHSCORES
```
