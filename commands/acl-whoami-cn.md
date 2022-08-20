返回用于对当前连接进行身份验证的用户名。
新连接使用“默认”用户进行身份验证。他们
可以更改用户使用`AUTH`.

@return

@bulk字符串回复：当前连接的用户名。

@examples

    > ACL WHOAMI
    "default"
