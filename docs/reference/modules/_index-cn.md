***

## 标题： “Redis modules API”&#xA;linkTitle： “Modules API”&#xA;体重： 1&#xA;描述： >&#xA;编写 Redis 模块简介&#xA;别名：&#xA;\- /主题/模块介绍

模块文档由以下页面组成：

*   Redis 模块简介（此文件）。关于 Redis Modules 系统和 API 的概述。在这里开始阅读是个好主意。
*   [实现本机数据类型](/topics/modules-native-types)涵盖了将本机数据类型实现到模块中。
*   [阻止操作](/topics/modules-blocking-ops)演示如何编写不会立即回复但会阻止客户端而不阻止 Redis 服务器的阻止命令，并尽可能提供回复。
*   [Redis modules API 參考](/topics/modules-api-ref)从 Module.c 生成的 RedisModule 函数的顶部注释。这是一个很好的参考，以便了解每个函数的工作原理。

Redis 模块使得使用外部设备扩展 Redis 功能成为可能
模块，快速实现具有功能的新 Redis 命令
类似于核心本身内部可以做的事情。

Redis 模块是动态库，可以加载到 Redis 中
启动，或使用`MODULE LOAD`命令。Redis 导出了一个 C API，在
单个 C 头文件的形式，称为`redismodule.h`.模块是指
用C语言编写，但是可以使用C++或其他语言
具有 C 绑定功能。

模块的设计是为了加载到不同版本的Redis中，
因此，给定的模块不需要设计或重新编译，以便
使用特定版本的 Redis 运行。因此，该模块将
使用特定 API 版本注册到 Redis 核心。当前接口
版本为“1”。

本文档介绍 Redis 模块的 alpha 版本。原料药，功能
其他细节将来可能会发生变化。

## 加载模块

为了测试您正在开发的模块，您可以加载该模块
使用以下`redis.conf`配置指令：

    loadmodule /path/to/mymodule.so

也可以使用以下命令在运行时加载模块：

    MODULE LOAD /path/to/mymodule.so

要列出所有已加载的模块，请使用：

    MODULE LIST

最后，您可以使用
以下命令：

    MODULE UNLOAD mymodule

请注意，`mymodule`上面不是没有`.so`后缀，但
而是模块用于将自身注册到 Redis 核心中的名称。
该名称可以使用以下命令获取`MODULE LIST`.但是，这是很好的做法
动态库的文件名与模块的名称相同
用于将自身注册到 Redis 核心中。

## 您可以编写的最简单的模块

为了展示一个模块的不同部分，这里我们展示一个非常
实现输出随机数的命令的简单模块。

    #include "redismodule.h"
    #include <stdlib.h>

    int HelloworldRand_RedisCommand(RedisModuleCtx *ctx, RedisModuleString **argv, int argc) {
        RedisModule_ReplyWithLongLong(ctx,rand());
        return REDISMODULE_OK;
    }

    int RedisModule_OnLoad(RedisModuleCtx *ctx, RedisModuleString **argv, int argc) {
        if (RedisModule_Init(ctx,"helloworld",1,REDISMODULE_APIVER_1)
            == REDISMODULE_ERR) return REDISMODULE_ERR;

        if (RedisModule_CreateCommand(ctx,"helloworld.rand",
            HelloworldRand_RedisCommand, "fast random",
            0, 0, 0) == REDISMODULE_ERR)
            return REDISMODULE_ERR;

        return REDISMODULE_OK;
    }

示例模块有两个函数。一个实现一个命令，称为
你好世界。兰特。此函数是该模块特有的。但是
其他函数调用`RedisModule_OnLoad()`必须存在于每个
Redis 模块。它是要初始化的模块的入口点，
注册其命令，以及可能的其他私有数据结构
它使用。

请注意，模块最好使用
模块的名称后跟一个点，最后是命令名称，
就像在`HELLOWORLD.RAND`.这样就不太可能
有碰撞。

请注意，如果不同的模块具有冲突命令，则不会
能够同时在Redis中工作，因为该功能
`RedisModule_CreateCommand`在其中一个模块中将失败，因此该模块
加载将中止返回错误条件。

## 模块初始化

上面的示例显示了函数的用法`RedisModule_Init()`.
它应该是模块调用的第一个函数`OnLoad`功能。
以下是函数原型：

    int RedisModule_Init(RedisModuleCtx *ctx, const char *modulename,
                         int module_version, int api_version);

