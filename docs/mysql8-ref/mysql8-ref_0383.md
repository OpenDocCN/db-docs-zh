# 8.2.23 基于 SQL 的账户活动审计

> 原文：[`dev.mysql.com/doc/refman/8.0/en/account-activity-auditing.html`](https://dev.mysql.com/doc/refman/8.0/en/account-activity-auditing.html)

应用程序可以使用以下准则执行基于 SQL 的审计，将数据库活动与 MySQL 账户关联起来。

MySQL 账户对应于`mysql.user`系统表中的行。当客户端成功连接时，服务器会将客户端验证为此表中的特定行。此行中的`User`和`Host`列值唯一标识账户，并对应于 SQL 语句中写入账户名称的`'*`user_name`*'@'*`host_name`*'`格式。

用于验证客户端的账户决定客户端拥有哪些权限。通常，可以调用`CURRENT_USER()`函数来确定客户端用户的账户是什么。其值是从账户的`user`表行的`User`和`Host`列构造的。

然而，在某些情况下，`CURRENT_USER()`值不对应于客户端用户，而对应于不同的账户。这发生在不基于客户端账户进行权限检查的情况下：

+   使用`SQL SECURITY DEFINER`特性定义的存储过程（过程和函数）

+   使用`SQL SECURITY DEFINER`特性定义的视图

+   触发器和事件

在这些情境中，权限检查针对`DEFINER`账户进行，而`CURRENT_USER()`指的是该账户，而不是调用存储过程或视图的客户端账户或导致触发器激活的账户。要确定调用用户，可以调用`USER()`函数，该函数返回一个值，指示客户端提供的实际用户名和客户端连接的主机。然而，这个值不一定直接对应`user`表中的账户，因为`USER()`值从不包含通配符，而账户值（由`CURRENT_USER()`返回）可能包含用户名和主机名通配符。

例如，空用户名称匹配任何用户，因此`''@'localhost'`账户允许客户端以任何用户名作为本地主机的匿名用户连接。在这种情况下，如果客户端以本地主机的`user1`连接，`USER()`和`CURRENT_USER()`返回不同的值：

```sql
mysql> SELECT USER(), CURRENT_USER();
+-----------------+----------------+
| USER()          | CURRENT_USER() |
+-----------------+----------------+
| user1@localhost | @localhost     |
+-----------------+----------------+
```

账户的主机名部分也可以包含通配符。如果主机名包含`'%'`或`'_'`模式字符或使用网络掩码表示法，该账户可以用于从多个主机连接的客户端，并且`CURRENT_USER()`值不会指示哪一个。例如，账户`'user2'@'%.example.com'`可以被`user2`用于从`example.com`域中的任何主机连接。如果`user2`从`remote.example.com`连接，`USER()`和`CURRENT_USER()`返回不同的值：

```sql
mysql> SELECT USER(), CURRENT_USER();
+--------------------------+---------------------+
| USER()                   | CURRENT_USER()      |
+--------------------------+---------------------+
| user2@remote.example.com | user2@%.example.com |
+--------------------------+---------------------+
```

如果应用程序必须调用`USER()`进行用户审计（例如，如果它在触发器内部进行审计），但必须能够将`USER()`值与`user`表中的账户关联起来，则必须避免在`User`或`Host`列中包含通配符的账户。具体来说，不要允许`User`为空（这会创建一个匿名用户账户），也不要在`Host`值中允许模式字符或网络掩码表示法。所有账户必须具有非空的`User`值和字面的`Host`值。

关于前面的例子，`''@'localhost'`和`'user2'@'%.example.com'`账户应该被修改，不要使用通配符：

```sql
RENAME USER ''@'localhost' TO 'user1'@'localhost';
RENAME USER 'user2'@'%.example.com' TO 'user2'@'remote.example.com';
```

如果`user2`必须能够从`example.com`域中的多个主机连接，应该为每个主机单独创建一个账户。

从`CURRENT_USER()`或`USER()`值中提取用户名称或主机名部分，请使用`SUBSTRING_INDEX()`函数：

```sql
mysql> SELECT SUBSTRING_INDEX(CURRENT_USER(),'@',1);
+---------------------------------------+
| SUBSTRING_INDEX(CURRENT_USER(),'@',1) |
+---------------------------------------+
| user1                                 |
+---------------------------------------+

mysql> SELECT SUBSTRING_INDEX(CURRENT_USER(),'@',-1);
+----------------------------------------+
| SUBSTRING_INDEX(CURRENT_USER(),'@',-1) |
+----------------------------------------+
| localhost                              |
+----------------------------------------+
```
