`BRPOP`是阻止列表弹出基元。
它是阻塞版本的`RPOP`因为它在存在时会阻止连接
没有要从任何给定列表中弹出的元素。
从第一个非空列表的尾部弹出一个元素，其中
给定的密钥按照给定的顺序进行检查。

查看[BLPOP 文档][cb]对于确切的语义，因为`BRPOP`是
与`BLPOP`唯一的区别是它弹出的元素来自
列表的尾部，而不是从头部弹出。

[cb]: /commands/blpop

@return

@array回复：具体而言：

*   一个`nil`当无法弹出任何元素并且超时过期时，多批量。
*   双元素多主体，第一个元素是键的名称
    其中，一个元素被弹出，第二个元素是
    弹出的元素。

@examples

    redis> DEL list1 list2
    (integer) 0
    redis> RPUSH list1 a b c
    (integer) 3
    redis> BRPOP list1 list2 0
    1) "list1"
    2) "c"
