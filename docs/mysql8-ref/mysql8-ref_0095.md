# 2.9.3 测试服务器

> 原文：[`dev.mysql.com/doc/refman/8.0/en/testing-server.html`](https://dev.mysql.com/doc/refman/8.0/en/testing-server.html)

在数据目录初始化并启动服务器后，执行一些简单的测试以确保其正常工作。本节假定您当前的位置是 MySQL 安装目录，并且其中包含一个包含此处使用的 MySQL 程序的 `bin` 子目录。如果不是这样，请相应调整命令路径名称。

或者，将 `bin` 目录添加到您的 `PATH` 环境变量设置中。这样可以使您的 shell（命令解释器）正确找到 MySQL 程序，从而可以通过仅输入程序名称而不是路径名称来运行程序。参见 6.2.9 节“设置环境变量”。

使用 **mysqladmin** 来验证服务器是否正在运行。以下命令提供了简单的测试，以检查服务器是否正在运行并响应连接：

```sql
$> bin/mysqladmin version
$> bin/mysqladmin variables
```

如果无法连接到服务器，请指定 `-u root` 选项以连接为 `root`。如果您已经为 `root` 账户分配了密码，还需要在命令行中指定 `-p` 并在提示时输入密码。例如：

```sql
$> bin/mysqladmin -u root -p version
Enter password: *(enter root password here)*
```

**mysqladmin version** 的输出会根据您的平台和 MySQL 版本略有不同，但应该类似于这里显示的内容：

```sql
$> bin/mysqladmin version
mysqladmin  Ver 14.12 Distrib 8.0.36, for pc-linux-gnu on i686
...

Server version          8.0.36
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /var/lib/mysql/mysql.sock
Uptime:                 14 days 5 hours 5 min 21 sec

Threads: 1  Questions: 366  Slow queries: 0
Opens: 0  Flush tables: 1  Open tables: 19
Queries per second avg: 0.000
```

要查看您可以使用 **mysqladmin** 进行的其他���作，请使用 `--help` 选项调用它。

验证您可以关闭服务器（如果 `root` 账户已经有密码，请包括 `-p` 选项）：

```sql
$> bin/mysqladmin -u root shutdown
```

验证您可以再次启动服务器。可以通过使用 **mysqld_safe** 或直接调用 **mysqld** 来执行此操作。例如：

```sql
$> bin/mysqld_safe --user=mysql &
```

如果 **mysqld_safe** 失败，请参阅 2.9.2.1 节“启动 MySQL 服务器时出现问题的故障排除”。

运行一些简单的测试，以验证您可以从服务器检索信息。输出应类似于此处显示的内容。

使用 **mysqlshow** 查看存在哪些数据库：

```sql
$> bin/mysqlshow
+--------------------+
|     Databases      |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

安装的数据库列表可能有所不同，但始终包括至少 `mysql` 和 `information_schema`。

如果您指定了数据库名称，**mysqlshow** 将显示数据库中的表列表：

```sql
$> bin/mysqlshow mysql
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

使用**mysql**程序从`mysql`模式中的表中选择信息：

```sql
$> bin/mysql -e "SELECT User, Host, plugin FROM mysql.user" mysql
+------+-----------+-----------------------+
| User | Host      | plugin                |
+------+-----------+-----------------------+
| root | localhost | caching_sha2_password |
+------+-----------+-----------------------+
```

此时，您的服务器正在运行，您可以访问它。如果您尚未为初始帐户分配密码，请按照 Section 2.9.4, “Securing the Initial MySQL Account”中的说明加强安全性。

关于**mysql**，**mysqladmin**和**mysqlshow**的更多信息，请参见 Section 6.5.1, “mysql — The MySQL Command-Line Client”，Section 6.5.2, “mysqladmin — A MySQL Server Administration Program”和 Section 6.5.7, “mysqlshow — Display Database, Table, and Column Information”。
