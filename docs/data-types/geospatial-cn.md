---
title: "Redis geospatial"
linkTitle: "Geospatial"
weight: 80
description: >
    Introduction to the Redis Geospatial data type
---

Redis 地理空间索引允许您存储坐标并进行搜索。
此数据结构对于查找给定半径或边界框内的邻近点非常有用。

## 例子

假设您正在构建一个移动应用程序, 可让您找到离您当前位置最近的所有电动汽车充电站。

向地理空间索引添加多个位置：

    > GEOADD locations:ca -122.27652 37.805186 station:1
    (integer) 1
    > GEOADD locations:ca -122.2674626 37.8062344 station:2
    (integer) 1
    > GEOADD locations:ca -122.2469854 37.8104049 station:3
    (integer) 1

查找给定位置半径 1 公里范围内的所有位置, 然后将距离返回到每个位置：

    > GEOSEARCH locations:ca FROMLONLAT -122.2612767 37.7936847 BYRADIUS 5 km WITHDIST
    1) 1) "station:1"
       2) "1.8523"
    2) 1) "station:2"
       2) "1.4979"
    3) 1) "station:3"
       2) "2.2441"

## 基本命令

*   `GEOADD`将位置添加到给定的地理空间索引 (请注意, 使用此命令, 经度位于纬度之前) 。
*   `GEOSEARCH`返回具有给定半径或边界框的位置。

查看[地理空间索引命令的完整列表](https://redis.io/commands/?group=geo).

## 了解更多信息

*   [Redis Geosspaceal Explained](https://www.youtube.com/watch?v=qftiVQraxmI)通过向您展示如何构建当地公园景点的地图来介绍地理空间索引。
*   [Redis University's RU101](https://university.redis.com/courses/ru101/)详细介绍了 Redis 地理空间索引。
