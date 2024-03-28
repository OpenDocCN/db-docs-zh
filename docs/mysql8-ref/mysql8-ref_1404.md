> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-account.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-privilege-checks-account.html)

#### 19.3.3.1 用于复制 PRIVILEGE_CHECKS_USER 账户的特权

使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句指定的用户账户作为复制通道的`PRIVILEGE_CHECKS_USER`账户必须具有`REPLICATION_APPLIER`特权，否则复制应用程序线程将无法启动。如第 19.3.3 节“复制特权检查”所述，该账户需要进一步的特权，足以应用在复制通道上预期的所有预期事务。这些特权仅在执行相关事务时才会检查。

强烈建议对使用`PRIVILEGE_CHECKS_USER`账户保护的复制通道使用基于行的二进制日志记录（`binlog_format=ROW`）。对于基于语句的二进制日志记录，`PRIVILEGE_CHECKS_USER`账户可能需要一些管理员级别的特权才能成功执行事务。从 MySQL 8.0.19 开始，可以将`REQUIRE_ROW_FORMAT`设置应用于受保护的通道，限制通道执行需要这些特权的事件。

`REPLICATION_APPLIER`特权明确或隐含允许`PRIVILEGE_CHECKS_USER`账户执行复制线程需要执行的以下操作：

+   设置系统变量`gtid_next`，`original_commit_timestamp`，`original_server_version`，`immediate_server_version`，以及`pseudo_replica_mode`或`pseudo_slave_mode`的值，以在执行事务时应用适当的元数据和行为。

+   执行内部使用的`BINLOG`语句来应用**mysqlbinlog**输出，前提是该账户还具有这些语句中的表和操作的权限。

+   更新系统表 `mysql.gtid_executed`、`mysql.slave_relay_log_info`、`mysql.slave_worker_info` 和 `mysql.slave_master_info`，以更新复制元数据。（如果事件明确访问这些表以进行其他目的，必须在这些表上授予适当的权限。）

+   应用二进制日志 `Table_map_log_event`，提供表元数据但不进行任何数据库更改。

如果 `CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO` 语句的 `REQUIRE_TABLE_PRIMARY_KEY_CHECK` 选项设置为默认的 `STREAM`，`PRIVILEGE_CHECKS_USER` 账户需要具有足够权限设置受限制的会话变量，以便它可以在会话期间更改与源复制的设置匹配的 `sql_require_primary_key` 系统变量的值。`SESSION_VARIABLES_ADMIN` 权限赋予该账户此功能。此权限还允许账户应用使用使用 `--disable-log-bin` 选项创建的 **mysqlbinlog** 输出。如果将 `REQUIRE_TABLE_PRIMARY_KEY_CHECK` 设置为 `ON` 或 `OFF`，则副本始终在复制操作中使用该值作为 `sql_require_primary_key` 系统变量的值，因此不需要这些会话管理级别的权限。

如果使用表加密，`table_encryption_privilege_check` 系统变量设置为 `ON`，并且涉及任何事件的表空间的加密设置与应用服务器的默认加密设置（由 `default_table_encryption` 系统变量指定）不同，`PRIVILEGE_CHECKS_USER` 账户需要具有 `TABLE_ENCRYPTION_ADMIN` 权限才能覆盖默认加密设置。强烈建议不要授予此权限。相反，确保副本上的默认加密设置与其复制的表空间的加密状态匹配，并且复制组成员具有相同的默认加密设置，这样就不需要该权限。

为了从中继日志执行特定的复制事务，或根据需要执行来自 **mysqlbinlog** 输出的事务，`PRIVILEGE_CHECKS_USER` 账户必须具有以下权限：

+   对于以行格式记录的行插入（记录为`Write_rows_log_event`），需要在相关表上具有`INSERT`权限。

+   对于以行格式记录的行更新（记录为`Update_rows_log_event`），需要在相关表上具有`UPDATE`权限。

+   对于以行格式记录的行删除（记录为`Delete_rows_log_event`），需要在相关表上具有`DELETE`权限。

如果正在使用基于语句的二进制日志记录（不建议在`PRIVILEGE_CHECKS_USER`账户下使用），对于像`BEGIN`或`COMMIT`或以语句格式记录的 DML（记录为`Query_log_event`）的事务控制语句，`PRIVILEGE_CHECKS_USER`账户需要有执行事件中包含的语句所需的权限。

如果需要在复制通道上执行`LOAD DATA`操作，请使用基于行的二进制日志记录（`binlog_format=ROW`）。使用此日志记录格式时，不需要`FILE`权限来执行事件，因此不要给`PRIVILEGE_CHECKS_USER`账户此权限。强烈建议在使用`PRIVILEGE_CHECKS_USER`账户保护的复制通道上使用基于行的二进制日志记录。如果为通道设置了`REQUIRE_ROW_FORMAT`，则需要基于行的二进制日志记录。处理`LOAD DATA`事件创建的任何临时文件的`Format_description_log_event`会在不进行权限检查的情况下进行。有关更多信息，请参见 Section 19.5.1.19, “Replication and LOAD DATA”。

如果`init_replica`或`init_slave`系统变量被设置为指定在复制 SQL 线程启动时要执行的一个或多个 SQL 语句，则`PRIVILEGE_CHECKS_USER`账户必须具有执行这些语句所需的权限。

不建议将任何 ACL 权限授予`PRIVILEGE_CHECKS_USER`账户，包括`CREATE USER`，`CREATE ROLE`，`DROP ROLE`和`GRANT OPTION`，也不允许该账户更新`mysql.user`表。拥有这些权限，该账户可以用于在服务器上创建或修改用户账户。为避免源服务器上发出的 ACL 语句被复制到安全通道以供执行（在缺少这些权限的情况下会失败），您可以在所有 ACL 语句之前发出`SET sql_log_bin = 0`，在其后发出`SET sql_log_bin = 1`，以从源二进制日志中省略这些语句。或者，您可以在执行所有 ACL 语句之前设置一个专用的当前数据库，并使用复制过滤器（`--binlog-ignore-db`）来在副本中过滤掉这个数据库。
