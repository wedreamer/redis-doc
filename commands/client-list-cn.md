这`CLIENT LIST`命令返回有关客户端的信息和统计信息
连接服务器采用大部分人类可读的格式。

可以使用其中一个可选子命令来筛选列表。这`TYPE type`子命令按客户端的类型筛选列表, 其中*类型*是其中之一`normal`,`master`,`replica`和`pubsub`.请注意, 客户端被`MONITOR`命令属于`normal`类。

这`ID`filter 仅返回 ID 与`client-id`参数。

@return

@bulk字符串回复：一个唯一的字符串, 格式如下：

*   每行一个客户端连接 (以 LF 分隔) 
*   每条线由一系列`property=value`分隔的字段
    由空格字符。

以下是字段的含义：

*   `id`：唯一的 64 位客户端 ID
*   `addr`：客户端的地址/端口
*   `laddr`：本地地址客户端连接到的地址/端口 (绑定地) ) 
*   `fd`：与套接字对应的文件描述符
*   `name`：客户端设置的名称`CLIENT SETNAME`
*   `age`：连接的总持续时间 (以秒为单) ) 
*   `idle`：连接的空闲时间 (以秒为单) ) 
*   `flags`：客户端标志 (见下) ) 
*   `db`：当前数据库 ID
*   `sub`：频道订阅数
*   `psub`：模式匹配订阅的数量
*   `ssub`：分片通道订阅数。已在 Redis 7.0.3 中添加
*   `multi`：MULTI/EXEC 上下文中的命令数
*   `qbuf`：查询缓冲区长度 (0 表示没有挂起的查) ) 
*   `qbuf-free`：查询缓冲区的可用空间 (0 表示缓冲区已) ) 
*   `argv-mem`：下一个命令的不完整参数 (已从查询缓冲区中提) ) 
*   `multi-mem`：缓冲的多命令耗尽内存。已在 Redis 7.0 中添加
*   `obl`：输出缓冲器长度
*   `oll`：输出列表长度 (当缓冲区已满, , 回复在此列表中排) ) 
*   `omem`：输出缓冲区内存使用情况
*   `tot-mem`：此客户端在其各种缓冲区中消耗的总内存
*   `events`：文件描述符事件 (见下) ) 
*   `cmd`：上次播放的命令
*   `user`：客户端的已验证用户名
*   `redir`：当前客户端跟踪重定向的客户端 ID
*   `resp`：客户端 RESP 协议版本。已在 Redis 7.0 中添加

客户端标志可以是以下各项的组合：

    A: connection to be closed ASAP
    b: the client is waiting in a blocking operation
    c: connection to be closed after writing entire reply
    d: a watched keys has been modified - EXEC will fail
    i: the client is waiting for a VM I/O (deprecated)
    M: the client is a master
    N: no specific flag set
    O: the client is a client in MONITOR mode
    P: the client is a Pub/Sub subscriber
    r: the client is in readonly mode against a cluster node
    S: the client is a replica node connection to this instance
    u: the client is unblocked
    U: the client is connected via a Unix domain socket
    x: the client is in a MULTI/EXEC context
    t: the client enabled keys tracking in order to perform client side caching
    R: the client tracking target client is invalid
    B: the client enabled broadcast tracking mode 

文件描述符事件可以是：

    r: the client socket is readable (event loop)
    w: the client socket is writable (event loop)

## 笔记

出于调试目的, 会定期添加新字段。有些可以删除
在未来。使用此命令的版本安全 Redis 客户端应解析
相应地输出 (即处理优雅缺失的字, , 跳过
未知字段) 。
