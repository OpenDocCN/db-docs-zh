# 8.2.2 MySQL 提供的权限

> 原文：[`dev.mysql.com/doc/refman/8.0/en/privileges-provided.html`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html)

授予 MySQL 账户的权限确定账户可以执行哪些操作。MySQL 权限在适用的上下文和操作级别上有所不同：

+   管理权限使用户能够管理 MySQL 服务器的操作。这些权限是全局的，因为它们不针对特定数据库。

+   数据库权限适用于数据库及其中的所有对象。这些权限可以针对特定数据库授予，也可以全局授予，以便适用于所有数据库。

+   对于诸如表、索引、视图和存储过程等数据库对象的权限可以针对数据库中的特定对象授予，也可以针对数据库中给定类型的所有对象（例如，数据库中的所有表）或全局授予所有数据库中给定类型的所有对象。

权限还有静态（内置到服务器中）或动态（在运行时定义）的区别。权限是静态还是动态会影响其是否可以授予用户账户和角色。有关静态和动态权限之间的区别，请参阅静态与动态权限。

关于账户权限的信息存储在`mysql`系统数据库的授权表中。有关这些表的结构和内容的描述，请参阅第 8.2.3 节，“授权表”。MySQL 服务器在启动时将授权表的内容读入内存，并在第 8.2.13 节，“权限更改生效时”所示的情况下重新加载它们。服务器基于内存中的授权表副本做出访问控制决策。

重要

一些 MySQL 版本对授权表进行更改以添加新的权限或功能。为确保您能够利用任何新功能，每次升级 MySQL 时都要将授权表更新到当前结构。请参阅第三章，*升级 MySQL*。

以下各节总结了可用权限，提供了每个权限的更详细描述，并提供了使用指南。

+   可用权限摘要

+   静态权限描述

+   动态权限描述

+   授权指南

+   静态权限与动态权限

+   从 SUPER 迁移账户到动态权限

#### 可用权限摘要

下表显示了在 `GRANT` 和 `REVOKE` 语句中使用的静态权限名称，以及授予表格中与每个权限相关联的列名以及权限适用的上下文。

**表 8.2 GRANT 和 REVOKE 的允许静态权限**