这`Init`函数宣布 Redis 核心，该模块具有给定的
名称，其版本（由报告`MODULE LIST`），这是自愿的
以使用特定版本的 API。

如果 API 版本错误，则名称已被采用，或者有其他
类似错误，函数将返回`REDISMODULE_ERR`和模块
`OnLoad`函数应尽快返回，但出现错误。

在`Init`函数被调用，没有其他 API 函数可以调用，
否则，模块将发生故障，Redis 实例将崩溃。

第二个函数称为，`RedisModule_CreateCommand`，按顺序使用
将命令注册到 Redis 核心中。以下是原型：

    int RedisModule_CreateCommand(RedisModuleCtx *ctx, const char *name,
                                  RedisModuleCmdFunc cmdfunc, const char *strflags,
                                  int firstkey, int lastkey, int keystep);

如您所见，大多数 Redis 模块 API 调用都采用第一个参数
这`context`的模块，以便它们具有对模块的引用
调用它，命令和执行给定命令的客户端，依此类推。

要创建新命令，上述函数需要上下文，该命令的
name，指向实现命令的函数的指针，命令的标志
以及命令参数中键名的位置。

实现该命令的函数必须具有以下原型：

    int mycommand(RedisModuleCtx *ctx, RedisModuleString **argv, int argc);

命令函数参数只是将要传递的上下文
到所有其他 API 调用、命令参数向量和总数
的参数，由用户传递。

如您所见，参数作为指向特定数据的指针提供
类型，`RedisModuleString`.这是您拥有的 API 的不透明数据类型
函数访问和使用，直接访问其字段是永远不需要的。

放大示例命令实现，我们可以找到另一个调用：

    int RedisModule_ReplyWithLongLong(RedisModuleCtx *ctx, long long integer);

此函数向调用该命令的客户端返回一个整数，
与其他 Redis 命令完全相同，例如`INCR`或`SCARD`.

## 模块清理

在大多数情况下，不需要特殊清理。
卸载模块后，Redis 将自动注销命令和
取消订阅通知。
但是，在模块包含一些持久内存或
配置中，模块可以包括可选的`RedisModule_OnUnload`
功能。
如果模块提供此函数，它将在模块卸载期间调用
过程。
以下是函数原型：

    int RedisModule_OnUnload(RedisModuleCtx *ctx);

这`OnUnload`功能可以通过返回来防止模块卸载
`REDISMODULE_ERR`.
否则`REDISMODULE_OK`应返回。

## Redis 模块的设置和依赖关系

Redis 模块不依赖于 Redis 或其他一些库，也不依赖于 Redis 或其他库
需要用特定的`redismodule.h`文件。挨次
要创建新模块，只需复制最新版本的`redismodule.h`
在源代码树中，链接所需的所有库，然后创建
一个动态库，具有`RedisModule_OnLoad()`函数符号
出口。

该模块将能够加载到不同版本的Redis中。

可以将模块设计为支持较新和较旧的 Redis 版本，其中某些 API 函数并非在所有版本中都可用。
如果 API 函数未在当前运行的 Redis 版本中实现，则函数指针设置为 NULL。
这允许模块在使用函数之前检查函数是否存在：

    if (RedisModule_SetCommandInfo != NULL) {
        RedisModule_SetCommandInfo(cmd, &info);
    }

在最新版本的`redismodule.h`，方便宏`RMAPI_FUNC_SUPPORTED(funcname)`已定义。
使用宏或仅与 NULL 进行比较是个人喜好的问题。

# 将配置参数传递给 Redis 模块

当模块加载了`MODULE LOAD`命令，或使用
`loadmodule`指令`redis.conf`文件，用户能够通过
通过在模块后添加参数来配置模块参数
文件名：

    loadmodule mymodule.so foo bar 1234

在上面的示例中，字符串`foo`,`bar`和`1234`将会通过
到模块`OnLoad()`函数在`argv`参数作为数组
的 RedisModuleString 指针。传递的参数数为`argc`.

访问这些字符串的方式将在本文件的其余部分进行说明。
公文。通常，模块将存储模块配置参数
在一些`static`全局变量，可以访问模块范围，以便
配置可以更改不同命令的行为。

## 使用 RedisModuleString 对象

命令参数向量`argv`传递给模块的命令，以及
其他模块 API 函数的返回值，类型为`RedisModuleString`.

