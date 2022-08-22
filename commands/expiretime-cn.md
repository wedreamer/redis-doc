返回绝对 Unix 时间戳 (自 1970 年 1 月 1 日起) , 以秒为单位, 给定密钥将在该时间戳处过期。

另请参见`PEXPIRETIME`命令, 该命令以毫秒分辨率返回相同的信息。

@return

@integer回复：以秒为单位使 Unix 时间戳过期, 或为负值以发出错误信号 (请参阅下面的说) ) 。

*   命令返回`-1`如果密钥存在但没有关联的过期时间。
*   命令返回`-2`如果密钥不存在。

@examples

```cli
SET mykey "Hello"
EXPIREAT mykey 33177117420
EXPIRETIME mykey
```
