返回或存储 中包含的元素[列表][tdtl],[设置][tdts]或
[排序集][tdtss]在`key`.

还有`SORT_RO`此命令的只读变体。

默认情况下, 排序为数字, 元素按其值进行比较
解释为双精度浮点数。
这是`SORT`最简单的形式：

[tdtl]: /topics/data-types#lists

[tdts]: /topics/data-types#set

[tdtss]: /topics/data-types#sorted-sets

    SORT mylist

若`mylist`是一个数字列表, 此命令将返回相同的列表
元素按从小到大的顺序排序。
要对数字从大到小进行排序, 请使用`!DESC`修饰语：

    SORT mylist DESC

什么时候`mylist`包含字符串值, 并且您希望对它们进行排序
在词典编纂上, 使用`!ALPHA`修饰语：

    SORT mylist ALPHA

Redis 可识别 UTF-8, 前提是您正确设置了`!LC_COLLATE`环境
变量。

可以使用`!LIMIT`修饰语。
此修饰符采用`offset`参数, 指定元素数
跳过和`count`参数, 指定要从中返回的元素数
起价`offset`.
下面的示例将返回排序版本的 10 个元素`mylist`,
从元素 0  (`offset`从零开始) ：

    SORT mylist LIMIT 0 10

几乎所有修饰符都可以一起使用。
下面的示例将返回前 5 个元素, 按字典顺序排序
按降序排列：

    SORT mylist LIMIT 0 5 ALPHA DESC

## 按外部键排序

有时您希望使用外部键作为权重对元素进行排序以进行比较
而不是比较列表中的实际元素, 设置或排序集。
假设列表`mylist`包含元素`1`,`2`和`3`代表
存储在 中对象的唯一 ID`object_1`,`object_2`和`object_3`.
当这些对象具有存储在 中的关联权重时`weight_1`,`weight_2`和
`weight_3`,`SORT`可以指示使用这些权重进行排序`mylist`跟
以下语句：

    SORT mylist BY weight_*

这`BY`选项采用模式 (等于`weight_*`在这个例子), ) , 那就是
用于生成用于排序的键。
这些键名是用第一次出现的`*`与
列表中元素的实际值  (`1`,`2`和`3`在此示例) ) 。

## 跳过对元素进行排序

这`!BY`选项还可以采用不存在的键, 从而导致`SORT`以跳过
排序操作。
如果要检索外部密钥, 这将非常有用 (请参阅`!GET`选择
下面) 没有排序的开销。

    SORT mylist BY nosort

## 检索外部键

前面的示例仅返回排序的 ID。
在某些情况下, 获取实际对象而不是其 ID 更有用
(`object_1`,`object_2`和`object_3`).
根据列表、集或排序集中的元素检索外部键可以
使用以下命令完成：

    SORT mylist BY weight_* GET object_*

这`!GET`选项可以多次使用, 以便为每个选项获取更多密钥
元素, 设置或排序集。

也可以`!GET`使用特殊模式的元素本身`#`:

    SORT mylist BY weight_* GET object_* GET #

## 使用外部密钥的限制

启用时`Redis cluster-mode`无法保证处理命令的节点上存在外部密钥。
在这种情况下, 任何使用`GET`或`BY`哪个引用外部键模式将导致命令失败并显示错误。

从 Redis 7.0 开始, 任何使用`GET`或`BY`仅当运行命令的当前用户具有完全密钥读取权限时, 才会允许引用外部密钥模式。
可以通过以下方式为用户设置全键读取权限, 例如, 指定`'%R~*'`或`'~*`具有相关的命令访问规则。
您可以查看`ACL SETUSER`命令手册, 了解有关设置 ACL 访问规则的更多信息。
如果未设置完全密钥读取权限, 则该命令将失败并显示错误。

## 存储 SORT 操作的结果

默认情况下, `SORT`将排序后的元素返回到客户端。
随着`!STORE`选项, 结果将存储在指定的列表中
键, 而不是返回到客户端。

    SORT mylist BY weight_* STORE resultkey

一个有趣的模式使用`SORT ... STORE`包括关联
`EXPIRE`超时到生成的键, 以便在应用程序中结果
的`SORT`操作可以缓存一段时间。
其他客户端将使用缓存的列表而不是调用`SORT`对于每个
请求。
当密钥超时时, 可以通过以下方式创建缓存的更新版本：
叫`SORT ... STORE`再。

请注意, 为了正确实现此模式, 请务必避免
多个客户端同时重建缓存。
这里需要某种锁定 (例如使用`SETNX`).

## 在 中使用哈希`!BY`和`!GET`

可以使用`!BY`和`!GET`针对哈希字段的选项, 其中
以下语法：

    SORT mylist BY weight_*->fieldname GET object_*->fieldname

字符串`->`用于将键名与哈希字段名分开。
如上文所述, 密钥将被替换, 哈希值存储在结果中
访问键以检索指定的哈希字段。

@return

@array回复：不通过`store`选项, 该命令返回已排序元素的列表。
@integer回复：当`store`选项, 该命令返回目标列表中已排序元素的数量。
