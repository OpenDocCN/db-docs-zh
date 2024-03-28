> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-create-user.html`](https://dev.mysql.com/doc/refman/8.0/en/show-create-user.html)

#### 15.7.7.12 显示创建用户语句

```sql
SHOW CREATE USER *user*
```

此语句显示创建指定用户的`CREATE USER`语句。如果用户不存在，则会出现错误。该语句需要对`mysql`系统模式的`SELECT`权限，除了查看当前用户的信息。对于当前用户，在`IDENTIFIED AS`子句中显示密码哈希值需要对`mysql.user`系统表的`SELECT`权限；否则，哈希值显示为`<secret>`。

要命名账户，请使用第 8.2.4 节“指定账户名称”中描述的格式。如果省略账户名的主机名部分，则默认为`'%'`。还可以指定`CURRENT_USER`或`CURRENT_USER()`来引用与当前会话关联的账户。

在`SHOW CREATE USER`的输出中，`IDENTIFIED WITH`子句中显示的密码哈希值可能包含不可打印字符，对终端显示和其他环境产生不良影响。启用`print_identified_with_as_hex`系统变量（自 MySQL 8.0.17 起可用）会导致`SHOW CREATE USER`将这些哈希值显示为十六进制字符串，而不是常规字符串文字。即使启用此变量，不包含不可打印字符的哈希值仍会显示为常规字符串文字。

```sql
mysql> CREATE USER 'u1'@'localhost' IDENTIFIED BY 'secret';
mysql> SET print_identified_with_as_hex = ON;
mysql> SHOW CREATE USER 'u1'@'localhost'\G
*************************** 1\. row ***************************
CREATE USER for u1@localhost: CREATE USER `u1`@`localhost`
IDENTIFIED WITH 'caching_sha2_password'
AS 0x244124303035240C7745603626313D613C4C10633E0A104B1E14135A544A7871567245614F4872344643546336546F624F6C7861326932752F45622F4F473273597557627139
REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK
PASSWORD HISTORY DEFAULT PASSWORD REUSE INTERVAL DEFAULT
PASSWORD REQUIRE CURRENT DEFAULT
```

要显示授予账户的权限，请使用`SHOW GRANTS`语句。参见第 15.7.7.21 节“SHOW GRANTS Statement”。
