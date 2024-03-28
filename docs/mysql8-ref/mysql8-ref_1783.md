> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskstats-1sec.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskstats-1sec.html)

#### 25.6.16.32 ndbinfo diskstats_1sec 表

`diskstats_1sec`表提供了过去 20 秒内对磁盘数据表空间的写入信息。

`diskstat`表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `block_instance`

    `PGMAN`的报告实例 ID

+   `pages_made_dirty`

    指定的 1 秒间隔内使页面变脏的页数

+   `reads_issued`

    指定的 1 秒间隔内发出的读取

+   `reads_completed`

    指定的 1 秒间隔内完成的读取

+   `writes_issued`

    指定的 1 秒间隔内发出的写入

+   `writes_completed`

    指定的 1 秒间隔内完成的写入

+   `log_writes_issued`

    指定的 1 秒间隔内，页面写入需要日志写入的次数

+   `log_writes_completed`

    指定的 1 秒间隔内完成的日志写入次数

+   `get_page_calls_issued`

    指定的 1 秒间隔内发出的`get_page()`调用次数

+   `get_page_reqs_issued`

    `get_page()`调用导致等待 I/O 或 I/O 已开始完成的次数，在指定的 1 秒间隔内

+   `get_page_reqs_completed`

    等待 I/O 或 I/O 完成的`get_page()`调用次数，在指定的 1 秒间隔内已完成

+   `seconds_ago`

    过去的时间间隔内的 1 秒间隔数，适用于此行

##### 备注

此表中的每一行对应于`PGMAN`的一个实例，在从 0 到 19 秒前的 1 秒间隔内发生；每个 LDM 线程有一个这样的实例，每个数据节点还有一个额外的实例。
