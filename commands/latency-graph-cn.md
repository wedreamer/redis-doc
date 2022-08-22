为指定的事件生成 ASCII 样式图形。

`LATENCY GRAPH`让您直观地了解`event`通过最先进的可视化。它可用于在诉诸于从中解析原始数据的手段之前快速掌握情况`LATENCY HISTORY`或外部工具。

的有效值`event`是：

*   `active-defrag-cycle`
*   `aof-fsync-always`
*   `aof-stat`
*   `aof-rewrite-diff-write`
*   `aof-rename`
*   `aof-write`
*   `aof-write-active-child`
*   `aof-write-alone`
*   `aof-write-pending-fsync`
*   `command`
*   `expire-cycle`
*   `eviction-cycle`
*   `eviction-del`
*   `fast-command`
*   `fork`
*   `rdb-unlink-temp-file`

@examples

    127.0.0.1:6379> latency reset command
    (integer) 0
    127.0.0.1:6379> debug sleep .1
    OK
    127.0.0.1:6379> debug sleep .2
    OK
    127.0.0.1:6379> debug sleep .3
    OK
    127.0.0.1:6379> debug sleep .5
    OK
    127.0.0.1:6379> debug sleep .4
    OK
    127.0.0.1:6379> latency graph command
    command - high 500 ms, low 101 ms (all time high 500 ms)
    --------------------------------------------------------------------------------
       #_
      _||
     _|||
    _||||

    11186
    542ss
    sss

每个图形列下的垂直标签表示秒数, 
事件发生的分钟、小时或天前。例如, “15s”表示
第一个图形事件发生在15秒前。

图形以最小-最大比例进行规范化, 以便零 (下划线
在下行中) 是最小值, 较高行中的 # 是最大值。

有关详细信息, 请参阅[延迟监控框架页面][lm].

[lm]: /topics/latency-monitor

@return

@bulk字符串回复