通常，您直接将模块字符串传递给其他 API 调用，但有时
您可能需要直接访问字符串对象。

有几个函数可以处理字符串对象：

    const char *RedisModule_StringPtrLen(RedisModuleString *string, size_t *len);

上述函数通过返回字符串的指针并设置其
长度在`len`.
您永远不应该写入字符串对象指针，正如您从
`const`指针限定符。

但是，如果需要，可以使用以下命令创建新的字符串对象
应用程序接口：

    RedisModuleString *RedisModule_CreateString(RedisModuleCtx *ctx, const char *ptr, size_t len);

上述命令返回的字符串必须使用相应的
调用`RedisModule_FreeString()`:

    void RedisModule_FreeString(RedisModuleString *str);

但是，如果您想避免必须释放字符串，则自动内存
管理，在本文件后面介绍，可以是一个很好的替代方案，由
为你做。

请注意，通过参数向量提供的字符串`argv`永远不需要
被释放。您只需要释放您创建的新字符串或新字符串
由其他 API 返回，其中指定返回的字符串必须
获得自由。

## 从数字创建字符串或将字符串解析为数字

从整数创建新字符串是一个非常常见的操作，因此
是执行此操作的函数：

    RedisModuleString *mystr = RedisModule_CreateStringFromLongLong(ctx,10);

类似地，为了将字符串解析为数字：

    long long myval;
    if (RedisModule_StringToLongLong(ctx,argv[1],&myval) == REDISMODULE_OK) {
        /* Do something with 'myval' */
    }

## 从模块访问 Redis 密钥

大多数 Redis 模块为了有用，必须与 Redis 交互
数据空间（这并不总是正确的，例如，ID生成器可以
切勿触摸 Redis 键）。Redis 模块有两个不同的 API，以便
访问 Redis 数据空间，一个是提供非常
快速访问和一组用于操作 Redis 数据结构的函数。
另一个API更高级别，允许调用Redis命令和
获取结果，类似于Lua脚本访问Redis的方式。

高级 API 对于访问 Redis 功能也很有用
不作为 API 提供。

一般来说，模块开发人员应该更喜欢低级API，因为命令
使用低级 API 实现，以与速度相当的速度运行
的本机 Redis 命令。但是，肯定有用例
更高级别的 API。例如，通常瓶颈可能是处理
数据，但不访问它。

另请注意，有时使用低级 API 并不比
更高级别。

## 调用 Redis 命令

用于访问 Redis 的高级 API 是`RedisModule_Call()`
函数，以及访问
返回的回复对象`Call()`.

`RedisModule_Call`使用特殊的调用约定，并带有格式说明符
用于指定要作为参数传递的对象类型
到函数。

仅使用命令名称和参数列表调用 Redis 命令。
但是，在调用命令时，参数可能来自不同
字符串类型：空终止的 C 字符串，RedisModuleString 对象作为
从`argv`参数，二进制
带有指针和长度等的安全 C 缓冲区。

例如，如果我想打电话`INCRBY`使用第一个参数（键）
在参数向量中接收的字符串`argv`，这是一个数组
的 RedisModuleString 对象指针，以及一个表示
数字“10”作为第二个参数（增量），我将使用以下
函数调用：

    RedisModuleCallReply *reply;
    reply = RedisModule_Call(ctx,"INCRBY","sc",argv[1],"10");

第一个参数是上下文，第二个参数始终以 null 结尾
具有命令名称的 C 字符串。第三个参数是格式说明符
其中，每个字符对应于将遵循的参数的类型。
在上述情况下`"sc"`表示一个 RedisModuleString 对象，以及一个 null
终止的 C 字符串。其他参数只是两个参数
指定。事实上`argv[1]`是一个 RedisModuleString 和`"10"`为空值
终止的 C 字符串。

以下是格式说明符的完整列表：

*   **c**-- 空终止的 C 字符串指针。
*   **b**-- C 缓冲区，需要两个参数：C 字符串指针和`size_t`长度。
*   **s**-- RedisModuleString as received in`argv`或者由其他 Redis 模块 API 返回 RedisModuleString 对象。
*   **l**-- 长整型。
*   **v**-- RedisModuleString 对象数组。
*   **!**-- 此修饰符仅告诉函数将命令复制到副本和 AOF。从参数解析的角度来看，它被忽略了。
*   **一个**-- 此修饰符，当`!`，告诉禁止AOF传播：命令将仅传播到副本。
*   **R**-- 此修饰符，当`!`，告诉禁止复制副本传播：如果启用，命令将仅传播到 AOF。

