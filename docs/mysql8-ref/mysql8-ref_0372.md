# 8.2.12 使用部分撤销进行权限限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partial-revokes.html`](https://dev.mysql.com/doc/refman/8.0/en/partial-revokes.html)

在 MySQL 8.0.16 之前，不可能授予全局适用的权限，除了某些模式。从 MySQL 8.0.16 开始，如果启用了`partial_revokes`系统变量，则可以实现。具体来说，对于在全局级别具有权限的用户，`partial_revokes`使得可以撤销特定模式的权限，同时保留其他模式的权限。因此，施加的权限限制可能对具有全局权限但不应被允许访问某些模式的账户的管理很有用。例如，可以允许一个账户修改任何表，除了`mysql`系统模式中的表。

+   使用部分撤销

+   部分撤销与显式模式授权

+   禁用部分撤销

+   部分撤销和复制

注意

为简洁起见，此处显示的`CREATE USER`语句不包括密码。在生产环境中，请始终分配账户密码。

#### 使用部分撤销

`partial_revokes`系统变量控制是否可以对账户施加权限限制。默认情况下，`partial_revokes`被禁用，尝试部分撤销全局权限会产生错误：

```sql
mysql> CREATE USER u1;
mysql> GRANT SELECT, INSERT ON *.* TO u1;
mysql> REVOKE INSERT ON world.* FROM u1;
ERROR 1141 (42000): There is no such grant defined for user 'u1' on host '%'
```

要允许`REVOKE`操作，请启用`partial_revokes`：

```sql
SET PERSIST partial_revokes = ON;
```

`SET PERSIST`为运行中的 MySQL 实例设置一个值。它还保存该值，使其在后续服务器重启时保留。要更改运行中的 MySQL 实例的值，而不使其在后续重启时保留，使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “SET Syntax for Variable Assignment”。

启用`partial_revokes`后，部分撤销成功：

```sql
mysql> REVOKE INSERT ON world.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+------------------------------------------+
| Grants for u1@%                          |
+------------------------------------------+
| GRANT SELECT, INSERT ON *.* TO `u1`@`%`  |
| REVOKE INSERT ON `world`.* FROM `u1`@`%` |
+------------------------------------------+
```

`SHOW GRANTS`在其输出中将部分撤销列为`REVOKE`语句。结果表明`u1`具有全局`SELECT`和`INSERT`权限，只是`INSERT`无法用于`world`模式中的表。也就是说，`u1`对`world`表的访问是只读的。

服务器记录通过部分撤销在`mysql.user`系统表中实施的权限限制。如果一个账户有部分撤销，其`User_attributes`列的值具有`Restrictions`属性：

```sql
mysql> SELECT User, Host, User_attributes->>'$.Restrictions'
       FROM mysql.user WHERE User_attributes->>'$.Restrictions' <> '';
+------+------+------------------------------------------------------+
| User | Host | User_attributes->>'$.Restrictions'                   |
+------+------+------------------------------------------------------+
| u1   | %    | [{"Database": "world", "Privileges": ["INSERT"]}] |
+------+------+------------------------------------------------------+
```

注意

尽管可以对任何模式施加部分撤销，但对`mysql`系统模式的权限限制特别有用，作为防止常规账户修改系统账户的策略的一部分。请参阅防止常规账户操纵系统账户。

部分撤销操作受到以下条件的约束：

+   可以使用部分撤销对不存在的模式施加限制，但只有在撤销的权限是全局授予的情况下才可以。如果权限没有全局授予，为不存在的模式撤销它会产生错误。

+   部分撤销仅适用于模式级。您不能对仅全局适用的权限（如`FILE`或`BINLOG_ADMIN`）或表、列或例程权限使用部分撤销。

+   在权限分配中，启用`partial_revokes`会导致 MySQL 将模式名称中未转义的`_`和`%` SQL 通配符字符解释为文字字符，就好像它们已经被转义为`\_`和`\%`一样。因为这会改变 MySQL 解释权限的方式，建议在启用`partial_revokes`的安装中避免未转义的通配符字符在权限分配中出现。

如前所述，模式级权限的部分撤销在`SHOW GRANTS`输出中显示为`REVOKE`语句。这与`SHOW GRANTS`表示“普通”模式级权限的方式不同：

