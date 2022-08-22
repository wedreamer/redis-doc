在以下情况下返回`member`是存储在`key`.

@return

@integer回复, 具体而言：

*   `1`如果元素是集合的成员。
*   `0`如果元素不是集合的成员, 或者如果`key`不存在。

@examples

```cli
SADD myset "one"
SISMEMBER myset "one"
SISMEMBER myset "two"
```