该函数返回一个`RedisModuleCallReply`对象关于成功，关于
返回错误 NULL。

当命令名称无效时返回 NULL，格式说明符使用
无法识别的字符，或者当使用
参数数量错误。在上述情况下`errno`var 设置为`EINVAL`.在启用了集群的实例中，当目标
键是关于非本地哈希槽的。在这种情况下`errno`设置为`EPERM`.

## 使用 RedisModuleCallReply 对象。

`RedisModuleCall`返回可使用
`RedisModule_CallReply*`功能系列。

为了获得类型或应答（对应于其中一种数据类型）
受 Redis 协议支持），该函数`RedisModule_CallReplyType()`
用于：

    reply = RedisModule_Call(ctx,"INCRBY","sc",argv[1],"10");
    if (RedisModule_CallReplyType(reply) == REDISMODULE_REPLY_INTEGER) {
        long long myval = RedisModule_CallReplyInteger(reply);
        /* Do something with myval. */
    }

有效的回复类型包括：

*   `REDISMODULE_REPLY_STRING`批量字符串或状态回复。
*   `REDISMODULE_REPLY_ERROR`错误。
*   `REDISMODULE_REPLY_INTEGER`有符号 64 位整数。
*   `REDISMODULE_REPLY_ARRAY`答复数组。
*   `REDISMODULE_REPLY_NULL`空回复。

字符串、错误和数组具有关联的长度。对于字符串和错误
长度对应于字符串的长度。对于数组，长度
是元素的数量。要获取回复长度，请使用以下函数
用于：

    size_t reply_len = RedisModule_CallReplyLength(reply);

为了获得整数回复的值，使用以下函数，如上面的示例所示：

    long long reply_integer_val = RedisModule_CallReplyInteger(reply);

使用错误类型的回复对象调用，上述函数始终
返回`LLONG_MIN`.

数组回复的子元素通过以下方式访问：

    RedisModuleCallReply *subreply;
    subreply = RedisModule_CallReplyArrayElement(reply,idx);

如果您尝试访问超出范围的元素，则上述函数返回 NULL。

字符串和错误（类似于字符串，但具有不同的类型）可以
通过以下方式使用进行访问，确保永远不要写入
生成的指针（作为`const`指针，以便
滥用必须非常明确）：

    size_t len;
    char *ptr = RedisModule_CallReplyStringPtr(reply,&len);

如果回复类型不是字符串或错误，则返回 NULL。

RedisCallReply 对象与模块字符串对象不同
（RedisModuleString types）。但是，有时您可能需要传递回复
类型为字符串或整数，用于需要模块字符串的 API 函数。

在这种情况下，您可能需要评估是否使用低级别
API 可能是实现命令的更简单方法，或者您可以使用
以下函数，以便从
字符串、错误或整数类型的调用回复：

    RedisModuleString *mystr = RedisModule_CreateStringFromCallReply(myreply);

如果答复的类型不正确，则返回 NULL。
返回的字符串对象应随`RedisModule_FreeString()`
像往常一样，或通过启用自动内存管理（请参阅相应的
部分）。

## 释放呼叫回复对象

必须使用 释放回复对象`RedisModule_FreeCallReply`.对于阵列，
您只需要释放顶级回复，而不是嵌套回复。
目前模块实现提供了保护，以避免
崩溃，如果你释放一个嵌套的回复对象的错误，但是这个功能
不能保证永远在这里，所以不应该被认为是一部分
的 API。

如果使用自动内存管理（本文档稍后将介绍）
你不需要免费回复（但如果你想释放，你仍然可以
内存尽快）。

## 从 Redis 命令返回值

与普通的 Redis 命令一样，通过模块实现的新命令必须是
能够将值返回给调用方。该 API 导出一组函数
这个目标，为了返回Redis协议的通常类型，以及
元素等类型的数组。此外，错误也可以与任何
错误字符串和代码（错误代码是
错误消息，如“BUSY 服务器正忙”错误中的“BUSY”字符串
消息）。

调用所有向客户端发送回复的函数
`RedisModule_ReplyWith<something>`.

要返回错误，请使用：

    RedisModule_ReplyWithError(RedisModuleCtx *ctx, const char *err);

