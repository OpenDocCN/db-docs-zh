> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-disabling.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-disabling.html)

#### 8.4.5.9 禁用审计日志

`audit_log_disable`变量在 MySQL 8.0.28 中引入，允许禁用所有连接和已连接会话的审计日志记录。`audit_log_disable`变量可以在 MySQL 服务器选项文件中设置，在命令行启动字符串中设置，或者在运行时使用`SET`语句设置；例如：

```sql
SET GLOBAL audit_log_disable = true;
```

将`audit_log_disable`设置为 true 会禁用审计日志插件。当`audit_log_disable`重新设置为`false`时（默认设置），插件将重新启用。

使用`audit_log_disable = true`启动审计日志插件会生成警告（`ER_WARN_AUDIT_LOG_DISABLED`），并显示以下消息：Audit Log is disabled. Enable it with audit_log_disable = false. 将`audit_log_disable`设置为 false 也会生成警告。当`audit_log_disable`设置为 true 时，审计日志函数调用和变量更改会生成会话警告。

设置`audit_log_disable`的运行时值需要`AUDIT_ADMIN`权限，除了通常需要设置全局系统变量运行时值的`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。
