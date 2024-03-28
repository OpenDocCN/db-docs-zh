# 15.1.8 ALTER SERVER 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-server.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-server.html)

```sql
ALTER SERVER  *server_name*
    OPTIONS (*option* [, *option*] ...)
```

改变`*`server_name`*`的服务器信息，调整`CREATE SERVER`语句中允许的任何选项。相应的`mysql.servers`表中的字段将相应更新。此语句需要`SUPER`权限。

例如，要更新`USER`选项：

```sql
ALTER SERVER s OPTIONS (USER 'sally');
```

`ALTER SERVER`会导致隐式提交。参见 Section 15.3.3, “Statements That Cause an Implicit Commit”。

无论使用的日志格式如何，`ALTER SERVER`都不会被写入二进制日志。
