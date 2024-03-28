# 9.6.1 使用 myisamchk 进行崩溃恢复

> 原文：[`dev.mysql.com/doc/refman/8.0/en/myisam-crash-recovery.html`](https://dev.mysql.com/doc/refman/8.0/en/myisam-crash-recovery.html)

这一部分描述了如何检查和处理 MySQL 数据库中的数据损坏。如果您的表经常损坏，您应该尝试找出原因。请参阅第 B.3.3.3 节，“如果 MySQL 经常崩溃该怎么办”。

有关`MyISAM`表如何变得损坏的解释，请参阅第 18.2.4 节，“MyISAM 表问题”。

如果您使用外部锁定禁用（默认情况下）运行**mysqld**，您不能可靠地使用**myisamchk**来检查一个表，当**mysqld**正在使用相同的表时。如果您可以确定没有人可以在您运行**myisamchk**检查表时使用**mysqld**访问表，您只需在开始检查表之前执行**mysqladmin flush-tables**。如果您无法保证这一点，您必须在检查表时停止**mysqld**。如果您运行**myisamchk**来检查**mysqld**同时正在更新的表，即使表没有损坏，您可能会收到警告。

如果服务器启用了外部锁定，您可以随时使用**myisamchk**来检查表。在这种情况下，如果服务器尝试更新一个正在使用的表，服务器会等待**myisamchk**完成后才继续。

如果您使用**myisamchk**来修复或优化表，您*必须*始终确保**mysqld**服务器未使用该表（如果禁用外部锁定也适用）。如果不停止**mysqld**，您至少应在运行**myisamchk**之前执行**mysqladmin flush-tables**。如果服务器和**myisamchk**同时访问表，您的表*可能会损坏*。

在执行崩溃恢复时，重要的是要理解数据库中每个`MyISAM`表*`tbl_name`*对应于数据库目录中的三个文件，如下表所示。

| 文件 | 目的 |
| --- | --- |
| `*`tbl_name`*.MYD` | 数据文件 |
| `*`tbl_name`*.MYI` | 索引文件 |

这三种文件类型中的每一种都可能以各种方式损坏，但问题最常发生在数据文件和索引文件中。

**myisamchk** 通过逐行创建`.MYD`数据文件的副本来工作。它通过删除旧的`.MYD`文件并将新文件重命名为原始文件名来结束修复阶段。如果使用`--quick`，**myisamchk** 不会创建临时的`.MYD`文件，而是假设`.MYD`文件是正确的，并且仅生成一个新的索引文件而不触及`.MYD`文件。这是安全的，因为**myisamchk**会自动检测`.MYD`文件是否损坏，并在损坏时中止修复。您还可以两次指定`--quick`选项给**myisamchk**。在这种情况下，**myisamchk**不会在某些错误（如重复键错误）上中止，而是尝试通过修改`.MYD`文件来解决这些错误。通常只有在磁盘空间不足以执行正常修复时，才有必要使用两个`--quick`选项。在这种情况下，您至少应在运行**myisamchk**之前备份表。
