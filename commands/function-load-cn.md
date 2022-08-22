将库加载到 Redis。

该命令获取单个必需参数, 该参数是实现库的源代码。
库负载必须以 Shebang 语句开头, 该语句提供有关库的元数据 (如要使用的引擎和库名称) 。
社邦格式：`#!<engine name> name=<library name>`.当前引擎名称必须为`lua`.

对于 Lua 引擎, 实现应声明一个或多个入口点到库, 并带有[`redis.register_function()`应用程序接口](/topics/lua-api#redis.register_function).
加载后, 您可以使用`FCALL` (或`FCALL_RO` (如果适用) 命令。

尝试加载具有已存在的名称的库时, Redis 服务器将返回错误。
这`REPLACE`修饰符更改此行为, 并用新内容覆盖现有库。

在以下情况下, 该命令将返回错误：

*   无效*引擎名称*提供了。
*   库的名称已存在, 但没有`REPLACE`修饰语。
*   使用另一个库中已存在的名称创建库中的函数 (即使`REPLACE`已指定) 。
*   引擎在创建库的函数时失败 (例如, 由于编译错误) 。
*   库未声明任何函数。

欲了解更多信息, 请参阅[Redis 函数简介](/topics/functions-intro).

@return

@string - 已加载的库名称

@examples

下面的示例将创建一个名为`mylib`具有单个功能, `myfunc`, 返回它得到的第一个参数。

    redis> FUNCTION LOAD "#!lua name=mylib \n redis.register_function('myfunc', function(keys, args) return args[1] end)"
    mylib
    redis> FCALL myfunc 0 hello
    "hello"
