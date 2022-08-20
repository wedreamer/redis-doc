该命令显示最近的 ACL 安全事件的列表：

1.  无法对其连接进行身份验证`AUTH`或`HELLO`.
2.  命令被拒绝，因为违反了当前的 ACL 规则。
3.  命令被拒绝，因为当前 ACL 规则中不允许访问密钥。

可选参数指定要显示的条目数。默认情况下
最多返回十个故障。特别`RESET`参数清除日志。
从最近的条目开始显示。

@return

调用以显示安全事件时：

@array回复：ACL 安全事件的列表。

当调用`RESET`:

@simple字符串回复：`OK`如果安全日志已清除。

@examples

    > AUTH someuser wrongpassword
    (error) WRONGPASS invalid username-password pair
    > ACL LOG 1
    1)  1) "count"
        2) (integer) 1
        3) "reason"
        4) "auth"
        5) "context"
        6) "toplevel"
        7) "object"
        8) "AUTH"
        9) "username"
       10) "someuser"
       11) "age-seconds"
       12) "4.0960000000000001"
       13) "client-info"
       14) "id=6 addr=127.0.0.1:63026 fd=8 name= age=9 idle=0 flags=N db=0 sub=0 psub=0 multi=-1 qbuf=48 qbuf-free=32720 obl=0 oll=0 omem=0 events=r cmd=auth user=default"
