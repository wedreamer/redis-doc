---
title: "Redis functions"
linkTitle: "Functions"
weight: 1
description: >
   Scripting with Redis 7 and beyond
aliases:
    - /topics/functions-intro
---

Redis Functions 是一个 API, 用于管理要在服务器上执行的代码。此功能在 Redis 7 中可用, 取代了[评估](/docs/manual/programmability/eval-intro)在Redis的先前版本中。

## 序幕 (或者, Eval脚本有什么问题？

以前版本的 Redis 使脚本只能通过`EVAL`命令, 它允许发送 Lua 脚本以供服务器执行。
核心用例[评估脚本](/topics/eval-intro)在 Redis 中以高效和原子的方式执行部分应用程序逻辑。
此类脚本可以跨多个键执行条件更新, 可能组合了几种不同的数据类型。

用`EVAL`要求应用程序每次都发送整个脚本以供执行。
由于这会导致网络和脚本编译开销, 因此 Redis 以`EVALSHA`命令。通过首次呼叫`SCRIPT LOAD`要获取脚本的 SHA1, 应用程序可以在之后仅使用其摘要重复调用它。

根据设计, Redis 仅缓存加载的脚本。
这意味着脚本缓存可能随时丢失, 例如在调用后`SCRIPT FLUSH`, 则在重新启动服务器后或故障转移到副本时。
如果缺少脚本, 应用程序负责在运行时重新加载脚本。
基本假设是脚本是应用程序的一部分, 而不是由 Redis 服务器维护的。

这种方法适合许多轻量级脚本用例, 但是一旦应用程序变得复杂并且更依赖于脚本, 就会引入一些困难, 即：

