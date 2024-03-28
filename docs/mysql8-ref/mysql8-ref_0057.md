# 2.3.6 Windows 后安装程序

> 原文：[`dev.mysql.com/doc/refman/8.0/en/windows-postinstallation.html`](https://dev.mysql.com/doc/refman/8.0/en/windows-postinstallation.html)

存在 GUI 工具可以执行本节描述的大部分任务，包括：

+   MySQL Installer：用于安装和升级 MySQL 产品。

+   MySQL Workbench：管理 MySQL 服务器并编辑 SQL 语句。

如有必要，初始化数据目录并创建 MySQL 授权表。由 MySQL Installer 执行的 Windows 安装操作会自动初始化数据目录。对于从 ZIP 存档包安装，请按照第 2.9.1 节，“初始化数据目录”中描述的步骤初始化数据目录。

关于密码，如果您使用 MySQL Installer 安装了 MySQL，则可能已经为初始的`root`账户分配了密码。（参见第 2.3.3 节，“Windows 的 MySQL Installer”。）否则，请使用第 2.9.4 节，“保护初始 MySQL 账户”中给出的密码分配过程。

在分配密码之前，您可能希望尝试运行一些客户端程序，以确保您可以连接到服务器并且它正常运行。确保服务器正在运行（参见第 2.3.4.5 节，“首次启动服务器”）。您还可以设置一个 MySQL 服务，该服务在 Windows 启动时自动运行（参见第 2.3.4.8 节，“将 MySQL 设置为 Windows 服务”）。

这些说明假定您当前的位置是 MySQL 安装目录，并且该目录包含一个`bin`子目录，其中包含此处使用的 MySQL 程序。如果不是这样，请相应调整命令路径名称。

如果您使用 MySQL Installer 安装了 MySQL（参见第 2.3.3 节，“Windows 的 MySQL Installer”），默认安装目录是`C:\Program Files\MySQL\MySQL Server 8.0`：

```sql
C:\> cd "C:\Program Files\MySQL\MySQL Server 8.0"
```

从 ZIP 存档包安装的常见安装位置是`C:\mysql`：

```sql
C:\> cd C:\mysql
```

或者，将`bin`目录添加到您的`PATH`环境变量设置中。这样可以使您的命令解释器正确找到 MySQL 程序，因此您可以通过仅输入程序名称而不是路径名称来运行程序。请参见第 2.3.4.7 节，“自定义 MySQL 工具的 PATH”。

运行服务器后，发出以下命令以验证您可以从服务器检索信息。输出应该类似于这里显示的内容。

使用**mysqlshow**查看存在哪些数据库：

```sql
C:\> bin\mysqlshow
+--------------------+
|     Databases      |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

安装的数据库列表可能有所不同，但始终至少包括`mysql`和`information_schema`。

如果不存在正确的 MySQL 帐户，则前面的命令（以及其他 MySQL 程序的命令，如**mysql**）可能无法正常工作。例如，程序可能会出现错误，或者您可能无法查看所有数据库。如果使用 MySQL Installer 安装 MySQL，则`root`用户会自动创建，并使用您提供的密码。在这种情况下，您应该使用`-u root`和`-p`选项。（如果您已经保护了初始的 MySQL 帐户，则必须使用这些选项。）使用`-p`，客户端程序会提示输入`root`密码。例如：

```sql
C:\> bin\mysqlshow -u root -p
Enter password: *(enter root password here)* +--------------------+
|     Databases      |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

如果指定数据库名称，**mysqlshow**会显示数据库中的表列表：

```sql
C:\> bin\mysqlshow mysql
Database: mysql
+---------------------------+
|          Tables           |
+---------------------------+
| columns_priv              |
| component                 |
| db                        |
| default_roles             |
| engine_cost               |
| func                      |
| general_log               |
| global_grants             |
| gtid_executed             |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| innodb_index_stats        |
| innodb_table_stats        |
| ndb_binlog_index          |
| password_history          |
| plugin                    |
| procs_priv                |
| proxies_priv              |
| role_edges                |
| server_cost               |
| servers                   |
| slave_master_info         |
| slave_relay_log_info      |
| slave_worker_info         |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
```

使用**mysql**程序从`mysql`数据库中的表中选择信息：

```sql
C:\> bin\mysql -e "SELECT User, Host, plugin FROM mysql.user" mysql
+------+-----------+-----------------------+
| User | Host      | plugin                |
+------+-----------+-----------------------+
| root | localhost | caching_sha2_password |
+------+-----------+-----------------------+
```

有关**mysql**和**mysqlshow**的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”，以及 Section 6.5.7, “mysqlshow — Display Database, Table, and Column Information”。
