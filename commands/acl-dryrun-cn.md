模拟给定用户对给定命令的执行。
此命令可用于测试给定用户的权限，而无需启用该用户或导致运行该命令的副作用。

@return

@simple字符串回复：`OK`关于成功。
@bulk字符串回复：描述用户无法执行命令的原因的错误。

@examples

    > ACL SETUSER VIRGINIA +SET ~*
    "OK"
    > ACL DRYRUN VIRGINIA SET foo bar
    "OK"
    > ACL DRYRUN VIRGINIA GET foo bar
    "This user has no permissions to run the 'GET' command"
