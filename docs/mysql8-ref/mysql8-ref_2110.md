> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-innodb-redo-log-files-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-innodb-redo-log-files-table.html)

#### 29.12.21.4 `innodb_redo_log_files`表

`innodb_redo_log_files`表包含每个活动的`InnoDB`重做日志文件的一行。此表在 MySQL 8.0.30 中引入。

`innodb_redo_log_files`表具有以下列：

+   `FILE_ID`

    重做日志文件的 ID。该值对应于重做日志文件编号。

+   `FILE_NAME`

    重做日志文件的路径和文件名。

+   `START_LSN`

    重做日志文件中第一个块的日志序列号。

+   `END_LSN`

    最后一个块后的日志序列号在重做日志文件中。

+   `SIZE_IN_BYTES`

    文件中重做日志数据的大小，以字节为单位。数据大小从`END_LSN`到开始`>START_LSN`进行测量。由于文件头（2048 字节）不包括在此列报告的值中，因此磁盘上的重做日志文件大小略大。

+   `IS_FULL`

    重做日志文件是否已满。值为 0 表示文件中有空闲空间。值为 1 表示文件已满。

+   `CONSUMER_LEVEL`

    保留以供将来使用。
