该命令返回为现有 ACL 用户定义的所有规则。

具体来说，它列出了用户的 ACL 标志、密码哈希、命令、键模式、通道模式（在版本 6.2 中添加）和选择器（在版本 7.0 中添加）。
如果向用户添加更多元数据，将来可能会返回其他信息。

命令规则始终以与`ACL SETUSER`命令。
在 7.0 版之前，键和通道作为模式数组返回，但在 7.0 版之后，它们现在也以与`ACL SETUSER`命令。
注意：此命令规则描述反映了用户的有效权限，因此尽管它可能与用于配置用户的规则集不同，但在功能上仍然相同。

选择器按应用于用户的顺序列出，并包括有关命令、键模式和通道模式的信息。

@array回复：用户的 ACL 规则定义的列表。

如果`user`不存在返回@nil回复。

@examples

下面是用户配置的示例

    > ACL SETUSER sample on nopass +GET allkeys &* (+SET ~key2)
    "OK"
    > ACL GETUSER sample
    1) "flags"
    2) 1) "on"
       2) "allkeys"
       3) "nopass"
    3) "passwords"
    4) (empty array)
    5) "commands"
    6) "+@all"
    7) "keys"
    8) "~*"
    9) "channels"
    10) "&*"
    11) "selectors"
    12) 1) 1) "commands"
           6) "+SET"
           7) "keys"
           8) "~key2"
           9) "channels"
           10) "&*"
