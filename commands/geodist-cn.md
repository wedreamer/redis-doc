返回由排序集表示的地理空间索引中两个成员之间的距离。

给定一个表示地理空间索引的排序集，使用`GEOADD`命令，则该命令返回指定单位中两个指定成员之间的距离。

如果缺少一个或两个成员，则该命令返回 NULL。

单位必须是下列单位之一，并且默认为米：

*   **m**表示米。
*   **公里**公里。
*   **米**数英里。
*   **英尺**脚。

假设地球是一个完美的球体，则计算距离，因此在边缘情况下，误差可能高达0.5%。

@return

@bulk字符串回复，具体而言：

该命令以双精度形式返回距离（表示为字符串）
在指定单位中，如果缺少一个或两个元素，则为 NULL。

@examples

```cli
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEODIST Sicily Palermo Catania
GEODIST Sicily Palermo Catania km
GEODIST Sicily Palermo Catania mi
GEODIST Sicily Foo Bar
```
