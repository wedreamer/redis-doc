
Redis 在内置命令集中有一些阻塞命令。
最常用的是`BLPOP`（或对称`BRPOP`） 哪个块
等待元素到达列表中。

关于阻止命令的有趣事实是它们不会阻止
整个服务器，但只是调用它们的客户端。通常原因
块是我们期望发生一些外部事件：这可能是
Redis 数据结构中的一些变化，例如`BLPOP`案例，
在线程中发生长时间计算，以接收来自
网络，等等。

Redis模块也能够实现阻塞命令，
本文档显示了 API 的工作原理并描述了一些模式
可用于对阻塞命令进行建模。

注意：此 API 当前是*实验的*，因此它只能在以下情况下使用：
宏`REDISMODULE_EXPERIMENTAL_API`已定义。这是必需的，因为
这些调用仍未进入设计的最后阶段，因此可能会更改
将来，某些部分可能会被弃用，依此类推。

要使用模块的这一部分，请包括模块标头，如下所示：

    #define REDISMODULE_EXPERIMENTAL_API
    #include "redismodule.h"

## 阻止和恢复的工作原理。

*注意：您可能需要检查`helloblock.c`Redis 源代码树中的示例
内部`src/modules`目录，用于简单易懂的示例
关于如何应用阻塞 API。*

在 Redis 模块中，命令由回调函数实现，
在调用特定命令时由 Redis 核心调用
由用户执行。通常，回调终止其执行发送
有些人回复客户。请改用以下函数，
实现模块命令的函数可以请求客户端
被置于阻塞状态：

    RedisModuleBlockedClient *RedisModule_BlockClient(RedisModuleCtx *ctx, RedisModuleCmdFunc reply_callback, RedisModuleCmdFunc timeout_callback, void (*free_privdata)(void*), long long timeout_ms);

该函数返回一个`RedisModuleBlockedClient`对象，这是后来的
用于取消阻止客户端。参数具有以下特征
意义：

*   `ctx`是命令执行上下文，就像通常在 API 的其余部分中一样。
*   `reply_callback`是回调，具有相同的普通命令函数原型，当客户端解除阻止以向客户端返回回复时调用。
*   `timeout_callback`是回调，具有与客户端到达`ms`超时。
*   `free_privdata`是为释放私有数据而调用的回调。私有数据是指向用于取消阻止客户端的 API 之间传递的某些数据的指针，指向将回复发送到客户端的回调。我们将在本文档后面看到此机制的工作原理。
*   `ms`是超时（以毫秒为单位）。达到超时时，将调用超时回调，并自动中止客户端。

客户端被阻止后，可以使用以下 API 取消阻止它：

    int RedisModule_UnblockClient(RedisModuleBlockedClient *bc, void *privdata);

该函数将返回的被阻止的客户端对象作为参数
上一次调用`RedisModule_BlockClient()`，然后取消阻止客户端。
在客户端解除阻止之前，立即`reply_callback`功能
调用客户端被阻止时指定的：此函数将
有权访问`privdata`此处使用的指针。

重要说明：上述函数是线程安全的，可以从内部调用
一个线程执行一些工作，以便实现阻塞的命令
客户端。

这`privdata`数据将使用`free_privdata`
客户端解除阻塞时的回调。这很有用**自回复以来
可能永远不会调用回调**如果客户端超时或断开连接
从服务器，因此重要的是它由外部功能决定
负责在需要时释放通过的数据。

为了更好地理解API的工作原理，我们可以想象编写一个命令。
阻止客户端一秒钟，然后发送回复“Hello！”

注意：未实施 arity 检查和其他不重要的事情
int他的命令，为了举个简单的例子。

    int Example_RedisCommand(RedisModuleCtx *ctx, RedisModuleString **argv,
                             int argc)
    {
        RedisModuleBlockedClient *bc =
            RedisModule_BlockClient(ctx,reply_func,timeout_func,NULL,0);

        pthread_t tid;
        pthread_create(&tid,NULL,threadmain,bc);

        return REDISMODULE_OK;
    }

    void *threadmain(void *arg) {
        RedisModuleBlockedClient *bc = arg;

        sleep(1); /* Wait one second and unblock. */
        RedisModule_UnblockClient(bc,NULL);
    }

上述命令尽快阻止客户端，生成一个线程，该线程将
等待一秒钟，将取消阻止客户端。让我们检查回复和
超时回调，在我们的例子中非常相似，因为它们
只需使用不同的回复类型回复客户端即可。

    int reply_func(RedisModuleCtx *ctx, RedisModuleString **argv,
                   int argc)
    {
        return RedisModule_ReplyWithSimpleString(ctx,"Hello!");
    }

    int timeout_func(RedisModuleCtx *ctx, RedisModuleString **argv,
                   int argc)
    {
        return RedisModule_ReplyWithNull(ctx);
    }

回复回调只是将“Hello！”字符串发送到客户端。
这里重要的一点是，当
客户端已从线程中解除阻止。

超时命令返回`NULL`，因为它经常发生在实际
Redis 阻止命令超时。

## 取消阻止时传递回复数据

