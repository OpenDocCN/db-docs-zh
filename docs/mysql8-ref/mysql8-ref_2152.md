> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-latest-file-io.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-latest-file-io.html)

#### 30.4.3.15 最新的文件 I/O 和 x$latest_file_io 视图

这些视图按文件和线程分组总结文件 I/O 活动。默认情况下，按最近的 I/O 排序行。

`latest_file_io` 和 `x$latest_file_io` 视图具有以下列：

+   `thread`

    对于前台线程，与线程相关联的帐户（以及 TCP/IP 连接的端口号）。对于后台线程，线程名称和线程 ID。

+   `file`

    文件路径名。

+   `latency`

    文件 I/O 事件的等待时间。

+   `operation`

    操作类型。

+   `requested`

    请求的数据字节数。
