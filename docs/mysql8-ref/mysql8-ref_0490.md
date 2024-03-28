# 9.4 使用 mysqldump 进行备份

> [`dev.mysql.com/doc/refman/8.0/en/using-mysqldump.html`](https://dev.mysql.com/doc/refman/8.0/en/using-mysqldump.html)

9.4.1 使用 mysqldump 以 SQL 格式导出数据

9.4.2 重新加载 SQL 格式备份

9.4.3 使用 mysqldump 以分隔文本格式导出数据

9.4.4 重新加载分隔文本格式备份

9.4.5 mysqldump 技巧

提示

考虑使用 MySQL Shell dump 工具，提供多线程并行导出、文件压缩、进度信息显示，以及云功能，如 Oracle Cloud Infrastructure Object Storage 流式传输，以及 MySQL HeatWave Service 兼容性检查和修改。导出的数据可以轻松导入到 MySQL Server 实例或 MySQL HeatWave Service DB System 中，使用 MySQL Shell load dump 工具。MySQL Shell 的安装说明可在这里找到。

本节描述了如何使用**mysqldump**生成导出文件，以及如何重新加载导出文件。导出文件可以以多种方式使用：

+   作为备份，以便在数据丢失时进行数据恢复。

+   作为设置副本的数据源。

+   作为实验的数据源：

    +   复制数据库的副本，可以在不更改原始数据的情况下使用。

    +   用于测试潜在的升级不兼容性。

**mysqldump**生成两种类型的输出，取决于是否给出`--tab`选项：

+   没有`--tab`选项时，**mysqldump**将 SQL 语句写入标准输出。这些输出包括用于创建导出对象（数据库、表、存储过程等）的`CREATE`语句，以及用于将数据加载到表中的`INSERT`语句。输出可以保存在文件中，并使用**mysql**稍后重新加载，以重新创建导出的对象。有选项可修改 SQL 语句的格式，并控制导出哪些对象。

+   使用`--tab`，**mysqldump**为每个转储的表生成两个输出文件。服务器将一个文件写为制表符分隔的文本，每行一个表行。该文件在输出目录中命名为`*`tbl_name`*.txt`。服务器还向**mysqldump**发送一个`CREATE TABLE`语句，该语句被写入一个名为`*`tbl_name`*.sql`的文件中。
