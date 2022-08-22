
Redis模块可以在高层次上访问Redis内置的数据结构, 
通过调用 Redis 命令, 并在低级别通过操作数据结构
径直。

通过使用这些功能, 可以在现有抽象的基础上构建新的抽象
Redis 数据结构, 或使用字符串 DMA 对模块进行编码
将数据结构转换为Redis字符串, 可以创建以下模块：
*感觉像*他们正在导出新的数据类型。但是, 对于更复杂的
问题, 这还不够, 而新的数据结构的实施
在模块内部是必需的。

我们将 Redis 模块实现新数据结构的能力称为
感觉像原生的Redis**本机类型支持**.本文档介绍
Redis 模块系统导出的 API, 用于创建新数据
结构和处理RDB文件中的序列化, 重写过程
在 AOF 中, 类型报告通过`TYPE`命令, 依此类推。

## 本机类型概述

导出本机类型的模块由以下主要部分组成：

*   实现某种新数据结构和在新数据结构上运行的命令。
*   一组回调, 用于处理：RDB 保存、RDB 加载、AOF 重写、释放与键关联的值、计算要与`DEBUG DIGEST`命令。
*   一个 9 个字符的名称, 对于每个模块本机数据类型是唯一的。
*   一种编码版本, 用于将特定于模块的数据版本保存到 RDB 文件中, 以便模块能够从 RDB 文件加载较旧的表示形式。

虽然要处理RDB加载, 保存和AOF重写乍一看可能看起来很复杂, 但模块API提供了非常高级的功能来处理所有这些, 而无需用户处理读/写错误, 因此实际上, 为Redis编写新的数据结构是一项简单的任务。

一个**非常简单**了解本机类型实现的完整示例
在 Redis 发行版中可用`/modules/hellotype.c`文件。
鼓励读者通过查看此示例来阅读文档
实现, 看看事情是如何在实践中应用的。

# 注册新数据类型

为了将新的本机类型注册到 Redis 核心中, 该模块需要
以声明将保存对数据类型的引用的全局变量。
用于注册数据类型的 API 将返回一个数据类型引用, 该引用将
存储在全局变量中。

    static RedisModuleType *MyType;
    #define MYTYPE_ENCODING_VERSION 0

    int RedisModule_OnLoad(RedisModuleCtx *ctx) {
    RedisModuleTypeMethods tm = {
        .version = REDISMODULE_TYPE_METHOD_VERSION,
        .rdb_load = MyTypeRDBLoad,
        .rdb_save = MyTypeRDBSave,
        .aof_rewrite = MyTypeAOFRewrite,
        .free = MyTypeFree
    };

        MyType = RedisModule_CreateDataType(ctx, "MyType-AZ",
    	MYTYPE_ENCODING_VERSION, &tm);
        if (MyType == NULL) return REDISMODULE_ERR;
    }

从上面的示例中可以看出, 需要一个 API 调用才能
注册新类型。但是, 许多函数指针作为
参数。有些是可选的, 有些是强制性的。以上套装
方法数量*必须*通过, 而`.digest`和`.mem_usage`是可选的
并且当前实际上不受模块内部支持, 因此对于
现在你可以忽略它们。

这`ctx`参数是我们在`OnLoad`功能。
类型`name`是字符集中的 9 个字符的名称, 其中包括
从`A-Z`,`a-z`,`0-9`, 加上下划线`_`和减号`-`字符。

请注意, **此名称必须是唯一的**对于 Redis 中的每种数据类型
生态系统, 所以要有创意, 如果它使
sense, 并尝试使用将类型名称与名称混合的约定
的模块作者, 以创建一个 9 个字符的唯一名称。

**注意：**非常重要的是, 名称正好是 9 个字符或
该类型的注册将失败。阅读更多内容以了解原因。

例如, 如果我正在构建一个*B 树*数据结构和我的名字是*安蒂雷兹*
我将调用我的类型**btree1-az**.名称, 转换为 64 位整数, 
在保存类型时存储在RDB文件中, 并且在
加载 RDB 数据是为了解析哪个模块可以加载数据。如果 Redis
找不到匹配的模块, 整数被转换回名称, 以便
为用户提供一些关于为了加载而缺少哪个模块的线索
数据。

类型名称还用作`TYPE`命令 (调用时) 
具有保存已注册类型的密钥。

