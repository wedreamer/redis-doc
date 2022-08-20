获取的价值`key`并选择性地设置其过期时间。
`GETEX`类似于`GET`，但是一个包含其他选项的写入命令。

## 选项

这`GETEX`命令支持一组修改其行为的选项：

*   `EX` *秒*-- 设置指定的过期时间（以秒为单位）。
*   `PX` *毫秒*-- 设置指定的过期时间，以毫秒为单位。
*   `EXAT` *时间戳秒*-- 设置密钥过期的指定 Unix 时间（以秒为单位）。
*   `PXAT` *时间戳-毫秒*-- 设置密钥过期的指定 Unix 时间（以毫秒为单位）。
*   `PERSIST`-- 删除与密钥关联的生存时间。

@return

@bulk字符串回复：的值`key`或`nil`什么时候`key`不存在。

@examples

```cli
SET mykey "Hello"
GETEX mykey
TTL mykey
GETEX mykey EX 60
TTL mykey
```
