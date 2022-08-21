
**注意：本文档由 Redis 的创建者 Salvatore Sanfilippo 在 Redis 开发早期（约 2010 年）编写，并不一定反映最新的 Redis 实现。**

## 为什么需要事件库？

让我们通过一系列的问答来解决这个问题。

问：您期望网络服务器始终处于什么状态？<br/>
答：监视其侦听端口上的入站连接并接受它们。

问：调用 \[接受]（http://man.cx/accept%282%29 接受）会产生一个描述符。我该怎么办？<br/>
答：保存描述符并对其执行非阻塞读/写操作。

问：为什么读/写必须是非阻塞的？<br/>
答：如果文件操作（甚至 Unix 中的套接字也是文件）被阻塞，例如，当服务器在文件 I/O 操作中被阻塞时，服务器如何接受其他连接请求。

问：我想我必须在套接字上做很多这样的非阻塞操作，看看它什么时候准备好了。我说的对吗？<br/>
答：是的。这就是事件库为您所做的。现在你明白了。

问：事件库如何执行它们所做的事情？<br/>
答：它们使用操作系统的轮询工具以及计时器。

问：那么是否有任何开源事件库可以执行您刚才描述的内容？<br/>
答：是的。`libevent`和`libev`是两个这样的事件库，我可以从头顶上回忆起。