这`encver`参数是模块用于存储数据的编码版本
在 RDB 文件中。例如, 我可以从编码版本0开始, 
但后来当我发布模块的2.0版本时, 我可以将编码切换到
更好的东西。新模块将使用编码版本 1 进行注册, 
因此, 当它保存新的RDB文件时, 新版本将存储在磁盘上。然而
加载 RDB 文件时, 模块`rdb_load`即使
找到不同编码版本 (和编码版本) 的数据
作为参数传递给`rdb_load`), , 以便模块仍然可以加载旧的
RDB 文件。

最后一个参数是用于将类型方法传递给
注册功能：`rdb_load`,`rdb_save`,`aof_rewrite`,`digest`和
`free`和`mem_usage`都是具有以下原型和用途的回调：

    typedef void *(*RedisModuleTypeLoadFunc)(RedisModuleIO *rdb, int encver);
    typedef void (*RedisModuleTypeSaveFunc)(RedisModuleIO *rdb, void *value);
    typedef void (*RedisModuleTypeRewriteFunc)(RedisModuleIO *aof, RedisModuleString *key, void *value);
    typedef size_t (*RedisModuleTypeMemUsageFunc)(void *value);
    typedef void (*RedisModuleTypeDigestFunc)(RedisModuleDigest *digest, void *value);
    typedef void (*RedisModuleTypeFreeFunc)(void *value);

*   `rdb_load`从 RDB 文件加载数据时调用。它以与`rdb_save`生产。
*   `rdb_save`在将数据保存到 RDB 文件时调用。
*   `aof_rewrite`在重写 AOF 时调用, 并且模块需要告诉 Redis 重新创建给定密钥内容的命令序列。
*   `digest`在以下情况下调用`DEBUG DIGEST`执行并找到持有此模块类型的键。目前尚未实现, 因此该函数将留空。
*   `mem_usage`当`MEMORY`命令要求特定键消耗的总内存, 并用于获取模块值使用的字节数。
*   `free`当具有模块本机类型的键通过以下方式删除时调用`DEL`或者以任何其他方式, 为了让模块回收与这样的值相关联的内存。

## 好吧, 但是*为什么*模块类型需要 9 个字符的名称？

呵呵, 我明白你需要明白这一点, 所以这里有一个非常具体的
解释。

当 Redis 保留到 RDB 文件时, 特定数据类型需要的模块
也要坚持下去。现在RDB文件是键值对的序列
如下所示：

    [1 byte type] [key] [a type specific value]

1 字节类型标识字符串、列表、集等。在案例中
的模块数据, 它被设置为一个特殊值`module data`, 但
当然这还不够, 我们需要链接特定信息所需的信息
值, 该值具有能够加载和处理它的特定模块类型。

因此, 当我们保存一个`type specific value`关于一个模块, 我们以它为前缀
一个 64 位整数。64 位足以存储所需的信息
为了查找可以处理该特定类型的模块, 但是
足够短, 我们可以在RDB中存储的每个模块值添加前缀
而不会使最终的 RDB 文件太大。同时, 此解决方案
在值前面加上 64 位前缀*签名*不需要做
奇怪的事情, 比如在RDB标头中定义特定于模块的列表
类型。一切都很简单。

因此, 您可以以64位存储的内容, 以便识别给定的模块
可靠的方法？好吧, 如果你构建一个由64个符号组成的字符集, 你可以
轻松存储6位的9个字符, 并且您只剩下10位, 即
用于存储*编码版本*的类型, 以便
同一类型可以在未来发展, 并提供不同和更多
RDB 文件的高效或更新的序列化格式。

因此, 在每个模块值之前存储的 64 位前缀如下所示：

    6|6|6|6|6|6|6|6|6|10

前 9 个元素是 6 位字符, 最后 10 位是
编码版本。

当RDB文件被装回时, 它读取64位值, 屏蔽最终值
10 位, 并在模块类型缓存中搜索匹配的模块。
当找到匹配的 RDB 文件值时, 调用加载 RDB 文件值的方法
以 10 位编码版本作为参数, 以便模块知道
要加载的数据布局版本 (如果可以支持多个版本) 。