1.  所有客户端应用程序实例都必须维护所有脚本的副本。这意味着要有一些机制将脚本更新应用于应用程序的所有实例。
2.  在[交易](/topics/transactions)增加事务因缺少脚本而失败的可能性。更有可能失败会使使用缓存脚本作为工作流的构建块变得不那么有吸引力。
3.  SHA1 摘要毫无意义, 使得调试系统非常困难 (例如, 在`MONITOR`会话) 。
4.  当天真地使用时, `EVAL`促进一种反模式, 在这种模式中, 客户端应用程序呈现逐字脚本, 而不是负责任地使用[`!KEYS`和`ARGV`Lua API](/topics/lua-api#runtime-globals).
5.  因为它们是短暂的, 所以一个脚本不能调用另一个脚本。这使得在脚本之间共享和重用代码几乎是不可能的, 而不是客户端预处理 (请参阅第一点) 。

为了满足这些需求, 同时避免对已经建立和备受喜爱的临时脚本进行重大更改, Redis v7.0 引入了 Redis Functions。

## 什么是 Redis 函数？

Redis 函数是从临时脚本编写的进化步骤。

函数提供与脚本相同的核心功能, 但它们是数据库的一流软件工件。
Redis 将功能作为数据库的一个组成部分进行管理, 并通过数据持久化和复制确保其可用性。
由于函数是数据库的一部分, 因此在使用前声明, 因此应用程序不需要在运行时加载它们, 也不会面临事务中止的风险。
使用函数的应用程序仅依赖于其 API, 而不依赖于数据库中的嵌入式脚本逻辑。

临时脚本被视为应用程序域的一部分, 而函数则使用用户提供的逻辑扩展数据库服务器本身。
它们可用于公开由核心 Redis 命令组成的更丰富的 API, 类似于模块, 开发一次, 在启动时加载, 并由各种应用程序/客户端重复使用。
每个函数都有一个唯一的用户定义名称, 这使得调用和跟踪其执行变得更加容易。

Redis Functions的设计还试图在用于编写函数的编程语言和服务器管理它们之间划分界限。
Lua是Redis目前作为嵌入式执行引擎支持的唯一语言解释器, 它意味着简单易学。
然而, 选择Lua作为一种语言仍然给许多Redis用户带来了挑战。

Redis Functions 特性不对实现的语言做任何假设。
作为函数定义的一部分的执行引擎处理运行它。
理论上, 只要引擎遵守多个规则 (例如终止执行函数的能力) , 就可以使用任何语言执行函数。

目前, 如上所述, Redis附带了一个嵌入式[Lua 5.1](/topics/lua-api)发动机。
计划在未来支持其他引擎。
Redis函数可以使用Lua的所有可用功能来临时脚本, 
唯一的例外是[Redis Lua scripts debugger](/topics/ldb).

函数还通过启用代码共享来简化开发。
每个函数都属于一个库, 任何给定的库都可以由多个函数组成。
库的内容是不可变的, 并且不允许有选择地更新其函数。
相反, 库作为一个整体进行更新, 其所有功能都在一个操作中。
这允许从同一库中的其他函数调用函数, 或者通过使用库内部方法中的公共代码在函数之间共享代码, 该方法也可以采用语言本机参数。

如上所述, 函数旨在更好地支持通过逻辑架构维护数据实体的一致视图的用例。
因此, 函数与数据本身一起存储。
函数还保存到 AOF 文件, 并从主文件复制到副本, 因此它们与数据本身一样持久。
当 Redis 用作临时缓存时, 需要其他机制 (如下所述) 来使函数更持久。

与 Redis 中的所有其他操作一样, 函数的执行是原子的。
函数的执行会在整个时间段内阻止所有服务器活动, 类似于[交易](/topics/transactions).
这些语义意味着脚本的所有效果要么尚未发生, 要么已经发生。
已执行函数的阻塞语义始终适用于所有连接的客户端。
由于运行函数会阻止 Redis 服务器, 因此函数应快速完成执行, 因此应避免使用长时间运行的函数。

## 加载库和函数

让我们通过一些有形的示例和Lua片段来探索Redis函数。

在这一点上, 如果你不熟悉Lua, 特别是在Redis中, 你可能会从查看中的一些例子中受益。[评估脚本简介](/topics/eval-intro)和[Lua API](/topics/lua-api)页面, 以便更好地掌握语言。

每个 Redis 函数都属于加载到 Redis 的单个库。
将库加载到数据库是使用`FUNCTION LOAD`命令。
该命令获取库有效负载作为输入, 
库负载必须以 Shebang 语句开头, 该语句提供有关库的元数据 (如要使用的引擎和库名称) 。
Shebang 格式为：

    #!<engine name> name=<library name>

让我们尝试加载一个空库：

    redis> FUNCTION LOAD "#!lua name=mylib\n"
    (error) ERR No functions registered

该错误是预期的, 因为加载的库中没有函数。每个库都需要包含至少一个已注册的函数才能成功加载。
已注册的函数已命名, 并充当库的入口点。
当目标执行引擎处理`FUNCTION LOAD`命令, 它注册库的函数。

Lua 引擎在加载时编译和评估库源代码, 并期望通过调用`redis.register_function()`应用程序接口。

以下代码片段演示了一个简单的库, 该库注册了一个名为*诺克诺克*, 返回字符串回复：

```lua
#!lua name=mylib
redis.register_function(
  'knockknock',
  function() return 'Who\'s there?' end
)
```

在上面的例子中, 我们向Lua的函数提供了两个参数`redis.register_function()`API：其注册名称和回调。

我们可以加载我们的库并使用`FCALL`调用已注册的函数：

    redis> FUNCTION LOAD "#!lua name=mylib\nredis.register_function('knockknock', function() return 'Who\\'s there?' end)"
    mylib
    redis> FCALL knockknock 0
    "Who's there?"

请注意, `FUNCTION LOAD`命令返回已加载库的名称, 此名称以后可以使用`FUNCTION LIST`和`FUNCTION DELETE`.

我们提供`FCALL`具有两个参数：函数的注册名称和数值`0`.此数值指示其后面的键名数 (以相同的方式) `EVAL`和`EVALSHA`工作) 。

