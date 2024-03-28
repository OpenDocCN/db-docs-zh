# 7.1.9 使用系统变量

> 原文：[`dev.mysql.com/doc/refman/8.0/en/using-system-variables.html`](https://dev.mysql.com/doc/refman/8.0/en/using-system-variables.html)

7.1.9.1 系统变量权限

7.1.9.2 动态系统变量

7.1.9.3 持久化系统变量

7.1.9.4 非持久性和持久受限制的系统变量

7.1.9.5 结构化系统变量

MySQL 服务器维护许多系统变量来配置其操作。第 7.1.8 节，“服务器系统变量”，描述了这些变量的含义。每个系统变量都有一个默认值。系统变量可以在服务器启动时使用命令行选项或选项文件进行设置。大多数系统变量可以通过`SET`语句在服务器运行时动态更改，这使您可以修改服务器的操作而无需停止和重新启动它。您还可以在表达式中使用系统变量的值。

许多系统变量是内置的。系统变量也可以由服务器插件或组件安装：

+   当服务器插件安装时，由插件实现的系统变量会暴露出来，并且其名称以插件名称开头。例如，`audit_log`插件实现了一个名为`audit_log_policy`的系统变量。

+   当组件实现的系统变量在安装组件时暴露出来时，其名称以组件特定前缀开头。例如，`log_filter_dragnet`错误日志过滤组件实现了一个名为`log_error_filter_rules`的系统变量，其完整名称为`dragnet.log_error_filter_rules`。要引用此变量，请使用完整名称。

系统变量存在两个范围。全局变量影响服务器的整体操作。会话变量影响单个客户端连接的操作。给定的系统变量可以同时具有全局值和会话值。全局和会话系统变量之间的关系如下：

+   当服务器启动时，它会将每个全局变量初始化为其默认值。这些默认值可以通过在命令行或选项文件中指定的选项进行更改。（参见第 6.2.2 节，“指定程序选项”.）

+   服务器还为每个连接的客户端维护一组会话变量。客户端的会话变量在连接时使用相应全局变量的当前值进行初始化。例如，客户端的 SQL 模式由会话`sql_mode`值控制，该值在客户端连接到全局`sql_mode`值时进行初始化。

    对于某些系统变量，会话值不是从相应的全局值初始化的；如果是这样，变量描述中会指出。

可以通过在命令行或选项文件中使用选项在服务器启动时全局设置系统变量的值。在启动时，系统变量的语法与命令选项相同，因此在变量名称中，破折号和下划线可以互换使用。例如，`--general_log=ON`和`--general-log=ON`是等效的。

当您使用启动选项设置需要数字值的变量时，可以使用`K`、`M`或`G`（大写或小写）的后缀表示 1024、1024²或 1024³的倍数；即，单位为千字节、兆字节或千兆字节。从 MySQL 8.0.14 开始，后缀也可以是`T`、`P`和`E`，表示 1024⁴、1024⁵或 1024⁶的倍数。因此，以下命令启动服务器时，排序缓冲区大小为 256 千字节，最大数据包大小为 1 千兆字节：

```sql
mysqld --sort-buffer-size=256K --max-allowed-packet=1G
```

在选项文件中，这些变量设置如下：

```sql
[mysqld]
sort_buffer_size=256K
max_allowed_packet=1G
```

后缀字母的大小写不重要；`256K`和`256k`是等效的，`1G`和`1g`也是等效的。

要限制可以在运行时使用`SET`语句设置的系统变量的最大值，请在服务器启动时使用形式为`--maximum-*`var_name`*=*`value`*`的选项指定此最大值。例如，要防止`sort_buffer_size`的值在运行时增加到超过 32MB，使用选项`--maximum-sort-buffer-size=32M`。

许多系统变量是动态的，可以通过使用`SET`语句在运行时进行更改。有关列表，请参见 Section 7.1.9.2，“动态系统变量”。要使用`SET`更改系统变量，请按名称引用，可选地在前面加上修饰符。在运行时，系统变量名称必须使用下划线而不是破折号编写。以下示例简要说明了此语法：

+   设置全局系统变量：

    ```sql
    SET GLOBAL max_connections = 1000;
    SET @@GLOBAL.max_connections = 1000;
    ```

+   将全局系统变量持久化到`mysqld-auto.cnf`文件（并设置运行时值）：

    ```sql
    SET PERSIST max_connections = 1000;
    SET @@PERSIST.max_connections = 1000;
    ```

+   将全局系统变量持久化到`mysqld-auto.cnf`文件（而不设置运行时值）:

    ```sql
    SET PERSIST_ONLY back_log = 1000;
    SET @@PERSIST_ONLY.back_log = 1000;
    ```

+   设置会话系统变量：

    ```sql
    SET SESSION sql_mode = 'TRADITIONAL';
    SET @@SESSION.sql_mode = 'TRADITIONAL';
    SET @@sql_mode = 'TRADITIONAL';
    ```

有关`SET`语法的完整详细信息，请参阅 Section 15.7.6.1, “SET Syntax for Variable Assignment”。有关设置和持久化系统变量所需的特权要求的描述，请参阅 Section 7.1.9.1, “System Variable Privileges”

在服务器启动时设置变量时，可以使用值乘数后缀，但不能在运行时使用`SET`设置值。另一方面，使用`SET`可以使用表达式分配变量的值，在服务器启动时设置变量时不适用。例如，以下行中的第一行在服务器启动时是合法的，但第二行不是：

```sql
$> mysql --max_allowed_packet=16M
$> mysql --max_allowed_packet=16*1024*1024
```

相反，以下行中的第二行在运行时是合法的，但第一行不是：

```sql
mysql> SET GLOBAL max_allowed_packet=16M;
mysql> SET GLOBAL max_allowed_packet=16*1024*1024;
```

要显示系统变量名称和值，请使用`SHOW VARIABLES`语句：

```sql
mysql> SHOW VARIABLES;
+---------------------------------+-----------------------------------+
| Variable_name                   | Value                             |
+---------------------------------+-----------------------------------+
| auto_increment_increment        | 1                                 |
| auto_increment_offset           | 1                                 |
| automatic_sp_privileges         | ON                                |
| back_log                        | 151                               |
| basedir                         | /home/mysql/                      |
| binlog_cache_size               | 32768                             |
| bulk_insert_buffer_size         | 8388608                           |
| character_set_client            | utf8mb4                           |
| character_set_connection        | utf8mb4                           |
| character_set_database          | utf8mb4                           |
| character_set_filesystem        | binary                            |
| character_set_results           | utf8mb4                           |
| character_set_server            | utf8mb4                           |
| character_set_system            | utf8mb3                           |
| character_sets_dir              | /home/mysql/share/charsets/       |
| check_proxy_users               | OFF                               |
| collation_connection            | utf8mb4_0900_ai_ci                |
| collation_database              | utf8mb4_0900_ai_ci                |
| collation_server                | utf8mb4_0900_ai_ci                |
...
| innodb_autoextend_increment     | 8                                 |
| innodb_buffer_pool_size         | 8388608                           |
| innodb_commit_concurrency       | 0                                 |
| innodb_concurrency_tickets      | 500                               |
| innodb_data_file_path           | ibdata1:10M:autoextend            |
| innodb_data_home_dir            |                                   |
...
| version                         | 8.0.31                            |
| version_comment                 | Source distribution               |
| version_compile_machine         | x86_64                            |
| version_compile_os              | Linux                             |
| version_compile_zlib            | 1.2.12                            |
| wait_timeout                    | 28800                             |
+---------------------------------+-----------------------------------+
```

使用`LIKE`子句，语句仅显示与模式匹配的变量。要获取特定变量名，请使用如下所示的`LIKE`子句：

```sql
SHOW VARIABLES LIKE 'max_join_size';
SHOW SESSION VARIABLES LIKE 'max_join_size';
```

要获取与模式匹配的变量列表，请在`LIKE`子句中使用`%`通配符字符：

```sql
SHOW VARIABLES LIKE '%size%';
SHOW GLOBAL VARIABLES LIKE '%size%';
```

通配符字符可以在要匹配的模式中的任何位置使用。严格来说，因为`_`是匹配任意单个字符的通配符，您应该将其转义为`\_`以确实匹配它。实际上，这很少是必要的。

对于`SHOW VARIABLES`，如果既不指定`GLOBAL`也不指定`SESSION`，MySQL 将返回`SESSION`值。

在设置仅限`GLOBAL`变量时需要`GLOBAL`关键字但在检索时不需要的原因是为了防止未来出现问题：

+   如果删除具有与`GLOBAL`变量相同名称的`SESSION`变量，则具有足够权限修改全局变量的客户端可能会意外更改`GLOBAL`变量，而不仅仅是自己会话的`SESSION`变量。

+   如果添加具有与`GLOBAL`变量相同名称的`SESSION`变量，则打算更改`GLOBAL`变量的客户端可能只会发现自己的`SESSION`变量被更改。
