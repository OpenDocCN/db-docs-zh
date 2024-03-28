> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskstat.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-diskstat.html)

#### 25.6.16.31 ndbinfo diskstat 表

`diskstat`表提供了关于过去 1 秒内对磁盘数据表空间的写入的信息。

`diskstat`表包含以下列：

+   `node_id`

    此节点的节点 ID

+   `block_instance`

    报告`PGMAN`实例的 ID

+   `pages_made_dirty`

    过去一秒内使页面变脏的次数

+   `reads_issued`

    过去一秒内发出的读取次数

+   `reads_completed`

    过去一秒内完成的读取次数

+   `writes_issued`

    过去一秒内发出的写入次数

+   `writes_completed`

    过去一秒内完成的写入次数

+   `log_writes_issued`

    过去一秒内页面写入需要进行日志写入的次数

+   `log_writes_completed`

    过去一秒内完成的日志写入次数

+   `get_page_calls_issued`

    过去一秒内发出的`get_page()`调用次数

+   `get_page_reqs_issued`

    过去一秒内`get_page()`调用导致等待 I/O 或已经开始的 I/O 完成的次数

+   `get_page_reqs_completed`

    过去一秒内等待 I/O 或 I/O 完成的`get_page()`调用次数

##### 注意

此表中的每一行对应一个`PGMAN`实例；每个 LDM 线程对应一个这样的实例，每个数据节点还有一个额外的实例。
