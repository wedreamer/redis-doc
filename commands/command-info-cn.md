返回有关多个 Redis 命令的详细信息的@array回复。

结果格式与`COMMAND`除了您可以指定哪些命令
返回。

如果您请求有关不存在的命令的详细信息，则返回这些命令
位置将为零。

@return

@array回复：命令详细信息的嵌套列表。

@examples

```cli
COMMAND INFO get set eval
COMMAND INFO foo evalsha config bar
```
