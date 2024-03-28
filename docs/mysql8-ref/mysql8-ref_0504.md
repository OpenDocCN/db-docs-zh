# 9.6 MyISAM 表维护和崩溃恢复

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-table-maintenance.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-table-maintenance.html)

9.6.1 使用 myisamchk 进行崩溃恢复

9.6.2 如何检查 MyISAM 表中的错误

9.6.3 如何修复 MyISAM 表

9.6.4 MyISAM 表优化

9.6.5 设置 MyISAM 表维护计划

本节讨论如何使用**myisamchk**来检查或修复`MyISAM`表（用于存储数据和索引的`.MYD`和`.MYI`文件的表）。有关一般**myisamchk**背景，请参阅 Section 6.6.4, “myisamchk — MyISAM Table-Maintenance Utility”。其他表修复信息可在 Section 3.14, “Rebuilding or Repairing Tables or Indexes”找到。

您可以使用**myisamchk**来检查、修复或优化数据库表。以下各节描述了如何执行这些操作以及如何设置表维护计划。有关使用**myisamchk**获取有关您的表的信息，请参阅 Section 6.6.4.5, “Obtaining Table Information with myisamchk”。

即使使用**myisamchk**进行表修复是相当安全的，但在进行修复或任何可能对表进行大量更改的维护操作之前，始终最好先备份*数据*。

**myisamchk**影响索引的操作可能导致`MyISAM` `FULLTEXT`索引使用与 MySQL 服务器使用的值不兼容的全文参数进行重建。为避免此问题，请遵循 Section 6.6.4.1, “myisamchk General Options”中的指南。

`MyISAM`表维护也可以使用执行类似于**myisamchk**的操作的 SQL 语句来完成：

+   要检查`MyISAM`表，请使用`CHECK TABLE`。

+   要修复`MyISAM`表，请使用`REPAIR TABLE`。

+   要优化`MyISAM`表，请使用`OPTIMIZE TABLE`。

+   要分析`MyISAM`表，请使用`ANALYZE TABLE`。

有关这些语句的更多信息，请参阅 Section 15.7.3, “Table Maintenance Statements”。

这些语句可以直接使用，也可以通过**mysqlcheck**客户端程序使用。这些语句相对于**myisamchk**的一个优点是服务器完成所有工作。使用**myisamchk**时，您必须确保服务器不同时使用表，以避免**myisamchk**和服务器之间发生不必要的交互。