现在, 关于所有这些的有趣之处在于, 如果相反, 模块类型
无法解析, 因为没有具有此签名的已加载模块, 
我们可以将64位值转换回9个字符的名称, 然后打印
用户的错误, 其中包含模块类型名称！这样她或他
立即意识到出了什么问题。

## 设置和获取密钥

在`RedisModule_OnLoad()`功能
我们还需要能够将 Redis 密钥设置为我们的本机类型。

这通常发生在将数据写入键的命令的上下文中。
本机类型API允许设置和获取模块本机数据类型的键, 
并测试给定键是否已与特定数据的值相关联
类型。

API 使用普通模块`RedisModule_OpenKey()`低级别密钥访问
接口, 以便处理这个问题。这是设置
Redis 密钥的本机类型私有数据结构：

    RedisModuleKey *key = RedisModule_OpenKey(ctx,keyname,REDISMODULE_WRITE);
    struct some_private_struct *data = createMyDataStructure();
    RedisModule_ModuleTypeSetValue(key,MyType,data);

功能`RedisModule_ModuleTypeSetValue()`与打开的钥匙手柄一起使用
用于写入, 并得到三个参数：键句柄, 对
本机类型, 如在类型注册期间获得的, 最后是`void*`
指针, 其中包含实现模块本机类型的私有数据。

请注意, Redis 对您的数据包含的内容一无所知。它将
只需按顺序调用您在方法注册期间提供的回调
以对类型执行操作。

同样, 我们可以使用以下函数从密钥中检索私有数据：

    struct some_private_struct *data;
    data = RedisModule_ModuleTypeGetValue(key);

我们还可以测试一个键, 以将我们的本机类型作为值：

    if (RedisModule_ModuleTypeGetType(key) == MyType) {
        /* ... do something ... */
    }

但是, 对于调用执行正确的操作, 我们需要检查密钥是否
为空, 如果它包含正确类型的值, 依此类推。所以
用于实现写入本机类型的命令的惯用代码
是沿着这些路线：

    RedisModuleKey *key = RedisModule_OpenKey(ctx,argv[1],
        REDISMODULE_READ|REDISMODULE_WRITE);
    int type = RedisModule_KeyType(key);
    if (type != REDISMODULE_KEYTYPE_EMPTY &&
        RedisModule_ModuleTypeGetType(key) != MyType)
    {
        return RedisModule_ReplyWithError(ctx,REDISMODULE_ERRORMSG_WRONGTYPE);
    }

然后, 如果我们成功验证密钥的类型是否错误, 并且
我们要写到它, 我们通常想要创建一个新的数据结构, 如果
键为空, 或检索对与 文件关联的值的引用
键 (如果已有一个) ：

    /* Create an empty value object if the key is currently empty. */
    struct some_private_struct *data;
    if (type == REDISMODULE_KEYTYPE_EMPTY) {
        data = createMyDataStructure();
        RedisModule_ModuleTypeSetValue(key,MyTyke,data);
    } else {
        data = RedisModule_ModuleTypeGetValue(key);
    }
    /* Do something with 'data'... */

## 免费方法

如前所述, 当 Redis 需要释放持有本机类型的密钥时
值, 它需要模块的帮助才能释放内存。这
是我们通过的原因`free`类型注册期间的回调：

    typedef void (*RedisModuleTypeFreeFunc)(void *value);

free 方法的简单实现可以是这样的, 
假设我们的数据结构由单个分配组成：

    void MyTypeFreeCallback(void *value) {
        RedisModule_Free(value);
    }

然而, 一个更真实的世界将调用一些执行更多
复杂的内存回收, 通过将空洞指针投射到某个结构
并释放构成值的所有资源。

## RDB 加载和保存方法

RDB 保存和加载回调需要创建 (并装回) 一个
磁盘上数据类型的表示形式。Redis 提供了一个高级 API
可以自动将以下类型存储在 RDB 文件中：

*   无符号 64 位整数。
*   有符号 64 位整数。
*   双打。
*   字符串。

由模块决定使用上述基础找到可行的表示形式
类型。但请注意, 虽然整数和双精度值是存储的
并加载到体系结构中, 并且*字节序*不可知的方式, 如果你使用
保存API的原始字符串, 例如, 在磁盘上保存结构, 您
必须自己关心这些细节。

