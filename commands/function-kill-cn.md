杀死当前正在执行的函数。

这`FUNCTION KILL`命令只能用于在执行期间未修改数据集的函数（因为停止只读函数不会违反脚本引擎保证的原子性）。

欲了解更多信息，请参阅[Redis 函数简介](/topics/functions-intro).

@return

@simple字符串回复
