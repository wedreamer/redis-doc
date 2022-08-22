`PEXPIREAT`具有与相同的效果和语义`EXPIREAT`, 但 Unix 时间在
密钥将过期的指定时间以毫秒为单位, 而不是以秒为单位。

## 选项

这`PEXPIREAT`命令支持自 Redis 7.0 以来的一组选项：

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
PEXPIREAT mykey 1555555555005
TTL mykey
PTTL mykey
```