错误类型错误的键有一个预定义的错误字符串：

    REDISMODULE_ERRORMSG_WRONGTYPE

用法示例：

    RedisModule_ReplyWithError(ctx,"ERR invalid arguments");

我们已经看到了如何回复`long long`在上面的例子中：

    RedisModule_ReplyWithLongLong(ctx,12345);

要使用不能包含二进制值或换行符的简单字符串进行回复，
（所以适合发送小词，比如“OK”）我们使用：

    RedisModule_ReplyWithSimpleString(ctx,"OK");

可以使用二进制安全的“批量字符串”进行回复，使用
两种不同的功能：

    int RedisModule_ReplyWithStringBuffer(RedisModuleCtx *ctx, const char *buf, size_t len);

    int RedisModule_ReplyWithString(RedisModuleCtx *ctx, RedisModuleString *str);

第一个函数获取 C 指针和长度。第二个 RedisModuleString
对象。根据您手头的源类型使用其中之一。

为了用数组回复，你只需要使用一个函数来发出
数组长度，后跟对上述函数的调用次数与数字一样多
数组的元素是：

    RedisModule_ReplyWithArray(ctx,2);
    RedisModule_ReplyWithStringBuffer(ctx,"age",3);
    RedisModule_ReplyWithLongLong(ctx,22);

返回嵌套数组很容易，您的嵌套数组元素只使用另一个
调用`RedisModule_ReplyWithArray()`后跟发出的调用
子数组元素。

## 返回具有动态长度的数组

有时不可能事先知道
一个数组。例如，假设一个 Redis 模块实现了一个 FACTOR
给定一个数字输出质因数的命令。而不是
对数字进行因子分解，将质因数存储到数组中，以及
稍后生成命令回复，更好的解决方案是启动一个数组
在长度未知的地方回复，并在以后设置。这已经完成了
带有特殊参数`RedisModule_ReplyWithArray()`:

    RedisModule_ReplyWithArray(ctx, REDISMODULE_POSTPONED_LEN);

上面的调用启动一个数组回复，以便我们可以使用其他`ReplyWith`调用
以生成数组项。最后为了设定长度，
使用以下调用：

    RedisModule_ReplySetArrayLength(ctx, number_of_items);

对于 FACTOR 命令，这转换为一些类似的代码
对此：

    RedisModule_ReplyWithArray(ctx, REDISMODULE_POSTPONED_LEN);
    number_of_factors = 0;
    while(still_factors) {
        RedisModule_ReplyWithLongLong(ctx, some_factor);
        number_of_factors++;
    }
    RedisModule_ReplySetArrayLength(ctx, number_of_factors);

此功能的另一个常见用例是迭代
一些集合，只返回通过某种过滤的那些。

可以有多个嵌套数组，其中包含延迟的答复。
每次调用`SetArray()`将设置最新对应的长度
调用`ReplyWithArray()`:

    RedisModule_ReplyWithArray(ctx, REDISMODULE_POSTPONED_LEN);
    ... generate 100 elements ...
    RedisModule_ReplyWithArray(ctx, REDISMODULE_POSTPONED_LEN);
    ... generate 10 elements ...
    RedisModule_ReplySetArrayLength(ctx, 10);
    RedisModule_ReplySetArrayLength(ctx, 100);

这将创建一个 100 items 数组，最后一个元素是 10 items 数组。

## 类型和类型检查

通常命令需要检查参数的数量和键的类型
是正确的。为了报告错误的arity，有一个特定的功能
叫`RedisModule_WrongArity()`.用法是微不足道的：

    if (argc != 2) return RedisModule_WrongArity(ctx);

检查错误的类型涉及打开密钥并检查类型：

    RedisModuleKey *key = RedisModule_OpenKey(ctx,argv[1],
        REDISMODULE_READ|REDISMODULE_WRITE);

    int keytype = RedisModule_KeyType(key);
    if (keytype != REDISMODULE_KEYTYPE_STRING &&
        keytype != REDISMODULE_KEYTYPE_EMPTY)
    {
        RedisModule_CloseKey(key);
        return RedisModule_ReplyWithError(ctx,REDISMODULE_ERRORMSG_WRONGTYPE);
    }

请注意，您经常希望继续执行命令，如果键
是预期类型，或者如果它是空的。

## 对密钥的低级别访问

对键的低级访问允许对关联的值对象执行操作
直接到键，其速度类似于 Redis 内部使用的速度
实现内置命令。

