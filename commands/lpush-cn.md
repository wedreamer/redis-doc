在存储在`key`.
如果`key`不存在, 在执行推送之前将其创建为空列表
操作。
什么时候`key`保存不是列表的值, 则返回错误。

只需一次命令调用即可推送多个元素
在命令末尾指定多个参数。
元素一个接一个地插入到列表的头部, 从
最左边的元素到最右边的元素。
例如, 命令`LPUSH mylist a b c`将生成一个列表
含`c`作为第一个元素, `b`作为第二个元素和`a`作为第三个元素。

@return

@integer回复：推送操作后列表的长度。

@examples

```cli
LPUSH mylist "world"
LPUSH mylist "hello"
LRANGE mylist 0 -1
```
