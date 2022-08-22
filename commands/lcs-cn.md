LCS 命令实现最长的通用子序列算法。请注意, 这与最长的常用字符串算法不同, 因为字符串中的匹配字符不需要连续。

例如, “foo”和“fao”之间的LCS是“fo”, 因为从左到右扫描两个字符串, 最长的通用字符集由第一个“f”组成, 然后是“o”。

LCS对于评估两个字符串的相似程度非常有用。字符串可以表示许多内容。例如, 如果两个字符串是DNA序列, LCS将提供两个DNA序列之间相似性的度量。如果字符串表示由某个用户编辑的某些文本, 则 LCS 可以表示新文本与旧文本的不同程度, 依此类推。

请注意, 此算法运行在`O(N*M)`time, 其中 N 是第一个字符串的长度, M 是第二个字符串的长度。因此, 要么旋转不同的Redis实例以运行此算法, 要么确保针对非常小的字符串运行它。

    > MSET key1 ohmytext key2 mynewtext
    OK
    > LCS key1 key2
    "mytext"

有时我们只需要比赛的长度：

    > LCS key1 key2 LEN
    (integer) 6

然而, 通常非常有用的是知道每个字符串中的匹配位置：

    > LCS key1 key2 IDX
    1) "matches"
    2) 1) 1) 1) (integer) 4
             2) (integer) 7
          2) 1) (integer) 5
             2) (integer) 8
       2) 1) 1) (integer) 2
             2) (integer) 3
          2) 1) (integer) 0
             2) (integer) 1
    3) "len"
    4) (integer) 6

匹配是从最后一个到第一个生成, 因为这是如何
该算法有效, 并且以相同的顺序发出东西更有效。
上面的数组表示第一个匹配项 (数组的第二个元素) 
位于第一个字符串的位置 2-3 和第二个字符串的位置 0-1 之间。
然后4-7和5-8之间还有另一场比赛。

要将匹配项列表限制为给定最小长度的匹配项, 请执行以下操作：

    > LCS key1 key2 IDX MINMATCHLEN 4
    1) "matches"
    2) 1) 1) 1) (integer) 4
             2) (integer) 7
          2) 1) (integer) 5
             2) (integer) 8
    3) "len"
    4) (integer) 6

最后还要有匹配len：

    > LCS key1 key2 IDX MINMATCHLEN 4 WITHMATCHLEN
    1) "matches"
    2) 1) 1) 1) (integer) 4
             2) (integer) 7
          2) 1) (integer) 5
             2) (integer) 8
          3) (integer) 4
    3) "len"
    4) (integer) 6

@return

*   如果不使用修饰符, 则返回表示最长公共子字符串的字符串。
*   什么时候`LEN`给定命令返回最长公共子字符串的长度。
*   什么时候`IDX`给定命令返回一个数组, 其中包含LCS长度和两个字符串中的所有范围, 每个字符串的开始和结束偏移量, 其中存在匹配项。什么时候`WITHMATCHLEN`给定每个表示匹配项的数组也将具有匹配项的长度 (请参阅示) ) 。
