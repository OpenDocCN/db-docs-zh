# 17.20.3 设置 InnoDB memcached 插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-setup.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-setup.html)

本节描述了如何在 MySQL 服务器上设置`daemon_memcached`插件。由于**memcached**守护程序与 MySQL 服务器紧密集成，以避免网络流量并最小化延迟，因此你需要在使用此功能的每个 MySQL 实例上执行此过程。

注意

在设置`daemon_memcached`插件之前，请参考 Section 17.20.5, “InnoDB memcached 插件的安全注意事项” 以了解所需的安全程序，以防止未经授权的访问。

#### 先决条件

+   `daemon_memcached`插件仅支持 Linux、Solaris 和 macOS 平台。其他操作系统不受支持。

+   在从源代码构建 MySQL 时，你必须使用 `-DWITH_INNODB_MEMCACHED=ON` 构建选项。此构建选项在 MySQL 插件目录（`plugin_dir`）中生成两个共享库，这些库是运行`daemon_memcached`插件所需的：

    +   `libmemcached.so`：**memcached** 守护程序插件到 MySQL。

    +   `innodb_engine.so`：**memcached** 的`InnoDB` API 插件。

+   必须安装`libevent`。

    +   如果你没有从源代码构建 MySQL，那么`libevent`库不会包含在你的安装中。使用你的操作系统的安装方法来安装`libevent` 1.4.12 或更高版本。例如，根据操作系统的不同，你可能会使用`apt-get`、`yum`或`port install`。例如，在 Ubuntu Linux 上，使用：

        ```sql
        sudo apt-get install libevent-dev
        ```

    +   如果你从源代码发布中安装了 MySQL，`libevent` 1.4.12 已经捆绑在包中，并位于 MySQL 源代码目录的顶层。如果你使用捆绑版本的`libevent`，则无需采取任何操作。如果你想使用本地系统版本的`libevent`，你必须使用 `-DWITH_LIBEVENT` 构建选项设置为`system`或`yes`来构建 MySQL。

#### 安装和配置 InnoDB memcached 插件

1.  通过运行位于`*`MYSQL_HOME`*/share`中的`innodb_memcached_config.sql`配置脚本，配置`daemon_memcached`插件以与`InnoDB`表交互。此脚本安装了包含三个必需表（`cache_policies`、`config_options`和`containers`）的`innodb_memcache`数据库。它还在`test`数据库中安装了`demo_test`示例表。

    ```sql
    mysql> source *MYSQL_HOME*/share/innodb_memcached_config.sql
    ```

    运行`innodb_memcached_config.sql`脚本是一次性操作。如果以后卸载并重新安装`daemon_memcached`插件，表将保留在原位。

    ```sql
    mysql> USE innodb_memcache;
    mysql> SHOW TABLES;
    +---------------------------+
    | Tables_in_innodb_memcache |
    +---------------------------+
    | cache_policies            |
    | config_options            |
    | containers                |
    +---------------------------+

    mysql> USE test;
    mysql> SHOW TABLES;
    +----------------+
    | Tables_in_test |
    +----------------+
    | demo_test      |
    +----------------+
    ```

    在这些表中，`innodb_memcache.containers` 表最重要。`containers` 表中的条目提供到 `InnoDB` 表列的映射。每个使用 `daemon_memcached` 插件的 `InnoDB` 表都需要在 `containers` 表中有一个条目。

    `innodb_memcached_config.sql` 脚本在 `containers` 表中插入一个条目，为 `demo_test` 表提供映射。它还在 `demo_test` 表中插入一行数据。这些数据允许你在设置完成后立即验证安装。

    ```sql
    mysql> SELECT * FROM innodb_memcache.containers\G
    *************************** 1\. row ***************************
                      name: aaa
                 db_schema: test
                  db_table: demo_test
               key_columns: c1
             value_columns: c2
                     flags: c3
                cas_column: c4
        expire_time_column: c5
    unique_idx_name_on_key: PRIMARY 
    mysql> SELECT * FROM test.demo_test;
    +----+------------------+------+------+------+
    | c1 | c2               | c3   | c4   | c5   |
    +----+------------------+------+------+------+
    | AA | HELLO, HELLO     |    8 |    0 |    0 |
    +----+------------------+------+------+------+
    ```

    有关 `innodb_memcache` 表和 `demo_test` 示例表的更多信息，请参见 第 17.20.8 节，“InnoDB memcached 插件内部”。

1.  通过运行 `INSTALL PLUGIN` 语句激活 `daemon_memcached` 插件：

    ```sql
    mysql> INSTALL PLUGIN daemon_memcached soname "libmemcached.so";
    ```

    一旦插件安装完成，每次 MySQL 服务器重新启动时都会自动激活。

#### 验证 InnoDB 和 memcached 设置

要验证 `daemon_memcached` 插件设置，使用 **telnet** 会话发出 **memcached** 命令。默认情况下，**memcached** 守护程序监听端口 11211。