打开密钥后，将返回一个密钥指针，该指针将与所有
其他低级 API 调用，以便对密钥或其执行操作
关联的值。

因为 API 意味着非常快，所以它不能做太多的运行时
检查，因此用户必须了解要遵循的某些规则：

*   多次打开同一键，其中至少打开一个实例进行写入，未定义，并可能导致崩溃。
*   当密钥处于打开状态时，只能通过低级密钥 API 访问它。例如，打开一个密钥，然后使用`RedisModule_Call()`API 将导致崩溃。但是，打开密钥，使用低级 API 执行一些操作，关闭它，然后使用其他 API 管理相同的密钥，然后再次打开它以执行更多工作，这是安全的。

为了打开一个键`RedisModule_OpenKey`使用函数。它返回
一个键指针，我们将用于所有后续调用以访问和修改
值：

    RedisModuleKey *key;
    key = RedisModule_OpenKey(ctx,argv[1],REDISMODULE_READ);

第二个参数是键名，它必须是`RedisModuleString`对象。
第三个参数是模式：`REDISMODULE_READ`或`REDISMODULE_WRITE`.
可以使用`|`按位或打开键的两种模式
两种模式。目前，也可以访问打开以进行写入的密钥以进行读取
但这被认为是一个实现细节。正确的模式应该
用于合理的模块。

您可以打开不存在的密钥进行写入，因为将创建密钥
当尝试写入密钥时执行。但是，打开按键时
只是为了阅读，`RedisModule_OpenKey`如果密钥不返回 NULL
存在。

使用完密钥后，可以使用以下命令将其关闭：

    RedisModule_CloseKey(key);

请注意，如果启用了自动内存管理，则不会强制
关闭键。当模块函数返回时，Redis 将注意关闭
所有的钥匙仍然打开。

## 获取密钥类型

要获取密钥的值，请使用`RedisModule_KeyType()`功能：

    int keytype = RedisModule_KeyType(key);

它返回以下值之一：

    REDISMODULE_KEYTYPE_EMPTY
    REDISMODULE_KEYTYPE_STRING
    REDISMODULE_KEYTYPE_LIST
    REDISMODULE_KEYTYPE_HASH
    REDISMODULE_KEYTYPE_SET
    REDISMODULE_KEYTYPE_ZSET

以上只是通常的 Redis 密钥类型，并添加了一个空
type，表示键指针与空键相关联
尚不存在。

## 创建新密钥

要创建新密钥，请打开它进行写入，然后使用一个密钥写入它
的关键写入功能。例：

    RedisModuleKey *key;
    key = RedisModule_OpenKey(ctx,argv[1],REDISMODULE_WRITE);
    if (RedisModule_KeyType(key) == REDISMODULE_KEYTYPE_EMPTY) {
        RedisModule_StringSet(key,argv[2]);
    }

## 删除密钥

只需使用：

    RedisModule_DeleteKey(key);

函数返回`REDISMODULE_ERR`如果密钥未打开以进行写入。
请注意，删除密钥后，将对其进行设置以成为目标
通过新的按键命令。例如`RedisModule_KeyType()`会再来的就是
一个空键，并写入它将创建一个新键，可能是另一个新键
类型（取决于所使用的 API）。

## 管理密钥过期 （TTL）

为了控制密钥过期，提供了两个功能，可以设置，
修改、获取和取消设置与密钥关联的生存时间。

使用一个函数来查询打开的密钥的当前过期：

    mstime_t RedisModule_GetExpire(RedisModuleKey *key);

该函数返回密钥的生存时间（以毫秒为单位），或者
`REDISMODULE_NO_EXPIRE`作为特殊值来表示键没有关联
过期或根本不存在（您可以区分检查中的两种情况
如果密钥类型为`REDISMODULE_KEYTYPE_EMPTY`).

为了更改密钥的过期，将改用以下函数：

    int RedisModule_SetExpire(RedisModuleKey *key, mstime_t expire);

在不存在的密钥上调用时，`REDISMODULE_ERR`返回，因为
该函数只能将过期与现有打开的密钥（不存在）相关联
open 键仅在创建数据类型为新值时有用
特定的写入操作）。

再次`expire`时间以毫秒为单位指定。如果密钥当前具有
没有过期，设置了新的过期。如果密钥已过期，则为
替换为新值。

