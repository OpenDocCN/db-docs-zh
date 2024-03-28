> 原文：[`dev.mysql.com/doc/refman/8.0/en/corrupted-myisam-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/corrupted-myisam-tables.html)

#### 18.2.4.1 损坏的 MyISAM 表

即使`MyISAM`表格式非常可靠（由 SQL 语句对表所做的所有更改都在语句返回之前写入），如果发生以下任何事件，仍然可能出现损坏的表：

+   **mysqld**进程在写入过程中被终止。

+   发生意外的计算机关闭（例如，计算机被关闭）。

+   硬件故障。

+   您正在使用外部程序（例如**myisamchk**）同时修改服务器正在修改的表。

+   MySQL 或`MyISAM`代码中的软件错误。

损坏表的典型症状包括：

+   从表中选择数据时出现以下错误：

    ```sql
    Incorrect key file for table: '...'. Try to repair it
    ```

+   查询在表中找不到行或返回不完整的结果。

您可以使用`CHECK TABLE`语句检查`MyISAM`表的健康状况，并使用`REPAIR TABLE`修复损坏的`MyISAM`表。当**mysqld**未运行时，您还可以使用**myisamchk**命令检查或修复表。请参阅第 15.7.3.2 节，“CHECK TABLE Statement”，第 15.7.3.5 节，“REPAIR TABLE Statement”，以及第 6.6.4 节，“myisamchk — MyISAM 表维护实用程序”。

如果您的表经常损坏，您应该尝试确定为什么会发生这种情况。最重要的是要知道表是否因意外服务器退出而损坏。您可以通过查看错误日志中最近的`重新启动的 mysqld`消息来轻松验证这一点。如果有这样的消息，那么表损坏很可能是服务器死机的结果。否则，损坏可能发生在正常操作期间。这是一个错误。您应该尝试创建一个可重现的测试用例来演示问题。请参阅第 B.3.3.3 节，“如果 MySQL 一直崩溃该怎么办”，以及第 7.9 节，“调试 MySQL”。
