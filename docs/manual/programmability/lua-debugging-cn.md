***

标题： 在 Redis 中调试 Lua 脚本
链接标题： 调试 Lua
描述： 如何使用内置的 Lua 调试器
体重： 4
别名：

*   /topics/ldb

***

从版本3.2开始，Redis包括一个完整的Lua调试器，可以
用于使编写复杂的 Redis 脚本的任务更加简单。

代号为 LDB 的 Redis Lua 调试器具有以下重要功能：

*   它使用服务器-客户端模型，因此它是一个远程调试器。
    Redis 服务器充当调试服务器，而默认客户端是`redis-cli`.
    但是，可以通过遵循服务器实现的简单协议来开发其他客户端。
*   默认情况下，每个新的调试会话都是一个分叉会话。
    这意味着在调试 Redis Lua 脚本时，服务器不会阻塞，可用于开发或并行执行多个调试会话。
    这也意味着变化是**回滚**脚本调试会话完成后，可以使用与上一个调试会话完全相同的 Redis 数据集再次重新启动新的调试会话。
*   按需提供另一种同步（非分叉）调试模型，以便可以保留对数据集的更改。
    在此模式下，服务器在调试会话处于活动状态时进行阻止。
*   支持分步执行。
*   支持静态和动态断点。
*   支持将调试的脚本记录到调试器控制台中。
*   检查 Lua 变量。
*   跟踪脚本执行的 Redis 命令。
*   Redis和Lua值的漂亮打印。
*   无限循环和长执行检测，用于模拟断点。

## 快速入门

开始使用Lua调试器的一种简单方法是观看此视频
介绍：

<iframe width="560" height="315" src="https://www.youtube.com/embed/IMvRfStaoyM" frameborder="0" allowfullscreen></iframe>

> 重要提示：请确保避免使用 Redis 生产服务器调试 Lua 脚本。
> 请改用开发服务器。
> 另请注意，使用同步调试模式（不是默认设置）会导致 Redis 服务器在调试会话持续期间一直处于阻塞状态。

启动新的调试会话`redis-cli`请执行下列操作：

1.  使用您的首选编辑器在某个文件中创建脚本。假设您正在编辑位于`/tmp/script.lua`.
2.  通过以下方式启动调试会话：

    ./redis-cli --ldb --eval /tmp/script.lua

请注意，使用`--eval`选项`redis-cli`您可以将键名和参数传递给脚本，以逗号分隔，如以下示例所示：

    ./redis-cli --ldb --eval /tmp/script.lua mykey somekey , arg1 arg2

您将进入一个特殊模式，其中`redis-cli`不再接受其正常
命令，但改为打印帮助屏幕并通过未修改的调试
命令直接发送到 Redis。

唯一未传递到 Redis 调试器的命令是：

*   `quit`-- 这将终止调试会话。
    这就像删除所有断点并使用`continue`调试命令。
    此外，该命令将从`redis-cli`.
*   `restart`-- 调试会话将从头开始重新启动，**从文件重新加载新版本的脚本**.
    因此，正常的调试周期涉及在经过一些调试后修改脚本，并调用`restart`以便使用新脚本更改再次开始调试。
*   `help`-- 此命令传递给 Redis Lua 调试器，该调试器将打印如下所示的命令列表：

<!---->

    lua debugger> help
    Redis Lua debugger help:
    [h]elp               Show this help.
    [s]tep               Run current line and stop again.
    [n]ext               Alias for step.
    [c]continue          Run till next breakpoint.
    [l]list              List source code around current line.
    [l]list [line]       List source code around [line].
                         line = 0 means: current position.
    [l]list [line] [ctx] In this form [ctx] specifies how many lines
                         to show before/after [line].
    [w]hole              List all source code. Alias for 'list 1 1000000'.
    [p]rint              Show all the local variables.
    [p]rint <var>        Show the value of the specified variable.
                         Can also show global vars KEYS and ARGV.
    [b]reak              Show all breakpoints.
    [b]reak <line>       Add a breakpoint to the specified line.
    [b]reak -<line>      Remove breakpoint from the specified line.
    [b]reak 0            Remove all breakpoints.
    [t]race              Show a backtrace.
    [e]eval <code>       Execute some Lua code (in a different callframe).
    [r]edis <cmd>        Execute a Redis command.
    [m]axlen [len]       Trim logged Redis replies and Lua var dumps to len.
                         Specifying zero as <len> means unlimited.
    [a]abort             Stop the execution of the script. In sync
                         mode dataset changes will be retained.

    Debugger functions you can call from Lua scripts:
    redis.debug()        Produce logs in the debugger console.
    redis.breakpoint()   Stop execution as if there was a breakpoint in the
                         next line of code.

请注意，当您启动调试器时，它将在**步进模式**.
它将停止在执行脚本之前实际执行某些操作的第一行。

从这一点开始，您通常会致电`step`为了执行该行并转到下一行。
在您执行步骤时，Redis 将显示服务器执行的所有命令，如以下示例所示：

    * Stopped at 1, stop reason = step over
    -> 1   redis.call('ping')
    lua debugger> step
    <redis> ping
    <reply> "+PONG"
    * Stopped at 2, stop reason = step over

这`<redis>`和`<reply>`行显示由行执行的命令只是
已执行，并且来自服务器的答复。请注意，这仅在步进模式下发生。
如果您使用`continue`为了执行脚本直到下一个断点，命令不会转储到屏幕上，以防止输出过多。

## 终止调试会话

