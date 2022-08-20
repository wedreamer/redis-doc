设置`key`以握住字符串`value`.
如果`key`已经持有一个值，它被覆盖，无论其类型如何。
与密钥关联的任何先前生存时间在成功时将被丢弃`SET`操作。

## 选项

这`SET`命令支持一组修改其行为的选项：

*   `EX` *秒*-- 设置指定的过期时间（以秒为单位）。
*   `PX` *毫秒*-- 设置指定的过期时间，以毫秒为单位。
*   `EXAT` *时间戳秒*-- 设置密钥过期的指定 Unix 时间（以秒为单位）。
*   `PXAT` *时间戳-毫秒*-- 设置密钥过期的指定 Unix 时间（以毫秒为单位）。
*   `NX`-- 仅当密钥尚不存在时才设置该密钥。
*   `XX`-- 仅当密钥已存在时才设置该密钥。
*   `KEEPTTL`-- 保留与密钥关联的生存时间。
*   `!GET`-- 返回存储在 key 处的旧字符串，如果 key 不存在，则返回 nil。返回错误，并且`SET`如果存储在键处的值不是字符串，则中止。

注意：由于`SET`命令选项可以替换`SETNX`,`SETEX`,`PSETEX`,`GETSET`，则可能会在 Redis 的未来版本中弃用并最终删除这些命令。

@return

@simple字符串回复：`OK`如果`SET`已正确执行。

@nil回复：`(nil)`如果`SET`未执行操作，因为用户指定了`NX`或`XX`选项，但未满足条件。

如果命令是随`!GET`选项，上述选项不适用。相反，它将按如下方式回复，无论`SET`实际执行：

@bulk字符串回复：存储在键处的旧字符串值。

@nil回复：`(nil)`如果密钥不存在。

@examples

```cli
SET mykey "Hello"
GET mykey

SET anotherkey "will expire in a minute" EX 60
```

## 模式

**注意：**不鼓励以下模式，以支持[红锁算法](https://redis.io/topics/distlock)它的实现稍微复杂一些，但提供了更好的保证并且是容错的。

命令`SET resource-name anystring NX EX max-lock-time`是使用 Redis 实现锁定系统的简单方法。

如果上述命令返回，客户端可以获取锁`OK`（如果命令返回 Nil，则在一段时间后重试），然后使用以下命令删除锁`DEL`.

达到过期时间后，锁将自动释放。

可以使此系统更可靠地修改解锁架构，如下所示：

*   不要设置固定字符串，而是设置一个不可猜测的大随机字符串，称为 token。
*   而不是释放锁`DEL`，发送一个脚本，该脚本仅在值匹配时才删除键。

这样可以避免客户端在过期时间后尝试释放锁，删除稍后获取锁的另一个客户端创建的密钥。

解锁脚本的示例类似于以下内容：

    if redis.call("get",KEYS[1]) == ARGV[1]
    then
        return redis.call("del",KEYS[1])
    else
        return 0
    end

应使用以下命令调用脚本`EVAL ...script... 1 resource-name token-value`
