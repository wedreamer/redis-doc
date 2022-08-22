返回有效[地理哈希](https://en.wikipedia.org/wiki/Geohash)表示一个或多个元素在表示地理空间索引的排序集值中的位置的字符串 (其中元素是使用`GEOADD`).

通常, Redis 使用 Geohash 的变体来表示元素的位置
使用 52 位整数对位置进行编码的技术。编码为
与标准相比也不同, 因为初始最小值和最大值
在编码和解码过程中使用的坐标是不同的。这
命令但是**返回一个标准的 Geohash**以字符串的形式显示为
描述在[维基百科条目](https://en.wikipedia.org/wiki/Geohash)并且与[geohash.org](http://geohash.org)网站。

## 地理哈希字符串属性

该命令返回 11 个字符的 Geohash 字符串, 因此不会丢失任何精度
与 Redis 内部 52 位表示形式相比。返回的Geohashes
具有以下属性：

1.  它们可以缩短, 从右侧删除字符。它将失去精度, 但仍将指向同一区域。
2.  可以在以下情况下使用它们`geohash.org`网址, 例如`http://geohash.org/<geohash-string>`.这是一个[此类网址的示例](http://geohash.org/sqdtr74hyu0).
3.  具有类似前缀的字符串在附近, 但相反的情况并非如此, 具有不同前缀的字符串也可能在附近。

@return

@array回复, 具体而言：

该命令返回一个数组, 其中每个元素都是 Geohash 对应
作为参数传递给命令的每个成员名称。

@examples

```cli
GEOADD Sicily 13.361389 38.115556 "Palermo" 15.087269 37.502669 "Catania"
GEOHASH Sicily Palermo Catania
```
