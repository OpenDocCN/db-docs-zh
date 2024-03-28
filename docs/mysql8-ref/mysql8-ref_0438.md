> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-security.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-security.html)

#### 8.4.5.3 MySQL 企业审计安全考虑

默认情况下，审计日志插件生成的审计日志文件的内容未加密，可能包含敏感信息，如 SQL 语句的文本。出于安全考虑，审计日志文件应写入仅对 MySQL 服务器和有合法查看日志原因的用户可访问的目录。默认文件名为`audit.log`，位于数据目录中。可以通过在服务器启动时设置`audit_log_file`系统变量来更改此设置。由于日志轮换，可能存在其他审计日志文件。

为了增强安全性，启用审计日志文件加密。参见加密审计日志文件。
