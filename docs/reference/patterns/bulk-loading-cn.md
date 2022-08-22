批量加载是加载包含大量预先存在数据的 Redis 的过程。理想情况下, 您希望快速有效地执行此操作。本文档介绍在 Redis 中批量加载数据的一些策略。

## 使用 Redis 协议进行批量加载

使用普通的 Redis 客户端执行批量加载不是一个好主意
有几个原因：发送一个接一个命令的幼稚方法
很慢, 因为您必须为每个命令的往返时间付费。
可以使用流水线, 但用于批量加载许多记录
您需要在阅读回复的同时编写新命令
确保尽可能快地插入。

只有一小部分客户端支持非阻塞 I/O, 而不是所有
客户端能够以有效的方式解析回复, 以便最大化
吞吐量。出于所有这些原因, 将数据批量导入到
Redis是生成一个包含Redis协议的文本文件, 以原始格式, 
以便调用插入所需数据所需的命令。

例如, 如果我需要生成一个有数十亿的大型数据集
格式为：'keyN -> ValueN' 我将创建一个包含
以下 Redis 协议格式的命令：

    SET Key0 Value0
    SET Key1 Value1
    ...
    SET KeyN ValueN

创建此文件后, 剩余的操作是将其提供给 Redis
尽可能快。在过去, 执行此操作的方法是使用
`netcat`使用以下命令：

    (cat data.txt; sleep 10) | nc localhost 6379 > /dev/null

然而, 这不是执行批量导入的非常可靠的方法, 因为netcat
不知道所有数据何时传输, 无法检查
错误。在 2.6 或更高版本的 Redis 中, `redis-cli`效用
支持名为**管道模式**设计是为了执行
批量加载。

使用管道模式, 要运行的命令如下所示：

    cat data.txt | redis-cli --pipe

这将产生类似于以下内容的输出：

    All data transferred. Waiting for the last reply...
    Last reply received from server.
    errors: 0, replies: 1000000

redis-cli 实用程序还将确保仅重定向收到的错误
从 Redis 实例到标准输出。

### 生成 Redis 协议

Redis协议的生成和解析非常简单, 并且
[记录在这里](/topics/protocol).但是为了生成协议
批量加载的目标您不需要了解每个细节
协议, 但只是每个命令都通过以下方式表示：

    *<args><cr><lf>
    $<len><cr><lf>
    <arg0><cr><lf>
    <arg1><cr><lf>
    ...
    <argN><cr><lf>

哪里`<cr>`表示“\r” (或 ASCII 字符 13) 和`<lf>`表示“\n” (或 ASCII 字符 10) 。

例如, 命令**SET 键值**由以下协议表示：

    *3<cr><lf>
    $3<cr><lf>
    SET<cr><lf>
    $3<cr><lf>
    key<cr><lf>
    $5<cr><lf>
    value<cr><lf>

或表示为带引号的字符串：

    "*3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n"

您需要为批量加载生成的文件仅由命令组成
以上述方式表示, 一个接一个。

以下 Ruby 函数生成有效的协议：

    def gen_redis_proto(*cmd)
        proto = ""
        proto << "*"+cmd.length.to_s+"\r\n"
        cmd.each{|arg|
            proto << "$"+arg.to_s.bytesize.to_s+"\r\n"
            proto << arg.to_s+"\r\n"
        }
        proto
    end

    puts gen_redis_proto("SET","mykey","Hello World!").inspect

使用上述函数, 可以轻松生成键值对
在上面的例子中, 使用此程序：

    (0...1000).each{|n|
        STDOUT.write(gen_redis_proto("SET","Key#{n}","Value#{n}"))
    }

我们可以直接在管道中运行程序到redis-cli, 以便执行我们的
第一个批量导入会话。

    $ ruby proto.rb | redis-cli --pipe
    All data transferred. Waiting for the last reply...
    Last reply received from server.
    errors: 0, replies: 1000

### 管道模式在引擎盖下的工作原理

在 redis-cli 的管道模式中需要的魔力是像 netcat 一样快
并且仍然能够了解服务器何时发送了最后一次回复
同时。

这是通过以下方式获得的：

*   redis-cli --pipe 尝试将数据尽可能快地发送到服务器。
*   同时, 它会在可用时读取数据, 尝试解析数据。
*   一旦没有更多的数据要从stdin读取, 它就会发送一个特殊的**回波**
    带有随机 20 字节字符串的命令：我们确定这是最新的命令
    已发送, 并且我们确信如果收到相同的回复, 则可以匹配回复检查
    20 字节作为批量回复。
*   发送此特殊的最终命令后, 将开始接收回复的代码
    以将回复与这 20 个字节进行匹配。当到达匹配的答复时, 它
    可以成功退出。

使用此技巧, 我们不需要解析发送到服务器的协议
为了了解我们要发送的命令数量, 但只是回复。

但是, 在解析回复时, 我们会对解析的所有回复进行计数器
以便最后我们能够告诉用户命令的数量
通过批量插入会话传输到服务器。
