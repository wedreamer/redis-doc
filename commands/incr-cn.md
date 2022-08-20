递增存储在`key`由一个。
如果该项不存在，则将其设置为`0`在执行操作之前。
如果键包含错误类型的值或包含
不能表示为整数的字符串。
此操作限制为 64 位有符号整数。

**注意**：这是一个字符串操作，因为 Redis 没有专用的
整数类型。
存储在键处的字符串被解释为 base-10**64 位签名
整数**以执行操作。

Redis 将整数存储在其整数表示形式中，因此对于字符串值
实际保存整数，存储字符串没有开销
整数的表示形式。

@return

@integer回复：值`key`增量之后

@examples

```cli
SET mykey "10"
INCR mykey
GET mykey
```

## 模式：计数器

计数器模式是您可以使用 Redis atomic 做的最明显的事情
增量操作。
这个想法是简单地发送一个`INCR`每次操作时对 Redis 执行命令
发生。
例如，在Web应用程序中，我们可能想知道有多少页面浏览量
用户做了一年中的每一天。

为此，Web应用程序可以在每次用户时简单地递增一个键。
执行页面视图，创建将用户 ID 与
表示当前日期的字符串。

这个简单的模式可以通过多种方式进行扩展：

*   可以使用`INCR`和`EXPIRE`一起在每个页面视图有
    一个计数器仅计算相隔小于
    指定的秒数。
*   客户端可以使用 GETSET 以原子方式获取当前计数器值
    并将其重置为零。
*   使用其他原子递增/递减命令，如`DECR`或`INCRBY`它
    可以处理可能变大或变小的值，具体取决于
    由用户执行的操作。
    例如，想象一下在线游戏中不同用户的得分。

## 模式：速率限制器

速率限制器模式是一个特殊的计数器，用于将速率限制在
可以执行的操作。
这种模式的经典物化涉及限制
可以对公共 API 执行的请求。

我们提供了此模式的两种实现，使用`INCR`，我们假设
要解决的问题是将API调用次数限制为最大值
*每个 IP 地址每秒 10 个请求*.

## 模式：速率限制器 1

此模式更简单、更直接的实现如下：

    FUNCTION LIMIT_API_CALL(ip)
    ts = CURRENT_UNIX_TIME()
    keyname = ip+":"+ts
    MULTI
        INCR(keyname)
        EXPIRE(keyname,10)
    EXEC
    current = RESPONSE_OF_INCR_WITHIN_MULTI
    IF current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        PERFORM_API_CALL()
    END

基本上，我们为每个IP，每个不同的秒都有一个计数器。
但是，此计数器始终递增，将过期时间设置为 10 秒，以便
当当前秒不同时，Redis 会自动删除它们
一。

注意使用的`MULTI`和`EXEC`为了确保我们都能
递增并在每次 API 调用时设置过期。

## 模式：速率限制器 2

替代实现使用单个计数器，但稍微复杂一些
在没有竞争条件的情况下正确对待它。
我们将检查不同的变体。

    FUNCTION LIMIT_API_CALL(ip):
    current = GET(ip)
    IF current != NULL AND current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        value = INCR(ip)
        IF value == 1 THEN
            EXPIRE(ip,1)
        END
        PERFORM_API_CALL()
    END

计数器的创建方式使其只能存活一秒钟，从
从当前第二个请求中执行的第一个请求开始。
如果在同一秒内有超过10个请求，计数器将到达
值大于 10，否则它将过期并从 0 重新开始。

**在上面的代码中存在一个争用条件**.
如果由于某种原因客户端执行`INCR`命令，但不执行
这`EXPIRE`密钥将被泄露，直到我们再次看到相同的IP地址。

这可以很容易地固定转动`INCR`可选`EXPIRE`进入路亚
使用`EVAL`命令（仅在 Redis 版本后可用）
2.6).

    local current
    current = redis.call("incr",KEYS[1])
    if current == 1 then
        redis.call("expire",KEYS[1],1)
    end

有一种不同的方法可以在不使用脚本的情况下解决此问题，方法是使用
Redis 列表而不是计数器。
实现更复杂，使用更高级的功能，但具有
记住当前执行
API 调用，根据应用程序的不同，这可能有用或不有用。

    FUNCTION LIMIT_API_CALL(ip)
    current = LLEN(ip)
    IF current > 10 THEN
        ERROR "too many requests per second"
    ELSE
        IF EXISTS(ip) == FALSE
            MULTI
                RPUSH(ip,ip)
                EXPIRE(ip,1)
            EXEC
        ELSE
            RPUSHX(ip,ip)
        END
        PERFORM_API_CALL()
    END

这`RPUSHX`命令仅在键已存在时才推送元素。

请注意，我们这里有一场比赛，但这不是问题：`EXISTS`可能返回
false，但在我们内部创建密钥之前，密钥可能由另一个客户端创建
这`MULTI`/`EXEC`块。
但是，在极少数情况下，这场比赛只会错过API调用，因此速率
限制仍将正常工作。
