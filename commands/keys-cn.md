返回匹配的所有键`pattern`.

虽然此操作的时间复杂度为 O（N），但常数时间为
相当低。
例如，在入门级笔记本电脑上运行的Redis可以扫描100万个密钥。
数据库在 40 毫秒内。

**警告**：考虑`KEYS`作为仅在生产中使用的命令
环境极其小心。
当针对大型数据库执行时，它可能会破坏性能。
此命令用于调试和特殊操作，如更改
您的键空间布局。
不要使用`KEYS`在常规应用程序代码中。
如果您正在寻找一种在密钥空间的子集中查找密钥的方法，请考虑
用`SCAN`或[集][tdts].

[tdts]: /topics/data-types#sets

支持的球形样式模式：

*   `h?llo`比赛`hello`,`hallo`和`hxllo`
*   `h*llo`比赛`hllo`和`heeeello`
*   `h[ae]llo`比赛`hello`和`hallo,`但不是`hillo`
*   `h[^e]llo`比赛`hallo`,`hbllo`, ...但不是`hello`
*   `h[a-b]llo`比赛`hallo`和`hbllo`

用`\`以转义特殊字符，如果要逐字匹配它们。

@return

@array回复：匹配键列表`pattern`.

@examples

```cli
MSET firstname Jack lastname Stuntman age 35
KEYS *name*
KEYS a??
KEYS *
```
