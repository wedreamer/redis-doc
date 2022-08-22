该命令显示 Redis 服务器中当前处于活动状态的 ACL 规则。每
返回数组中的行定义了不同的用户, 格式为
与 redis.conf 文件或外部 ACL 文件中的用法相同, 因此您可以
将 ACL LIST 命令返回的内容直接剪切并粘贴到
如果您愿意, 可以配置文件 (但请务必检查`ACL SAVE`).

@return

字符串数组。

@examples

    > ACL LIST
    1) "user antirez on #9f86d081884c7d659a2feaa0c55ad015a3bf4f1b2b0b822cd15d6c15b0f00a08 ~objects:* &* +@all -@admin -@dangerous"
    2) "user default on nopass ~* &* +@all"
