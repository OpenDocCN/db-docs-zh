> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-log-files.html`](https://dev.mysql.com/doc/refman/8.0/en/using-log-files.html)

#### 7.9.1.6 使用服务器日志查找**mysqld**错误原因

在启用了一般查询日志的情况下启动**mysqld**之前，你应该使用**myisamchk**检查所有表。参见第七章，*MySQL 服务器管理*。

如果**mysqld**死机或挂起，你应该启动启用了一般查询日志的**mysqld**。参见第 7.4.3 节，“一般查询日志”。当**mysqld**再次死机时，你可以检查日志文件末尾的查询，看看是哪个查询导致了**mysqld**死机。

如果使用默认的一般查询日志文件，日志将存储在数据库目录中，文件名为`*`host_name`*.log`。在大多数情况下，导致**mysqld**死机的是日志文件中的最后一个查询，但如果可能的话，你应该通过重新启动**mysqld**并从**mysql**命令行工具执行找到的查询来验证这一点。如果这样可以，你还应该测试所有未完成的复杂查询。

你也可以尝试对所有执行时间较长的`SELECT`语句使用`EXPLAIN`命令，以确保**mysqld**正确使用索引。参见第 15.8.2 节，“EXPLAIN 语句”。

通过启用慢查询日志，你可以找到执行时间较长的查询。参见第 7.4.5 节，“慢查询日志”。

如果在错误日志中找到文本`mysqld restarted`（通常是一个名为`*`host_name`*.err`的文件），那么你可能已经找到了导致**mysqld**失败的查询。如果发生这种情况，你应该使用**myisamchk**检查所有表（参见第七章，*MySQL 服务器管理*），并测试 MySQL 日志文件中的查询是否有失败的情况。如果找到这样的查询，首先尝试升级到最新的 MySQL 版本。如果这没有帮助，请报告一个 bug，参见第 1.5 节，“如何报告错误或问题”。

如果你使用`myisam_recover_options`系统变量启动了**mysqld**，MySQL 会自动检查并尝试修复`MyISAM`表，如果它们被标记为'未正确关闭'或'崩溃'。如果发生这种情况，MySQL 会在`hostname.err`文件中写入一个条目`'Warning: Checking table ...'`，接着是`Warning: Repairing table`，如果需要修复表。如果你在**mysqld**在意外死机之前没有出现大量这些错误，那么可能出现了问题，需要进一步调查。参见 Section 7.1.7, “Server Command Options”。

当服务器检测到`MyISAM`表损坏时，它会将额外信息写入错误日志，例如源文件的名称和行号，以及访问该表的线程列表。例如：`Got an error from thread_id=1, mi_dynrec.c:368`。这是在错误报告中包含的有用信息。

如果**mysqld**意外死机，这并不是一个好兆头，但在这种情况下，你不应该调查`Checking table...`的消息，而是应该尝试找出**mysqld**为何死机。
