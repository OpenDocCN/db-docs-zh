> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-role.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-role.html)

#### 15.7.1.4 DROP ROLE Statement

```sql
DROP ROLE [IF EXISTS] *role* [, *role* ] ...
```

`DROP ROLE` 移除一个或多个角色（具有特权的命名集合）。要使用此语句，您必须具有全局`DROP ROLE`或`CREATE USER`特权。当启用`read_only`系统变量时，`DROP ROLE` 还需要`CONNECTION_ADMIN`特权（或已弃用的`SUPER`特权）。

从 MySQL 8.0.16 开始，具有`CREATE USER`特权的用户可以使用此语句删除已锁定或未锁定的账户。具有`DROP ROLE`特权的用户只能使用此语句删除已锁定的账户（未锁定的账户可能是用于登录到服务器的用户账户，而不仅仅是角色）。

在`mandatory_roles`系统变量值中命名的角色不能被删除。

`DROP ROLE` 对所有命名角色要么成功，要么回滚并且如果发生任何错误则不会产生影响。默认情况下，如果尝试删除不存在的角色，则会发生错误。如果给出了`IF EXISTS`子句，则该语句会对每个不存在的命名角色产生警告，而不是错误。

如果成功，该语句将被写入二进制日志，但如果失败则不会；在这种情况下，将发生回滚并且不会进行任何更改。写入二进制日志的语句包括所有命名角色。如果给出了`IF EXISTS`子句，则即使是不存在且未被删除的角色也会被包括在内。

每个角色名称使用第 8.2.5 节“指定角色名称”中描述的格式。例如：

```sql
DROP ROLE 'admin', 'developer';
DROP ROLE 'webapp'@'localhost';
```

如果省略角色名称的主机名部分，默认为`'%'`。

一个被撤销的角色会自动从授予该角色的任何用户账户（或角色）中撤销。在该账户的任何当前会话中，其调整后的特权将从执行下一个语句开始生效。

有关角色使用示例，请参见第 8.2.10 节“使用角色”。
