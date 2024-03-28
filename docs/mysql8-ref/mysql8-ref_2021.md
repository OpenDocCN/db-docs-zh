> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-file-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-file-instances-table.html)

#### 29.12.3.2 文件实例表

当执行文件 I/O 仪表化时，`file_instances` 表列出了性能模式看到的所有文件。如果磁盘上的文件从未被打开过，则不会显示在 `file_instances` 表中。当从磁盘中删除文件时，它也会从 `file_instances` 表中删除。

`file_instances` 表包含以下列：

+   `FILE_NAME`

    文件名。

+   `EVENT_NAME`

    与文件关联的仪器名称。

+   `OPEN_COUNT`

    文件上的打开句柄计数。如果文件被打开然后关闭，它被打开了 1 次，但 `OPEN_COUNT` 是 0。要列出服务器当前打开的所有文件，请使用 `WHERE OPEN_COUNT > 0`。

`file_instances` 表包含以下索引：

+   主键 (`FILE_NAME`)

+   (`EVENT_NAME`) 上的索引

`TRUNCATE TABLE` 不允许用于 `file_instances` 表。
