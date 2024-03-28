# 28.3.46 INFORMATION_SCHEMA USER_ATTRIBUTES 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html)

[`USER_ATTRIBUTES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html)表（自 MySQL 8.0.21 起可用）提供有关用户评论和用户属性的信息。它从`mysql.user`系统表中获取其值。

[`USER_ATTRIBUTES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html)表具有以下列：

+   `USER`

    适用于`ATTRIBUTE`列值的帐户的用户名部分。

+   `HOST`

    适用于`ATTRIBUTE`列值的帐户的主机名部分。

+   `ATTRIBUTE`

    属于由`USER`和`HOST`列指定的帐户的用户评论、用户属性或两者。该值以 JSON 对象表示。属性的显示方式与使用带有`ATTRIBUTE`或`COMMENT`选项的[`CREATE USER`](https://dev.mysql.com/doc/refman/8.0/en/create-user.html)和[`ALTER USER`](https://dev.mysql.com/doc/refman/8.0/en/alter-user.html)语句设置的方式完全相同。评论显示为具有`comment`作为键的键值对。有关更多信息和示例，请参阅[CREATE USER Comment and Attribute Options](https://dev.mysql.com/doc/refman/8.0/en/create-user.html#create-user-comments-attributes)。

#### 注意

+   [`USER_ATTRIBUTES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html) 是一个非标准的`INFORMATION_SCHEMA`表。

+   要仅获取给定用户的用户评论作为未引用字符串，您可以使用以下查询：

    ```sql
    mysql> SELECT ATTRIBUTE->>"$.comment" AS Comment
     ->     FROM INFORMATION_SCHEMA.USER_ATTRIBUTES
     ->     WHERE USER='bill' AND HOST='localhost';
    +-----------+
    | Comment   |
    +-----------+
    | A comment |
    +-----------+
    ```

    同样，您可以使用其键获取给定用户属性的未引用值。

+   在 MySQL 8.0.22 之前，任何人都可以访问[`USER_ATTRIBUTES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html)的内容。从 MySQL 8.0.22 开始，可以按以下方式访问[`USER_ATTRIBUTES`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-user-attributes-table.html)的内容：

    +   如果：

        +   当前线程是一个复制线程。

        +   访问控制系统尚未初始化（例如，服务器是使用[`--skip-grant-tables`](https://dev.mysql.com/doc/refman/8.0/en/server-options.html#option_mysqld_skip-grant-tables)选项启动的）。

        +   当前经过身份验证的帐户具有对`mysql.user`系统表的[`UPDATE`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_update)或[`SELECT`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_select)权限。

        +   当前经过身份验证的帐户具有[`CREATE USER`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_create-user)和[`SYSTEM_USER`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_system-user)权限。

    +   否则，当前经过身份验证的账户可以看到该账户的行。此外，如果该账户具有`CREATE USER`权限但没有`SYSTEM_USER`权限，它可以看到所有其他没有`SYSTEM_USER`权限的账户的行。

有关指定账户注释和属性的更多信息，请参阅第 15.7.1.3 节，“CREATE USER Statement”。
