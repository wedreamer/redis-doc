删除库及其所有函数。

此命令将删除名为*库名称*以及其中的所有功能。
如果库不存在，服务器将返回错误。

欲了解更多信息，请参阅[Redis 函数简介](/topics/functions-intro).

@return

@simple字符串回复

@examples

    redis> FUNCTION LOAD Lua mylib "redis.register_function('myfunc', function(keys, args) return 'hello' end)"
    OK
    redis> FCALL myfunc 0
    "hello"
    redis> FUNCTION DELETE mylib
    OK
    redis> FCALL myfunc 0
    (error) ERR Function not found
