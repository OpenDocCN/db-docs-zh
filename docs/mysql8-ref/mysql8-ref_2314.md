> 原文：[`dev.mysql.com/doc/refman/8.0/en/table-corruption.html`](https://dev.mysql.com/doc/refman/8.0/en/table-corruption.html)

#### B.3.2.17 表损坏问题

如果您已经使用`myisam_recover_options`系统变量启动了**mysqld**，MySQL 会自动检查并尝试修复`MyISAM`表，如果它们被标记为'未正确关闭'或'崩溃'。如果发生这种情况，MySQL 会在`hostname.err`文件中写入一个条目`'Warning: Checking table ...'`，接着是`Warning: Repairing table`，如果需要修复表。如果您在没有**mysqld**意外死机的情况下频繁收到这些错误，那么可能出现了问题，需要进一步调查。

当服务器检测到`MyISAM`表损坏时，它会将额外信息写入错误日志，例如源文件的名称和行号，以及访问该表的线程列表。示例：`Got an error from thread_id=1, mi_dynrec.c:368`。这是在错误报告中包含的有用信息。

另请参阅 Section 7.1.7, “Server Command Options”，以及 Section 7.9.1.7, “Making a Test Case If You Experience Table Corruption”。
