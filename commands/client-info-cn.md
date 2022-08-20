该命令以大部分人类可读的格式返回有关当前客户端连接的信息和统计信息。

回复格式与`CLIENT LIST`，并且内容仅包含有关当前客户端的信息。

@examples

```cli
CLIENT INFO
```

@return

@bulk字符串回复：一个唯一的字符串，如`CLIENT LIST`页，用于当前客户端。
