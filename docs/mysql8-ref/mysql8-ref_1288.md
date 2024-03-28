# 17.20.8 InnoDB memcached 插件内部

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-internals.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-internals.html)

#### InnoDB memcached 插件的 InnoDB API

`InnoDB` **memcached**引擎通过`InnoDB` API 访问`InnoDB`，其中大部分直接采用自嵌入式`InnoDB`。 `InnoDB` API 函数作为回调函数传递给`InnoDB` **memcached**引擎。 `InnoDB` API 函数直接访问`InnoDB`表，大多数是 DML 操作，除了`TRUNCATE TABLE`。

**memcached**命令通过`InnoDB` **memcached** API 实现。以下表格概述了如何将**memcached**命令映射到 DML 或 DDL 操作。

**表 17.27 memcached 命令及相关的 DML 或 DDL 操作**

| memcached 命令 | DML 或 DDL 操作 |
| --- | --- |
| `get` | 读取/获取命令 |
| `set` | 一次搜索，然后是`INSERT`或`UPDATE`（取决于键是否存在） |
| `add` | 一次搜索，然后是`INSERT`或`UPDATE` |
| `replace` | 一次搜索，然后是`UPDATE` |
| `append` | 一次搜索，然后是`UPDATE`（在`UPDATE`之前将数据追加到结果中） |
| `prepend` | 一次搜索，然后是`UPDATE`（在`UPDATE`之前将数据前置到结果中） |
| `incr` | 一次搜索，然后是`UPDATE` |
| `decr` | 一次搜索，然后是`UPDATE` |
| `delete` | 一次搜索，然后是`DELETE` |
| `flush_all` | `TRUNCATE TABLE`（DDL） |
| memcached 命令 | DML 或 DDL 操作 |

#### InnoDB memcached 插件配置表

本节描述了`daemon_memcached`插件使用的配置表。 `cache_policies`表，`config_options`表和`containers`表由`innodb_memcached_config.sql`配置脚本在`innodb_memcache`数据库中创建。

```sql
mysql> USE innodb_memcache;
Database changed
mysql> SHOW TABLES;
+---------------------------+
| Tables_in_innodb_memcache |
+---------------------------+
| cache_policies            |
| config_options            |
| containers                |
+---------------------------+
```

#### cache_policies 表

`cache_policies`表定义了`InnoDB` `memcached`安装的缓存策略。您可以在单个缓存策略中为`get`，`set`，`delete`和`flush`操作指定单独的策略。所有操作的默认设置为`innodb_only`。

+   `innodb_only`: 使用`InnoDB`作为数据存储。

+   `cache_only`: 使用**memcached**引擎作为数据存储。

+   `caching`: 同时使用`InnoDB`和**memcached**引擎作为数据存储。在这种情况下，如果**memcached**在内存中找不到键，则会在`InnoDB`表中搜索该值。

+   `disable`: 禁用缓存。

**表 17.28 cache_policies 列**

| 列 | 描述 |
| --- | --- |
| `policy_name` | 缓存策略的名称。默认缓存策略名称为`cache_policy`。 |
| `get_policy` | get 操作的缓存策略。有效值为`innodb_only`，`cache_only`，`caching`或`disabled`。默认设置为`innodb_only`。 |
| `set_policy` | set 操作的缓存策略。有效值为`innodb_only`，`cache_only`，`caching`或`disabled`。默认设置为`innodb_only`。 |
| `delete_policy` | 删除操作的缓存策略。有效值为`innodb_only`、`cache_only`、`caching`或`disabled`。默认设置为`innodb_only`。 |
| `flush_policy` | 刷新操作的缓存策略。有效值为`innodb_only`、`cache_only`、`caching`或`disabled`。默认设置为`innodb_only`。 |

#### config_options 表

`config_options`表存储可以使用 SQL 在运行时更改的与**memcached**相关的设置。支持的配置选项是`separator`和`table_map_delimiter`。

**表 17.29 config_options 列**

| 列 | 描述 |
| --- | --- |

| `Name` | **memcached**相关配置选项的名称。`config_options`表支持以下配置选项：

+   `separator`：用于将长字符串的值分隔为单独的值的分隔符，当定义了多个`value_columns`时。默认情况下，分隔符是`&#124;`字符。例如，如果您将`col1, col2`定义为值列，并将`&#124;`定义为分隔符，则可以发出以下**memcached**命令将值分别插入到`col1`和`col2`中：

    ```sql
    set keyx 10 0 19
    valuecolx&#124;valuecoly
    ```

    `valuecol1x`存储在`col1`中，`valuecoly`存储在`col2`中。

+   `table_map_delimiter`：在使用`@@`符号访问特定表中的键时，用于分隔模式名称和表名称的字符。例如，`@@t1.some_key`和`@@t2.some_key`具有相同的键值，但存储在不同的表中。

|

| `Value` | 分配给与**memcached**相关的配置选项的值。 |
| --- | --- |

#### containers 表

`containers`表是三个配置表中最重要的一个。用于存储**memcached**值的每个`InnoDB`表必须在`containers`表中有一个条目。该条目提供了`InnoDB`表列和容器表列之间的映射，这对于`memcached`与`InnoDB`表一起工作是必需的。

`containers`表包含`test.demo_test`表的默认条目，该表是由`innodb_memcached_config.sql`配置脚本创建的。要使用`daemon_memcached`插件与自己的`InnoDB`表，必须在`containers`表中创建一个条目。

**表 17.30 containers 列**

