此命令控制在执行的下一个命令中对密钥的跟踪
通过连接，当在 中启用跟踪时`OPTIN`或`OPTOUT`模式。
请检查
[客户端缓存文档](/topics/client-side-caching)为
背景信息。

启用跟踪 Redis 后，使用`CLIENT TRACKING`命令，它是
可以指定`OPTIN`或`OPTOUT`选项，以便按键
在只读命令不会被服务器自动记住
稍后将失效。当我们在`OPTIN`模式，我们可以启用
通过调用来跟踪下一个命令中的密钥`CLIENT CACHING yes`
紧挨着它。同样，当我们在`OPTOUT`模式和键
通常都是跟踪的，我们可以避免在下一个命令中的键被
跟踪使用`CLIENT CACHING no`.

基本上，该命令在连接中设置一个状态，该状态仅有效
对于下一个命令执行，这将修改客户端的行为
跟踪。

@return

@simple字符串回复：`OK`或错误（如果参数不是 yes 或 no）。
