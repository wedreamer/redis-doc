这`TIME`命令以两个项目列表的形式返回当前服务器时间：Unix
时间戳和当前秒内已经过的微秒数。
基本上界面与其中一个非常相似`gettimeofday`系统
叫。

@return

@array回复, 具体而言：

包含两个元素的多批量回复：

*   unix 时间 (以秒为单位) 。
*   微秒。

@examples

```cli
TIME
TIME
```
