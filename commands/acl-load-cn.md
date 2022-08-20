当 Redis 配置为使用 ACL 文件时（使用`aclfile`配置
选项），此命令将从文件中重新加载 ACL，并替换所有
当前 ACL 规则及其文件中定义的规则。该命令使
肯定有一个*全有或全无*行为，即：

*   如果文件中的每一行都有效，则加载所有 ACL。
*   如果文件中的一行或多行无效，则不会加载任何内容，并且将继续使用服务器内存中定义的旧 ACL 规则。

@return

@simple字符串回复：`OK`关于成功。

由于多种原因，该命令可能会失败并显示错误：如果文件不可读，如果文件内部存在错误，在这种情况下，错误将在错误中报告给用户。最后，如果服务器未配置为使用外部 ACL 文件，则该命令将失败。

@examples

    > ACL LOAD
    +OK

    > ACL LOAD
    -ERR /tmp/foo:1: Unknown command or category name in ACL...
