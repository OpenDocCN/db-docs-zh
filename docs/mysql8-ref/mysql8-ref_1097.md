> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-grants.html`](https://dev.mysql.com/doc/refman/8.0/en/show-grants.html)

#### 15.7.7.21 SHOW GRANTS Statement

```sql
SHOW GRANTS
    [FOR *user_or_role*
        [USING *role* [, *role*] ...]]

*user_or_role*: {
    *user* (see Section 8.2.4, “Specifying Account Names”)
  | *role* (see Section 8.2.5, “Specifying Role Names”.
}
```

此语句显示了分配给 MySQL 用户账户或角色的特权和角色，以`GRANT`语句的形式呈现，必须执行以复制特权和角色分配。

注意

要显示 MySQL 账户的非特权信息，请使用`SHOW CREATE USER`语句。请参阅 Section 15.7.7.12, “SHOW CREATE USER Statement”。

`SHOW GRANTS`需要对`mysql`系统模式的`SELECT`特权，除了显示当前用户的特权和角色。

要为`SHOW GRANTS`命名账户或角色，请使用与`GRANT`语句相同的格式（例如，`'jeffrey'@'localhost'`）：

```sql
mysql> SHOW GRANTS FOR 'jeffrey'@'localhost';
+------------------------------------------------------------------+
| Grants for jeffrey@localhost                                     |
+------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `jeffrey`@`localhost`                      |
| GRANT SELECT, INSERT, UPDATE ON `db1`.* TO `jeffrey`@`localhost` |
+------------------------------------------------------------------+
```

如果省略主机部分，则默认为`'%'`。有关指定账户和角色名称的其他信息，请参阅 Section 8.2.4, “Specifying Account Names”和 Section 8.2.5, “Specifying Role Names”。

要显示授予当前用户的特权（您用于连接到服务器的账户）可以使用以下任何语句之一：

```sql
SHOW GRANTS;
SHOW GRANTS FOR CURRENT_USER;
SHOW GRANTS FOR CURRENT_USER();
```

如果在定义者上下文中使用`SHOW GRANTS FOR CURRENT_USER`（或任何等效语法），比如在以定义者而不是调用者特权执行的存储过程中，显示的授权是定义者的而不是调用者的。

在 MySQL 8.0 中与之前的系列相比，`SHOW GRANTS`不再在其全局特权输出中显示`ALL PRIVILEGES`，因为全局级别的`ALL PRIVILEGES`的含义取决于定义了哪些动态特权。相反，`SHOW GRANTS`明确列出了每个授予的全局特权：

```sql
mysql> SHOW GRANTS FOR 'root'@'localhost';
+---------------------------------------------------------------------+
| Grants for root@localhost                                           |
+---------------------------------------------------------------------+
| GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, RELOAD,         |
| SHUTDOWN, PROCESS, FILE, REFERENCES, INDEX, ALTER, SHOW DATABASES,  |
| SUPER, CREATE TEMPORARY TABLES, LOCK TABLES, EXECUTE, REPLICATION   |
| SLAVE, REPLICATION CLIENT, CREATE VIEW, SHOW VIEW, CREATE ROUTINE,  |
| ALTER ROUTINE, CREATE USER, EVENT, TRIGGER, CREATE TABLESPACE,      |
| CREATE ROLE, DROP ROLE ON *.* TO `root`@`localhost` WITH GRANT      |
| OPTION                                                              |
| GRANT PROXY ON ''@'' TO `root`@`localhost` WITH GRANT OPTION        |
+---------------------------------------------------------------------+
```

处理`SHOW GRANTS`输出的应用程序应相应调整。

在全局级别，如果为任何静态全局特权授予了`GRANT OPTION`，则适用于所有授予的静态全局特权，但适用于单独授予的动态特权。`SHOW GRANTS`以这种方式显示全局特权：

+   一行列出所有授予的静态特权，如果有的话，包括适当时的`WITH GRANT OPTION`。

+   一行列出所有授予的动态权限，如果有的话，包括 `WITH GRANT OPTION`。

+   一行列出所有授予的动态权限，如果有的话，不带 `WITH GRANT OPTION`。

使用可选的 `USING` 子句，`SHOW GRANTS` 使您能够检查用户角色的权限。`USING` 子句中命名的每个角色必须授予用户。

假设用户 `u1` 被分配角色 `r1` 和 `r2`，如下所示：

```sql
CREATE ROLE 'r1', 'r2';
GRANT SELECT ON db1.* TO 'r1';
GRANT INSERT, UPDATE, DELETE ON db1.* TO 'r2';
CREATE USER 'u1'@'localhost' IDENTIFIED BY 'u1pass';
GRANT 'r1', 'r2' TO 'u1'@'localhost';
```

不带 `USING` 的 `SHOW GRANTS` 显示授予的角色：

```sql
mysql> SHOW GRANTS FOR 'u1'@'localhost';
+---------------------------------------------+
| Grants for u1@localhost                     |
+---------------------------------------------+
| GRANT USAGE ON *.* TO `u1`@`localhost`      |
| GRANT `r1`@`%`,`r2`@`%` TO `u1`@`localhost` |
+---------------------------------------------+
```

添加 `USING` 子句会导致语句还显示与子句中命名的每个角色相关联的权限：

```sql
mysql> SHOW GRANTS FOR 'u1'@'localhost' USING 'r1';
+---------------------------------------------+
| Grants for u1@localhost                     |
+---------------------------------------------+
| GRANT USAGE ON *.* TO `u1`@`localhost`      |
| GRANT SELECT ON `db1`.* TO `u1`@`localhost` |
| GRANT `r1`@`%`,`r2`@`%` TO `u1`@`localhost` |
+---------------------------------------------+
mysql> SHOW GRANTS FOR 'u1'@'localhost' USING 'r2';
+-------------------------------------------------------------+
| Grants for u1@localhost                                     |
+-------------------------------------------------------------+
| GRANT USAGE ON *.* TO `u1`@`localhost`                      |
| GRANT INSERT, UPDATE, DELETE ON `db1`.* TO `u1`@`localhost` |
| GRANT `r1`@`%`,`r2`@`%` TO `u1`@`localhost`                 |
+-------------------------------------------------------------+
mysql> SHOW GRANTS FOR 'u1'@'localhost' USING 'r1', 'r2';
+---------------------------------------------------------------------+
| Grants for u1@localhost                                             |
+---------------------------------------------------------------------+
| GRANT USAGE ON *.* TO `u1`@`localhost`                              |
| GRANT SELECT, INSERT, UPDATE, DELETE ON `db1`.* TO `u1`@`localhost` |
| GRANT `r1`@`%`,`r2`@`%` TO `u1`@`localhost`                         |
+---------------------------------------------------------------------+
```

注意

授予给帐户的权限始终有效，但角色不是。帐户的活动角色可以根据 `activate_all_roles_on_login` 系统变量的值、帐户默认角色以及会话内是否执行了 `SET ROLE` 而在会话间和会话内部有所不同。

MySQL 8.0.16 及更高版本支持对全局权限进行部分撤销，使得全局权限可以限制应用于特定模式（参见 第 8.2.12 节，“使用部分撤销进行权限限制”）。为了指示已经为特定模式撤销的全局模式权限，`SHOW GRANTS` 输出包括 `REVOKE` 语句：

```sql
mysql> SET PERSIST partial_revokes = ON;
mysql> CREATE USER u1;
mysql> GRANT SELECT, INSERT, DELETE ON *.* TO u1;
mysql> REVOKE SELECT, INSERT ON mysql.* FROM u1;
mysql> REVOKE DELETE ON world.* FROM u1;
mysql> SHOW GRANTS FOR u1;
+--------------------------------------------------+
| Grants for u1@%                                  |
+--------------------------------------------------+
| GRANT SELECT, INSERT, DELETE ON *.* TO `u1`@`%`  |
| REVOKE SELECT, INSERT ON `mysql`.* FROM `u1`@`%` |
| REVOKE DELETE ON `world`.* FROM `u1`@`%`         |
+--------------------------------------------------+
```

`SHOW GRANTS` 不显示对命名帐户可用但授予给不同帐户的权限。例如，如果存在匿名帐户，则命名帐户可能能够使用其权限，但 `SHOW GRANTS` 不会显示它们。

`SHOW GRANTS` 显示在 `mandatory_roles` 系统变量值中命名的强制角色如下：

+   不带 `FOR` 子句的 `SHOW GRANTS` 显示当前用户的权限，并包括强制角色。

+   `SHOW GRANTS FOR *`user`*` 显示命名用户的权限，不包括强制角色。

这种行为有利于使用`SHOW GRANTS FOR *`user`*`输出的应用程序，以确定哪些权限明确授予了指定用户。如果输出包括强制角色，将很难区分明确授予用户的角色和强制角色。

对于当前用户，应用程序可以使用`SHOW GRANTS`或`SHOW GRANTS FOR CURRENT_USER`来确定权限，无论是否具有强制角色。
