---
title: "Redis Lua API reference"
linkTitle: "Lua API"
weight: 3
description: >
   Executing Lua in Redis
aliases:
    - /topics/lua-api
---

Redis 包括一个嵌入式[Lua 5.1](https://www.lua.org/)译员。
解释器运行用户定义的[临时脚本](/topics/eval-intro)和[功能](/topics/functions-intro).脚本在沙盒上下文中运行, 只能访问特定的 Lua 包。本页介绍在执行上下文中可用的包和 API。

## 沙盒上下文

沙盒 Lua 上下文尝试防止意外误用并减少来自服务器环境的潜在威胁。

脚本不应尝试访问 Redis 服务器的基础主机系统。
这包括文件系统、网络以及执行 API 支持以外的任何其他系统调用尝试。

脚本应仅对存储在 Redis 中的数据以及作为其执行参数提供的数据进行操作。

### 全局变量和函数

沙盒 Lua 执行上下文阻止全局变量和函数的声明。
全局变量的阻止已到位, 以确保脚本和函数不会尝试维护除 Redis 中存储的数据之外的任何运行时上下文。
在 (有些不常) ）用例中, 需要在执行之间维护上下文, 
您应该将上下文存储在 Redis 的密钥空间中。

Redis 在尝试执行以下代码段时, 将返回“脚本尝试创建全局变量 'my_global_variable'”错误：

```lua
my_global_variable = 'some value'
```

对于以下全局函数声明, 情况类似：

```lua
function my_global_funcion()
  -- Do something amazing
end
```

当您的脚本尝试访问运行时上下文中未定义的任何全局变量时, 您也会收到类似的错误：

```lua
-- The following will surely raise an error
return an_undefined_global_variable
```

相反, 所有变量和函数定义都需要声明为本地定义。
为此, 您需要在[*当地*](https://www.lua.org/manual/5.1/manual.html#2.4.7)声明的关键字。
例如, Redis 将认为以下代码段完全有效：

```lua
local my_local_variable = 'some value'

local function my_local_function()
  -- Do something else, but equally amazing
end
```

**注意：**
沙盒尝试阻止使用全局变量。
使用Lua的调试功能或其他方法 (例如更改用于实现全局保护的元表以绕过沙) ）并不难。
但是, 很难意外地规避保护。
如果用户弄乱了 Lua 全局状态, 则无法保证 AOF 和复制的一致性。
换句话说, 就是不要这样做。

### 导入的 Lua 模块

在沙盒执行上下文中不支持使用导入的 Lua 模块。
沙盒执行上下文通过禁用 Lua 的[`require`功能](https://www.lua.org/pil/8.1.html).

Redis 附带的、您可以在脚本中使用的唯一库列在[运行时库](#runtime-libraries)部分。

## 运行时全局变量

虽然沙盒阻止用户声明全局变量, 但执行上下文会预先填充其中的几个全局变量。

### 这*雷迪斯*单身 人士

这*雷迪斯*单例是可从所有脚本访问的对象实例。
它提供了从脚本与 Redis 交互的 API。
其描述如下[下面](#redis_object).

### <a name="the-keys-global-variable"></a>这*钥匙*全局变量

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   在功能中可用：无

**重要：**
为了确保在独立部署和群集部署中正确执行脚本, 函数访问的所有键名称都必须显式提供为输入键参数。
脚本**应该只**访问键, 其名称作为输入参数给出。
脚本**永远不应该**访问键具有以编程方式生成的名称或基于存储在数据库中的数据结构的内容。

这*钥匙*全局变量仅适用于[临时脚本](/topics/eval-intro).
它预填充了所有键名输入参数。

### <a name="the-argv-global-variable"></a>这*断续器*全局变量

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   在功能中可用：无

这*断续器*全局变量仅在[临时脚本](/topics/eval-intro).
它预先填充了所有常规输入参数。

## <a name="redis_object"></a>*雷迪斯*对象

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

Redis Lua 执行上下文始终提供名为*雷迪斯*.
这*雷迪斯*实例使脚本能够与运行它的 Redis 服务器进行交互。
以下是*雷迪斯*对象实例。

### <a name="redis.call"></a> `redis.call(command [,arg...])`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

这`redis.call()`函数调用给定的 Redis 命令并返回其应答。
它的输入是命令和参数, 一旦调用, 它就会在 Redis 中执行命令并返回回复。

例如, 我们可以将`ECHO`命令, 并返回其回复, 如下所示：

```lua
return redis.call('ECHO', 'Echo, echo... eco... o...')
```

如果以及何时`redis.call()`触发运行时异常, 原始异常作为错误自动引发回用户。
因此, 尝试执行以下临时脚本将失败并生成运行时异常, 因为`ECHO`只接受零个或一个参数：

```lua
redis> EVAL "return redis.call('ECHO', 'Echo,', 'echo... ', 'eco... ', 'o...')" 0
(error) ERR Error running script (call to b0345693f4b77517a711221050e76d24ae60b7f7): @user_script:1: @user_script: 1: Wrong number of args calling Redis command from script
```

请注意, 调用可能由于各种原因而失败, 请参阅[在低内存条件下执行](/topics/eval-intro#execution-under-low-memory-conditions)和[脚本标志](#script_flags)

要处理 Redis 运行时错误, 请改用 'redis.pcall ) ）。

### <a name="redis.pcall"></a> `redis.pcall(command [,arg...])`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

此函数允许处理 Redis 服务器引发的运行时错误。
这`redis.pcall()`函数的行为与[`redis.call()`](#redis.call), 除了它：

*   始终返回答复。
*   从不引发运行时异常, 并代替它返回[`redis.error_reply`](#redis.error_reply)如果服务器引发运行时异常。

下面演示如何使用`redis.pcall()`从临时脚本的上下文中截获和处理运行时异常。

```lua
local reply = redis.pcall('ECHO', unpack(ARGV))
if reply['err'] ~= nil then
  -- Handle the error sometime, but for now just log it
  redis.log(redis.LOG_WARNING, reply['err'])
  reply['err'] = 'Something is wrong, but no worries, everything is under control'
end
return reply
```

使用多个参数计算此脚本将返回：

    redis> EVAL "..." 0 hello world
    (error) Something is wrong, but no worries, everything is under control

### <a name="redis.error_reply"></a> `redis.error_reply(x)`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

这是一个帮助器函数, 它返回[错误回复](/topics/protocol#resp-errors).
帮助程序接受单个字符串参数, 并返回一个 Lua 表, 其中包含*犯 错*字段设置为该字符串。

以下代码的结果是*错误1*和*错误2*在所有意图和目的上都是相同的：

```lua
local text = 'My very special error'
local reply1 = { err = text }
local reply2 = redis.error_reply(text)
```

因此, 这两种形式都是有效的, 可以作为从脚本返回错误回复的方法：

    redis> EVAL "return { err = 'My very special table error' }" 0
    (error) My very special table error
    redis> EVAL "return redis.error_reply('My very special reply error')" 0
    (error) My very special reply error

要恢复 Redis 状态, 请参阅[`redis.status_reply()`](#redis.status_reply).
请参阅[数据类型转换](#data-type-conversion)用于返回其他响应类型。

### <a name="redis.status_reply"></a> `redis.status_reply(x)`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

这是一个帮助器函数, 它返回[简单字符串回复](/topics/protocol#resp-simple-strings).
“OK”是标准 Redis 状态回复的一个示例。
Lua API 将状态回复表示为具有单个字段的表, *还行*, 使用简单的状态字符串进行设置。

以下代码的结果是*状态1*和*状态2*在所有意图和目的上都是相同的：

```lua
local text = 'Frosty'
local status1 = { ok = text }
local status2 = redis.status_reply(text)
```

因此, 这两种形式作为从脚本返回状态答复的有效方法：

    redis> EVAL "return { ok = 'TICK' }" 0
    TICK
    redis> EVAL "return redis.status_reply('TOCK')" 0
    TOCK

要返回 Redis 错误回复, 请参阅[`redis.error_reply()`](#redis.error_reply).
请参阅[数据类型转换](#data-type-conversion)用于返回其他响应类型。

### <a name="redis.sha1hex"></a> `redis.sha1hex(x)`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

此函数返回其单个字符串参数的 SHA1 十六进制摘要。

例如, 您可以获取空字符串的 SHA1 摘要：

    redis> EVAL "return redis.sha1hex('')" 0
    "da39a3ee5e6b4b0d3255bfef95601890afd80709"

### <a name="redis.log"></a> `redis.log(level, message)`

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

此函数写入 Redis 服务器日志。

它需要两个输入参数：日志级别和消息。
该消息是要写入日志文件的字符串。
日志级别可以是以下列：

*   `redis.LOG_DEBUG`
*   `redis.LOG_VERBOSE`
*   `redis.LOG_NOTICE`
*   `redis.LOG_WARNING`

这些级别映射到服务器的日志级别。
日志仅记录等于或大于服务器级别的消息`loglevel`配置指令。

以下代码段：

```lua
redis.log(redis.LOG_WARNING, 'Something is terribly wrong')
```

将在服务器的日志中生成类似于以下内容的行：

    [32343] 22 Mar 15:21:39 # Something is terribly wrong

### <a name="redis.setresp"></a> `redis.setresp(x)`

*   自版本以来： 6.0.0
*   在脚本中可用：是
*   功能可用：是

此函数允许执行脚本在[Redis 序列化协议  (RES) ）](/topics/protocol)返回的答复的版本[`redis.call()](#redis.call) and [`redis.pall) (）](#redis.pcall).
它期望单个数字参数作为协议的版本。
默认协议版本为*2*, 但可以切换到版本*3*.

以下是切换到 RESP3 回复的示例：

```lua
redis.setresp(3)
```

请参阅[数据类型转换](#data-type-conversion)了解有关类型转换的详细信息。

### <a name="redis.set_repl"></a> `redis.set_repl(x)`

*   自版本以来： 3.2.0
*   在脚本中可用：是
*   在功能中可用：无

**注意：**
仅当使用脚本效果复制时, 此功能才可用。
使用逐字脚本复制时调用它将导致错误。
从 Redis 版本 2.6.0 开始, 脚本是逐字复制的, 这意味着脚本的源代码被发送以供副本执行并存储在 AOF 中。
版本 3.2.0 中添加的替代复制模式仅允许复制脚本的效果。
从 Redis 版本 7.0 开始, 不再支持脚本复制, 唯一可用的复制模式是脚本效果复制。

**警告：**
这是一项高级功能。滥用可能会违反绑定 Redis 主服务器、其副本和 AOF 内容以保存相同逻辑内容的协定而造成损害。

此函数允许脚本断言对其效果如何传播到副本和之后的 AOF 的控制。
脚本的影响是它调用的 Redis 写入命令。

默认情况下, 将复制脚本执行的所有写入命令。
但是, 有时更好地控制此行为可能会有所帮助。
例如, 当仅在主数据中存储中间值时, 可能会出现这种情况。

考虑一个脚本, 它与两组相交, 并将结果存储在临时密钥中`SUNIONSTORE`.
然后, 它选取五个随机元素  (`SRANDMEMBER) ） 从十字路口和商店  (`SAD) `） 它们在另一组中。
最后, 在返回之前, 它将删除存储两个源集交集的临时密钥。

在这种情况下, 只需要复制具有五个随机选择的元素的新集合。
复制`SUNIONSTORE`命令和临时密钥的“DEL”ition是不必要的和浪费的。

这`redis.set_repl()`function 指示服务器如何根据复制来处理后续写入命令。
它接受仅属于以下之一的单个输入参数：

*   `redis.REPL_ALL`：将效果复制到 AOF 和复制副本。
*   `redis.REPL_AOF`：将效果复制到单独使用 AOF。
*   `redis.REPL_REPLICA`：将效果单独复制到复制品。
*   `redis.REPL_SLAVE`：与 相同`REPL_REPLICA`, 为向后兼容而保留。
*   `redis.REPL_NONE`：完全禁用效果复制。

默认情况下, 脚本引擎初始化为`redis.REPL_ALL`设置脚本开始执行的时间。
您可以致电`redis.set_repl()`在脚本执行期间随时运行, 以在不同的复制模式之间切换。

下面是一个简单的示例：

```lua
redis.replicate_commands() -- Enable effects replication in versions lower than Redis v7.0
redis.call('SET', KEYS[1], ARGV[1])
redis.set_repl(redis.REPL_NONE)
redis.call('SET', KEYS[2], ARGV[2])
redis.set_repl(redis.REPL_ALL)
redis.call('SET', KEYS[3], ARGV[3])
```

如果通过调用来运行此脚本`EVAL "..." 3 A B C 1 2 3`, 结果将是只有键*一个*和*C*在复制副本和 AOF 上创建。

### <a name="redis.replicate_commands"></a> `redis.replicate_commands()`

*   自版本以来： 3.2.0
*   直到版本： 7.0.0
*   在脚本中可用：是
*   在功能中可用：无

此函数将脚本的复制模式从逐字复制切换到效果复制。
您可以使用它来覆盖 Redis 在版本 7.0 之前使用的默认逐字脚本复制模式。

**注意：**
从 Redis v7.0 开始, 不再支持逐字脚本复制。
默认且仅支持脚本复制模式的是脚本效果的复制。
欲了解更多信息, 请参阅[`Replicating commands instead of scripts`](/topics/eval-intro#replicating-commands-instead-of-scripts)

### <a name="redis.breakpoint"></a>  `redis.breakpoint()`

*   自版本以来： 3.2.0
*   在脚本中可用：是
*   在功能中可用：无

此函数在使用 Redis Lua 调试器] (/topics/ld) ） 时触发断点。

### <a name="redis.debug"></a> `redis.debug(x)`

*   自版本以来： 3.2.0
*   在脚本中可用：是
*   在功能中可用：无

此函数在[Redis Lua 调试器](/topics/ldb)安慰。

### <a name="redis.acl_check_cmd"></a> `redis.acl_check_cmd(command [,arg...])`

*   自版本以来： 7.0.0
*   在脚本中可用：是
*   功能可用：是

此函数用于检查运行脚本的当前用户是否具有[前交叉韧带](/topics/acl)使用给定参数执行给定命令的权限。

返回值为布尔值`true`如果当前用户有权执行命令 (通过调用[redis.call](#redis.call)或[redis.pcall](#redis.pcall) ） 或`false`以防万一他们没有。

如果传递的命令或其参数无效, 该函数将引发错误。

### <a name="redis.register_function"></a> `redis.register_function`

*   自版本以来： 7.0.0
*   在脚本中可用：否
*   功能可用：是

此函数只能从`FUNCTION LOAD`命令。
调用时, 它会将函数注册到加载的库中。
可以使用位置参数或命名参数调用该函数。

#### <a name="redis.register_function_pos_args"></a>位置参数：`redis.register_function(name, callback)`

第一个参数`redis.register_function`是表示函数名称的 Lua 字符串。
第二个参数`redis.register_function`是一个 Lua 函数。

用法示例：

    redis> FUNCTION LOAD "#!lua name=mylib\n redis.register_function('noop', function() end)"

#### <a name="redis.register_function_named_args"></a>命名参数：`redis.register_function{function_name=name, callback=callback, flags={flag1, flag2, ..}, description=description}`

命名参数变体接受以下参数：

*   *function_name*：函数的名称。
*   *回调*：函数的回调。
*   *标志*：一个字符串数组, 每个字符串都有一个函数标志 (可) ）。
*   *描述*：函数的描述 (可) ）。

双*function_name*和*回调*是强制性的。

用法示例：

    redis> FUNCTION LOAD "#!lua name=mylib\n redis.register_function{function_name='noop', callback=function() end, flags={ 'no-writes' }, description='Does nothing'}"

#### <a name="script_flags"></a>脚本标志

**重要：**
请谨慎使用脚本标志, 如果误用, 可能会产生负面影响。
请注意, Eval 脚本的默认值与下面提到的函数的默认值不同, 请参阅[评估标志](/docs/manual/programmability/eval-intro/#eval-flags)

注册函数或加载 Eval 脚本时, 服务器不知道它如何访问数据库。
默认情况下, Redis 假定所有脚本都读取和写入数据。
这将导致以下行为：

1.  他们可以读取和写入数据。
2.  它们可以在群集模式下运行, 并且无法运行访问不同哈希槽的键的命令。
3.  拒绝对陈旧副本执行, 以避免读取不一致。
4.  拒绝在低内存下执行, 以避免超过配置的阈值。

您可以使用以下标志并指示服务器以不同的方式对待脚本的执行：

*   `no-writes`：此标志表示脚本仅读取数据, 但从不写入。

    默认情况下, Redis 将拒绝执行标记的脚本 (函数和评估脚本[家当](/topics/eval-intro#eval-flags) ） 针对只读副本, 因为它们可能会尝试执行写入操作。
    同样, 服务器也不允许调用脚本`FCALL_RO`/`EVAL_RO`.
    最后, 当数据持久性由于磁盘错误而面临风险时, 也会阻止执行。

    使用此标志允许执行脚本：

    1.  跟`FCALL_RO`/`EVAL_RO`
    2.  在只读副本上。
    3.  即使存在磁盘错误 (Redis 无法持久存在, 因此会拒绝写) ）。
    4.  当超过内存限制时, 因为它意味着脚本不会增加内存消耗 (请参阅`allow-oom`) ）

    但是, 请注意, 如果脚本尝试调用写入命令, 服务器将返回错误。
    另请注意, 目前`PUBLISH`,`SPUBLISH`和`PFCOUNT`在脚本中也被视为写入命令, 因为它们可以尝试将命令传播到副本和 AOF 文件。

    欲了解更多信息, 请参阅[只读脚本](/docs/manual/programmability/#read-only_scripts)

*   `allow-oom`：使用此标志允许脚本在服务器内存不足  (OO) ） 时执行。

    除非使用, 否则 Redis 将拒绝执行标记的脚本 (函数和评估脚本[家当](/topics/eval-intro#eval-flags) ） 处于 OOM 状态时。
    此外, 使用此标志时, 脚本可以调用任何 Redis 命令, 包括在此状态下通常不允许的命令。
    指定`no-writes`或使用`FCALL_RO`/`EVAL_RO`还意味着脚本可以在 OOM 状态下运行 (无需指定`allow-oom`)

*   `allow-stale`：一个标志, 允许运行标记的脚本 (函数和评估脚本[家当](/topics/eval-intro#eval-flags) ） 针对过时的复制副本, 当`replica-serve-stale-data`配置设置为`no`.

    可以设置 Redis, 通过让陈旧的副本返回运行时错误来防止数据一致性问题使用旧数据。
    对于不访问数据的脚本, 可以设置此标志以允许陈旧的 Redis 副本运行脚本。
    但请注意, 脚本仍将无法执行任何访问过时数据的命令。

*   `no-cluster`：该标志会导致脚本在 Redis 集群模式下返回错误。

    Redis 允许在独立模式和集群模式下执行脚本。
    设置此标志可防止对群集中的节点执行脚本。

*   `allow-cross-slot-keys`：允许脚本从多个插槽访问密钥的标志。

    Redis 通常会阻止任何单个命令访问散列到多个槽的密钥。
    此标志允许脚本中断此规则并访问脚本中访问多个槽的密钥。
    脚本的声明密钥仍始终需要散列到单个插槽。
    不鼓励从多个插槽访问密钥, 因为应用程序应设计为一次仅从单个插槽访问密钥, 从而允许在 Redis 服务器之间移动插槽。

    禁用群集模式时, 此标志不起作用。

请参考[函数标志](/docs/manual/programmability/functions-intro/#function-flags)和[评估标志](/docs/manual/programmability/eval-intro/#eval-flags)作为详细示例。

### <a name="redis.redis_version"></a> `redis.REDIS_VERSION`

*   自版本以来： 7.0.0
*   在脚本中可用：是
*   功能可用：是

以 Lua 字符串的形式返回当前 Redis 服务器版本。
回复的格式为`MM.mm.PP`哪里：

*   **毫米：**是主要版本。
*   **毫米：**是次要版本。
*   **聚丙烯：**是修补程序级别。

### <a name="redis.redis_version_num"></a> `redis.REDIS_VERSION_NUM`

*   自版本以来： 7.0.0
*   在脚本中可用：是
*   功能可用：是

以数字形式返回当前 Redis 服务器版本。
回复是一个十六进制值, 结构为`0x00MMmmPP`哪里：

*   **毫米：**是主要版本。
*   **毫米：**是次要版本。
*   **聚丙烯：**是修补程序级别。

## 数据类型转换

除非引发运行时异常, `redis.call()`和`redis.pcall()`将已执行命令的回复返回到 Lua 脚本。
Redis从这些函数的回复会自动转换为Lua的本机数据类型。

同样, 当 Lua 脚本返回带有`return`关键词
该回复将自动转换为 Redis 的协议。

换个说法;Redis的回复和Lua的数据类型之间存在一对一的映射, Lua的数据类型与Lua的数据类型之间存在一对一的映射。[Redis Protocol](/topics/protocol)数据类型。
基础设计是这样的, 如果将Redis类型转换为Lua类型并转换回Redis类型, 则结果与初始值相同。

从 Redis 协议回复 (即来自`redis.call()`和`redis.pcall()`到 Lua 的数据类型取决于脚本使用的 Redis 序列化协议版本。
脚本执行期间的默认协议版本是 RESP2。
该脚本可以通过调用`redis.setresp()`功能。

从脚本返回的 Lua 数据类型进行类型转换取决于用户选择的协议 (请参阅`HELLO`命) ）。

以下部分介绍了每个协议版本在 Lua 和 Redis 之间的类型转换规则。

### RESP2 到 Lua 类型转换

默认情况下, 以下类型转换规则适用于执行的上下文以及调用后`redis.setresp(2)`:

*   [RESP2 整数回复](/topics/protocol#resp-integers)-> 路亚数
*   [RESP2 批量字符串回复](/topics/protocol#resp-bulk-strings)-> Lua 字符串
*   [RESP2 阵列回复](/topics/protocol#resp-arrays)-> Lua 表 (可能嵌套了其他 Redis 数据类) ）
*   [RESP2 状态回复](/topics/protocol#resp-simple-strings)-> Lua 表与单*还行*包含状态字符串的字段
*   [RESP2 错误回复](/topics/protocol#resp-errors)-> Lua 表与单*犯 错*包含错误字符串的字段
*   [RESP2 空批量回复](/topics/protocol#null-elements-in-arrays)和[空多批量回复](/topics/protocol#resp-arrays)-> Lua 假布尔类型

## Lua 到 RESP2 类型转换

默认情况下以及用户调用后, 以下类型转换规则将适用`HELLO 2`:

*   Lua 数字 ->[RESP2 整数回复](/topics/protocol#resp-integers) (数字转换为整) ）
*   Lua 字符串 ->[RESP 批量字符串回复](/topics/protocol#resp-bulk-strings)
*   Lua 表 (索引, 非关联数) ）->[RESP2 阵列回复](/topics/protocol#resp-arrays) (在第一个 Lua 处截断`nil`表中遇到的值 () 果有）
*   Lua 表与单个*还行*字段 ->[RESP2 状态回复](/topics/protocol#resp-simple-strings)
*   Lua 表与单个*犯 错*字段 ->[RESP2 错误回复](/topics/protocol#resp-errors)
*   Lua 布尔假 ->[RESP2 空批量回复](/topics/protocol#null-elements-in-arrays)

还有一个附加的 Lua 到 Redis 转换规则, 它没有相应的 Redis 到 Lua 转换规则：

*   Lua 布尔值`true`->[RESP2 整数回复](/topics/protocol#resp-integers)值为 1。

关于将 Lua 转换为 Redis 数据类型, 还有三个附加规则需要注意：

*   Lua有一个单一的数字类型, Lua数字。
    整数和浮点数之间没有区别。
    因此, 我们始终将Lua数字转换为整数回复, 删除数字的小数部分 (如果) ）。
    **如果要返回 Lua 浮点数, 则应将其作为字符串返回**,
    与 Redis 本身完全相同 (例如, 请参阅`ZSCORE`命) ）。
*   有[没有简单的方法可以在Lua数组中拥有nils](http://www.lua.org/pil/19.1.html)由于
    到 Lua 的表语义。
    因此, 当 Redis 将 Lua 数组转换为 RESP 时, 当遇到 Lua 时, 转换将停止`nil`价值。
*   当 Lua 表是包含键及其各自值的关联数组时, 转换后的 Redis 回复将**不**包括它们。

Lua 到 RESP2 类型转换示例：

    redis> EVAL "return 10" 0
    (integer) 10

    redis> EVAL "return { 1, 2, { 3, 'Hello World!' } }" 0
    1) (integer) 1
    2) (integer) 2
    3) 1) (integer) 3
       1) "Hello World!"

    redis> EVAL "return redis.call('get','foo')" 0
    "bar"

最后一个示例演示如何接收和返回确切的返回值`redis.call()` (或`redis.pcall()) ） 中, 因为如果直接调用了命令, 则会返回该命令。

下面的示例演示如何处理连接 nils 和键的浮点数和数组：

    redis> EVAL "return { 1, 2, 3.3333, somekey = 'somevalue', 'foo', nil , 'bar' }" 0
    1) (integer) 1
    2) (integer) 2
    3) (integer) 3
    4) "foo"

如您所见, 浮点值*3.333*转换为整数*3*这*某键*键及其值被省略, 并且字符串“bar”不返回, 因为存在`nil`前面的值。

### RESP3 到 Lua 类型转换

[RESP3](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md)是的较新版本[Redis 序列化协议](/topics/protocol).
从 Redis v6.0 开始, 它可作为选择加入选项提供。

正在执行的脚本可以调用[`redis.setresp`](#redis.setresp)函数在其执行期间, 并切换用于从 Redis 命令返回回复的协议版本 (可通过[`redis.call()`](#redis.call)或[`redis.pcall()`](#redis.pcall)).

一旦 Redis 的回复采用 RESP3 协议, 所有[RESP2 到 Lua 的转换](#resp2-to-lua-type-conversion)规则适用, 但增加了以下内容：

*   [RESP3 地图回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#map-type)-> Lua 表与单*地图*字段, 其中包含表示地图的字段和值的 Lua 表。
*   [注册教育储蓄计划集回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#set-reply)-> Lua 表与单*设置*字段包含一个 Lua 表, 该表将集合的元素表示为字段, 每个字段的 Lua 布尔值为`true`.
*   [RESP3 空值](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#null-reply)->鲁阿`nil`.
*   [RESP3 真实回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#boolean-reply)-> Lua 真布尔值。
*   [RESP3 错误回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#boolean-reply)-> Lua 错误布尔值。
*   [RESP3 双重回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#double-type)-> Lua 表与单*得分*字段包含表示双精度值的 Lua 数字。
*   [RESP3 大数字回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#big-number-type)-> Lua 表与单*big_number*字段包含表示大数字值的 Lua 字符串。
*   [Redis 逐字字符串回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#verbatim-string-type)-> Lua 表与单*verbatim_string*包含具有两个字段的 Lua 表的字段, *字符串*和*格式*, 分别表示逐字字符串及其格式。

**注意：**
RESP3[大数字](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#big-number-type)和[逐字字符串](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#verbatim-string-type)仅从 Redis v7.0 及更高版本开始支持回复。
此外, 目前, RESP3的[属性](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#attribute-type),[流字符串](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#streamed-strings)和[流聚合数据类型](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#streamed-aggregate-data-types)不受 Redis Lua API 支持。

### Lua 到 RESP3 类型转换

无论脚本选择的协议版本设置为使用 \[`redis.setresp()`函数] 当它调用`redis.call()`或`redis.pcall()`, 用户可以选择使用 RESP3 (使用`HELLO 3`命) ）表示连接。
尽管传入客户端连接的默认协议是 RESP2, 但脚本应遵循用户的首选项并返回充分类型的 RESP3 回复, 因此以下规则适用于[Lua 到 RESP2 类型转换](#lua-to-resp2-type-conversion)部分, 如果是这种情况。

*   Lua 布尔值 ->[RESP3 布尔回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#boolean-reply) (请注意, 与 RESP2 相比, 这是一个变化, 在 RESP2 中返回布尔 Lua`true`已将数字 1 返回到 Redis 客户端, 并返回`false`用于返回`null`.
*   Lua 表与单个*地图*设置为关联 Lua 表的字段 ->[RESP3 地图回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#map-type).
*   将单个\_set字段设置为关联 Lua 表的 Lua 表 ->[RESP3 设置回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#set-type).值可以设置为任何值, 无论如何都会被丢弃。
*   Lua 表与单个*双*字段到关联 Lua 表 ->[RESP3 双重回复](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#double-type).
*   Lua nil ->[RESP3 空值](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md#null-reply).

但是, 如果使用 RESP2 协议设置连接, 并且即使脚本使用 RESP3 类型的响应进行回复, Redis 也会自动执行回复的 RESP3 到 RESP2 转换, 就像常规命令一样。
这意味着, 例如, 将 RESP3 映射类型返回到 RESP2 连接将导致 repy 转换为由交替字段名称及其值组成的平面 RESP2 数组, 而不是 RESP3 映射。

## 有关脚本的其他说明

### 用`SELECT`在脚本中

您可以致电`SELECT`命令来自 Lua 脚本, 就像任何普通的客户端连接一样。
但是, 该行为的一个微妙方面在 Redis 版本 2.8.11 和 2.8.12 之间发生了变化。
在 Redis 版本 2.8.12 之前, Lua 脚本选择的数据库是*设置为当前数据库*对于已调用它的客户端连接。
从 Redis 版本 2.8.12 开始, Lua 脚本选择的数据库仅影响脚本的执行上下文, 不会修改调用脚本的客户端选择的数据库。
补丁级别版本之间的这种语义变化是必需的, 因为旧行为本质上与Redis的复制不兼容, 并引入了错误。

## 运行时库

Redis Lua 运行时上下文始终附带几个预导入的库。

以下[标准 Lua 库](https://www.lua.org/manual/5.1/manual.html#5)可供使用：

*   这[*字符串操作 (字符) ）*图书馆](https://www.lua.org/manual/5.1/manual.html#5.4)
*   这[*表操作 () ）*图书馆](https://www.lua.org/manual/5.1/manual.html#5.5)
*   这[*数学函数 (数) ）*图书馆](https://www.lua.org/manual/5.1/manual.html#5.6)

此外, 脚本还可以加载和访问以下外部库：

*   这[*结构*图书馆](#struct-library)
*   这[*cjson*图书馆](#cjson-library)
*   这[*cmsgpack*图书馆](#cmsgpack-library)
*   这[*比托普*图书馆](#bitop-library)

### <a name="struct-library"></a> *结构*图书馆

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

*结构*是一个库, 用于在Lua中打包和分解类似C的结构。
它提供以下功能：

*   [`struct.pack()`](#struct.pack)
*   [`struct.unpack()`](#struct.unpack)
*   [`struct.size()`](#struct.size)

所有*结构*的函数期望它们的第一个参数是[格式字符串](#struct-formats).

#### <a name="struct-formats"></a> *结构*格式

以下是 的有效格式字符串*结构*的函数：

*   `>`： 大端序
*   `<`： 小端序
*   `![num]`：对齐
*   `x`：填充
*   `b/B`：有符号/无符号字节
*   `h/H`：有符号/无符号短
*   `l/L`：有符号/无符号长
*   `T`： size_t
*   `i/In`：带大小的有符号/无符号整数*n* (默认为整型的大) ）
*   `cn`：序列*n*字符 (从/到字符) ）;打包时, n == 0 表示
    整个字符串;解包时, n == 0 表示使用先前读取的数字作为
    字符串的长度。
*   `s`：以零结尾的字符串
*   `f`：浮子
*   `d`： 双
*   ` ` (空) ）：忽略

#### <a name="struct.pack"></a> `struct.pack(x)`

此函数从值返回结构编码的字符串。
它接受[*结构*格式字符串](#struct-formats)作为其第一个参数, 后跟要编码的值。

用法示例：

    redis> EVAL "return struct.pack('HH', 1, 2)" 0
    "\x01\x00\x02\x00"

#### <a name="struct.unpack"></a> `struct.unpack(x)`

此函数从结构返回解码的值。
它接受[*结构*格式字符串](#struct-formats)作为其第一个参数, 后跟编码结构的字符串。

用法示例：

    redis> EVAL "return { struct.unpack('HH', ARGV[1]) }" 0 "\x01\x00\x02\x00"
    1) (integer) 1
    2) (integer) 2
    3) (integer) 5

#### <a name="struct.size"></a> `struct.size(x)`

此函数返回结构的大小 (以字节为单) ）。
它接受[*结构*格式字符串](#struct-formats)作为其唯一的参数。

用法示例：

    redis> EVAL "return struct.size('HH')" 0
    (integer) 4

### <a name="cjson-library"></a> *cjson*图书馆

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

这*cjson*库提供快速[断续器](https://json.org)从 Lua 进行编码和解码。
它提供了这些功能。

#### <a name="cjson.encode()"></a> `cjson.encode(x)`

此函数返回作为其参数提供的 Lua 数据类型的 JSON 编码字符串。

用法示例：

    redis> EVAL "return cjson.encode({ ['foo'] = 'bar' })" 0
    "{\"foo\":\"bar\"}"

#### <a name="cjson.decode()"></a> `cjson.decode(x)`

此函数从作为其参数提供的 JSON 编码字符串返回 Lua 数据类型。

用法示例：

    redis> EVAL "return cjson.decode(ARGV[1])['foo']" 0 '{"foo":"bar"}'
    "bar"

### <a name="cmsgpack-library"></a> *cmsgpack*图书馆

*   自版本以来： 2.6.0
*   在脚本中可用：是
*   功能可用：是

这*cmsgpack*库提供快速[留言包](https://msgpack.org/index.html)从 Lua 进行编码和解码。
它提供了这些功能。

#### <a name="cmsgpack.pack()"></a> `cmsgpack.pack(x)`

此函数返回 Lua 数据类型的打包字符串编码, 它作为参数给出。

用法示例：

    redis> EVAL "return cmsgpack.pack({'foo', 'bar', 'baz'})" 0
    "\x93\xa3foo\xa3bar\xa3baz"

#### <a name="cmsgpack.unpack()"></a> `cmsgpack.unpack(x)`

此函数通过解码其输入字符串参数返回解压缩的值。

用法示例：

    redis> EVAL "return cmsgpack.unpack(ARGV[1])" 0 "\x93\xa3foo\xa3bar\xa3baz"
    1) "foo"
    2) "bar"
    3) "baz"

### <a name="bitop-library"></a> *位*图书馆

*   自版本以来： 2.8.18
*   在脚本中可用：是
*   功能可用：是

这*位*库提供对数字的按位运算。
其文档驻留在[Lua BitOp 文档](http://bitop.luajit.org/api.html)
它提供以下功能。

#### <a name="bit.tobit()"></a> `bit.tobit(x)`

将数字规范化为位操作的数字范围并返回该数字范围。

用法示例：

    redis> EVAL 'return bit.tobit(1)' 0
    (integer) 1

#### <a name="bit.tohex()"></a> `bit.tohex(x [,n])`

将其第一个参数转换为十六进制字符串。十六进制位数由可选 second 参数的绝对值给出。

用法示例：

    redis> EVAL 'return bit.tohex(422342)' 0
    "000671c6"

#### <a name="bit.bnot()"></a> `bit.bnot(x)`

按位返回**不**其论点。

#### <a name="bit.ops"></a> `bit.bnot(x)` `bit.bor(x1 [,x2...])`,`bit.band(x1 [,x2...])`和`bit.bxor(x1 [,x2...])`

按位返回**或**位**和**或按位**异或**其所有论点。
请注意, 允许使用两个以上的参数。

用法示例：

    redis> EVAL 'return bit.bor(1,2,4,8,16,32,64,128)' 0
    (integer) 255

#### <a name="bit.shifts"></a> `bit.lshift(x, n)`,`bit.rshift(x, n)`和`bit.arshift(x, n)`

返回按位逻辑**左移**、按位逻辑**右移**或按位**算术右移**其第一个参数的位数由第二个参数给出的位数。

#### <a name="bit.ro"></a> `bit.rol(x, n)`和`bit.ror(x, n)`

按位返回**左旋转**或按位**右旋**其第一个参数的位数由第二个参数给出的位数。
在一侧移出的位在另一侧移回。

#### <a name="bit.bswap()"></a> `bit.bswap(x)`

交换其参数的字节并返回它。
这可用于将小端 32 位数字转换为大端 32 位数字, 反之亦然。
