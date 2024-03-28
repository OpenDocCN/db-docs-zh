# 8.2.16 服务器处理过期密码

> 原文：[`dev.mysql.com/doc/refman/8.0/en/expired-password-handling.html`](https://dev.mysql.com/doc/refman/8.0/en/expired-password-handling.html)

MySQL 提供了密码过期功能，使数据库管理员可以要求用户重置他们的密码。密码可以手动过期，也可以根据自动过期策略（参见第 8.2.15 节，“密码管理”）。

`ALTER USER`语句可以启用账户密码过期。例如：

```sql
ALTER USER 'myuser'@'localhost' PASSWORD EXPIRE;
```

对于使用过期密码的账户的每个连接，服务器要么断开客户端连接，要么将客户端限制在“沙盒模式”中，其中服务器只允许客户端执行重置过期密码所需的操作。服务器采取的操作取决于客户端和服务器设置，稍后将讨论。

如果服务器断开客户端连接，它会返回一个`ER_MUST_CHANGE_PASSWORD_LOGIN`错误：

```sql
$> mysql -u myuser -p
Password: ******
ERROR 1862 (HY000): Your password has expired. To log in you must
change it using a client that supports expired passwords.
```

如果服务器将客户端限制在沙盒模式中，客户端会话内允许执行以下操作：

+   客户端可以使用`ALTER USER`或`SET PASSWORD`重置账户密码。完成后，服务器将为该会话以及使用该账户的后续连接恢复正常访问。

    注意

    尽管可以通过将过期密码重置为当前值来“重置”过期密码，但作为良好策略，最好选择一个不同的密码。数据库管理员可以通过建立适当的密码重用策略来强制执行不重复使用。参见密码重用策略。

+   在 MySQL 8.0.27 之前，客户端可以使用`SET`语句。从 MySQL 8.0.27 开始，不再允许此操作。

对于会话中不允许的任何操作，服务器会返回一个`ER_MUST_CHANGE_PASSWORD`错误：

```sql
mysql> USE performance_schema;
ERROR 1820 (HY000): You must reset your password using ALTER USER
statement before executing this statement. 
mysql> SELECT 1;
ERROR 1820 (HY000): You must reset your password using ALTER USER
statement before executing this statement.
```

这通常发生在交互式调用**mysql**客户端时，因为默认情况下这种调用会被放置在沙盒模式中。要恢复正常功能，请选择一个新密码。

对于**mysql**客户端的非交互式调用（例如，在批处理模式下），如果密码过期，服务器通常会断开客户端的连接。为了允许非交互式的**mysql**调用保持连接，以便可以更改密码（使用沙盒模式中允许的语句），请在**mysql**命令中添加`--connect-expired-password`选项。

如前所述，服务器是否断开过期密码客户端的连接或将其限制在沙盒模式取决于客户端和服务器设置的组合。以下讨论描述了相关设置及其交互方式。

注意

此讨论仅适用于密码过期的帐户。如果客户端使用未过期的密码连接，则服务器会正常处理客户端。

在客户端端，给定的客户端指示它是否可以处理过期密码的沙盒模式。对于使用 C 客户端库的客户端，有两种方法可以做到这一点：

+   在连接之前向`mysql_options()`传递`MYSQL_OPT_CAN_HANDLE_EXPIRED_PASSWORDS`标志：

    ```sql
    bool arg = 1;
    mysql_options(mysql,
                  MYSQL_OPT_CAN_HANDLE_EXPIRED_PASSWORDS,
                  &arg);
    ```

    这是在**mysql**客户端中使用的技术，如果以交互方式调用或使用`--connect-expired-password`选项，则启用`MYSQL_OPT_CAN_HANDLE_EXPIRED_PASSWORDS`。

+   在连接时向`mysql_real_connect()`传递`CLIENT_CAN_HANDLE_EXPIRED_PASSWORDS`标志：

    ```sql
    MYSQL mysql;
    mysql_init(&mysql);
    if (!mysql_real_connect(&mysql,
                            host, user, password, db,
                            port, unix_socket,
                            CLIENT_CAN_HANDLE_EXPIRED_PASSWORDS))
    {
      ... handle error ...
    }
    ```

其他 MySQL 连接器有其自己的约定，用于指示准备处理沙盒模式。请参阅您感兴趣的连接器的文档。

在服务器端，如果客户端指示它可以处理过期密码，服务器会将其置于沙盒模式。

如果客户端没有指示它可以处理过期密码（或者使用无法指示的较旧版本的客户端库），则服务器的操作取决于`disconnect_on_expired_password`系统变量的值：

+   如果`disconnect_on_expired_password`已启用（默认情况下），服务器将以`ER_MUST_CHANGE_PASSWORD_LOGIN`错误断开客户端的连接。

+   如果`disconnect_on_expired_password`已禁用，则服务器将客户端置于沙盒模式。
