在[交易][tt]并恢复
连接状态为正常。

[tt]: /topics/transactions

使用时`WATCH`,`EXEC`仅当监视的键是
未修改，允许[检查和设置机制][ttc].

[ttc]: /topics/transactions#cas

@return

@array-reply：每个元素都是对
原子事务。

使用时`WATCH`,`EXEC`如果执行已中止，可以返回@nil回复。
