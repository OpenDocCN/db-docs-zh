> 原文：[`dev.mysql.com/doc/refman/8.0/en/set-default-role.html`](https://dev.mysql.com/doc/refman/8.0/en/set-default-role.html)

#### 15.7.1.9 SET DEFAULT ROLE Statement

```sql
SET DEFAULT ROLE
    {NONE | ALL | *role* [, *role* ] ...}
    TO *user* [, *user* ] ...
```

对于紧跟在`TO`关键字后面的每个*`user`*，此语句定义了用户连接到服务器并进行身份验证时或用户在会话期间执���`SET ROLE DEFAULT`语句时激活的角色。

`SET DEFAULT ROLE`是`ALTER USER ... DEFAULT ROLE`的替代语法（参见第 15.7.1.1 节，“ALTER USER Statement”）。然而，`ALTER USER`只能为单个用户设置默认角色，而`SET DEFAULT ROLE`可以为多个用户设置默认角色。另一方面，您可以为`ALTER USER`语句指定`CURRENT_USER`作为用户名，而对于`SET DEFAULT ROLE`则不行。

`SET DEFAULT ROLE`需要以下权限：

+   为另一个用户设置默认角色需要全局`CREATE USER`权限，或者对`mysql.default_roles`系统表的`UPDATE`权限。

+   为自己设置默认角色不需要特殊权限，只要你想要作为默认角色的角色已经被授予。

每个角色名称使用第 8.2.5 节，“指定角色名称”中描述的格式。例如：

```sql
SET DEFAULT ROLE 'admin', 'developer' TO 'joe'@'10.0.0.1';
```

如果省略角色名称的主机名部分，则默认为`'%'`。

在`DEFAULT ROLE`关键字后面的子句允许这些值：

+   `NONE`: 将默认设置为`NONE`（无角色）。

+   `ALL`: 将默认设置为授予给账户的所有角色。

+   `*`role`* [, *`role`* ] ...`: 将默认设置为指定的角色，这些角色必须在执行`SET DEFAULT ROLE`时存在并被授予给账户。

注意

`SET DEFAULT ROLE`和`SET ROLE DEFAULT`是不同的语句：

+   `SET DEFAULT ROLE`定义了在账户会话中默认激活哪些账户角色。

+   `SET ROLE DEFAULT`将当前会话中的活动角色设置为当前账户的默认角色。

有关角色使用示例，请参见第 8.2.10 节，“使用角色”。
