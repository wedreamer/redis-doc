此命令与`DEL`：它删除指定的键。
就像`DEL`如果密钥不存在, 则忽略该密钥。但是命令
在不同的线程中执行实际的内存回收, 因此它不是
阻塞, 而`DEL`是。这是命令名称的来源：
命令只是**取消链接**来自键空间的键。实际删除
稍后将异步发生。

@return

@integer回复：取消链接的键数。

@examples

```cli
SET key1 "Hello"
SET key2 "World"
UNLINK key1 key2 key3
```
