> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-file-io-type.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-user-summary-by-file-io-type.html)

#### 30.4.3.43 用户按文件 I/O 类型和 x$user_summary_by_file_io_type 视图

这些视图按用户和事件类型对文件 I/O 进行汇总。默认情况下，按用户和降序总延迟排序行。

`user_summary_by_file_io_type`和`x$user_summary_by_file_io_type`视图具有以下列：

+   `用户`

    客户端用户名。基础性能模式表中`USER`列为`NULL`的行被假定为后台线程，并以`background`主机名报告。

+   `事件名称`

    文件 I/O 事件名称。

+   `总`

    用户文件 I/O 事件的总发生次数。

+   `延迟`

    用户文件 I/O 事件定时发生的总等待时间。

+   `最大延迟`

    用户文件 I/O 事件定时发生的最大单个等待时间。
