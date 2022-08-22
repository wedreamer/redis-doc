<!-- This file is generated from module.c using
     utils/generate-module-api-doc.rb -->

## 部分

*   [堆分配原始函数](#section-heap-allocation-raw-functions)
*   [命令接口](#section-commands-api)
*   [模块信息和时间测量](#section-module-information-and-time-measurement)
*   [模块的自动内存管理](#section-automatic-memory-management-for-modules)
*   [字符串对象 API](#section-string-objects-apis)
*   [回复接口](#section-reply-apis)
*   [命令复制 API](#section-commands-replication-api)
*   [数据库和密钥 API – 通用 API](#section-db-and-key-apis-generic-api)
*   [字符串类型的密钥 API](#section-key-api-for-string-type)
*   [列表类型的关键 API](#section-key-api-for-list-type)
*   [排序集类型的密钥 API](#section-key-api-for-sorted-set-type)
*   [排序集迭代器的密钥 API](#section-key-api-for-sorted-set-iterator)
*   [哈希类型的密钥 API](#section-key-api-for-hash-type)
*   [流类型的密钥 API](#section-key-api-for-stream-type)
*   [从模块调用 Redis 命令](#section-calling-redis-commands-from-modules)
*   [模块数据类型](#section-modules-data-types)
*   [RDB 加载和保存功能](#section-rdb-loading-and-saving-functions)
*   [密钥摘要 API (模块类型的调试摘要接口) ](#section-key-digest-api-debug-digest-interface-for-modules-types)
*   [模块数据类型的 AOF API](#section-aof-api-for-modules-data-types)
*   [IO 上下文处理](#section-io-context-handling)
*   [伐木](#section-logging)
*   [阻止模块中的客户端](#section-blocking-clients-from-modules)
*   [线程安全上下文](#section-thread-safe-contexts)
*   [模块键空间通知 API](#section-module-keyspace-notifications-api)
*   [模块集群 API](#section-modules-cluster-api)
*   [模块定时器 API](#section-modules-timers-api)
*   [Modules EventLoop API](#section-modules-eventloop-api)
*   [模块 ACL API](#section-modules-acl-api)
*   [模块字典 API](#section-modules-dictionary-api)
*   [模块信息字段](#section-modules-info-fields)
*   [模块实用程序 API](#section-modules-utility-apis)
*   [模块 API 导出/导入](#section-modules-api-exporting-importing)
*   [模块命令筛选器 API](#section-module-command-filter-api)
*   [扫描密钥空间和哈希](#section-scanning-keyspace-and-hashes)
*   [模块分叉 API](#section-module-fork-api)
*   [服务器挂钩实现](#section-server-hooks-implementation)
*   [模块配置接口](#section-module-configurations-api)
*   [密钥逐出 API](#section-key-eviction-api)
*   [其他接口](#section-miscellaneous-apis)
*   [碎片整理 API](#section-defrag-api)
*   [功能指标](#section-function-index)

<span id="section-heap-allocation-raw-functions"></span>

## 堆分配原始函数

通过 Redis 键考虑使用这些函数分配的内存
逐出算法, 并在 Redis 内存使用情况信息中报告。

<span id="RedisModule_Alloc"></span>

### `RedisModule_Alloc`

    void *RedisModule_Alloc(size_t bytes);

**可用日期：**4.0.0

使用方式`malloc()`.使用此功能分配的内存在
Redis INFO 内存, 用于根据最大内存设置逐出密钥
并且通常被视为 Redis 分配的内存。
您应该避免使用`malloc()`.
如果无法分配足够的内存, 则此函数将死机。

<span id="RedisModule_TryAlloc"></span>

### `RedisModule_TryAlloc`

    void *RedisModule_TryAlloc(size_t bytes);

**可用日期：**7.0.0

似[`RedisModule_Alloc`](#RedisModule_Alloc), 但在分配失败时返回 NULL
惊慌失措。

<span id="RedisModule_Calloc"></span>

### `RedisModule_Calloc`

    void *RedisModule_Calloc(size_t nmemb, size_t size);

**可用日期：**4.0.0

使用方式`calloc()`.使用此功能分配的内存在
Redis INFO 内存, 用于根据最大内存设置逐出密钥
并且通常被视为 Redis 分配的内存。
您应该避免使用`calloc()`径直。

<span id="RedisModule_Realloc"></span>

### `RedisModule_Realloc`

    void* RedisModule_Realloc(void *ptr, size_t bytes);

**可用日期：**4.0.0

使用方式`realloc()`用于获取的内存[`RedisModule_Alloc()`](#RedisModule_Alloc).

<span id="RedisModule_Free"></span>

### `RedisModule_Free`

    void RedisModule_Free(void *ptr);

**可用日期：**4.0.0

使用方式`free()`用于由[`RedisModule_Alloc()`](#RedisModule_Alloc)和
[`RedisModule_Realloc()`](#RedisModule_Realloc).但是, 您永远不应该尝试使用
[`RedisModule_Free()`](#RedisModule_Free)分配的内存`malloc()`在模块中。

<span id="RedisModule_Strdup"></span>

### `RedisModule_Strdup`

    char *RedisModule_Strdup(const char *str);

**可用日期：**4.0.0

喜欢`strdup()`但返回分配的内存[`RedisModule_Alloc()`](#RedisModule_Alloc).

<span id="RedisModule_PoolAlloc"></span>

### `RedisModule_PoolAlloc`

    void *RedisModule_PoolAlloc(RedisModuleCtx *ctx, size_t bytes);

**可用日期：**4.0.0

返回堆分配的内存, 该内存将在
模块回调函数返回。主要适用于小额分配
寿命短, 必须在回调返回时释放
无论如何。返回的内存与体系结构字大小对齐
如果至少请求字大小字节, 否则它只是
与下一个 2 的幂对齐, 因此例如, 3 字节的请求是
4 个字节对齐, 而 2 个字节的请求对齐 2 个字节。

没有 realloc 样式函数, 因为当需要它时, 可以使用
池分配器不是一个好主意。

该函数在以下情况下返回 NULL：`bytes`为 0。

<span id="section-commands-api"></span>

## 命令接口

这些函数用于实现自定义 Redis 命令。

有关示例, 请参阅<https://redis.io/topics/modules-intro>.

<span id="RedisModule_IsKeysPositionRequest"></span>

### `RedisModule_IsKeysPositionRequest`

    int RedisModule_IsKeysPositionRequest(RedisModuleCtx *ctx);

**可用日期：**4.0.0

如果模块命令是用
标志“getkeys-api”, 以特殊方式调用以获取密钥位置
而不是被执行。否则返回零。

<span id="RedisModule_KeyAtPosWithFlags"></span>

### `RedisModule_KeyAtPosWithFlags`

    void RedisModule_KeyAtPosWithFlags(RedisModuleCtx *ctx, int pos, int flags);

**可用日期：**7.0.0

当调用模块命令以获取
键, 因为它在注册期间被标记为“getkeys-api”, 
命令实现使用
[`RedisModule_IsKeysPositionRequest()`](#RedisModule_IsKeysPositionRequest)API 并在
以报告键。

支持的标志是[`RedisModule_SetCommandInfo`](#RedisModule_SetCommandInfo)看`REDISMODULE_CMD_KEY_`\*.

以下是如何使用它的示例：

    if (RedisModule_IsKeysPositionRequest(ctx)) {
        RedisModule_KeyAtPosWithFlags(ctx, 2, REDISMODULE_CMD_KEY_RO | REDISMODULE_CMD_KEY_ACCESS);
        RedisModule_KeyAtPosWithFlags(ctx, 1, REDISMODULE_CMD_KEY_RW | REDISMODULE_CMD_KEY_UPDATE | REDISMODULE_CMD_KEY_ACCESS);
    }

注意：在上面的示例中, 获取密钥 API 可能已由密钥规范 (首选) 处理。
仅当无法声明涵盖所有键的键规范时, 才需要实现 getkeys-api。

<span id="RedisModule_KeyAtPos"></span>

### `RedisModule_KeyAtPos`

    void RedisModule_KeyAtPos(RedisModuleCtx *ctx, int pos);

**可用日期：**4.0.0

此 API 以前存在[`RedisModule_KeyAtPosWithFlags`](#RedisModule_KeyAtPosWithFlags)已添加, 现已弃用, 并且
可用于与旧版本兼容, 在密钥规范和标志之前
被引入。

<span id="RedisModule_IsChannelsPositionRequest"></span>

### `RedisModule_IsChannelsPositionRequest`

    int RedisModule_IsChannelsPositionRequest(RedisModuleCtx *ctx);

**可用日期：**7.0.0

如果模块命令是用
标志“getchannels-api”, 以特殊方式调用以获取通道位置
而不是被执行。否则返回零。

<span id="RedisModule_ChannelAtPosWithFlags"></span>

### `RedisModule_ChannelAtPosWithFlags`

    void RedisModule_ChannelAtPosWithFlags(RedisModuleCtx *ctx,
                                           int pos,
                                           int flags);

**可用日期：**7.0.0

当调用模块命令以获取
通道, 因为它在
注册时, 命令实现检查此特殊调用
使用[`RedisModule_IsChannelsPositionRequest()`](#RedisModule_IsChannelsPositionRequest)API 和使用此
函数以报告通道。

支持的标志包括：

*   `REDISMODULE_CMD_CHANNEL_SUBSCRIBE`：此命令将订阅频道。
*   `REDISMODULE_CMD_CHANNEL_UNSUBSCRIBE`：此命令将取消订阅此频道。
*   `REDISMODULE_CMD_CHANNEL_PUBLISH`：此命令将发布到此频道。
*   `REDISMODULE_CMD_CHANNEL_PATTERN`：不是在特定频道上操作, 而是在任何频道上执行操作
    由模式指定的通道。这是相同的访问权限
    由 PSUBSCRIBE 和 PUNSUBSCRIBE 命令使用
    在雷迪斯。不适用于“发布”权限。

以下是如何使用它的示例：

    if (RedisModule_IsChannelsPositionRequest(ctx)) {
        RedisModule_ChannelAtPosWithFlags(ctx, 1, REDISMODULE_CMD_CHANNEL_SUBSCRIBE | REDISMODULE_CMD_CHANNEL_PATTERN);
        RedisModule_ChannelAtPosWithFlags(ctx, 1, REDISMODULE_CMD_CHANNEL_PUBLISH);
    }

注意：声明通道的一种用法是评估 ACL 权限。在这方面, 
始终允许取消订阅, 因此只会根据订阅和
发布权限。这优先于使用[`RedisModule_ACLCheckChannelPermissions`](#RedisModule_ACLCheckChannelPermissions)因为
它允许在执行命令之前检查 ACL。

<span id="RedisModule_CreateCommand"></span>

### `RedisModule_CreateCommand`

    int RedisModule_CreateCommand(RedisModuleCtx *ctx,
                                  const char *name,
                                  RedisModuleCmdFunc cmdfunc,
                                  const char *strflags,
                                  int firstkey,
                                  int lastkey,
                                  int keystep);

**可用日期：**4.0.0

在 Redis 服务器中注册一个新命令, 该命令将由
使用 RedisModule 调用函数指针 'cmdfunc'
公约。函数返回`REDISMODULE_ERR`如果指定的命令
name 已忙或传递了一组无效标志, 否则
`REDISMODULE_OK`返回并注册新命令。

此函数必须在模块初始化期间调用
内部`RedisModule_OnLoad()`功能。在外部调用此函数
的初始化函数未定义。

命令函数类型如下：

     int MyCommand_RedisCommand(RedisModuleCtx *ctx, RedisModuleString **argv, int argc);

并且应该总是回来`REDISMODULE_OK`.

标志集“strflags”指定命令的行为, 并且应该
作为由空格分隔的单词组成的 C 字符串传递, 如
示例“写拒绝-oom”。标志集是：

*   **“写”**：该命令可以修改数据集 (也可以读取
    从它) 。
*   **“只读”**：该命令从键返回数据, 但从不写入。
*   **“管理员”**：该命令是管理命令 (可能会更改) 
    复制或执行类似任务) 。
*   **“deny-oom”**：该命令可能使用额外的内存, 并且应该是
    在内存不足的情况下被拒绝。
*   **“拒绝脚本”**：不允许在 Lua 脚本中使用此命令。
*   **“允许加载”**：在服务器加载数据时允许此命令。
    仅命令不与数据集交互
    应允许在此模式下运行。如果不确定
    不要使用此标志。
*   **“酒吧”**：该命令在发布/订阅频道上发布内容。
*   **“随机”**：即使启动, 命令也可能具有不同的输出
    从相同的输入参数和键值。
    从 Redis 7.0 开始, 此标志已被弃用。
    将命令声明为“随机”可以使用
    命令提示, 请参阅 https://redis.io/topics/command-tips。
*   **“允许过时”**：允许在不这样做的从属服务器上运行该命令
    提供陈旧数据。如果你不知道什么, 就不要使用
    这意味着。
*   **“无显示器”**：不要在监视器上传播命令。在以下情况下使用此参数
    该命令在参数之间具有敏感数据。
*   **“无慢日志”**：不要在慢日志中记录此命令。在以下情况下使用此参数
    该命令在参数之间具有敏感数据。
*   **“很快”**：命令时间复杂度不大
    比 O (log (N) )  中 N 是集合的大小或
    表示正常可伸缩性的任何其他内容
    命令问题。
*   **“getkeys-api”**：该命令实现要返回的接口
    作为键的参数。在启动/停止/步进时使用
    由于命令语法, 这还不够。
*   **“无群集”**：该命令不应在 Redis 集群中注册
    因为 不是为使用它而设计的, 因为, 
    例如, 无法报告位置
    键, 以编程方式创建键名, 或任何
    其他原因。
*   **“无身份验证”**：此命令可由未经身份验证的客户端运行。
    通常, 这由使用的命令使用
    以对客户端进行身份验证。
*   **“可能复制”**：此命令可能会生成复制流量, 甚至
    虽然它不是写入命令。
*   **“无强制键”**：此命令可能采用的所有键都是可选的
*   **“阻止”**：该命令有可能阻止客户端。
*   **“允许-忙碌”**：在服务器被阻止时允许该命令
    脚本或慢速模块命令, 请参见
    RedisModule_Yield。
*   **“getchannels-api”**：该命令实现要返回的接口
    作为通道的参数。

最后三个参数指定新命令的哪些参数是
Redis keys.看<https://redis.io/commands/command>了解更多信息。

*   `firstkey`：第一个参数 (键) 的基于一个索引。
    位置 0 始终是命令名称本身。
    0 表示没有键的命令。
*   `lastkey`：最后一个参数 (键) 的基于一个的索引。
    负数是指从最后一个向后计数
    参数  (-1 表示提供的最后一个参数) 
    0 表示没有键的命令。
*   `keystep`：在第一个键索引和最后一个键索引之间单步执行。
    0 表示没有键的命令。

此信息由 ACL、群集和`COMMAND`命令。

注意：上述方案的作用有限, 可以
仅用于查找存在于常量索引处的键。
对于非平凡的键参数, 您可以传递 0, 0, 0 并使用
[`RedisModule_SetCommandInfo`](#RedisModule_SetCommandInfo)以使用更高级的方案设置关键规格。

<span id="RedisModule_GetCommand"></span>

### `RedisModule_GetCommand`

    RedisModuleCommand *RedisModule_GetCommand(RedisModuleCtx *ctx,
                                               const char *name);

**可用日期：**7.0.0

按命令名称获取表示模块命令的不透明结构。
此结构用于某些与命令相关的 API。

如果出现以下错误, 则返回 NULL：

*   未找到命令
*   该命令不是模块命令
*   该命令不属于调用模块

<span id="RedisModule_CreateSubcommand"></span>

### `RedisModule_CreateSubcommand`

    int RedisModule_CreateSubcommand(RedisModuleCommand *parent,
                                     const char *name,
                                     RedisModuleCmdFunc cmdfunc,
                                     const char *strflags,
                                     int firstkey,
                                     int lastkey,
                                     int keystep);

**可用日期：**7.0.0

非常类似于[`RedisModule_CreateCommand`](#RedisModule_CreateCommand)除了它用于创建
一个子命令, 与另一个容器命令相关联。

示例：如果模块具有配置命令 MODULE.CONFIG, 则
GET 和 SET 应该是单独的子命令, 而 MODULE.CONFIG 是
命令, 但不应用于有效的`funcptr`:

     if (RedisModule_CreateCommand(ctx,"module.config",NULL,"",0,0,0) == REDISMODULE_ERR)
         return REDISMODULE_ERR;

     RedisModuleCommand *parent = RedisModule_GetCommand(ctx,,"module.config");

     if (RedisModule_CreateSubcommand(parent,"set",cmd_config_set,"",0,0,0) == REDISMODULE_ERR)
        return REDISMODULE_ERR;

     if (RedisModule_CreateSubcommand(parent,"get",cmd_config_get,"",0,0,0) == REDISMODULE_ERR)
        return REDISMODULE_ERR;

返回`REDISMODULE_OK`关于成功和`REDISMODULE_ERR`如果出现以下错误：

*   解析时出错`strflags`
*   命令标记为`no-cluster`但集群模式已启用
*   `parent`已经是一个子命令 (我们不允许多个级别的命令嵌套) 
*   `parent`是具有实现  (`RedisModuleCmdFunc`)  (父命令应该是子命令的纯容器) 
*   `parent`已具有名为`name`

<span id="RedisModule_SetCommandInfo"></span>

### `RedisModule_SetCommandInfo`

    int RedisModule_SetCommandInfo(RedisModuleCommand *command,
                                   const RedisModuleCommandInfo *info);

**可用日期：**7.0.0

设置其他命令信息。

影响`COMMAND`,`COMMAND INFO`和`COMMAND DOCS`簇
ACL, 用于过滤参数数错误的命令
调用到达模块代码。

此函数可以在创建命令后调用[`RedisModule_CreateCommand`](#RedisModule_CreateCommand)
并获取命令指针使用[`RedisModule_GetCommand`](#RedisModule_GetCommand).信息可以
每个命令只设置一次, 并具有以下结构：

    typedef struct RedisModuleCommandInfo {
        const RedisModuleCommandInfoVersion *version;
        const char *summary;
        const char *complexity;
        const char *since;
        RedisModuleCommandHistoryEntry *history;
        const char *tips;
        int arity;
        RedisModuleCommandKeySpec *key_specs;
        RedisModuleCommandArg *args;
    } RedisModuleCommandInfo;

除`version`是可选的。字段说明：

*   `version`：此字段支持与不同 Redis 版本的兼容性。
    始终将此字段设置为`REDISMODULE_COMMAND_INFO_VERSION`.

*   `summary`：命令的简短说明 (可选) 。

*   `complexity`：复杂性描述 (可选) 。

*   `since`：引入命令的版本 (可选) 。
    注意：指定的版本应该是模块的版本, 而不是 Redis 版本。

*   `history`：数组`RedisModuleCommandHistoryEntry` (可选), , 即
    具有以下字段的结构：

          const char *since;
          const char *changes;

    `since`是一个版本字符串, 并且`changes`是描述
    变化。数组由一个归零的条目终止, 即一个条目
    两个字符串都设置为 NULL。

*   `tips`：有关此命令的一串空格分隔提示, 用于
    客户端和代理。看<https://redis.io/topics/command-tips>.

*   `arity`：参数数, 包括命令名称本身。积极因素
    number 指定参数的确切数量和负数
    指定最小数量的参数, 因此请使用 -N 表示 >= N。
    在将调用传递给模块之前验证调用, 因此这可以替换
    在模块命令实现内部进行 arity 检查。值为 0 (或
    省略 arity 字段) 等效于 -2 (如果命令具有子命令) 
    否则为 -1。

*   `key_specs`：数组`RedisModuleCommandKeySpec`, 由
    元素内存集为零。这是一个试图描述
    关键论点的立场比旧的更好[`RedisModule_CreateCommand`](#RedisModule_CreateCommand)参数
    `firstkey`,`lastkey`,`keystep`如果这三个不是, 则需要
    足以描述关键位置。有两个步骤来检索密钥
    位置：*开始搜索* (BS)  在哪个索引中应找到第一个, , 以及
    *查找键* (FK) 相对于BS的输, , 描述了我们如何
    将哪些参数是键。此外, 还有关键的特定标志。

    键规范导致 在 中给出的三元组 (第一键、最后一键、键步) 
    RedisModule_CreateCommand重新计算, 但提供仍然有用
    这三个参数RedisModule_CreateCommand, 以更好地支持旧的Redis
    RedisModule_SetCommandInfo不可用的版本。

    请注意, 密钥规范不能完全取代“getkeys-api” (请参阅
    RedisModule_CreateCommand、RedisModule_IsKeysPositionRequest和RedisModule_KeyAtPosWithFlags) 所以
    提供两个关键规格并实现
    getkeys-api.

    密钥规范具有以下结构：

          typedef struct RedisModuleCommandKeySpec {
              const char *notes;
              uint64_t flags;
              RedisModuleKeySpecBeginSearchType begin_search_type;
              union {
                  struct {
                      int pos;
                  } index;
                  struct {
                      const char *keyword;
                      int startfrom;
                  } keyword;
              } bs;
              RedisModuleKeySpecFindKeysType find_keys_type;
              union {
                  struct {
                      int lastkey;
                      int keystep;
                      int limit;
                  } range;
                  struct {
                      int keynumidx;
                      int firstkey;
                      int keystep;
                  } keynum;
              } fk;
          } RedisModuleCommandKeySpec;

    RedisModuleCommandKeySpec 字段的说明：

    *   `notes`：有关此关键规范的可选注释或说明。

    *   `flags`：按位或键规范标志, 如下所述。

    *   `begin_search_type`：这描述了如何发现第一个密钥。
        有两种方法可以确定第一个键：

        *   `REDISMODULE_KSPEC_BS_UNKNOWN`：没有办法分辨出
            键参数启动。
        *   `REDISMODULE_KSPEC_BS_INDEX`：键参数从常量索引开始。
        *   `REDISMODULE_KSPEC_BS_KEYWORD`：键参数在
            特定关键字。

    *   `bs`：这是一个工会, 其中`index`或`keyword`使用分支
        取决于`begin_search_type`田。

        *   `bs.index.pos`：我们从中开始搜索键的索引。
            (`REDISMODULE_KSPEC_BS_INDEX`仅此而已。

        *   `bs.keyword.keyword`：指示
            关键参数的开头。(`REDISMODULE_KSPEC_BS_KEYWORD`仅此而已。

        *   `bs.keyword.startfrom`：argv 中要从中开始的索引
            搜索。可以是负数, 这意味着从末尾开始搜索, 
            反之亦然。示例：-2 表示从
            倒数第二个论点。(`REDISMODULE_KSPEC_BS_KEYWORD`仅此而已。

    *   `find_keys_type`：在“开始搜索”之后, 这描述了哪个
        参数是键。这些策略是：

        *   `REDISMODULE_KSPEC_BS_UNKNOWN`：没有办法分辨出
            找到键参数。
        *   `REDISMODULE_KSPEC_FK_RANGE`：键在特定索引处结束 (或
            相对于最后一个参数) 。
        *   `REDISMODULE_KSPEC_FK_KEYNUM`：有一个参数包含
            键本身之前某个位置的键参数数。

        `find_keys_type`和`fk`如果此键规范描述
        只有一个键。

    *   `fk`：这是一个工会, 其中`range`或`keynum`使用分支
        取决于`find_keys_type`田。

        *   `fk.range` (对于`REDISMODULE_KSPEC_FK_RANGE`) ：具有
            以下字段：

            *   `lastkey`：相对于结果的最后一个键的索引
                开始搜索步骤。可以是负数, 在这种情况下, 它不是
                相对。-1 表示最后一个参数, -2 表示
                最后, 依此类推。

            *   `keystep`：找到一个参数后, 我们应该跳过多少个参数
                键, 为了找到下一个？

            *   `limit`： 如果`lastkey`是-1, 我们使用`limit`停止搜索
                通过一个因素。0 和 1 表示无限制。2 表示 1/2 的
                剩余的参数, 3 表示 1/3, 依此类推。

        *   `fk.keynum` (对于`REDISMODULE_KSPEC_FK_KEYNUM`) ：具有
            以下字段：

            *   `keynumidx`：包含
                键即将到来, 相对于开始搜索步骤的结果。

            *   `firstkey`：第一键相对于结果的索引
                开始搜索步骤。 (通常就在之后`keynumidx`在
                哪种情况应设置为`keynumidx + 1`.)

            *   `keystep`：找到一个参数后, 我们应该跳过多少个参数
                键, 为了找到下一个？

    密钥规范标志：

    前四个是指命令实际对*值或
    密钥的元数据*, 而不一定是用户数据或其影响
    它。每个密钥规范可能正好有一个。任何操作
    未明确删除、覆盖或只读将被标记为
    乌尔曼。

    *   `REDISMODULE_CMD_KEY_RO`：只读。读取键的值, 但
        不一定返回它。

    *   `REDISMODULE_CMD_KEY_RW`：读写。修改存储在
        键或其元数据的值。

    *   `REDISMODULE_CMD_KEY_OW`：覆盖。覆盖存储在
        键的值。

    *   `REDISMODULE_CMD_KEY_RM`：删除密钥。

    接下来的四个参考*密钥值内的用户数据*, 而不是
    元数据, 如 LRU、类型、基数。它指的是逻辑运算
    在用户的数据 (实际输入字符串或TTL) , , 是
    已使用/已返回/已复制/已更改。它不是指修改或
    返回元数据 (如类型、计数、数据存在) 。访问可以是
    与其中一个写入操作“插入”、“删除”或“更新”结合使用。任何
    写不是插入或删除将是更新。

    *   `REDISMODULE_CMD_KEY_ACCESS`：返回、复制或使用用户数据
        从键的值。

    *   `REDISMODULE_CMD_KEY_UPDATE`：将数据更新为值, 新值可能
        取决于旧值。

    *   `REDISMODULE_CMD_KEY_INSERT`：将数据添加到值, 没有机会
        修改或删除现有数据。

    *   `REDISMODULE_CMD_KEY_DELETE`：从 中显式删除某些内容
        键的值。

    其他标志：

    *   `REDISMODULE_CMD_KEY_NOT_KEY`：密钥实际上不是密钥, 而是
        应在集群模式下路由, 就好像它是密钥一样。

    *   `REDISMODULE_CMD_KEY_INCOMPLETE`：键规范可能不会指出全部
        它应该覆盖的键。

    *   `REDISMODULE_CMD_KEY_VARIABLE_FLAGS`：某些键可能具有不同
        标志取决于参数。

*   `args`：数组`RedisModuleCommandArg`, 由元素 memset 终止
    为零。`RedisModuleCommandArg`是具有所描述的字段的结构
    下面。

          typedef struct RedisModuleCommandArg {
              const char *name;
              RedisModuleCommandArgType type;
              int key_spec_index;
              const char *token;
              const char *summary;
              const char *since;
              int flags;
              struct RedisModuleCommandArg *subargs;
          } RedisModuleCommandArg;

    字段说明：

    *   `name`：参数的名称。

    *   `type`：参数的类型。有关详细信息, 请参阅下文。类型
        `REDISMODULE_ARG_TYPE_ONEOF`和`REDISMODULE_ARG_TYPE_BLOCK`需要
        具有子参数的参数, 即`subargs`.

    *   `key_spec_index`：如果`type`是`REDISMODULE_ARG_TYPE_KEY`你必须
        提供与此参数关联的键规范的索引。看
        `key_specs`以上。如果参数不是键, 则可以指定 -1。

    *   `token`：参数前面的标记 (可选) 。示例：
        论点`seconds`在`SET`有一个令牌`EX`.如果参数由
        仅标记 (例如`NX`在`SET`)  的类型应为
        `REDISMODULE_ARG_TYPE_PURE_TOKEN`和`value`应为空。

    *   `summary`：参数的简短说明 (可选) 。

    *   `since`：包含此参数的第一个版本 (可选) 。

    *   `flags`：按位或宏`REDISMODULE_CMD_ARG_*`.见下文。

    *   `value`：参数的显示值。此字符串应
        从 的输出创建命令语法时显示
        `COMMAND`.如果`token`不是 NULL, 它还应该显示。

    解释`RedisModuleCommandArgType`:

    *   `REDISMODULE_ARG_TYPE_STRING`：字符串参数。
    *   `REDISMODULE_ARG_TYPE_INTEGER`：整数参数。
    *   `REDISMODULE_ARG_TYPE_DOUBLE`：双精度浮点参数。
    *   `REDISMODULE_ARG_TYPE_KEY`：表示键名的字符串参数。
    *   `REDISMODULE_ARG_TYPE_PATTERN`：字符串, 但正则表达式模式。
    *   `REDISMODULE_ARG_TYPE_UNIX_TIME`：整数, 但 Unix 时间戳。
    *   `REDISMODULE_ARG_TYPE_PURE_TOKEN`：参数没有占位符。
        它只是一个没有值的令牌。示例：`KEEPTTL`的选项
        `SET`命令。
    *   `REDISMODULE_ARG_TYPE_ONEOF`：当用户只能选择其中之一时使用
        一些子参数。需要`subargs`.示例：`NX`和`XX`
        选项`SET`.
    *   `REDISMODULE_ARG_TYPE_BLOCK`：当想要组合在一起时使用
        几个子参数, 通常用于对它们应用某些内容, 例如
        使整个组成为“可选”。需要`subargs`.示例：
        `LIMIT offset count`参数`ZRANGE`.

    命令参数标志的说明：

    *   `REDISMODULE_CMD_ARG_OPTIONAL`：参数是可选的 (如 GET in
        SET 命令) 。
    *   `REDISMODULE_CMD_ARG_MULTIPLE`：参数可能会重复 (如
        密钥在 DEL 中) 。
    *   `REDISMODULE_CMD_ARG_MULTIPLE_TOKEN`：争论可能会重复, 
        它的令牌也是如此 (如`GET pattern`在排序中) 。

关于成功`REDISMODULE_OK`返回。出错时`REDISMODULE_ERR`返回
和`errno`如果提供了无效信息, 则设置为 EINVAL;如果提供的信息, 则设置为 EEXIST
已设置。如果信息无效, 则会记录一条警告, 说明
信息的哪一部分是无效的以及为什么。

<span id="section-module-information-and-time-measurement"></span>

## 模块信息和时间测量

<span id="RedisModule_IsModuleNameBusy"></span>

### `RedisModule_IsModuleNameBusy`

    int RedisModule_IsModuleNameBusy(const char *name);

**可用日期：**4.0.3

如果模块名称繁忙, 则返回非零值。
否则返回零。

<span id="RedisModule_Milliseconds"></span>

### `RedisModule_Milliseconds`

    long long RedisModule_Milliseconds(void);

**可用日期：**4.0.0

返回当前 UNIX 时间 (以毫秒为单位) 。

<span id="RedisModule_MonotonicMicroseconds"></span>

### `RedisModule_MonotonicMicroseconds`

    uint64_t RedisModule_MonotonicMicroseconds(void);

**可用日期：**7.0.0

相对于任意时间点的微秒返回计数器。

<span id="RedisModule_BlockedClientMeasureTimeStart"></span>

### `RedisModule_BlockedClientMeasureTimeStart`

    int RedisModule_BlockedClientMeasureTimeStart(RedisModuleBlockedClient *bc);

**可用日期：**6.2.0

标记将用作计算开始时间的时间点
经过的执行时间, 当[`RedisModule_BlockedClientMeasureTimeEnd()`](#RedisModule_BlockedClientMeasureTimeEnd)被调用。
在同一命令中, 您可以多次调用
[`RedisModule_BlockedClientMeasureTimeStart()`](#RedisModule_BlockedClientMeasureTimeStart)和[`RedisModule_BlockedClientMeasureTimeEnd()`](#RedisModule_BlockedClientMeasureTimeEnd)
将独立的时间间隔累积到后台持续时间。
此方法始终返回`REDISMODULE_OK`.

<span id="RedisModule_BlockedClientMeasureTimeEnd"></span>

### `RedisModule_BlockedClientMeasureTimeEnd`

    int RedisModule_BlockedClientMeasureTimeEnd(RedisModuleBlockedClient *bc);

**可用日期：**6.2.0

标记将用作结束时间的时间点
以计算已用的执行时间。
关于成功`REDISMODULE_OK`返回。
此方法仅返回`REDISMODULE_ERR`如果没有开始时间
先前定义的 ( 含义[`RedisModule_BlockedClientMeasureTimeStart`](#RedisModule_BlockedClientMeasureTimeStart)未调用 ) 。

<span id="RedisModule_Yield"></span>

### `RedisModule_Yield`

    void RedisModule_Yield(RedisModuleCtx *ctx, int flags, const char *busy_reply);

**可用日期：**7.0.0

此 API 允许模块让 Redis 处理后台任务, 以及一些
命令, 长时间阻塞执行模块命令。
该模块可以定期调用此 API。
这些标志是这些标志的一个掩码：

*   `REDISMODULE_YIELD_FLAG_NONE`：无特殊标志, 可以执行一些背景
    操作, 但不处理客户端命令。
*   `REDISMODULE_YIELD_FLAG_CLIENTS`：Redis 还可以处理客户端命令。

这`busy_reply`参数是可选的, 可用于控制详细
错误字符串之后`-BUSY`错误代码。

当`REDISMODULE_YIELD_FLAG_CLIENTS`已使用, Redis 将仅启动
在 由 定义的时间之后处理客户端命令
`busy-reply-threshold`配置, 在这种情况下, Redis将开始拒绝大多数
命令`-BUSY`错误, 但允许标有`allow-busy`
要执行的标志。
此 API 还可以在线程安全上下文中使用 (锁定时) 和期间
加载 (在`rdb_load`回调, 在这种情况下, 它将拒绝命令
\-LOADING 错误) 

<span id="RedisModule_SetModuleOptions"></span>

### `RedisModule_SetModuleOptions`

    void RedisModule_SetModuleOptions(RedisModuleCtx *ctx, int options);

**可用日期：**6.0.0

设置定义功能的标志或行为位标志。

`REDISMODULE_OPTIONS_HANDLE_IO_ERRORS`:
通常, 模块不需要为此烦恼, 因为该过程将只是
但是, 如果发生读取错误, 则终止, 设置此标志将允许
如果启用了 repl-diskless-load, 则重新加载以工作。
模块应使用[`RedisModule_IsIOError`](#RedisModule_IsIOError)读取后, 使用前
读取的数据, 如果出现错误, 则向上传播, 并且
能够释放部分填充的值及其所有分配。

`REDISMODULE_OPTION_NO_IMPLICIT_SIGNAL_MODIFIED`:
看[`RedisModule_SignalModifiedKey()`](#RedisModule_SignalModifiedKey).

`REDISMODULE_OPTIONS_HANDLE_REPL_ASYNC_LOAD`:
设置此标志表示模块对无盘异步复制的感知 (repl-diskless-load=swapdb) 
并且 redis 可以在复制期间提供读取服务, 而不是以 LOAD 状态阻止。

<span id="RedisModule_SignalModifiedKey"></span>

### `RedisModule_SignalModifiedKey`

    int RedisModule_SignalModifiedKey(RedisModuleCtx *ctx,
                                      RedisModuleString *keyname);

**可用日期：**6.0.0

从用户的角度修改密钥的信号 (即使 WATCH 无效
和客户端缓存) 。

当打开以进行写入的密钥关闭时, 将自动完成此操作, 除非
选项`REDISMODULE_OPTION_NO_IMPLICIT_SIGNAL_MODIFIED`已使用
[`RedisModule_SetModuleOptions()`](#RedisModule_SetModuleOptions).

<span id="section-automatic-memory-management-for-modules"></span>

## 模块的自动内存管理

<span id="RedisModule_AutoMemory"></span>

### `RedisModule_AutoMemory`

    void RedisModule_AutoMemory(RedisModuleCtx *ctx);

**可用日期：**4.0.0

启用自动内存管理。

该函数必须作为命令实现的第一个函数调用
想要使用自动内存。

启用后, 自动内存管理会跟踪并自动释放
命令返回后, 键、呼叫回复和 Redis 字符串对象。在大多数
在这种情况下, 这消除了调用以下函数的需要：

1.  [`RedisModule_CloseKey()`](#RedisModule_CloseKey)
2.  [`RedisModule_FreeCallReply()`](#RedisModule_FreeCallReply)
3.  [`RedisModule_FreeString()`](#RedisModule_FreeString)

这些功能仍然可以在启用自动内存管理的情况下使用, 
例如, 优化进行大量分配的循环。

<span id="section-string-objects-apis"></span>

## 字符串对象 API

<span id="RedisModule_CreateString"></span>

### `RedisModule_CreateString`

    RedisModuleString *RedisModule_CreateString(RedisModuleCtx *ctx,
                                                const char *ptr,
                                                size_t len);

**可用日期：**4.0.0

创建新的模块字符串对象。必须释放返回的字符串
跟[`RedisModule_FreeString()`](#RedisModule_FreeString), 除非启用了自动内存。

该字符串是通过复制`len`字节开始
在`ptr`.不保留对传递缓冲区的引用。

模块上下文'ctx'是可选的, 如果要创建, 则可能为 NULL
超出上下文范围的字符串。但是, 在这种情况下, 自动
内存管理将不可用, 并且字符串内存必须为
手动管理。

<span id="RedisModule_CreateStringPrintf"></span>

### `RedisModule_CreateStringPrintf`

    RedisModuleString *RedisModule_CreateStringPrintf(RedisModuleCtx *ctx,
                                                      const char *fmt,
                                                      ...);

**可用日期：**4.0.0

从 printf 格式和参数创建新的模块字符串对象。
返回的字符串必须释放[`RedisModule_FreeString()`](#RedisModule_FreeString)除非
启用自动内存。

该字符串是使用 sds 格式化程序函数创建的`sdscatvprintf()`.

如有必要, 传递的上下文“ctx”可能为 NULL, 请参阅
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_CreateStringFromLongLong"></span>

### `RedisModule_CreateStringFromLongLong`

    RedisModuleString *RedisModule_CreateStringFromLongLong(RedisModuleCtx *ctx,
                                                            long long ll);

**可用日期：**4.0.0

喜欢[`RedisModule_CreateString()`](#RedisModule_CreateString), 但创建一个从`long long`
整数, 而不是采用缓冲区及其长度。

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

如有必要, 传递的上下文“ctx”可能为 NULL, 请参阅
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_CreateStringFromULongLong"></span>

### `RedisModule_CreateStringFromULongLong`

    RedisModuleString *RedisModule_CreateStringFromULongLong(RedisModuleCtx *ctx,
                                                             unsigned long long ull);

**可用日期：**7.0.3

喜欢[`RedisModule_CreateString()`](#RedisModule_CreateString), 但创建一个从`unsigned long long`
整数, 而不是采用缓冲区及其长度。

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

如有必要, 传递的上下文“ctx”可能为 NULL, 请参阅
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_CreateStringFromDouble"></span>

### `RedisModule_CreateStringFromDouble`

    RedisModuleString *RedisModule_CreateStringFromDouble(RedisModuleCtx *ctx,
                                                          double d);

**可用日期：**6.0.0

喜欢[`RedisModule_CreateString()`](#RedisModule_CreateString), 但创建一个从双精度值开始的字符串
而不是采取缓冲区及其长度。

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

<span id="RedisModule_CreateStringFromLongDouble"></span>

### `RedisModule_CreateStringFromLongDouble`

    RedisModuleString *RedisModule_CreateStringFromLongDouble(RedisModuleCtx *ctx,
                                                              long double ld,
                                                              int humanfriendly);

**可用日期：**6.0.0

喜欢[`RedisModule_CreateString()`](#RedisModule_CreateString), 但创建一个从长字符串开始
双。

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

如有必要, 传递的上下文“ctx”可能为 NULL, 请参阅
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_CreateStringFromString"></span>

### `RedisModule_CreateStringFromString`

    RedisModuleString *RedisModule_CreateStringFromString(RedisModuleCtx *ctx,
                                                          const RedisModuleString *str);

**可用日期：**4.0.0

喜欢[`RedisModule_CreateString()`](#RedisModule_CreateString), 但创建一个从另一个字符串开始的字符串
`RedisModuleString`.

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

如有必要, 传递的上下文“ctx”可能为 NULL, 请参阅
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_CreateStringFromStreamID"></span>

### `RedisModule_CreateStringFromStreamID`

    RedisModuleString *RedisModule_CreateStringFromStreamID(RedisModuleCtx *ctx,
                                                            const RedisModuleStreamID *id);

**可用日期：**6.2.0

从流 ID 创建字符串。返回的字符串必须随
[`RedisModule_FreeString()`](#RedisModule_FreeString), 除非启用了自动内存。

传递的上下文`ctx`如有必要, 可能为空。查看
[`RedisModule_CreateString()`](#RedisModule_CreateString)文档以获取更多信息。

<span id="RedisModule_FreeString"></span>

### `RedisModule_FreeString`

    void RedisModule_FreeString(RedisModuleCtx *ctx, RedisModuleString *str);

**可用日期：**4.0.0

释放通过其中一个 Redis 模块 API 调用获得的模块字符串对象
返回新的字符串对象。

即使自动内存管理, 也可以调用此函数
已启用。在这种情况下, 字符串将尽快释放并删除
从字符串池到末尾释放。

如果字符串是使用 NULL 上下文 “ctx” 创建的, 则还可以
释放字符串时将 ctx 作为 NULL 传递 (但传递上下文不会
创建任何问题) 。使用上下文创建的字符串也应该通过
上下文, 因此, 如果您以后要脱离上下文释放字符串, 请确保
以使用 NULL 上下文创建它。

<span id="RedisModule_RetainString"></span>

### `RedisModule_RetainString`

    void RedisModule_RetainString(RedisModuleCtx *ctx, RedisModuleString *str);

**可用日期：**4.0.0

每次调用此函数, 都会使字符串“str”需要
额外调用[`RedisModule_FreeString()`](#RedisModule_FreeString)为了真正
释放字符串。请注意, 自动释放获得的字符串
使模块自动内存管理计数
[`RedisModule_FreeString()`](#RedisModule_FreeString)调用 (它只是自动执行) 。

通常, 您希望在同时调用此函数时
满足以下条件：

1.  您已启用自动内存管理。
2.  您想要创建字符串对象。
3.  您创建的那些字符串对象需要存在*后*回调
    函数 (例如命令实现) 创建它们返回。

通常, 您希望这样做是为了存储创建的字符串对象
进入您自己的数据结构, 例如在实现新数据时
类型。

请注意, 关闭内存管理后, 不需要
对 RetainString ()  的任何调, , 因为创建字符串将始终产生结果
在回调函数返回后存在的字符串中, 如果
不执行 FreeString ()  调用。

可以使用 NULL 上下文调用此函数。

当字符串要保留较长时间时, 这是很好的
练习也要打电话[`RedisModule_TrimStringAllocation()`](#RedisModule_TrimStringAllocation)为了
优化内存使用率。

引用来自其他线程的保留字符串的线程化模块*必须*
保留字符串后立即显式修剪分配。不做
因此可能会导致自动修剪, 这不是线程安全的。

<span id="RedisModule_HoldString"></span>

### `RedisModule_HoldString`

    RedisModuleString* RedisModule_HoldString(RedisModuleCtx *ctx,
                                              RedisModuleString *str);

**可用日期：**6.0.7

可以使用此功能代替[`RedisModule_RetainString()`](#RedisModule_RetainString).
两者之间的主要区别在于, 此功能将始终
成功, 而[`RedisModule_RetainString()`](#RedisModule_RetainString)可能由于
断言。

该函数返回一个指针`RedisModuleString`, 归所有
由调用方提供。它需要调用[`RedisModule_FreeString()`](#RedisModule_FreeString)免费
为上下文禁用自动内存管理时的字符串。
启用自动内存管理后, 您可以调用
[`RedisModule_FreeString()`](#RedisModule_FreeString)或者让自动化释放它。

此功能比[`RedisModule_CreateStringFromString()`](#RedisModule_CreateStringFromString)
因为只要有可能, 它就可以避免复制底层
`RedisModuleString`.使用此功能的缺点是
可能无法使用[`RedisModule_StringAppendBuffer()`](#RedisModule_StringAppendBuffer)在
返回`RedisModuleString`.

可以使用 NULL 上下文调用此函数。

当字符串要长时间保留时, 这是很好的
练习也要打电话[`RedisModule_TrimStringAllocation()`](#RedisModule_TrimStringAllocation)为了
优化内存使用率。

引用来自其他线程的已保存字符串的线程模块*必须*
在保留字符串后立即显式修剪分配。不做
因此可能会导致自动修剪, 这不是线程安全的。

<span id="RedisModule_StringPtrLen"></span>

### `RedisModule_StringPtrLen`

    const char *RedisModule_StringPtrLen(const RedisModuleString *str,
                                         size_t *len);

**可用日期：**4.0.0

给定一个字符串模块对象, 此函数返回字符串指针
和字符串的长度。返回的指针和长度应仅
用于只读访问, 从不修改。

<span id="RedisModule_StringToLongLong"></span>

### `RedisModule_StringToLongLong`

    int RedisModule_StringToLongLong(const RedisModuleString *str, long long *ll);

**可用日期：**4.0.0

将字符串转换为`long long`整数, 将其存储在`*ll`.
返回`REDISMODULE_OK`关于成功。如果字符串无法解析
作为有效的, 严格的`long long` (前后无空格), , `REDISMODULE_ERR`
返回。

<span id="RedisModule_StringToULongLong"></span>

### `RedisModule_StringToULongLong`

    int RedisModule_StringToULongLong(const RedisModuleString *str,
                                      unsigned long long *ull);

**可用日期：**7.0.3

将字符串转换为`unsigned long long`整数, 将其存储在`*ull`.
返回`REDISMODULE_OK`关于成功。如果字符串无法解析
作为有效的, 严格的`unsigned long long` (前后无空格), , `REDISMODULE_ERR`
返回。

<span id="RedisModule_StringToDouble"></span>

### `RedisModule_StringToDouble`

    int RedisModule_StringToDouble(const RedisModuleString *str, double *d);

**可用日期：**4.0.0

将字符串转换为双精度值, 将其存储在`*d`.
返回`REDISMODULE_OK`成功或`REDISMODULE_ERR`如果字符串是
不是双精度值的有效字符串表示形式。

<span id="RedisModule_StringToLongDouble"></span>

### `RedisModule_StringToLongDouble`

    int RedisModule_StringToLongDouble(const RedisModuleString *str,
                                       long double *ld);

**可用日期：**6.0.0

将字符串转换为长双精度值, 将其存储在`*ld`.
返回`REDISMODULE_OK`成功或`REDISMODULE_ERR`如果字符串是
不是双精度值的有效字符串表示形式。

<span id="RedisModule_StringToStreamID"></span>

### `RedisModule_StringToStreamID`

    int RedisModule_StringToStreamID(const RedisModuleString *str,
                                     RedisModuleStreamID *id);

**可用日期：**6.2.0

将字符串转换为流 ID, 并将其存储在`*id`.
返回`REDISMODULE_OK`成功与回报`REDISMODULE_ERR`如果字符串
不是流 ID 的有效字符串表示形式。特殊 ID“+”和
允许使用“-”。

<span id="RedisModule_StringCompare"></span>

### `RedisModule_StringCompare`

    int RedisModule_StringCompare(RedisModuleString *a, RedisModuleString *b);

**可用日期：**4.0.0

比较两个字符串对象, 分别返回 -1、0 或 1, 如果
a < b,  a == b,  a > b.字符串逐个字节比较为两个
二进制 blob 没有任何编码护理/排序规则尝试。

<span id="RedisModule_StringAppendBuffer"></span>

### `RedisModule_StringAppendBuffer`

    int RedisModule_StringAppendBuffer(RedisModuleCtx *ctx,
                                       RedisModuleString *str,
                                       const char *buf,
                                       size_t len);

**可用日期：**4.0.0

将指定的缓冲区追加到字符串“str”。该字符串必须是
由用户创建的字符串, 仅被引用一次, 否则
`REDISMODULE_ERR`返回, 并且不执行该操作。

<span id="RedisModule_TrimStringAllocation"></span>

### `RedisModule_TrimStringAllocation`

    void RedisModule_TrimStringAllocation(RedisModuleString *str);

**可用日期：**7.0.0

修剪可能为`RedisModuleString`.

有时`RedisModuleString`可能为
它比必需的, 通常用于构造的 argv 参数
从网络缓冲区。此函数通过重新分配来优化此类字符串
它们的内存, 这对于不是短暂但存在但又短的字符串很有用
保留期限较长。

此操作是*不是线程安全的*并且只应在以下情况下调用
不保证对字符串的并发访问。将其用于 argv
在字符串可能可用之前模块命令中的字符串
到其他线程通常是安全的。

目前, Redis 还可能在
返回模块命令。但是, 明确地执行此操作仍应
首选选项：

1.  Redis的未来版本可能会放弃自动修剪。
2.  当前实现的自动修剪是*不是线程安全的*.
    操作最近保留的字符串的后台线程可能最终
    在具有自动修剪的争用条件下, 这可能导致
    数据损坏。

<span id="section-reply-apis"></span>

## 回复接口

这些函数用于向客户端发送答复。

大多数函数始终返回`REDISMODULE_OK`所以你可以用它与
'return' 以便从命令实现返回：

    if (... some condition ...)
        return RedisModule_ReplyWithLongLong(ctx,mycount);

### 使用收集功能回复

启动集合回复后, 模块必须调用其他
`ReplyWith*`样式函数, 以便发出集合的元素。
集合类型包括：数组、映射、集和属性。

生成包含许多未知元素的集合时
在此之前, 可以使用特殊标志调用该函数
`REDISMODULE_POSTPONED_LEN`(`REDISMODULE_POSTPONED_ARRAY_LEN`在过去), , 
并且实际元素数可以在以后设置`RedisModule_ReplySet`\*长度 () 
调用 (如果有多个“打开”计数, 则将设置最新的“打开”计数) 。

<span id="RedisModule_WrongArity"></span>

### `RedisModule_WrongArity`

    int RedisModule_WrongArity(RedisModuleCtx *ctx);

**可用日期：**4.0.0

发送有关给定命令的参数数的错误, 
引用错误消息中的命令名称。返回`REDISMODULE_OK`.

例：

    if (argc != 3) return RedisModule_WrongArity(ctx);

<span id="RedisModule_ReplyWithLongLong"></span>

### `RedisModule_ReplyWithLongLong`

    int RedisModule_ReplyWithLongLong(RedisModuleCtx *ctx, long long ll);

**可用日期：**4.0.0

向客户端发送一个整数答复, 其中包含指定的`long long`价值。
该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithError"></span>

### `RedisModule_ReplyWithError`

    int RedisModule_ReplyWithError(RedisModuleCtx *ctx, const char *err);

**可用日期：**4.0.0

回复错误“错误”。

请注意, “错误”必须包含所有错误, 包括
初始错误代码。该函数仅提供初始“-”, 因此
例如, 用法是：

    RedisModule_ReplyWithError(ctx,"ERR Wrong Type");

而不仅仅是：

    RedisModule_ReplyWithError(ctx,"Wrong Type");

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithSimpleString"></span>

### `RedisModule_ReplyWithSimpleString`

    int RedisModule_ReplyWithSimpleString(RedisModuleCtx *ctx, const char *msg);

**可用日期：**4.0.0

使用简单字符串  (`+... \r\n`在 RESP 协议中) 。此回复
仅当发送具有 small 的非二进制字符串时才适用
开销, 如“确定”或类似的回复。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithArray"></span>

### `RedisModule_ReplyWithArray`

    int RedisModule_ReplyWithArray(RedisModuleCtx *ctx, long len);

**可用日期：**4.0.0

使用数组类型的“len”元素进行回复。

启动数组应答后, 模块必须使`len`呼叫其他
`ReplyWith*`样式函数, 以便发出数组的元素。
有关更多详细信息, 请参阅回复 API 部分。

用[`RedisModule_ReplySetArrayLength()`](#RedisModule_ReplySetArrayLength)以设置延迟长度。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithMap"></span>

### `RedisModule_ReplyWithMap`

    int RedisModule_ReplyWithMap(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

使用 RESP3 映射类型的“len”对进行回复。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

启动地图回复后, 模块必须使`len*2`呼叫其他
`ReplyWith*`样式函数, 以便发出地图的元素。
有关更多详细信息, 请参阅回复 API 部分。

如果连接的客户端正在使用 RESP2, 则回复将转换为平面
数组。

用[`RedisModule_ReplySetMapLength()`](#RedisModule_ReplySetMapLength)以设置延迟长度。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithSet"></span>

### `RedisModule_ReplyWithSet`

    int RedisModule_ReplyWithSet(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

使用 RESP3 集类型的“len”元素进行回复。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

启动集合回复后, 模块必须使`len`呼叫其他
`ReplyWith*`样式函数, 以便发出集合的元素。
有关更多详细信息, 请参阅回复 API 部分。

如果连接的客户端正在使用 RESP2, 则回复将转换为
数组类型。

用[`RedisModule_ReplySetSetLength()`](#RedisModule_ReplySetSetLength)以设置延迟长度。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithAttribute"></span>

### `RedisModule_ReplyWithAttribute`

    int RedisModule_ReplyWithAttribute(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

向答复添加属性 (元数据) 。应在添加
实际回复。看<https://github.com/antirez/RESP3/blob/master/spec.md>#attribute型

启动属性的回复后, 模块必须使`len*2`呼叫其他
`ReplyWith*`样式函数, 以便发出属性映射的元素。
有关更多详细信息, 请参阅回复 API 部分。

用[`RedisModule_ReplySetAttributeLength()`](#RedisModule_ReplySetAttributeLength)以设置延迟长度。

不受 RESP2 支持, 将返回`REDISMODULE_ERR`否则
该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithNullArray"></span>

### `RedisModule_ReplyWithNullArray`

    int RedisModule_ReplyWithNullArray(RedisModuleCtx *ctx);

**可用日期：**6.0.0

使用空数组回复客户端, 在 RESP3 中只需空, 
RESP2 中的空数组。

注意：在 RESP3 中, 空回复和 Null 回复之间没有区别
NullArray 回复, 因此为了防止歧义, 最好避免
使用此 API 并使用[`RedisModule_ReplyWithNull`](#RedisModule_ReplyWithNull)相反。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithEmptyArray"></span>

### `RedisModule_ReplyWithEmptyArray`

    int RedisModule_ReplyWithEmptyArray(RedisModuleCtx *ctx);

**可用日期：**6.0.0

使用空数组回复客户端。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplySetArrayLength"></span>

### `RedisModule_ReplySetArrayLength`

    void RedisModule_ReplySetArrayLength(RedisModuleCtx *ctx, long len);

**可用日期：**4.0.0

什么时候[`RedisModule_ReplyWithArray()`](#RedisModule_ReplyWithArray)与参数一起使用
`REDISMODULE_POSTPONED_LEN`, 因为我们事先不知道数字
我们将作为数组的元素输出的项目, 此函数
将注意设置数组长度。

由于可能有多个数组回复挂起, 未知
length, 此函数保证始终设置最新的数组长度
这是以推迟的方式创建的。

例如, 为了输出一个像\[1, \[10, 20, 30]]这样的数组, 我们
可以写：

     RedisModule_ReplyWithArray(ctx,REDISMODULE_POSTPONED_LEN);
     RedisModule_ReplyWithLongLong(ctx,1);
     RedisModule_ReplyWithArray(ctx,REDISMODULE_POSTPONED_LEN);
     RedisModule_ReplyWithLongLong(ctx,10);
     RedisModule_ReplyWithLongLong(ctx,20);
     RedisModule_ReplyWithLongLong(ctx,30);
     RedisModule_ReplySetArrayLength(ctx,3); // Set len of 10,20,30 array.
     RedisModule_ReplySetArrayLength(ctx,2); // Set len of top array

请注意, 在上面的示例中, 没有理由推迟数组
长度, 因为我们产生固定数量的元素, 但在实践中
代码可以使用迭代器或其他方法来创建输出, 因此
这不容易提前计算出元素的数量。

<span id="RedisModule_ReplySetMapLength"></span>

### `RedisModule_ReplySetMapLength`

    void RedisModule_ReplySetMapLength(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

非常类似于[`RedisModule_ReplySetArrayLength`](#RedisModule_ReplySetArrayLength)除了`len`应该
正好是一半的数量`ReplyWith*`在 中调用的函数
地图的上下文。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

<span id="RedisModule_ReplySetSetLength"></span>

### `RedisModule_ReplySetSetLength`

    void RedisModule_ReplySetSetLength(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

非常类似于[`RedisModule_ReplySetArrayLength`](#RedisModule_ReplySetArrayLength)
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

<span id="RedisModule_ReplySetAttributeLength"></span>

### `RedisModule_ReplySetAttributeLength`

    void RedisModule_ReplySetAttributeLength(RedisModuleCtx *ctx, long len);

**可用日期：**7.0.0

非常类似于[`RedisModule_ReplySetMapLength`](#RedisModule_ReplySetMapLength)
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

如果出现以下情况, 则不得调用[`RedisModule_ReplyWithAttribute`](#RedisModule_ReplyWithAttribute)返回错误。

<span id="RedisModule_ReplyWithStringBuffer"></span>

### `RedisModule_ReplyWithStringBuffer`

    int RedisModule_ReplyWithStringBuffer(RedisModuleCtx *ctx,
                                          const char *buf,
                                          size_t len);

**可用日期：**4.0.0

使用批量字符串回复, 并接受输入 C 缓冲区指针和长度。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithCString"></span>

### `RedisModule_ReplyWithCString`

    int RedisModule_ReplyWithCString(RedisModuleCtx *ctx, const char *buf);

**可用日期：**5.0.6

使用批量字符串回复, 并输入一个 C 缓冲区指针, 该指针
假定为空值终止。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithString"></span>

### `RedisModule_ReplyWithString`

    int RedisModule_ReplyWithString(RedisModuleCtx *ctx, RedisModuleString *str);

**可用日期：**4.0.0

使用批量字符串回复, 接受输入`RedisModuleString`对象。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithEmptyString"></span>

### `RedisModule_ReplyWithEmptyString`

    int RedisModule_ReplyWithEmptyString(RedisModuleCtx *ctx);

**可用日期：**6.0.0

使用空字符串进行答复。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithVerbatimStringType"></span>

### `RedisModule_ReplyWithVerbatimStringType`

    int RedisModule_ReplyWithVerbatimStringType(RedisModuleCtx *ctx,
                                                const char *buf,
                                                size_t len,
                                                const char *ext);

**可用日期：**7.0.0

使用不应转义或筛选的二进制安全字符串进行回复
输入C缓冲指针, 长度和3个字符的类型/扩展名。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithVerbatimString"></span>

### `RedisModule_ReplyWithVerbatimString`

    int RedisModule_ReplyWithVerbatimString(RedisModuleCtx *ctx,
                                            const char *buf,
                                            size_t len);

**可用日期：**6.0.0

使用不应转义或筛选的二进制安全字符串进行回复
输入C缓冲指针和长度。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithNull"></span>

### `RedisModule_ReplyWithNull`

    int RedisModule_ReplyWithNull(RedisModuleCtx *ctx);

**可用日期：**4.0.0

使用 NULL 回复客户端。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithBool"></span>

### `RedisModule_ReplyWithBool`

    int RedisModule_ReplyWithBool(RedisModuleCtx *ctx, int b);

**可用日期：**7.0.0

使用 RESP3 布尔类型进行答复。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

在 RESP3 中, 这是布尔类型
在 RESP2 中, 它是一个字符串响应, 分别为 true 和 false 的“1”和“0”。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithCallReply"></span>

### `RedisModule_ReplyWithCallReply`

    int RedisModule_ReplyWithCallReply(RedisModuleCtx *ctx,
                                       RedisModuleCallReply *reply);

**可用日期：**4.0.0

准确回复 Redis 命令返回给我们的内容[`RedisModule_Call()`](#RedisModule_Call).
此功能在我们使用时很有用[`RedisModule_Call()`](#RedisModule_Call)为了
执行一些命令, 因为我们希望准确地回复客户端
我们通过命令获得的相同回复。

返回：

*   `REDISMODULE_OK`关于成功。
*   `REDISMODULE_ERR`如果给定的回复是 RESP3 格式, 但客户端需要 RESP2。
    如果出现错误, 模块编写者负责翻译回复
    到 RESP2 (或通过返回错误以不同的方式处理它) 。请注, , 对于
    模块编写方便, 可以通过`0`作为 fmt 的参数
    参数[`RedisModule_Call`](#RedisModule_Call)以便`RedisModuleCallReply`将以相同的方式返回
    在当前客户端上下文中设置的协议 (RESP2 或 RESP3) 。

<span id="RedisModule_ReplyWithDouble"></span>

### `RedisModule_ReplyWithDouble`

    int RedisModule_ReplyWithDouble(RedisModuleCtx *ctx, double d);

**可用日期：**4.0.0

使用 RESP3 双精度类型进行回复。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

发送获取的字符串回复, 将双精度“d”转换为批量字符串。
这个函数基本上等价于将双精度转换为
将字符串放入 C 缓冲区, 然后调用该函数
[`RedisModule_ReplyWithStringBuffer()`](#RedisModule_ReplyWithStringBuffer)与缓冲区和长度。

在 RESP3 中, 字符串被标记为双精度, 而在 RESP2 中, 它只是一个普通字符串
用户将不得不解析。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithBigNumber"></span>

### `RedisModule_ReplyWithBigNumber`

    int RedisModule_ReplyWithBigNumber(RedisModuleCtx *ctx,
                                       const char *bignum,
                                       size_t len);

**可用日期：**7.0.0

使用 RESP3 BigNumber 类型进行回复。
访问<https://github.com/antirez/RESP3/blob/master/spec.md>有关 RESP3 的更多信息。

在 RESP3 中, 这是一个长度字符串`len`标记为 BigNumber, 
但是, 这取决于调用方以确保它是有效的BigNumber。
在 RESP2 中, 这只是一个普通的批量字符串响应。

该函数始终返回`REDISMODULE_OK`.

<span id="RedisModule_ReplyWithLongDouble"></span>

### `RedisModule_ReplyWithLongDouble`

    int RedisModule_ReplyWithLongDouble(RedisModuleCtx *ctx, long double ld);

**可用日期：**6.0.0

发送获得的字符串回复, 将长双精度“ld”转换为批量
字符串。这个函数基本上等同于转换长双精度
将字符串放入 C 缓冲区, 然后调用该函数
[`RedisModule_ReplyWithStringBuffer()`](#RedisModule_ReplyWithStringBuffer)与缓冲区和长度。
双字符串使用人类可读的格式 (请参阅
`addReplyHumanLongDouble`在 networking.c 中) 。

该函数始终返回`REDISMODULE_OK`.

<span id="section-commands-replication-api"></span>

## 命令复制 API

<span id="RedisModule_Replicate"></span>

### `RedisModule_Replicate`

    int RedisModule_Replicate(RedisModuleCtx *ctx,
                              const char *cmdname,
                              const char *fmt,
                              ...);

**可用日期：**4.0.0

将指定的命令和参数复制到从属服务器和 AOF, 作为效果
调用命令实现的执行。

复制的命令始终包装到 MULTI/EXEC 中, 
包含在给定模块命令中复制的所有命令
执行。但是, 命令复制[`RedisModule_Call()`](#RedisModule_Call)
是第一个项目, 复制的项目[`RedisModule_Replicate()`](#RedisModule_Replicate)
将在执行委员会之前全部跟进。

模块应尝试使用一个接口或另一个接口。

此命令遵循完全相同的接口[`RedisModule_Call()`](#RedisModule_Call),
因此必须传递一组格式说明符, 后跟参数
与提供的格式说明符匹配。

请参考[`RedisModule_Call()`](#RedisModule_Call)了解更多信息。

使用特殊的“A”和“R”修饰符, 调用方可以排除
AOF 或来自指定命令传播的副本。
否则, 默认情况下, 该命令将在两个通道中传播。

#### 有关从线程安全上下文调用此函数的注意事项：

通常, 当您从回调调用此函数时, 实现
模块命令, 或 Redis 模块 API 提供的任何其他回调, 
Redis 将在
, 并将传播包装在 MULTI/EXEC 中的所有命令
交易。但是, 从线程安全上下文调用此函数时
可以存活未定义的时间, 并且可以锁定/解锁
随意, 行为不同：不发出 MULTI/EXEC 包装器
, 并将指定的命令插入到 AOF 和复制流中
马上。

#### 返回值

命令返回`REDISMODULE_ERR`如果格式说明符无效
或命令名称不属于已知命令。

<span id="RedisModule_ReplicateVerbatim"></span>

### `RedisModule_ReplicateVerbatim`

    int RedisModule_ReplicateVerbatim(RedisModuleCtx *ctx);

**可用日期：**4.0.0

此函数将完全按照调用的方式复制命令
由客户。请注意, 此函数不会将命令包装到
一个 MULTI/EXEC 节, 因此不应将其与其他复制混合使用
命令。

基本上, 当您想要传播时, 这种形式的复制很有用
对从属服务器和AOF文件的命令与它被调用的完全相同, 因为
该命令可以重新执行以确定地重新创建
从旧状态开始的新状态。

该函数始终返回`REDISMODULE_OK`.

<span id="section-db-and-key-apis-generic-api"></span>

## 数据库和密钥 API – 通用 API

<span id="RedisModule_GetClientId"></span>

### `RedisModule_GetClientId`

    unsigned long long RedisModule_GetClientId(RedisModuleCtx *ctx);

**可用日期：**4.0.0

返回调用当前活动模块的当前客户端的 ID
命令。返回的 ID 具有以下一些保证：

1.  每个不同客户端的 ID 都不同, 因此如果同一客户端
    多次执行一个模块命令, 可以识别为
    具有相同的 ID, 否则 ID 将不同。
2.  ID 单调增加。稍后连接到服务器的客户端
    保证获得的 ID 大于以前看到的任何过去的 ID。

有效 ID 介于 1 到 2^64 - 1 之间。如果返回 0, 则表示没有办法
以在当前调用函数的上下文中获取 ID。

获取ID后, 可以检查命令是否执行
实际上发生在AOF加载的上下文中, 使用此宏：

     if (RedisModule_IsAOFClient(RedisModule_GetClientId(ctx)) {
         // Handle it differently.
     }

<span id="RedisModule_GetClientUserNameById"></span>

### `RedisModule_GetClientUserNameById`

    RedisModuleString *RedisModule_GetClientUserNameById(RedisModuleCtx *ctx,
                                                         uint64_t id);

**可用日期：**6.2.1

返回具有指定客户机 ID 的客户机使用的 ACL 用户名。
可以使用以下命令获取客户端 ID[`RedisModule_GetClientId()`](#RedisModule_GetClientId)应用程序接口。如果客户端不这样做
exist, 则返回 NULL 并将 errno 设置为 ENOENT。如果客户端不是
使用 ACL 用户, 返回 NULL 并将 errno 设置为 ENOTSUP

<span id="RedisModule_GetClientInfoById"></span>

### `RedisModule_GetClientInfoById`

    int RedisModule_GetClientInfoById(void *ci, uint64_t id);

**可用日期：**6.0.0

返回有关具有指定 ID 的客户端的信息 (即
以前通过[`RedisModule_GetClientId()`](#RedisModule_GetClientId)API) 。如果
客户端存在, `REDISMODULE_OK`返回, 否则`REDISMODULE_ERR`
返回。

当客户端存在并且`ci`指针不为 NULL, 但指向
类型的结构`RedisModuleClientInfoV`1、先前初始化用
正确`REDISMODULE_CLIENTINFO_INITIALIZER_V1`, 则填充结构
具有以下字段：

     uint64_t flags;         // REDISMODULE_CLIENTINFO_FLAG_*
     uint64_t id;            // Client ID
     char addr[46];          // IPv4 or IPv6 address.
     uint16_t port;          // TCP port.
     uint16_t db;            // Selected DB.

注意：客户端 ID 在此调用的上下文中是无用的, 因为我们
已经知道, 但是相同的结构可以用于其他
我们不知道客户端 ID 但结构相同的上下文
返回。

标志具有以下含义：

    REDISMODULE_CLIENTINFO_FLAG_SSL          Client using SSL connection.
    REDISMODULE_CLIENTINFO_FLAG_PUBSUB       Client in Pub/Sub mode.
    REDISMODULE_CLIENTINFO_FLAG_BLOCKED      Client blocked in command.
    REDISMODULE_CLIENTINFO_FLAG_TRACKING     Client with keys tracking on.
    REDISMODULE_CLIENTINFO_FLAG_UNIXSOCKET   Client using unix domain socket.
    REDISMODULE_CLIENTINFO_FLAG_MULTI        Client in MULTI state.

但是, 传递 NULL 是一种在紧急情况下检查客户端是否存在的方法
我们对任何其他信息不感兴趣。

当我们需要客户端信息结构时, 这是正确的用法
返回：

     RedisModuleClientInfo ci = REDISMODULE_CLIENTINFO_INITIALIZER;
     int retval = RedisModule_GetClientInfoById(&ci,client_id);
     if (retval == REDISMODULE_OK) {
         printf("Address: %s\n", ci.addr);
     }

<span id="RedisModule_GetClientNameById"></span>

### `RedisModule_GetClientNameById`

    RedisModuleString *RedisModule_GetClientNameById(RedisModuleCtx *ctx,
                                                     uint64_t id);

**可用日期：**7.0.3

返回具有给定 ID 的客户端连接的名称。

如果客户机 ID 不存在, 或者客户机没有与 关联的名称
它, 返回 NULL。

<span id="RedisModule_SetClientNameById"></span>

### `RedisModule_SetClientNameById`

    int RedisModule_SetClientNameById(uint64_t id, RedisModuleString *name);

**可用日期：**7.0.3

设置具有给定 ID 的客户端的名称。这等效于客户端调用
`CLIENT SETNAME name`.

返回`REDISMODULE_OK`关于成功。失败时, `REDISMODULE_ERR`返回
和 errno 的设置如下：

*   如果客户端不存在, 则为 ENOENT
*   EINVAL (如果名称包含无效字符) 

<span id="RedisModule_PublishMessage"></span>

### `RedisModule_PublishMessage`

    int RedisModule_PublishMessage(RedisModuleCtx *ctx,
                                   RedisModuleString *channel,
                                   RedisModuleString *message);

**可用日期：**6.0.0

向订阅者发布消息 (请参阅 PUBLISH 命令) 。

<span id="RedisModule_PublishMessageShard"></span>

### `RedisModule_PublishMessageShard`

    int RedisModule_PublishMessageShard(RedisModuleCtx *ctx,
                                        RedisModuleString *channel,
                                        RedisModuleString *message);

**可用日期：**7.0.0

将消息发布到分片订阅者 (请参阅 SPUBLISH 命令) 。

<span id="RedisModule_GetSelectedDb"></span>

### `RedisModule_GetSelectedDb`

    int RedisModule_GetSelectedDb(RedisModuleCtx *ctx);

**可用日期：**4.0.0

返回当前选定的数据库。

<span id="RedisModule_GetContextFlags"></span>

### `RedisModule_GetContextFlags`

    int RedisModule_GetContextFlags(RedisModuleCtx *ctx);

**可用日期：**4.0.3

返回当前上下文的标志。这些标志提供了有关
当前请求上下文 (无论客户端是 Lua 脚本还是在 MULTI 中), , 
以及一般的Redis实例, 即复制和持久性。

但是, 即使使用 NULL 上下文也可以调用此函数
在这种情况下, 将不会报告以下标志：

*   LUA、多路、复制、脏污 (有关详细信息, 请参阅下文) 。

可用标志及其含义：

*   `REDISMODULE_CTX_FLAGS_LUA`：命令在 Lua 脚本中运行

*   `REDISMODULE_CTX_FLAGS_MULTI`：命令在事务内部运行

*   `REDISMODULE_CTX_FLAGS_REPLICATED`：命令是通过复制发送的
    由 MASTER 提供的链接

*   `REDISMODULE_CTX_FLAGS_MASTER`：Redis 实例是主实例

*   `REDISMODULE_CTX_FLAGS_SLAVE`：Redis 实例是从属实例

*   `REDISMODULE_CTX_FLAGS_READONLY`：Redis 实例是只读的

*   `REDISMODULE_CTX_FLAGS_CLUSTER`：Redis 实例处于集群模式

*   `REDISMODULE_CTX_FLAGS_AOF`：Redis 实例已启用 AOF

*   `REDISMODULE_CTX_FLAGS_RDB`：实例已启用 RDB

*   `REDISMODULE_CTX_FLAGS_MAXMEMORY`：实例设置了最大内存

*   `REDISMODULE_CTX_FLAGS_EVICT`：最大记忆已设置并具有驱逐
    可能删除密钥的策略

*   `REDISMODULE_CTX_FLAGS_OOM`：Redis 根据内存不足
    最大记忆设置。

*   `REDISMODULE_CTX_FLAGS_OOM_WARNING`：之前剩余的内存少于 25%
    达到最大记忆水平。

*   `REDISMODULE_CTX_FLAGS_LOADING`：服务器正在加载 RDB/AOF

*   `REDISMODULE_CTX_FLAGS_REPLICA_IS_STALE`：与主站没有活动链接。

*   `REDISMODULE_CTX_FLAGS_REPLICA_IS_CONNECTING`：复制副本正在尝试
    与主服务器连接。

*   `REDISMODULE_CTX_FLAGS_REPLICA_IS_TRANSFERRING`：主 -> 副本 RDB
    传输正在进行中。

*   `REDISMODULE_CTX_FLAGS_REPLICA_IS_ONLINE`：复制副本具有活动链接
    与其主人。这是
    与陈旧状态相反。

*   `REDISMODULE_CTX_FLAGS_ACTIVE_CHILD`：目前有一些背景
    进程活动 (RDB、AUX 或模块) 。

*   `REDISMODULE_CTX_FLAGS_MULTI_DIRTY`：下一个 EXEC 将因脏污而失败
    CAS (触摸过的按键) 。

*   `REDISMODULE_CTX_FLAGS_IS_CHILD`：Redis 当前正在内部运行
    后台子进程。

*   `REDISMODULE_CTX_FLAGS_RESP3`：指示附加到此客户端的
    上下文正在使用 RESP3。

<span id="RedisModule_AvoidReplicaTraffic"></span>

### `RedisModule_AvoidReplicaTraffic`

    int RedisModule_AvoidReplicaTraffic();

**可用日期：**6.0.0

如果客户端将 CLIENT PAUSE 命令发送到服务器或
如果 Redis 集群执行手动故障转移, 则暂停客户端。
当我们有一个带有副本的主节点并且想要写入时, 这是必需的, 
无需向复制通道添加更多数据, 则副本
复制偏移量, 与主服务器之一匹配。当这种情况发生时, 它是
安全地故障转移主服务器而不会丢失数据。

但是, 模块可以通过调用来生成流量[`RedisModule_Call()`](#RedisModule_Call)跟
“！” 标志, 或通过调用[`RedisModule_Replicate()`](#RedisModule_Replicate), 在外部上下文中
命令执行, 例如超时回调, 线程安全
上下文, 等等。当模块会产生太多的流量时, 它会
主副本和副本偏移量将很难匹配, 因为有
是要在复制通道中发送的更多数据。

因此, 模块可能希望尝试避免非常繁重的背景工作
创建数据到复制通道的效果, 当这个功能
返回 true。这对于具有背景的模块非常有用
垃圾回收任务, 或执行写入并复制此类写入的任务
定期在计时器回调或其他定期回调中。

<span id="RedisModule_SelectDb"></span>

### `RedisModule_SelectDb`

    int RedisModule_SelectDb(RedisModuleCtx *ctx, int newid);

**可用日期：**4.0.0

更改当前选定的数据库。如果 id
超出范围。

请注意, 客户端将保留当前选定的数据库, 即使在
由调用此函数的模块实现的 Redis 命令
返回。

如果模块命令希望更改其他数据库中的某些内容, 并且
返回原始的那个, 它应该调用[`RedisModule_GetSelectedDb()`](#RedisModule_GetSelectedDb)
之前, 以便在返回之前恢复旧的数据库编号。

<span id="RedisModule_KeyExists"></span>

### `RedisModule_KeyExists`

    int RedisModule_KeyExists(RedisModuleCtx *ctx, robj *keyname);

**可用日期：**7.0.0

检查密钥是否存在, 而不影响其上次访问时间。

这相当于调用[`RedisModule_OpenKey`](#RedisModule_OpenKey)与模式`REDISMODULE_READ`|
`REDISMODULE_OPEN_KEY_NOTOUCH`, 然后检查是否返回 NULL, 如果没有, 
叫[`RedisModule_CloseKey`](#RedisModule_CloseKey)在打开的密钥上。

<span id="RedisModule_OpenKey"></span>

### `RedisModule_OpenKey`

    RedisModuleKey *RedisModule_OpenKey(RedisModuleCtx *ctx,
                                        robj *keyname,
                                        int mode);

**可用日期：**4.0.0

返回表示 Redis 密钥的句柄, 以便可以
以密钥句柄作为参数来调用其他 API 以执行
对密钥的操作。

返回值是表示键的句柄, 该句柄必须是
闭合方式[`RedisModule_CloseKey()`](#RedisModule_CloseKey).

如果该键不存在并且请求了 WRITE 模式, 则句柄
仍然返回, 因为可以对
一个尚不存在的密钥 (例如, 在
列表推送操作) 。如果模式只是 REA, , 并且
键不存在, 则返回 NULL。但是, 它仍然是安全的
叫[`RedisModule_CloseKey()`](#RedisModule_CloseKey)和[`RedisModule_KeyType()`](#RedisModule_KeyType)在空值上
价值。

<span id="RedisModule_CloseKey"></span>

### `RedisModule_CloseKey`

    void RedisModule_CloseKey(RedisModuleKey *key);

**可用日期：**4.0.0

关闭键控点。

<span id="RedisModule_KeyType"></span>

### `RedisModule_KeyType`

    int RedisModule_KeyType(RedisModuleKey *key);

**可用日期：**4.0.0

返回密钥的类型。如果键指针为 NULL, 则
`REDISMODULE_KEYTYPE_EMPTY`返回。

<span id="RedisModule_ValueLength"></span>

### `RedisModule_ValueLength`

    size_t RedisModule_ValueLength(RedisModuleKey *key);

**可用日期：**4.0.0

返回与键关联的值的长度。
对于字符串, 这是字符串的长度。对于所有其他类型
是元素的数量 (只是计算哈希的键) 。

如果键指针为 NULL 或键为空, 则返回零。

<span id="RedisModule_DeleteKey"></span>

### `RedisModule_DeleteKey`

    int RedisModule_DeleteKey(RedisModuleKey *key);

**可用日期：**4.0.0

如果密钥处于打开状态以进行写入, 请将其删除, 并将密钥设置为
接受新写入作为空键 (将按需创建) 。
关于成功`REDISMODULE_OK`返回。如果密钥未打开
写作`REDISMODULE_ERR`返回。

<span id="RedisModule_UnlinkKey"></span>

### `RedisModule_UnlinkKey`

    int RedisModule_UnlinkKey(RedisModuleKey *key);

**可用日期：**4.0.7

如果密钥处于打开状态以进行写入, 请取消链接 (即在
非阻塞方式, 不立即回收内存) 并将密钥设置为
接受新写入作为空键 (将按需创建) 。
关于成功`REDISMODULE_OK`返回。如果密钥未打开
写作`REDISMODULE_ERR`返回。

<span id="RedisModule_GetExpire"></span>

### `RedisModule_GetExpire`

    mstime_t RedisModule_GetExpire(RedisModuleKey *key);

**可用日期：**4.0.0

返回密钥过期值, 即剩余 TTL 的毫秒数。
如果没有 TTL 与密钥关联, 或者密钥为空, 
`REDISMODULE_NO_EXPIRE`返回。

<span id="RedisModule_SetExpire"></span>

### `RedisModule_SetExpire`

    int RedisModule_SetExpire(RedisModuleKey *key, mstime_t expire);

**可用日期：**4.0.0

为密钥设置新的过期时间。如果特殊过期
`REDISMODULE_NO_EXPIRE`已设置, 如果存在
一个 (与“持久化”命令相同) 。

请注意, 过期必须以正整数的形式提供, 表示
密钥应具有的 TTL 毫秒数。

函数返回`REDISMODULE_OK`成功或`REDISMODULE_ERR`如果
密钥未打开以进行写入, 或者为空密钥。

<span id="RedisModule_GetAbsExpire"></span>

### `RedisModule_GetAbsExpire`

    mstime_t RedisModule_GetAbsExpire(RedisModuleKey *key);

**可用日期：**6.2.2

返回密钥过期值, 作为绝对 Unix 时间戳。
如果没有 TTL 与密钥关联, 或者密钥为空, 
`REDISMODULE_NO_EXPIRE`返回。

<span id="RedisModule_SetAbsExpire"></span>

### `RedisModule_SetAbsExpire`

    int RedisModule_SetAbsExpire(RedisModuleKey *key, mstime_t expire);

**可用日期：**6.2.2

为密钥设置新的过期时间。如果特殊过期
`REDISMODULE_NO_EXPIRE`已设置, 如果存在
一个 (与“持久化”命令相同) 。

请注意, 过期必须以正整数的形式提供, 表示
密钥应具有的绝对 Unix 时间戳。

函数返回`REDISMODULE_OK`成功或`REDISMODULE_ERR`如果
密钥未打开以进行写入, 或者为空密钥。

<span id="RedisModule_ResetDataset"></span>

### `RedisModule_ResetDataset`

    void RedisModule_ResetDataset(int restart_aof, int async);

**可用日期：**6.0.0

执行与 FLUSHALL 类似的操作, 并选择性地启动新的 AOF 文件 (如果启用) 
如果`restart_aof`为 true, 则必须确保触发此调用的命令不是
传播到 AOF 文件。
当 async 设置为 true 时, 数据库内容将由后台线程释放。

<span id="RedisModule_DbSize"></span>

### `RedisModule_DbSize`

    unsigned long long RedisModule_DbSize(RedisModuleCtx *ctx);

**可用日期：**6.0.0

返回当前数据库中的键数。

<span id="RedisModule_RandomKey"></span>

### `RedisModule_RandomKey`

    RedisModuleString *RedisModule_RandomKey(RedisModuleCtx *ctx);

**可用日期：**6.0.0

返回随机键的名称, 如果当前 db 为空, 则返回 NULL。

<span id="RedisModule_GetKeyNameFromOptCtx"></span>

### `RedisModule_GetKeyNameFromOptCtx`

    const RedisModuleString *RedisModule_GetKeyNameFromOptCtx(RedisModuleKeyOptCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的密钥的名称。

<span id="RedisModule_GetToKeyNameFromOptCtx"></span>

### `RedisModule_GetToKeyNameFromOptCtx`

    const RedisModuleString *RedisModule_GetToKeyNameFromOptCtx(RedisModuleKeyOptCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的目标键的名称。

<span id="RedisModule_GetDbIdFromOptCtx"></span>

### `RedisModule_GetDbIdFromOptCtx`

    int RedisModule_GetDbIdFromOptCtx(RedisModuleKeyOptCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的 dbid。

<span id="RedisModule_GetToDbIdFromOptCtx"></span>

### `RedisModule_GetToDbIdFromOptCtx`

    int RedisModule_GetToDbIdFromOptCtx(RedisModuleKeyOptCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的目标 dbid。

<span id="section-key-api-for-string-type"></span>

## 字符串类型的密钥 API

另请参见[`RedisModule_ValueLength()`](#RedisModule_ValueLength), 返回字符串的长度。

<span id="RedisModule_StringSet"></span>

### `RedisModule_StringSet`

    int RedisModule_StringSet(RedisModuleKey *key, RedisModuleString *str);

**可用日期：**4.0.0

如果密钥处于打开状态以进行写入, 请将指定的字符串“str”设置为
键的值, 删除旧值 (如果有) 。
关于成功`REDISMODULE_OK`返回。如果密钥未打开
写入或存在活动迭代器, `REDISMODULE_ERR`返回。

<span id="RedisModule_StringDMA"></span>

### `RedisModule_StringDMA`

    char *RedisModule_StringDMA(RedisModuleKey *key, size_t *len, int mode);

**可用日期：**4.0.0

为 DMA 访问准备键关联的字符串值, 并返回
指针和大小 (通过引用), , 用户可用于读取或
修改字符串就地通过指针直接访问它。

“模式”由按位 OR-ing 以下标志组成：

    REDISMODULE_READ -- Read access
    REDISMODULE_WRITE -- Write access

如果未请求 DMA 进行写入, 则返回的指针应
仅以只读方式访问。

出现错误 (类型错误) , , 返回 NULL。

DMA 访问规则：

1.  从目前起就不应该调用其他键写入函数
    指针被获取, 对于我们想要使用DMA访问的所有时间
    以读取或修改字符串。

2.  每次[`RedisModule_StringTruncate()`](#RedisModule_StringTruncate)调用, 以继续使用 DMA
    访问[`RedisModule_StringDMA()`](#RedisModule_StringDMA)应再次调用以重新获取
    新的指针和长度。

3.  如果返回的指针不是 NULL, 但长度为零, 则否
    可以触摸字节 (字符串为空, 或键本身为空) 
    所以一个[`RedisModule_StringTruncate()`](#RedisModule_StringTruncate)如果要放大, 应使用呼叫
    该字符串, 稍后再次调用 StringDMA ()  以获取指针。

<span id="RedisModule_StringTruncate"></span>

### `RedisModule_StringTruncate`

    int RedisModule_StringTruncate(RedisModuleKey *key, size_t newlen);

**可用日期：**4.0.0

如果密钥已打开以进行写入并且是字符串类型, 请调整其大小并填充
如果新长度大于旧长度, 则为零字节。

在这次电话会议之后, [`RedisModule_StringDMA()`](#RedisModule_StringDMA)必须再次调用才能继续
使用新指针进行 DMA 访问。

函数返回`REDISMODULE_OK`关于成功, 以及`REDISMODULE_ERR`上
错误, 即密钥未打开以进行写入, 不是字符串
或请求调整大小超过 512 MB。

如果该键为空, 则使用新的字符串值创建一个字符串键
除非请求的新长度值为零。

<span id="section-key-api-for-list-type"></span>

## 列表类型的关键 API

许多列表函数按索引访问元素。由于列表位于
本质是一个双链表, 按索引访问元素通常是
O (N)  操作。但, , 如果按顺序或与
索引靠近在一起, 函数经过优化以从中查找索引
以前的索引, 而不是从列表的末尾搜索。

这样就可以使用简单的 for 循环高效地完成迭代：

    long n = RedisModule_ValueLength(key);
    for (long i = 0; i < n; i++) {
        RedisModuleString *elem = RedisModule_ListGet(key, i);
        // Do stuff...
    }

请注意, 修改列表后使用[`RedisModule_ListPop`](#RedisModule_ListPop),[`RedisModule_ListSet`](#RedisModule_ListSet)或
[`RedisModule_ListInsert`](#RedisModule_ListInsert), 则内部迭代器无效, 因此下一个操作
将需要线性寻道。

以任何其他方式修改列表, 例如使用[`RedisModule_Call()`](#RedisModule_Call), 而键
是打开的会混淆内部迭代器, 如果密钥
在此类修改后使用。在这种情况下, 必须重新打开密钥。

另请参见[`RedisModule_ValueLength()`](#RedisModule_ValueLength), 返回列表的长度。

<span id="RedisModule_ListPush"></span>

### `RedisModule_ListPush`

    int RedisModule_ListPush(RedisModuleKey *key,
                             int where,
                             RedisModuleString *ele);

**可用日期：**4.0.0

将元素推入列表, 根据“where”参数从头到尾
(`REDISMODULE_LIST_HEAD`或`REDISMODULE_LIST_TAIL`).如果密钥引用
空键打开进行写入, 该键就被创建。关于成功, `REDISMODULE_OK`
返回。失败时, `REDISMODULE_ERR`返回并`errno`设置为
遵循：

*   EINVAL if key or ele is NULL.
*   如果密钥属于列表以外的其他类型, 则为 ENOTSUP。
*   EBADF (如果未打开密钥进行写入) 。

注意：在 Redis 7.0 之前, `errno`未由此函数设置。

<span id="RedisModule_ListPop"></span>

### `RedisModule_ListPop`

    RedisModuleString *RedisModule_ListPop(RedisModuleKey *key, int where);

**可用日期：**4.0.0

从列表中弹出一个元素, 并将其作为模块字符串对象返回
用户应该自由使用[`RedisModule_FreeString()`](#RedisModule_FreeString)或通过启用
自动记忆。这`where`参数指定元素是否应为
从列表的开头或结尾弹出  (`REDISMODULE_LIST_HEAD`或
`REDISMODULE_LIST_TAIL`).失败时, 该命令返回 NULL 并设置
`errno`如下：

*   如果键为空, 则为 EINVAL。
*   如果密钥为空或除列表以外的其他类型, 则为 ENOTSUP。
*   EBADF (如果未打开密钥进行写入) 。

注意：在 Redis 7.0 之前, `errno`未由此函数设置。

<span id="RedisModule_ListGet"></span>

### `RedisModule_ListGet`

    RedisModuleString *RedisModule_ListGet(RedisModuleKey *key, long index);

**可用日期：**7.0.0

返回索引处的元素`index`存储在 列表中`key`, 如
林德克斯命令。该元素应使用自由[`RedisModule_FreeString()`](#RedisModule_FreeString)或使用
自动内存管理。

索引从零开始, 因此 0 表示第一个元素, 1 表示第二个元素
等等。负指数可用于指定从
列表的尾部。此处, -1 表示最后一个元素, -2 表示倒数第二个元素
等等。

当在给定键和索引处找不到任何值时, 将返回 NULL 并
`errno`设置如下：

*   如果键为空, 则为 EINVAL。
*   如果密钥不是列表, 则为 ENOTSUP。
*   EBADF (如果未打开密钥进行读取) 。
*   如果索引不是列表中的有效索引, 则为 EDOM。

<span id="RedisModule_ListSet"></span>

### `RedisModule_ListSet`

    int RedisModule_ListSet(RedisModuleKey *key,
                            long index,
                            RedisModuleString *value);

**可用日期：**7.0.0

替换索引处的元素`index`存储在 列表中`key`.

索引从零开始, 因此 0 表示第一个元素, 1 表示第二个元素
等等。负指数可用于指定从
列表的尾部。此处, -1 表示最后一个元素, -2 表示倒数第二个元素
等等。

关于成功, `REDISMODULE_OK`返回。失败时, `REDISMODULE_ERR`是
返回和`errno`设置如下：

*   如果键或值为 NULL, 则为 EINVAL。
*   如果密钥不是列表, 则为 ENOTSUP。
*   EBADF (如果未打开密钥进行写入) 。
*   如果索引不是列表中的有效索引, 则为 EDOM。

<span id="RedisModule_ListInsert"></span>

### `RedisModule_ListInsert`

    int RedisModule_ListInsert(RedisModuleKey *key,
                               long index,
                               RedisModuleString *value);

**可用日期：**7.0.0

在给定索引处插入元素。

索引从零开始, 因此 0 表示第一个元素, 1 表示第二个元素
等等。负指数可用于指定从
列表的尾部。此处, -1 表示最后一个元素, -2 表示倒数第二个元素
等等。索引是元素插入后的索引。

关于成功, `REDISMODULE_OK`返回。失败时, `REDISMODULE_ERR`是
返回和`errno`设置如下：

*   如果键或值为 NULL, 则为 EINVAL。
*   ENOTSUP 如果键的类型比列表。
*   EBADF (如果未打开密钥进行写入) 。
*   如果索引不是列表中的有效索引, 则为 EDOM。

<span id="RedisModule_ListDelete"></span>

### `RedisModule_ListDelete`

    int RedisModule_ListDelete(RedisModuleKey *key, long index);

**可用日期：**7.0.0

删除给定索引处的元素。索引从 0 开始。负指数
也可以使用, 从列表的末尾开始计数。

关于成功, `REDISMODULE_OK`返回。失败时, `REDISMODULE_ERR`是
返回和`errno`设置如下：

*   如果键或值为 NULL, 则为 EINVAL。
*   如果密钥不是列表, 则为 ENOTSUP。
*   EBADF (如果未打开密钥进行写入) 。
*   如果索引不是列表中的有效索引, 则为 EDOM。

<span id="section-key-api-for-sorted-set-type"></span>

## 排序集类型的密钥 API

另请参见[`RedisModule_ValueLength()`](#RedisModule_ValueLength), 返回已排序集的长度。

<span id="RedisModule_ZsetAdd"></span>

### `RedisModule_ZsetAdd`

    int RedisModule_ZsetAdd(RedisModuleKey *key,
                            double score,
                            RedisModuleString *ele,
                            int *flagsptr);

**可用日期：**4.0.0

将新元素添加到具有指定“score”的排序集中。
如果该元素已存在, 则更新分数。

如果键是空的打开键, 则在值为时创建新的排序集
用于写入的设置。

其他标志可以通过指针传递给函数, 标志
都用于接收输入和在函数时传递状态
返回。如果未使用特殊标志, 则“flagsptr”可以为 NULL。

输入标志是：

    REDISMODULE_ZADD_XX: Element must already exist. Do nothing otherwise.
    REDISMODULE_ZADD_NX: Element must not exist. Do nothing otherwise.
    REDISMODULE_ZADD_GT: If element exists, new score must be greater than the current score. 
                         Do nothing otherwise. Can optionally be combined with XX.
    REDISMODULE_ZADD_LT: If element exists, new score must be less than the current score.
                         Do nothing otherwise. Can optionally be combined with XX.

输出标志是：

    REDISMODULE_ZADD_ADDED: The new element was added to the sorted set.
    REDISMODULE_ZADD_UPDATED: The score of the element was updated.
    REDISMODULE_ZADD_NOP: No operation was performed because XX or NX flags.

成功后, 函数返回`REDISMODULE_OK`.在以下错误上
`REDISMODULE_ERR`返回：

*   未打开密钥进行写入。
*   密钥的类型错误。
*   “分数”双精度值不是数字  (NaN) 。

<span id="RedisModule_ZsetIncrby"></span>

### `RedisModule_ZsetIncrby`

    int RedisModule_ZsetIncrby(RedisModuleKey *key,
                               double score,
                               RedisModuleString *ele,
                               int *flagsptr,
                               double *newscore);

**可用日期：**4.0.0

此功能的工作方式完全相同[`RedisModule_ZsetAdd()`](#RedisModule_ZsetAdd), 但不是设置
新分数, 现有元素的分数递增, 或者如果
元素尚不存在, 则假设旧分数为
零。

输入和输出标志以及返回值具有相同的精确度
意思是, 唯一的区别是这个函数将返回
`REDISMODULE_ERR`即使“分数”是有效的双精度数, 但要将其相加
将现有分数结果转换为 NaN (非数字) 条件。

此函数有一个附加字段“newscore”, 如果不是, 则填充 NULL
与元素的新分数在增量后, 如果没有错误
返回。

<span id="RedisModule_ZsetRem"></span>

### `RedisModule_ZsetRem`

    int RedisModule_ZsetRem(RedisModuleKey *key,
                            RedisModuleString *ele,
                            int *deleted);

**可用日期：**4.0.0

从排序集中删除指定的元素。
函数返回`REDISMODULE_OK`关于成功, 以及`REDISMODULE_ERR`
在下列情况之一：

*   未打开密钥进行写入。
*   密钥的类型错误。

返回值并不表示该元素确实是
删除 (因为它存在) 或不存, , 只是如果函数被执行
成功。

为了知道元素是否被删除, 附加参数
必须传递“已删除”, 通过引用填充整数
根据操作结果将其设置为 1 或 0。
如果调用方不感兴趣, 则“已删除”参数可以为 NULL
以了解该元素是否确实被删除。

空键将通过不执行任何操作来正确处理。

<span id="RedisModule_ZsetScore"></span>

### `RedisModule_ZsetScore`

    int RedisModule_ZsetScore(RedisModuleKey *key,
                              RedisModuleString *ele,
                              double *score);

**可用日期：**4.0.0

成功时, 检索与排序集元素关联的双倍分数
“ele”和返回`REDISMODULE_OK`.否则`REDISMODULE_ERR`返回
以发出以下条件之一的信号：

*   排序集中没有这样的元素“ele”。
*   键不是已排序的集合。
*   密钥是打开的空密钥。

<span id="section-key-api-for-sorted-set-iterator"></span>

## 排序集迭代器的密钥 API

<span id="RedisModule_ZsetRangeStop"></span>

### `RedisModule_ZsetRangeStop`

    void RedisModule_ZsetRangeStop(RedisModuleKey *key);

**可用日期：**4.0.0

停止排序集迭代。

<span id="RedisModule_ZsetRangeEndReached"></span>

### `RedisModule_ZsetRangeEndReached`

    int RedisModule_ZsetRangeEndReached(RedisModuleKey *key);

**可用日期：**4.0.0

返回“范围结束”标志值以指示迭代结束。

<span id="RedisModule_ZsetFirstInScoreRange"></span>

### `RedisModule_ZsetFirstInScoreRange`

    int RedisModule_ZsetFirstInScoreRange(RedisModuleKey *key,
                                          double min,
                                          double max,
                                          int minex,
                                          int maxex);

**可用日期：**4.0.0

设置一个排序集迭代器, 查找指定元素中的第一个元素
范围。返回`REDISMODULE_OK`如果迭代器已正确初始化
否则`REDISMODULE_ERR`在以下情况下返回：

1.  存储在键处的值不是排序集或键为空。

该范围是根据两个双精度值“min”和“max”指定的。
使用以下两个宏, 两者都可以是无限的：

*   `REDISMODULE_POSITIVE_INFINITE`对于正无限值
*   `REDISMODULE_NEGATIVE_INFINITE`对于负无限值

'minex' 和 'maxex' 参数, 如果为 true, 则分别设置一个范围
其中最小值和最大值是独占的 (不包括在内), , 而不是
包容。

<span id="RedisModule_ZsetLastInScoreRange"></span>

### `RedisModule_ZsetLastInScoreRange`

    int RedisModule_ZsetLastInScoreRange(RedisModuleKey *key,
                                         double min,
                                         double max,
                                         int minex,
                                         int maxex);

**可用日期：**4.0.0

完全像[`RedisModule_ZsetFirstInScoreRange()`](#RedisModule_ZsetFirstInScoreRange)但最后一个元素
而是选择范围作为迭代的开始。

<span id="RedisModule_ZsetFirstInLexRange"></span>

### `RedisModule_ZsetFirstInLexRange`

    int RedisModule_ZsetFirstInLexRange(RedisModuleKey *key,
                                        RedisModuleString *min,
                                        RedisModuleString *max);

**可用日期：**4.0.0

设置一个排序集迭代器, 查找指定元素中的第一个元素
词典编纂范围。返回`REDISMODULE_OK`如果迭代器正确
以其他方式初始化`REDISMODULE_ERR`在
以下情况：

1.  存储在键处的值不是排序集或键为空。
2.  词典范围“最小”和“最大”格式无效。

“最小值”和“最大值”应作为两个提供`RedisModuleString`对象
格式与传递给 ZRANGEBYLEX 命令的参数格式相同。
该函数不获取对象的所有权, 因此可以释放它们
设置迭代器后尽快完成。

<span id="RedisModule_ZsetLastInLexRange"></span>

### `RedisModule_ZsetLastInLexRange`

    int RedisModule_ZsetLastInLexRange(RedisModuleKey *key,
                                       RedisModuleString *min,
                                       RedisModuleString *max);

**可用日期：**4.0.0

完全像[`RedisModule_ZsetFirstInLexRange()`](#RedisModule_ZsetFirstInLexRange)但最后一个元素
而是选择范围作为迭代的开始。

<span id="RedisModule_ZsetRangeCurrentElement"></span>

### `RedisModule_ZsetRangeCurrentElement`

    RedisModuleString *RedisModule_ZsetRangeCurrentElement(RedisModuleKey *key,
                                                           double *score);

**可用日期：**4.0.0

返回活动排序集迭代器的当前排序集元素
如果迭代器中指定的范围不包含任何
元素。

<span id="RedisModule_ZsetRangeNext"></span>

### `RedisModule_ZsetRangeNext`

    int RedisModule_ZsetRangeNext(RedisModuleKey *key);

**可用日期：**4.0.0

转到排序集迭代器的下一个元素。如果存在
下一个元素, 如果我们已经在最新的元素或范围, 则为 0
根本不包括任何项目。

<span id="RedisModule_ZsetRangePrev"></span>

### `RedisModule_ZsetRangePrev`

    int RedisModule_ZsetRangePrev(RedisModuleKey *key);

**可用日期：**4.0.0

转到排序的集合迭代器的上一个元素。如果存在
前一个元素, 如果我们已经在第一个元素或范围, 则为 0
根本不包括任何项目。

<span id="section-key-api-for-hash-type"></span>

## 哈希类型的密钥 API

另请参见[`RedisModule_ValueLength()`](#RedisModule_ValueLength), 返回哈希中的字段数。

<span id="RedisModule_HashSet"></span>

### `RedisModule_HashSet`

    int RedisModule_HashSet(RedisModuleKey *key, int flags, ...);

**可用日期：**4.0.0

将指定哈希字段的字段设置为指定值。
如果密钥是打开以进行写入的空密钥, 则使用空密钥创建
哈希值, 以便设置指定的字段。

该函数是可变参数, 用户必须指定字段对
名称和值, 均为`RedisModuleString`指针 (除非
已设置 CFIELD 选项, 请参阅后面部分) 。在字段/值-ptr 对的末, , 
必须将 NULL 指定为最后一个参数, 以指示参数的结束
在可变参数功能中。

将哈希 argv\[1] 设置为值 argv\[2] 的示例：

     RedisModule_HashSet(key,REDISMODULE_HASH_NONE,argv[1],argv[2],NULL);

该函数还可用于删除字段 (如果存在) 
通过将它们设置为指定的值`REDISMODULE_HASH_DELETE`:

     RedisModule_HashSet(key,REDISMODULE_HASH_NONE,argv[1],
                         REDISMODULE_HASH_DELETE,NULL);

命令的行为会随着指定标志的更改, 这些标志可以是
设置为`REDISMODULE_HASH_NONE`如果不需要特殊行为。

    REDISMODULE_HASH_NX: The operation is performed only if the field was not
                         already existing in the hash.
    REDISMODULE_HASH_XX: The operation is performed only if the field was
                         already existing, so that a new value could be
                         associated to an existing filed, but no new fields
                         are created.
    REDISMODULE_HASH_CFIELDS: The field names passed are null terminated C
                              strings instead of RedisModuleString objects.
    REDISMODULE_HASH_COUNT_ALL: Include the number of inserted fields in the
                                returned number, in addition to the number of
                                updated and deleted fields. (Added in Redis
                                6.2.)

除非指定了 NX, 否则该命令将覆盖旧字段值
新的。

使用时`REDISMODULE_HASH_CFIELDS`, 则使用
正常的C字符串, 例如删除字段“foo”如下
可以使用的代码：

     RedisModule_HashSet(key,REDISMODULE_HASH_CFIELDS,"foo",
                         REDISMODULE_HASH_DELETE,NULL);

返回值：

调用前哈希中存在的字段数, 这些字段已
已更新 (其旧值已被新值替换) 或删除。如果
旗`REDISMODULE_HASH_COUNT_ALL`已设置, 插入的字段以前未设置
哈希中存在的也将被计算在内。

如果返回值为零, `errno`设置如下 (从 Redis 6.2 开始) ：

*   如果设置了任何未知标志或键为 NULL, 则为 EINVAL。
*   如果密钥与非哈希值相关联, 则为 ENOTSUP。
*   EBADF (如果密钥未打开以进行写入) 。
*   如果未按上述返回值下所述计算任何字段, 则为 ENOENT。
    这实际上不是一个错误。如果所有字段都可以为零, 则返回值可以为零
    刚刚创建并且`COUNT_ALL`标志未设置, 或者是否进行了更改
    由于 NX 和 XX 标志而返回。

注意：此函数的返回值语义非常不同
介于 Redis 6.2 和更早版本之间。使用它的模块应确定
Redis版本并相应地处理它。

<span id="RedisModule_HashGet"></span>

### `RedisModule_HashGet`

    int RedisModule_HashGet(RedisModuleKey *key, int flags, ...);

**可用日期：**4.0.0

从哈希值获取字段。此函数使用变量调用
参数数, 交替字段名称 (作为`RedisModuleString`
指针)  与指向`RedisModuleString`指, , 即设置为
字段的值 (如果字段存在) 或 NULL (如果字段不存在) 。
在字段/值-ptr 对的末尾, 必须将 NULL 指定为最后一个
参数, 用于指示可变参数函数中参数的结束。

下面是一个用法示例：

     RedisModuleString *first, *second;
     RedisModule_HashGet(mykey,REDISMODULE_HASH_NONE,argv[1],&first,
                         argv[2],&second,NULL);

与[`RedisModule_HashSet()`](#RedisModule_HashSet)可以指定命令的行为
传递不同于`REDISMODULE_HASH_NONE`:

`REDISMODULE_HASH_CFIELDS`：字段名称为空终止的 C 字符串。

`REDISMODULE_HASH_EXISTS`：而不是设置字段的值
期待一个`RedisModuleString`指针对指针, 函数刚好
报告字段是否存在, 并需要整数指针
作为每对的第二个元素。

示例`REDISMODULE_HASH_CFIELDS`:

     RedisModuleString *username, *hashedpass;
     RedisModule_HashGet(mykey,REDISMODULE_HASH_CFIELDS,"username",&username,"hp",&hashedpass, NULL);

示例`REDISMODULE_HASH_EXISTS`:

     int exists;
     RedisModule_HashGet(mykey,REDISMODULE_HASH_EXISTS,argv[1],&exists,NULL);

函数返回`REDISMODULE_OK`关于成功和`REDISMODULE_ERR`如果
密钥不是哈希值。

内存管理：

返回的`RedisModuleString`对象应随
[`RedisModule_FreeString()`](#RedisModule_FreeString), 或通过启用自动内存管理。

<span id="section-key-api-for-stream-type"></span>

## 流类型的密钥 API

有关流的介绍, 请参阅<https://redis.io/topics/streams-intro>.

类型`RedisModuleStreamID`, 用于流函数, 是一个结构
具有两个 64 位字段, 定义为

    typedef struct RedisModuleStreamID {
        uint64_t ms;
        uint64_t seq;
    } RedisModuleStreamID;

另请参见[`RedisModule_ValueLength()`](#RedisModule_ValueLength), 返回流的长度, 以及
转换函数[`RedisModule_StringToStreamID()`](#RedisModule_StringToStreamID)和[`RedisModule_CreateStringFromStreamID()`](#RedisModule_CreateStringFromStreamID).

<span id="RedisModule_StreamAdd"></span>

### `RedisModule_StreamAdd`

    int RedisModule_StreamAdd(RedisModuleKey *key,
                              int flags,
                              RedisModuleStreamID *id,
                              RedisModuleString **argv,
                              long numfields);

**可用日期：**6.2.0

将条目添加到流中。像XADD没有修剪。

*   `key`：存储流 (或将要存储) 的密钥
*   `flags`：一个有点的领域
    *   `REDISMODULE_STREAM_ADD_AUTOID`：自动分配流 ID, 例如
        `*`在 XADD 命令中。
*   `id`：如果`AUTOID`标志已设置, 这是分配的 ID 所在的位置
    返回。如果`AUTOID`已设置, 如果您不关心接收
    ID. 如果`AUTOID`未设置, 这是请求的 ID。
*   `argv`：指向大小数组的指针`numfields * 2`包含
    字段和值。
*   `numfields`：中的字段值对数`argv`.

返回`REDISMODULE_OK`如果已添加条目。失败时, 
`REDISMODULE_ERR`返回并`errno`设置如下：

*   使用无效参数调用 EINVAL
*   如果键引用的值不是流, 则为 ENOTSUP
*   EBADF (如果未打开密钥进行写入) 
*   如果给定的 ID 为 0-0 或不大于
    流 (仅当未设置 AUTOID 标志时) 
*   EFBIG (如果流已到达最后一个可能的 ID) 
*   如果元素太大而无法存储, 则为 ERANGE。

<span id="RedisModule_StreamDelete"></span>

### `RedisModule_StreamDelete`

    int RedisModule_StreamDelete(RedisModuleKey *key, RedisModuleStreamID *id);

**可用日期：**6.2.0

从流中删除条目。

*   `key`：打开密钥进行写入, 未启动流迭代器。
*   `id`：要删除的条目的流 ID。

返回`REDISMODULE_OK`关于成功。失败时, `REDISMODULE_ERR`返回
和`errno`设置如下：

*   使用无效参数调用 EINVAL
*   ENOTSUP 如果键引用非流类型的值, 或者如果
    键为空
*   EBADF (如果密钥未打开以进行写入, 或者如果流迭代器是
    与密钥关联
*   如果不存在具有给定流 ID 的条目, 则为 ENOENT

另请参见[`RedisModule_StreamIteratorDelete()`](#RedisModule_StreamIteratorDelete)用于删除当前条目, 同时
使用流迭代器进行迭代。

<span id="RedisModule_StreamIteratorStart"></span>

### `RedisModule_StreamIteratorStart`

    int RedisModule_StreamIteratorStart(RedisModuleKey *key,
                                        int flags,
                                        RedisModuleStreamID *start,
                                        RedisModuleStreamID *end);

**可用日期：**6.2.0

设置流迭代器。

*   `key`：打开流密钥以供读取[`RedisModule_OpenKey()`](#RedisModule_OpenKey).
*   `flags`:
    *   `REDISMODULE_STREAM_ITERATOR_EXCLUSIVE`：不包含`start`和`end`
        在迭代范围内。
    *   `REDISMODULE_STREAM_ITERATOR_REVERSE`：以相反的顺序迭代, 开始
        从`end`的范围。
*   `start`：范围的下限。将 NULL 用作
    流。
*   `end`：范围的上限。使用 NULL 作为流的末尾。

返回`REDISMODULE_OK`关于成功。失败时, `REDISMODULE_ERR`返回
和`errno`设置如下：

*   使用无效参数调用 EINVAL
*   ENOTSUP 如果键引用非流类型的值, 或者如果
    键为空
*   EBADF (如果密钥未打开以进行写入, 或者如果流迭代器是
    已与密钥关联
*   EDOM if`start`或`end`超出有效范围

返回`REDISMODULE_OK`关于成功和`REDISMODULE_ERR`如果密钥没有
引用流或是否给出了无效参数。

使用以下命令检索流 ID[`RedisModule_StreamIteratorNextID()`](#RedisModule_StreamIteratorNextID)和
对于每个流 ID, 使用以下命令检索字段和值
[`RedisModule_StreamIteratorNextField()`](#RedisModule_StreamIteratorNextField).通过调用来释放迭代器
[`RedisModule_StreamIteratorStop()`](#RedisModule_StreamIteratorStop).

示例 (省略错误处理) ：

    RedisModule_StreamIteratorStart(key, 0, startid_ptr, endid_ptr);
    RedisModuleStreamID id;
    long numfields;
    while (RedisModule_StreamIteratorNextID(key, &id, &numfields) ==
           REDISMODULE_OK) {
        RedisModuleString *field, *value;
        while (RedisModule_StreamIteratorNextField(key, &field, &value) ==
               REDISMODULE_OK) {
            //
            // ... Do stuff ...
            //
            RedisModule_FreeString(ctx, field);
            RedisModule_FreeString(ctx, value);
        }
    }
    RedisModule_StreamIteratorStop(key);

<span id="RedisModule_StreamIteratorStop"></span>

### `RedisModule_StreamIteratorStop`

    int RedisModule_StreamIteratorStop(RedisModuleKey *key);

**可用日期：**6.2.0

停止使用 创建的流迭代器[`RedisModule_StreamIteratorStart()`](#RedisModule_StreamIteratorStart)和
回收其内存。

返回`REDISMODULE_OK`关于成功。失败时, `REDISMODULE_ERR`返回
和`errno`设置如下：

*   EINVAL (如果使用空键调用) 
*   ENOTSUP 如果键引用非流类型的值, 或者如果
    键为空
*   EBADF (如果密钥未打开以进行写入, 或者没有流迭代器) 
    与密钥关联

<span id="RedisModule_StreamIteratorNextID"></span>

### `RedisModule_StreamIteratorNextID`

    int RedisModule_StreamIteratorNextID(RedisModuleKey *key,
                                         RedisModuleStreamID *id,
                                         long *numfields);

**可用日期：**6.2.0

查找下一个流条目并返回其流 ID 和
领域。

*   `key`：已使用流迭代器启动的键
    [`RedisModule_StreamIteratorStart()`](#RedisModule_StreamIteratorStart).
*   `id`：返回的流 ID。NULL, 如果您不在乎。
*   `numfields`：找到的流条目中的字段数。空, 如果您
    不在乎。

返回`REDISMODULE_OK`和集`*id`和`*numfields`如果找到条目。
失败时, `REDISMODULE_ERR`返回并`errno`设置如下：

*   EINVAL (如果使用空键调用) 
*   ENOTSUP 如果键引用非流类型的值, 或者如果
    键为空
*   EBADF (如果没有流迭代器与密钥关联) 
*   如果迭代器范围内没有更多条目, 则为 ENOENT

在实践中, 如果[`RedisModule_StreamIteratorNextID()`](#RedisModule_StreamIteratorNextID)在成功调用后调用
自[`RedisModule_StreamIteratorStart()`](#RedisModule_StreamIteratorStart)并且使用相同的密钥, 可以安全地假设
一`REDISMODULE_ERR`返回值表示没有更多条目。

用[`RedisModule_StreamIteratorNextField()`](#RedisModule_StreamIteratorNextField)以检索字段和值。
请参阅示例[`RedisModule_StreamIteratorStart()`](#RedisModule_StreamIteratorStart).

<span id="RedisModule_StreamIteratorNextField"></span>

### `RedisModule_StreamIteratorNextField`

    int RedisModule_StreamIteratorNextField(RedisModuleKey *key,
                                            RedisModuleString **field_ptr,
                                            RedisModuleString **value_ptr);

**可用日期：**6.2.0

检索当前流 ID 的下一个字段及其相应的值
在流迭代中。调用后应重复调用此函数
[`RedisModule_StreamIteratorNextID()`](#RedisModule_StreamIteratorNextID)以获取每个字段-值对。

*   `key`：已启动流迭代器的键。
*   `field_ptr`：这是返回字段的位置。
*   `value_ptr`：这是返回值的位置。

返回`REDISMODULE_OK`和点`*field_ptr`和`*value_ptr`新鲜
分配`RedisModuleString`对象。释放字符串对象
如果启用了自动内存, 则回调完成时自动执行。上
失败`REDISMODULE_ERR`返回并`errno`设置如下：

*   EINVAL (如果使用空键调用) 
*   ENOTSUP 如果键引用非流类型的值, 或者如果
    键为空
*   EBADF (如果没有流迭代器与密钥关联) 
*   如果当前流条目中没有更多字段, 则 ENOENT

在实践中, 如果[`RedisModule_StreamIteratorNextField()`](#RedisModule_StreamIteratorNextField)在成功后调用
调用[`RedisModule_StreamIteratorNextID()`](#RedisModule_StreamIteratorNextID)并且使用相同的密钥, 可以安全地假设
那一个`REDISMODULE_ERR`返回值表示不再有字段。

请参阅示例[`RedisModule_StreamIteratorStart()`](#RedisModule_StreamIteratorStart).

<span id="RedisModule_StreamIteratorDelete"></span>

### `RedisModule_StreamIteratorDelete`

    int RedisModule_StreamIteratorDelete(RedisModuleKey *key);

**可用日期：**6.2.0

在迭代时删除当前流条目。

此函数可在以下时间后调用[`RedisModule_StreamIteratorNextID()`](#RedisModule_StreamIteratorNextID)或之后
调用[`RedisModule_StreamIteratorNextField()`](#RedisModule_StreamIteratorNextField).

返回`REDISMODULE_OK`关于成功。失败时, `REDISMODULE_ERR`返回
和`errno`设置如下：

*   如果键为空, 则为 EINVAL
*   ENOTSUP 如果密钥为空或属于流以外的其他类型
*   EBADF 如果未打开密钥进行写入, 并且未启动任何迭代器
*   如果迭代器没有当前流条目, 则为 ENOENT

<span id="RedisModule_StreamTrimByLength"></span>

### `RedisModule_StreamTrimByLength`

    long long RedisModule_StreamTrimByLength(RedisModuleKey *key,
                                             int flags,
                                             long long length);

**可用日期：**6.2.0

按长度修剪流, 类似于使用 MAXLEN 的 XTRIM。

*   `key`：打开键进行写入。
*   `flags`： 一点点
    *   `REDISMODULE_STREAM_TRIM_APPROX`：如果能提高性能, 则减少修剪, 
        像 XTRIM 与`~`.
*   `length`：修剪后要保留的流条目数。

返回已删除的条目数。失败时, 负值为
返回和`errno`设置如下：

*   使用无效参数调用 EINVAL
*   ENOTSUP 如果密钥为空或属于流以外的类型
*   EBADF (如果未打开密钥进行写入) 

<span id="RedisModule_StreamTrimByID"></span>

### `RedisModule_StreamTrimByID`

    long long RedisModule_StreamTrimByID(RedisModuleKey *key,
                                         int flags,
                                         RedisModuleStreamID *id);

**可用日期：**6.2.0

按 ID 修剪流, 类似于使用 MINID 的 XTRIM。

*   `key`：打开键进行写入。
*   `flags`： 一点点
    *   `REDISMODULE_STREAM_TRIM_APPROX`：如果能提高性能, 则减少修剪, 
        像 XTRIM 与`~`.
*   `id`：修剪后要保留的最小流 ID。

返回已删除的条目数。失败时, 负值为
返回和`errno`设置如下：

*   使用无效参数调用 EINVAL
*   ENOTSUP 如果密钥为空或属于流以外的类型
*   EBADF (如果未打开密钥进行写入) 

<span id="section-calling-redis-commands-from-modules"></span>

## 从模块调用 Redis 命令

[`RedisModule_Call()`](#RedisModule_Call)向 Redis 发送命令。其余函数处理回复。

<span id="RedisModule_FreeCallReply"></span>

### `RedisModule_FreeCallReply`

    void RedisModule_FreeCallReply(RedisModuleCallReply *reply);

**可用日期：**4.0.0

释放呼叫回复及其包含的所有嵌套回复 (如果它是
数组。

<span id="RedisModule_CallReplyType"></span>

### `RedisModule_CallReplyType`

    int RedisModule_CallReplyType(RedisModuleCallReply *reply);

**可用日期：**4.0.0

将答复类型作为下列值之一返回：

*   `REDISMODULE_REPLY_UNKNOWN`
*   `REDISMODULE_REPLY_STRING`
*   `REDISMODULE_REPLY_ERROR`
*   `REDISMODULE_REPLY_INTEGER`
*   `REDISMODULE_REPLY_ARRAY`
*   `REDISMODULE_REPLY_NULL`
*   `REDISMODULE_REPLY_MAP`
*   `REDISMODULE_REPLY_SET`
*   `REDISMODULE_REPLY_BOOL`
*   `REDISMODULE_REPLY_DOUBLE`
*   `REDISMODULE_REPLY_BIG_NUMBER`
*   `REDISMODULE_REPLY_VERBATIM_STRING`
*   `REDISMODULE_REPLY_ATTRIBUTE`

<span id="RedisModule_CallReplyLength"></span>

### `RedisModule_CallReplyLength`

    size_t RedisModule_CallReplyLength(RedisModuleCallReply *reply);

**可用日期：**4.0.0

返回回复类型长度 (如果适用) 。

<span id="RedisModule_CallReplyArrayElement"></span>

### `RedisModule_CallReplyArrayElement`

    RedisModuleCallReply *RedisModule_CallReplyArrayElement(RedisModuleCallReply *reply,
                                                            size_t idx);

**可用日期：**4.0.0

返回数组回复的第 'idx' 个嵌套调用回复元素, 或 NULL
如果回复类型错误或索引超出范围。

<span id="RedisModule_CallReplyInteger"></span>

### `RedisModule_CallReplyInteger`

    long long RedisModule_CallReplyInteger(RedisModuleCallReply *reply);

**可用日期：**4.0.0

返回`long long`的整数回复。

<span id="RedisModule_CallReplyDouble"></span>

### `RedisModule_CallReplyDouble`

    double RedisModule_CallReplyDouble(RedisModuleCallReply *reply);

**可用日期：**7.0.0

返回双回复的双精度值。

<span id="RedisModule_CallReplyBigNumber"></span>

### `RedisModule_CallReplyBigNumber`

    const char *RedisModule_CallReplyBigNumber(RedisModuleCallReply *reply,
                                               size_t *len);

**可用日期：**7.0.0

返回大数字回复的大数字值。

<span id="RedisModule_CallReplyVerbatim"></span>

### `RedisModule_CallReplyVerbatim`

    const char *RedisModule_CallReplyVerbatim(RedisModuleCallReply *reply,
                                              size_t *len,
                                              const char **format);

**可用日期：**7.0.0

返回逐字字符串回复的值, 
可以给出一个可选的输出参数来获取逐字回复格式。

<span id="RedisModule_CallReplyBool"></span>

### `RedisModule_CallReplyBool`

    int RedisModule_CallReplyBool(RedisModuleCallReply *reply);

**可用日期：**7.0.0

返回布尔回复的布尔值。

<span id="RedisModule_CallReplySetElement"></span>

### `RedisModule_CallReplySetElement`

    RedisModuleCallReply *RedisModule_CallReplySetElement(RedisModuleCallReply *reply,
                                                          size_t idx);

**可用日期：**7.0.0

返回设置回复的第 'idx' 个嵌套呼叫回复元素, 或 NULL
如果回复类型错误或索引超出范围。

<span id="RedisModule_CallReplyMapElement"></span>

### `RedisModule_CallReplyMapElement`

    int RedisModule_CallReplyMapElement(RedisModuleCallReply *reply,
                                        size_t idx,
                                        RedisModuleCallReply **key,
                                        RedisModuleCallReply **val);

**可用日期：**7.0.0

检索映射回复的第 'idx' 键和值。

返回：

*   `REDISMODULE_OK`关于成功。
*   `REDISMODULE_ERR`如果 idx 超出范围或回复类型错误。

这`key`和`value`参数用于通过引用返回, 并且可能是
如果不需要, 则为空。

<span id="RedisModule_CallReplyAttribute"></span>

### `RedisModule_CallReplyAttribute`

    RedisModuleCallReply *RedisModule_CallReplyAttribute(RedisModuleCallReply *reply);

**可用日期：**7.0.0

返回给定答复的属性, 如果不存在任何属性, 则返回 NULL。

<span id="RedisModule_CallReplyAttributeElement"></span>

### `RedisModule_CallReplyAttributeElement`

    int RedisModule_CallReplyAttributeElement(RedisModuleCallReply *reply,
                                              size_t idx,
                                              RedisModuleCallReply **key,
                                              RedisModuleCallReply **val);

**可用日期：**7.0.0

检索属性回复的第 'idx' 个键和值。

返回：

*   `REDISMODULE_OK`关于成功。
*   `REDISMODULE_ERR`如果 idx 超出范围或回复类型错误。

这`key`和`value`参数用于通过引用返回, 并且可能是
如果不需要, 则为空。

<span id="RedisModule_CallReplyStringPtr"></span>

### `RedisModule_CallReplyStringPtr`

    const char *RedisModule_CallReplyStringPtr(RedisModuleCallReply *reply,
                                               size_t *len);

**可用日期：**4.0.0

返回字符串或错误回复的指针和长度。

<span id="RedisModule_CreateStringFromCallReply"></span>

### `RedisModule_CreateStringFromCallReply`

    RedisModuleString *RedisModule_CreateStringFromCallReply(RedisModuleCallReply *reply);

**可用日期：**4.0.0

从字符串、错误或类型的调用回复中返回新的字符串对象
整数。否则 (错误的回复类型) 返回 NULL。

<span id="RedisModule_Call"></span>

### `RedisModule_Call`

    RedisModuleCallReply *RedisModule_Call(RedisModuleCtx *ctx,
                                           const char *cmdname,
                                           const char *fmt,
                                           ...);

**可用日期：**4.0.0

导出的 API 以从模块调用任何 Redis 命令。

*   **cmdname**：要调用的 Redis 命令。
*   **断续器**：命令参数的格式说明符字符串。每
    的参数应由有效的类型规范指定。这
    格式说明符也可以包含修饰符`!`,`A`,`3`和`R`哪
    没有相应的参数。

    *   `b`-- 参数是缓冲区, 紧跟另一个缓冲区
        参数, 即缓冲区的长度。
    *   `c`-- 该参数是指向纯 C 字符串 (以 null 结尾) 的指针。
    *   `l`-- 参数是`long long`整数。
    *   `s`-- 这个参数是一个 RedisModuleString。
    *   `v`-- 参数是 RedisModuleString 的向量。
    *   `!`-- 将 Redis 命令及其参数发送到副本和 AOF。
    *   `A`-- 抑制 AOF 传播, 仅发送到副本 (需要`!`).
    *   `R`-- 禁止复制副本传播, 仅发送到 AOF (需要`!`).
    *   `3`-- 返回 RESP3 回复。这将更改命令回复。
        例如, HGETALL 返回一个映射而不是一个平面数组。
    *   `0`-- 以自动模式返回回复, 即回复格式将为
        与附加到给定 RedisModuleCtx 的客户端相同。这将
        可能在要将答复直接传递给客户端时使用。
    *   `C`-- 检查命令是否可以根据 ACL 规则执行。
    *   `S`-- 在脚本模式下运行命令, 这意味着它将引发
        如果脚本中不允许的命令, 则会出现错误
         (标记有`deny-script`标志)  被调用 (如 SHUTDOWN) 。
        此外, 在脚本模式下, 如果存在
        没有足够的良好副本 (配置了`min-replicas-to-write`)
        或者当服务器无法保存到磁盘时。
    *   `W`-- 不允许运行任何写入命令 (标记为`write`标志) 。
    *   `M`-- 不允许`deny-oom`超过内存限制时标记的命令。
    *   `E`-- 将错误作为 RedisModuleCallReply 返回。如果之前有错误
        调用该命令, 使用 errno 机制返回错误。
        此标志允许将错误也作为错误 CallReply 获取
        相关的错误消息。
*   **...**：Redis 命令的实际参数。

关于成功`RedisModuleCallReply`返回对象, 否则
返回 NULL 并将 errno 设置为以下值：

*   EBADF：格式说明符错误。
*   EINVAL：错误的命令 arity。
*   ENOENT：命令不存在。
*   EPERM：在群集实例中操作, 密钥在非本地插槽中。
*   EROFS：发送写入命令时在群集实例中执行的操作
    处于只读状态。
*   ENETDOWN：集群关闭时在集群实例中操作。
*   ENOTSUP：指定模块上下文没有 ACL 用户
*   EACCES：根据 ACL 规则, 无法执行命令
*   ENOSPC：不允许写入或拒绝 oom 命令
*   ESPIPE：在脚本模式下不允许使用命令

示例代码片段：

     reply = RedisModule_Call(ctx,"INCRBY","sc",argv[1],"10");
     if (RedisModule_CallReplyType(reply) == REDISMODULE_REPLY_INTEGER) {
       long long myval = RedisModule_CallReplyInteger(reply);
       // Do something with myval.
     }

此 API 记录如下：<https://redis.io/topics/modules-intro>

<span id="RedisModule_CallReplyProto"></span>

### `RedisModule_CallReplyProto`

    const char *RedisModule_CallReplyProto(RedisModuleCallReply *reply,
                                           size_t *len);

**可用日期：**4.0.0

返回指向命令返回的协议的指针和长度
返回回复对象。

<span id="section-modules-data-types"></span>

## 模块数据类型

当字符串 DMA 或使用现有数据结构是不够的时, 它是
可以从头开始创建新的数据类型并将其导出到
雷迪斯。模块必须提供一组回调来处理
导出的新值 (例如, 为了提供 RDB 保存/加载, 
AOF 重写, 依此类推) 。在本节, , 我们将定义此 API。

<span id="RedisModule_CreateDataType"></span>

### `RedisModule_CreateDataType`

    moduleType *RedisModule_CreateDataType(RedisModuleCtx *ctx,
                                           const char *name,
                                           int encver,
                                           void *typemethods_ptr);

**可用日期：**4.0.0

注册模块导出的新数据类型。参数是
以后。如需深入的文档, 请查看 API 模块
文档, 尤其是<https://redis.io/topics/modules-native-types>.

*   **名字**：在 Redis 中必须唯一的 9 个字符的数据类型名称
    模块生态系统。要有创意...并且不会有碰撞。用
    字符集 A-Z a-z 9-0, 加上两个“-\_”字符。一个好的
    想法是使用, 例如`<typename>-<vendor>`.例如
    “tree-AntZ”可能意味着“@antirez树数据结构”。同时使用两者
    小写和大写字母有助于防止冲突。

*   **恩克维尔**：编码版本, 即序列化的版本
    用于持久保存数据的模块。只要“名”即可
    匹配, RDB加载将被调度到类型回调
    无论使用什么“encver”, 但是模块可以理解
    它必须加载的编码是模块的旧版本。
    例如, 模块“tree-AntZ”最初使用encver=0。后
    升级后, 它开始以不同的格式序列化数据
    并使用 encver=1 注册类型。但是, 此模块可能
    如果`rdb_load`
    回调能够检查 encver 值并采取相应的行动。
    encver 必须是介于 0 和 1023 之间的正值。

*   **typemethods_ptr**是指向`RedisModuleTypeMethods`结构
    应使用方法回调和结构进行填充
    版本, 如以下示例所示：

          RedisModuleTypeMethods tm = {
              .version = REDISMODULE_TYPE_METHOD_VERSION,
              .rdb_load = myType_RDBLoadCallBack,
              .rdb_save = myType_RDBSaveCallBack,
              .aof_rewrite = myType_AOFRewriteCallBack,
              .free = myType_FreeCallBack,

              // Optional fields
              .digest = myType_DigestCallBack,
              .mem_usage = myType_MemUsageCallBack,
              .aux_load = myType_AuxRDBLoadCallBack,
              .aux_save = myType_AuxRDBSaveCallBack,
              .free_effort = myType_FreeEffortCallBack,
              .unlink = myType_UnlinkCallBack,
              .copy = myType_CopyCallback,
              .defrag = myType_DefragCallback

              // Enhanced optional fields
              .mem_usage2 = myType_MemUsageCallBack2,
              .free_effort2 = myType_FreeEffortCallBack2,
              .unlink2 = myType_UnlinkCallBack2,
              .copy2 = myType_CopyCallback2,
          }

*   **rdb_load**：从 RDB 文件加载数据的回调函数指针。

*   **rdb_save**：将数据保存到 RDB 文件的回调函数指针。

*   **aof_rewrite**：将数据重写为命令的回调函数指针。

*   **消化**：用于`DEBUG DIGEST`.

*   **自由**：可以释放类型值的回调函数指针。

*   **aux_save**：一个回调函数指针, 用于将键空间数据保存到 RDB 文件。
    “当”参数为`REDISMODULE_AUX_BEFORE_RDB`或`REDISMODULE_AUX_AFTER_RDB`.

*   **aux_load**：从 RDB 文件中加载出键空间数据的回调函数指针。
    似`aux_save`返回`REDISMODULE_OK`关于成功, 否则错误。

*   **free_effort**：一个回调函数指针, 用于确定模块的
    内存需要延迟回收。该模块应返回
    释放值。例如：要释放多少个指针。请注意, 如果
    返回 0, 我们将始终执行异步免费操作。

*   **取消链接**：用于通知模块密钥具有的回调函数指针
    已通过 redis 从 DB 中删除, 可能很快就会被后台线程释放。请注意, 
    它不会在 FLUSHALL/FLUSHDB 上调用 (同步和异步), , 并且模块可以使用
    `RedisModuleEvent_FlushDB`钩进去。

*   **复制**：用于创建指定密钥副本的回调函数指针。
    该模块应执行指定值的深层复制并返回它。
    此外, 还提供了有关源键和目标键名称的提示。
    NULL 返回值被视为错误, 复制操作将失败。
    注意：如果目标键存在并且正在被覆盖, 则复制回调将为
    首先调用, 然后对要替换的值进行空闲回调。

*   **碎片整理**：用于请求模块进行碎片整理的回调函数指针
    一个键。然后, 模块应迭代指针并调用相关`RedisModule_Defrag*()`
    用于对指针或复杂类型进行碎片整理的函数。该模块应继续
    迭代只要[`RedisModule_DefragShouldStop()`](#RedisModule_DefragShouldStop)返回一个零值, 并返回
    如果已完成, 则为零值;如果还有更多工作要做, 则为非零值。如果更多工作
    需要做, [`RedisModule_DefragCursorSet()`](#RedisModule_DefragCursorSet)和[`RedisModule_DefragCursorGet()`](#RedisModule_DefragCursorGet)可用于跟踪
    这适用于不同的调用。
    通常, 碎片整理机制调用回调没有时间限制, 因此
    [`RedisModule_DefragShouldStop()`](#RedisModule_DefragShouldStop)始终返回零。“后期碎片整理”机制具有以下特点：
    时间限制和提供游标支持仅用于已确定的键
    具有显著的内部复杂性。若要确定这一点, 请使用碎片整理机制
    使用`free_effort`回调和“活动碎片整理最大扫描字段”配置指令。
    注意：该值作为传递`void**`并且该函数有望更新
    指针 (如果顶级值指针已碎片整理并因此发生更改) 。

*   **mem_usage2**：类似于`mem_usage`, 但提供了`RedisModuleKeyOptCtx`参数
    以便可以获得元信息, 例如键名和数据库ID, 以及
    这`sample_size`用于大小估计 (请参见“内存使用”命令) 。

*   **free_effort2**：类似于`free_effort`, 但提供了`RedisModuleKeyOptCtx`参数
    以便可以获得元信息, 例如键名和数据库 ID。

*   **取消链接2**：类似于`unlink`, 但提供了`RedisModuleKeyOptCtx`参数
    以便可以获得元信息, 例如键名和数据库 ID。

*   **复制2**：类似于`copy`, 但提供了`RedisModuleKeyOptCtx`参数
    以便可以获得元信息, 例如键名和数据库 ID。

注意：模块名称“AAAAAAAAA”被保留并产生错误, 它
碰巧也相当蹩脚。

如果已经有一个模块注册具有相同名称的类型, 
如果模块名称或 encver 无效, 则返回 NULL。
否则, 新类型将注册到 Redis 中, 并且引用
类型`RedisModuleType`返回：函数的调用方应存储
这个引用到一个全局变量中, 以便将来在
模块类型 API, 因为单个模块可以注册多个类型。
示例代码片段：

     static RedisModuleType *BalancedTreeType;

     int RedisModule_OnLoad(RedisModuleCtx *ctx) {
         // some code here ...
         BalancedTreeType = RedisModule_CreateDataType(...);
     }

<span id="RedisModule_ModuleTypeSetValue"></span>

### `RedisModule_ModuleTypeSetValue`

    int RedisModule_ModuleTypeSetValue(RedisModuleKey *key,
                                       moduleType *mt,
                                       void *value);

**可用日期：**4.0.0

如果键处于打开状态以进行写入, 请设置指定的模块类型对象
作为键的值, 删除旧值 (如果有) 。
关于成功`REDISMODULE_OK`返回。如果密钥未打开
写入或存在活动迭代器, `REDISMODULE_ERR`返回。

<span id="RedisModule_ModuleTypeGetType"></span>

### `RedisModule_ModuleTypeGetType`

    moduleType *RedisModule_ModuleTypeGetType(RedisModuleKey *key);

**可用日期：**4.0.0

若[`RedisModule_KeyType()`](#RedisModule_KeyType)返回`REDISMODULE_KEYTYPE_MODULE`上
键, 返回存储在键处的值的模块类型指针。

如果键为 NULL、未与模块类型关联或为空, 
然后返回 NULL。

<span id="RedisModule_ModuleTypeGetValue"></span>

### `RedisModule_ModuleTypeGetValue`

    void *RedisModule_ModuleTypeGetValue(RedisModuleKey *key);

**可用日期：**4.0.0

若[`RedisModule_KeyType()`](#RedisModule_KeyType)返回`REDISMODULE_KEYTYPE_MODULE`上
键, 返回存储在键处的模块类型低级值, 如
它是由用户通过以下方式设置的[`RedisModule_ModuleTypeSetValue()`](#RedisModule_ModuleTypeSetValue).

如果键为 NULL、未与模块类型关联或为空, 
然后返回 NULL。

<span id="section-rdb-loading-and-saving-functions"></span>

## RDB 加载和保存功能

<span id="RedisModule_IsIOError"></span>

### `RedisModule_IsIOError`

    int RedisModule_IsIOError(RedisModuleIO *io);

**可用日期：**6.0.0

如果任何以前的 IO API 失败, 则返回 true。
为`Load*`API`REDISMODULE_OPTIONS_HANDLE_IO_ERRORS`标志必须设置为
[`RedisModule_SetModuleOptions`](#RedisModule_SetModuleOptions)第一。

<span id="RedisModule_SaveUnsigned"></span>

### `RedisModule_SaveUnsigned`

    void RedisModule_SaveUnsigned(RedisModuleIO *io, uint64_t value);

**可用日期：**4.0.0

将无符号的 64 位值保存到 RDB 文件中。此函数应仅
在`rdb_save`模块实现新方法的方法
数据类型。

<span id="RedisModule_LoadUnsigned"></span>

### `RedisModule_LoadUnsigned`

    uint64_t RedisModule_LoadUnsigned(RedisModuleIO *io);

**可用日期：**4.0.0

从 RDB 文件加载无符号的 64 位值。此函数应仅
在`rdb_load`模块实现的方法
新的数据类型。

<span id="RedisModule_SaveSigned"></span>

### `RedisModule_SaveSigned`

    void RedisModule_SaveSigned(RedisModuleIO *io, int64_t value);

**可用日期：**4.0.0

喜欢[`RedisModule_SaveUnsigned()`](#RedisModule_SaveUnsigned)但对于有符号的 64 位值。

<span id="RedisModule_LoadSigned"></span>

### `RedisModule_LoadSigned`

    int64_t RedisModule_LoadSigned(RedisModuleIO *io);

**可用日期：**4.0.0

喜欢[`RedisModule_LoadUnsigned()`](#RedisModule_LoadUnsigned)但对于有符号的 64 位值。

<span id="RedisModule_SaveString"></span>

### `RedisModule_SaveString`

    void RedisModule_SaveString(RedisModuleIO *io, RedisModuleString *s);

**可用日期：**4.0.0

在`rdb_save`模块类型的方法, 保存
字符串到RDB文件中, 将一个作为输入`RedisModuleString`.

该字符串可以在以后加载[`RedisModule_LoadString()`](#RedisModule_LoadString)或
其他 加载系列函数, 需要内部序列化字符串
RDB 文件。

<span id="RedisModule_SaveStringBuffer"></span>

### `RedisModule_SaveStringBuffer`

    void RedisModule_SaveStringBuffer(RedisModuleIO *io,
                                      const char *str,
                                      size_t len);

**可用日期：**4.0.0

喜欢[`RedisModule_SaveString()`](#RedisModule_SaveString)但需要一个原始的C指针和长度
作为输入。

<span id="RedisModule_LoadString"></span>

### `RedisModule_LoadString`

    RedisModuleString *RedisModule_LoadString(RedisModuleIO *io);

**可用日期：**4.0.0

在`rdb_load`模块数据类型的方法, 加载字符串
从 RDB 文件中, 该文件之前保存为[`RedisModule_SaveString()`](#RedisModule_SaveString)
函数系列。

返回的字符串是新分配的字符串`RedisModuleString`对象, 以及
用户应该在某个时候通过调用来释放它[`RedisModule_FreeString()`](#RedisModule_FreeString).

如果数据结构不将字符串存储为`RedisModuleString`对象
类似功能[`RedisModule_LoadStringBuffer()`](#RedisModule_LoadStringBuffer)可以改用。

<span id="RedisModule_LoadStringBuffer"></span>

### `RedisModule_LoadStringBuffer`

    char *RedisModule_LoadStringBuffer(RedisModuleIO *io, size_t *lenptr);

**可用日期：**4.0.0

喜欢[`RedisModule_LoadString()`](#RedisModule_LoadString)但返回一个堆分配的字符串, 该字符串
已分配与[`RedisModule_Alloc()`](#RedisModule_Alloc), 并可以使用 调整大小或释放
[`RedisModule_Realloc()`](#RedisModule_Realloc)或[`RedisModule_Free()`](#RedisModule_Free).

字符串的大小存储在'\*lenptr' (如果不是 NULL) 。
返回的字符串不会自动以 NULL 终止, 而是加载
与存储在 RDB 文件中的内容完全相同。

<span id="RedisModule_SaveDouble"></span>

### `RedisModule_SaveDouble`

    void RedisModule_SaveDouble(RedisModuleIO *io, double value);

**可用日期：**4.0.0

在`rdb_save`模块数据类型的方法, 保存双精度
值添加到 RDB 文件。双精度数可以是有效数、NaN 或无穷大。
可以加载回值[`RedisModule_LoadDouble()`](#RedisModule_LoadDouble).

<span id="RedisModule_LoadDouble"></span>

### `RedisModule_LoadDouble`

    double RedisModule_LoadDouble(RedisModuleIO *io);

**可用日期：**4.0.0

在`rdb_save`模块数据类型的方法, 装回
双倍值保存者[`RedisModule_SaveDouble()`](#RedisModule_SaveDouble).

<span id="RedisModule_SaveFloat"></span>

### `RedisModule_SaveFloat`

    void RedisModule_SaveFloat(RedisModuleIO *io, float value);

**可用日期：**4.0.0

在`rdb_save`模块数据类型的方法, 保存浮点数
值添加到 RDB 文件。浮点数可以是有效数字、NaN 或无穷大。
可以加载回值[`RedisModule_LoadFloat()`](#RedisModule_LoadFloat).

<span id="RedisModule_LoadFloat"></span>

### `RedisModule_LoadFloat`

    float RedisModule_LoadFloat(RedisModuleIO *io);

**可用日期：**4.0.0

在`rdb_save`模块数据类型的方法, 装回
浮点值保存者[`RedisModule_SaveFloat()`](#RedisModule_SaveFloat).

<span id="RedisModule_SaveLongDouble"></span>

### `RedisModule_SaveLongDouble`

    void RedisModule_SaveLongDouble(RedisModuleIO *io, long double value);

**可用日期：**6.0.0

在`rdb_save`模块数据类型的方法, 保存长双精度
值添加到 RDB 文件。双精度数可以是有效数、NaN 或无穷大。
可以加载回值[`RedisModule_LoadLongDouble()`](#RedisModule_LoadLongDouble).

<span id="RedisModule_LoadLongDouble"></span>

### `RedisModule_LoadLongDouble`

    long double RedisModule_LoadLongDouble(RedisModuleIO *io);

**可用日期：**6.0.0

在`rdb_save`模块数据类型的方法, 装回
长双倍值保存者[`RedisModule_SaveLongDouble()`](#RedisModule_SaveLongDouble).

<span id="section-key-digest-api-debug-digest-interface-for-modules-types"></span>

## 密钥摘要 API (模块类型的调试摘要接口) 

<span id="RedisModule_DigestAddStringBuffer"></span>

### `RedisModule_DigestAddStringBuffer`

    void RedisModule_DigestAddStringBuffer(RedisModuleDigest *md,
                                           const char *ele,
                                           size_t len);

**可用日期：**4.0.0

向摘要中添加新元素。此函数可以多次调用
一个接一个的元素, 对于构成给定元素的所有元素
数据结构。函数调用后必须跟有
[`RedisModule_DigestEndSequence`](#RedisModule_DigestEndSequence)最终, 当所有元素
始终按给定的顺序添加。请参阅 Redis 模块数据类型
文档以获取更多信息。但是, 这是一个使用Redis的快速示例
以数据类型为例。

添加无序元素序列 (例如, 在 Redis 的情况下) 
Set), , 要使用的模式是：

    foreach element {
        AddElement(element);
        EndSequence();
    }

因为集合没有顺序, 所以添加的每个元素都有一个位置
不依赖于另一个。但是, 如果我们的元素是
成对排序, 就像哈希的字段值对一样, 那么应该
用：

    foreach key,value {
        AddElement(key);
        AddElement(value);
        EndSequence();
    }

因为键和值将始终按上述顺序排列, 而相反
单个键值对可以出现在 Redis 哈希值的任何位置。

有序元素列表将通过以下方式实现：

    foreach element {
        AddElement(element);
    }
    EndSequence();

<span id="RedisModule_DigestAddLongLong"></span>

### `RedisModule_DigestAddLongLong`

    void RedisModule_DigestAddLongLong(RedisModuleDigest *md, long long ll);

**可用日期：**4.0.0

喜欢[`RedisModule_DigestAddStringBuffer()`](#RedisModule_DigestAddStringBuffer)但需要一个`long long`作为输入
在将其添加到摘要之前将其转换为字符串。

<span id="RedisModule_DigestEndSequence"></span>

### `RedisModule_DigestEndSequence`

    void RedisModule_DigestEndSequence(RedisModuleDigest *md);

**可用日期：**4.0.0

请参阅文档`RedisModule_DigestAddElement()`.

<span id="RedisModule_LoadDataTypeFromStringEncver"></span>

### `RedisModule_LoadDataTypeFromStringEncver`

    void *RedisModule_LoadDataTypeFromStringEncver(const RedisModuleString *str,
                                                   const moduleType *mt,
                                                   int encver);

**可用日期：**7.0.0

解码模块数据类型“mt”的序列化表示形式, 采用特定编码版本“encver”
从字符串“str”并返回新分配的值, 如果解码失败, 则返回 NULL。

此调用基本上重用了 '`rdb_load`' 回调哪些模块数据类型
实现, 以便允许模块任意序列化/反序列化
键, 类似于 Redis 'DUMP' 和 'RESTORE' 命令的实现方式。

模块通常应使用`REDISMODULE_OPTIONS_HANDLE_IO_ERRORS`标志和
确保反序列化代码正确检查并处理 IO 错误
 (释放分配的缓冲区并返回 NULL) 。

如果不这样做, Redis将处理损坏的 (或只是截断的) 序列化
通过生成错误消息并终止进程来获取数据。

<span id="RedisModule_LoadDataTypeFromString"></span>

### `RedisModule_LoadDataTypeFromString`

    void *RedisModule_LoadDataTypeFromString(const RedisModuleString *str,
                                             const moduleType *mt);

**可用日期：**6.0.0

似[`RedisModule_LoadDataTypeFromStringEncver`](#RedisModule_LoadDataTypeFromStringEncver), API 的原始版本, 保留
实现向后兼容性。

<span id="RedisModule_SaveDataTypeToString"></span>

### `RedisModule_SaveDataTypeToString`

    RedisModuleString *RedisModule_SaveDataTypeToString(RedisModuleCtx *ctx,
                                                        void *data,
                                                        const moduleType *mt);

**可用日期：**6.0.0

将模块数据类型“mt”值“data”编码为序列化形式, 并将其返回
作为新分配的`RedisModuleString`.

此调用基本上重用了 '`rdb_save`' 回调哪些模块数据类型
实现, 以便允许模块任意序列化/反序列化
键, 类似于 Redis 'DUMP' 和 'RESTORE' 命令的实现方式。

<span id="RedisModule_GetKeyNameFromDigest"></span>

### `RedisModule_GetKeyNameFromDigest`

    const RedisModuleString *RedisModule_GetKeyNameFromDigest(RedisModuleDigest *dig);

**可用日期：**7.0.0

返回当前正在处理的密钥的名称。

<span id="RedisModule_GetDbIdFromDigest"></span>

### `RedisModule_GetDbIdFromDigest`

    int RedisModule_GetDbIdFromDigest(RedisModuleDigest *dig);

**可用日期：**7.0.0

返回当前正在处理的密钥的数据库 ID。

<span id="section-aof-api-for-modules-data-types"></span>

## 模块数据类型的 AOF API

<span id="RedisModule_EmitAOF"></span>

### `RedisModule_EmitAOF`

    void RedisModule_EmitAOF(RedisModuleIO *io,
                             const char *cmdname,
                             const char *fmt,
                             ...);

**可用日期：**4.0.0

在 AOF 重写过程中向 AOF 发出命令。此函数
仅在`aof_rewrite`导出的数据类型的方法
通过模块。该命令的工作方式与[`RedisModule_Call()`](#RedisModule_Call)在途中
参数被传递, 但它不会返回任何内容作为错误
处理由 Redis 本身执行。

<span id="section-io-context-handling"></span>

## IO 上下文处理

<span id="RedisModule_GetKeyNameFromIO"></span>

### `RedisModule_GetKeyNameFromIO`

    const RedisModuleString *RedisModule_GetKeyNameFromIO(RedisModuleIO *io);

**可用日期：**5.0.5

返回当前正在处理的密钥的名称。
不能保证密钥名称始终可用, 因此可能会返回 NULL。

<span id="RedisModule_GetKeyNameFromModuleKey"></span>

### `RedisModule_GetKeyNameFromModuleKey`

    const RedisModuleString *RedisModule_GetKeyNameFromModuleKey(RedisModuleKey *key);

**可用日期：**6.0.0

返回`RedisModuleString`与密钥的名称从`RedisModuleKey`.

<span id="RedisModule_GetDbIdFromModuleKey"></span>

### `RedisModule_GetDbIdFromModuleKey`

    int RedisModule_GetDbIdFromModuleKey(RedisModuleKey *key);

**可用日期：**7.0.0

返回密钥的数据库 ID, 从`RedisModuleKey`.

<span id="RedisModule_GetDbIdFromIO"></span>

### `RedisModule_GetDbIdFromIO`

    int RedisModule_GetDbIdFromIO(RedisModuleIO *io);

**可用日期：**7.0.0

返回当前正在处理的密钥的数据库 ID。
无法保证此信息始终可用, 因此可能会返回 -1。

<span id="section-logging"></span>

## 伐木

<span id="RedisModule_Log"></span>

### `RedisModule_Log`

    void RedisModule_Log(RedisModuleCtx *ctx,
                         const char *levelstr,
                         const char *fmt,
                         ...);

**可用日期：**4.0.0

向标准 Redis 日志生成日志消息, 格式接受
类似 printf 的说明符, 而 level 是描述日志的字符串
级别, 以便在发出日志时使用, 并且必须是下列值之一：

*   “调试” (`REDISMODULE_LOGLEVEL_DEBUG`)
*   “详细” (`REDISMODULE_LOGLEVEL_VERBOSE`)
*   “通知” (`REDISMODULE_LOGLEVEL_NOTICE`)
*   “警告” (`REDISMODULE_LOGLEVEL_WARNING`)

如果指定的日志级别无效, 则默认使用详细。
对日志行的长度有固定限制, 此函数能够
发出, 此限制未指定, 但保证超过
几行文本。

如果无法在
实例线程或回调的调用方, 在这种情况下, 通用“模块”
将使用而不是模块名称。

<span id="RedisModule_LogIOError"></span>

### `RedisModule_LogIOError`

    void RedisModule_LogIOError(RedisModuleIO *io,
                                const char *levelstr,
                                const char *fmt,
                                ...);

**可用日期：**4.0.0

记录来自 RDB/AOF 序列化回调的错误。

当回调返回临界值时, 应使用此函数
错误给调用方, 因为无法加载或保存某些数据
关键原因。

<span id="RedisModule__Assert"></span>

### `RedisModule__Assert`

    void RedisModule__Assert(const char *estr, const char *file, int line);

**可用日期：**6.0.0

类似 Redis 的断言函数。

宏`RedisModule_Assert(expression)`建议使用, 而不是
直接调用此函数。

失败的断言将关闭服务器并生成日志记录信息
看起来与 Redis 本身生成的信息相同。

<span id="RedisModule_LatencyAddSample"></span>

### `RedisModule_LatencyAddSample`

    void RedisModule_LatencyAddSample(const char *event, mstime_t latency);

**可用日期：**6.0.0

允许将事件添加到延迟监视器, 以便延迟观察
命令。如果延迟小于配置的
延迟监视器阈值。

<span id="section-blocking-clients-from-modules"></span>

## 阻止模块中的客户端

有关阻止模块中的命令的指南, 请参阅
<https://redis.io/topics/modules-blocking-ops>.

<span id="RedisModule_BlockClient"></span>

### `RedisModule_BlockClient`

    RedisModuleBlockedClient *RedisModule_BlockClient(RedisModuleCtx *ctx,
                                                      RedisModuleCmdFunc reply_callback,
                                                      RedisModuleCmdFunc timeout_callback,
                                                      void (*free_privdata)(RedisModuleCtx*, void*),
                                                      long long timeout_ms);

**可用日期：**4.0.0

在阻塞命令的上下文中阻止客户端, 返回句柄
稍后将使用, 以便通过调用来取消阻止客户端
[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient).参数指定回调函数
和超时, 在此之后客户端将解除阻止。

回调在以下上下文中调用：

    reply_callback:   called after a successful RedisModule_UnblockClient()
                      call in order to reply to the client and unblock it.

    timeout_callback: called when the timeout is reached or if `CLIENT UNBLOCK`
                      is invoked, in order to send an error to the client.

    free_privdata:    called in order to free the private data that is passed
                      by RedisModule_UnblockClient() call.

注意：[`RedisModule_UnblockClient`](#RedisModule_UnblockClient)应该为每个被阻止的客户端调用, 
即使客户端被终止、超时或断开连接。如果不这样做
将导致内存泄漏。

在某些情况下, [`RedisModule_BlockClient()`](#RedisModule_BlockClient)不能使用：

1.  如果客户端是 Lua 脚本。
2.  如果客户端正在执行 MULTI 块。

在这些情况下, 调用[`RedisModule_BlockClient()`](#RedisModule_BlockClient)将**不**阻止
客户端, 但改为生成特定的错误回复。

注册`timeout_callback`功能也可以解锁
使用`CLIENT UNBLOCK`命令, 这将触发超时回调。
如果未注册回调函数, 则被阻止的客户端将是
被视为未处于阻塞状态, 并且`CLIENT UNBLOCK`会再来
零值。

测量后台时间：默认情况下, 在阻止的命令中花费的时间
不考虑总命令持续时间。要包括这样的时间, 你应该
用[`RedisModule_BlockedClientMeasureTimeStart()`](#RedisModule_BlockedClientMeasureTimeStart)和[`RedisModule_BlockedClientMeasureTimeEnd()`](#RedisModule_BlockedClientMeasureTimeEnd)一
或在后台工作中多次阻止命令。

<span id="RedisModule_BlockClientOnKeys"></span>

### `RedisModule_BlockClientOnKeys`

    RedisModuleBlockedClient *RedisModule_BlockClientOnKeys(RedisModuleCtx *ctx,
                                                            RedisModuleCmdFunc reply_callback,
                                                            RedisModuleCmdFunc timeout_callback,
                                                            void (*free_privdata)(RedisModuleCtx*, void*),
                                                            long long timeout_ms,
                                                            RedisModuleString **keys,
                                                            int numkeys,
                                                            void *privdata);

**可用日期：**6.0.0

此调用类似于[`RedisModule_BlockClient()`](#RedisModule_BlockClient), 但是在这种情况下, 我们
不要只是屏蔽客户端, 还要要求 Redis 自动取消屏蔽它
一旦某些键变得“就绪”, 即包含更多数据。

基本上, 这与典型的Redis命令通常执行的操作类似, 
像BLPOP或BZPOPMAX：如果无法尽快提供服务, 客户端就会阻止, 
稍后, 当键接收到新数据 (例如列表推送) , , 
客户端已解除阻止并提供。

但是, 对于此模块API, 当客户端未被阻止时？

1.  如果对具有关联阻止操作的类型的密钥进行阻止, 
    就像列表, 排序集, 流等一样, 客户端可能是
    一旦相关密钥被通常操作作为目标, 则取消阻止
    取消阻止该类型的本机阻止操作。因此, 如果我们阻止
    在列表键上, RPUSH 命令可以取消阻止我们的客户, 依此类推。
2.  如果要实现本机数据类型, 或者要添加新的
    解封条件除了“1”, 还可以调用模块API
    [`RedisModule_SignalKeyAsReady()`](#RedisModule_SignalKeyAsReady).

无论如何, 我们无法确定客户端是否应该被解除阻止, 只是因为
键被发出准备就绪的信号：例如, 连续操作可能会更改
键, 或在此客户端提供服务之前排队, 修改密钥
以及使它再次空。因此, 当客户端被阻止时
[`RedisModule_BlockClientOnKeys()`](#RedisModule_BlockClientOnKeys)回复回调在之后不被调用
[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient)被调用, 但每次将密钥标记为就绪时：
如果回复回调可以为客户端提供服务, 则返回`REDISMODULE_OK`
并且客户端已解除阻止, 否则它将返回`REDISMODULE_ERR`
我们稍后再试。

回复回调可以访问发出准备就绪信号的密钥
调用接口[`RedisModule_GetBlockedClientReadyKey()`](#RedisModule_GetBlockedClientReadyKey), 返回
只是键的字符串名称作为`RedisModuleString`对象。

多亏了这个系统, 我们可以设置复杂的阻塞场景, 比如
仅当列表包含至少 5 个项目或其他项目时取消阻止客户端
更花哨的逻辑。

请注意, 另一个区别是[`RedisModule_BlockClient()`](#RedisModule_BlockClient), 就是在这里
我们在阻止客户端时直接传递私人数据：它将
稍后在回复回调中可访问。通常当阻塞
[`RedisModule_BlockClient()`](#RedisModule_BlockClient)回复客户端的私人数据是
呼叫时通过[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient)但在这里解锁
是由Redis本身执行的, 所以我们需要有一些私人数据之前
手。私有数据用于存储有关特定信息的任何信息
取消阻止正在实现的操作。此类信息将是
使用`free_privdata`用户提供的回调。

但是, 回复回调将能够访问参数向量
命令, 因此通常不需要私有数据。

注：一般情况下[`RedisModule_UnblockClient`](#RedisModule_UnblockClient)不应该是
调用在密钥上被阻止的客户端 (密钥将
准备就绪或将发生超时) 。如果由于某种原因你确实想要
呼叫RedisModule_UnblockClient有可能：客户端将是
像超时一样处理 (必须实现超时
在这种情况下回调) 。

<span id="RedisModule_SignalKeyAsReady"></span>

### `RedisModule_SignalKeyAsReady`

    void RedisModule_SignalKeyAsReady(RedisModuleCtx *ctx, RedisModuleString *key);

**可用日期：**6.0.0

使用此函数是为了可能取消阻止被阻止的客户端
在键上[`RedisModule_BlockClientOnKeys()`](#RedisModule_BlockClientOnKeys).调用此函数时, 
为此密钥阻止的所有客户端都将获得他们的`reply_callback`叫。

注意：如果信号键不存在, 则该函数不起作用。

<span id="RedisModule_UnblockClient"></span>

### `RedisModule_UnblockClient`

    int RedisModule_UnblockClient(RedisModuleBlockedClient *bc, void *privdata);

**可用日期：**4.0.0

取消阻止被阻止的客户端`RedisModule_BlockedClient`.这将触发
要调用的回复回调, 以便回复客户端。
“privdata”参数可以通过回复回调访问, 因此
此函数的调用方可以传递所需的任何值, 以便
实际回复客户。

“privdata”的常见用法是计算某些东西的线程
需要传递给客户端, 包括但不限于一些慢速
以计算回复或通过网络获得的一些回复。

注1：这个函数可以从模块生成的线程中调用。

注意 2：当我们取消阻止使用 API 阻止密钥的客户端时
[`RedisModule_BlockClientOnKeys()`](#RedisModule_BlockClientOnKeys), 则此处未使用 privdata 参数。
取消阻止已使用此 API 阻止密钥的客户端仍将
需要客户端得到一些回复, 所以函数会使用
“超时”处理程序, 以便执行此操作 (在
[`RedisModule_BlockClientOnKeys()`](#RedisModule_BlockClientOnKeys)可从超时访问
回调方式[`RedisModule_GetBlockedClientPrivateData`](#RedisModule_GetBlockedClientPrivateData)).

<span id="RedisModule_AbortBlock"></span>

### `RedisModule_AbortBlock`

    int RedisModule_AbortBlock(RedisModuleBlockedClient *bc);

**可用日期：**4.0.0

中止被阻止的客户端阻止操作：客户端将被解除阻止
不触发任何回调。

<span id="RedisModule_SetDisconnectCallback"></span>

### `RedisModule_SetDisconnectCallback`

    void RedisModule_SetDisconnectCallback(RedisModuleBlockedClient *bc,
                                           RedisModuleDisconnectFunc callback);

**可用日期：**5.0.0

设置在被阻止的客户端断开连接时将调用的回调
在模块有机会调用之前[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient)

通常, 您要在那里做的是清理模块状态
以便您可以致电[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient)安全, 否则
如果超时很大, 客户端将永远保持阻止状态。

笔记：

1.  在这里调用Reply\*家庭功能是不安全的, 它也是
    无用, 因为客户端消失了。

2.  如果客户端由于以下原因而断开连接, 则不会调用此回调
    超时。在这种情况下, 客户端会自动解除阻止
    并调用超时回调。

<span id="RedisModule_IsBlockedReplyRequest"></span>

### `RedisModule_IsBlockedReplyRequest`

    int RedisModule_IsBlockedReplyRequest(RedisModuleCtx *ctx);

**可用日期：**4.0.0

如果调用了模块命令以填充
回复被阻止的客户端。

<span id="RedisModule_IsBlockedTimeoutRequest"></span>

### `RedisModule_IsBlockedTimeoutRequest`

    int RedisModule_IsBlockedTimeoutRequest(RedisModuleCtx *ctx);

**可用日期：**4.0.0

如果调用了模块命令以填充
回复已超时的被阻止客户端。

<span id="RedisModule_GetBlockedClientPrivateData"></span>

### `RedisModule_GetBlockedClientPrivateData`

    void *RedisModule_GetBlockedClientPrivateData(RedisModuleCtx *ctx);

**可用日期：**4.0.0

通过以下方式获取私有数据集[`RedisModule_UnblockClient()`](#RedisModule_UnblockClient)

<span id="RedisModule_GetBlockedClientReadyKey"></span>

### `RedisModule_GetBlockedClientReadyKey`

    RedisModuleString *RedisModule_GetBlockedClientReadyKey(RedisModuleCtx *ctx);

**可用日期：**6.0.0

获取在上下文中调用回复回调时准备好的密钥
的客户端被阻止[`RedisModule_BlockClientOnKeys()`](#RedisModule_BlockClientOnKeys).

<span id="RedisModule_GetBlockedClientHandle"></span>

### `RedisModule_GetBlockedClientHandle`

    RedisModuleBlockedClient *RedisModule_GetBlockedClientHandle(RedisModuleCtx *ctx);

**可用日期：**5.0.0

获取与给定上下文关联的被阻止客户端。
这在被阻止的客户端的回复和超时回调中很有用, 
在模块有时具有被阻止的客户端句柄引用之前
周围, 想清理一下。

<span id="RedisModule_BlockedClientDisconnected"></span>

### `RedisModule_BlockedClientDisconnected`

    int RedisModule_BlockedClientDisconnected(RedisModuleCtx *ctx);

**可用日期：**5.0.0

如果调用被阻止客户端的空闲回调时, 返回 true, 
客户端被解除阻止的原因是它断开连接
当它被阻止时。

<span id="section-thread-safe-contexts"></span>

## 线程安全上下文

<span id="RedisModule_GetThreadSafeContext"></span>

### `RedisModule_GetThreadSafeContext`

    RedisModuleCtx *RedisModule_GetThreadSafeContext(RedisModuleBlockedClient *bc);

**可用日期：**4.0.0

返回可在线程内用于创建 Redis 上下文的上下文
使用某些模块 API 进行调用。如果 'bc' 不是 NULL, 则模块将
绑定到被阻止的客户端, 并且可以使用
`RedisModule_Reply*`函数族累积一个回复时
客户端将被解除阻止。否则, 线程安全上下文将是
由特定客户端分离。

要调用非应答 API, 必须使用以下命令准备线程安全上下文：

    RedisModule_ThreadSafeContextLock(ctx);
    ... make your call here ...
    RedisModule_ThreadSafeContextUnlock(ctx);

使用时不需要这样做`RedisModule_Reply*`函数,  假设
在创建上下文时使用了被阻止的客户端, 否则
不`RedisModule_Reply`\*应该打电话。

注意：如果要创建分离的线程安全上下文 (bc 为 NULL), , 
考虑使用[`RedisModule_GetDetachedThreadSafeContext`](#RedisModule_GetDetachedThreadSafeContext)这也将保留
模块 ID, 因此对于日志记录更有用。

<span id="RedisModule_GetDetachedThreadSafeContext"></span>

### `RedisModule_GetDetachedThreadSafeContext`

    RedisModuleCtx *RedisModule_GetDetachedThreadSafeContext(RedisModuleCtx *ctx);

**可用日期：**6.0.9

返回不与任何上下文关联的分离线程安全上下文
特定的被阻止客户端, 但与模块的上下文相关联。

这对于希望保持全局上下文的模块非常有用
长期, 用于日志记录等目的。

<span id="RedisModule_FreeThreadSafeContext"></span>

### `RedisModule_FreeThreadSafeContext`

    void RedisModule_FreeThreadSafeContext(RedisModuleCtx *ctx);

**可用日期：**4.0.0

释放线程安全上下文。

<span id="RedisModule_ThreadSafeContextLock"></span>

### `RedisModule_ThreadSafeContextLock`

    void RedisModule_ThreadSafeContextLock(RedisModuleCtx *ctx);

**可用日期：**4.0.0

在执行线程安全 API 调用之前获取服务器锁。
对于`RedisModule_Reply*`有呼叫时
连接到线程安全上下文的被阻止客户端。

<span id="RedisModule_ThreadSafeContextTryLock"></span>

### `RedisModule_ThreadSafeContextTryLock`

    int RedisModule_ThreadSafeContextTryLock(RedisModuleCtx *ctx);

**可用日期：**6.0.8

似[`RedisModule_ThreadSafeContextLock`](#RedisModule_ThreadSafeContextLock)但这个功能
如果已获取服务器锁, 则不会阻止。

如果成功 (已获取锁定) `REDISMODULE_OK`返, , 
否则`REDISMODULE_ERR`返回并设置 errno
因此。

<span id="RedisModule_ThreadSafeContextUnlock"></span>

### `RedisModule_ThreadSafeContextUnlock`

    void RedisModule_ThreadSafeContextUnlock(RedisModuleCtx *ctx);

**可用日期：**4.0.0

在执行线程安全 API 调用后释放服务器锁。

<span id="section-module-keyspace-notifications-api"></span>

## 模块键空间通知 API

<span id="RedisModule_SubscribeToKeyspaceEvents"></span>

### `RedisModule_SubscribeToKeyspaceEvents`

    int RedisModule_SubscribeToKeyspaceEvents(RedisModuleCtx *ctx,
                                              int types,
                                              RedisModuleNotificationFunc callback);

**可用日期：**4.0.9

订阅密钥空间通知。这是
键空间通知 API。模块可以注册要通知的回调
当键空间事件发生时。

通知事件按其类型 (字符串事件、设置事件、
等), , 并且订阅者回调仅接收与特定事件匹配的事件
事件类型的掩码。

订阅通知时使用[`RedisModule_SubscribeToKeyspaceEvents`](#RedisModule_SubscribeToKeyspaceEvents)
模块必须提供事件类型掩码, 表示订阅者的事件
是感兴趣的。这可以是以下任何标志的 ORed 掩码：

*   `REDISMODULE_NOTIFY_GENERIC`：通用命令, 如 DEL、EXPIRE、重命名
*   `REDISMODULE_NOTIFY_STRING`：字符串事件
*   `REDISMODULE_NOTIFY_LIST`：列出事件
*   `REDISMODULE_NOTIFY_SET`：设置事件
*   `REDISMODULE_NOTIFY_HASH`：哈希事件
*   `REDISMODULE_NOTIFY_ZSET`：排序设置事件
*   `REDISMODULE_NOTIFY_EXPIRED`：过期事件
*   `REDISMODULE_NOTIFY_EVICTED`：驱逐事件
*   `REDISMODULE_NOTIFY_STREAM`：流事件
*   `REDISMODULE_NOTIFY_MODULE`：模块类型事件
*   `REDISMODULE_NOTIFY_KEYMISS`：未按键事件
*   `REDISMODULE_NOTIFY_ALL`：所有事件 (不包括) `REDISMODULE_NOTIFY_KEYMISS`)
*   `REDISMODULE_NOTIFY_LOADED`：仅适用于模块的特殊通知, 
    指示密钥是从持久性加载的。
    请注意, 当此事件触发时, 给定的键
    不能保留, 使用RedisModule_CreateStringFromString
    相反。

我们不区分关键事件和关键空间事件, 它是向上的
以筛选基于键执行的操作。

订阅者签名为：

    int (*RedisModuleNotificationFunc) (RedisModuleCtx *ctx, int type,
                                        const char *event,
                                        RedisModuleString *key);

`type`是事件类型位, 必须与注册时给出的掩码匹配
时间。事件字符串是正在执行的实际命令, key 是
相关的 Redis 密钥。

通知回调使用不能
用于向客户端发送任何内容, 并具有事件所在的 db 编号
作为其选定的 db 编号出现。

请注意, 没有必要在 redis.conf 中启用通知
模块通知工作。

警告：通知回调以同步方式执行, 
因此, 通知回调必须很快, 否则它们会减慢 Redis 的速度。
如果需要执行长时间操作, 请使用线程卸载它们。

看<https://redis.io/topics/notifications>了解更多信息。

<span id="RedisModule_GetNotifyKeyspaceEvents"></span>

### `RedisModule_GetNotifyKeyspaceEvents`

    int RedisModule_GetNotifyKeyspaceEvents();

**可用日期：**6.0.0

获取已配置的通知键空间事件的位图 (可以使用
用于其他过滤`RedisModuleNotificationFunc`)

<span id="RedisModule_NotifyKeyspaceEvent"></span>

### `RedisModule_NotifyKeyspaceEvent`

    int RedisModule_NotifyKeyspaceEvent(RedisModuleCtx *ctx,
                                        int type,
                                        const char *event,
                                        RedisModuleString *key);

**可用日期：**6.0.0

公开通知密钥空间事件到模块

<span id="section-modules-cluster-api"></span>

## 模块集群 API

<span id="RedisModule_RegisterClusterMessageReceiver"></span>

### `RedisModule_RegisterClusterMessageReceiver`

    void RedisModule_RegisterClusterMessageReceiver(RedisModuleCtx *ctx,
                                                    uint8_t type,
                                                    RedisModuleClusterMessageReceiver callback);

**可用日期：**5.0.0

为类型为“type”的群集消息注册回调接收器。如果有
已经是注册的回调, 这将替换回调函数
使用提供的那个, 否则如果回调设置为 NULL 并且存在
已经是此函数的回调, 该回调未注册
 (因此, 此 API 调用也用于删除接收器) 。

<span id="RedisModule_SendClusterMessage"></span>

### `RedisModule_SendClusterMessage`

    int RedisModule_SendClusterMessage(RedisModuleCtx *ctx,
                                       const char *target_id,
                                       uint8_t type,
                                       const char *msg,
                                       uint32_t len);

**可用日期：**5.0.0

在以下情况下, 向群集中的所有节点发送消息：`target`为空, 否则
在指定的目标处, 即`REDISMODULE_NODE_ID_LEN`字节节点 ID, 作为
由接收方回调或节点迭代函数返回。

函数返回`REDISMODULE_OK`如果消息已成功发送, 
否则, 如果节点未连接或此类节点 ID 未映射到任何节点 ID
已知群集节点, `REDISMODULE_ERR`返回。

<span id="RedisModule_GetClusterNodesList"></span>

### `RedisModule_GetClusterNodesList`

    char **RedisModule_GetClusterNodesList(RedisModuleCtx *ctx, size_t *numnodes);

**可用日期：**5.0.0

返回一个字符串指针数组, 每个字符串指针指向一个簇
节点 ID 正好`REDISMODULE_NODE_ID_LEN`字节 (不带任何空术语) 。
返回的节点 ID 数存储到`*numnodes`.
但是, 如果此函数由未运行 Redis 的模块调用
实例启用了 Redis 集群, 则返回 NULL。

返回的 ID 可以与[`RedisModule_GetClusterNodeInfo()`](#RedisModule_GetClusterNodeInfo)挨次
以获取有关单节点的更多信息。

此函数返回的数组必须使用函数释放
[`RedisModule_FreeClusterNodesList()`](#RedisModule_FreeClusterNodesList).

例：

    size_t count, j;
    char **ids = RedisModule_GetClusterNodesList(ctx,&count);
    for (j = 0; j < count; j++) {
        RedisModule_Log(ctx,"notice","Node %.*s",
            REDISMODULE_NODE_ID_LEN,ids[j]);
    }
    RedisModule_FreeClusterNodesList(ids);

<span id="RedisModule_FreeClusterNodesList"></span>

### `RedisModule_FreeClusterNodesList`

    void RedisModule_FreeClusterNodesList(char **ids);

**可用日期：**5.0.0

释放获得的节点列表[`RedisModule_GetClusterNodesList`](#RedisModule_GetClusterNodesList).

<span id="RedisModule_GetMyClusterID"></span>

### `RedisModule_GetMyClusterID`

    const char *RedisModule_GetMyClusterID(void);

**可用日期：**5.0.0

返回此节点 ID  (`REDISMODULE_CLUSTER_ID_LEN`字节)  或 NULL (如果群集
已禁用。

<span id="RedisModule_GetClusterSize"></span>

### `RedisModule_GetClusterSize`

    size_t RedisModule_GetClusterSize(void);

**可用日期：**5.0.0

返回群集中的节点数, 而不考虑其状态
 (握手, 无地址, ...) 以便活动节点的数量实际上可能
较小, 但不超过此数字。如果实例不在
集群模式, 返回零。

<span id="RedisModule_GetClusterNodeInfo"></span>

### `RedisModule_GetClusterNodeInfo`

    int RedisModule_GetClusterNodeInfo(RedisModuleCtx *ctx,
                                       const char *id,
                                       char *ip,
                                       char *master_id,
                                       int *port,
                                       int *flags);

**可用日期：**5.0.0

填充 ID 为指定“id”的节点的指定信息, 
然后返回`REDISMODULE_OK`.否则, 如果节点 ID 的格式无效
或者此本地节点的 POV 中不存在节点 ID, `REDISMODULE_ERR`
返回。

参数`ip`,`master_id`,`port`和`flags`可以是 NULL, 以防我们不这样做
需要填充回某些信息。如果`ip`和`master_id` (仅填充
如果实例是从属实例) 被指, , 它们指向缓冲区保持
至少`REDISMODULE_NODE_ID_LEN`字节。字符串写回为`ip`
和`master_id`不以空值终止。

报告的标志列表如下：

*   `REDISMODULE_NODE_MYSELF`：此节点
*   `REDISMODULE_NODE_MASTER`：节点是主节点
*   `REDISMODULE_NODE_SLAVE`：节点是副本
*   `REDISMODULE_NODE_PFAIL`：我们看到节点出现故障
*   `REDISMODULE_NODE_FAIL`：群集同意节点出现故障
*   `REDISMODULE_NODE_NOFAILOVER`：从机配置为从不故障转移

<span id="RedisModule_SetClusterFlags"></span>

### `RedisModule_SetClusterFlags`

    void RedisModule_SetClusterFlags(RedisModuleCtx *ctx, uint64_t flags);

**可用日期：**5.0.0

设置 Redis 集群标志以更改
Redis Cluster, 特别是以禁用某些功能为目标。
这对于使用群集 API 以创建的模块非常有用
不同的分布式系统, 但仍希望使用 Redis 集群
消息总线。可以设置的标志：

*   `CLUSTER_MODULE_FLAG_NO_FAILOVER`
*   `CLUSTER_MODULE_FLAG_NO_REDIRECTION`

具有以下效果：

*   `NO_FAILOVER`：防止 Redis 集群从属服务器故障转移死主服务器。
    还会禁用副本迁移功能。

*   `NO_REDIRECTION`：每个节点将接受任何密钥, 而无需尝试执行
    根据 Redis 集群算法进行分区。
    插槽信息仍将传播到
    群集, 但没有效果。

<span id="section-modules-timers-api"></span>

## 模块定时器 API

模块定时器是一种高精度的“绿色定时器”抽象, 其中
每个模块甚至可以毫无问题地注册数百万个计时器, 即使
实际的事件循环将只有一个计时器, 用于唤醒
模块计时器子系统, 以便处理下一个事件。

所有计时器都存储在一个基数树中, 按过期时间排序, 当
主Redis事件循环计时器回调被调用, 我们尝试处理所有
计时器已经一个接一个地过期。然后我们重新进入活动
循环注册一个计时器, 该计时器将在下一个进程模块时过期
计时器将过期。

每当活动计时器列表降至零时, 我们都会取消注册
主事件循环计时器, 因此当此类功能
未使用。

<span id="RedisModule_CreateTimer"></span>

### `RedisModule_CreateTimer`

    RedisModuleTimerID RedisModule_CreateTimer(RedisModuleCtx *ctx,
                                               mstime_t period,
                                               RedisModuleTimerProc callback,
                                               void *data);

**可用日期：**5.0.0

创建一个新的计时器, 该计时器将在以下时间后触发`period`毫秒, 并将调用
使用指定的函数`data`作为参数。返回的计时器 ID 可以是
用于从计时器获取信息或在计时器触发之前停止计时器。
请注意, 对于重复计时器的常见用例 (重新注册
的计时器在`RedisModuleTimerProc`回调) 当
此 API 称为：
如果在“回调”的开头调用它, 则意味着
该事件将在每个“周期”触发。
如果在“回调”的末尾调用它, 则意味着
事件之间将有“周期”毫秒级的间隔。
 (如果执行“回调”所需的时间可以忽略不计, 则两者
上面的语句意味着相同) 

<span id="RedisModule_StopTimer"></span>

### `RedisModule_StopTimer`

    int RedisModule_StopTimer(RedisModuleCtx *ctx,
                              RedisModuleTimerID id,
                              void **data);

**可用日期：**5.0.0

停止计时器, 返回`REDISMODULE_OK`如果计时器被找到, 则属于
调用模块, 并且已停止, 否则`REDISMODULE_ERR`返回。
如果不是 NULL, 则在以下情况下, 数据指针设置为数据参数的值
计时器已创建。

<span id="RedisModule_GetTimerInfo"></span>

### `RedisModule_GetTimerInfo`

    int RedisModule_GetTimerInfo(RedisModuleCtx *ctx,
                                 RedisModuleTimerID id,
                                 uint64_t *remaining,
                                 void **data);

**可用日期：**5.0.0

获取有关计时器的信息：触发前的剩余时间
 (以毫秒为单位), , 以及与计时器关联的私有数据指针。
如果指定的计时器不存在或属于其他模块
不返回任何信息, 函数返回`REDISMODULE_ERR`否则
`REDISMODULE_OK`返回。如果出现以下情况, 则剩余的参数或数据可以为 NULL：
调用方不需要某些信息。

<span id="section-modules-eventloop-api"></span>

## Modules EventLoop API

<span id="RedisModule_EventLoopAdd"></span>

### `RedisModule_EventLoopAdd`

    int RedisModule_EventLoopAdd(int fd,
                                 int mask,
                                 RedisModuleEventLoopFunc func,
                                 void *user_data);

**可用日期：**7.0.0

将管道/套接字事件添加到事件循环。

*   `mask`必须是下列值之一：

    *   `REDISMODULE_EVENTLOOP_READABLE`
    *   `REDISMODULE_EVENTLOOP_WRITABLE`
    *   `REDISMODULE_EVENTLOOP_READABLE | REDISMODULE_EVENTLOOP_WRITABLE`

关于成功`REDISMODULE_OK`返回, 否则
`REDISMODULE_ERR`返回, 并将 errno 设置为以下值：

*   ERANGE：`fd`为负数或高于`maxclients`Redis config.
*   EINVAL：`callback`为空或`mask`值无效。

`errno`如果发生内部错误, 可能会采用其他值。

例：

    void onReadable(int fd, void *user_data, int mask) {
        char buf[32];
        int bytes = read(fd,buf,sizeof(buf));
        printf("Read %d bytes \n", bytes);
    }
    RedisModule_EventLoopAdd(fd, REDISMODULE_EVENTLOOP_READABLE, onReadable, NULL);

<span id="RedisModule_EventLoopDel"></span>

### `RedisModule_EventLoopDel`

    int RedisModule_EventLoopDel(int fd, int mask);

**可用日期：**7.0.0

从事件循环中删除管道/套接字事件。

*   `mask`必须是下列值之一：

    *   `REDISMODULE_EVENTLOOP_READABLE`
    *   `REDISMODULE_EVENTLOOP_WRITABLE`
    *   `REDISMODULE_EVENTLOOP_READABLE | REDISMODULE_EVENTLOOP_WRITABLE`

关于成功`REDISMODULE_OK`返回, 否则
`REDISMODULE_ERR`返回, 并将 errno 设置为以下值：

*   ERANGE：`fd`为负数或高于`maxclients`Redis config.
*   EINVAL：`mask`值无效。

<span id="RedisModule_EventLoopAddOneShot"></span>

### `RedisModule_EventLoopAddOneShot`

    int RedisModule_EventLoopAddOneShot(RedisModuleEventLoopOneShotFunc func,
                                        void *user_data);

**可用日期：**7.0.0

可以从其他线程调用此函数以触发 Redis 上的回调
主线程。关于成功`REDISMODULE_OK`返回。如果`func`为空
`REDISMODULE_ERR`返回, 并将 errno 设置为 EINVAL。

<span id="section-modules-acl-api"></span>

## 模块 ACL API

在 Redis 中的身份验证和授权中实现挂钩。

<span id="RedisModule_CreateModuleUser"></span>

### `RedisModule_CreateModuleUser`

    RedisModuleUser *RedisModule_CreateModuleUser(const char *name);

**可用日期：**6.0.0

创建模块可用于对客户端进行身份验证的 Redis ACL 用户。
获取用户后, 模块应设置该用户可以执行的操作
使用`RedisModule_SetUserACL()`功能。配置后, 用户
可用于对连接进行身份验证, 具有指定的
ACL 规则, 使用`RedisModule_AuthClientWithUser()`功能。

请注意：

*   此处创建的用户未由 ACL 命令列出。
*   此处创建的用户不会检查是否存在重复的名称, 因此
    调用此函数的模块, 以处理不创建用户的问题
    具有相同的名称。
*   创建的用户可用于对多个 Redis 连接进行身份验证。

调用方以后可以使用该函数释放用户
[`RedisModule_FreeModuleUser()`](#RedisModule_FreeModuleUser).调用此函数时, 如果有
客户端仍使用此用户进行身份验证, 它们已断开连接。
释放用户的函数只应在调用方真正使用时使用
想要使用户无效以定义具有不同的新用户
能力。

<span id="RedisModule_FreeModuleUser"></span>

### `RedisModule_FreeModuleUser`

    int RedisModule_FreeModuleUser(RedisModuleUser *user);

**可用日期：**6.0.0

释放给定用户并断开所有已
已通过身份验证。看[`RedisModule_CreateModuleUser`](#RedisModule_CreateModuleUser)了解详细用法。

<span id="RedisModule_SetModuleUserACL"></span>

### `RedisModule_SetModuleUserACL`

    int RedisModule_SetModuleUserACL(RedisModuleUser *user, const char* acl);

**可用日期：**6.0.0

设置通过 redis 模块创建的用户的权限
接口。语法与 ACL SETUSER 相同, 因此请参考
acl.c 中的文档以获取更多信息。看[`RedisModule_CreateModuleUser`](#RedisModule_CreateModuleUser)
了解详细用法。

返回`REDISMODULE_OK`关于成功和`REDISMODULE_ERR`失败时
并将设置一个描述操作失败原因的错误。

<span id="RedisModule_GetCurrentUserName"></span>

### `RedisModule_GetCurrentUserName`

    RedisModuleString *RedisModule_GetCurrentUserName(RedisModuleCtx *ctx);

**可用日期：**7.0.0

检索当前上下文后面的客户端连接的用户名。
用户名可以稍后使用, 以便获得`RedisModuleUser`.
有关详细信息, 请参阅[`RedisModule_GetModuleUserFromUserName`](#RedisModule_GetModuleUserFromUserName).

返回的字符串必须随[`RedisModule_FreeString()`](#RedisModule_FreeString)或由
启用自动内存管理。

<span id="RedisModule_GetModuleUserFromUserName"></span>

### `RedisModule_GetModuleUserFromUserName`

    RedisModuleUser *RedisModule_GetModuleUserFromUserName(RedisModuleString *name);

**可用日期：**7.0.0

一个`RedisModuleUser`可用于检查是否可以执行命令、键或通道或
根据与该用户关联的 ACL 规则进行访问。
当模块要对常规 ACL 用户 (不是由[`RedisModule_CreateModuleUser`](#RedisModule_CreateModuleUser)),
它可以得到`RedisModuleUser`从此 API, 基于检索到的用户名[`RedisModule_GetCurrentUserName`](#RedisModule_GetCurrentUserName).

由于可以随时删除一般 ACL 用户, 因此`RedisModuleUser`应仅在上下文中使用
调用此函数的位置。为了在该上下文中执行 ACL 检查, 模块可以存储用户名, 
并在任何其他上下文中调用此 API。

如果用户被禁用或用户不存在, 则返回 NULL。
调用方稍后应使用该函数释放用户[`RedisModule_FreeModuleUser()`](#RedisModule_FreeModuleUser).

<span id="RedisModule_ACLCheckCommandPermissions"></span>

### `RedisModule_ACLCheckCommandPermissions`

    int RedisModule_ACLCheckCommandPermissions(RedisModuleUser *user,
                                               RedisModuleString **argv,
                                               int argc);

**可用日期：**7.0.0

根据与之关联的 ACL 检查用户是否可以执行该命令。

关于成功`REDISMODULE_OK`返回, 否则
`REDISMODULE_ERR`返回, 并将 errno 设置为以下值：

*   ENOENT：指定的命令不存在。
*   EACCES：根据 ACL 规则, 无法执行命令

<span id="RedisModule_ACLCheckKeyPermissions"></span>

### `RedisModule_ACLCheckKeyPermissions`

    int RedisModule_ACLCheckKeyPermissions(RedisModuleUser *user,
                                           RedisModuleString *key,
                                           int flags);

**可用日期：**7.0.0

检查用户是否可以根据附加到用户的 ACL 访问密钥
和表示密钥访问的标志。这些标志与
逻辑操作的键规范。这些标志记录在[`RedisModule_SetCommandInfo`](#RedisModule_SetCommandInfo)如
这`REDISMODULE_CMD_KEY_ACCESS`,`REDISMODULE_CMD_KEY_UPDATE`,`REDISMODULE_CMD_KEY_INSERT`,
和`REDISMODULE_CMD_KEY_DELETE`标志。

如果未提供任何标志, 则用户仍需要对
此命令成功返回。

如果用户能够访问密钥, 则`REDISMODULE_OK`返回, 否则
`REDISMODULE_ERR`返回, 并将 errno 设置为以下值之一：

*   EINVAL：提供的标志无效。
*   EACCESS：用户无权访问密钥。

<span id="RedisModule_ACLCheckChannelPermissions"></span>

### `RedisModule_ACLCheckChannelPermissions`

    int RedisModule_ACLCheckChannelPermissions(RedisModuleUser *user,
                                               RedisModuleString *ch,
                                               int flags);

**可用日期：**7.0.0

检查用户是否可以根据给定的给定内容访问 pubsub 频道
访问标志。看[`RedisModule_ChannelAtPosWithFlags`](#RedisModule_ChannelAtPosWithFlags)有关
可以传入的可能标志。

如果用户能够访问 pubsub 频道, 则`REDISMODULE_OK`返回, 否则
`REDISMODULE_ERR`返回, 并将 errno 设置为以下值之一：

*   EINVAL：提供的标志无效。
*   EACCESS：用户无权访问 pubsub 频道。

<span id="RedisModule_ACLAddLogEntry"></span>

### `RedisModule_ACLAddLogEntry`

    int RedisModule_ACLAddLogEntry(RedisModuleCtx *ctx,
                                   RedisModuleUser *user,
                                   RedisModuleString *object,
                                   RedisModuleACLLogEntryReason reason);

**可用日期：**7.0.0

在 ACL 日志中添加一个新条目。
返回`REDISMODULE_OK`关于成功和`REDISMODULE_ERR`出错时。

有关 ACL 日志的更多信息, 请参阅<https://redis.io/commands/acl-log>

<span id="RedisModule_AuthenticateClientWithUser"></span>

### `RedisModule_AuthenticateClientWithUser`

    int RedisModule_AuthenticateClientWithUser(RedisModuleCtx *ctx,
                                               RedisModuleUser *module_user,
                                               RedisModuleUserChangedFunc callback,
                                               void *privdata,
                                               uint64_t *client_id);

**可用日期：**6.0.0

使用提供的 redis acl 用户对当前上下文的用户进行身份验证。
返回`REDISMODULE_ERR`如果用户被禁用。

有关回调的信息, 请参阅身份验证客户端使用用户, `client_id`,
和身份验证的一般用法。

<span id="RedisModule_AuthenticateClientWithACLUser"></span>

### `RedisModule_AuthenticateClientWithACLUser`

    int RedisModule_AuthenticateClientWithACLUser(RedisModuleCtx *ctx,
                                                  const char *name,
                                                  size_t len,
                                                  RedisModuleUserChangedFunc callback,
                                                  void *privdata,
                                                  uint64_t *client_id);

**可用日期：**6.0.0

使用提供的 redis acl 用户对当前上下文的用户进行身份验证。
返回`REDISMODULE_ERR`如果用户被禁用或用户不存在。

有关回调的信息, 请参阅身份验证客户端使用用户, `client_id`,
和身份验证的一般用法。

<span id="RedisModule_DeauthenticateAndCloseClient"></span>

### `RedisModule_DeauthenticateAndCloseClient`

    int RedisModule_DeauthenticateAndCloseClient(RedisModuleCtx *ctx,
                                                 uint64_t client_id);

**可用日期：**6.0.0

取消身份验证并关闭客户端。客户机资源不会
立即释放, 但将在后台作业中清理。这是
取消对客户端进行身份验证的推荐方法, 因为大多数客户端无法
处理用户变得已取消身份验证。返回`REDISMODULE_ERR`当
客户端不存在, 并且`REDISMODULE_OK`当操作成功时。

客户端 ID 从[`RedisModule_AuthenticateClientWithUser`](#RedisModule_AuthenticateClientWithUser)和
[`RedisModule_AuthenticateClientWithACLUser`](#RedisModule_AuthenticateClientWithACLUser)API, 但可以通过以下方式获得
客户端 API 或通过服务器事件。

此函数不是线程安全的, 必须在上下文中执行
命令或线程安全上下文。

<span id="RedisModule_RedactClientCommandArgument"></span>

### `RedisModule_RedactClientCommandArgument`

    int RedisModule_RedactClientCommandArgument(RedisModuleCtx *ctx, int pos);

**可用日期：**7.0.0

密文在给定位置指定的客户端命令参数。已编辑的论点
在面向用户的命令 (如 SLOWLOG 或 MONITOR) 中进行模糊处, , 以及
从不写入服务器日志。此命令可以在
相同的位置。

请注意, 命令名称位置 0 无法编校。

返回`REDISMODULE_OK`如果参数已编辑, 并且`REDISMODULE_ERR`如果有
传入的参数无效或位置位于客户端外部
参数范围。

<span id="RedisModule_GetClientCertificate"></span>

### `RedisModule_GetClientCertificate`

    RedisModuleString *RedisModule_GetClientCertificate(RedisModuleCtx *ctx,
                                                        uint64_t client_id);

**可用日期：**6.0.9

返回客户端用于进行身份验证的 X.509 客户端证书
此连接。

返回值为已分配`RedisModuleString`这是 X.509 证书
以 PEM (Base64) 格式编码。它应该由调用方释放 (或自动释放) 。

在下列情况下返回 NULL 值：

*   连接 ID 不存在
*   连接不是 TLS 连接
*   连接是 TLS 连接, 但未使用客户端证书

<span id="section-modules-dictionary-api"></span>

## 模块字典 API

实现一个排序的字典 (实际上由基数树支持) 
通常的 get / set / del / num-items API, 以及一个迭代器
能够来回走动。

<span id="RedisModule_CreateDict"></span>

### `RedisModule_CreateDict`

    RedisModuleDict *RedisModule_CreateDict(RedisModuleCtx *ctx);

**可用日期：**5.0.0

创建新词典。“ctx”指针可以是当前模块上下文
或 NULL, 具体取决于您想要的内容。请遵循以下规则：

1.  如果计划保留对此字典的引用, 请使用 NULL 上下文
    这将在您创建模块回调的时间中幸存下来。
2.  如果在创建时没有可用的上下文, 请使用 NULL 上下文
    字典 (当然...) 。
3.  但是, 如果
    字典的生存时间仅限于回调范围。在此
    如果启用, 您可以享受自动内存管理, 这将
    回收字典内存, 以及返回的字符串
    下一个 / 上一个字典迭代器调用。

<span id="RedisModule_FreeDict"></span>

### `RedisModule_FreeDict`

    void RedisModule_FreeDict(RedisModuleCtx *ctx, RedisModuleDict *d);

**可用日期：**5.0.0

免费使用 创建的字典[`RedisModule_CreateDict()`](#RedisModule_CreateDict).您需要通过
上下文指针“ctx”仅当字典是使用
上下文, 而不是传递 NULL。

<span id="RedisModule_DictSize"></span>

### `RedisModule_DictSize`

    uint64_t RedisModule_DictSize(RedisModuleDict *d);

**可用日期：**5.0.0

返回字典的大小 (键数) 。

<span id="RedisModule_DictSetC"></span>

### `RedisModule_DictSetC`

    int RedisModule_DictSetC(RedisModuleDict *d,
                             void *key,
                             size_t keylen,
                             void *ptr);

**可用日期：**5.0.0

将指定的键存储到字典中, 将其值设置为
指针“ptr”。如果成功添加了密钥, 因为它没有添加
已经存在, `REDISMODULE_OK`返回。否则, 如果密钥已
存在函数返回`REDISMODULE_ERR`.

<span id="RedisModule_DictReplaceC"></span>

### `RedisModule_DictReplaceC`

    int RedisModule_DictReplaceC(RedisModuleDict *d,
                                 void *key,
                                 size_t keylen,
                                 void *ptr);

**可用日期：**5.0.0

喜欢[`RedisModule_DictSetC()`](#RedisModule_DictSetC)但会用新的钥匙替换钥匙
值 (如果密钥已存在) 。

<span id="RedisModule_DictSet"></span>

### `RedisModule_DictSet`

    int RedisModule_DictSet(RedisModuleDict *d, RedisModuleString *key, void *ptr);

**可用日期：**5.0.0

喜欢[`RedisModule_DictSetC()`](#RedisModule_DictSetC)但把钥匙作为一个`RedisModuleString`.

<span id="RedisModule_DictReplace"></span>

### `RedisModule_DictReplace`

    int RedisModule_DictReplace(RedisModuleDict *d,
                                RedisModuleString *key,
                                void *ptr);

**可用日期：**5.0.0

喜欢[`RedisModule_DictReplaceC()`](#RedisModule_DictReplaceC)但把钥匙作为一个`RedisModuleString`.

<span id="RedisModule_DictGetC"></span>

### `RedisModule_DictGetC`

    void *RedisModule_DictGetC(RedisModuleDict *d,
                               void *key,
                               size_t keylen,
                               int *nokey);

**可用日期：**5.0.0

返回存储在指定键处的值。该函数返回 NULL
在密钥不存在的情况下, 或者如果您实际存储
在键处为空。因此, 如果“nokey”指针不是NULL, 则它将
如果键不存在, 则通过引用设置为 1;如果键不存在, 则通过引用设置为 0
存在。

<span id="RedisModule_DictGet"></span>

### `RedisModule_DictGet`

    void *RedisModule_DictGet(RedisModuleDict *d,
                              RedisModuleString *key,
                              int *nokey);

**可用日期：**5.0.0

喜欢[`RedisModule_DictGetC()`](#RedisModule_DictGetC)但把钥匙作为一个`RedisModuleString`.

<span id="RedisModule_DictDelC"></span>

### `RedisModule_DictDelC`

    int RedisModule_DictDelC(RedisModuleDict *d,
                             void *key,
                             size_t keylen,
                             void *oldval);

**可用日期：**5.0.0

从字典中删除指定的键, 返回`REDISMODULE_OK`如果
找到并删除了密钥, 或者`REDISMODULE_ERR`如果有
字典中没有这样的键。当操作成功时, 如果
'oldval' 不是 NULL, 则 '\*oldval' 被设置为存储在
键在被删除之前。使用此功能可以获得
指向值的指针 (例如, 为了释放它), , 而不
必须致电[`RedisModule_DictGet()`](#RedisModule_DictGet)在删除密钥之前。

<span id="RedisModule_DictDel"></span>

### `RedisModule_DictDel`

    int RedisModule_DictDel(RedisModuleDict *d,
                            RedisModuleString *key,
                            void *oldval);

**可用日期：**5.0.0

喜欢[`RedisModule_DictDelC()`](#RedisModule_DictDelC)但获取密钥作为`RedisModuleString`.

<span id="RedisModule_DictIteratorStartC"></span>

### `RedisModule_DictIteratorStartC`

    RedisModuleDictIter *RedisModule_DictIteratorStartC(RedisModuleDict *d,
                                                        const char *op,
                                                        void *key,
                                                        size_t keylen);

**可用日期：**5.0.0

返回一个迭代器, 设置以便从指定的
键, 通过应用运算符 “op”, 它只是一个字符串, 指定
用于查找第一个元素的比较运算符。这
可用的运算符包括：

*   `^`– 寻找第一个 (词典上较小) 键。
*   `$`– 寻找最后一个 (字典上更大) 的键。
*   `>`– 查找大于指定键的第一个元素。
*   `>=`– 查找大于或等于指定键的第一个元素。
*   `<`– 查找小于指定键的第一个元素。
*   `<=`– 查找小于或等于指定键的第一个元素。
*   `==`– 查找与指定键完全匹配的第一个元素。

请注意, 对于`^`和`$`传递的密钥未被使用, 用户可以
只需传递长度为 0 的 NULL。

如果无法根据
密钥和操作员已通过, [`RedisModule_DictNext()`](#RedisModule_DictNext)/ Prev ()  将返回
`REDISMODULE_ERR`在第一次调用时, 否则它们将生成元素。

<span id="RedisModule_DictIteratorStart"></span>

### `RedisModule_DictIteratorStart`

    RedisModuleDictIter *RedisModule_DictIteratorStart(RedisModuleDict *d,
                                                       const char *op,
                                                       RedisModuleString *key);

**可用日期：**5.0.0

完全像[`RedisModule_DictIteratorStartC`](#RedisModule_DictIteratorStartC), 但密钥作为传递
`RedisModuleString`.

<span id="RedisModule_DictIteratorStop"></span>

### `RedisModule_DictIteratorStop`

    void RedisModule_DictIteratorStop(RedisModuleDictIter *di);

**可用日期：**5.0.0

释放创建的迭代器[`RedisModule_DictIteratorStart()`](#RedisModule_DictIteratorStart).此调用
是必需的, 否则会在模块中引入内存泄漏。

<span id="RedisModule_DictIteratorReseekC"></span>

### `RedisModule_DictIteratorReseekC`

    int RedisModule_DictIteratorReseekC(RedisModuleDictIter *di,
                                        const char *op,
                                        void *key,
                                        size_t keylen);

**可用日期：**5.0.0

创建后[`RedisModule_DictIteratorStart()`](#RedisModule_DictIteratorStart), 则有可能
使用以下命令更改迭代器的当前选定元素
接口调用。基于运算符和键的结果与
函数[`RedisModule_DictIteratorStart()`](#RedisModule_DictIteratorStart), 但是在本例中
返回值只是`REDISMODULE_OK`如果找到所寻求的元素, 
或`REDISMODULE_ERR`如果无法寻求指定的
元素。您可以根据需要多次重新查找迭代器。

<span id="RedisModule_DictIteratorReseek"></span>

### `RedisModule_DictIteratorReseek`

    int RedisModule_DictIteratorReseek(RedisModuleDictIter *di,
                                       const char *op,
                                       RedisModuleString *key);

**可用日期：**5.0.0

喜欢[`RedisModule_DictIteratorReseekC()`](#RedisModule_DictIteratorReseekC)但把钥匙作为一个
`RedisModuleString`.

<span id="RedisModule_DictNextC"></span>

### `RedisModule_DictNextC`

    void *RedisModule_DictNextC(RedisModuleDictIter *di,
                                size_t *keylen,
                                void **dataptr);

**可用日期：**5.0.0

返回字典迭代器的当前项`di`和步骤
下一个元素。如果迭代器已经生成了最后一个元素, 并且在那里
没有要返回的其他元素, 则返回 NULL, 否则返回指针
提供表示键的字符串, 并且`*keylen`长度
由引用设置 (如果 keylen 不是 NULL) 。这`*dataptr, , 如果不是空值
设置为存储在返回键处的指针的值作为辅助
数据 (由[`RedisModule_DictSet`](#RedisModule_DictSet)API) 。

用法示例：

     ... create the iterator here ...
     char *key;
     void *data;
     while((key = RedisModule_DictNextC(iter,&keylen,&data)) != NULL) {
         printf("%.*s %p\n", (int)keylen, key, data);
     }

返回的指针的类型为 void, 因为有时它有意义
将其投射到`char*`有时到无符号`char*`取决于
事实上它包含或不包含二进制数据, 所以这个API结束得更多
使用舒适。

返回的指针的有效性直到下次调用
下一个/上一个迭代器步骤。此外, 指针不再有效, 一旦
迭代器被释放。

<span id="RedisModule_DictPrevC"></span>

### `RedisModule_DictPrevC`

    void *RedisModule_DictPrevC(RedisModuleDictIter *di,
                                size_t *keylen,
                                void **dataptr);

**可用日期：**5.0.0

此功能与[`RedisModule_DictNext()`](#RedisModule_DictNext)但回来后
迭代器中当前选定的元素, 它选择上一个
元素 (字典上较小) 而不是下一个元素。

<span id="RedisModule_DictNext"></span>

### `RedisModule_DictNext`

    RedisModuleString *RedisModule_DictNext(RedisModuleCtx *ctx,
                                            RedisModuleDictIter *di,
                                            void **dataptr);

**可用日期：**5.0.0

喜欢`RedisModuleNextC()`, 而不是返回内部分配的
缓冲区和键长度, 它直接返回分配的模块字符串对象
在指定的上下文“ctx”中 (可能与主
应用程序接口[`RedisModule_CreateString`](#RedisModule_CreateString)).

返回的字符串对象应在使用后手动解除分配
或者通过使用激活自动内存管理的上下文。

<span id="RedisModule_DictPrev"></span>

### `RedisModule_DictPrev`

    RedisModuleString *RedisModule_DictPrev(RedisModuleCtx *ctx,
                                            RedisModuleDictIter *di,
                                            void **dataptr);

**可用日期：**5.0.0

喜欢[`RedisModule_DictNext()`](#RedisModule_DictNext)但在返回当前选定的
元素中, 它选择前一个元素 (字典
更小), , 而不是下一个。

<span id="RedisModule_DictCompareC"></span>

### `RedisModule_DictCompareC`

    int RedisModule_DictCompareC(RedisModuleDictIter *di,
                                 const char *op,
                                 void *key,
                                 size_t keylen);

**可用日期：**5.0.0

将迭代器当前指向的元素与指定的元素进行比较
元素由 key/keylen 给出, 根据运算符 'op'  (集合
有效运算符与[`RedisModule_DictIteratorStart`](#RedisModule_DictIteratorStart)).
如果比较成功, 命令将返回`REDISMODULE_OK`
否则`REDISMODULE_ERR`返回。

当我们只想发出词典范围时, 这很有用, 因此
在循环中, 当我们迭代元素时, 我们还可以检查我们是否仍然
在范围内。

函数返回`REDISMODULE_ERR`如果迭代器到达
元素的结束条件也是如此。

<span id="RedisModule_DictCompare"></span>

### `RedisModule_DictCompare`

    int RedisModule_DictCompare(RedisModuleDictIter *di,
                                const char *op,
                                RedisModuleString *key);

**可用日期：**5.0.0

喜欢[`RedisModule_DictCompareC`](#RedisModule_DictCompareC)但获取与当前进行比较的密钥
迭代器键作为`RedisModuleString`.

<span id="section-modules-info-fields"></span>

## 模块信息字段

<span id="RedisModule_InfoAddSection"></span>

### `RedisModule_InfoAddSection`

    int RedisModule_InfoAddSection(RedisModuleInfoCtx *ctx, const char *name);

**可用日期：**6.0.0

用于在添加任何字段之前开始新节。节名称将
前缀为`<modulename>_`并且只能包含 A-Z, a-z, 0-9。
NULL 或空字符串表示默认部分 (仅`<modulename>`)  被使用。
当返回值为`REDISMODULE_ERR`, 则应该并且将跳过该部分。

<span id="RedisModule_InfoBeginDictField"></span>

### `RedisModule_InfoBeginDictField`

    int RedisModule_InfoBeginDictField(RedisModuleInfoCtx *ctx, const char *name);

**可用日期：**6.0.0

启动一个字典字段, 类似于 INFO KEYSPACE 中的字段。正常使用
`RedisModule_InfoAddField`\* 将项目添加到此字段的函数, 以及
终止于[`RedisModule_InfoEndDictField`](#RedisModule_InfoEndDictField).

<span id="RedisModule_InfoEndDictField"></span>

### `RedisModule_InfoEndDictField`

    int RedisModule_InfoEndDictField(RedisModuleInfoCtx *ctx);

**可用日期：**6.0.0

结束字典字段, 请参阅[`RedisModule_InfoBeginDictField`](#RedisModule_InfoBeginDictField)

<span id="RedisModule_InfoAddFieldString"></span>

### `RedisModule_InfoAddFieldString`

    int RedisModule_InfoAddFieldString(RedisModuleInfoCtx *ctx,
                                       const char *field,
                                       RedisModuleString *value);

**可用日期：**6.0.0

使用者`RedisModuleInfoFunc`以添加信息字段。
每个字段将自动以`<modulename>_`.
字段名称或值不得包含`\r\n`或`:`.

<span id="RedisModule_InfoAddFieldCString"></span>

### `RedisModule_InfoAddFieldCString`

    int RedisModule_InfoAddFieldCString(RedisModuleInfoCtx *ctx,
                                        const char *field,
                                        const char *value);

**可用日期：**6.0.0

看[`RedisModule_InfoAddFieldString()`](#RedisModule_InfoAddFieldString).

<span id="RedisModule_InfoAddFieldDouble"></span>

### `RedisModule_InfoAddFieldDouble`

    int RedisModule_InfoAddFieldDouble(RedisModuleInfoCtx *ctx,
                                       const char *field,
                                       double value);

**可用日期：**6.0.0

看[`RedisModule_InfoAddFieldString()`](#RedisModule_InfoAddFieldString).

<span id="RedisModule_InfoAddFieldLongLong"></span>

### `RedisModule_InfoAddFieldLongLong`

    int RedisModule_InfoAddFieldLongLong(RedisModuleInfoCtx *ctx,
                                         const char *field,
                                         long long value);

**可用日期：**6.0.0

看[`RedisModule_InfoAddFieldString()`](#RedisModule_InfoAddFieldString).

<span id="RedisModule_InfoAddFieldULongLong"></span>

### `RedisModule_InfoAddFieldULongLong`

    int RedisModule_InfoAddFieldULongLong(RedisModuleInfoCtx *ctx,
                                          const char *field,
                                          unsigned long long value);

**可用日期：**6.0.0

看[`RedisModule_InfoAddFieldString()`](#RedisModule_InfoAddFieldString).

<span id="RedisModule_RegisterInfoFunc"></span>

### `RedisModule_RegisterInfoFunc`

    int RedisModule_RegisterInfoFunc(RedisModuleCtx *ctx, RedisModuleInfoFunc cb);

**可用日期：**6.0.0

注册 INFO 命令的回调。回调应添加 INFO 字段
通过调用`RedisModule_InfoAddField*()`功能。

<span id="RedisModule_GetServerInfo"></span>

### `RedisModule_GetServerInfo`

    RedisModuleServerInfoData *RedisModule_GetServerInfo(RedisModuleCtx *ctx,
                                                         const char *section);

**可用日期：**6.0.0

获取有关服务器的信息, 类似于从
INFO 命令。此函数采用可选的“section”参数, 该参数可能
为空。返回值保存输出, 并可用于
[`RedisModule_ServerInfoGetField`](#RedisModule_ServerInfoGetField)和一样, 以获得各个字段。
完成后, 需要释放它[`RedisModule_FreeServerInfo`](#RedisModule_FreeServerInfo)或与
自动内存管理机制 (如果启用) 。

<span id="RedisModule_FreeServerInfo"></span>

### `RedisModule_FreeServerInfo`

    void RedisModule_FreeServerInfo(RedisModuleCtx *ctx,
                                    RedisModuleServerInfoData *data);

**可用日期：**6.0.0

使用[`RedisModule_GetServerInfo()`](#RedisModule_GetServerInfo).您需要通过
上下文指针“ctx”仅当字典是使用
上下文, 而不是传递 NULL。

<span id="RedisModule_ServerInfoGetField"></span>

### `RedisModule_ServerInfoGetField`

    RedisModuleString *RedisModule_ServerInfoGetField(RedisModuleCtx *ctx,
                                                      RedisModuleServerInfoData *data,
                                                      const char* field);

**可用日期：**6.0.0

从收集的数据中获取字段的值[`RedisModule_GetServerInfo()`](#RedisModule_GetServerInfo).你
仅当要使用自动内存时才需要传递上下文指针“ctx”
释放返回字符串的机制。如果
未找到字段。

<span id="RedisModule_ServerInfoGetFieldC"></span>

### `RedisModule_ServerInfoGetFieldC`

    const char *RedisModule_ServerInfoGetFieldC(RedisModuleServerInfoData *data,
                                                const char* field);

**可用日期：**6.0.0

似[`RedisModule_ServerInfoGetField`](#RedisModule_ServerInfoGetField), 但返回一个字符\*, 该字符不应被释放, 而应释放调用方。

<span id="RedisModule_ServerInfoGetFieldSigned"></span>

### `RedisModule_ServerInfoGetFieldSigned`

    long long RedisModule_ServerInfoGetFieldSigned(RedisModuleServerInfoData *data,
                                                   const char* field,
                                                   int *out_err);

**可用日期：**6.0.0

从收集的数据中获取字段的值[`RedisModule_GetServerInfo()`](#RedisModule_GetServerInfo).如果
字段未找到, 或者不是数字或超出范围, 返回值将为
0, 和可选`out_err`参数将设置为`REDISMODULE_ERR`.

<span id="RedisModule_ServerInfoGetFieldUnsigned"></span>

### `RedisModule_ServerInfoGetFieldUnsigned`

    unsigned long long RedisModule_ServerInfoGetFieldUnsigned(RedisModuleServerInfoData *data,
                                                              const char* field,
                                                              int *out_err);

**可用日期：**6.0.0

从收集的数据中获取字段的值[`RedisModule_GetServerInfo()`](#RedisModule_GetServerInfo).如果
字段未找到, 或者不是数字或超出范围, 返回值将为
0, 和可选`out_err`参数将设置为`REDISMODULE_ERR`.

<span id="RedisModule_ServerInfoGetFieldDouble"></span>

### `RedisModule_ServerInfoGetFieldDouble`

    double RedisModule_ServerInfoGetFieldDouble(RedisModuleServerInfoData *data,
                                                const char* field,
                                                int *out_err);

**可用日期：**6.0.0

从收集的数据中获取字段的值[`RedisModule_GetServerInfo()`](#RedisModule_GetServerInfo).如果
字段未找到, 或者不是双精度, 返回值将为 0, 并且
自选`out_err`参数将设置为`REDISMODULE_ERR`.

<span id="section-modules-utility-apis"></span>

## 模块实用程序 API

<span id="RedisModule_GetRandomBytes"></span>

### `RedisModule_GetRandomBytes`

    void RedisModule_GetRandomBytes(unsigned char *dst, size_t len);

**可用日期：**5.0.0

在计数器模式下使用 SHA1 返回随机字节, 并带有 /dev/urandom
初始化的种子。此功能速度很快, 因此可用于生成
许多字节对操作系统熵池没有任何影响。
目前, 此函数不是线程安全的。

<span id="RedisModule_GetRandomHexChars"></span>

### `RedisModule_GetRandomHexChars`

    void RedisModule_GetRandomHexChars(char *dst, size_t len);

**可用日期：**5.0.0

喜欢[`RedisModule_GetRandomBytes()`](#RedisModule_GetRandomBytes)而不是将字符串设置为
随机字节字符串设置为 随机字符 在
十六进制字符集 \[0-9a-f]。

<span id="section-modules-api-exporting-importing"></span>

## 模块 API 导出/导入

<span id="RedisModule_ExportSharedAPI"></span>

### `RedisModule_ExportSharedAPI`

    int RedisModule_ExportSharedAPI(RedisModuleCtx *ctx,
                                    const char *apiname,
                                    void *func);

**可用日期：**5.0.4

此函数由模块调用, 以便导出一些带有
名。其他模块可以通过调用
对称函数[`RedisModule_GetSharedAPI()`](#RedisModule_GetSharedAPI)并将返回值强制转换为
正确的函数指针。

该函数将返回`REDISMODULE_OK`如果尚未使用该名称, 
否则`REDISMODULE_ERR`将被退回, 并且没有操作将被
执行。

重要说明：apiname 参数应为具有静态的字符串文本
辈子。API 依赖于这样一个事实, 即它在
未来。

<span id="RedisModule_GetSharedAPI"></span>

### `RedisModule_GetSharedAPI`

    void *RedisModule_GetSharedAPI(RedisModuleCtx *ctx, const char *apiname);

**可用日期：**5.0.4

请求导出的 API 指针。返回值只是一个 void 指针
此函数的调用方将需要强制转换为右侧
函数指针, 因此这是模块之间的私有协定。

如果请求的 API 不可用, 则返回 NULL。因为
模块可以在不同的时间以不同的顺序加载, 这
函数调用应该放在一些模块通用API注册中
step, 每次模块尝试执行
命令需要外部 API：如果无法解析某些 API, 则
命令应返回错误。

下面是一个示例：

    int ... myCommandImplementation() {
       if (getExternalAPIs() == 0) {
            reply with an error here if we cannot have the APIs
       }
       // Use the API:
       myFunctionPointer(foo);
    }

函数 registerAPI ()  是：

    int getExternalAPIs(void) {
        static int api_loaded = 0;
        if (api_loaded != 0) return 1; // APIs already resolved.

        myFunctionPointer = RedisModule_GetOtherModuleAPI("...");
        if (myFunctionPointer == NULL) return 0;

        return 1;
    }

<span id="section-module-command-filter-api"></span>

## 模块命令筛选器 API

<span id="RedisModule_RegisterCommandFilter"></span>

### `RedisModule_RegisterCommandFilter`

    RedisModuleCommandFilter *RedisModule_RegisterCommandFilter(RedisModuleCtx *ctx,
                                                                RedisModuleCommandFilterFunc callback,
                                                                int flags);

**可用日期：**5.0.5

注册新的命令筛选器函数。

命令过滤使模块可以通过插入来扩展 Redis
进入所有命令的执行流。

在 Redis 执行之前调用已注册的筛选器*任何*命令。 这
包括核心 Redis 命令和由任何模块注册的命令。 这
过滤器适用于所有执行路径, 包括：

1.  客户端调用。
2.  调用通过[`RedisModule_Call()`](#RedisModule_Call)由任何模块。
3.  通过 Lua 调用`redis.call()`.
4.  从主服务器复制命令。

筛选器在特殊的筛选器上下文中执行, 该上下文不同且更多
限制为`RedisModuleCtx`. 由于筛选器会影响任何命令, 因此
必须以非常有效的方式实施, 以减少对性能的影响
在雷迪斯上。 所有需要有效上下文的 Redis 模块 API 调用 (例如
[`RedisModule_Call()`](#RedisModule_Call),[`RedisModule_OpenKey()`](#RedisModule_OpenKey)等) 在 中不受支持
筛选上下文。

这`RedisModuleCommandFilterCtx`可用于检查或修改
已执行命令及其参数。 当过滤器在 Redis 之前执行时
开始处理命令, 任何更改都会影响命令的方式
处理。 例如, 模块可以通过以下方式覆盖 Redis 命令：

1.  注册`MODULE.SET`命令, 该命令实现
    雷迪斯`SET`命令。
2.  注册一个命令过滤器, 用于检测`SET`在特定
    键的模式。 一旦检测到, 过滤器将替换第一个
    参数从`SET`自`MODULE.SET`.
3.  当过滤器执行完成时, Redis 会考虑新的命令名称
    因此执行模块自己的命令。

请注意, 在上面的用例中, 如果`MODULE.SET`本身用途
[`RedisModule_Call()`](#RedisModule_Call)过滤器也将应用于该调用。 如果
这是不想要的, `REDISMODULE_CMDFILTER_NOSELF`标志可以在以下情况下设置
注册筛选器。

这`REDISMODULE_CMDFILTER_NOSELF`标志阻止执行流
源自模块自己的[`RedisModule_Call()`](#RedisModule_Call)从到达过滤器。 这
标志对所有执行流 (包括嵌套流) 有, , 只要
执行从模块的命令上下文或线程安全开始
与阻止命令关联的上下文。

分离的线程安全上下文是*不*与模块关联且无法
受此标志保护。

如果注册了多个过滤器 (由相同或不同的模块), , 则它们
按注册顺序执行。

<span id="RedisModule_UnregisterCommandFilter"></span>

### `RedisModule_UnregisterCommandFilter`

    int RedisModule_UnregisterCommandFilter(RedisModuleCtx *ctx,
                                            RedisModuleCommandFilter *filter);

**可用日期：**5.0.5

注销命令筛选器。

<span id="RedisModule_CommandFilterArgsCount"></span>

### `RedisModule_CommandFilterArgsCount`

    int RedisModule_CommandFilterArgsCount(RedisModuleCommandFilterCtx *fctx);

**可用日期：**5.0.5

返回筛选命令具有的参数数。 数量
参数包括命令本身。

<span id="RedisModule_CommandFilterArgGet"></span>

### `RedisModule_CommandFilterArgGet`

    RedisModuleString *RedisModule_CommandFilterArgGet(RedisModuleCommandFilterCtx *fctx,
                                                       int pos);

**可用日期：**5.0.5

返回指定的命令参数。 第一个参数 (位置 0) 是
命令本身, 其余的都是用户提供的参数。

<span id="RedisModule_CommandFilterArgInsert"></span>

### `RedisModule_CommandFilterArgInsert`

    int RedisModule_CommandFilterArgInsert(RedisModuleCommandFilterCtx *fctx,
                                           int pos,
                                           RedisModuleString *arg);

**可用日期：**5.0.5

通过在指定的参数处插入新参数来修改筛选的命令
位置。 指定的`RedisModuleString`参数可由 Redis 使用
在过滤器上下文被销毁后, 所以它不能是自动记忆
分配、释放或在其他地方使用。

<span id="RedisModule_CommandFilterArgReplace"></span>

### `RedisModule_CommandFilterArgReplace`

    int RedisModule_CommandFilterArgReplace(RedisModuleCommandFilterCtx *fctx,
                                            int pos,
                                            RedisModuleString *arg);

**可用日期：**5.0.5

通过将现有参数替换为新参数来修改筛选的命令。
指定的`RedisModuleString`参数可以由 Redis 在
过滤器上下文被破坏, 因此它不得自动分配内存, 释放
或在其他地方使用。

<span id="RedisModule_CommandFilterArgDelete"></span>

### `RedisModule_CommandFilterArgDelete`

    int RedisModule_CommandFilterArgDelete(RedisModuleCommandFilterCtx *fctx,
                                           int pos);

**可用日期：**5.0.5

通过删除指定处的参数来修改筛选的命令
位置。

<span id="RedisModule_MallocSize"></span>

### `RedisModule_MallocSize`

    size_t RedisModule_MallocSize(void* ptr);

**可用日期：**6.0.0

对于通过[`RedisModule_Alloc()`](#RedisModule_Alloc)或
[`RedisModule_Realloc()`](#RedisModule_Realloc), 返回为其分配的内存量。
请注意, 这可能与我们分配的内存不同 (更大) 
使用分配调用, 因为有时基础分配器
将分配更多内存。

<span id="RedisModule_MallocUsableSize"></span>

### `RedisModule_MallocUsableSize`

    size_t RedisModule_MallocUsableSize(void *ptr);

**可用日期：**7.0.1

似[`RedisModule_MallocSize`](#RedisModule_MallocSize), 区别在于[`RedisModule_MallocUsableSize`](#RedisModule_MallocUsableSize)
返回模块的可用内存大小。

<span id="RedisModule_MallocSizeString"></span>

### `RedisModule_MallocSizeString`

    size_t RedisModule_MallocSizeString(RedisModuleString* str);

**可用日期：**7.0.0

与 相同[`RedisModule_MallocSize`](#RedisModule_MallocSize), 除了它适用于`RedisModuleString`指针。

<span id="RedisModule_MallocSizeDict"></span>

### `RedisModule_MallocSizeDict`

    size_t RedisModule_MallocSizeDict(RedisModuleDict* dict);

**可用日期：**7.0.0

与 相同[`RedisModule_MallocSize`](#RedisModule_MallocSize), 除了它适用于`RedisModuleDict`指针。
请注意, 返回的值只是底层结构的开销, 
它不包括键和值的分配大小。

<span id="RedisModule_GetUsedMemoryRatio"></span>

### `RedisModule_GetUsedMemoryRatio`

    float RedisModule_GetUsedMemoryRatio();

**可用日期：**6.0.0

返回一个介于 0 到 1 之间的数字, 指示内存量
目前使用的, 相对于Redis的“maxmemory”配置。

*   0 - 未配置内存限制。
*   介于 0 和 1 之间 - 在 0-1 范围内规范化使用的内存百分比。
*   正好为 1 - 已达到内存限制。
*   大于 1 - 使用的内存数超过配置的限制。

<span id="section-scanning-keyspace-and-hashes"></span>

## 扫描密钥空间和哈希

<span id="RedisModule_ScanCursorCreate"></span>

### `RedisModule_ScanCursorCreate`

    RedisModuleScanCursor *RedisModule_ScanCursorCreate();

**可用日期：**6.0.0

创建要使用的新游标[`RedisModule_Scan`](#RedisModule_Scan)

<span id="RedisModule_ScanCursorRestart"></span>

### `RedisModule_ScanCursorRestart`

    void RedisModule_ScanCursorRestart(RedisModuleScanCursor *cursor);

**可用日期：**6.0.0

重新启动现有游标。将重新扫描密钥。

<span id="RedisModule_ScanCursorDestroy"></span>

### `RedisModule_ScanCursorDestroy`

    void RedisModule_ScanCursorDestroy(RedisModuleScanCursor *cursor);

**可用日期：**6.0.0

销毁游标结构。

<span id="RedisModule_Scan"></span>

### `RedisModule_Scan`

    int RedisModule_Scan(RedisModuleCtx *ctx,
                         RedisModuleScanCursor *cursor,
                         RedisModuleScanCB fn,
                         void *privdata);

**可用日期：**6.0.0

扫描 API, 允许模块扫描 中的所有键和值
选定的数据库。

扫描实现的回调。

    void scan_callback(RedisModuleCtx *ctx, RedisModuleString *keyname,
                       RedisModuleKey *key, void *privdata);

*   `ctx`：为扫描提供的 redis 模块上下文。
*   `keyname`：归调用方所有, 在此之后使用时需要保留
    功能。
*   `key`：保存有关键和值的信息, 它是尽力提供的, 在
    在某些情况下, 它可能是 NULL, 在这种情况下, 用户应该 (可以) 使用
    [`RedisModule_OpenKey()`](#RedisModule_OpenKey) (还有 CloseKey) 。
    当它被提供时, 它由调用者拥有, 并且当
    回调返回。
*   `privdata`：提供给的用户数据[`RedisModule_Scan()`](#RedisModule_Scan).

它应该使用的方式：

     RedisModuleScanCursor *c = RedisModule_ScanCursorCreate();
     while(RedisModule_Scan(ctx, c, callback, privateData));
     RedisModule_ScanCursorDestroy(c);

也可以在锁定时从另一个线程使用此API
在实际调用期间获取[`RedisModule_Scan`](#RedisModule_Scan):

     RedisModuleScanCursor *c = RedisModule_ScanCursorCreate();
     RedisModule_ThreadSafeContextLock(ctx);
     while(RedisModule_Scan(ctx, c, callback, privateData)){
         RedisModule_ThreadSafeContextUnlock(ctx);
         // do some background job
         RedisModule_ThreadSafeContextLock(ctx);
     }
     RedisModule_ScanCursorDestroy(c);

如果有更多的元素要扫描, 该函数将返回 1, 并且
否则为 0, 如果调用失败, 则可能设置 errno。

还可以使用以下命令重新启动现有游标[`RedisModule_ScanCursorRestart`](#RedisModule_ScanCursorRestart).

重要说明：此 API 与
从它提供的保证的角度来看。这意味着 API
可能会报告重复的密钥, 但保证至少报告一次
从扫描过程开始到结束的每个密钥。

注意：如果在回调中执行数据库更改, 则应注意
数据库的内部状态可能会更改。例如, 它是安全的
删除或修改当前密钥, 但删除任何密钥可能不安全
其他键。
此外, 在迭代时玩Redis键空间可能会有
返回更多重复项的效果。一种安全的模式是存储密钥
要在其他位置修改的名称, 并对键执行操作
稍后在迭代完成时。但是, 这可能会花费很多
内存, 因此在以下情况下仅对当前键进行操作可能是有意义的
在迭代期间可能, 因为这是安全的。

<span id="RedisModule_ScanKey"></span>

### `RedisModule_ScanKey`

    int RedisModule_ScanKey(RedisModuleKey *key,
                            RedisModuleScanCursor *cursor,
                            RedisModuleScanKeyCB fn,
                            void *privdata);

**可用日期：**6.0.0

扫描 API, 允许模块扫描哈希、设置或排序的设置键中的元素

扫描实现的回调。

    void scan_callback(RedisModuleKey *key, RedisModuleString* field, RedisModuleString* value, void *privdata);

*   key - 为扫描提供的 redis 密钥上下文。
*   字段 - 字段名称, 由调用方拥有, 如果使用, 需要保留
    在此功能之后。
*   value - value string 或 NULL 表示 set 类型, 由调用方拥有, 需要
    如果在此功能之后使用, 则保留。
*   私有数据 - 提供给[`RedisModule_ScanKey`](#RedisModule_ScanKey).

它应该使用的方式：

     RedisModuleScanCursor *c = RedisModule_ScanCursorCreate();
     RedisModuleKey *key = RedisModule_OpenKey(...)
     while(RedisModule_ScanKey(key, c, callback, privateData));
     RedisModule_CloseKey(key);
     RedisModule_ScanCursorDestroy(c);

也可以从另一个线程使用此API, 同时在
实际调用[`RedisModule_ScanKey`](#RedisModule_ScanKey), 然后每次都重新打开密钥：

     RedisModuleScanCursor *c = RedisModule_ScanCursorCreate();
     RedisModule_ThreadSafeContextLock(ctx);
     RedisModuleKey *key = RedisModule_OpenKey(...)
     while(RedisModule_ScanKey(ctx, c, callback, privateData)){
         RedisModule_CloseKey(key);
         RedisModule_ThreadSafeContextUnlock(ctx);
         // do some background job
         RedisModule_ThreadSafeContextLock(ctx);
         RedisModuleKey *key = RedisModule_OpenKey(...)
     }
     RedisModule_CloseKey(key);
     RedisModule_ScanCursorDestroy(c);

如果有更多的元素要扫描, 该函数将返回 1, 否则返回 0, 
如果调用失败, 可能会设置 errno。
还可以使用以下命令重新启动现有游标[`RedisModule_ScanCursorRestart`](#RedisModule_ScanCursorRestart).

注意：迭代对象时, 某些操作是不安全的。例如
而API保证至少返回一次所有元素
从头到尾一致地存在于数据结构中
的迭代 (参见 HSCAN 和类似命令文档), , 更多
你玩元素, 你可能会得到越多的重复。通常
删除数据结构的当前元素是安全的, 同时删除
您正在迭代的密钥不安全。

<span id="section-module-fork-api"></span>

## 模块分叉 API

<span id="RedisModule_Fork"></span>

### `RedisModule_Fork`

    int RedisModule_Fork(RedisModuleForkDoneHandler cb, void *user_data);

**可用日期：**6.0.0

使用 的当前冻结快照创建后台子进程
主进程, 您可以在后台进行一些处理, 而无需
影响/冻结流量, 无需线程和 GIL 锁定。
请注意, Redis 只允许一个并发分叉。
当孩子想退出时, 它应该调用[`RedisModule_ExitFromChild`](#RedisModule_ExitFromChild).
如果父母想杀死孩子, 它应该打电话给[`RedisModule_KillForkChild`](#RedisModule_KillForkChild)
完成的处理程序回调将在父进程上执行, 当
孩子存在 (但被杀时不存在) 
返回：失败时返回 -1, 成功时父进程将获得正 PID
的子进程将得到 0。

<span id="RedisModule_SendChildHeartbeat"></span>

### `RedisModule_SendChildHeartbeat`

    void RedisModule_SendChildHeartbeat(double progress);

**可用日期：**6.2.0

建议模块偶尔从 fork 子级调用此函数, 
以便它可以向父级报告进度和COW内存, 这将是
在 INFO 中报告。
这`progress`参数应介于 0 和 1 之间, 如果不可用, 则应介于 -1 之间。

<span id="RedisModule_ExitFromChild"></span>

### `RedisModule_ExitFromChild`

    int RedisModule_ExitFromChild(int retcode);

**可用日期：**6.0.0

当您想要终止子进程时, 从子进程调用它。
retcode 将提供给在父进程上执行的 done 处理程序。

<span id="RedisModule_KillForkChild"></span>

### `RedisModule_KillForkChild`

    int RedisModule_KillForkChild(int child_pid);

**可用日期：**6.0.0

可用于从父进程中终止分叉的子进程。
`child_pid`将是 的返回值[`RedisModule_Fork`](#RedisModule_Fork).

<span id="section-server-hooks-implementation"></span>

## 服务器挂钩实现

<span id="RedisModule_SubscribeToServerEvent"></span>

### `RedisModule_SubscribeToServerEvent`

    int RedisModule_SubscribeToServerEvent(RedisModuleCtx *ctx,
                                           RedisModuleEvent event,
                                           RedisModuleEventCallback callback);

**可用日期：**6.0.0

注册以在指定的服务器事件时通过回调收到通知
发生。调用回调时将事件作为参数, 并附加一个
参数, 它是一个 void 指针, 应大小写为特定类型
这是特定于事件的 (但许多事件将只使用 NULL, 因为它们不
有其他信息要传递给回调) 。

如果回调为 NULL 并且存在以前的订阅, 则模块
将被取消订阅。如果存在以前的订阅和回调
不为空, 旧的回调将被替换为新的回调。

回调必须属于以下类型：

    int (*RedisModuleEventCallback)(RedisModuleCtx *ctx,
                                    RedisModuleEvent eid,
                                    uint64_t subevent,
                                    void *data);

“ctx”是一个正常的 Redis 模块上下文, 回调可以在
以调用其他模块 API。“开斋节”是事件本身, 这个
仅在模块订阅了多个事件的情况下有用：使用
此结构的“id”字段可以检查事件
是我们使用此回调注册的事件之一。“下属”字段
取决于触发的事件。

最后, 可以填充“data”指针, 仅对于某些事件, 使用
更多相关数据。

以下是可用作“eid”和相关子事件的事件列表：

*   `RedisModuleEvent_ReplicationRoleChanged`:

    当实例从主实例切换时调用此事件
    复制或相反, 但事件是
    当复制副本仍然是副本但开始
    使用其他主控进行复制。

    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_REPLROLECHANGED_NOW_MASTER`
    *   `REDISMODULE_SUBEVENT_REPLROLECHANGED_NOW_REPLICA`

    “data”字段可以通过回调转换为
    `RedisModuleReplicationInfo`具有以下字段的结构：

          int master; // true if master, false if replica
          char *masterhost; // master instance hostname for NOW_REPLICA
          int masterport; // master instance port for NOW_REPLICA
          char *replid1; // Main replication ID
          char *replid2; // Secondary replication ID
          uint64_t repl1_offset; // Main replication offset
          uint64_t repl2_offset; // Offset of replid2 validity

*   `RedisModuleEvent_Persistence`

    当 RDB 保存或 AOF 重写开始时调用此事件
    和结束。以下子事件可用：

    *   `REDISMODULE_SUBEVENT_PERSISTENCE_RDB_START`
    *   `REDISMODULE_SUBEVENT_PERSISTENCE_AOF_START`
    *   `REDISMODULE_SUBEVENT_PERSISTENCE_SYNC_RDB_START`
    *   `REDISMODULE_SUBEVENT_PERSISTENCE_SYNC_AOF_START`
    *   `REDISMODULE_SUBEVENT_PERSISTENCE_ENDED`
    *   `REDISMODULE_SUBEVENT_PERSISTENCE_FAILED`

    上述事件不仅在用户调用
    相关命令, 如 BGSAVE, 以及保存操作时
    或 AOF 重写由于内部服务器触发器而发生。
    SYNC_RDB_START 由于
    SAVE 命令、FLUSHALL 或服务器关闭, 以及另一个 RDB 和
    AOF 子事件在后台分叉子事件中执行, 因此
    模块执行的操作只能影响生成的 AOF 或 RDB, 
    但不会在父流程中反映并影响连接
    客户端和命令。另请注意, AOF_START子事件可能会结束
    在带有rdb前导码的AOF的情况下保存RDB内容。

*   `RedisModuleEvent_FlushDB`

    FLUSHALL、FLUSHDB 或内部冲洗 (例如
    由于复制, 在副本同步之后) 
    发生。以下子事件可用：

    *   `REDISMODULE_SUBEVENT_FLUSHDB_START`
    *   `REDISMODULE_SUBEVENT_FLUSHDB_END`

    数据指针可以投射到 RedisModuleFlushInfo
    具有以下字段的结构：

          int32_t async;  // True if the flush is done in a thread.
                          // See for instance FLUSHALL ASYNC.
                          // In this case the END callback is invoked
                          // immediately after the database is put
                          // in the free list of the thread.
          int32_t dbnum;  // Flushed database number, -1 for all the DBs
                          // in the case of the FLUSHALL operation.

    启动事件称为*以前*操作已启动, 因此
    允许回调调用 DBSIZE 或其他操作
    尚未释放的密钥空间。

*   `RedisModuleEvent_Loading`

    加载操作时调用：启动时, 当服务器
    启动, 但也在第一次同步后
    副本正在从主服务器加载 RDB 文件。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_LOADING_RDB_START`
    *   `REDISMODULE_SUBEVENT_LOADING_AOF_START`
    *   `REDISMODULE_SUBEVENT_LOADING_REPL_START`
    *   `REDISMODULE_SUBEVENT_LOADING_ENDED`
    *   `REDISMODULE_SUBEVENT_LOADING_FAILED`

    请注意, 在以下情况下, AOF 加载可能从 RDB 数据开始
    rdb-preamble, 在这种情况下, 您只会收到AOF_START事件。

*   `RedisModuleEvent_ClientChange`

    在客户端连接或断开连接时调用。
    数据指针可以投射到 RedisModuleClientInfo
    结构, 记录在 RedisModule_GetClientInfoById ()  中。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_CLIENT_CHANGE_CONNECTED`
    *   `REDISMODULE_SUBEVENT_CLIENT_CHANGE_DISCONNECTED`

*   `RedisModuleEvent_Shutdown`

    服务器正在关闭。没有可用的子事件。

*   `RedisModuleEvent_ReplicaChange`

    当实例 (可以是
    主副本或副本)  获取新的联机副, , 否则丢失
    副本, 因为它已断开连接。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_REPLICA_CHANGE_ONLINE`
    *   `REDISMODULE_SUBEVENT_REPLICA_CHANGE_OFFLINE`

    到目前为止没有其他信息可用：未来版本
    的 Redis 将有一个 API 来枚举副本
    已连接及其状态。

*   `RedisModuleEvent_CronLoop`

    每次 Redis 调用 serverCron ()  时都会调用此事件
    功能为了做一定的簿记。模块
    需要做的操作时不时可以使用这个回调。
    通常, Redis 每秒调用此函数 10 次, 但
    这根据“hz”配置而变化。
    没有可用的子事件。

    数据指针可以投射到 RedisModuleCronLoop
    具有以下字段的结构：

          int32_t hz;  // Approximate number of events per second.

*   `RedisModuleEvent_MasterLinkChange`

    这是对副本调用的, 以便在
    复制链路与我们的主节点一起变为功能 (向上), , 
    或者当它下降时。请注意, 不考虑该链接
    当我们刚刚连接到主站时, 但只有当
    复制正在正确进行。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_MASTER_LINK_UP`
    *   `REDISMODULE_SUBEVENT_MASTER_LINK_DOWN`

*   `RedisModuleEvent_ModuleChange`

    当加载新模块或卸载一个模块时, 将调用此事件。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_MODULE_LOADED`
    *   `REDISMODULE_SUBEVENT_MODULE_UNLOADED`

    数据指针可以投射到 RedisModuleModuleChange
    具有以下字段的结构：

          const char* module_name;  // Name of module loaded or unloaded.
          int32_t module_version;  // Module version.

*   `RedisModuleEvent_LoadingProgress`

    在 RDB 或 AOF 文件时反复调用此事件
    正在加载。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_LOADING_PROGRESS_RDB`
    *   `REDISMODULE_SUBEVENT_LOADING_PROGRESS_AOF`

    数据指针可以投射到 RedisModuleLoadingProgress
    具有以下字段的结构：

          int32_t hz;  // Approximate number of events per second.
          int32_t progress;  // Approximate progress between 0 and 1024,
                             // or -1 if unknown.

*   `RedisModuleEvent_SwapDB`

    当 SWAPDB 命令成功时, 将调用此事件
    执行。
    对于此事件调用, 目前没有可用的子事件。

    数据指针可以投射到 RedisModuleSwapDbInfo
    具有以下字段的结构：

          int32_t dbnum_first;    // Swap Db first dbnum
          int32_t dbnum_second;   // Swap Db second dbnum

*   `RedisModuleEvent_ReplBackup`

    警告：复制备份事件自 Redis 7.0 起已弃用, 并且永远不会触发。
    请参阅RedisModuleEvent_ReplAsyncLoad以了解异步复制加载事件
    现在, 当 repl-diskless-load 设置为 swapdb 时会触发这些操作。

    当 repl-diskless-load config 设置为 swapdb 时调用, 
    而 redis 需要备份当前数据库的
    以后可以恢复。包含全局数据和
    也许对于aux_load和aux_save回调, 可能需要使用这个
    通知备份/恢复/丢弃其全局变量。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_REPL_BACKUP_CREATE`
    *   `REDISMODULE_SUBEVENT_REPL_BACKUP_RESTORE`
    *   `REDISMODULE_SUBEVENT_REPL_BACKUP_DISCARD`

*   `RedisModuleEvent_ReplAsyncLoad`

    当 repl-diskless-load 配置设置为 swapdb 和具有相同主服务器的复制时调用
    发生数据集历史记录 (匹配的复制 ID) 。
    在这种情况下, redis 在从套接字加载内存中的新数据库时提供当前数据集。
    模块必须声明它们支持此机制才能激活它, 通过
    REDISMODULE_OPTIONS_HANDLE_REPL_ASYNC_LOAD标志。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_REPL_ASYNC_LOAD_STARTED`
    *   `REDISMODULE_SUBEVENT_REPL_ASYNC_LOAD_ABORTED`
    *   `REDISMODULE_SUBEVENT_REPL_ASYNC_LOAD_COMPLETED`

*   `RedisModuleEvent_ForkChild`

    当分叉子级 (AOFRW、RDBSAVE、模块分叉等) 出生/死亡时调用
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_FORK_CHILD_BORN`
    *   `REDISMODULE_SUBEVENT_FORK_CHILD_DIED`

*   `RedisModuleEvent_EventLoop`

    在每个事件循环迭代时调用, 在事件循环进行之前调用一次
    睡觉或醒来后。
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_EVENTLOOP_BEFORE_SLEEP`
    *   `REDISMODULE_SUBEVENT_EVENTLOOP_AFTER_SLEEP`

*   `RedisModule_Event_Config`

    在发生配置事件时调用
    以下子事件可用：

    *   `REDISMODULE_SUBEVENT_CONFIG_CHANGE`

    数据指针可以投射到 RedisModuleConfigChange
    具有以下字段的结构：

          const char **config_names; // An array of C string pointers containing the
                                     // name of each modified configuration item 
          uint32_t num_changes;      // The number of elements in the config_names array

函数返回`REDISMODULE_OK`如果模块已成功订阅
对于指定的事件。如果 API 是从错误的上下文或不支持的事件调用的
则给出`REDISMODULE_ERR`返回。

<span id="RedisModule_IsSubEventSupported"></span>

### `RedisModule_IsSubEventSupported`

    int RedisModule_IsSubEventSupported(RedisModuleEvent event, int64_t subevent);

**可用日期：**6.0.9

对于给定的服务器事件和子事件, 如果
不支持 subevent, 否则不为零。

<span id="section-module-configurations-api"></span>

## 模块配置接口

<span id="RedisModule_RegisterStringConfig"></span>

### `RedisModule_RegisterStringConfig`

    int RedisModule_RegisterStringConfig(RedisModuleCtx *ctx,
                                         const char *name,
                                         const char *default_val,
                                         unsigned int flags,
                                         RedisModuleConfigGetStringFunc getfn,
                                         RedisModuleConfigSetStringFunc setfn,
                                         RedisModuleConfigApplyFunc applyfn,
                                         void *privdata);

**可用日期：**7.0.0

创建一个字符串配置, Redis 用户可以通过 Redis 配置文件与之交互, 
`CONFIG SET`,`CONFIG GET`和`CONFIG REWRITE`命令。

实际的配置值归模块所有, 并且`getfn`,`setfn`和可选
`applyfn`提供给 Redis 的回调, 以便访问或操作
价值。这`getfn`回调从模块中检索值, 而`setfn`
回调提供要存储到模块配置中的值。
可选`applyfn`回调在`CONFIG SET`命令修改了一个或
使用`setfn`回调, 可用于原子应用配置
在几个配置一起更改之后。
如果有多个配置`applyfn`由单个设置的回调`CONFIG SET`
命令, 如果它们的`applyfn`功能和`privdata`指针
是相同的, 并且回调将只运行一次。
两者`setfn`和`applyfn`如果提供的值无效, 可能会返回错误, 或者
不能使用。
该配置还为 Redis 验证的值声明了一个类型, 并且
提供给模块。配置系统提供以下类型：

*   Redis 字符串：二进制安全字符串数据。
*   枚举：在注册期间提供的有限数量的字符串标记之一。
*   数字：64 位有符号整数, 还支持最小值和最大值。
*   布尔：是或否值。

这`setfn`回调有望返回`REDISMODULE_OK`当值成功时
应用的。它也可以返回`REDISMODULE_ERR`如果无法应用该值, 并且
\*错误指针可以用`RedisModuleString`要提供给客户端的错误消息。
这`RedisModuleString`从集合回调返回后, 将由 redis 释放。

所有配置都使用名称, 类型, 默认值, 创建的私有数据进行注册
在回调中可用, 以及修改配置行为的几个标志。
名称只能包含字母数字字符或短划线。支持的标志包括：

*   `REDISMODULE_CONFIG_DEFAULT`：配置的默认标志。这将创建一个可在启动后修改的配置。
*   `REDISMODULE_CONFIG_IMMUTABLE`：此配置只能提供加载时间。
*   `REDISMODULE_CONFIG_SENSITIVE`：此配置中存储的值将从所有日志记录中进行编辑。
*   `REDISMODULE_CONFIG_HIDDEN`：名称隐藏于`CONFIG GET`具有模式匹配。
*   `REDISMODULE_CONFIG_PROTECTED`：此配置只能根据启用保护配置的值进行修改。
*   `REDISMODULE_CONFIG_DENY_LOADING`：在服务器加载数据时, 此配置不可修改。
*   `REDISMODULE_CONFIG_MEMORY`：对于数字配置, 此配置会将数据单位表示法转换为其等效字节。
*   `REDISMODULE_CONFIG_BITFLAGS`：对于枚举配置, 此配置将允许将多个条目合并为位标志。

如果未通过配置文件提供, 则在启动时使用默认值来设置值
或命令行。默认值也用于在配置重写时进行比较。

笔记：

1.  在字符串配置集上, 传递给 set 回调的字符串将在执行后释放, 并且模块必须保留它。
2.  在字符串配置获取时, 字符串将不会被使用, 并且在执行后将有效。

示例实现：

    RedisModuleString *strval;
    int adjustable = 1;
    RedisModuleString *getStringConfigCommand(const char *name, void *privdata) {
        return strval;
    }

    int setStringConfigCommand(const char *name, RedisModuleString *new, void *privdata, RedisModuleString **err) {
       if (adjustable) {
           RedisModule_Free(strval);
           RedisModule_RetainString(NULL, new);
           strval = new;
           return REDISMODULE_OK;
       }
       *err = RedisModule_CreateString(NULL, "Not adjustable.", 15);
       return REDISMODULE_ERR;
    }
    ...
    RedisModule_RegisterStringConfig(ctx, "string", NULL, REDISMODULE_CONFIG_DEFAULT, getStringConfigCommand, setStringConfigCommand, NULL, NULL);

如果注册失败, `REDISMODULE_ERR`返回并返回以下项之一
errno 设置：

*   EINVAL：提供的标志对于注册无效, 或者配置的名称包含无效字符。
*   易趣：已使用提供的配置名称。

<span id="RedisModule_RegisterBoolConfig"></span>

### `RedisModule_RegisterBoolConfig`

    int RedisModule_RegisterBoolConfig(RedisModuleCtx *ctx,
                                       const char *name,
                                       int default_val,
                                       unsigned int flags,
                                       RedisModuleConfigGetBoolFunc getfn,
                                       RedisModuleConfigSetBoolFunc setfn,
                                       RedisModuleConfigApplyFunc applyfn,
                                       void *privdata);

**可用日期：**7.0.0

创建服务器客户端可以通过
`CONFIG SET`,`CONFIG GET`和`CONFIG REWRITE`命令。看
[`RedisModule_RegisterStringConfig`](#RedisModule_RegisterStringConfig)有关配置的详细信息。

<span id="RedisModule_RegisterEnumConfig"></span>

### `RedisModule_RegisterEnumConfig`

    int RedisModule_RegisterEnumConfig(RedisModuleCtx *ctx,
                                       const char *name,
                                       int default_val,
                                       unsigned int flags,
                                       const char **enum_values,
                                       const int *int_values,
                                       int num_enum_vals,
                                       RedisModuleConfigGetEnumFunc getfn,
                                       RedisModuleConfigSetEnumFunc setfn,
                                       RedisModuleConfigApplyFunc applyfn,
                                       void *privdata);

**可用日期：**7.0.0

创建服务器客户端可以通过
`CONFIG SET`,`CONFIG GET`和`CONFIG REWRITE`命令。
枚举配置是一组对应于整数值的字符串标记, 其中
字符串值向 Redis 客户端公开, 但值传递给 Redis 和
模块 是整数值。这些值定义在`enum_values`, 数组
以空值终止的 c 字符串, 以及`int_vals`, 一个枚举值的数组, 其
索引合作伙伴`enum_values`.
示例实现：
const char \*enum_vals\[3] = {“first”,  “second”,  “third”};
const int int_vals\[3] = {0,  2,  4};
整型 enum_val = 0;

     int getEnumConfigCommand(const char *name, void *privdata) {
         return enum_val;
     }
      
     int setEnumConfigCommand(const char *name, int val, void *privdata, const char **err) {
         enum_val = val;
         return REDISMODULE_OK;
     }
     ...
     RedisModule_RegisterEnumConfig(ctx, "enum", 0, REDISMODULE_CONFIG_DEFAULT, enum_vals, int_vals, 3, getEnumConfigCommand, setEnumConfigCommand, NULL, NULL);

请注意, 您可以使用`REDISMODULE_CONFIG_BITFLAGS`以便多个枚举字符串
可以组合成一个整数作为位标志, 在这种情况下, 您可能希望
对枚举进行排序, 以便首先显示首选组合。

看[`RedisModule_RegisterStringConfig`](#RedisModule_RegisterStringConfig)有关配置的详细一般信息。

<span id="RedisModule_RegisterNumericConfig"></span>

### `RedisModule_RegisterNumericConfig`

    int RedisModule_RegisterNumericConfig(RedisModuleCtx *ctx,
                                          const char *name,
                                          long long default_val,
                                          unsigned int flags,
                                          long long min,
                                          long long max,
                                          RedisModuleConfigGetNumericFunc getfn,
                                          RedisModuleConfigSetNumericFunc setfn,
                                          RedisModuleConfigApplyFunc applyfn,
                                          void *privdata);

**可用日期：**7.0.0

创建一个整数配置, 服务器客户端可以通过
`CONFIG SET`,`CONFIG GET`和`CONFIG REWRITE`命令。看
[`RedisModule_RegisterStringConfig`](#RedisModule_RegisterStringConfig)有关配置的详细信息。

<span id="RedisModule_LoadConfigs"></span>

### `RedisModule_LoadConfigs`

    int RedisModule_LoadConfigs(RedisModuleCtx *ctx);

**可用日期：**7.0.0

在模块加载时应用所有挂起的配置。这应该被称为
在为 中的模块注册了所有配置之后`RedisModule_OnLoad`.
当在以下任一位置提供配置时, 需要调用此 API`MODULE LOADEX`
或作为启动参数提供。

<span id="section-key-eviction-api"></span>

## 密钥逐出 API

<span id="RedisModule_SetLRU"></span>

### `RedisModule_SetLRU`

    int RedisModule_SetLRU(RedisModuleKey *key, mstime_t lru_idle);

**可用日期：**6.0.0

为基于 LRU 的逐出设置密钥上次访问时间。如果
服务器的最大内存策略是基于 LFU 的。值是空闲时间 (以毫秒为单位) 。
返回`REDISMODULE_OK`如果 LRU 已更新, `REDISMODULE_ERR`否则。

<span id="RedisModule_GetLRU"></span>

### `RedisModule_GetLRU`

    int RedisModule_GetLRU(RedisModuleKey *key, mstime_t *lru_idle);

**可用日期：**6.0.0

获取上次访问时间的密钥。
如果服务器的逐出策略为
基于LFU。
返回`REDISMODULE_OK`如果密钥有效。

<span id="RedisModule_SetLFU"></span>

### `RedisModule_SetLFU`

    int RedisModule_SetLFU(RedisModuleKey *key, long long lfu_freq);

**可用日期：**6.0.0

设置密钥访问频率。仅当服务器的最大内存策略时才相关
是基于LFU的。
该频率是一个对数计数器, 用于指示
仅访问频率 (必须为 <= 255) 。
返回`REDISMODULE_OK`如果 LFU 已更新, `REDISMODULE_ERR`否则。

<span id="RedisModule_GetLFU"></span>

### `RedisModule_GetLFU`

    int RedisModule_GetLFU(RedisModuleKey *key, long long *lfu_freq);

**可用日期：**6.0.0

获取密钥访问频率, 如果服务器的逐出策略不是
基于LFU。
返回`REDISMODULE_OK`如果密钥有效。

<span id="section-miscellaneous-apis"></span>

## 其他接口

<span id="RedisModule_GetContextFlagsAll"></span>

### `RedisModule_GetContextFlagsAll`

    int RedisModule_GetContextFlagsAll();

**可用日期：**6.0.9

使用返回值返回完整的上下文标志掩码
模块可以检查是否支持一组特定的标志
由正在使用的 redis 服务器版本提供。
例：

       int supportedFlags = RedisModule_GetContextFlagsAll();
       if (supportedFlags & REDISMODULE_CTX_FLAGS_MULTI) {
             // REDISMODULE_CTX_FLAGS_MULTI is supported
       } else{
             // REDISMODULE_CTX_FLAGS_MULTI is not supported
       }

<span id="RedisModule_GetKeyspaceNotificationFlagsAll"></span>

### `RedisModule_GetKeyspaceNotificationFlagsAll`

    int RedisModule_GetKeyspaceNotificationFlagsAll();

**可用日期：**6.0.9

使用返回值返回完整的键空间通知掩码
模块可以检查是否支持一组特定的标志
由正在使用的 redis 服务器版本提供。
例：

       int supportedFlags = RedisModule_GetKeyspaceNotificationFlagsAll();
       if (supportedFlags & REDISMODULE_NOTIFY_LOADED) {
             // REDISMODULE_NOTIFY_LOADED is supported
       } else{
             // REDISMODULE_NOTIFY_LOADED is not supported
       }

<span id="RedisModule_GetServerVersion"></span>

### `RedisModule_GetServerVersion`

    int RedisModule_GetServerVersion();

**可用日期：**6.0.9

返回格式为 0x00MMmmpp 的 redis 版本。
示例 6.0.7 的返回值将0x00060007。

<span id="RedisModule_GetTypeMethodVersion"></span>

### `RedisModule_GetTypeMethodVersion`

    int RedisModule_GetTypeMethodVersion();

**可用日期：**6.2.0

返回当前 redis 服务器运行时值`REDISMODULE_TYPE_METHOD_VERSION`.
您可以在呼叫时使用它[`RedisModule_CreateDataType`](#RedisModule_CreateDataType)要知道哪些字段
`RedisModuleTypeMethods`将得到支持, 哪些将被忽略。

<span id="RedisModule_ModuleTypeReplaceValue"></span>

### `RedisModule_ModuleTypeReplaceValue`

    int RedisModule_ModuleTypeReplaceValue(RedisModuleKey *key,
                                           moduleType *mt,
                                           void *new_value,
                                           void **old_value);

**可用日期：**6.0.0

替换分配给模块类型的值。

密钥必须打开以进行写入, 具有现有值, 并具有 moduleType
与调用方指定的匹配。

与[`RedisModule_ModuleTypeSetValue()`](#RedisModule_ModuleTypeSetValue)这将释放旧值, 此函数
只需将旧值与新值交换即可。

函数返回`REDISMODULE_OK`关于成功, `REDISMODULE_ERR`错误时
如：

1.  未打开密钥进行写入。
2.  键不是模块数据类型键。
3.  键是除“mt”以外的模块数据类型。

如果`old_value`为非 NULL, 则通过引用返回旧值。

<span id="RedisModule_GetCommandKeysWithFlags"></span>

### `RedisModule_GetCommandKeysWithFlags`

    int *RedisModule_GetCommandKeysWithFlags(RedisModuleCtx *ctx,
                                             RedisModuleString **argv,
                                             int argc,
                                             int *num_keys,
                                             int **out_flags);

**可用日期：**7.0.0

对于指定的命令, 解析其参数并返回一个数组
包含所有键名参数的索引。此函数是
本质上是一种更有效的方法`COMMAND GETKEYS`.

这`out_flags`参数是可选的, 可以设置为 NULL。
当提供时, 它充满了`REDISMODULE_CMD_KEY_`匹配中的标志
具有返回数组的键索引的索引。

NULL 返回值指示指定的命令没有键, 或者
错误条件。错误条件通过设置错误来指示
如下：

*   ENOENT：指定的命令不存在。
*   EINVAL：指定的命令无效。

注意：返回的数组不是 Redis 模块对象, 因此它不会
即使使用自动内存, 也会自动释放。呼叫者
必须显式调用[`RedisModule_Free()`](#RedisModule_Free)以释放它, 与`out_flags`指针如果
使用。

<span id="RedisModule_GetCommandKeys"></span>

### `RedisModule_GetCommandKeys`

    int *RedisModule_GetCommandKeys(RedisModuleCtx *ctx,
                                    RedisModuleString **argv,
                                    int argc,
                                    int *num_keys);

**可用日期：**6.0.9

与[`RedisModule_GetCommandKeysWithFlags`](#RedisModule_GetCommandKeysWithFlags)当不需要标志时。

<span id="RedisModule_GetCurrentCommandName"></span>

### `RedisModule_GetCurrentCommandName`

    const char *RedisModule_GetCurrentCommandName(RedisModuleCtx *ctx);

**可用日期：**6.2.5

返回当前正在运行的命令的名称

<span id="section-defrag-api"></span>

## 碎片整理 API

<span id="RedisModule_RegisterDefragFunc"></span>

### `RedisModule_RegisterDefragFunc`

    int RedisModule_RegisterDefragFunc(RedisModuleCtx *ctx,
                                       RedisModuleDefragFunc cb);

**可用日期：**6.2.0

为全局数据 (即模块中的任何内容) 注册碎片整理回调
可以分配未绑定到特定数据类型的分配。

<span id="RedisModule_DefragShouldStop"></span>

### `RedisModule_DefragShouldStop`

    int RedisModule_DefragShouldStop(RedisModuleDefragCtx *ctx);

**可用日期：**6.2.0

当数据类型碎片整理回调迭代复杂结构时, 此
应定期调用函数。零 (假) 返回
表示回调可以继续其工作。非零值 (真) 
表示它应该停止。

当停止时, 回调可以使用[`RedisModule_DefragCursorSet()`](#RedisModule_DefragCursorSet)以存储其
位置, 以便以后可以使用[`RedisModule_DefragCursorGet()`](#RedisModule_DefragCursorGet)以恢复碎片整理。

当停止并且还有更多工作要做时, 回调应该
返回 1。否则, 应返回 0。

注意：模块应考虑调用此函数的频率, 
因此, 在调用之间进行小批量工作通常是有意义的。

<span id="RedisModule_DefragCursorSet"></span>

### `RedisModule_DefragCursorSet`

    int RedisModule_DefragCursorSet(RedisModuleDefragCtx *ctx,
                                    unsigned long cursor);

**可用日期：**6.2.0

存储任意游标值以供将来重复使用。

仅当出现以下情况时才应调用此函数[`RedisModule_DefragShouldStop()`](#RedisModule_DefragShouldStop)已返回非零
值, 并且碎片整理回调即将退出, 而没有完全迭代其
数据类型。

此行为保留给执行后期碎片整理的情况。晚
为实现`free_effort`回调和
返回`free_effort`大于碎片整理的值
“活动碎片整理最大扫描字段”配置指令。

较小的键, 未实现的键`free_effort`或全球
在后期碎片整理模式下不调用碎片整理回调。在这些情况下, 一个
调用此函数将返回`REDISMODULE_ERR`.

该模块可以使用游标来表示进入
模块的数据类型。模块还可以存储与游标相关的其他游标
信息在本地, 并使用光标作为标志, 指示何时
开始遍历新密钥。这是可能的, 因为API使
保证多个密钥的并发碎片整理将
不执行。

<span id="RedisModule_DefragCursorGet"></span>

### `RedisModule_DefragCursorGet`

    int RedisModule_DefragCursorGet(RedisModuleDefragCtx *ctx,
                                    unsigned long *cursor);

**可用日期：**6.2.0

获取以前存储的游标值[`RedisModule_DefragCursorSet()`](#RedisModule_DefragCursorSet).

如果未要求执行后期碎片整理操作, `REDISMODULE_ERR`将被退回, 并且
应忽略游标。看[`RedisModule_DefragCursorSet()`](#RedisModule_DefragCursorSet)有关更多详细信息
对游标进行碎片整理。

<span id="RedisModule_DefragAlloc"></span>

### `RedisModule_DefragAlloc`

    void *RedisModule_DefragAlloc(RedisModuleDefragCtx *ctx, void *ptr);

**可用日期：**6.2.0

对先前由 分配的内存分配进行碎片整理[`RedisModule_Alloc`](#RedisModule_Alloc),[`RedisModule_Calloc`](#RedisModule_Calloc)等。
碎片整理过程涉及分配新的内存块和复制
内容, 如`realloc()`.

如果不需要碎片整理, 则返回 NULL, 并且操作具有
没有其他效果。

如果返回非 NULL 值, 则调用方应改用新指针
并更新对旧指针的任何引用, 这不能
再次使用。

<span id="RedisModule_DefragRedisModuleString"></span>

### `RedisModule_DefragRedisModuleString`

    RedisModuleString *RedisModule_DefragRedisModuleString(RedisModuleDefragCtx *ctx,
                                                           RedisModuleString *str);

**可用日期：**6.2.0

对 a 进行碎片整理`RedisModuleString`先前由[`RedisModule_Alloc`](#RedisModule_Alloc),[`RedisModule_Calloc`](#RedisModule_Calloc)等。
看[`RedisModule_DefragAlloc()`](#RedisModule_DefragAlloc)有关碎片整理过程的详细信息
工程。

注意：只能对具有单个引用的字符串进行碎片整理。
通常, 这意味着字符串保留[`RedisModule_RetainString`](#RedisModule_RetainString)或[`RedisModule_HoldString`](#RedisModule_HoldString)
可能无法进行碎片整理。一个例外是命令 argvs, 如果保留
由模块, 将最终得到一个引用 (因为引用
在 Redis 端, 一旦命令回调返回, 就会被删除) 。

<span id="RedisModule_GetKeyNameFromDefragCtx"></span>

### `RedisModule_GetKeyNameFromDefragCtx`

    const RedisModuleString *RedisModule_GetKeyNameFromDefragCtx(RedisModuleDefragCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的密钥的名称。
不能保证密钥名称始终可用, 因此可能会返回 NULL。

<span id="RedisModule_GetDbIdFromDefragCtx"></span>

### `RedisModule_GetDbIdFromDefragCtx`

    int RedisModule_GetDbIdFromDefragCtx(RedisModuleDefragCtx *ctx);

**可用日期：**7.0.0

返回当前正在处理的密钥的数据库 ID。
无法保证此信息始终可用, 因此可能会返回 -1。

<span id="section-function-index"></span>

## 功能指标

*   [`RedisModule_ACLAddLogEntry`](#RedisModule_ACLAddLogEntry)
*   [`RedisModule_ACLCheckChannelPermissions`](#RedisModule_ACLCheckChannelPermissions)
*   [`RedisModule_ACLCheckCommandPermissions`](#RedisModule_ACLCheckCommandPermissions)
*   [`RedisModule_ACLCheckKeyPermissions`](#RedisModule_ACLCheckKeyPermissions)
*   [`RedisModule_AbortBlock`](#RedisModule_AbortBlock)
*   [`RedisModule_Alloc`](#RedisModule_Alloc)
*   [`RedisModule_AuthenticateClientWithACLUser`](#RedisModule_AuthenticateClientWithACLUser)
*   [`RedisModule_AuthenticateClientWithUser`](#RedisModule_AuthenticateClientWithUser)
*   [`RedisModule_AutoMemory`](#RedisModule_AutoMemory)
*   [`RedisModule_AvoidReplicaTraffic`](#RedisModule_AvoidReplicaTraffic)
*   [`RedisModule_BlockClient`](#RedisModule_BlockClient)
*   [`RedisModule_BlockClientOnKeys`](#RedisModule_BlockClientOnKeys)
*   [`RedisModule_BlockedClientDisconnected`](#RedisModule_BlockedClientDisconnected)
*   [`RedisModule_BlockedClientMeasureTimeEnd`](#RedisModule_BlockedClientMeasureTimeEnd)
*   [`RedisModule_BlockedClientMeasureTimeStart`](#RedisModule_BlockedClientMeasureTimeStart)
*   [`RedisModule_Call`](#RedisModule_Call)
*   [`RedisModule_CallReplyArrayElement`](#RedisModule_CallReplyArrayElement)
*   [`RedisModule_CallReplyAttribute`](#RedisModule_CallReplyAttribute)
*   [`RedisModule_CallReplyAttributeElement`](#RedisModule_CallReplyAttributeElement)
*   [`RedisModule_CallReplyBigNumber`](#RedisModule_CallReplyBigNumber)
*   [`RedisModule_CallReplyBool`](#RedisModule_CallReplyBool)
*   [`RedisModule_CallReplyDouble`](#RedisModule_CallReplyDouble)
*   [`RedisModule_CallReplyInteger`](#RedisModule_CallReplyInteger)
*   [`RedisModule_CallReplyLength`](#RedisModule_CallReplyLength)
*   [`RedisModule_CallReplyMapElement`](#RedisModule_CallReplyMapElement)
*   [`RedisModule_CallReplyProto`](#RedisModule_CallReplyProto)
*   [`RedisModule_CallReplySetElement`](#RedisModule_CallReplySetElement)
*   [`RedisModule_CallReplyStringPtr`](#RedisModule_CallReplyStringPtr)
*   [`RedisModule_CallReplyType`](#RedisModule_CallReplyType)
*   [`RedisModule_CallReplyVerbatim`](#RedisModule_CallReplyVerbatim)
*   [`RedisModule_Calloc`](#RedisModule_Calloc)
*   [`RedisModule_ChannelAtPosWithFlags`](#RedisModule_ChannelAtPosWithFlags)
*   [`RedisModule_CloseKey`](#RedisModule_CloseKey)
*   [`RedisModule_CommandFilterArgDelete`](#RedisModule_CommandFilterArgDelete)
*   [`RedisModule_CommandFilterArgGet`](#RedisModule_CommandFilterArgGet)
*   [`RedisModule_CommandFilterArgInsert`](#RedisModule_CommandFilterArgInsert)
*   [`RedisModule_CommandFilterArgReplace`](#RedisModule_CommandFilterArgReplace)
*   [`RedisModule_CommandFilterArgsCount`](#RedisModule_CommandFilterArgsCount)
*   [`RedisModule_CreateCommand`](#RedisModule_CreateCommand)
*   [`RedisModule_CreateDataType`](#RedisModule_CreateDataType)
*   [`RedisModule_CreateDict`](#RedisModule_CreateDict)
*   [`RedisModule_CreateModuleUser`](#RedisModule_CreateModuleUser)
*   [`RedisModule_CreateString`](#RedisModule_CreateString)
*   [`RedisModule_CreateStringFromCallReply`](#RedisModule_CreateStringFromCallReply)
*   [`RedisModule_CreateStringFromDouble`](#RedisModule_CreateStringFromDouble)
*   [`RedisModule_CreateStringFromLongDouble`](#RedisModule_CreateStringFromLongDouble)
*   [`RedisModule_CreateStringFromLongLong`](#RedisModule_CreateStringFromLongLong)
*   [`RedisModule_CreateStringFromStreamID`](#RedisModule_CreateStringFromStreamID)
*   [`RedisModule_CreateStringFromString`](#RedisModule_CreateStringFromString)
*   [`RedisModule_CreateStringFromULongLong`](#RedisModule_CreateStringFromULongLong)
*   [`RedisModule_CreateStringPrintf`](#RedisModule_CreateStringPrintf)
*   [`RedisModule_CreateSubcommand`](#RedisModule_CreateSubcommand)
*   [`RedisModule_CreateTimer`](#RedisModule_CreateTimer)
*   [`RedisModule_DbSize`](#RedisModule_DbSize)
*   [`RedisModule_DeauthenticateAndCloseClient`](#RedisModule_DeauthenticateAndCloseClient)
*   [`RedisModule_DefragAlloc`](#RedisModule_DefragAlloc)
*   [`RedisModule_DefragCursorGet`](#RedisModule_DefragCursorGet)
*   [`RedisModule_DefragCursorSet`](#RedisModule_DefragCursorSet)
*   [`RedisModule_DefragRedisModuleString`](#RedisModule_DefragRedisModuleString)
*   [`RedisModule_DefragShouldStop`](#RedisModule_DefragShouldStop)
*   [`RedisModule_DeleteKey`](#RedisModule_DeleteKey)
*   [`RedisModule_DictCompare`](#RedisModule_DictCompare)
*   [`RedisModule_DictCompareC`](#RedisModule_DictCompareC)
*   [`RedisModule_DictDel`](#RedisModule_DictDel)
*   [`RedisModule_DictDelC`](#RedisModule_DictDelC)
*   [`RedisModule_DictGet`](#RedisModule_DictGet)
*   [`RedisModule_DictGetC`](#RedisModule_DictGetC)
*   [`RedisModule_DictIteratorReseek`](#RedisModule_DictIteratorReseek)
*   [`RedisModule_DictIteratorReseekC`](#RedisModule_DictIteratorReseekC)
*   [`RedisModule_DictIteratorStart`](#RedisModule_DictIteratorStart)
*   [`RedisModule_DictIteratorStartC`](#RedisModule_DictIteratorStartC)
*   [`RedisModule_DictIteratorStop`](#RedisModule_DictIteratorStop)
*   [`RedisModule_DictNext`](#RedisModule_DictNext)
*   [`RedisModule_DictNextC`](#RedisModule_DictNextC)
*   [`RedisModule_DictPrev`](#RedisModule_DictPrev)
*   [`RedisModule_DictPrevC`](#RedisModule_DictPrevC)
*   [`RedisModule_DictReplace`](#RedisModule_DictReplace)
*   [`RedisModule_DictReplaceC`](#RedisModule_DictReplaceC)
*   [`RedisModule_DictSet`](#RedisModule_DictSet)
*   [`RedisModule_DictSetC`](#RedisModule_DictSetC)
*   [`RedisModule_DictSize`](#RedisModule_DictSize)
*   [`RedisModule_DigestAddLongLong`](#RedisModule_DigestAddLongLong)
*   [`RedisModule_DigestAddStringBuffer`](#RedisModule_DigestAddStringBuffer)
*   [`RedisModule_DigestEndSequence`](#RedisModule_DigestEndSequence)
*   [`RedisModule_EmitAOF`](#RedisModule_EmitAOF)
*   [`RedisModule_EventLoopAdd`](#RedisModule_EventLoopAdd)
*   [`RedisModule_EventLoopAddOneShot`](#RedisModule_EventLoopAddOneShot)
*   [`RedisModule_EventLoopDel`](#RedisModule_EventLoopDel)
*   [`RedisModule_ExitFromChild`](#RedisModule_ExitFromChild)
*   [`RedisModule_ExportSharedAPI`](#RedisModule_ExportSharedAPI)
*   [`RedisModule_Fork`](#RedisModule_Fork)
*   [`RedisModule_Free`](#RedisModule_Free)
*   [`RedisModule_FreeCallReply`](#RedisModule_FreeCallReply)
*   [`RedisModule_FreeClusterNodesList`](#RedisModule_FreeClusterNodesList)
*   [`RedisModule_FreeDict`](#RedisModule_FreeDict)
*   [`RedisModule_FreeModuleUser`](#RedisModule_FreeModuleUser)
*   [`RedisModule_FreeServerInfo`](#RedisModule_FreeServerInfo)
*   [`RedisModule_FreeString`](#RedisModule_FreeString)
*   [`RedisModule_FreeThreadSafeContext`](#RedisModule_FreeThreadSafeContext)
*   [`RedisModule_GetAbsExpire`](#RedisModule_GetAbsExpire)
*   [`RedisModule_GetBlockedClientHandle`](#RedisModule_GetBlockedClientHandle)
*   [`RedisModule_GetBlockedClientPrivateData`](#RedisModule_GetBlockedClientPrivateData)
*   [`RedisModule_GetBlockedClientReadyKey`](#RedisModule_GetBlockedClientReadyKey)
*   [`RedisModule_GetClientCertificate`](#RedisModule_GetClientCertificate)
*   [`RedisModule_GetClientId`](#RedisModule_GetClientId)
*   [`RedisModule_GetClientInfoById`](#RedisModule_GetClientInfoById)
*   [`RedisModule_GetClientNameById`](#RedisModule_GetClientNameById)
*   [`RedisModule_GetClientUserNameById`](#RedisModule_GetClientUserNameById)
*   [`RedisModule_GetClusterNodeInfo`](#RedisModule_GetClusterNodeInfo)
*   [`RedisModule_GetClusterNodesList`](#RedisModule_GetClusterNodesList)
*   [`RedisModule_GetClusterSize`](#RedisModule_GetClusterSize)
*   [`RedisModule_GetCommand`](#RedisModule_GetCommand)
*   [`RedisModule_GetCommandKeys`](#RedisModule_GetCommandKeys)
*   [`RedisModule_GetCommandKeysWithFlags`](#RedisModule_GetCommandKeysWithFlags)
*   [`RedisModule_GetContextFlags`](#RedisModule_GetContextFlags)
*   [`RedisModule_GetContextFlagsAll`](#RedisModule_GetContextFlagsAll)
*   [`RedisModule_GetCurrentCommandName`](#RedisModule_GetCurrentCommandName)
*   [`RedisModule_GetCurrentUserName`](#RedisModule_GetCurrentUserName)
*   [`RedisModule_GetDbIdFromDefragCtx`](#RedisModule_GetDbIdFromDefragCtx)
*   [`RedisModule_GetDbIdFromDigest`](#RedisModule_GetDbIdFromDigest)
*   [`RedisModule_GetDbIdFromIO`](#RedisModule_GetDbIdFromIO)
*   [`RedisModule_GetDbIdFromModuleKey`](#RedisModule_GetDbIdFromModuleKey)
*   [`RedisModule_GetDbIdFromOptCtx`](#RedisModule_GetDbIdFromOptCtx)
*   [`RedisModule_GetDetachedThreadSafeContext`](#RedisModule_GetDetachedThreadSafeContext)
*   [`RedisModule_GetExpire`](#RedisModule_GetExpire)
*   [`RedisModule_GetKeyNameFromDefragCtx`](#RedisModule_GetKeyNameFromDefragCtx)
*   [`RedisModule_GetKeyNameFromDigest`](#RedisModule_GetKeyNameFromDigest)
*   [`RedisModule_GetKeyNameFromIO`](#RedisModule_GetKeyNameFromIO)
*   [`RedisModule_GetKeyNameFromModuleKey`](#RedisModule_GetKeyNameFromModuleKey)
*   [`RedisModule_GetKeyNameFromOptCtx`](#RedisModule_GetKeyNameFromOptCtx)
*   [`RedisModule_GetKeyspaceNotificationFlagsAll`](#RedisModule_GetKeyspaceNotificationFlagsAll)
*   [`RedisModule_GetLFU`](#RedisModule_GetLFU)
*   [`RedisModule_GetLRU`](#RedisModule_GetLRU)
*   [`RedisModule_GetModuleUserFromUserName`](#RedisModule_GetModuleUserFromUserName)
*   [`RedisModule_GetMyClusterID`](#RedisModule_GetMyClusterID)
*   [`RedisModule_GetNotifyKeyspaceEvents`](#RedisModule_GetNotifyKeyspaceEvents)
*   [`RedisModule_GetRandomBytes`](#RedisModule_GetRandomBytes)
*   [`RedisModule_GetRandomHexChars`](#RedisModule_GetRandomHexChars)
*   [`RedisModule_GetSelectedDb`](#RedisModule_GetSelectedDb)
*   [`RedisModule_GetServerInfo`](#RedisModule_GetServerInfo)
*   [`RedisModule_GetServerVersion`](#RedisModule_GetServerVersion)
*   [`RedisModule_GetSharedAPI`](#RedisModule_GetSharedAPI)
*   [`RedisModule_GetThreadSafeContext`](#RedisModule_GetThreadSafeContext)
*   [`RedisModule_GetTimerInfo`](#RedisModule_GetTimerInfo)
*   [`RedisModule_GetToDbIdFromOptCtx`](#RedisModule_GetToDbIdFromOptCtx)
*   [`RedisModule_GetToKeyNameFromOptCtx`](#RedisModule_GetToKeyNameFromOptCtx)
*   [`RedisModule_GetTypeMethodVersion`](#RedisModule_GetTypeMethodVersion)
*   [`RedisModule_GetUsedMemoryRatio`](#RedisModule_GetUsedMemoryRatio)
*   [`RedisModule_HashGet`](#RedisModule_HashGet)
*   [`RedisModule_HashSet`](#RedisModule_HashSet)
*   [`RedisModule_HoldString`](#RedisModule_HoldString)
*   [`RedisModule_InfoAddFieldCString`](#RedisModule_InfoAddFieldCString)
*   [`RedisModule_InfoAddFieldDouble`](#RedisModule_InfoAddFieldDouble)
*   [`RedisModule_InfoAddFieldLongLong`](#RedisModule_InfoAddFieldLongLong)
*   [`RedisModule_InfoAddFieldString`](#RedisModule_InfoAddFieldString)
*   [`RedisModule_InfoAddFieldULongLong`](#RedisModule_InfoAddFieldULongLong)
*   [`RedisModule_InfoAddSection`](#RedisModule_InfoAddSection)
*   [`RedisModule_InfoBeginDictField`](#RedisModule_InfoBeginDictField)
*   [`RedisModule_InfoEndDictField`](#RedisModule_InfoEndDictField)
*   [`RedisModule_IsBlockedReplyRequest`](#RedisModule_IsBlockedReplyRequest)
*   [`RedisModule_IsBlockedTimeoutRequest`](#RedisModule_IsBlockedTimeoutRequest)
*   [`RedisModule_IsChannelsPositionRequest`](#RedisModule_IsChannelsPositionRequest)
*   [`RedisModule_IsIOError`](#RedisModule_IsIOError)
*   [`RedisModule_IsKeysPositionRequest`](#RedisModule_IsKeysPositionRequest)
*   [`RedisModule_IsModuleNameBusy`](#RedisModule_IsModuleNameBusy)
*   [`RedisModule_IsSubEventSupported`](#RedisModule_IsSubEventSupported)
*   [`RedisModule_KeyAtPos`](#RedisModule_KeyAtPos)
*   [`RedisModule_KeyAtPosWithFlags`](#RedisModule_KeyAtPosWithFlags)
*   [`RedisModule_KeyExists`](#RedisModule_KeyExists)
*   [`RedisModule_KeyType`](#RedisModule_KeyType)
*   [`RedisModule_KillForkChild`](#RedisModule_KillForkChild)
*   [`RedisModule_LatencyAddSample`](#RedisModule_LatencyAddSample)
*   [`RedisModule_ListDelete`](#RedisModule_ListDelete)
*   [`RedisModule_ListGet`](#RedisModule_ListGet)
*   [`RedisModule_ListInsert`](#RedisModule_ListInsert)
*   [`RedisModule_ListPop`](#RedisModule_ListPop)
*   [`RedisModule_ListPush`](#RedisModule_ListPush)
*   [`RedisModule_ListSet`](#RedisModule_ListSet)
*   [`RedisModule_LoadConfigs`](#RedisModule_LoadConfigs)
*   [`RedisModule_LoadDataTypeFromString`](#RedisModule_LoadDataTypeFromString)
*   [`RedisModule_LoadDataTypeFromStringEncver`](#RedisModule_LoadDataTypeFromStringEncver)
*   [`RedisModule_LoadDouble`](#RedisModule_LoadDouble)
*   [`RedisModule_LoadFloat`](#RedisModule_LoadFloat)
*   [`RedisModule_LoadLongDouble`](#RedisModule_LoadLongDouble)
*   [`RedisModule_LoadSigned`](#RedisModule_LoadSigned)
*   [`RedisModule_LoadString`](#RedisModule_LoadString)
*   [`RedisModule_LoadStringBuffer`](#RedisModule_LoadStringBuffer)
*   [`RedisModule_LoadUnsigned`](#RedisModule_LoadUnsigned)
*   [`RedisModule_Log`](#RedisModule_Log)
*   [`RedisModule_LogIOError`](#RedisModule_LogIOError)
*   [`RedisModule_MallocSize`](#RedisModule_MallocSize)
*   [`RedisModule_MallocSizeDict`](#RedisModule_MallocSizeDict)
*   [`RedisModule_MallocSizeString`](#RedisModule_MallocSizeString)
*   [`RedisModule_MallocUsableSize`](#RedisModule_MallocUsableSize)
*   [`RedisModule_Milliseconds`](#RedisModule_Milliseconds)
*   [`RedisModule_ModuleTypeGetType`](#RedisModule_ModuleTypeGetType)
*   [`RedisModule_ModuleTypeGetValue`](#RedisModule_ModuleTypeGetValue)
*   [`RedisModule_ModuleTypeReplaceValue`](#RedisModule_ModuleTypeReplaceValue)
*   [`RedisModule_ModuleTypeSetValue`](#RedisModule_ModuleTypeSetValue)
*   [`RedisModule_MonotonicMicroseconds`](#RedisModule_MonotonicMicroseconds)
*   [`RedisModule_NotifyKeyspaceEvent`](#RedisModule_NotifyKeyspaceEvent)
*   [`RedisModule_OpenKey`](#RedisModule_OpenKey)
*   [`RedisModule_PoolAlloc`](#RedisModule_PoolAlloc)
*   [`RedisModule_PublishMessage`](#RedisModule_PublishMessage)
*   [`RedisModule_PublishMessageShard`](#RedisModule_PublishMessageShard)
*   [`RedisModule_RandomKey`](#RedisModule_RandomKey)
*   [`RedisModule_Realloc`](#RedisModule_Realloc)
*   [`RedisModule_RedactClientCommandArgument`](#RedisModule_RedactClientCommandArgument)
*   [`RedisModule_RegisterBoolConfig`](#RedisModule_RegisterBoolConfig)
*   [`RedisModule_RegisterClusterMessageReceiver`](#RedisModule_RegisterClusterMessageReceiver)
*   [`RedisModule_RegisterCommandFilter`](#RedisModule_RegisterCommandFilter)
*   [`RedisModule_RegisterDefragFunc`](#RedisModule_RegisterDefragFunc)
*   [`RedisModule_RegisterEnumConfig`](#RedisModule_RegisterEnumConfig)
*   [`RedisModule_RegisterInfoFunc`](#RedisModule_RegisterInfoFunc)
*   [`RedisModule_RegisterNumericConfig`](#RedisModule_RegisterNumericConfig)
*   [`RedisModule_RegisterStringConfig`](#RedisModule_RegisterStringConfig)
*   [`RedisModule_Replicate`](#RedisModule_Replicate)
*   [`RedisModule_ReplicateVerbatim`](#RedisModule_ReplicateVerbatim)
*   [`RedisModule_ReplySetArrayLength`](#RedisModule_ReplySetArrayLength)
*   [`RedisModule_ReplySetAttributeLength`](#RedisModule_ReplySetAttributeLength)
*   [`RedisModule_ReplySetMapLength`](#RedisModule_ReplySetMapLength)
*   [`RedisModule_ReplySetSetLength`](#RedisModule_ReplySetSetLength)
*   [`RedisModule_ReplyWithArray`](#RedisModule_ReplyWithArray)
*   [`RedisModule_ReplyWithAttribute`](#RedisModule_ReplyWithAttribute)
*   [`RedisModule_ReplyWithBigNumber`](#RedisModule_ReplyWithBigNumber)
*   [`RedisModule_ReplyWithBool`](#RedisModule_ReplyWithBool)
*   [`RedisModule_ReplyWithCString`](#RedisModule_ReplyWithCString)
*   [`RedisModule_ReplyWithCallReply`](#RedisModule_ReplyWithCallReply)
*   [`RedisModule_ReplyWithDouble`](#RedisModule_ReplyWithDouble)
*   [`RedisModule_ReplyWithEmptyArray`](#RedisModule_ReplyWithEmptyArray)
*   [`RedisModule_ReplyWithEmptyString`](#RedisModule_ReplyWithEmptyString)
*   [`RedisModule_ReplyWithError`](#RedisModule_ReplyWithError)
*   [`RedisModule_ReplyWithLongDouble`](#RedisModule_ReplyWithLongDouble)
*   [`RedisModule_ReplyWithLongLong`](#RedisModule_ReplyWithLongLong)
*   [`RedisModule_ReplyWithMap`](#RedisModule_ReplyWithMap)
*   [`RedisModule_ReplyWithNull`](#RedisModule_ReplyWithNull)
*   [`RedisModule_ReplyWithNullArray`](#RedisModule_ReplyWithNullArray)
*   [`RedisModule_ReplyWithSet`](#RedisModule_ReplyWithSet)
*   [`RedisModule_ReplyWithSimpleString`](#RedisModule_ReplyWithSimpleString)
*   [`RedisModule_ReplyWithString`](#RedisModule_ReplyWithString)
*   [`RedisModule_ReplyWithStringBuffer`](#RedisModule_ReplyWithStringBuffer)
*   [`RedisModule_ReplyWithVerbatimString`](#RedisModule_ReplyWithVerbatimString)
*   [`RedisModule_ReplyWithVerbatimStringType`](#RedisModule_ReplyWithVerbatimStringType)
*   [`RedisModule_ResetDataset`](#RedisModule_ResetDataset)
*   [`RedisModule_RetainString`](#RedisModule_RetainString)
*   [`RedisModule_SaveDataTypeToString`](#RedisModule_SaveDataTypeToString)
*   [`RedisModule_SaveDouble`](#RedisModule_SaveDouble)
*   [`RedisModule_SaveFloat`](#RedisModule_SaveFloat)
*   [`RedisModule_SaveLongDouble`](#RedisModule_SaveLongDouble)
*   [`RedisModule_SaveSigned`](#RedisModule_SaveSigned)
*   [`RedisModule_SaveString`](#RedisModule_SaveString)
*   [`RedisModule_SaveStringBuffer`](#RedisModule_SaveStringBuffer)
*   [`RedisModule_SaveUnsigned`](#RedisModule_SaveUnsigned)
*   [`RedisModule_Scan`](#RedisModule_Scan)
*   [`RedisModule_ScanCursorCreate`](#RedisModule_ScanCursorCreate)
*   [`RedisModule_ScanCursorDestroy`](#RedisModule_ScanCursorDestroy)
*   [`RedisModule_ScanCursorRestart`](#RedisModule_ScanCursorRestart)
*   [`RedisModule_ScanKey`](#RedisModule_ScanKey)
*   [`RedisModule_SelectDb`](#RedisModule_SelectDb)
*   [`RedisModule_SendChildHeartbeat`](#RedisModule_SendChildHeartbeat)
*   [`RedisModule_SendClusterMessage`](#RedisModule_SendClusterMessage)
*   [`RedisModule_ServerInfoGetField`](#RedisModule_ServerInfoGetField)
*   [`RedisModule_ServerInfoGetFieldC`](#RedisModule_ServerInfoGetFieldC)
*   [`RedisModule_ServerInfoGetFieldDouble`](#RedisModule_ServerInfoGetFieldDouble)
*   [`RedisModule_ServerInfoGetFieldSigned`](#RedisModule_ServerInfoGetFieldSigned)
*   [`RedisModule_ServerInfoGetFieldUnsigned`](#RedisModule_ServerInfoGetFieldUnsigned)
*   [`RedisModule_SetAbsExpire`](#RedisModule_SetAbsExpire)
*   [`RedisModule_SetClientNameById`](#RedisModule_SetClientNameById)
*   [`RedisModule_SetClusterFlags`](#RedisModule_SetClusterFlags)
*   [`RedisModule_SetCommandInfo`](#RedisModule_SetCommandInfo)
*   [`RedisModule_SetDisconnectCallback`](#RedisModule_SetDisconnectCallback)
*   [`RedisModule_SetExpire`](#RedisModule_SetExpire)
*   [`RedisModule_SetLFU`](#RedisModule_SetLFU)
*   [`RedisModule_SetLRU`](#RedisModule_SetLRU)
*   [`RedisModule_SetModuleOptions`](#RedisModule_SetModuleOptions)
*   [`RedisModule_SetModuleUserACL`](#RedisModule_SetModuleUserACL)
*   [`RedisModule_SignalKeyAsReady`](#RedisModule_SignalKeyAsReady)
*   [`RedisModule_SignalModifiedKey`](#RedisModule_SignalModifiedKey)
*   [`RedisModule_StopTimer`](#RedisModule_StopTimer)
*   [`RedisModule_Strdup`](#RedisModule_Strdup)
*   [`RedisModule_StreamAdd`](#RedisModule_StreamAdd)
*   [`RedisModule_StreamDelete`](#RedisModule_StreamDelete)
*   [`RedisModule_StreamIteratorDelete`](#RedisModule_StreamIteratorDelete)
*   [`RedisModule_StreamIteratorNextField`](#RedisModule_StreamIteratorNextField)
*   [`RedisModule_StreamIteratorNextID`](#RedisModule_StreamIteratorNextID)
*   [`RedisModule_StreamIteratorStart`](#RedisModule_StreamIteratorStart)
*   [`RedisModule_StreamIteratorStop`](#RedisModule_StreamIteratorStop)
*   [`RedisModule_StreamTrimByID`](#RedisModule_StreamTrimByID)
*   [`RedisModule_StreamTrimByLength`](#RedisModule_StreamTrimByLength)
*   [`RedisModule_StringAppendBuffer`](#RedisModule_StringAppendBuffer)
*   [`RedisModule_StringCompare`](#RedisModule_StringCompare)
*   [`RedisModule_StringDMA`](#RedisModule_StringDMA)
*   [`RedisModule_StringPtrLen`](#RedisModule_StringPtrLen)
*   [`RedisModule_StringSet`](#RedisModule_StringSet)
*   [`RedisModule_StringToDouble`](#RedisModule_StringToDouble)
*   [`RedisModule_StringToLongDouble`](#RedisModule_StringToLongDouble)
*   [`RedisModule_StringToLongLong`](#RedisModule_StringToLongLong)
*   [`RedisModule_StringToStreamID`](#RedisModule_StringToStreamID)
*   [`RedisModule_StringToULongLong`](#RedisModule_StringToULongLong)
*   [`RedisModule_StringTruncate`](#RedisModule_StringTruncate)
*   [`RedisModule_SubscribeToKeyspaceEvents`](#RedisModule_SubscribeToKeyspaceEvents)
*   [`RedisModule_SubscribeToServerEvent`](#RedisModule_SubscribeToServerEvent)
*   [`RedisModule_ThreadSafeContextLock`](#RedisModule_ThreadSafeContextLock)
*   [`RedisModule_ThreadSafeContextTryLock`](#RedisModule_ThreadSafeContextTryLock)
*   [`RedisModule_ThreadSafeContextUnlock`](#RedisModule_ThreadSafeContextUnlock)
*   [`RedisModule_TrimStringAllocation`](#RedisModule_TrimStringAllocation)
*   [`RedisModule_TryAlloc`](#RedisModule_TryAlloc)
*   [`RedisModule_UnblockClient`](#RedisModule_UnblockClient)
*   [`RedisModule_UnlinkKey`](#RedisModule_UnlinkKey)
*   [`RedisModule_UnregisterCommandFilter`](#RedisModule_UnregisterCommandFilter)
*   [`RedisModule_ValueLength`](#RedisModule_ValueLength)
*   [`RedisModule_WrongArity`](#RedisModule_WrongArity)
*   [`RedisModule_Yield`](#RedisModule_Yield)
*   [`RedisModule_ZsetAdd`](#RedisModule_ZsetAdd)
*   [`RedisModule_ZsetFirstInLexRange`](#RedisModule_ZsetFirstInLexRange)
*   [`RedisModule_ZsetFirstInScoreRange`](#RedisModule_ZsetFirstInScoreRange)
*   [`RedisModule_ZsetIncrby`](#RedisModule_ZsetIncrby)
*   [`RedisModule_ZsetLastInLexRange`](#RedisModule_ZsetLastInLexRange)
*   [`RedisModule_ZsetLastInScoreRange`](#RedisModule_ZsetLastInScoreRange)
*   [`RedisModule_ZsetRangeCurrentElement`](#RedisModule_ZsetRangeCurrentElement)
*   [`RedisModule_ZsetRangeEndReached`](#RedisModule_ZsetRangeEndReached)
*   [`RedisModule_ZsetRangeNext`](#RedisModule_ZsetRangeNext)
*   [`RedisModule_ZsetRangePrev`](#RedisModule_ZsetRangePrev)
*   [`RedisModule_ZsetRangeStop`](#RedisModule_ZsetRangeStop)
*   [`RedisModule_ZsetRem`](#RedisModule_ZsetRem)
*   [`RedisModule_ZsetScore`](#RedisModule_ZsetScore)
*   [`RedisModule__Assert`](#RedisModule\_\_Assert)