我们将立即解释如何使函数可以使用键名和其他参数。由于这个简单的示例不涉及键, 因此我们现在只使用 0。

## 输入键和常规参数

在我们转到以下示例之前, 了解 Redis 在作为键名称的参数和非键名称的参数之间的区别至关重要。

虽然 Redis 中的键名只是字符串, 但与任何其他字符串值不同, 这些值表示数据库中的键。
密钥的名称是 Redis 中的基本概念, 也是运行 Redis 集群的基础。

**重要：**
为了确保在独立部署和集群部署中正确执行 Redis 函数, 函数访问的所有键名称都必须显式提供为输入键参数。

函数的任何不是键名称的输入都是常规输入参数。

现在, 让我们假设我们的应用程序将其部分数据存储在 Redis 哈希中。
我们想要一个`HSET`- 类似于在所述哈希中设置和更新字段并将上次修改时间存储在名为的新字段中的方法`_last_modified_`.
我们可以实现一个函数来完成所有这些操作。

我们的函数将调用`TIME`获取服务器的时钟读数, 并使用新字段的值和修改的时间戳更新目标哈希。
我们将实现的函数接受以下输入参数：哈希的键名和要更新的字段值对。

用于 Redis Functions 的 Lua API 使这些输入可作为函数回调的第一个和第二个参数进行访问。
回调的第一个参数是一个 Lua 表, 其中填充了函数输入的所有键名。
同样, 回调的第二个参数由所有常规参数组成。

以下是我们的函数及其库注册的可能实现：

```lua
#!lua name=mylib

local function my_hset(keys, args)
  local hash = keys[1]
  local time = redis.call('TIME')[1]
  return redis.call('HSET', hash, '_last_modified_', time, unpack(args))
end

redis.register_function('my_hset', my_hset)
```

如果我们创建一个名为*我的.lua*它由库的定义组成, 我们可以像这样加载它 (不去除源代码中有用的空格) ：

```bash
$ cat mylib.lua | redis-cli -x FUNCTION LOAD REPLACE
```

我们添加了`REPLACE`对调用的修饰符`FUNCTION LOAD`告诉 Redis 我们要覆盖现有的库定义。
否则, 我们会从Redis那里得到一个错误, 抱怨该库已经存在。

现在, 库的更新代码已加载到 Redis, 我们可以继续调用我们的函数：

    redis> FCALL my_hset 1 myhash myfield "some value" another_field "another value"
    (integer) 3
    redis> HGETALL myhash
    1) "_last_modified_"
    2) "1640772721"
    3) "myfield"
    4) "some value"
    5) "another_field"
    6) "another value"

在本例中, 我们调用了`FCALL`跟*1*作为键名参数的数量。
这意味着函数的第一个输入参数是键的名称 (因此包含在回调的`keys`表) 。
在第一个参数之后, 所有后续输入参数都被视为常规参数, 并构成`args`表作为回调的第二个参数传递给回调。

## 扩展库

我们可以向库中添加更多函数, 以使应用程序受益。
在访问哈希数据时, 我们添加到哈希中的其他元数据字段不应包含在响应中。
另一方面, 我们确实希望提供获取给定哈希键的修改时间戳的方法。

我们将向库中添加两个新函数来实现这些目标：

1.  这`my_hgetall`Redis 函数将从给定的哈希键名称返回所有字段及其各自的值, 但不包括元数据 (即`_last_modified_`字段) 。
2.  这`my_hlastmodified`Redis 函数将返回给定哈希键名称的修改时间戳。

库的源代码可能如下所示：