如果密钥具有过期状态，并且特殊值`REDISMODULE_NO_EXPIRE`是
用作新的过期，过期将被删除，类似于 Redis
`PERSIST`命令。如果密钥已经是持久的，则没有操作
执行。

## 获取值的长度

有一个函数可以检索值的长度
与打开的密钥相关联。返回的长度是特定于值的，并且
字符串的字符串长度，以及聚合的元素数
数据类型（列表中有多少个元素，集合，排序集，哈希）。

    size_t len = RedisModule_ValueLength(key);

如果该键不存在，则函数返回 0：

## 字符串类型 API

设置新的字符串值，如 Redis`SET`命令执行，执行
用：

    int RedisModule_StringSet(RedisModuleKey *key, RedisModuleString *str);

该函数的工作方式与 Redis 完全相同`SET`命令本身，即如果
有一个先前的值（任何类型）它将被删除。

使用 DMA（直接内存）执行现有字符串值的访问
访问）以提高速度。API 将返回一个指针和一个长度，因此
可以直接访问并在需要时修改字符串。

    size_t len, j;
    char *myptr = RedisModule_StringDMA(key,&len,REDISMODULE_WRITE);
    for (j = 0; j < len; j++) myptr[j] = 'A';

在上面的示例中，我们直接在字符串上写入。请注意，如果需要
要写，一定要一定要要求`WRITE`模式。

仅当未使用密钥执行其他操作时，DMA 指针才有效
在使用指针之前，在 DMA 调用之后。

有时，当我们想要直接操作字符串时，我们需要更改
它们的大小也是如此。对于此范围，`RedisModule_StringTruncate`功能
使用。例：

    RedisModule_StringTruncate(mykey,1024);

该函数根据需要截断或放大字符串，并填充
如果以前的长度小于我们请求的新长度，则为零字节。
如果字符串自`key`与打开的空键相关联，
将创建一个字符串值并将其关联到该键。

注意，每次`StringTruncate()`被调用，我们需要重新获取
再次使用 DMA 指针，因为旧的指针可能无效。

## 列表类型 API

可以从列表值中推送和弹出值：

    int RedisModule_ListPush(RedisModuleKey *key, int where, RedisModuleString *ele);
    RedisModuleString *RedisModule_ListPop(RedisModuleKey *key, int where);

在这两个 API 中，`where`参数指定是从尾部推送还是弹出
或 head，使用以下宏：

    REDISMODULE_LIST_HEAD
    REDISMODULE_LIST_TAIL

返回的元素`RedisModule_ListPop()`就像字符串创建
`RedisModule_CreateString()`，则必须随
`RedisModule_FreeString()`或通过启用自动内存管理。

## 设置类型 API

工作正在进行中。

## 排序集类型 API

文档缺失，请参考里面的顶级评论`module.c`
用于以下功能：

*   `RedisModule_ZsetAdd`
*   `RedisModule_ZsetIncrby`
*   `RedisModule_ZsetScore`
*   `RedisModule_ZsetRem`

对于排序的集合迭代器：

*   `RedisModule_ZsetRangeStop`
*   `RedisModule_ZsetFirstInScoreRange`
*   `RedisModule_ZsetLastInScoreRange`
*   `RedisModule_ZsetFirstInLexRange`
*   `RedisModule_ZsetLastInLexRange`
*   `RedisModule_ZsetRangeCurrentElement`
*   `RedisModule_ZsetRangeNext`
*   `RedisModule_ZsetRangePrev`
*   `RedisModule_ZsetRangeEndReached`

## 哈希类型 API

文档缺失，请参考里面的顶级评论`module.c`
用于以下功能：

*   `RedisModule_HashSet`
*   `RedisModule_HashGet`

## 迭代聚合值

工作正在进行中。

## 复制命令

如果要像使用普通的 Redis 命令一样使用模块命令，请在
复制的 Redis 实例的上下文，或使用 AOF 文件进行持久性，
对于模块命令来说，以一致的方式处理其复制非常重要
道路。

使用更高级别的 API 调用命令时，将进行复制
如果在 格式字符串中使用 “！” 修饰符，则自动
`RedisModule_Call()`如以下示例所示：

    reply = RedisModule_Call(ctx,"INCRBY","!sc",argv[1],"10");

如您所见，格式说明符是`"!sc"`.爆炸不会被解析为
格式说明符，但它在内部将命令标记为“必须复制”。

