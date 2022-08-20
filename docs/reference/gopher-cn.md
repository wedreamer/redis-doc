***

标题：“Redis和Gopher协议”
链接标题： “Gopher protocol”
体重： 1
描述： Redis Gopher 协议实现
别名：

*   /topics/gopher

***

\*\* 注意：删除了对 Gopher 的支持是 Redis 7.0 \*\*

Redis 包含 Gopher 协议的实现，如
这[RFC 1436](https://www.ietf.org/rfc/rfc1436.txt).

Gopher协议在90年代后期非常流行。这是一种替代方案
到Web，服务器端和客户端的实现都非常简单
Redis服务器只有100行代码来实现这一点
支持。

你现在用Gopher做什么？好吧，地鼠从不*真*死了，和
最近有一个运动，以便Gopher更具层次性的内容
仅由要复活的纯文本文档组成。有些人想要更简单
互联网，别人认为主流互联网变得太多了
受控，为人们创造一个替代空间很酷
想要一点新鲜空气。

无论如何，在Redis的10岁生日，我们给了它Gopher协议。
作为礼物。

## 它是如何运作的

Redis Gopher 支持使用 Redis 的内联协议，特别是
两种无论如何都是非法的内联请求：空请求
或任何以“/”开头的请求（没有Redis命令启动）
用这样的斜杠）。正常的 RESP2/RESP3 请求完全不在
Gopher协议实现的路径，并且通常也是如此。

如果您在启用 Gopher 时打开与 Redis 的连接并将其发送
像“/foo”这样的字符串，如果有一个名为“/foo”的键，它通过
地鼠协议。

为了创建一个真正的Gopher“洞”（Gopher中Gopher站点的名称）
说话），你可能需要一个脚本，比如<https://github.com/antirez/gopher2redis>.

## 安全警告

如果您计划将 Redis 放在互联网上的可公开访问的地址中
到服务器地鼠页面**确保设置密码**添加到实例。
设置密码后：

1.  Gopher服务器（启用时，不是默认的）将通过Gopher杀死服务内容。
2.  但是，在客户端进行身份验证之前，无法调用其他命令。

因此，请使用`requirepass`选项来保护您的实例。

要启用 Gopher 支持，请使用以下配置行。

    gopher-enabled yes

访问不是字符串或不退出的密钥将生成
Gopher 协议格式中的错误。