```lua
#!lua name=mylib

local function my_hset(keys, args)
  local hash = keys[1]
  local time = redis.call('TIME')[1]
  return redis.call('HSET', hash, '_last_modified_', time, unpack(args))
end

local function my_hgetall(keys, args)
  redis.setresp(3)
  local hash = keys[1]
  local res = redis.call('HGETALL', hash)
  res['map']['_last_modified_'] = nil
  return res
end

local function my_hlastmodified(keys, args)
  local hash = keys[1]
  return redis.call('HGET', keys[1], '_last_modified_')
end

redis.register_function('my_hset', my_hset)
redis.register_function('my_hgetall', my_hgetall)
redis.register_function('my_hlastmodified', my_hlastmodified)
```

虽然上述所有内容都应该很简单, 但请注意`my_hgetall`还调用[`redis.setresp(3)`](/topics/lua-api#redis.setresp).
这意味着该函数期望[RESP3](https://github.com/redis/redis-specifications/blob/master/protocol/RESP3.md)呼叫后回复`redis.call()`, 与默认的 RESP2 协议不同, 该协议提供字典 (关联数组) 应答。
这样做允许函数删除 (或设置为`nil`就像Lua表的情况一样) 来自回复的特定字段, 在我们的例子中, `_last_modified_`田。

假设您已将库的实现保存在*我的.lua*文件中, 您可以将其替换为：

```bash
$ cat mylib.lua | redis-cli -x FUNCTION LOAD REPLACE
```

加载后, 您可以使用`FCALL`:

    redis> FCALL my_hgetall 1 myhash
    1) "myfield"
    2) "some value"
    3) "another_field"
    4) "another value"
    redis> FCALL my_hlastmodified 1 myhash
    "1640772721"

您还可以通过`FUNCTION LIST`命令：

    redis> FUNCTION LIST
    1) 1) "library_name"
       2) "mylib"
       3) "engine"
       4) "LUA"
       5) "functions"
       6) 1) 1) "name"
             2) "my_hset"
             3) "description"
             4) (nil)
          2) 1) "name"
             2) "my_hgetall"
             3) "description"
             4) (nil)
          3) 1) "name"
             2) "my_hlastmodified"
             3) "description"
             4) (nil)

您可以看到, 使用新功能更新我们的库很容易。

## 重用库中的代码