| 权限 | 授予表格列 | 上下文 |
| --- | --- | --- |
| [`ALL [PRIVILEGES]`](privileges-provided.html#priv_all) | “所有权限”的同义词 | 服务器管理 |
| `ALTER` | `Alter_priv` | 表格 |
| `ALTER ROUTINE` | `Alter_routine_priv` | 存储过程 |
| `CREATE` | `Create_priv` | 数据库、表格或索引 |
| `CREATE ROLE` | `Create_role_priv` | 服务器管理 |
| `CREATE ROUTINE` | `Create_routine_priv` | 存储过程 |
| `CREATE TABLESPACE` | `Create_tablespace_priv` | 服务器管理 |
| `CREATE TEMPORARY TABLES` | `Create_tmp_table_priv` | 表格 |
| `CREATE USER` | `Create_user_priv` | 服务器管理 |
| `CREATE VIEW` | `Create_view_priv` | 视图 |
| `DELETE` | `Delete_priv` | 表格 |
| `DROP` | `Drop_priv` | 数据库、表格或视图 |
| `DROP ROLE` | `Drop_role_priv` | 服务器管理 |
| `EVENT` | `Event_priv` | 数据库 |
| `EXECUTE` | `Execute_priv` | 存储过程 |
| `FILE` | `File_priv` | 服务器主机文件访问 |
| `GRANT OPTION` | `Grant_priv` | 数据库、表格或存储过程 |
| `INDEX` | `Index_priv` | 表格 |
| `INSERT` | `Insert_priv` | 表格或列 |
| `LOCK TABLES` | `Lock_tables_priv` | 数据库 |
| `PROCESS` | `Process_priv` | 服务器管理 |
| `PROXY` | 查看 `proxies_priv` 表格 | 服务器管理 |
| `REFERENCES` | `References_priv` | 数据库或表 |
| `RELOAD` | `Reload_priv` | 服务器管理 |
| `REPLICATION CLIENT` | `Repl_client_priv` | 服务器管理 |
| `REPLICATION SLAVE` | `Repl_slave_priv` | 服务器管理 |
| `SELECT` | `Select_priv` | 表或列 |
| `SHOW DATABASES` | `Show_db_priv` | 服务器管理 |
| `SHOW VIEW` | `Show_view_priv` | 视图 |
| `SHUTDOWN` | `Shutdown_priv` | 服务器管理 |
| `SUPER` | `Super_priv` | 服务器管理 |
| `TRIGGER` | `Trigger_priv` | 表 |
| `UPDATE` | `Update_priv` | 表或列 |
| `USAGE` | “无权限”的同义词 | 服务器管理 |
| Privilege | Grant Table Column | Context |

下表显示了在`GRANT`和`REVOKE`语句中使用的动态权限名称，以及权限适用的上下文。

**表 8.3 GRANT 和 REVOKE 的可允许动态权限**

| Privilege | Context |
| --- | --- |
| `APPLICATION_PASSWORD_ADMIN` | 双密码管理 |
| `AUDIT_ABORT_EXEMPT` | 允许被审计日志过滤器阻止的查询 |
| `AUDIT_ADMIN` | 审计日志管理 |
| `AUTHENTICATION_POLICY_ADMIN` | 认证管理 |
| `BACKUP_ADMIN` | 备份管理 |
| `BINLOG_ADMIN` | 备份和复制管理 |
| `BINLOG_ENCRYPTION_ADMIN` | 备份和复制管理 |
| `CLONE_ADMIN` | 克隆管理 |
| `CONNECTION_ADMIN` | 服务器管理 |
| `ENCRYPTION_KEY_ADMIN` | 服务器管理 |
| `FIREWALL_ADMIN` | 防火墙管理 |
| `FIREWALL_EXEMPT` | 防火墙管理 |
| `FIREWALL_USER` | 防火墙管理 |
| `FLUSH_OPTIMIZER_COSTS` | 服务器管理 |
| `FLUSH_STATUS` | 服务器管理 |
| `FLUSH_TABLES` | 服务器管理 |
| `FLUSH_USER_RESOURCES` | 服务器管理 |
| `GROUP_REPLICATION_ADMIN` | 复制管理 |
| `GROUP_REPLICATION_STREAM` | 复制管理 |
| `INNODB_REDO_LOG_ARCHIVE` | 重做日志归档管理 |
| `INNODB_REDO_LOG_ENABLE` | 重做日志管理 |
| `MASKING_DICTIONARIES_ADMIN` | 服务器管理 |
| `NDB_STORED_USER` | NDB 集群 |
| `PASSWORDLESS_USER_ADMIN` | 认证管理 |
| `PERSIST_RO_VARIABLES_ADMIN` | 服务器管理 |
| `REPLICATION_APPLIER` | 用于复制通道的`PRIVILEGE_CHECKS_USER` |
| `REPLICATION_SLAVE_ADMIN` | 复制管理 |
| `RESOURCE_GROUP_ADMIN` | 资源组管理 |
| `RESOURCE_GROUP_USER` | 资源组管理 |
| `ROLE_ADMIN` | 服务器管理 |
| `SENSITIVE_VARIABLES_OBSERVER` | 服务器管理 |
| `SESSION_VARIABLES_ADMIN` | 服务器管理 |
| `SET_USER_ID` | 服务器管理 |
| `SHOW_ROUTINE` | 服务器管理 |
| `SKIP_QUERY_REWRITE` | 服务器管理 |
| `SYSTEM_USER` | 服务器管理 |
| `SYSTEM_VARIABLES_ADMIN` | 服务器管理 |
| `TABLE_ENCRYPTION_ADMIN` | 服务器管理 |
| `TELEMETRY_LOG_ADMIN` | 用于 AWS 上 MySQL HeatWave 的遥测日志管理 |
| `TP_CONNECTION_ADMIN` | 线程池管理 |
| `VERSION_TOKEN_ADMIN` | 服务器管理 |
| `XA_RECOVER_ADMIN` | 服务器管理 |
| 权限 | 上下文 |

#### 静态权限描述

静态权限是内置到服务器中的，与在运行时定义的动态权限相对。以下列表描述了 MySQL 中每个静态权限的可用性。

特定的 SQL 语句可能具有比此处指示的更具体的权限要求。如果是这样，那么相关语句的描述将提供详细信息。

+   `ALL`, `ALL PRIVILEGES`

    这些权限标识符是“在给定权限级别下可用的所有权限”的简写（除了 `GRANT OPTION`）。例如，在全局或表级别授予 `ALL` 将授予所有全局权限或所有表级权限。

+   `ALTER`

    允许使用 `ALTER TABLE` 语句更改表的结构。`ALTER TABLE` 还需要 `CREATE` 和 `INSERT` 权限。重命名表需要在旧表上具有 `ALTER` 和 `DROP` 权限，在新表上具有 `CREATE` 和 `INSERT` 权限。

+   `ALTER ROUTINE`

    允许使用更改或删除存储例程（存储过程和函数）的语句。对于授予权限的范围内的例程，以及用户不是例程 `DEFINER` 的用户的例程，还允许访问除例程定义之外的例程属性。

+   `CREATE`

    允许使用创建新数据库和表的语句。

+   `CREATE ROLE`

    允许使用 `CREATE ROLE` 语句。（`CREATE USER` 权限也允许使用 `CREATE ROLE` 语句。）请参阅 Section 8.2.10, “Using Roles”。

    `CREATE ROLE`和`DROP ROLE`权限不像`CREATE USER`那样强大，因为它们只能用于创建和删除帐户。它们不能像`CREATE USER`那样用于修改帐户属性或重命名帐户。请参阅用户和角色的互换性。

+   `CREATE ROUTINE`

    允许使用创建存储例程（存储过程和函数）的语句。对于授予权限的范围内的例程，以及用户不是被命名为例程`DEFINER`的用户的例程，还可以访问除例程定义之外的例程属性。

+   `CREATE TABLESPACE`

    允许使用创建、修改或删除表空间和日志文件组的语句。

+   `CREATE TEMPORARY TABLES`

    允许使用`CREATE TEMPORARY TABLE`语句创建临时表。

    会话创建临时表后，服务器不会对表进行进一步的权限检查。创建会话可以对表执行任何操作，例如`DROP TABLE`，`INSERT`，`UPDATE`或`SELECT`。更多信息，请参阅 Section 15.1.20.2, “CREATE TEMPORARY TABLE Statement”。

+   `CREATE USER`

    允许使用`ALTER USER`，`CREATE ROLE`，`CREATE USER`，`DROP ROLE`，`DROP USER`，`RENAME USER`和`REVOKE ALL PRIVILEGES`语句。

+   `CREATE VIEW`

    允许使用`CREATE VIEW`语句。

+   `DELETE`

    允许从数据库中删除表中的行。

+   `DROP`

    允许使用删除（移除）现有数据库、表和视图的语句。在分区表上使用`ALTER TABLE ... DROP PARTITION`语句需要`DROP`权限。对于`TRUNCATE TABLE`也需要`DROP`权限。

+   `DROP ROLE`

    允许使用`DROP ROLE`语句。（`CREATE USER`权限也允许使用`DROP ROLE`语句。）参见第 8.2.10 节，“使用角色”。

    `CREATE ROLE`和`DROP ROLE`权限不如`CREATE USER`强大，因为它们只能用于创建和删除账户。它们不能像`CREATE USER`那样修改账户属性或重命名账户。参见用户和角色的互换性。

+   `EVENT`

    允许使用创建、修改、删除或显示事件调度器事件的语句。

+   `EXECUTE`

    允许使用执行存储例程（存储过程和函数）的语句。对于权限被授予的范围内的例程以及用户不是例程`DEFINER`的用户，还允许访问例程定义之外的例程属性。

+   `FILE`

    影响以下操作和服务器行为：

    +   使用`LOAD DATA`和`SELECT ... INTO OUTFILE`语句以及`LOAD_FILE()`函数在服务器主机上读写文件。拥有`FILE`权限的用户可以读取服务器主机上任何世界可读或 MySQL 服务器可读的文件。（这意味着用户可以读取任何数据库目录中的文件，因为服务器可以访问这些文件。）

    +   允许在 MySQL 服务器具有写入权限的任何目录中创建新文件。这包括包含实现权限表的文件的服务器数据目录。

    +   允许使用`DATA DIRECTORY`或`INDEX DIRECTORY`表选项用于`CREATE TABLE`语句。

    作为安全措施，服务器不会覆盖现有文件。

    要限制可以读取和写入文件的位置，请将`secure_file_priv`系统变量设置为特定目录。请参阅第 7.1.8 节，“服务器系统变量”。

+   `GRANT OPTION`

    允许您向其他用户授予或撤销您自己拥有的权限。

+   `INDEX`

    允许使用创建或删除（移除）索引的语句。`INDEX`适用于现有表。如果您对表具有`CREATE`权限，可以在`CREATE TABLE`语句中包含索引定义。

+   `INSERT`

    允许向数据库中的表插入行。`INSERT`也适用于`ANALYZE TABLE`、`OPTIMIZE TABLE`和`REPAIR TABLE`表维护语句。

+   `LOCK TABLES`

    允许使用显式的`LOCK TABLES`语句来锁定具有`SELECT`权限的表。这包括使用写锁，阻止其他会话读取被锁定的表。

+   `PROCESS`

    `PROCESS`权限控制对服务器内执行的线程信息的访问（即会话执行的语句的信息）。可以通过`SHOW PROCESSLIST`语句、**mysqladmin processlist**命令、信息模式`PROCESSLIST`表和性能模式`processlist`表访问线程信息，具体如下：

    +   拥有`PROCESS`权限的用户可以访问所有线程的信息，甚至包括其他用户的线程。

    +   没有`PROCESS`权限，非匿名用户可以访问自己的线程信息，但不能访问其他用户的线程信息，匿名用户则无法访问线程信息。

    注意

    性能模式`threads`表还提供线程信息，但表访问使用不同的权限模型。请参阅第 29.12.21.8 节，“The threads Table”。

    `PROCESS`权限还允许使用`SHOW ENGINE`语句，访问`INFORMATION_SCHEMA` `InnoDB`表（名称以`INNODB_`开头的表），以及（从 MySQL 8.0.21 开始）访问`INFORMATION_SCHEMA` `FILES`表。

+   `PROXY`

    允许一个用户冒充或成为另一个用户。请参阅第 8.2.19 节，“Proxy Users”。

+   `REFERENCES`

    创建外键约束需要父表的`REFERENCES`权限。

+   `RELOAD`

    `RELOAD`使以下操作生效：

    +   使用`FLUSH`语句。

    +   使用**mysqladmin**命令等同于`FLUSH`操作：`flush-hosts`，`flush-logs`，`flush-privileges`，`flush-status`，`flush-tables`，`flush-threads`，`refresh`和`reload`。

        `reload`命令告诉服务器重新加载授权表到内存中。`flush-privileges`是`reload`的同义词。`refresh`命令关闭并重新打开日志文件并刷新所有表。其他`flush-*`xxx`*`命令执行类似于`refresh`的功能，但更具体，可能在某些情况下更可取。例如，如果要刷新日志文件，`flush-logs`比`refresh`更好。

    +   使用**mysqldump**选项执行各种`FLUSH`操作：`--flush-logs`和`--master-data`。

    +   使用`RESET MASTER`和`RESET REPLICA`（或在 MySQL 8.0.22 之前，`RESET SLAVE`）语句。

+   `REPLICATION CLIENT`

    允许使用`SHOW MASTER STATUS`、`SHOW REPLICA STATUS`和`SHOW BINARY LOGS`语句。

+   `复制从服务器`

    允许帐户请求在复制源服务器上对数据库进行的更新，使用`SHOW REPLICAS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE HOSTS`）、`SHOW RELAYLOG EVENTS`和`SHOW BINLOG EVENTS`语句。还需要此权限才能使用**mysqlbinlog**选项`--read-from-remote-server`（`-R`）、`--read-from-remote-source`和`--read-from-remote-master`。将此权限授予由副本用于连接到当前服务器作为其复制源服务器的帐户。

+   `SELECT`

    允许从数据库中选择表中的行。只有当`SELECT`语句实际访问表时，才需要`SELECT`权限。一些`SELECT`语句不访问表，可以在没有任何数据库权限的情况下执行。例如，您可以使用`SELECT`作为一个简单的计算器来评估不涉及表的表达式：

    ```sql
    SELECT 1+1;
    SELECT PI()*2;
    ```

    还需要`SELECT`权限用于读取列值的其他语句。例如，在`UPDATE`语句中，需要为在*`col_name`*=*`expr`*赋值右侧引用的列或在`DELETE`或`UPDATE`语句的`WHERE`子句中命名的列。

    用于`EXPLAIN`中使用的表或视图需要`SELECT`权限，包括视图定义中的任何基础表。

+   `显示数据库`

    通过发出`SHOW DATABASE`语句，使账户能够查看数据库名称。没有此权限的账户只能看到他们拥有某些权限的数据库，并且如果服务器是使用`--skip-show-database`选项启动的，则根本无法使用该语句。

    注意

    因为任何静态全局权限被视为所有数据库的权限，任何静态全局权限使用户可以使用`SHOW DATABASES`或通过检查`INFORMATION_SCHEMA`的`SCHEMATA`表来查看所有数据库名称，除了通过部分撤销在数据库级别限制的数据库。

+   `SHOW VIEW`

    允许使用`SHOW CREATE VIEW`语句。此权限也适用于与`EXPLAIN`一起使用的视图。

+   `SHUTDOWN`

    允许使用`SHUTDOWN`和`RESTART`语句，**mysqladmin shutdown**命令，以及`mysql_shutdown()` C API 函数。

+   `SUPER`

    `SUPER`是一个强大且影响深远的权限，不应轻易授予。如果一个账户只需要执行`SUPER`操作的子集，可能可以通过代替授予一个或多个动态权限来实现所需的权限集，每个动态权限都提供更有限的功能。请参阅动态权限描述。

    注意

    `SUPER`已被弃用，您应该期望它在将来的 MySQL 版本中被移除。请参阅从 SUPER 迁移到动态权限的账户迁移。

    `SUPER`影响以下操作和服务器行为：

    +   允许在运行时更改系统变量：

        +   允许对全局系统变量进行服务器配置更改，使用`SET GLOBAL`和`SET PERSIST`。

            相应的动态权限是`SYSTEM_VARIABLES_ADMIN`。

        +   允许设置需要特殊权限的受限会话系统变量。

            对应的动态特权是`SESSION_VARIABLES_ADMIN`。

        参见 Section 7.1.9.1，“系统变量特权”。

    +   启用对全局事务特性的更改（参见 Section 15.3.7，“SET TRANSACTION 语句”）。

        对应的动态特权是`SYSTEM_VARIABLES_ADMIN`。

    +   启用帐户启动和停止复制，包括组复制。

        对于常规复制，对应的动态特权是`REPLICATION_SLAVE_ADMIN`，对于组复制是`GROUP_REPLICATION_ADMIN`。

    +   启用使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）、`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）和`CHANGE REPLICATION FILTER`语句。

        对应的动态特权是`REPLICATION_SLAVE_ADMIN`。

    +   通过`PURGE BINARY LOGS`和`BINLOG`语句实现二进制日志控制。

        对应的动态特权是`BINLOG_ADMIN`。

    +   启用在执行视图或存储程序时设置有效授权 ID。拥有此特权的用户可以在视图或存储程序的`DEFINER`属性中指定任何帐户。

        对应的动态特权是`SET_USER_ID`。

    +   启用使用`CREATE SERVER`、`ALTER SERVER`和`DROP SERVER`语句。

    +   启用**mysqladmin debug**命令的使用。

    +   启用`InnoDB`加密密钥轮换。

        对应的动态特权是`ENCRYPTION_KEY_ADMIN`。

    +   启用执行版本令牌函数。

        对应的动态特权是`VERSION_TOKEN_ADMIN`。

    +   启用授予和撤销角色，使用`GRANT`语句的`WITH ADMIN OPTION`子句，以及`ROLES_GRAPHML()`函数结果中的非空`<graphml>`元素内容。

        相应的动态特权是`ROLE_ADMIN`。

    +   允许控制不允许非`SUPER`账户的客户端连接：

        +   允许使用`KILL`语句或**mysqladmin kill**命令终止属于其他账户的线程。（一个账户始终可以终止自己的线程。）

        +   当`SUPER`客户端连接时，服务器不执行`init_connect`系统变量内容。

        +   即使达到由`max_connections`系统变量配置的连接限制，服务器也会接受来自`SUPER`客户端的一个连接。

        +   处于离线模式（启用`offline_mode`）的服务器不会在下一个客户端请求时终止`SUPER`客户端连接，并接受来自`SUPER`客户端的新连接。

        +   当`read_only`系统变量启用时，可以执行更新操作。这适用于显式表更新，以及使用诸如`GRANT`和`REVOKE`等更新表的账户管理语句。

        前述连接控制操作对应的动态特权是`CONNECTION_ADMIN`。

    如果启用了二进制日志记录，您可能还需要`SUPER`特权来创建或更改存储函数，如 Section 27.7, “Stored Program Binary Logging”中所述。

+   `TRIGGER`

    启用触发器操作。您必须对表具有此特权才能为该表创建、删除、执行或显示触发器。

    当触发器被激活（由具有执行`INSERT`、`UPDATE`或`DELETE`语句权限的用户激活，与触发器相关联的表），触发器执行要求定义触发器的用户仍然对表具有`TRIGGER`特权。

+   `UPDATE`

    允许更新数据库中表中的行。

+   `USAGE`

    此特权说明符代表“无特权”。它在全局级别与`GRANT`一起使用，用于指定诸如`WITH GRANT OPTION`之类的子句，而不在特权列表中命名特定帐户特权。`SHOW GRANTS`显示`USAGE`以指示帐户在特权级别上没有特权。

#### 动态特权描述

动态特权在运行时定义，与内置于服务器中的静态特权相对。以下列表描述了 MySQL 中每个可用的动态特权。

大多数动态特权在服务器启动时定义。其他特权由特定组件或插件定义，如特权描述中所示。在这种情况下，除非启用定义它的组件或插件，否则该特权不可用。

特定的 SQL 语句可能具有比此处指示的更具体的特权要求。如果是这样，相关语句的描述提供详细信息。

+   `APPLICATION_PASSWORD_ADMIN`（在 MySQL 8.0.14 中添加）

    对于双密码功能，此特权允许使用`RETAIN CURRENT PASSWORD`和`DISCARD OLD PASSWORD`子句，适用于您自己的帐户的`ALTER USER`和`SET PASSWORD`语句。大多数用户只需要一个密码，因此需要此特权来操作自己的次要密码。

    如果要允许帐户操作所有帐户的次要密码，则应授予`CREATE USER`特权，而不是`APPLICATION_PASSWORD_ADMIN`。

    有关双密码使用的更多信息，请参见第 8.2.15 节，“密码管理”。

+   `AUDIT_ABORT_EXEMPT`（在 MySQL 8.0.28 中添加）

    允许在审计日志过滤器中由“中止”项目阻止的查询。此特权由`audit_log`插件定义；请参见第 8.4.5 节，“MySQL 企业审计”。

    在 MySQL 8.0.28 或更高版本中创建的带有`SYSTEM_USER`权限的帐户在创建时会自动分配`AUDIT_ABORT_EXEMPT`权限。在进行 MySQL 8.0.28 或更高版本的升级过程中，如果没有现有帐户被分配该权限，则具有`SYSTEM_USER`权限的现有帐户也会被分配`AUDIT_ABORT_EXEMPT`权限。因此，具有`SYSTEM_USER`权限的帐户可用于在审核配置错误后恢复对系统的访问。

+   `AUDIT_ADMIN`

    启用审核日志配置。此权限由`audit_log`插件定义；请参阅第 8.4.5 节，“MySQL 企业审计”。

+   `BACKUP_ADMIN`

    启用执行`LOCK INSTANCE FOR BACKUP`语句和访问性能模式`log_status`表。

    注意

    除了`BACKUP_ADMIN`权限外，还需要对`log_status`表的`SELECT`权限才能访问。

    在从早期版本升级到 MySQL 8.0 时，具有`RELOAD`权限的用户在执行就地升级时会自动被授予`BACKUP_ADMIN`权限。

+   `AUTHENTICATION_POLICY_ADMIN`（MySQL 8.0.27 中新增）

    `authentication_policy`系统变量对`CREATE USER`和`ALTER USER`语句中的身份验证相关子句的使用施加了一定的约束。具有`AUTHENTICATION_POLICY_ADMIN`权限的用户不受这些约束的限制。（对于否则不允许的语句会发出警告。）

    有关`authentication_policy`强加的约束的详细信息，请参阅该变量的描述。

+   `BINLOG_ADMIN`

    通过`PURGE BINARY LOGS`和`BINLOG`语句启用二进制日志控制。

+   `BINLOG_ENCRYPTION_ADMIN`

    启用设置系统变量`binlog_encryption`，该变量激活或停用二进制日志文件和中继日志文件的加密。这种能力不是由`BINLOG_ADMIN`、`SYSTEM_VARIABLES_ADMIN` 或 `SESSION_VARIABLES_ADMIN` 权限提供的。相关的系统变量`binlog_rotate_encryption_master_key_at_startup`，在服务器重新启动时自动旋转二进制日志主密钥，不需要此权限。

+   `CLONE_ADMIN`

    启用执行`CLONE`语句。包括`BACKUP_ADMIN` 和 `SHUTDOWN` 权限。

+   `CONNECTION_ADMIN`

    启用使用`KILL`语句或**mysqladmin kill** 命令来终止属于其他账户的线程。 (一个账户始终可以终止自己的线程。)

    启用设置与客户端连接相关的系统变量，或绕过与客户端连接相关的限制。从 MySQL 8.0.31 开始，需要`CONNECTION_ADMIN` 权限来激活 MySQL 服务器的离线模式，这是通过将`offline_mode`系统变量的值更改为`ON`来完成的。

    `CONNECTION_ADMIN` 权限使具有该权限的管理员可以绕过这些系统变量的影响：

    +   `init_connect`: 当`CONNECTION_ADMIN` 客户端连接时，服务器不会执行`init_connect`系统变量内容。

    +   `max_connections`: 即使达到由`max_connections`系统变量配置的连接限制，服务器也会接受来自`CONNECTION_ADMIN` 客户端的一个连接。

    +   `offline_mode`：处于离线模式的服务器（启用了`offline_mode`）不会在下一个客户端请求时终止`CONNECTION_ADMIN`客户端连接，并接受来自`CONNECTION_ADMIN`客户端的新连接。

    +   `read_only`：即使启用了`read_only`系统变量，也可以执行来自`CONNECTION_ADMIN`客户端的更新。这适用于显式表更新，以及更新隐式更新表的账户管理语句，如`GRANT`和`REVOKE`。

    Group Replication 组成员需要`CONNECTION_ADMIN`特权，以便在涉及的服务器中的一个处于离线模式时，Group Replication 连接不会被终止。如果使用 MySQL 通信堆栈（`group_replication_communication_stack = MYSQL`），没有此特权，处于离线模式的成员将被从组中驱逐。

+   `ENCRYPTION_KEY_ADMIN`

    启用`InnoDB`加密密钥轮换。

+   `FIREWALL_ADMIN`

    启用用户管理任何用户的防火墙规则。此特权由`MYSQL_FIREWALL`插件定义；参见第 8.4.7 节，“MySQL 企业防火墙”。

+   `FIREWALL_EXEMPT`（MySQL 8.0.27 中添加）

    拥有此特权的用户不受防火墙限制。此特权由`MYSQL_FIREWALL`插件定义；参见第 8.4.7 节，“MySQL 企业防火墙”。

+   `FIREWALL_USER`

    启用用户更新其自己的防火墙规则。此特权由`MYSQL_FIREWALL`插件定义；参见第 8.4.7 节，“MySQL 企业防火墙”。

+   `FLUSH_OPTIMIZER_COSTS`（MySQL 8.0.23 中添加）

    启用`FLUSH OPTIMIZER_COSTS`语句的使用。

+   `FLUSH_STATUS`（MySQL 8.0.23 中添加）

    启用`FLUSH STATUS`语句的使用。

+   `FLUSH_TABLES`（MySQL 8.0.23 中添加）

    启用`FLUSH TABLES`语句的使用。

+   `FLUSH_USER_RESOURCES`（MySQL 8.0.23 中添加）

    允许使用`FLUSH USER_RESOURCES`语句。

+   `GROUP_REPLICATION_ADMIN`

    允许账户启动和停止组复制，使用`START GROUP REPLICATION`和`STOP GROUP REPLICATION`语句，更改`group_replication_consistency`系统变量的全局设置，并使用`group_replication_set_write_concurrency()`和`group_replication_set_communication_protocol()`函数。授予此权限给用于管理属于复制组的服务器的账户。

+   `GROUP_REPLICATION_STREAM`

    允许用户账户用于建立组复制的组通信连接。当 MySQL 通信堆栈用于组复制时（`group_replication_communication_stack=MYSQL`），必须授予恢复用户此权限。

+   `INNODB_REDO_LOG_ARCHIVE`

    允许账户激活和停用重做日志归档。

+   `INNODB_REDO_LOG_ENABLE`

    允许使用`ALTER INSTANCE {ENABLE|DISABLE} INNODB REDO_LOG`语句启用或禁用重做日志。MySQL 8.0.21 中引入。

    参见禁用重做日志。

+   `MASKING_DICTIONARIES_ADMIN`

    允许账户使用`masking_dictionary_term_add()`和`masking_dictionary_term_remove()`组件函数添加和移除字典术语。账户还需要此动态权限使用`masking_dictionary_remove()`函数移除完整字典，该函数会移除`mysql.masking_dictionaries`表中与命名字典相关的所有术语。

    请参阅 第 8.5 节，“MySQL 企业数据脱敏和去标识化”。

+   `NDB_STORED_USER`

    使用户或角色及其权限能够在所有加入给定 NDB 集群的 `NDB` 启用的 MySQL 服务器之间共享和同步。此特权仅在启用 `NDB` 存储引擎时可用。

    对给定用户或角色的权限更改或撤销会立即与所有连接的 MySQL 服务器（SQL 节点）同步。您应该注意，不能保证来自不同 SQL 节点的影响权限的多个语句以相同顺序在所有 SQL 节点上执行。因此，强烈建议所有用户管理都从一个指定的 SQL 节点进行。

    `NDB_STORED_USER` 是一个全局特权，必须使用 `ON *.*` 进行授予或撤销。尝试为此特权设置任何其他范围将导致错误。这个特权可以授予大多数应用程序和管理用户，但不能授予系统保留帐户，如 `mysql.session@localhost` 或 `mysql.infoschema@localhost`。

    被授予 `NDB_STORED_USER` 特权的用户存储在 `NDB` 中（因此被所有 SQL 节点共享），具有此特权的角色也是如此。仅仅被授予具有 `NDB_STORED_USER` 的角色的用户 *不* 存储在 `NDB` 中；每个 `NDB` 存储的用户必须显式授予该特权。

    有关在 `NDB` 中如何工作的更详细信息，请参阅 第 25.6.13 节，“特权同步和 NDB_STORED_USER”。

    `NDB_STORED_USER` 特权从 NDB 8.0.18 开始提供。

+   `PASSWORDLESS_USER_ADMIN`（在 MySQL 8.0.27 中添加）

    此特权适用于无密码用户帐户：

    +   对于帐户创建，执行 `CREATE USER` 创建无密码帐户的用户必须具有 `PASSWORDLESS_USER_ADMIN` 特权。

    +   在复制环境中，`PASSWORDLESS_USER_ADMIN` 特权适用于复制用户，并允许为配置为无密码身份验证的用户帐户复制 `ALTER USER ... MODIFY` 语句。

    有关无密码身份验证的信息，请参阅 FIDO 无密码身份验证。

+   `PERSIST_RO_VARIABLES_ADMIN`

    对于还具有`SYSTEM_VARIABLES_ADMIN`的用户，`PERSIST_RO_VARIABLES_ADMIN`允许使用`SET PERSIST_ONLY`将全局系统变量持久化到数据目录中的`mysqld-auto.cnf`选项文件中。此语句类似于`SET PERSIST`，但不修改运行时全局系统变量值。这使得`SET PERSIST_ONLY`适用于配置只能在服务器启动时设置的只读系统变量。

    参见 第 7.1.9.1 节，“系统变量权限”。

+   `REPLICATION_APPLIER`

    启用账户作为复制通道的`PRIVILEGE_CHECKS_USER`，并在**mysqlbinlog**输出中执行`BINLOG`语句。授予此权限给通过`CHANGE REPLICATION SOURCE TO`（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`（在 MySQL 8.0.23 之前）分配的账户，以为复制通道提供安全上下文，并处理这些通道上的复制错误。除了`REPLICATION_APPLIER`权限外，您还必须为账户授予执行复制通道接收的事务或包含在**mysqlbinlog**输出中的所需权限，例如更新受影响的表。有关更多信息，请参见 第 19.3.3 节，“复制权限检查”。

+   `REPLICATION_SLAVE_ADMIN`

    允许帐户连接到复制源服务器，使用`START REPLICA`和`STOP REPLICA`语句启动和停止复制，并使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）以及`CHANGE REPLICATION FILTER`语句。授予此特权给由副本使用的帐户，以连接到当前服务器作为其复制源服务器。此特权不适用于组复制；对于组复制，请使用`GROUP_REPLICATION_ADMIN`。

+   `RESOURCE_GROUP_ADMIN`

    允许资源组管理，包括创建、修改和删除资源组，以及将线程和语句分配给资源组。拥有此特权的用户可以执行与资源组相关的任何操作。

+   `RESOURCE_GROUP_USER`

    允许将线程和语句分配给资源组。拥有此特权的用户可以使用`SET RESOURCE GROUP`语句和`RESOURCE_GROUP`优化提示。

+   `ROLE_ADMIN`

    允许授予和撤销角色，使用`GRANT`语句的`WITH ADMIN OPTION`子句，以及`ROLES_GRAPHML()`函数结果中的非空`<graphml>`元素内容。需要设置`mandatory_roles`系统变量的值。

+   `SENSITIVE_VARIABLES_OBSERVER`（在 MySQL 8.0.29 中添加）

    允许持有者查看性能模式表中敏感系统变量的值`global_variables`，`session_variables`，`variables_by_thread`，和`persisted_variables`，发出`SELECT`语句以返回它们的值，并在会话跟踪器中跟踪连接的更改。没有此权限的用户无法查看或跟踪这些系统变量的值。请参见持久化敏感系统变量。

+   `SERVICE_CONNECTION_ADMIN`

    允许连接到仅允许管理连接的网络接口（参见第 7.1.12.1 节，“连接接口”）。

+   `SESSION_VARIABLES_ADMIN`（在 MySQL 8.0.14 中添加）

    对于大多数系统变量，设置会话值不需要特殊权限，任何用户都可以执行以影响当前会话。对于一些系统变量，设置会话值可能会影响当前会话之外的效果，因此是一项受限操作。对于这些情况，`SESSION_VARIABLES_ADMIN`权限使用户能够设置会话值。

    如果系统变量受限并需要特殊权限来设置会话值，则变量描述会指示该限制。例如包括`binlog_format`，`sql_log_bin`，和`sql_log_off`。

    在 MySQL 8.0.14 之前，当添加了`SESSION_VARIABLES_ADMIN`时，受限会话系统变量只能由具有`SYSTEM_VARIABLES_ADMIN`或`SUPER`权限的用户设置。

    `SESSION_VARIABLES_ADMIN` 特权是`SYSTEM_VARIABLES_ADMIN` 和 `SUPER` 特权的子集。拥有这些特权之一的用户也被允许设置受限制的会话变量，并且通过暗示具有`SESSION_VARIABLES_ADMIN`，无需显式授予`SESSION_VARIABLES_ADMIN`。

    另请参阅第 7.1.9.1 节，“系统变量特权”。

+   `SET_USER_ID`

    允许在执行视图或存储程序时设置有效的授权 ID。拥有此特权的用户可以将任何帐户指定为视图或存储程序的`DEFINER`属性。存储程序将以指定帐户的权限执行，因此请确保遵循第 27.6 节，“存储对象访问控制”中列出的风险最小化准则。

    截至 MySQL 8.0.22，`SET_USER_ID` 还允许覆盖旨在防止（可能是无意的）导致存储对象变成孤立或导致当前孤立的存储对象被采用的安全检查。有关详细信息，请参阅孤立存储对象。

+   `SHOW_ROUTINE`（MySQL 8.0.20 中添加）

    允许用户访问所有存储例程（存储过程和函数）的定义和属性，即使用户未被命名为例程的`DEFINER`。此访问权限包括：

    +   信息模式`ROUTINES`表的内容。

    +   `SHOW CREATE FUNCTION` 和 `SHOW CREATE PROCEDURE` 语句。

    +   `SHOW FUNCTION CODE` 和 `SHOW PROCEDURE CODE` 语句。

    +   `SHOW FUNCTION STATUS` 和 `SHOW PROCEDURE STATUS` 语句。

    在 MySQL 8.0.20 之前，用户要访问未定义的例程的定义，用户必须拥有全局`SELECT` 权限，这是非常广泛的。从 8.0.20 开始，可以授予`SHOW_ROUTINE` 权限，这是一个范围更受限制的权限，允许访问例程定义。（也就是说，管理员可以从不需要的用户那里撤销全局`SELECT` 并授予`SHOW_ROUTINE`）。这使得一个账户可以备份存储例程而不需要广泛的权限。

+   `SKIP_QUERY_REWRITE`（MySQL 8.0.31 中添加）

    拥有此权限的用户发出的查询不会被`Rewriter`插件重写（参见第 7.6.4 节，“Rewriter 查询重写插件”）。

    此权限应授予发出不应被重写的管理或控制语句的用户，以及用于应用来自复制源的语句的`PRIVILEGE_CHECKS_USER`账户（参见第 19.3.3 节，“复制权限检查”）。

+   `SYSTEM_USER`（MySQL 8.0.16 中添加）

    `SYSTEM_USER` 权限区分系统用户和普通用户：

    +   拥有`SYSTEM_USER` 权限的用户是系统用户。

    +   没有`SYSTEM_USER` 权限的用户是普通用户。

    `SYSTEM_USER` 权限影响用户可以应用其它权限的账户，以及用户是否受到其他账户的保护：

    +   系统用户可以修改系统账户和普通账户。也就是说，拥有在普通账户上执行特定操作的适当权限的用户，通过拥有`SYSTEM_USER` 权限也可以在系统账户上执行该操作。只有拥有适当权限的系统用户才能修改系统账户，普通用户无法修改。

    +   拥有适当权限的普通用户可以修改普通账户，但不能修改系统账户。系统账户可以被拥有适当权限的系统用户和普通用户修改。

    这也意味着，由拥有`SYSTEM_USER` 权限的用户创建的数据库对象不能被没有该权限的用户修改或删除。对于定义者拥有此权限的例程也适用。

    更多信息，请参见第 8.2.11 节，“账户类别”。

    由于`SYSTEM_USER`权限为系统帐户提供的修改保护不适用于具有对`mysql`系统模式特权的常规帐户，因此可以直接修改该模式中的授权表。为了完全保护，请不要向常规帐户授予`mysql`模式特权。请参见防止常规帐户操纵系统帐户。

    如果使用了`audit_log`插件（参见第 8.4.5 节，“MySQL 企业审计”），从 MySQL 8.0.28 开始，具有`SYSTEM_USER`权限的帐户将自动被分配`AUDIT_ABORT_EXEMPT`权限，允许他们的查询即使在过滤器中配置了“中止”项目也能执行。具有`SYSTEM_USER`权限的帐户因此可用于在审计配置错误后恢复对系统的访问。

+   `SYSTEM_VARIABLES_ADMIN`

    影响以下操作和服务器行为：

    +   允许在运行时更改系统变量：

        +   允许使用`SET GLOBAL`和`SET PERSIST`对全局系统变量进行服务器配置更改。

        +   允许使用`SET PERSIST_ONLY`对全局系统变量进行服务器配置更改，如果用户还具有`PERSIST_RO_VARIABLES_ADMIN`权限。

        +   允许设置需要特殊权限的受限会话系统变量。实际上，`SYSTEM_VARIABLES_ADMIN`意味着未明确授予`SESSION_VARIABLES_ADMIN`。

        另请参阅第 7.1.9.1 节，“系统变量权限”。

    +   允许更改全局事务特性（参见第 15.3.7 节，“SET TRANSACTION 语句”）。

+   `TABLE_ENCRYPTION_ADMIN`（在 MySQL 8.0.16 中添加）

    当启用`table_encryption_privilege_check`时，允许用户覆盖默认加密设置；参见为模式和通用表空间定义加密默认值。

+   `TELEMETRY_LOG_ADMIN`

    启用遥测日志配置。此特权由部署在 AWS 上的 MySQL HeatWave 的`telemetry_log`插件定义。

+   `TP_CONNECTION_ADMIN`

    允许使用特权连接连接到服务器。当达到由`thread_pool_max_transactions_limit`定义的限制时，不允许新连接。特权连接忽略事务限制，并允许连接到服务器以增加事务限制、移除限制或终止运行中的事务。此特权不会默认授予任何用户。要建立特权连接，发起连接的用户必须具有`TP_CONNECTION_ADMIN`特权。

    当达到由`thread_pool_max_transactions_limit`定义的限制时，特权连接可以执行语句并启动事务。特权连接被放置在`Admin`线程组中。参见特权连接。

+   `VERSION_TOKEN_ADMIN`

    允许执行版本令牌函数。此特权由`version_tokens`插件定义；参见第 7.6.6 节，“版本令牌”。

+   `XA_RECOVER_ADMIN`

    允许执行`XA RECOVER`语句；参见第 15.3.8.1 节，“XA 事务 SQL 语句”。

    在 MySQL 8.0 之前，任何用户都可以执行 `XA RECOVER` 语句来发现未完成的准备好的 XA 事务的 XID 值，可能导致由非启动者执行提交或回滚 XA 事务。在 MySQL 8.0 中，`XA RECOVER` 仅允许拥有 `XA_RECOVER_ADMIN` 权限的用户执行，这个权限预期只授予有需要的管理用户。例如，如果 XA 应用程序崩溃并且需要找到应用程序启动的未完成事务以便回滚，那么可能会出现这种情况。这种权限要求阻止用户发现除自己之外的未完成准备好的 XA 事务的 XID 值。它不会影响 XA 事务的正常提交或回滚，因为启动它的用户知道它的 XID。

#### 权限授予指南

为账户授予它所需的权限是一个好主意。在授予 `FILE` 和管理权限时应特别小心：

+   `FILE` 可以被滥用来读取 MySQL 服务器在服务器主机上可以读取的任何文件到数据库表中。这包括所有可读取的文件和服务器数据目录中的文件。然后可以使用 `SELECT` 访问该表，将其内容传输到客户端主机。

+   `GRANT OPTION` 允许用户将他们的权限授予其他用户。拥有不同权限并且具有 `GRANT OPTION` 权限的两个用户可以组合权限。

+   `ALTER` 可能被用于通过重命名表来破坏权限系统。

+   `SHUTDOWN` 可以被滥用来通过终止服务器来完全拒绝其他用户的服务。

+   `PROCESS` 可以用于查看当前执行语句的纯文本，包括设置或更改密码的语句。

+   `SUPER` 可以用于终止其他会话或更改服务器操作方式。

+   为 `mysql` 系统数据库本身授予的权限可以用于更改密码和其他访问权限信息：

    +   密码被加密存储，因此恶意用户不能简单地读取它们以了解明文密码。但是，具有对 `mysql.user` 系统表 `authentication_string` 列的写入访问权限的用户可以更改账户的密码，然后使用该账户连接到 MySQL 服务器。

    +   为 `mysql` 系统数据库授予的 `INSERT` 或 `UPDATE` 权限使用户能够添加权限或修改现有权限。

    +   为 `mysql` 系统数据库的 `DROP` 允许用户删除权限表，甚至是数据库本身。

#### 静态权限与动态权限

MySQL 支持静态和动态权限：

+   静态权限内置于服务器中。它们始终可供授予给用户帐户，并且无法注销。

+   动态权限可以在运行时注册和注销。这会影响它们的可用性：未注册的动态权限无法授予。

例如，`SELECT` 和 `INSERT` 权限是静态的，并且始终可用，而实现它的组件已启用时才会提供动态权限。

本节的其余部分描述了 MySQL 中动态权限的工作原理。讨论中使用术语“组件”，但同样适用于插件。

注意

服务器管理员应该了解哪些服务器组件定义了动态权限。对于 MySQL 发行版，定义动态权限的组件的文档描述了这些权限。

第三方组件也可能定义动态权限；管理员应该了解这些权限，并且不安装可能会与服务器操作发生冲突或危害的组件。例如，如果两个组件都定义了相同名称的权限，则一个组件会与另一个组件发生冲突。组件开发人员可以通过选择以组件名称为前缀的权限名称来减少此类情况发生的可能性。

服务器在内存中维护已注册的动态权限集。在服务器关闭时进行注销。

通常，定义动态权限的组件在安装时会注册它们，在初始化序列期间。当卸载时，组件不会注销其已注册的动态权限。（这是当前的做法，而不是要求。也就是说，组件可以，但不会在任何时候注销其注册的权限。）

尝试注册已经注册的动态权限不会出现警告或错误。考虑以下语句序列：

```sql
INSTALL COMPONENT 'my_component';
UNINSTALL COMPONENT 'my_component';
INSTALL COMPONENT 'my_component';
```

第一个 `INSTALL COMPONENT` 语句注册了组件 `my_component` 定义的任何权限，但 `UNINSTALL COMPONENT` 不会注销它们。对于第二个 `INSTALL COMPONENT` 语句，它注册的组件权限已经被发现已经注册，但不会出现警告或错误。

动态权限仅适用于全局级别。服务器在`mysql.global_grants`系统表中存储有关动态权限对用户帐户的当前分配的信息：

+   服务器在启动时自动注册`global_grants`中命名的权限（除非提供了`--skip-grant-tables`选项）。

+   `GRANT`和`REVOKE`语句修改`global_grants`的内容。

+   在`global_grants`中列出的动态权限分配是持久的。它们在服务器关闭时不会被移除。

示例：以下语句授予用户`u1`在副本上控制复制（包括组复制）和修改系统变量所需的权限：

```sql
GRANT REPLICATION_SLAVE_ADMIN, GROUP_REPLICATION_ADMIN, BINLOG_ADMIN
ON *.* TO 'u1'@'localhost';
```

授予的动态权限将出现在`SHOW GRANTS`语句和`INFORMATION_SCHEMA` `USER_PRIVILEGES`表的输出中。

对于`GRANT`和`REVOKE`在全局级别，任何未被识别为静态的命名权限将与当前注册的动态权限集合进行检查，如果找到则授予。否则，将发生错误以指示未知的权限标识符。

对于`GRANT`和`REVOKE`在全局级别，`ALL [PRIVILEGES]`的含义包括所有静态全局权限，以及当前注册的所有动态权限：

+   `GRANT ALL` 在全局级别授予所有静态全局权限和当前注册的所有动态权限。在执行`GRANT`语句后注册的动态权限不会追溯地授予任何帐户。

+   `REVOKE ALL` 在全局级别撤销所有授予的静态全局权限和所有授予的动态权限。

`FLUSH PRIVILEGES`语句读取`global_grants`表中的动态权限分配，并注册在那里找到的任何未注册的权限。

有关 MySQL Server 和 MySQL 发行版中包含的组件提供的动态权限的描述，请参见 Section 8.2.2, “MySQL 提供的权限”。

#### 从 SUPER 迁移帐户到动态权限

在 MySQL 8.0 中，许多以前需要`SUPER`权限的操作也与更有限范围的动态权限相关联。（有关这些权限的描述，请参见第 8.2.2 节，“MySQL 提供的权限”。）每个这样的操作可以通过授予相关的动态权限而不是`SUPER`权限来允许给账户。这种变化通过使 DBA 避免授予`SUPER`权限并更紧密地调整用户权限以符合允许的操作来提高安全性。`SUPER`现已被弃用；预计它将在 MySQL 的未来版本中被移除。

当移除`SUPER`权限时，以前需要`SUPER`权限的操作将失败，除非被授予`SUPER`权限的账户迁移到适当的动态权限。使用以下说明来实现这一目标，以便在`SUPER`权限被移除之前账户已准备就绪：

1.  执行此查询以识别被授予`SUPER`权限的账户：

    ```sql
    SELECT GRANTEE FROM INFORMATION_SCHEMA.USER_PRIVILEGES
    WHERE PRIVILEGE_TYPE = 'SUPER';
    ```

1.  对于由上述查询确定的每个账户，确定其需要`SUPER`权限的操作。然后授予相应操作的动态权限，并撤销`SUPER`。

    例如，如果`'u1'@'localhost'`需要`SUPER`权限用于二进制日志清除和系统变量修改，以下语句将对账户进行所需更改：

    ```sql
    GRANT BINLOG_ADMIN, SYSTEM_VARIABLES_ADMIN ON *.* TO 'u1'@'localhost';
    REVOKE SUPER ON *.* FROM 'u1'@'localhost';
    ```

    在修改了所有适用账户之后，第一步中的`INFORMATION_SCHEMA`查询应产生一个空结果集。