当脚本自然终止时，调试会话结束，并且
`redis-cli`在其正常的非调试模式下返回。您可以重新启动
使用`restart`命令照常。

停止调试会话的另一种方法是中断`redis-cli`
手动按`Ctrl+C`.请注意，任何事件也会破坏
关系`redis-cli`和`redis-server`将中断
调试会话。

服务器关闭时，所有分叉调试会话都将终止
下。

## 缩写调试命令

调试可能是一项非常重复的任务。出于这个原因，每个Redis
调试器命令以不同的字符开头，您可以使用单个
初始字符，以便引用命令。

例如，而不是键入`step`你可以直接输入`s`.

## 断点

添加和删除断点非常简单，如联机帮助中所述。
只需使用`b 1 2 3 4`在第 1、2、3、4 行中添加断点。
命令`b 0`删除所有断点。选定的断点可以是
删除使用我们要删除的断点所在的行作为参数，但以减号为前缀。
例如`b -3`从第 3 行中删除断点。

请注意，向 Lua 从不执行的行添加断点（如局部变量的声明或注释）将不起作用。
将添加断点，但由于脚本的这一部分永远不会执行，因此程序永远不会停止。

## 动态断点

使用`breakpoint`命令 可以将断点添加到特定
线。但是，有时我们只想停止程序的执行
当特殊的事情发生时。为此，您可以使用
`redis.breakpoint()`函数位于 Lua 脚本中。调用时模拟
下一行中将要执行的断点。

    if counter > 10 then redis.breakpoint() end

这个功能在调试时非常有用，这样我们就可以避免
继续多次手动执行脚本，直到达到给定条件
遇到。

## 同步模式

如前所述，但默认 LDB 使用带有回滚功能的分叉会话
脚本在调试时操作的所有数据更改。
在调试过程中，确定性通常是一件好事，因此连续
调试会话可以启动，而无需重置数据库内容
到其原始状态。

但是，为了跟踪某些错误，您可能希望保留执行的更改
到每个调试会话的键空间。当这是一个好主意你
应使用特殊选项启动调试器，`ldb-sync-mode`在`redis-cli`.

    ./redis-cli --ldb-sync-mode --eval /tmp/script.lua

> 注意：在此模式下的调试会话期间，Redis 服务器将无法访问，因此请谨慎使用。

在这种特殊模式下，`abort`命令可以停止脚本在中途将操作的更改带到数据集。
请注意，这与正常结束调试会话不同。
如果你只是打断`redis-cli`脚本将完全执行，然后会话终止。
取而代之的是`abort`您可以在中间中断脚本执行，并在需要时启动新的调试会话。

## 从脚本记录日志

这`redis.debug()`命令是一个功能强大的调试工具，可以
在 Redis Lua 脚本中调用，以便将内容记录到调试中
安慰：

    lua debugger> list
    -> 1   local a = {1,2,3}
       2   local b = false
       3   redis.debug(a,b)
    lua debugger> continue
    <debug> line 3: {1; 2; 3}, false

如果脚本在调试会话之外执行，`redis.debug()`根本没有效果。
请注意，该函数接受多个参数，这些参数在输出中以逗号和空格分隔。

正确显示表和嵌套表，以便使程序员调试脚本时可以轻松观察值。

## 检查程序状态`print`和`eval`

而`redis.debug()`函数可用于打印值
直接从Lua脚本内部，通常观察本地是有用的
单步执行或停止到断点时程序的变量。

这`print`命令就是这样做的，并在调用帧中执行查找
从当前一个回到以前的那个，直到顶层。
这意味着，即使我们进入Lua脚本中的嵌套函数，
我们仍然可以使用`print foo`查看`foo`在上下文中
的调用函数。在没有变量名称的情况下调用时，`print`将
打印所有变量及其各自的值。

这`eval`命令执行小段 Lua 脚本**当前调用框架的上下文之外**（使用当前 Lua 内部组件无法在当前调用帧的上下文中进行评估）。
但是，您可以使用此命令来测试 Lua 函数。

    lua debugger> e redis.sha1hex('foo')
    <retval> "0beec7b5ea3f0fdbc95d0dd47f3c5bc275da8a33"

## 调试客户端

LDB 使用客户端-服务器模型，其中 Redis 服务器充当调试服务器，该服务器使用[注册教育储蓄计划](/topics/protocol).而`redis-cli`是默认调试客户端，任何[客户](/clients)只要满足以下条件之一，就可以用于调试：

1.  客户端提供用于设置调试模式和控制调试会话的本机接口。
2.  客户端提供了一个接口，用于通过 RESP 发送任意命令。
3.  客户端允许将原始消息发送到 Redis 服务器。

例如，[Redis 插件](https://redis.com/blog/zerobrane-studio-plugin-for-redis-lua-scripts)为[零布兰一室公寓](http://studio.zerobrane.com/)与 LDB 集成使用[redis-lua](https://github.com/nrk/redis-lua).以下Lua代码是插件如何实现这一目标的简化示例：

```Lua
local redis = require 'redis'

-- add LDB's Continue command
redis.commands['ldbcontinue'] = redis.command('C')

-- script to be debugged
local script = [[
  local x, y = tonumber(ARGV[1]), tonumber(ARGV[2])
  local result = x * y
  return result
]]

local client = redis.connect('127.0.0.1', 6379)
client:script("DEBUG", "YES")
print(unpack(client:eval(script, 0, 6, 9)))
client:ldbcontinue()
```
