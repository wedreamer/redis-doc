返回上次成功执行数据库保存的 UNIX 时间。
客户端可以检查`BGSAVE`命令已成功读取`LASTSAVE`价值
然后发出`BGSAVE`命令并每隔 N 定期检查一次
秒，如果`LASTSAVE`改变。

@return

@integer回复：UNIX 时间戳。