1.  从 `test.demo_test` 表中检索数据。`demo_test` 表中的单行数据具有键值 `AA`。

    ```sql
    telnet localhost 11211
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    get AA
    VALUE AA 8 12
    HELLO, HELLO
    END
    ```

1.  使用 `set` 命令插入数据。

    ```sql
    set BB 10 0 16
    GOODBYE, GOODBYE
    STORED
    ```

    其中：

    +   `set` 是存储值的命令

    +   `BB` 是键

    +   `10` 是操作的标志；**memcached** 忽略但客户端可能用于指示任何类型的信息；如果未使用，请指定 `0`

    +   `0` 是过期时间（TTL）；如果未使用，请指定 `0`

    +   提供的值块的长度为 `16` 字节

    +   `GOODBYE, GOODBYE` 是存储的值

1.  通过连接到 MySQL 服务器并查询 `test.demo_test` 表来验证插入的数据是否存储在 MySQL 中。

    ```sql
    mysql> SELECT * FROM test.demo_test;
    +----+------------------+------+------+------+
    | c1 | c2               | c3   | c4   | c5   |
    +----+------------------+------+------+------+
    | AA | HELLO, HELLO     |    8 |    0 |    0 |
    | BB | GOODBYE, GOODBYE |   10 |    1 |    0 |
    +----+------------------+------+------+------+
    ```

1.  返回 telnet 会话并使用键 `BB` 检索之前插入的数据。

    ```sql
    get BB
    VALUE BB 10 16
    GOODBYE, GOODBYE
    END
    quit
    ```

如果关闭 MySQL 服务器，也会关闭集成的 **memcached** 服务器，进一步尝试访问 **memcached** 数据会因连接错误而失败。通常情况下，此时 **memcached** 数据也会消失，当 **memcached** 重新启动时，你需要应用逻辑来重新加载数据。然而，`InnoDB` **memcached** 插件会自动化这个过程。

当你重新启动 MySQL 时，`get` 操作会再次返回你在之前 **memcached** 会话中存储的键值对。当请求一个键并且相关值不在内存缓存中时，该值会自动从 MySQL `test.demo_test` 表中查询。

#### 创建新表和列映射

此示例展示了如何使用 `daemon_memcached` 插件设置自己的 `InnoDB` 表。

1.  创建一个`InnoDB`表。表必须具有具有唯一索引的键列。城市表的键列是`city_id`，定义为主键。表还必须包括用于`flags`，`cas`和`expiry`值的列。可能有一个或多个值列。`city`表有三个值列（`name`，`state`，`country`）。

    注意

    列名没有特殊要求，只要向`innodb_memcache.containers`表添加有效映射即可。

    ```sql
    mysql> CREATE TABLE city (
           city_id VARCHAR(32),
           name VARCHAR(1024),
           state VARCHAR(1024),
           country VARCHAR(1024),
           flags INT,
           cas BIGINT UNSIGNED, 
           expiry INT,
           primary key(city_id)
           ) ENGINE=InnoDB;
    ```

1.  向`innodb_memcache.containers`表添加一个条目，以便`daemon_memcached`插件知道如何访问`InnoDB`表。该条目必须满足`innodb_memcache.containers`表的定义。有关每个字段的描述，请参见第 17.20.8 节，“InnoDB memcached 插件内部”。

    ```sql
    mysql> DESCRIBE innodb_memcache.containers;
    +------------------------+--------------+------+-----+---------+-------+
    | Field                  | Type         | Null | Key | Default | Extra |
    +------------------------+--------------+------+-----+---------+-------+
    | name                   | varchar(50)  | NO   | PRI | NULL    |       |
    | db_schema              | varchar(250) | NO   |     | NULL    |       |
    | db_table               | varchar(250) | NO   |     | NULL    |       |
    | key_columns            | varchar(250) | NO   |     | NULL    |       |
    | value_columns          | varchar(250) | YES  |     | NULL    |       |
    | flags                  | varchar(250) | NO   |     | 0       |       |
    | cas_column             | varchar(250) | YES  |     | NULL    |       |
    | expire_time_column     | varchar(250) | YES  |     | NULL    |       |
    | unique_idx_name_on_key | varchar(250) | NO   |     | NULL    |       |
    +------------------------+--------------+------+-----+---------+-------+
    ```

    城市表的`innodb_memcache.containers`表条目定义为：

    ```sql
    mysql> INSERT INTO `innodb_memcache`.`containers` (
           `name`, `db_schema`, `db_table`, `key_columns`, `value_columns`,
           `flags`, `cas_column`, `expire_time_column`, `unique_idx_name_on_key`)
           VALUES ('default', 'test', 'city', 'city_id', 'name|state|country', 
           'flags','cas','expiry','PRIMARY');
    ```

    +   为`containers.name`列指定`default`，以将`city`表配置为与`daemon_memcached`插件一起使用的默认`InnoDB`表。

    +   多个`InnoDB`表列（`name`，`state`，`country`）使用“|”分隔符映射到`containers.value_columns`。

    +   `innodb_memcache.containers`表的`flags`，`cas_column`和`expire_time_column`字段在使用`daemon_memcached`插件的应用程序中通常不重要。但是，每个字段都需要指定一个指定的`InnoDB`表列。在插入数据时，如果未使用这些列，请为这些列指定`0`。

