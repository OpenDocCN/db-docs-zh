> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-files.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-files.html)

#### 15.1.20.1 由 CREATE TABLE 创建的文件

对于在文件表空间或通用表空间中创建的`InnoDB`表，表数据和相关索引存储在数据库目录中的一个.ibd 文件中。当在系统表空间中创建`InnoDB`表时，表数据和索引存储在代表系统表空间的 ibdata*文件中。`innodb_file_per_table`选项控制表是在文件表空间还是系统表空间中创建，默认情况下。`TABLESPACE`选项可用于将表放置在文件表空间、通用表空间或系统表空间中，而不受`innodb_file_per_table`设置的影响。

对于`MyISAM`表，存储引擎会创建数据和索引文件。因此，对于每个`MyISAM`表*`tbl_name`*，都会有两个磁盘文件。

| 文件 | 目的 |
| --- | --- |
| `*`tbl_name`*.MYD` | 数据文件 |
| `*`tbl_name`*.MYI` | 索引文件 |

第十八章，*替代存储引擎*，描述了每个存储引擎创建哪些文件来表示表。如果表名包含特殊字符，则表文件的名称将包含这些字符的编码版本，如第 11.2.4 节，“标识符映射到文件名”中所述。