+   当授予时，模式级权限通过其自己的`GRANT`语句在输出中表示：

    ```sql
    mysql> CREATE USER u1;
    mysql> GRANT UPDATE ON mysql.* TO u1;
    mysql> GRANT DELETE ON world.* TO u1;
    mysql> SHOW GRANTS FOR u1;
    +---------------------------------------+
    | Grants for u1@%                       |
    +---------------------------------------+
    | GRANT USAGE ON *.* TO `u1`@`%`        |
    | GRANT UPDATE ON `mysql`.* TO `u1`@`%` |
    | GRANT DELETE ON `world`.* TO `u1`@`%` |
    +---------------------------------------+
    ```

+   当撤销时，模式级权限会简单地从输出中消失。它们不会显示为`REVOKE`语句：

    ```sql
    mysql> REVOKE UPDATE ON mysql.* FROM u1;
    mysql> REVOKE DELETE ON world.* FROM u1;
    mysql> SHOW GRANTS FOR u1;
    +--------------------------------+
    | Grants for u1@%                |
    +--------------------------------+
    | GRANT USAGE ON *.* TO `u1`@`%` |
    +--------------------------------+
    ```

当用户授予特权时，授予者对特权的任何限制都会被受让者继承，除非受让者已经拥有该特权而没有限制。考虑以下两个用户，其中一个拥有全局`SELECT`特权：

```sql
CREATE USER u1, u2;
GRANT SELECT ON *.* TO u2;
```

假设一个管理用户`admin`有一个全局但部分撤销的`SELECT`特权：

```sql
mysql> CREATE USER admin;
mysql> GRANT SELECT ON *.* TO admin WITH GRANT OPTION;
mysql> REVOKE SELECT ON mysql.* FROM admin;
mysql> SHOW GRANTS FOR admin;
+------------------------------------------------------+
| Grants for admin@%                                   |
+------------------------------------------------------+
| GRANT SELECT ON *.* TO `admin`@`%` WITH GRANT OPTION |
| REVOKE SELECT ON `mysql`.* FROM `admin`@`%`          |
+------------------------------------------------------+
```

如果`admin`全局授予`u1`和`u2``SELECT`，则每个用户的结果不同：

+   如果`admin`全局授予`u1``SELECT`，而`u1`一开始没有`SELECT`特权，`u1`会继承`admin`的特权限制：

    ```sql
    mysql> GRANT SELECT ON *.* TO u1;
    mysql> SHOW GRANTS FOR u1;
    +------------------------------------------+
    | Grants for u1@%                          |
    +------------------------------------------+
    | GRANT SELECT ON *.* TO `u1`@`%`          |
    | REVOKE SELECT ON `mysql`.* FROM `u1`@`%` |
    +------------------------------------------+
    ```

+   另一方面，`u2`已经拥有全局`SELECT`特权而没有限制。`GRANT`只能添加到受让者的现有特权，而不能减少它们，因此如果`admin`全局授予`u2``SELECT`，`u2`不会继承`admin`的限制：

    ```sql
    mysql> GRANT SELECT ON *.* TO u2;
    mysql> SHOW GRANTS FOR u2;
    +---------------------------------+
    | Grants for u2@%                 |
    +---------------------------------+
    | GRANT SELECT ON *.* TO `u2`@`%` |
    +---------------------------------+
    ```

如果`GRANT`语句包括一个`AS *`user`*`子句，应用的特权限制是子句指定的用户/角色组合上的，而不是执行该语句的用户上的。有关`AS`子句的信息，请参见 Section 15.7.1.6, “GRANT Statement”。

授予给账户的新特权的限制将添加到该账户的任何现有限制中：

```sql
mysql> CREATE USER u1;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO u1;
mysql> REVOKE INSERT ON mysql.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+---------------------------------------------------------+
| Grants for u1@%                                         |
+---------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO `u1`@`%` |
| REVOKE INSERT ON `mysql`.* FROM `u1`@`%`                |
+---------------------------------------------------------+
mysql> REVOKE DELETE, UPDATE ON db2.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+---------------------------------------------------------+
| Grants for u1@%                                         |
+---------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO `u1`@`%` |
| REVOKE UPDATE, DELETE ON `db2`.* FROM `u1`@`%`          |
| REVOKE INSERT ON `mysql`.* FROM `u1`@`%`                |
+---------------------------------------------------------+
```

