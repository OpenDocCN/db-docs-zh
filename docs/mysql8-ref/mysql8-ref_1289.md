# 17.20.9 解决 InnoDB memcached 插件问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-troubleshoot.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-troubleshoot.html)

本节描述了在使用`InnoDB` **memcached**插件时可能遇到的问题。

+   如果在 MySQL 错误日志中遇到以下错误，则服务器可能无法启动：

    设置打开文件数的限制失败。尝试以 root 用户身份运行或请求较小的 maxconns 值。

    错误消息来自**memcached**守护程序。一个解决方案是提高操作系统打开文件数的限制。检查和增加打开文件限制的命令因操作系统而异。以下示例显示了 Linux 和 macOS 的命令：

    ```sql
    # Linux
    $> ulimit -n
    1024
    $> ulimit -n 4096
    $> ulimit -n
    4096

    # macOS
    $> ulimit -n
    256
    $> ulimit -n 4096
    $> ulimit -n
    4096
    ```

    另一种解决方案是减少允许**memcached**守护程序的并发连接数。为此，在 MySQL 配置文件中的`daemon_memcached_option`配置参数中编码`-c` **memcached**选项。`-c`选项的默认值为 1024。

    ```sql
    [mysqld]
    ...
    loose-daemon_memcached_option='-c 64'
    ```

+   要解决**memcached**守护程序无法存储或检索`InnoDB`表数据的问题，请在 MySQL 配置文件的`daemon_memcached_option`配置参数中编码`-vvv` **memcached**选项。检查 MySQL 错误日志以获取与**memcached**操作相关的调试输出。

    ```sql
    [mysqld]
    ...
    loose-daemon_memcached_option='-vvv'
    ```

+   如果指定用于保存**memcached**值的列的数据类型错误，例如数字类型而不是字符串类型，则尝试存储键值对将失败，而不会显示特定的错误代码或消息。

+   如果`daemon_memcached`插件导致 MySQL 服务器启动问题，您可以在故障排除时通过在 MySQL 配置文件的`[mysqld]`组下添加以下行来临时禁用`daemon_memcached`插件：

    ```sql
    daemon_memcached=OFF
    ```

    例如，如果在运行`innodb_memcached_config.sql`配置脚本设置必要的数据库和表之前运行`INSTALL PLUGIN`语句，则服务器可能会意外退出并无法启动。如果在`innodb_memcache.containers`表中错误配置条目，服务器也可能无法启动。

    要卸载 MySQL 实例的**memcached**插件，请发出以下语句：

    ```sql
    mysql> UNINSTALL PLUGIN daemon_memcached;
    ```

+   如果在同一台机器上运行多个 MySQL 实例，并且每个实例都启用了`daemon_memcached`插件，请使用`daemon_memcached_option`配置参数为每个`daemon_memcached`插件指定一个独特的**memcached**端口。

+   如果一个 SQL 语句无法找到`InnoDB`表或在表中找不到数据，但**memcached** API 调用检索到了预期的数据，那么可能是在`innodb_memcache.containers`表中缺少了`InnoDB`表的条目，或者您可能没有通过使用`@@*`table_id`*`标记发出`get`或`set`请求来切换到正确的`InnoDB`表。如果您在之后没有重新启动 MySQL 服务器的情况下更改了`innodb_memcache.containers`表中的现有条目，也可能会出现这个问题。自由形式的存储机制足够灵活，即使守护程序正在使用将值存储在单列中的`test.demo_test`表，您对存储或检索多列值（如`col1|col2|col3`）的请求可能仍然有效。

+   在为`daemon_memcached`插件定义自己的`InnoDB`表时，如果表中的列被定义为`NOT NULL`，请确保在将表的记录插入`innodb_memcache.containers`表时为`NOT NULL`列提供值。如果`innodb_memcache.containers`记录的`INSERT`语句包含的分隔值少于映射列的数量，未填充的列将被设置为`NULL`。尝试将`NULL`值插入`NOT NULL`列会导致`INSERT`失败，这可能只有在重新初始化`daemon_memcached`插件以应用对`innodb_memcache.containers`表的更改后才会显现。

+   如果`innodb_memcached.containers`表的`cas_column`和`expire_time_column`字段设置为`NULL`，则在尝试加载**memcached**插件时会返回以下错误：

    ```sql
    InnoDB_Memcached: column 6 in the entry for config table 'containers' in
    database 'innodb_memcache' has an invalid NULL value.
    ```

    **memcached**插件拒绝在`cas_column`和`expire_time_column`列中使用`NULL`。当这些列未使用时，将这些列的值设置为`0`。

+   随着**memcached**键和值的长度增加，您可能会遇到大小和长度限制。

    +   当键超过 250 字节时，**memcached**操作会返回错误。这是**memcached**中当前的固定限制。

    +   如果值的大小超过 768 字节、3072 字节或半个`innodb_page_size`值，可能会遇到`InnoDB`表限制。这些限制主要适用于如果您打算在值列上创建索引以使用 SQL 运行生成报表的查询时。有关详细信息，请参见第 17.22 节，“InnoDB 限制”。

    +   键-值组合的最大大小为 1 MB。

+   如果在不同版本的 MySQL 服务器之间共享配置文件，并且使用`daemon_memcached`插件的最新配置选项可能会导致在旧的 MySQL 版本上启动错误。为避免兼容性问题，请在选项名称前使用`loose`前缀。例如，使用`loose-daemon_memcached_option='-c 64'`而不是`daemon_memcached_option='-c 64'`。

+   没有限制或检查来验证字符集设置。**memcached**以字节形式存储和检索键和值，因此不受字符集的影响。但是，您必须确保**memcached**客户端和 MySQL 表使用相同的字符集。

+   **memcached**连接被阻止访问包含索引虚拟列的表。访问索引虚拟列需要回调到服务器，但**memcached**连接无法访问服务器代码。
