# 8.2.20 账户锁定

> 原文：[`dev.mysql.com/doc/refman/8.0/en/account-locking.html`](https://dev.mysql.com/doc/refman/8.0/en/account-locking.html)

MySQL 支持使用`ACCOUNT LOCK`和`ACCOUNT UNLOCK`子句对用户账户进行锁定和解锁，用于`CREATE USER`和`ALTER USER`语句：

+   在与`CREATE USER`一起使用时，这些子句指定新账户的初始锁定状态。如果没有任何子句，则账户将以未锁定状态创建。

    如果启用了`validate_password`组件，则不允许创建没有密码的账户，即使该账户被锁定。参见第 8.4.3 节，“密码验证组件”。

+   在与`ALTER USER`一起使用时，这些子句指定现有账户的新锁定状态。如果没有任何子句，��账户的锁定状态保持不变。

    从 MySQL 8.0.19 开始，`ALTER USER ... UNLOCK`解锁由于登录失败次数过多而暂时锁定的任何账户。参见第 8.2.15 节，“密码管理”。

账户锁定状态记录在`mysql.user`系统表的`account_locked`列中。`SHOW CREATE USER`的输出指示账户是锁定还是未锁定。

如果客户端尝试连接到一个被锁定的账户，连接尝试将失败。服务器会增加`Locked_connects`状态变量，指示尝试连接到被锁定账户的次数，返回一个`ER_ACCOUNT_HAS_BEEN_LOCKED`错误，并在错误日志中写入一条消息：

```sql
Access denied for user '*user_name*'@'*host_name*'.
Account is locked.
```

锁定账户不影响使用代理用户连接的能力，代理用户假定被锁定账户的身份。它也不影响执行具有指定被锁定账户的`DEFINER`属性的存储程序或视图的能力。也就是说，锁定账户不影响使用代理账户或存储程序或视图的能力。

账户锁定功能取决于`mysql.user`系统表中是否存在`account_locked`列。对于从 MySQL 版本旧于 5.7.6 的升级，请执行 MySQL 升级过程以确保该列存在。参见第三章，“升级 MySQL”。对于没有`account_locked`列的非升级安装，服务器将所有账户视为未锁定，使用`ACCOUNT LOCK`或`ACCOUNT UNLOCK`子句会产生错误。
