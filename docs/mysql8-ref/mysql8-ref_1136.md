# 16.2 移除基于文件的元数据存储

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-file-removal.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-file-removal.html)

在先前的 MySQL 版本中，字典数据部分存储在元数据文件中。基于文件的元数据存储的问题包括昂贵的文件扫描、易受文件系统相关错误的影响、处理复制和崩溃恢复失败状态的复杂代码，以及缺乏可扩展性，使得难以为新功能和关系对象添加元数据。

下面列出的元数据文件已从 MySQL 中移除。除非另有说明，先前存储在元数据文件中的数据现在存储在数据字典表中。

+   `.frm`文件：表元数据文件。随着`.frm`文件的移除：

    +   由`.frm`文件结构施加的 64KB 表定义大小限制已被移除。

    +   信息模式`TABLES`表的`VERSION`列报告了一个硬编码值`10`，这是 MySQL 5.7 中使用的最后一个`.frm`文件版本。

+   `.par`文件：分区定义文件。`InnoDB`在 MySQL 5.7 中引入对`InnoDB`表的本机分区支持后停止使用分区定义文件。

+   `.TRN`文件：触发器命名空间文件。

+   `.TRG`文件：触发器参数文件。

+   `.isl`文件：`InnoDB`符号链接文件，包含在数据目录之外创建的 file-per-table 表空间文件的位置。

+   `db.opt`文件：数据库配置文件。这些文件，每个数据库目录一个，包含数据库默认字符集属性。

+   `ddl_log.log`文件：该文件包含由数据定义语句（如`DROP TABLE`和`ALTER TABLE`）生成的元数据操作记录。
