调用函数。

函数通过`FUNCTION LOAD`命令。
第一个参数是加载的函数的名称。

第二个参数是输入键名参数的数量，后跟函数访问的所有键。
在 Lua 中，这些输入键的名称作为表提供给函数，该表是回调的第一个参数。

**重要：**
为了确保在独立部署和群集部署中正确执行函数，函数访问的所有键名称都必须显式提供为输入键参数。
功能**应该只**访问键，其名称作为输入参数给出。
功能**永远不应该**访问键具有以编程方式生成的名称或基于存储在数据库中的数据结构的内容。

任何其他输入参数**不应该**表示键的名称。
这些是常规参数，在 Lua 表中作为回调的第二个参数传递。

有关更多信息，请参阅[Redis 可编程性](/topics/programmability)和[Redis 函数简介](/topics/functions-intro)页面。

@examples

下面的示例将创建一个名为`mylib`具有单个功能，`myfunc`，返回它得到的第一个参数。

    redis> FUNCTION LOAD "#!lua name=mylib \n redis.register_function('myfunc', function(keys, args) return args[1] end)"
    "mylib"
    redis> FCALL myfunc 0 hello
    "hello"
