> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-timezone.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-timezone.html)

#### 19.5.1.33 复制和时区

默认情况下，源和副本服务器假定它们处于相同的时区。如果在不同时区的服务器之间进行复制，则必须在源和副本上设置时区。否则，依赖源本地时间的语句将无法正确复制，例如使用`NOW()`或`FROM_UNIXTIME()`函数的语句。

请验证源和副本的系统时区（`system_time_zone`）、服务器当前时区（`time_zone`的全局值）和会话时区（`time_zone`的会话值）的设置组合是否产生正确的结果。特别是，如果`time_zone`系统变量设置为值`SYSTEM`，表示服务器时区与系统时区相同，则这可能导致源和副本应用不同的时区。例如，源可能在二进制日志中写入以下语句：

```sql
SET @@session.time_zone='SYSTEM';
```

如果此源及其副本对其系统时区设置不同，则即使副本的全局`time_zone`值已设置为与源相匹配，此语句也可能在副本上产生意外结果。有关 MySQL Server 的时区设置说明以及如何更改它们，请参阅 第 7.1.15 节，“MySQL Server 时区支持”。

请参阅 第 19.5.1.14 节，“复制和系统函数”。
