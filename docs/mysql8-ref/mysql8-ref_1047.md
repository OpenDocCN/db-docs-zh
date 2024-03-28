> 原文：[`dev.mysql.com/doc/refman/8.0/en/grant.html`](https://dev.mysql.com/doc/refman/8.0/en/grant.html)

#### 15.7.1.6 GRANT 语句

```sql
GRANT
    *priv_type* [(*column_list*)]
      [, *priv_type* [(*column_list*)]] ...
    ON [*object_type*] *priv_level*
    TO *user_or_role* [, *user_or_role*] ...
    [WITH GRANT OPTION]
    [AS *user*
        [WITH ROLE
            DEFAULT
          | NONE
          | ALL
          | ALL EXCEPT *role* [, *role* ] ...
          | *role* [, *role* ] ...
        ]
    ]
}

GRANT PROXY ON *user_or_role*
    TO *user_or_role* [, *user_or_role*] ...
    [WITH GRANT OPTION]

GRANT *role* [, *role*] ...
    TO *user_or_role* [, *user_or_role*] ...
    [WITH ADMIN OPTION]

*object_type*: {
    TABLE
  | FUNCTION
  | PROCEDURE
}

*priv_level*: {
    *
  | *.*
  | *db_name*.*
  | *db_name.tbl_name*
  | *tbl_name*
  | *db_name*.*routine_name*
}

*user_or_role*: {
    *user* (see Section 8.2.4, “Specifying Account Names”)
  | *role* (see Section 8.2.5, “Specifying Role Names”)
}
```

`GRANT` 语句将特权和角色分配给 MySQL 用户账户和角色。`GRANT` 语句有几个方面，描述如下主题：

+   GRANT 概述

+   对象引用指南

+   账户名称

+   MySQL 支持的特权

+   全局特权

+   数据库特权

+   表特权

+   列特权

+   存储过程特权

+   代理用户特权

+   授予角色

+   `AS` 子句和特权限制

+   其他账户特性

+   MySQL 和标准 SQL 版本的 GRANT

##### GRANT 概述

`GRANT` 语句使系统管理员能够授予特权和角色，这些可以授予给用户账户和角色。这些语法限制适用：

+   `GRANT` 不能在同一语句中混合授予特权和角色。给定的 `GRANT` 语句必须授予特权或角色之一。

+   `ON` 子句区分语句是授予特权还是角色：

    +   有了 `ON`，该语句授予特权。

    +   没有 `ON`，该语句授予角色。

    +   允许将特权和角色同时分配给一个账户，但必须使用单独的 `GRANT` 语句，每个语句的语法适用于所要授予的内容。

有关角色的更多信息，请参见 第 8.2.10 节，“使用角色”。

要使用`GRANT`授予特权，您必须具有`GRANT OPTION`特权，并且必须具有您正在授予的特权。 （或者，如果您对`mysql`系统模式中的授权表具有`UPDATE`特权，则可以授予任何账户任何特权。）当启用`read_only`系统变量时，`GRANT`还需要`CONNECTION_ADMIN`特权（或已弃用的`SUPER`特权）。

`GRANT`对所有指定的用户和角色都成功时才会成功，如果出现任何错误，则会回滚并且不会产生任何效果。只有当对所有指定的用户和角色都成功时，该语句才会被写入二进制日志。

`REVOKE`语句与`GRANT`相关，允许管理员撤销账户特权。参见 Section 15.7.1.8, “REVOKE Statement”。

每个账户名称使用 Section 8.2.4, “Specifying Account Names”中描述的格式。每个角色名称使用 Section 8.2.5, “Specifying Role Names”中描述的格式。例如：

```sql
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT 'role1', 'role2' TO 'user1'@'localhost', 'user2'@'localhost';
GRANT SELECT ON world.* TO 'role3';
```

如果省略账户或角色名称的主机名部分，则默认为`'%'`。

通常，数据库管理员首先使用`CREATE USER`创建一个账户并定义其非特权特征，如密码、是否使用安全连接以及对服务器资源访问的限制，然后使用`GRANT`定义其特权。可以使用`ALTER USER`来更改现有账户的非特权特征。例如：

```sql
CREATE USER 'jeffrey'@'localhost' IDENTIFIED BY '*password*';
GRANT ALL ON db1.* TO 'jeffrey'@'localhost';
GRANT SELECT ON db2.invoice TO 'jeffrey'@'localhost';
ALTER USER 'jeffrey'@'localhost' WITH MAX_QUERIES_PER_HOUR 90;
```

从**mysql**程序中，当成功执行`GRANT`时，会显示`Query OK, 0 rows affected`。要确定操作的结果产生了哪些特权，请使用`SHOW GRANTS`。参见 Section 15.7.7.21, “SHOW GRANTS Statement”。

重要提示

在某些情况下，`GRANT`可能会记录在服务器日志中或在客户端的历史文件中，例如`~/.mysql_history`，这意味着任何具有读取该信息权限的人都可以读取明文密码。有关在服务器日志中发生这种情况的条件以及如何控制它的信息，请参阅第 8.1.2.3 节，“密码和日志记录”。有关客户端日志记录的类似信息，请参阅第 6.5.1.3 节，“mysql 客户端日志记录”。

`GRANT`支持长达 255 个字符的主机名（在 MySQL 8.0.17 之前为 60 个字符）。用户名最多可达 32 个字符。数据库、表、列和例程名称最多可达 64 个字符。

警告

*不要尝试通过修改`mysql.user`系统表来更改用户名称的允许长度。这样做会导致不可预测的行为，甚至可能使用户无法登录到 MySQL 服务器*。除非通过第三章，“升级 MySQL”中描述的过程，否则永远不要以任何方式更改`mysql`系统模式中表的结构。

##### 对象引用指南

在`GRANT`语句中，有几个对象需要引用，尽管在许多情况下引用是可选的：账户、角色、数据库、表、列和例程名称。例如，如果账户名中的*`user_name`*或*`host_name`*值作为未引用的标识符是合法的，那么你无需对其进行引用。然而，引号是必要的，以指定包含特殊字符（如`-`）的*`user_name`*字符串，或包含特殊字符或通配符字符（例如，`'test-user'@'%.com'`）的*`host_name`*字符串。分别引用用户名和主机名。

要指定引用值：

+   将数据库、表、列和例程名称引用为标识符。

+   将用户名称和主机名引用为标识符或字符串。

+   将密码作为字符串进行引用。

有关字符串引用和标识符引用的指南，请参阅第 11.1.1 节，“字符串文字”和第 11.2 节，“模式对象名称”。

重要

如下几段所述的通配符字符`%`和`_`在 MySQL 8.0.35 中已被弃用，因此可能在未来的 MySQL 版本中被移除。

在`GRANT`语句中指定数据库名称时允许使用`_`和`%`通配符（`GRANT ... ON *`db_name`*.*`）。这意味着，例如，要在数据库名称中使用`_`字符，可以在`GRANT`语句中使用`\`转义字符指定为`\_`，以防止用户能够访问与通配符模式匹配的其他数据库（例如，`GRANT ... ON `foo\_bar`.* TO ...`）。

包含通配符的多个`GRANT`语句可能对 DML 语句产生意外效果；在解析涉及通配符的授权时，MySQL 只考虑第一个匹配的授权。换句话说，如果一个用户有两个使用通配符匹配同一数据库的数据库级授权，那么将应用首先创建的授权。考虑使用以下语句创建的数据库`db`和表`t`：

```sql
mysql> CREATE DATABASE db;
Query OK, 1 row affected (0.01 sec)

mysql> CREATE TABLE db.t (c INT);
Query OK, 0 rows affected (0.01 sec)

mysql> INSERT INTO db.t VALUES ROW(1);
Query OK, 1 row affected (0.00 sec)
```

接下来（假设当前账户是 MySQL `root`账户或具有必要权限的其他账户），我们创建一个用户`u`，然后发出两个包含通配符的`GRANT`语句，如下所示：

```sql
mysql> CREATE USER u;
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT SELECT ON `d_`.* TO u;
Query OK, 0 rows affected (0.01 sec)

mysql> GRANT INSERT ON `d%`.* TO u;
Query OK, 0 rows affected (0.00 sec)

mysql> EXIT
```

```sql
Bye
```

如果我们结束会话，然后使用**mysql**客户端再次登录，这次作为**u**，我们会发现该账户只有第一个匹配授权提供的权限，而不是第二个：

```sql
$> mysql -uu -hlocalhost
```

```sql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 8.0.37-tr Source distribution

Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input
statement.

mysql> TABLE db.t;
+------+
| c    |
+------+
|    1 |
+------+
1 row in set (0.00 sec)

mysql> INSERT INTO db.t VALUES ROW(2);
ERROR 1142 (42000): INSERT command denied to user 'u'@'localhost' for table 't'
```

在权限分配中，MySQL 在以下情况下将数据库名称中未转义的`_`和`%` SQL 通配符字符解释为字面字符：

+   当数据库名称未用于在数据库级别授予权限，而是作为授予权限给其他对象（例如表或例程）的限定符时（例如，`GRANT ... ON *`db_name`*.*`tbl_name`*`）。

+   启用`partial_revokes`会导致 MySQL 将数据库名称中未转义的`_`和`%`通配符字符解释为字面字符，就好像它们已经被转义为`\_`和`\%`一样。因为这会改变 MySQL 解释权限的方式，建议在可能启用`partial_revokes`的安装中避免未转义的通配符字符在权限分配中出现。有关更多信息，请参见 Section 8.2.12，“使用部分撤销进行权限限制”。

##### 账户名称

在`GRANT`语句中的*`user`*值表示适用于该语句的 MySQL 账户。为了允许向来自任意主机的用户授予权限，MySQL 支持以`'*`user_name`*'@'*`host_name`*'`形式指定*`user`*值。

您可以在主机名中指定通配符。例如，`'*`user_name`*'@'%.example.com'`适用于`example.com`域中的*`user_name`*，而`'*`user_name`*'@'198.51.100.%'`适用于`198.51.100`类 C 子网中的*`user_name`*。

简单形式的`'*`user_name`*'`是`'*`user_name`*'@'%'`的同义词。

注意

MySQL 自动将授予`'*`username`*'@'%'`的所有权限也分配给`'*`username`*'@'localhost'`帐户。此行为在 MySQL 8.0.35 及更高版本中已弃用，并可能在将来的 MySQL 版本中删除。

*MySQL 不支持用户名称中的通配符*。要引用匿名用户，请使用带有空用户名称的帐户在`GRANT`语句中指定：

```sql
GRANT ALL ON test.* TO ''@'localhost' ...;
```

在这种情况下，任何使用正确密码从本地主机连接的用户都被允许访问匿名用户帐户关联的权限。

有关帐户名称中用户名和主机名值的附加信息，请参阅 Section 8.2.4, “Specifying Account Names”。

警告

如果允许本地匿名用户连接到 MySQL 服务器，则还应将所有本地用户的权限授予为`'*`user_name`*'@'localhost'`。否则，当命名用户尝试从本地计算机登录到 MySQL 服务器时，将在`mysql.user`系统表中使用`localhost`的匿名用户帐户。有关详细信息，请参阅 Section 8.2.6, “Access Control, Stage 1: Connection Verification”。

要确定此问题是否适用于您，请执行以下查询，列出任何匿名用户：

```sql
SELECT Host, User FROM mysql.user WHERE User='';
```

要避免刚才描述的问题，使用以下语句删除本地匿名用户帐户：

```sql
DROP USER ''@'localhost';
```

##### MySQL 支持的权限

以下表总结了可以为`GRANT`和`REVOKE`语句指定的静态和动态*`priv_type`*权限类型，以及可以授予每个权限的级别。有关每个权限的更多信息，请参阅 Section 8.2.2, “Privileges Provided by MySQL”。有关静态和动态权限之间的区别，请参阅 Static Versus Dynamic Privileges。

**Table 15.11 Permissible Static Privileges for GRANT and REVOKE**

| 权限 | 意义和可授予级别 |
| --- | --- |
| [`ALL [PRIVILEGES]`](privileges-provided.html#priv_all) | 在指定的访问级别授予所有权限，除了`GRANT OPTION`和`PROXY`。 |
| `ALTER` | 启用使用`ALTER TABLE`。级别：全局，数据库，表。 |
| `ALTER ROUTINE` | 启用存储过程的修改或删除。级别：全局，数据库，存储过程。 |
| `CREATE` | 启用数据库和表的创建。级别：全局，数据库，表。 |
| `CREATE ROLE` | 启用角色的创建。级别：全局。 |
| `CREATE ROUTINE` | 启用存储过程的创建。级别：全局，数据库。 |
| `CREATE TABLESPACE` | 启用创建、修改或删除表空间和日志文件组。级别：全局。 |
| `CREATE TEMPORARY TABLES` | 启用使用`CREATE TEMPORARY TABLE`。级别：全局，数据库。 |
| `CREATE USER` | 启用使用`CREATE USER`、`DROP USER`、`RENAME USER`和`REVOKE ALL PRIVILEGES`。级别：全局。 |
| `CREATE VIEW` | 启用视图的创建或修改。级别：全局，数据库，表。 |
| `DELETE` | 启用`DELETE`的使用。级别：全局，数据库，表。 |
| `DROP` | 启用数据库、表和视图的删除。级别：全局，数据库，表。 |
| `DROP ROLE` | 启用角色的删除。级别：全局。 |
| `EVENT` | 启用事件调度器的事件使用。级别：全局，数据库。 |
| `EXECUTE` | 启用用户执行存储过程。级别：全局，数据库，存储过程。 |
| `FILE` | 启用用户使服务器读取或写入文件。级别：全局。 |
| `GRANT OPTION` | 启用将权限授予或从其他帐户中移除的功能。级别：全局，数据库，表，存储过程，代理。 |
| `INDEX` | 启用索引的创建或删除。级别：全局，数据库，表。 |
| `INSERT` | 启用`INSERT`的使用。级别：全局，数据库，表，列。 |
| `LOCK TABLES` | 允许在具有`SELECT`权限的表上使用`LOCK TABLES`。级别：全局，数据库。 |
| `PROCESS` | 允许用户使用`SHOW PROCESSLIST`查看所有进程。级别：全局。 |
| `PROXY` | 允许��户代理。级别：从用户到用户。 |
| `REFERENCES` | 允许创建外键。级别：全局，数据库，表，列。 |
| `RELOAD` | 允许使用`FLUSH`操作。级别：全局。 |
| `REPLICATION CLIENT` | 允许用户查询源或副本服务器的位置。级别：全局。 |
| `REPLICATION SLAVE` | 允许副本从源读取二进制日志事件。级别：全局。 |
| `SELECT` | 允许使用`SELECT`。级别：全局，数据库，表，列。 |
| `SHOW DATABASES` | 允许使用`SHOW DATABASES`显示所有数据库。级别：全局。 |
| `SHOW VIEW` | 允许使用`SHOW CREATE VIEW`。级别：全局，数据库，表。 |
| `SHUTDOWN` | 允许使用**mysqladmin shutdown**。级别：全局。 |
| `SUPER` | 允许使用其他管理操作，如`CHANGE REPLICATION SOURCE TO`，`CHANGE MASTER TO`，`KILL`，`PURGE BINARY LOGS`，`SET GLOBAL`以及**mysqladmin debug**命令。级别：全局。 |
| `TRIGGER` | 允许触发器操作。级别：全局，数据库，表。 |
| `UPDATE` | 允许使用`UPDATE`。级别：全局，数据库，表，列。 |
| `USAGE` | “无权限”的同义词 |
| 权限 | 含义和可授权级别 |

**表 15.12 GRANT 和 REVOKE 的可允许动态权限**

| Privilege | 意义和可授权级别 |
| --- | --- |
| `APPLICATION_PASSWORD_ADMIN` | 启用双密码管理。级别：全局。 |
| `AUDIT_ABORT_EXEMPT` | 允许通过审计日志过滤器阻止的查询。级别：全局。 |
| `AUDIT_ADMIN` | 启用审计日志配置。级别：全局。 |
| `AUTHENTICATION_POLICY_ADMIN` | 启用认证策略管理。级别：全局。 |
| `BACKUP_ADMIN` | 启用备份管理。级别：全局。 |
| `BINLOG_ADMIN` | 启用二进制日志控制。级别：全局。 |
| `BINLOG_ENCRYPTION_ADMIN` | 启用二进制日志加密的激活和停用。级别：全局。 |
| `CLONE_ADMIN` | 启用克隆管理。级别：全局。 |
| `CONNECTION_ADMIN` | 启用连接限制/限制控制。级别：全局。 |
| `ENCRYPTION_KEY_ADMIN` | 启用`InnoDB`密钥轮换。级别：全局。 |
| `FIREWALL_ADMIN` | 启用防火墙规则管理，任何用户。级别：全局。 |
| `FIREWALL_EXEMPT` | 免除用户防火墙限制。级别：全局。 |
| `FIREWALL_USER` | 启用防火墙规则管理，自身。级别：全局。 |
| `FLUSH_OPTIMIZER_COSTS` | 启用优化器成本重新加载。级别：全局。 |
| `FLUSH_STATUS` | 启用状态指示器刷新。级别：全局。 |
| `FLUSH_TABLES` | 启用表刷新。级别：全局。 |
| `FLUSH_USER_RESOURCES` | 启用用户资源刷新。级别：全局。 |
| `GROUP_REPLICATION_ADMIN` | 启用组复制控制。级别：全局。 |
| `INNODB_REDO_LOG_ARCHIVE` | 启用重做日志归档管理。级别：全局。 |
| `INNODB_REDO_LOG_ENABLE` | 启用或禁用重做日志记录。级别：全局。 |
| `NDB_STORED_USER` | 启用在 SQL 节点（NDB 集群）之间共享用户或角色。级别：全局。 |
| `PASSWORDLESS_USER_ADMIN` | 启用无密码用户帐户管理。级别：全局。 |
| `PERSIST_RO_VARIABLES_ADMIN` | 启用持久化只读系统变量。级别：全局。 |
| `REPLICATION_APPLIER` | 作为复制通道的`PRIVILEGE_CHECKS_USER`。级别：全局。 |
| `REPLICATION_SLAVE_ADMIN` | 启用常规复制控制。级别：全局。 |
| `RESOURCE_GROUP_ADMIN` | 启用资源组管理。级别：全局。 |
| `RESOURCE_GROUP_USER` | 启用资源组管理。级别：全局。 |
| `ROLE_ADMIN` | 启用授予或撤销角色，使用`WITH ADMIN OPTION`。级别：全局。 |
| `SESSION_VARIABLES_ADMIN` | 启用设置受限会话系统变量。级别：全局。 |
| `SET_USER_ID` | 启用设置非自身`DEFINER`值。级别：全局。 |
| `SHOW_ROUTINE` | 启用访问存储过程定义。级别：全局。 |
| `SKIP_QUERY_REWRITE` | 不重写此用户执行的查询。级别：全局。 |
| `SYSTEM_USER` | 指定帐户为系统帐户。级别：全局。 |
| `SYSTEM_VARIABLES_ADMIN` | 启用修改或持久化全局系统变量。级别：全局。 |
| `TABLE_ENCRYPTION_ADMIN` | 启用覆盖默认加密设置。级别：全局。 |
| `TELEMETRY_LOG_ADMIN` | 启用在 AWS 上配置 MySQL HeatWave 的遥测日志。级别：全局。 |
| `TP_CONNECTION_ADMIN` | 启用线程池连接管理。级别：全局。 |
| `VERSION_TOKEN_ADMIN` | 启用版本令牌函数的使用。级别：全局。 |
| `XA_RECOVER_ADMIN` | 启用`XA RECOVER`执行。级别：全局。 |
| 权限 | 意义和可授权级别 |

触发器与表关联。要创建或删除触发器，必须具有表的`TRIGGER`权限，而不是触发器的权限。

在`GRANT` 语句中，[`ALL [PRIVILEGES]`](privileges-provided.html#priv_all) 或 `PROXY` 权限必须单独命名，不能与其他权限一起指定。[`ALL [PRIVILEGES]`](privileges-provided.html#priv_all) 代表在要授予权限的级别上可用的所有权限，但不包括`GRANT OPTION` 和 `PROXY` 权限。

MySQL 账户信息存储在 `mysql` 系统模式的表中。有关更多详细信息，请参阅 第 8.2 节“访问控制和账户管理” ，该节详细讨论了 `mysql` 系统模式和访问控制系统。

如果授权表包含包含大小写混合的数据库或表名的权限行，并且 `lower_case_table_names` 系统变量设置为非零值，则无法使用 `REVOKE` 来撤销这些权限。在这种情况下，需要直接操作授权表。(`GRANT` 在设置 `lower_case_table_names` 时不会创建这样的行，但在设置该变量之前可能已创建这样的行。`lower_case_table_names` 设置只能在服务器启动时配置。)

根据 `ON` 子句的语法，可以在几个级别授予权限。对于 `REVOKE`，相同的 `ON` 语法指定要移除的权限。

对于全局、数据库、表和例程级别，`GRANT ALL` 仅分配在您授予的级别存在的权限。例如，`GRANT ALL ON *`db_name`*.*` 是一个数据库级别的语句，因此不授予任何全局权限，如`FILE`。授予`ALL` 不会分配`GRANT OPTION` 或 `PROXY` 权限。

如果存在 *`object_type`* 子句，则当以下对象为表、存储函数或存储过程时，应指定为 `TABLE`、`FUNCTION` 或 `PROCEDURE`。

用户对数据库、表、列或例程拥有的权限形成逻辑`OR`的账户权限的逻辑和，包括全局级别。不可能通过在较低级别缺少该权限来否定在更高级别授予的权限。例如，此语句全局授予`SELECT`和`INSERT`权限：

```sql
GRANT SELECT, INSERT ON *.* TO u1;
```

全局授予的权限适用于所有数据库、表和列，即使在这些较低级别中没有授予。

从 MySQL 8.0.16 开始，如果启用了`partial_revokes`系统变量，可以显式拒绝在全局级别授予的权限：

```sql
GRANT SELECT, INSERT, UPDATE ON *.* TO u1;
REVOKE INSERT, UPDATE ON db1.* FROM u1;
```

前述语句的结果是，`SELECT`全局适用于所有表，而`INSERT`和`UPDATE`全局适用，除了`db1`中的表。对`db1`的账户访问是只读的。

权限检查过程的详细信息在第 8.2.7 节，“访问控制，阶段 2：请求验证”中介绍。

如果您为任何用户使用表、列或例程权限，服务器会检查所有用户的表、列和例程权限，这会稍微减慢 MySQL 的速度。同样，如果限制任何用户的查询、更新或连接次数，服务器必须监视这些值。

MySQL 允许您授予不存在的数据库或表的权限。对于表，要授予的权限必须包括`CREATE`权限。*这种行为是有意设计的*，旨在使数据库管理员能够为稍后创建的数据库或表准备用户账户和权限。

重要提示

*当您删除数据库或表时，MySQL 不会自动撤销任何权限*。但是，如果删除例程，则为该例程授予的任何例程级别权限将被撤销。

##### 全局权限

全局权限是管理性的，或者适用于给定服务器上的所有数据库。要分配全局权限，请使用`ON *.*`语法：

```sql
GRANT ALL ON *.* TO 'someuser'@'somehost';
GRANT SELECT, INSERT ON *.* TO 'someuser'@'somehost';
```

`CREATE TABLESPACE`、`CREATE USER`、`FILE`、`PROCESS`、`RELOAD`、`REPLICATION CLIENT`、`REPLICATION SLAVE`、`SHOW DATABASES`、`SHUTDOWN`和`SUPER`静态权限是管理权限，只能全局授予。

动态权限都是全局的，只能全局授予。

其他权限可以全局授予或在更具体的级别授予。

在全局级别授予的`GRANT OPTION`的影响对于静态和动态权限有所不同：

+   为任何静态全局权限授予的`GRANT OPTION`适用于所有静态全局权限。

+   为任何动态权限授予的`GRANT OPTION`仅适用于该动态权限。

在全局级别使用`GRANT ALL`将授予所有静态全局权限和所有当前注册的动态权限。在执行`GRANT`语句后注册的动态权限不会向任何帐户追溯授予。

MySQL 将全局权限存储在`mysql.user`系统表中。

##### 数据库权限

数据库权限适用于给定数据库中的所有对象。要分配数据库级别的权限，请使用`ON *`db_name`*.*`语法：

```sql
GRANT ALL ON mydb.* TO 'someuser'@'somehost';
GRANT SELECT, INSERT ON mydb.* TO 'someuser'@'somehost';
```

如果您使用`ON *`语法（而不是`ON *.*`），则权限将分配给默认数据库的数据库级别。如果没有默认数据库，则会出现错误。

`CREATE`、`DROP`、`EVENT`、`GRANT OPTION`、`LOCK TABLES`和`REFERENCES`权限可以在数据库级别指定。表或例程权限也可以在数据库级别指定，这样它们将适用于数据库中的所有表或例程。

MySQL 将数据库权限存储在`mysql.db`系统表中。

##### 表权限

表权限适用于给定表中的所有列。要分配表级别的权限，请使用`ON *`db_name.tbl_name`*`语法：

```sql
GRANT ALL ON mydb.mytbl TO 'someuser'@'somehost';
GRANT SELECT, INSERT ON mydb.mytbl TO 'someuser'@'somehost';
```

如果您指定*`tbl_name`*而不是*`db_name.tbl_name`*，则该语句适用于默认数据库中的*`tbl_name`*。如果没有默认数据库，则会出现错误。

表级别的*`priv_type`*值可以是`ALTER`、`CREATE VIEW`、`CREATE`、`DELETE`、`DROP`、`GRANT OPTION`、`INDEX`、`INSERT`、`REFERENCES`、`SELECT`、`SHOW VIEW`、`TRIGGER`和`UPDATE`。

表级权限适用于基本表和视图。它们不适用于使用`CREATE TEMPORARY TABLE`创建的表，即使表名匹配。有关`TEMPORARY`表权限的信息，请参见第 15.1.20.2 节，“CREATE TEMPORARY TABLE Statement”。

MySQL 将表权限存储在`mysql.tables_priv`系统表中。

##### 列权限

列权限适用于给定表中的单个列。在列级别授予权限时，每个权限都必须跟随列或列，括在括号内。

```sql
GRANT SELECT (col1), INSERT (col1, col2) ON mydb.mytbl TO 'someuser'@'somehost';
```

在列级（即使用*`column_list`*子句时）的*`priv_type`*值可以是`INSERT`、`REFERENCES`、`SELECT`和`UPDATE`。

MySQL 将列权限存储在`mysql.columns_priv`系统表中。

##### 存储例程权限

`ALTER ROUTINE`、`CREATE ROUTINE`、`EXECUTE`和`GRANT OPTION`权限适用于存储例程（过程和函数）。它们可以在全局和数据库级别授予。除了`CREATE ROUTINE`外，这些权限可以在单个例程的例程级别授予。

```sql
GRANT CREATE ROUTINE ON mydb.* TO 'someuser'@'somehost';
GRANT EXECUTE ON PROCEDURE mydb.myproc TO 'someuser'@'somehost';
```

例程级别的*`priv_type`*值可以是`ALTER ROUTINE`、`EXECUTE`和`GRANT OPTION`。`CREATE ROUTINE`不是例程级别的权限，因为您必须在全局或数据库级别具有权限才能首先创建例程。

MySQL 将例程级权限存储在`mysql.procs_priv`系统表中。

##### 代理用户权限

`PROXY` 权限允许一个用户代表另一个用户。代理用户冒充或者取代被代理用户的身份；也就是说，它承担了被代理用户的权限。

```sql
GRANT PROXY ON 'localuser'@'localhost' TO 'externaluser'@'somehost';
```

当授予`PROXY`时，它必须是`GRANT`语句中唯一命名的权限，并且唯一允许的`WITH`选项是`WITH GRANT OPTION`。

代理需要代理用户通过插件进行身份验证，当代理用户连接时，插件将返回被代理用户的名称给服务器，并且代理用户必须具有被代理用户的`PROXY`权限。有关详细信息和示例，请参见第 8.2.19 节，“代理用户”。

MySQL 将代理权限存储在`mysql.proxies_priv`系统表中。

##### 授予角色

没有`ON`子句的`GRANT`语法授予角色而不是单独的权限。角色是一组命名的权限集合；请参见第 8.2.10 节，“使用角色”。例如：

```sql
GRANT 'role1', 'role2' TO 'user1'@'localhost', 'user2'@'localhost';
```

每个要授予的角色必须存在，以及要授予的每个用户帐户或角色。从 MySQL 8.0.16 开始，无法将角色授予匿名用户。

授予角色并不会自动使角色处于活动状态。有关角色激活和停用的信息，请参见激活角色。

授予角色需要以下权限：

+   如果您拥有`ROLE_ADMIN`权限（或已弃用的`SUPER`权限），则可以向用户或角色授予或撤销任何角色。

+   如果您使用包含`WITH ADMIN OPTION`子句的`GRANT`语句授予了一个角色，那么您就能够将该角色授予其他用户或角色，或者从其他用户或角色中撤销该角色，只要在随后授予或撤销该角色时该角色处于活动状态。这包括使用`WITH ADMIN OPTION`本身的能力。

+   要授予具有`SYSTEM_USER`权限的角色，您必须具有`SYSTEM_USER`权限。

可以使用`GRANT`创建循环引用。例如：

```sql
CREATE USER 'u1', 'u2';
CREATE ROLE 'r1', 'r2';

GRANT 'u1' TO 'u1';   -- simple loop: u1 => u1
GRANT 'r1' TO 'r1';   -- simple loop: r1 => r1

GRANT 'r2' TO 'u2';
GRANT 'u2' TO 'r2';   -- mixed user/role loop: u2 => r2 => u2
```

允许循环授权引用，但不会向受权用户添加新的权限或角色，因为用户或角色已经拥有其权限和角色。

##### `AS` 子句和权限限制

从 MySQL 8.0.16 开始，`GRANT`有一个`AS *`user`* [WITH ROLE]`子句，用于指定关于语句执行所使用的权限上下文的附加信息。这种语法在 SQL 级别可见，尽管其主要目的是通过在二进制日志中显示部分撤销者强加的授权限制，从而实现所有节点之间的统一复制。有关部分撤销的信息，请参见第 8.2.12 节，“使用部分撤销进行权限限制”。

当指定`AS *`user`*`子句时，语句执行将考虑与命名用户相关联的任何权限限制，包括`WITH ROLE`指定的所有角色（如果存在）。结果是，实际授予的权限可能相对于指定的权限有所减少。

这些条件适用于`AS *`user`*`子句：

+   当命名的*`user`*具有权限限制时，`AS`才会生效（这意味着`partial_revokes`系统变量已启用）。

+   如果给定了`WITH ROLE`，则必须将所有命名的角色授予命名的*`user`*。

+   命名的*`user`*应该是一个 MySQL 帐户，指定为`'*`user_name`*'@'*`host_name`*'`，`CURRENT_USER`，或`CURRENT_USER()`。当前用户可以与`WITH ROLE`一起命名，以便执行用户希望`GRANT`以应用一组在当前会话中活动的角色不同的角色执行。

+   `AS`不能用于获取执行`GRANT`语句的用户不具备的权限。执行用户必须至少具有要授予的权限，但`AS`子句只能限制授予的权限，而不能提升它们。

+   关于要授予的权限，`AS`不能指定一个比执行`GRANT`语句的用户/角色组合拥有更多权限（更少限制）的用户。`AS`用户/角色组合可以拥有比执行用户更多的权限，但只有在语句不授予这些额外权限时才可以。

+   `AS`仅支持授予全局权限（`ON *.*`）。

+   `AS`不支持`PROXY`授权。

以下示例说明了`AS`子句的效果。创建一个具有一些全局权限以及对这些权限的限制的用户`u1`：

```sql
CREATE USER u1;
GRANT SELECT, INSERT, UPDATE, DELETE ON *.* TO u1;
REVOKE INSERT, UPDATE ON schema1.* FROM u1;
REVOKE SELECT ON schema2.* FROM u1;
```

同时创建一个角色`r1`，解除一些权限限制并将该角色授予`u1`：

```sql
CREATE ROLE r1;
GRANT INSERT ON schema1.* TO r1;
GRANT SELECT ON schema2.* TO r1;
GRANT r1 TO u1;
```

现在，使用一个没有自己权限限制的帐户，向多个用户授予相同的全局权限集，但每个用户都受`AS`子句施加的不同限制，并检查实际授予了哪些权限。

+   这里的`GRANT`语句没有`AS`子句，因此授予的权限正是指定的那些：

    ```sql
    mysql> CREATE USER u2;
    mysql> GRANT SELECT, INSERT, UPDATE ON *.* TO u2;
    mysql> SHOW GRANTS FOR u2;
    +-------------------------------------------------+
    | Grants for u2@%                                 |
    +-------------------------------------------------+
    | GRANT SELECT, INSERT, UPDATE ON *.* TO `u2`@`%` |
    +-------------------------------------------------+
    ```

+   这里的`GRANT`语句有一个`AS`子句，因此授予的权限是指定的那些，但应用了来自`u1`的限制：

    ```sql
    mysql> CREATE USER u3;
    mysql> GRANT SELECT, INSERT, UPDATE ON *.* TO u3 AS u1;
    mysql> SHOW GRANTS FOR u3;
    +----------------------------------------------------+
    | Grants for u3@%                                    |
    +----------------------------------------------------+
    | GRANT SELECT, INSERT, UPDATE ON *.* TO `u3`@`%`    |
    | REVOKE INSERT, UPDATE ON `schema1`.* FROM `u3`@`%` |
    | REVOKE SELECT ON `schema2`.* FROM `u3`@`%`         |
    +----------------------------------------------------+
    ```

    如前所述，`AS`子句只能添加权限限制；它不能提升权限。因此，尽管`u1`具有`DELETE`权限，但由于语句没有指定授予`DELETE`，所以这不包括在授予的权限中。

+   这里的`GRANT`语句的`AS`子句使角色`r1`对`u1`生效。该角色解除了`u1`的一些限制。因此，授予的权限有一些限制，但不像前一个`GRANT`语句那样多：

    ```sql
    mysql> CREATE USER u4;
    mysql> GRANT SELECT, INSERT, UPDATE ON *.* TO u4 AS u1 WITH ROLE r1;
    mysql> SHOW GRANTS FOR u4;
    +-------------------------------------------------+
    | Grants for u4@%                                 |
    +-------------------------------------------------+
    | GRANT SELECT, INSERT, UPDATE ON *.* TO `u4`@`%` |
    | REVOKE UPDATE ON `schema1`.* FROM `u4`@`%`      |
    +-------------------------------------------------+
    ```

如果一个`GRANT`语句包括一个`AS *`user`*`子句，则执行该语句的用户的权限限制将被忽略（而不是像在没有`AS`子句的情况下那样应用）。

##### 其他帐户特征

可选的`WITH`子句用于使用户能够向其他用户授予权限。`WITH GRANT OPTION`子句使用户能够将用户在指定权限级别拥有的任何权限授予其他用户。

要向一个帐户授予`GRANT OPTION`权限，而不改变其它权限，可以这样做：

```sql
GRANT USAGE ON *.* TO 'someuser'@'somehost' WITH GRANT OPTION;
```

谨慎授予`GRANT OPTION`权限，因为两个具有不同权限的用户可能能够结合权限！

你不能授予另一个用户你自己没有的权限；`GRANT OPTION`权限使你只能分配你自己拥有的权限。

请注意，当您在特定权限级别授予用户 `GRANT OPTION` 权限时，用户拥有的（或将来可能被授予的）该级别的任何权限也可以被该用户授予其他用户。假设您在数据库上授予用户 `INSERT` 权限。然后在数据库上授予 `SELECT` 权限并指定 `WITH GRANT OPTION`，那么该用户不仅可以给其他用户 `SELECT` 权限，还可以给予 `INSERT`。如果您然后在数据库上授予用户 `UPDATE` 权限，该用户可以授予 `INSERT`、`SELECT` 和 `UPDATE`。

对于非管理员用户，不应该在全局或 `mysql` 系统模式中授予 `ALTER` 权限。如果这样做，用户可以尝试通过重命名表来破坏权限系统！

有关与特定权限相关的安全风险的更多信息，请参阅 Section 8.2.2, “Privileges Provided by MySQL”。

##### MySQL 和标准 SQL 版本的 GRANT

MySQL 和标准 SQL 版本的 `GRANT` 之间最大的区别是：

+   MySQL 将权限与主机名和用户名的组合关联起来，而不仅仅是用户名。

+   标准 SQL 没有全局或数据库级别的权限，也不支持 MySQL 支持的所有权限类型。

+   MySQL 不支持标准 SQL 的 `UNDER` 权限。

+   标准 SQL 权限以分层方式结构化。如果您移除一个用户，该用户被授予的所有权限都将被撤销。如果您在 MySQL 中使用 `DROP USER` 也是如此。请参阅 Section 15.7.1.5, “DROP USER Statement”。

+   在标准 SQL 中，当您删除一个表时，该表的所有权限都将被撤销。在标准 SQL 中，当您撤销一个权限时，基于该权限授予的所有权限也将被撤销。在 MySQL 中，权限可以通过 `DROP USER` 或 `REVOKE` 语句来撤销。

+   在 MySQL 中，可以仅对表中的某些列拥有`INSERT`权限。在这种情况下，只要您为拥有`INSERT`权限的列插入值，您仍然可以在表上执行`INSERT`语句。如果严格 SQL 模式未启用，则省略的列将设置为它们的隐式默认值。在严格模式下，如果任何省略的列没有默认值，则该语句将被拒绝。（标准 SQL 要求您对所有列都拥有`INSERT`权限。）有关严格 SQL 模式和隐式默认值的信息，请参见第 7.1.11 节，“服务器 SQL 模式”和第 13.6 节，“数据类型默认值”。
