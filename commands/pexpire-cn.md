此命令的工作方式与`EXPIRE`但关键的生存时间是
以毫秒（而不是秒）为单位指定。

## 选项

这`PEXPIRE`命令支持自 Redis 7.0 以来的一组选项：

*   `NX`-- 仅当密钥没有过期时才设置过期时间
*   `XX`-- 仅当密钥具有现有到期时间时才设置到期时间
*   `GT`-- 仅当新到期时间大于当前到期时间时才设置到期时间
*   `LT`-- 仅当新的到期时间小于当前到期时间时才设置到期时间

非易失性密钥被视为无限 TTL，以便`GT`和`LT`.
这`GT`,`LT`和`NX`选项是互斥的。

@return

@integer回复，具体而言：

*   `1`如果设置了超时。
*   `0`如果未设置超时。例如.key不存在，或者由于提供的参数而跳过操作。

@examples

```cli
SET mykey "Hello"
PEXPIRE mykey 1500
TTL mykey
PTTL mykey
PEXPIRE mykey 1000 XX
TTL mykey
PEXPIRE mykey 1000 NX
TTL mykey
```