以下是执行 RDB 保存和加载的函数列表：

    void RedisModule_SaveUnsigned(RedisModuleIO *io, uint64_t value);
    uint64_t RedisModule_LoadUnsigned(RedisModuleIO *io);
    void RedisModule_SaveSigned(RedisModuleIO *io, int64_t value);
    int64_t RedisModule_LoadSigned(RedisModuleIO *io);
    void RedisModule_SaveString(RedisModuleIO *io, RedisModuleString *s);
    void RedisModule_SaveStringBuffer(RedisModuleIO *io, const char *str, size_t len);
    RedisModuleString *RedisModule_LoadString(RedisModuleIO *io);
    char *RedisModule_LoadStringBuffer(RedisModuleIO *io, size_t *lenptr);
    void RedisModule_SaveDouble(RedisModuleIO *io, double value);
    double RedisModule_LoadDouble(RedisModuleIO *io);

这些函数不需要从模块进行任何错误检查, 这可以
始终假定调用成功。

例如, 假设我有一个本机类型, 它实现了
双精度值, 具有以下结构：

    struct double_array {
        size_t count;
        double *values;
    };

我`rdb_save`方法可能如下所示：

    void DoubleArrayRDBSave(RedisModuleIO *io, void *ptr) {
        struct dobule_array *da = ptr;
        RedisModule_SaveUnsigned(io,da->count);
        for (size_t j = 0; j < da->count; j++)
            RedisModule_SaveDouble(io,da->values[j]);
    }

我们所做的是存储元素的数量, 后跟每个双精度
价值。因此, 稍后我们将不得不将结构加载到`rdb_load`
方法我们将执行如下操作：

    void *DoubleArrayRDBLoad(RedisModuleIO *io, int encver) {
        if (encver != DOUBLE_ARRAY_ENC_VER) {
            /* We should actually log an error here, or try to implement
               the ability to load older versions of our data structure. */
            return NULL;
        }

        struct double_array *da;
        da = RedisModule_Alloc(sizeof(*da));
        da->count = RedisModule_LoadUnsigned(io);
        da->values = RedisModule_Alloc(da->count * sizeof(double));
        for (size_t j = 0; j < da->count; j++)
            da->values[j] = RedisModule_LoadDouble(io);
        return da;
    }

加载回调只是从数据中重建数据结构
我们存储在RDB文件中。

请注意, 虽然在写入和读取的API上没有错误处理
从磁盘, 加载回调仍然可以在错误时返回NULL, 以防万一
它读起来不正确。在这种情况下, Redis只会恐慌。

## AOF 重写

    void RedisModule_EmitAOF(RedisModuleIO *io, const char *cmdname, const char *fmt, ...);

## 处理多种编码

    WORK IN PROGRESS

## 分配内存

数据类型应尝试使用的模块`RedisModule_Alloc()`函数系列
以便分配、重新分配和释放用于实现本机数据结构的堆内存 (有关详细信息, 请参阅其他 Redis 模块文档) 。

这不仅对于Redis能够考虑模块使用的内存很有用, 而且还具有更多优点：

*   Redis 使用`jemalloc`分配器, 通常可防止使用 libc 分配器可能导致的碎片问题。
*   从 RDB 文件加载字符串时, 本机类型 API 能够返回直接分配的字符串`RedisModule_Alloc()`, 使模块可以直接将此存储器链接到数据结构表示中, 避免了无用的数据副本。

即使您使用外部库来实现数据结构, 
模块 API 提供的分配函数与
`malloc()`,`realloc()`,`free()`和`strdup()`, 因此转换库
为了使用这些功能应该是微不足道的。

如果您有一个使用 libc 的外部库`malloc()`, 并且您想要
以避免手动将所有调用替换为 Redis Modules API 调用, 
一种方法可能是使用简单的宏来替换libc调用
使用 Redis API 调用。像这样的东西可以工作：

    #define malloc RedisModule_Alloc
    #define realloc RedisModule_Realloc
    #define free RedisModule_Free
    #define strdup RedisModule_Strdup

但是请记住, 将libc调用与Redis API调用混合在一起将导致
陷入麻烦和崩溃, 所以如果你使用宏替换调用, 你需要
确保所有调用都已正确替换, 并且代码
例如, 替换的调用永远不会尝试调用
`RedisModule_Free()`使用 libc 分配的指针`malloc()`.