除了将函数捆绑到数据库管理的软件工件中之外, 库还促进了代码共享。
我们可以向库中添加一个从其他函数调用的错误处理帮助程序函数。
帮助程序函数`check_keys()`验证输入*钥匙*表具有单个键。
成功后, 它会返回`nil`, 否则它将返回[错误回复](/topics/lua-api#redis.error_reply).

更新后的库的源代码将是：

```lua
#!lua name=mylib

local function check_keys(keys)
  local error = nil
  local nkeys = table.getn(keys)
  if nkeys == 0 then
    error = 'Hash key name not provided'
  elseif nkeys > 1 then
    error = 'Only one key name is allowed'
  end

  if error ~= nil then
    redis.log(redis.LOG_WARNING, error);
    return redis.error_reply(error)
  end
  return nil
end

local function my_hset(keys, args)
  local error = check_keys(keys)
  if error ~= nil then
    return error
  end

  local hash = keys[1]
  local time = redis.call('TIME')[1]
  return redis.call('HSET', hash, '_last_modified_', time, unpack(args))
end

local function my_hgetall(keys, args)
  local error = check_keys(keys)
  if error ~= nil then
    return error
  end

  redis.setresp(3)
  local hash = keys[1]
  local res = redis.call('HGETALL', hash)
  res['map']['_last_modified_'] = nil
  return res
end

local function my_hlastmodified(keys, args)
  local error = check_keys(keys)
  if error ~= nil then
    return error
  end

  local hash = keys[1]
  return redis.call('HGET', keys[1], '_last_modified_')
end

redis.register_function('my_hset', my_hset)
redis.register_function('my_hgetall', my_hgetall)
redis.register_function('my_hlastmodified', my_hlastmodified)
```

将 Redis 中的库替换为上述内容后, 您可以立即尝试新的错误处理机制：

    127.0.0.1:6379> FCALL my_hset 0 myhash nope nope
    (error) Hash key name not provided
    127.0.0.1:6379> FCALL my_hgetall 2 myhash anotherone
    (error) Only one key name is allowed

您的 Redis 日志文件中应包含类似于以下内容的行：

    ...
    20075:M 1 Jan 2022 16:53:57.688 # Hash key name not provided
    20075:M 1 Jan 2022 16:54:01.309 # Only one key name is allowed

## 群集中的函数

如上所述, Redis 会自动处理加载的函数到副本的传播。
在 Redis 集群中, 还需要将函数加载到所有集群节点。Redis 集群不会自动处理这种情况, 需要由集群管理员处理 (如模块加载、配置设置等) 。

由于函数的目标之一是独立于客户端应用程序, 因此这不应成为 Redis 客户端库职责的一部分。相反`redis-cli --cluster-only-masters --cluster call host:port FUNCTION LOAD ...`可用于在所有主节点上执行 load 命令。

另外, 请注意`redis-cli --cluster add-node`自动注意将加载的函数从现有节点之一传播到新节点。

## 函数和临时 Redis 实例

在某些情况下, 可能需要启动一个预加载了一组函数的新Redis服务器。常见的原因可能是：

*   在新环境中启动 Redis
*   重新启动使用函数的临时 (仅缓存) Redis

在这种情况下, 我们需要确保在 Redis 接受入站用户连接和命令之前预加载的函数可用。

为此, 可以使用`redis-cli --functions-rdb`以从现有服务器中提取函数。这将生成一个 RDB 文件, 该文件可由 Redis 在启动时加载。

## 函数标志

Redis 需要获得一些关于函数在执行时将如何表现的信息, 以便正确实施资源使用策略并保持数据一致性。

例如, Redis 需要知道某个函数是只读的, 然后才能允许它使用`FCALL_RO`在只读副本上。

默认情况下, Redis 假定所有函数都可以执行任意读取或写入操作。函数标志使得在注册时声明更具体的函数行为成为可能。让我们看看它是如何工作的。

在前面的示例中, 我们定义了两个仅读取数据的函数。我们可以尝试使用`FCALL_RO`针对只读副本。

    redis > FCALL_RO my_hgetall 1 myhash
    (error) ERR Can not execute a function with write flag using fcall_ro.

Redis 返回此错误是因为从理论上讲, 函数可以对数据库执行读取和写入操作。
作为一种保护措施, 默认情况下, Redis 假设该函数同时执行这两项操作, 因此它会阻止其执行。
在以下情况下, 服务器将回复此错误：

1.  执行函数`FCALL`针对只读副本。
2.  用`FCALL_RO`以执行函数。
3.  检测到磁盘错误 (Redis 无法持久存在, 因此拒绝写入) 。

在这些情况下, 您可以添加`no-writes`标记到函数的注册, 禁用安全措施并允许它们运行。
要使用标志注册函数, 请使用[命名参数](/topics/lua-api#redis.register_function_named_args)的变体`redis.register_function`.

库中更新的注册码代码段如下所示：

```lua
redis.register_function('my_hset', my_hset)
redis.register_function{
  function_name='my_hgetall',
  callback=my_hgetall,
  flags={ 'no-writes' }
}
redis.register_function{
  function_name='my_hlastmodified',
  callback=my_hlastmodified,
  flags={ 'no-writes' }
}
```

一旦我们替换了库, Redis就允许同时运行这两个库。`my_hgetall`和`my_hlastmodified`跟`FCALL_RO`针对只读副本：

    redis> FCALL_RO my_hgetall 1 myhash
    1) "myfield"
    2) "some value"
    3) "another_field"
    4) "another value"
    redis> FCALL_RO my_hlastmodified 1 myhash
    "1640772721"

有关完整的文档标志, 请参阅[脚本标志](/topics/lua-api#script_flags).
