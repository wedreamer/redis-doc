从提供的键名称列表中的第一个非空列表键中弹出一个或多个元素。

`LMPOP`和`BLMPOP`类似于以下更受限制的命令：

*   `LPOP`或`RPOP`它只需要一个键，并且可以返回多个元素。
*   `BLPOP`或`BRPOP`它接受多个键，但只从一个键返回一个元素。

看`BLMPOP`用于此命令的阻塞变体。

根据传递的参数，从第一个非空列表的左侧或右侧弹出元素。
返回的元素数限制为非空列表的长度与 count 参数（默认为 1）之间的较低值。

@return

@array回复：具体而言：

*   一个`nil`当没有元素可以弹出时。
*   一个双元素数组，其中第一个元素是从中弹出元素的键的名称，第二个元素是元素数组。

@examples

```cli
LMPOP 2 non1 non2 LEFT COUNT 10
LPUSH mylist "one" "two" "three" "four" "five"
LMPOP 1 mylist LEFT
LRANGE mylist 0 -1
LMPOP 1 mylist RIGHT COUNT 10
LPUSH mylist "one" "two" "three" "four" "five"
LPUSH mylist2 "a" "b" "c" "d" "e"
LMPOP 2 mylist mylist2 right count 3
LRANGE mylist 0 -1
LMPOP 2 mylist mylist2 right count 5
LMPOP 2 mylist mylist2 right count 10
EXISTS mylist mylist2
```
