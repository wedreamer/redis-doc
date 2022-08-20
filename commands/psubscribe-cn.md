为客户端订阅给定的模式。

支持的球形样式模式：

*   `h?llo`订阅`hello`,`hallo`和`hxllo`
*   `h*llo`订阅`hllo`和`heeeello`
*   `h[ae]llo`订阅`hello`和`hallo,`但不是`hillo`

用`\`以转义特殊字符，如果要逐字匹配它们。