如果使用上述编程风格，则没有问题。
然而，有时事情比这更复杂，你使用低级别
应用程序接口。在这种情况下，如果命令执行中没有副作用，并且
它始终如一地执行相同的工作，可以做的是
在用户执行命令时逐字复制该命令。要做到这一点，你只需
需要调用以下函数：

    RedisModule_ReplicateVerbatim(ctx);

使用上述 API 时，不应使用任何其他复制功能
因为它们不能保证混合良好。

但是，这不是唯一的选择。也可以准确地告诉
Redis 将哪些命令复制为命令执行的效果，使用
类似于`RedisModule_Call()`而是调用命令
将其发送到 AOF/副本流。例：

    RedisModule_Replicate(ctx,"INCRBY","cl","foo",my_increment);

可以致电`RedisModule_Replicate`多次，并且每个
将发出命令。发出的所有序列都包装在
`MULTI/EXEC`事务，以便 AOF 和复制效果是
与执行单个命令相同。

请注意，`Call()`复制和`Replicate()`复制有一个规则，
如果你想混合两种形式的复制（不一定是好的
如果有更简单的方法）。复制的命令`Call()`
始终是决赛中发出的第一个`MULTI/EXEC`块，而所有
发出的命令`Replicate()`将紧随其后。

## 自动内存管理

通常，当用C语言编写程序时，程序员需要管理
手动内存。这就是为什么Redis模块API有要发布的功能的原因。
字符串、关闭打开的键、免费回复等。

但是，鉴于命令是在包含的环境中执行的，并且
通过一套严格的API，Redis能够提供自动内存管理
到模块，以牺牲一些性能为代价（大多数时候，非常低
成本）。

启用自动内存管理后：

1.  您无需关闭打开的键。
2.  您不需要免费回复。
3.  你不需要释放 RedisModuleString 对象。

但是，如果您愿意，您仍然可以这样做。例如，自动内存
管理可能是活动的，但在一个循环中分配了很多字符串，
您可能仍然希望释放不再使用的字符串。

要启用自动内存管理，只需调用以下命令
命令实现开始时的函数：

    RedisModule_AutoMemory(ctx);

自动内存管理通常是要走的路，但经验丰富
C程序员可能不会为了获得一些速度和内存使用而使用它。
效益。

## 将内存分配到模块中

正常的C程序使用`malloc()`和`free()`为了分配和
动态释放内存。而在Redis模块中，malloc的使用是
在技术上没有被禁止，使用Redis模块要好得多
特定功能，可精确替代`malloc`,`free`,
`realloc`和`strdup`.这些函数是：

    void *RedisModule_Alloc(size_t bytes);
    void* RedisModule_Realloc(void *ptr, size_t bytes);
    void RedisModule_Free(void *ptr);
    void RedisModule_Calloc(size_t nmemb, size_t size);
    char *RedisModule_Strdup(const char *str);

他们的工作方式与他们的完全相同`libc`等效调用，无论它们使用什么
Redis 使用的相同分配器，以及使用这些分配的内存
函数由`INFO`内存部分中的命令是
在执行`maxmemory`政策，一般是
Redis 可执行文件的第一个公民。相反，该方法
使用 libc 在模块内部分配`malloc()`对 Redis 是透明的。

使用模块功能以分配内存的另一个原因
也就是说，在模块内创建本机数据类型时，RDB加载
函数可以直接返回反序列化字符串（来自 RDB 文件）
如`RedisModule_Alloc()`分配，因此它们可以直接用于
加载后填充数据结构，而不必复制它们
到数据结构。

## 池分配器

有时在命令实现中，需要执行许多
在命令结束时不会保留的小分配
执行，但只是执行命令本身的功能。

使用 Redis 池分配器可以更轻松地完成此工作：

    void *RedisModule_PoolAlloc(RedisModuleCtx *ctx, size_t bytes);

它的工作原理类似于`malloc()`，并返回与
下一个幂的二大于或等于`bytes`（用于最大对齐
8 字节）。但是，它以块为单位分配内存，因此它会产生开销
的分配很小，更重要的是，分配的内存
在命令返回时自动释放。

因此，一般来说，短期生活分配是游泳池的良好候选者。
分配器。

## 编写与 Redis 集群兼容的命令

文档缺失，请检查内部以下功能`module.c`:

    RedisModule_IsKeysPositionRequest(ctx);
    RedisModule_KeyAtPos(ctx,pos);
