> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-role.html`](https://dev.mysql.com/doc/refman/8.0/en/create-role.html)

#### 15.7.1.2 CREATE ROLE Statement

```sql
CREATE ROLE [IF NOT EXISTS] *role* [, *role* ] ...
```

`CREATE ROLE` 创建一个或多个角色，这些角色是命名的权限集合。要使用此语句，您必须具有全局`CREATE ROLE`或`CREATE USER`权限。当启用`read_only`系统变量时，`CREATE ROLE` 还需要`CONNECTION_ADMIN`权限（或已弃用的`SUPER`权限）。

创建角色时，角色被锁定，没有密码，并分配默认的身份验证插件。（这些角色属性可以由具有全局`CREATE USER`权限的用户稍后使用`ALTER USER`语句更改。）

`CREATE ROLE` 对所有命名角色要么成功，要么回滚并且不起作用，如果发生任何错误。默认情况下，如果尝试创建已经存在的角色，则会发生错误。如果给出了`IF NOT EXISTS`子句，则该语句对每个已经存在的命名角色产生警告，而不是错误。

如果成功，该语句将写入二进制日志，但如果失败则不会写入；在这种情况下，将发生回滚，不会进行任何更改。写入二进制日志的语句包括所有命名角色。如果给出了`IF NOT EXISTS`子句，则即使已经存在且未被创建的角色也会包括在内。

每个角色名称都使用第 8.2.5 节，“指定角色名称”中描述的格式。例如：

```sql
CREATE ROLE 'admin', 'developer';
CREATE ROLE 'webapp'@'localhost';
```

角色名称的主机名部分，如果省略，默认为`'%'`。

有关角色使用示例，请参见第 8.2.10 节，“使用角色”。