问：Redis 是否使用此类开源事件库来处理套接字 I/O？<br/>
答：不可以。对于各种[原因](http://groups.google.com/group/redis-db/browse_thread/thread/b52814e9ef15b8d0/)Redis使用自己的事件库。

## Redis 事件库

Redis 实现了自己的事件库。事件库在`ae.c`.

了解 Redis 事件库如何工作的最佳方法是了解 Redis 如何使用它。

## 事件循环初始化

`initServer`中定义的函数`redis.c`初始化 的多个字段`redisServer`结构变量。一个这样的字段是 Redis 事件循环`el`:

    aeEventLoop *el

`initServer`初始 化`server.el`通过调用字段`aeCreateEventLoop`定义于`ae.c`.的定义`aeEventLoop`如下：

    typedef struct aeEventLoop
    {
        int maxfd;
        long long timeEventNextId;
        aeFileEvent events[AE_SETSIZE]; /* Registered events */
        aeFiredEvent fired[AE_SETSIZE]; /* Fired events */
        aeTimeEvent *timeEventHead;
        int stop;
        void *apidata; /* This is used for polling API specific data */
        aeBeforeSleepProc *beforesleep;
    } aeEventLoop;

## `aeCreateEventLoop`

`aeCreateEventLoop`第一`malloc`s`aeEventLoop`然后调用结构`ae_epoll.c:aeApiCreate`.

`aeApiCreate` `malloc`s`aeApiState`有两个字段 -`epfd`持有`epoll`由调用从 返回的文件描述符[`epoll_create`](http://man.cx/epoll_create%282%29)和`events`属于类型`struct epoll_event`由 Linux 定义`epoll`图书馆。用途`events`字段将在后面描述。

接下来是`ae.c:aeCreateTimeEvent`.但在此之前`initServer`叫`anet.c:anetTcpServer`创建并返回*侦听描述符*.描述符侦听*端口 6379*默认情况下。返回的 *侦听描述符*存储在`server.fd`田。

## `aeCreateTimeEvent`

`aeCreateTimeEvent`接受以下参数：

*   `eventLoop`：这是`server.el`在`redis.c`
*   毫秒数：从计时器过期的当前时间开始的毫秒数。
*   `proc`：函数指针。存储计时器过期后必须调用的函数的地址。
*   `clientData`： 大部分`NULL`.
*   `finalizerProc`：指向在从时标事件列表中删除定时事件之前必须调用的函数的指针。

`initServer`调用`aeCreateTimeEvent`将定时事件添加到`timeEventHead`领域`server.el`.`timeEventHead`是指向此类定时事件列表的指针。调用`aeCreateTimeEvent`从`redis.c:initServer`函数如下：

    aeCreateTimeEvent(server.el /*eventLoop*/, 1 /*milliseconds*/, serverCron /*proc*/, NULL /*clientData*/, NULL /*finalizerProc*/);

`redis.c:serverCron`执行许多操作，帮助保持 Redis 正常运行。

## `aeCreateFileEvent`

本质`aeCreateFileEvent`函数是执行[`epoll_ctl`](http://man.cx/epoll_ctl)系统调用，其中添加了监视`EPOLLIN`事件*侦听描述符*创建者`anetTcpServer`并将其与`epoll`由调用`aeCreateEventLoop`.

以下是对确切内容的解释`aeCreateFileEvent`从 调用时执行`redis.c:initServer`.

`initServer`将以下参数传递给`aeCreateFileEvent`:

*   `server.el`：事件循环由`aeCreateEventLoop`.这`epoll`描述符来自`server.el`.
*   `server.fd`：*侦听描述符*还用作从`eventLoop->events`表并存储额外的信息，如回调函数。
*   `AE_READABLE`：表示`server.fd`必须注意`EPOLLIN`事件。
*   `acceptHandler`：在被监视的事件准备就绪时必须执行的函数。此函数指针存储在`eventLoop->events[server.fd]->rfileProc`.

这样就完成了 Redis 事件循环的初始化。

## 事件循环处理

`ae.c:aeMain`调用自`redis.c:main`执行处理在上一阶段初始化的事件循环的工作。

`ae.c:aeMain`调用`ae.c:aeProcessEvents`在处理挂起时间和文件事件的 while 循环中。

## `aeProcessEvents`

`ae.c:aeProcessEvents`通过调用查找将在最短时间内挂起的时间事件`ae.c:aeSearchNearestTimer`在事件循环上。在我们的例子中，事件循环中只有一个计时器事件是由`ae.c:aeCreateTimeEvent`.

请记住，计时器事件由`aeCreateTimeEvent`现在可能已经过去了，因为它的到期时间为一毫秒。由于计时器已过期，则`tvp` `timeval`结构变量初始化为零。

这`tvp`结构变量以及事件循环变量被传递给`ae_epoll.c:aeApiPoll`.

`aeApiPoll`函数执行[`epoll_wait`](http://man.cx/epoll_wait)在`epoll`描述符并填充`eventLoop->fired`表的详细信息：

*   `fd`：现在已准备好根据掩码值执行读/写操作的描述符。
*   `mask`：现在可以对相应的描述符执行的读/写事件。

`aeApiPoll`返回准备操作的此类文件事件的数目。现在将事情放在上下文中，如果任何客户端请求连接，那么`aeApiPoll`会注意到它并填充`eventLoop->fired`表，其中描述符的条目为*侦听描述符*和面具`AE_READABLE`.

现在`aeProcessEvents`调用`redis.c:acceptHandler`注册为回调。`acceptHandler`执行[接受](http://man.cx/accept)在*侦听描述符*返回*连接描述符*与客户端。`redis.c:createClient`在 上添加文件事件*连接描述符*通过调用`ae.c:aeCreateFileEvent`如下图所示：

    if (aeCreateFileEvent(server.el, c->fd, AE_READABLE,
        readQueryFromClient, c) == AE_ERR) {
        freeClient(c);
        return NULL;
    }

`c`是`redisClient`结构变量和`c->fd`是连接的描述符。

下一页`ae.c:aeProcessEvent`调用`ae.c:processTimeEvents`

## `processTimeEvents`

`ae.processTimeEvents`循环访问从 开始的时间事件列表`eventLoop->timeEventHead`.

对于已过去的每个定时事件`processTimeEvents`调用已注册的回调。在这种情况下，它调用唯一注册的定时事件回调，即`redis.c:serverCron`.回调返回的时间（以毫秒为单位），在此时间之后必须再次调用回调。此更改通过调用`ae.c:aeAddMilliSeconds`并将在下一次迭代中处理`ae.c:aeMain`同时循环。

就这样。
