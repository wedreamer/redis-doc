创建与值关联的键，该值是通过反序列化
提供的序列化值（通过以下方式获得）`DUMP`).

如果`ttl`为 0，则创建密钥时不会过期，否则指定
设置过期时间（以毫秒为单位）。

如果`ABSTTL`使用了修饰符，`ttl`应表示绝对值
[Unix 时间戳][hewowu]（以毫秒为单位），密钥将过期。

[hewowu]: http://en.wikipedia.org/wiki/Unix_time

出于驱逐目的，您可以使用`IDLETIME`或`FREQ`修饰 符。看
`OBJECT`了解更多信息。

`!RESTORE`在以下情况下将返回“目标键名称正忙”错误`key`已经
存在，除非您使用`REPLACE`修饰语。

`!RESTORE`检查 RDB 版本和数据校验和。
如果它们不匹配，则返回错误。

@return

@simple字符串回复：命令在成功时返回 OK。

@examples

    redis> DEL mykey
    0
    redis> RESTORE mykey 0 "\n\x17\x17\x00\x00\x00\x12\x00\x00\x00\x03\x00\
                            x00\xc0\x01\x00\x04\xc0\x02\x00\x04\xc0\x03\x00\
                            xff\x04\x00u#<\xc0;.\xe9\xdd"
    OK
    redis> TYPE mykey
    list
    redis> LRANGE mykey 0 -1
    1) "1"
    2) "2"
    3) "3"
