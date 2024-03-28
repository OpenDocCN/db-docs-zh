# 7.4.2 错误日志

> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log.html)

7.4.2.1 错误日志配置

7.4.2.2 默认错误日志目标配置

7.4.2.3 错误事件字段

7.4.2.4 错误日志过滤类型

7.4.2.5 基于优先级的错误日志过滤 (log_filter_internal)

7.4.2.6 基于规则的错误日志过滤 (log_filter_dragnet)

7.4.2.7 以 JSON 格式记录错误日志

7.4.2.8 记录错误日志到系统日志

7.4.2.9 错误日志输出格式

7.4.2.10 错误日志文件刷新和重命名

本节讨论如何配置 MySQL 服务器以将诊断消息记录到错误日志中。有关选择错误消息字符集和语言的信息，请参见 Section 12.6, “Error Message Character Set”，以及 Section 12.12, “Setting the Error Message Language”。

错误日志包含**mysqld**启动和关闭时间的记录。它还包含在服务器启动和关闭期间以及服务器运行时发生的错误、警告和注释的诊断消息。例如，如果**mysqld**注意到需要自动检查或修复表，它会向错误日志写入一条消息。

根据错误日志配置，错误消息也可能填充到性能模式`error_log`表中，以提供对日志的 SQL 接口，并使其内容可以被查询。参见 Section 29.12.21.2, “The error_log Table”。

在某些操作系统上，如果**mysqld**异常退出，错误日志中会包含堆栈跟踪。该跟踪可用于确定**mysqld**退出的位置。参见 Section 7.9, “Debugging MySQL”。

如果用于启动**mysqld**，**mysqld_safe**可能会向错误日志写入消息。例如，当**mysqld_safe**注意到异常的**mysqld**退出时，它会重新启动**mysqld**并向错误日志写入`mysqld restarted`消息。

以下部分讨论配置错误日志的方面。
