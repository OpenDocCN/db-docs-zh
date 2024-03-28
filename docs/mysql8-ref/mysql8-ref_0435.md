# 8.4.5 MySQL Enterprise Audit

> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log.html)

8.4.5.1 MySQL Enterprise Audit 元素

8.4.5.2 安装或卸载 MySQL Enterprise Audit

8.4.5.3 MySQL Enterprise Audit 安全考虑

8.4.5.4 审计日志文件格式

8.4.5.5 配置审计日志记录特性

8.4.5.6 读取审计日志文件

8.4.5.7 审计日志过滤

8.4.5.8 编写审计日志过滤器定义

8.4.5.9 禁用审计日志记录

8.4.5.10 传统模式审计日志过滤

8.4.5.11 审计日志参考

8.4.5.12 审计日志限制

注意

MySQL Enterprise Audit 是包含在 MySQL Enterprise Edition 中的扩展，这是一个商业产品。要了解更多关于商业产品的信息，请参阅 [`www.mysql.com/products/`](https://www.mysql.com/products/)。

MySQL Enterprise Edition 包括 MySQL Enterprise Audit，使用名为 `audit_log` 的服务器插件实现。MySQL Enterprise Audit 使用开放的 MySQL Audit API，以启用对特定 MySQL 服务器上执行的连接和查询活动的标准、基于策略的监视、记录和阻止。设计符合 Oracle 审计规范，MySQL Enterprise Audit 为受内部和外部监管指导的应用程序提供了一个开箱即用、易于使用的审计和合规解决方案。

安装后，审计插件使 MySQL 服务器能够生成一个包含服务器活动审计记录的日志文件。日志内容包括客户端连接和断开的时间，以及连接时执行的操作，例如访问哪些数据库和表。从 MySQL 8.0.30 开始，您可以添加每个查询的时间和大小的统计信息以检测异常值。

默认情况下，MySQL Enterprise Audit 使用 `mysql` 系统数据库中的表来持久存储过滤器和用户帐户数据。要使用不同的数据库，请在服务器启动时设置 `audit_log_database` 系统变量（从 MySQL 8.0.33 开始）。

安装审计插件后（请参阅 第 8.4.5.2 节，“安装或卸载 MySQL Enterprise Audit”），它会写入一个审计日志文件。默认情况下，文件名为 `audit.log`，位于服务器数据目录中。要更改文件名，请在服务器启动时设置 `audit_log_file` 系统变量。

默认情况下，审计日志文件内容以新式 XML 格式编写，不进行压缩或加密。要选择文件格式，请在服务器启动时设置`audit_log_format`系统变量。有关文件格式和内容的详细信息，请参见 Section 8.4.5.4, “Audit Log File Formats”。

有关控制日志记录方式的更多信息，包括审计日志文件命名和格式选择，请参见 Section 8.4.5.5, “配置审计日志记录特性”。要执行审计事件的过滤，请参见 Section 8.4.5.7, “审计日志过滤”。有关配置审计日志插件所使用的参数的描述，请参见审计日志选项和变量。

如果启用了审计日志插件，则性能模式（参见 Chapter 29, *MySQL 性能模式*）具有相应的仪器。要识别相关仪器，请使用以下查询：

```sql
SELECT NAME FROM performance_schema.setup_instruments
WHERE NAME LIKE '%/alog/%';
```
