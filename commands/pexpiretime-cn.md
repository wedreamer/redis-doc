`PEXPIRETIME`具有与 相同的语义`EXPIRETIME`, 但返回绝对 Unix 过期时间戳 (以毫秒而不是秒为单位) 。

@return

@integer回复：过期 Unix 时间戳 (以毫秒为单), ) , 或负值以发出错误信号 (请参阅下面的) 明) 。

*   命令返回`-1`如果密钥存在但没有关联的过期时间。
*   命令返回`-2`如果密钥不存在。

@examples

```cli
SET mykey "Hello"
PEXPIREAT mykey 33177117420000
PEXPIRETIME mykey
```