特权限制的聚合既适用于显式部分撤销特权（如刚刚显示的）又适用于从执行语句的用户或在`AS *`user`*`子句中提到的用户隐式继承限制时。

如果账户在模式上有特权限制：

+   该账户无法向其他账户授予对受限模式或其中任何对象的特权。

+   另一个没有限制的账户可以向受限账户授予受限模式或其中对象的特权。假设一个无限制用户执行以下语句：

    ```sql
    CREATE USER u1;
    GRANT SELECT, INSERT, UPDATE ON *.* TO u1;
    REVOKE SELECT, INSERT, UPDATE ON mysql.* FROM u1;
    GRANT SELECT ON mysql.user TO u1;          -- grant table privilege
    GRANT SELECT(Host,User) ON mysql.db TO u1; -- grant column privileges
    ```

    结果账户具有这些特权，能够在受限模式内执行有限操作：

    ```sql
    mysql> SHOW GRANTS FOR u1;
    +-----------------------------------------------------------+
    | Grants for u1@%                                           |
    +-----------------------------------------------------------+
    | GRANT SELECT, INSERT, UPDATE ON *.* TO `u1`@`%`           |
    | REVOKE SELECT, INSERT, UPDATE ON `mysql`.* FROM `u1`@`%`  |
    | GRANT SELECT (`Host`, `User`) ON `mysql`.`db` TO `u1`@`%` |
    | GRANT SELECT ON `mysql`.`user` TO `u1`@`%`                |
    +-----------------------------------------------------------+
    ```

如果账户对全局特权有限制，这些操作中的任何一个都会移除限制：

+   由没有对特权限制的账户全局授予账户特权。

+   在模式级别授予特权。

+   全局撤销特权。

考虑一个全局拥有多个特权但对`INSERT`、`UPDATE`和`DELETE`有限制的用户`u1`：

```sql
mysql> CREATE USER u1;
mysql> GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO u1;
mysql> REVOKE INSERT, UPDATE, DELETE ON mysql.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+----------------------------------------------------------+
| Grants for u1@%                                          |
+----------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO `u1`@`%`  |
| REVOKE INSERT, UPDATE, DELETE ON `mysql`.* FROM `u1`@`%` |
+----------------------------------------------------------+
```

从没有限制的帐户全局授予`u1`权限会移除权限限制。例如，要移除`INSERT`限制：

```sql
mysql> GRANT INSERT ON *.* TO u1;
mysql> SHOW GRANTS FOR u1;
+---------------------------------------------------------+
| Grants for u1@%                                         |
+---------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO `u1`@`%` |
| REVOKE UPDATE, DELETE ON `mysql`.* FROM `u1`@`%`        |
+---------------------------------------------------------+
```

在模式级别向`u1`授予权限会移除权限限制。例如，要移除`UPDATE`限制：

```sql
mysql> GRANT UPDATE ON mysql.* TO u1;
mysql> SHOW GRANTS FOR u1;
+---------------------------------------------------------+
| Grants for u1@%                                         |
+---------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO `u1`@`%` |
| REVOKE DELETE ON `mysql`.* FROM `u1`@`%`                |
+---------------------------------------------------------+
```

撤销全局特权会移除该特权，包括任何对其的限制。例如，要移除`DELETE`限制（以牺牲所有`DELETE`访问权限为代价）：

```sql
mysql> REVOKE DELETE ON *.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+-------------------------------------------------+
| Grants for u1@%                                 |
+-------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE ON *.* TO `u1`@`%` |
+-------------------------------------------------+
```

如果一个帐户在全局和模式级别都有权限，则必须在模式级别撤销两次才能实现部分撤销。假设`u1`拥有这些权限，其中`INSERT`在全局和`world`模式上都有：

```sql
mysql> CREATE USER u1;
mysql> GRANT SELECT, INSERT ON *.* TO u1;
mysql> GRANT INSERT ON world.* TO u1;
mysql> SHOW GRANTS FOR u1;
+-----------------------------------------+
| Grants for u1@%                         |
+-----------------------------------------+
| GRANT SELECT, INSERT ON *.* TO `u1`@`%` |
| GRANT INSERT ON `world`.* TO `u1`@`%`   |
+-----------------------------------------+
```