1.  更新`innodb_memcache.containers`表后，重新启动`daemon_memcache`插件以应用更改。

    ```sql
    mysql> UNINSTALL PLUGIN daemon_memcached;

    mysql> INSTALL PLUGIN daemon_memcached soname "libmemcached.so";
    ```

1.  使用 telnet，使用**memcached**的`set`命令向`city`表插入数据。

    ```sql
    telnet localhost 11211
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    set B 0 0 22
    BANGALORE|BANGALORE|IN
    STORED
    ```

1.  使用 MySQL，查询`test.city`表以验证您插入的数据是否已存储。

    ```sql
    mysql> SELECT * FROM test.city;
    +---------+-----------+-----------+---------+-------+------+--------+
    | city_id | name      | state     | country | flags | cas  | expiry |
    +---------+-----------+-----------+---------+-------+------+--------+
    | B       | BANGALORE | BANGALORE | IN      |     0 |    3 |      0 |
    +---------+-----------+-----------+---------+-------+------+--------+
    ```

1.  使用 MySQL，向`test.city`表插入额外数据。

    ```sql
    mysql> INSERT INTO city VALUES ('C','CHENNAI','TAMIL NADU','IN', 0, 0 ,0);
    mysql> INSERT INTO city VALUES ('D','DELHI','DELHI','IN', 0, 0, 0);
    mysql> INSERT INTO city VALUES ('H','HYDERABAD','TELANGANA','IN', 0, 0, 0);
    mysql> INSERT INTO city VALUES ('M','MUMBAI','MAHARASHTRA','IN', 0, 0, 0);
    ```

    注意

    如果未使用，建议为`flags`，`cas_column`和`expire_time_column`字段指定值`0`。

1.  使用 telnet，发出**memcached**的`get`命令以检索使用 MySQL 插入的数据。

    ```sql
    get H
    VALUE H 0 22
    HYDERABAD|TELANGANA|IN
    END
    ```

#### 配置 InnoDB memcached 插件

传统的`memcached`配置选项可以在 MySQL 配置文件或者**mysqld**启动字符串中指定，编码在`daemon_memcached_option`配置参数的参数中。`memcached`配置选项在插件加载时生效，这发生在每次启动 MySQL 服务器时。

例如，要使**memcached**监听端口 11222 而不是默认端口 11211，请将`-p11222`指定为`daemon_memcached_option`配置选项的参数：

```sql
mysqld .... --daemon_memcached_option="-p11222"
```

其他**memcached**选项可以编码在`daemon_memcached_option`字符串中。例如，您可以指定选项来减少最大同时连接数，更改键值对的最大内存大小，或者启用错误日志的调试消息等。

也有一些特定于`daemon_memcached`插件的配置选项。这些包括：

+   `daemon_memcached_engine_lib_name`：指定实现`InnoDB` **memcached**插件的共享库。默认设置为`innodb_engine.so`。

+   `daemon_memcached_engine_lib_path`：包含实现`InnoDB` **memcached**插件的共享库的目录路径。默认值为 NULL，表示插件目录。

+   `daemon_memcached_r_batch_size`：定义读操作（`get`）的批量提交大小。它指定在进行多少次**memcached**读取操作后发生提交。`daemon_memcached_r_batch_size`默认设置为 1，以便每个`get`请求访问`InnoDB`表中最近提交的数据，无论数据是通过**memcached**还是通过 SQL 更新的。当值大于 1 时，每次`get`调用都会增加读操作计数器。`flush_all`调用会重置读取和写入计数器。

+   `daemon_memcached_w_batch_size`：定义写操作（`set`、`replace`、`append`、`prepend`、`incr`、`decr`等）的批量提交大小。`daemon_memcached_w_batch_size`默认设置为 1，以防止在停机情况下丢失未提交的数据，并且使底层表上的 SQL 查询访问最新数据。当值大于 1 时，每次`add`、`set`、`incr`、`decr`和`delete`调用都会增加写操作计数器。`flush_all`调用会重置读取和写入计数器。

默认情况下，您不需要修改`daemon_memcached_engine_lib_name`或`daemon_memcached_engine_lib_path`。例如，如果您想要使用不同的存储引擎来**memcached**（如 NDB **memcached**引擎），则可以配置这些选项。

`daemon_memcached`插件配置参数可以在 MySQL 配置文件中或在**mysqld**启动字符串中指定。它们在加载`daemon_memcached`插件时生效。

在对`daemon_memcached`插件配置进行更改时，重新加载插件以应用更改。要这样做，请发出以下语句：

```sql
mysql> UNINSTALL PLUGIN daemon_memcached;

mysql> INSTALL PLUGIN daemon_memcached soname "libmemcached.so";
```

插件重新启动时会保留配置设置、所需表格和数据。

有关启用和禁用插件的更多信息，请参见第 7.6.1 节，“安装和卸载插件”。
