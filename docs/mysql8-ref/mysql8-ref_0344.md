> 原文：[`dev.mysql.com/doc/refman/8.0/en/reproducible-test-case.html`](https://dev.mysql.com/doc/refman/8.0/en/reproducible-test-case.html)

#### 7.9.1.7 创建测试用例，如果您遇到表格损坏

以下过程适用于`MyISAM`表格。有关遇到`InnoDB`表格损坏时应采取的步骤的信息，请参阅第 1.5 节“如何报告错误或问题”。

如果遇到损坏的`MyISAM`表格，或者**mysqld**在一些更新语句后总是失败，您可以通过以下方式测试问题是否可重现：

1.  使用**mysqladmin shutdown**停止 MySQL 守护程序。

1.  备份表格以防修复出现问题的极小可能性。

1.  使用**myisamchk -s database/*.MYI**检查所有表格。使用**myisamchk -r database/*`table`*.MYI**修复任何损坏的表格。

1.  备份表格的第二份备份。

1.  如果需要更多空间，请删除（或移动）MySQL 数据目录中的任何旧日志文件。

1.  启动**mysqld**并启用二进制日志。如果要找到导致**mysqld**崩溃的语句，还应该启用常规查询日志。请参阅第 7.4.3 节“常规查询日志”和第 7.4.4 节“二进制日志”。

1.  当您遇到崩溃的表格时，停止**mysqld**服务器。

1.  恢复备份。

1.  重新启动**mysqld**服务器，*不*启用二进制日志。

1.  使用**mysqlbinlog binary-log-file | mysql**重新执行语句。二进制日志保存在 MySQL 数据库目录中，名称为`hostname-bin.*`NNNNNN`*`。

1.  如果表格再次损坏或者您可以通过上述命令使**mysqld**停止运行，那么您已经找到了一个可重现的错误。按照第 1.5 节“如何报告错误或问题”中给出的说明，将表格和二进制日志通过 FTP 上传到我们的错误数据库。如果您是支持客户，您可以使用 MySQL 客户支持中心（[`www.mysql.com/support/`](https://www.mysql.com/support/)）通知 MySQL 团队有关问题，并尽快修复。