撤销`world`上的`INSERT`会撤销模式级别的特权（`SHOW GRANTS`不再显示模式级别的`GRANT`语句）：

```sql
mysql> REVOKE INSERT ON world.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+-----------------------------------------+
| Grants for u1@%                         |
+-----------------------------------------+
| GRANT SELECT, INSERT ON *.* TO `u1`@`%` |
+-----------------------------------------+
```

再次撤销`world`上的`INSERT`执行全局特权的部分撤销（`SHOW GRANTS`现在包括模式级别的`REVOKE`语句）：

```sql
mysql> REVOKE INSERT ON world.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+------------------------------------------+
| Grants for u1@%                          |
+------------------------------------------+
| GRANT SELECT, INSERT ON *.* TO `u1`@`%`  |
| REVOKE INSERT ON `world`.* FROM `u1`@`%` |
+------------------------------------------+
```

#### 部分撤销与明确的模式授予

为某些模式提供帐户访问权限，但不提供其他模式的访问权限，部分撤销提供了一种替代方法，可以在不授予全局特权的情况下明确授予模式级别的访问权限。这两种方法各有优缺点。

授予模式级别特权而不是全局特权：

+   添加新模式：默认情况下，现有帐户无法访问该模式。对于任何应该访问该模式的帐户，DBA 必须授予模式级别的访问权限。

+   添加新帐户：DBA 必须为每个应该访问的模式授予模式级别的访问权限。

在与部分撤销一起授予全局特权：

+   添加新模式：对于具有全局特权的现有帐户，可以访问该模式。对于任何应该无法访问该模式的帐户，DBA 必须添加部分撤销。

+   添加新帐户：DBA 必须授予全局特权，以及对每个受限制的模式进行部分撤销。

使用明确的模式级别授予方法更方便，适用于访问仅限于少数模式的帐户。使用部分撤销的方法更适用于对所有模式具有广泛访问权限的帐户，除了少数模式。

#### 禁用部分撤销

一旦启用，`partial_revokes` 如果任何帐户具有特权限制，则无法禁用。如果存在这样的帐户，则禁用 `partial_revokes` 将失败：

+   尝试在启动时禁用 `partial_revokes` 时，服务器会记录错误消息并启用 `partial_revokes`。

+   尝试在运行时禁用 `partial_revokes` 时，会发生错误，并且 `partial_revokes` 的值保持不变。

要在存在限制时禁用 `partial_revokes`，必须首先移除限制：

1.  确定哪些帐户具有部分撤销：

    ```sql
    SELECT User, Host, User_attributes->>'$.Restrictions'
    FROM mysql.user WHERE User_attributes->>'$.Restrictions' <> '';
    ```

1.  对于每个这样的帐户，删除其特权限制。假设前一步显示帐户 `u1` 具有这些限制：

    ```sql
    [{"Database": "world", "Privileges": ["INSERT", "DELETE"]
    ```

    可以通过多种方式进行限制移除：

    +   全局授予权限，无限制：

        ```sql
        GRANT INSERT, DELETE ON *.* TO u1;
        ```

    +   在模式级别授予权限：

        ```sql
        GRANT INSERT, DELETE ON world.* TO u1;
        ```

    +   全局撤销特权（假设不再需要）：

        ```sql
        REVOKE INSERT, DELETE ON *.* FROM u1;
        ```

    +   删除帐户本身（假设不再需要）：

        ```sql
        DROP USER u1;
        ```

删除所有特权限制后，可以禁用部分撤销：

```sql
SET PERSIST partial_revokes = OFF;
```

#### 部分撤销和复制

在复制场景中，如果任何主机上启用了 `partial_revokes`，则所有主机上都必须启用。否则，对于在复制发生的所有主机上部分撤销全局特权的 `REVOKE` 语句可能不会对所有主机产生相同的效果，可能导致复制不一致或错误。

当启用 `partial_revokes` 时，`GRANT` 语句的二进制日志中记录了扩展语法，包括发出该语句的当前用户及其当前活动角色。如果以这种方式记录的用户或角色在副本上不存在，则复制应用程序线程会在带有错误的 `GRANT` 语句处停止。确保在复制源服务器上发出或可能发出 `GRANT` 语句的所有用户帐户也存在于副本上，并且具有与源服务器上相同的角色集。
