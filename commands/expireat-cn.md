`EXPIREAT`具有与相同的效果和语义`EXPIRE`, 但
指定表示 TTL (生存时间) 的, 数, 需要
绝对[Unix 时间戳][hewowu] (自 1970 年 1 月 1 日起的秒) ) 。一个
过去的时间戳将立即删除密钥。

[hewowu]: http://en.wikipedia.org/wiki/Unix_time

有关命令的具体语义, 请参阅
`EXPIRE`.

## 背景

`EXPIREAT`引入是为了将相对超时转换为绝对超时
AOF 持久性模式的超时。
当然, 它可以直接用于指定给定密钥应在
将来的给定时间。

## 选项

这`EXPIREAT`命令支持一组选项：

*   `NX`-- 仅当密钥没有过期时才设置过期时间
*   `XX`-- 仅当密钥具有现有到期时间时才设置到期时间
*   `GT`-- 仅当新到期时间大于当前到期时间时才设置到期时间
*   `LT`-- 仅当新的到期时间小于当前到期时间时才设置到期时间

非易失性密钥被视为无限 TTL, 以便`GT`和`LT`.
这`GT`,`LT`和`NX`选项是互斥的。

@return

@integer回复, 具体而言：

*   `1`如果设置了超时。
*   `0`如果未设置超时。例如.key不存在, 或者由于提供的参数而跳过操作。

@examples

```cli
SET mykey "Hello"
EXISTS mykey
EXPIREAT mykey 1293840000
EXISTS mykey
```