上面的例子很容易理解，但缺乏一个重要的
实际阻塞命令实现的实际方面：通常
回复功能将需要知道要回复客户的内容，
并且此信息通常在客户端未被阻止时提供。

我们可以修改上面的例子，以便线程生成一个
等待一秒钟后的随机数。您可以将其视为
实际上是某种扩展操作。那么这个随机数
可以传递给回复函数，以便我们将其返回到命令
访客。为了使它正常工作，我们修改函数如下：

    void *threadmain(void *arg) {
        RedisModuleBlockedClient *bc = arg;

        sleep(1); /* Wait one second and unblock. */

        long *mynumber = RedisModule_Alloc(sizeof(long));
        *mynumber = rand();
        RedisModule_UnblockClient(bc,mynumber);
    }

如您所见，现在解锁呼叫正在传递一些私人数据，
那就是`mynumber`指针，指向回复回调。为了
获取此私有数据，回复回调将使用以下
功能：

    void *RedisModule_GetBlockedClientPrivateData(RedisModuleCtx *ctx);

所以我们的回复回调是这样修改的：

    int reply_func(RedisModuleCtx *ctx, RedisModuleString **argv,
                   int argc)
    {
        long *mynumber = RedisModule_GetBlockedClientPrivateData(ctx);
        /* IMPORTANT: don't free mynumber here, but in the
         * free privdata callback. */
        return RedisModule_ReplyWithLongLong(ctx,mynumber);
    }

请注意，我们还需要传递一个`free_privdata`阻塞时的功能
客户端与`RedisModule_BlockClient()`，因为已分配
必须释放长整型值。我们的回调将如下所示：

    void free_privdata(void *privdata) {
        RedisModule_Free(privdata);
    }

注意：重要的是要强调，私人数据最好在
`free_privdata`回调，因为可能无法调用回复函数
如果客户端断开连接或超时。

另请注意，私人数据也可以从超时访问
回调，始终使用`GetBlockedClientPrivateData()`应用程序接口。

## 中止对客户端的阻止

有时会出现的一个问题是，我们需要分配资源。
为了实现非阻塞命令。所以我们阻止客户端，
然后，例如，尝试创建一个线程，但线程创建函数
返回错误。在这种情况下该怎么办才能恢复？我们
不想让客户端被屏蔽，也不想打电话`UnblockClient()`
因为这将触发要调用的应答回调。

在这种情况下，最好的办法是使用以下函数：

    int RedisModule_AbortBlock(RedisModuleBlockedClient *bc);

实际上，这是如何使用它：

    int Example_RedisCommand(RedisModuleCtx *ctx, RedisModuleString **argv,
                             int argc)
    {
        RedisModuleBlockedClient *bc =
            RedisModule_BlockClient(ctx,reply_func,timeout_func,NULL,0);

        pthread_t tid;
        if (pthread_create(&tid,NULL,threadmain,bc) != 0) {
            RedisModule_AbortBlock(bc);
            RedisModule_ReplyWithError(ctx,"Sorry can't create a thread");
        }

        return REDISMODULE_OK;
    }

客户端将被解除阻止，但不会调用回复回调。

## 使用单个函数实现命令、回复和超时回调

可以使用以下函数来实现回复和
使用与实现主命令相同的函数进行回调
功能：

    int RedisModule_IsBlockedReplyRequest(RedisModuleCtx *ctx);
    int RedisModule_IsBlockedTimeoutRequest(RedisModuleCtx *ctx);

所以我可以重写示例命令，而无需使用单独的
回复和超时回调：

    int Example_RedisCommand(RedisModuleCtx *ctx, RedisModuleString **argv,
                             int argc)
    {
        if (RedisModule_IsBlockedReplyRequest(ctx)) {
            long *mynumber = RedisModule_GetBlockedClientPrivateData(ctx);
            return RedisModule_ReplyWithLongLong(ctx,mynumber);
        } else if (RedisModule_IsBlockedTimeoutRequest) {
            return RedisModule_ReplyWithNull(ctx);
        }

        RedisModuleBlockedClient *bc =
            RedisModule_BlockClient(ctx,reply_func,timeout_func,NULL,0);

        pthread_t tid;
        if (pthread_create(&tid,NULL,threadmain,bc) != 0) {
            RedisModule_AbortBlock(bc);
            RedisModule_ReplyWithError(ctx,"Sorry can't create a thread");
        }

        return REDISMODULE_OK;
    }

功能上是相同的，但有些人会更喜欢较少
详细实现，将大部分命令逻辑集中在
单个函数。

## 处理线程内的数据副本

一个有趣的模式，以便与实现
慢部分命令，就是用一个副本数据，这样
当在键中执行某些操作时，用户继续看到
旧版本。但是，当线程终止其工作时，
交换表示形式，并使用新的已处理版本。

这种方法的一个例子是
[神经 Redis 模块](https://github.com/antirez/neural-redis)
其中神经网络在不同的线程中训练，而
用户仍然可以执行和检查其旧版本。

## 三. 今后的工作

API目前正在进行中，以便允许Redis模块API
以安全的方式从线程调用，以便线程命令
可以访问数据空间并执行增量操作。

此功能没有ETA，但它可能会出现在
Redis 4.0 在某个时候发布。
