# 15.1.30 DROP SERVER 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-server.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-server.html)

```sql
DROP SERVER [ IF EXISTS ] *server_name*
```

删除名为`*`server_name`*`的服务器定义。`mysql.servers`表中的相应行将被删除。此语句需要`SUPER`权限。

删除表的服务器不会影响任何在创建时使用此连接信息的`FEDERATED`表。请参阅第 15.1.18 节，“CREATE SERVER Statement”。

`DROP SERVER` 会导致隐式提交。请参阅第 15.3.3 节，“导致隐式提交的语句”。

`DROP SERVER` 不会被写入二进制日志，无论使用的日志格式是什么。
