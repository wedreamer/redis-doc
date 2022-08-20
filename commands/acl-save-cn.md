当 Redis 配置为使用 ACL 文件时（使用`aclfile`配置
选项），此命令会将当前定义的 ACL 从服务器内存保存到 ACL 文件。

@return

@simple字符串回复：`OK`关于成功。

由于多种原因，该命令可能会失败并显示错误：如果无法写入文件或服务器未配置为使用外部 ACL 文件。

@examples

    > ACL SAVE
    +OK

    > ACL SAVE
    -ERR There was an error trying to save the ACLs. Please check the server logs for more information