| 列 | 描述 |
| --- | --- |
| `name` | 分配给容器的名称。如果没有使用`@@`符号按名称请求`InnoDB`表，则`daemon_memcached`插件将使用具有`containers.name`值为`default`的`InnoDB`表。如果没有这样的条目，则按`name`（升序）字母顺序排列的`containers`表中的第一个条目确定默认的`InnoDB`表。 |
| `db_schema` | 包含`InnoDB`表的数据库的名称。这是一个必填值。 |
| `db_table` | 存储**memcached**值的`InnoDB`表的名称。这是一个必填值。 |
| `key_columns` | 包含**memcached**操作查找键值的`InnoDB`表中的列。这是一个必填值。 |
| `value_columns` | 存储`memcached`数据的`InnoDB`表列（一个或多个）。可以使用`innodb_memcached.config_options`表中指定的分隔符字符指定多个列。默认情况下，分隔符是一个竖线字符（“ | ”）。要指定多个列，请使用定义的分隔符字符分隔它们。例如：`col1 | col2 | col3`。这是一个必需的值。 |
| `flags` | 用作**memcached**标志（与主值一起存储和检索的用户定义的数值）的`InnoDB`表列。如果**memcached**值映射到多个列，则标志值可以用作某些操作（如`incr`，`prepend`）的列指定符，以便在指定列上执行操作。例如，如果你已将`value_columns`映射到三个`InnoDB`表列，并且只想要增量操作在一个列上执行，请使用`flags`列指定该列。如果你不使用`flags`列，请设置值为`0`以指示未使用。 |
| `cas_column` | 存储比较和交换（cas）值的`InnoDB`表列。`cas_column`值与**memcached**如何将请求哈希到不同服务器并在内存中缓存数据有关。由于`InnoDB` **memcached**插件与单个**memcached**守护程序紧密集成，并且内存缓存机制由 MySQL 和 InnoDB 缓冲池处理，因此很少需要此列。如果你不使用此列，请设置值为`0`以指示未使用。 |
| `expire_time_column` | 存储过期值的`InnoDB`表列。`expire_time_column`值与**memcached**如何将请求哈希到不同服务器并在内存中缓存数据有关。由于`InnoDB` **memcached**插件与单个**memcached**守护程序紧密集成，并且内存缓存机制由 MySQL 和 InnoDB 缓冲池处理，因此很少需要此列。如果你不使用此列，请设置值为`0`以指示未使用。最大过期时间定义为`INT_MAX32`或 2147483647 秒（约 68 年）。 |
| `unique_idx_name_on_key` | 索引在关键列上的名称。它必须是一个唯一索引。它可以是主键或次要索引。最好使用`InnoDB`表的主键。使用主键可以避免在使用次要索引时进行查找。你不能为**memcached**查找创建一个覆盖索引；如果你尝试在键和值列上定义一个复合次要索引，`InnoDB`会返回错误。 |

##### 容器表列约束

+   您必须为`db_schema`、`db_name`、`key_columns`、`value_columns`和`unique_idx_name_on_key`提供值。如果未使用，将`flags`、`cas_column`和`expire_time_column`设置为`0`。如果未这样做，可能会导致设置失败。

+   `key_columns`: **memcached**键的最大限制为 250 个字符，由**memcached**强制执行。映射键必须是非空的`CHAR`或`VARCHAR`类型。

+   `value_columns`: 必须映射到`CHAR`、`VARCHAR`或`BLOB`列。没有长度限制，值可以为 NULL。

+   `cas_column`: `cas`值是 64 位整数。必须映射到至少 8 字节的`BIGINT`。如果您不使用此列，请设置值为`0`以指示未使用。

+   `expiration_time_column`: 必须映射到至少 4 字节的`INTEGER`。过期时间定义为 Unix 时间的 32 位整数（自 1970 年 1 月 1 日以来的秒数作为 32 位值），或从当前时间开始的秒数。对于后者，秒数不得超过 60*60*24*30（30 天内的秒数）。如果客户端发送的数字较大，则服务器将其视为真实的 Unix 时间值，而不是从当前时间的偏移量。如果不使用此列，请设置值为`0`以指示未使用。

+   `flags`: 必须映射到至少 32 位的`INTEGER`，可以为 NULL。如果您不使用此列，请设置值为`0`以指示未使用。

在插件加载时执行预检查以强制执行列约束。如果发现不匹配，插件将不加载。

##### 多值列映射

+   在插件初始化期间，当`InnoDB` **memcached**配置为`containers`表中定义的信息时，将验证`containers.value_columns`中定义的每个映射列与映射的`InnoDB`表。如果映射了多个`InnoDB`表列，则会检查以确保每个列存在且为正确类型。

+   在运行时，对于`memcached`插入操作，如果有比映射列数更多的分隔值，则仅取映射值的数量。例如，如果有六个映射列，并且提供了七个分隔值，则仅取前六个分隔值。第七个分隔值将被忽略。

+   如果分隔值少于映射的列数，未填充的列将被设置为 NULL。如果无法将未填充的列设置为 NULL，则插入操作将失败。

+   如果一个表的列数多于映射值，额外的列不会影响结果。

#### demo_test 示例表

`innodb_memcached_config.sql` 配置脚本在 `test` 数据库中创建了一个 `demo_test` 表，可用于在安装`InnoDB` **memcached** 插件后立即验证安装。

`innodb_memcached_config.sql` 配置脚本还在 `innodb_memcache.containers` 表中为 `demo_test` 表创建了一个条目。

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
