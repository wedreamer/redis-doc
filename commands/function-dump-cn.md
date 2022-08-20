返回已加载库的序列化负载。
您可以稍后使用`FUNCTION RESTORE`命令。

欲了解更多信息，请参阅[Redis 函数简介](/topics/functions-intro).

@return

@bulk字符串回复：序列化的有效负载

@examples

下面的示例演示如何使用`FUNCTION DUMP`然后它调用`FUNCTION FLUSH`删除所有库。
然后，它从序列化的有效负载中恢复原始库`FUNCTION RESTORE`.

    redis> FUNCTION DUMP
    "\xf6\x05mylib\x03LUA\x00\xc3@D@J\x1aredis.register_function('my@\x0b\x02', @\x06`\x12\x11keys, args) return`\x0c\a[1] end)\n\x00@\n)\x11\xc8|\x9b\xe4"
    redis> FUNCTION FLUSH
    OK
    redis> FUNCTION RESTORE "\xf6\x05mylib\x03LUA\x00\xc3@D@J\x1aredis.register_function('my@\x0b\x02', @\x06`\x12\x11keys, args) return`\x0c\a[1] end)\n\x00@\n)\x11\xc8|\x9b\xe4"
    OK
    redis> FUNCTION LIST
    1) 1) "library_name"
       2) "mylib"
       3) "engine"
       4) "LUA"
       5) "description"
       6) (nil)
       7) "functions"
       8) 1) 1) "name"
             2) "myfunc"
             3) "description"
             4) (nil)
