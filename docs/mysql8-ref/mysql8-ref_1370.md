> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html)

#### 19.1.6.4 二进制日志选项和变量

+   启动时与二进制日志一起使用的选项

+   与二进制日志一起使用的系统变量

您可以使用本节中描述的**mysqld**选项和系统变量来影响二进制日志的操作，以及控制写入二进制日志的语句。有关二进制日志的其他信息，请参见第 7.4.4 节，“二进制日志”。有关使用 MySQL 服务器选项和系统变量的其他信息，请参见第 7.1.7 节，“服务器命令选项”和第 7.1.8 节，“服务器系统变量”。

##### 启动时与二进制日志一起使用的选项

以下列表描述了启用和配置二进制日志的启动选项。稍后在本节中讨论与二进制日志一起使用的系统变量。

+   `--binlog-row-event-max-size=*`N`*`

    | 命令行格式 | `--binlog-row-event-max-size=#` |
    | --- | --- |
    | 系统变量 (≥ 8.0.14) | `binlog_row_event_max_size` |
    | 作用域 (≥ 8.0.14) | 全局 |
    | 动态 (≥ 8.0.14) | 否 |
    | `SET_VAR` 提示适用 (≥ 8.0.14) | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `256` |
    | 最大值 (64 位平台) | `18446744073709551615` |
    | 最大值 (32 位平台) | `4294967295` |
    | 单位 | 字节 |

    当使用基于行的二进制日志记录时，此设置是行级二进制日志事件的最大大小软限制，以字节为单位。在可能的情况下，存储在二进制日志中的行被分组为大小不超过此设置值的事件。如果无法拆分事件，则最大大小可能会超过。该值必须是（否则会被四舍五入为）256 的倍数。默认值为 8192 字节。

+   [`--log-bin[=*`base_name`*]`](replication-options-binary-log.html#option_mysqld_log-bin)

    | 命令行格式 | `--log-bin=file_name` |
    | --- | --- |
    | 类型 | 文件名 |

    指定用于二进制日志文件的基本名称。启用二进制日志记录后，服务器将所有更改数据的语句记录到二进制日志中，用于备份和复制。二进制日志是具有基本名称和数字扩展的文件序列。`--log-bin`选项值是日志序列的基本名称。服务器通过向基本名称添加数字后缀来按顺序创建二进制日志文件。

    如果不提供`--log-bin`选项，MySQL 将使用`binlog`作为二进制日志文件的默认基本名称。为了与早期版本兼容，如果提供了`--log-bin`选项但没有字符串或为空字符串，则基本名称默认为`*`host_name`*-bin`，使用主机机器的名称。

    二进制日志文件的默认位置是数据目录。您可以使用`--log-bin`选项指定另一个位置，通过在基本名称前添加一个绝对路径名来指定不同的目录。当服务器从跟踪已使用的二进制日志文件的二进制日志索引文件中读取条目时，它会检查条目是否包含相对路径。如果包含相对路径，则使用`--log-bin`选项设置的绝对路径将替换路径的相对部分。在二进制日志索引文件中记录的绝对路径保持不变；在这种情况下，必须手动编辑索引文件以启用新路径或路径的使用。二进制日志文件基本名称和任何指定的路径可作为`log_bin_basename`系统变量使用。

    在早期的 MySQL 版本中，默认情况下禁用了二进制日志记录，只有在指定了`--log-bin`选项时才会启用。从 MySQL 8.0 开始，默认情况下启用了二进制日志记录，无论您是否指定了`--log-bin`选项。唯一的例外是，如果您使用**mysqld**手动初始化数据目录，通过使用`--initialize`或`--initialize-insecure`选项调用它时，默认情况下会禁用二进制日志记录。在这种情况下，可以通过指定`--log-bin`选项来启用二进制日志记录。当启用二进制日志记录时，显示服务器上二进制日志记录状态的`log_bin`系统变量将设置为 ON。

    要禁用二进制日志记录，您可以在启动时指定`--skip-log-bin`或`--disable-log-bin`选项。如果指定了其中任何一个选项，并且同时指定了`--log-bin`选项，则后面指定的选项优先。当禁用二进制日志记录时，`log_bin`系统变量将设置为 OFF。

    当服务器上使用 GTIDs 时，如果在异常关闭后重新启动服务器时禁用二进制日志记录，可能会丢失一些 GTIDs，导致复制失败。在正常关闭时，当前二进制日志文件中的 GTID 集合将保存在`mysql.gtid_executed`表中。在未发生此情况的异常关闭后，在恢复过程中，GTIDs 将从二进制日志文件中添加到表中，前提是二进制日志记录仍然启用。如果在服务器重新启动时禁用了二进制日志记录，则服务器无法访问二进制日志文件以恢复 GTIDs，因此无法启动复制。在正常关闭后可以安全地禁用二进制日志记录。

    `--log-slave-updates`和`--slave-preserve-commit-order`选项需要二进制日志记录。如果禁用二进制日志记录，则省略这些选项，或指定`--log-slave-updates=OFF`和`--skip-slave-preserve-commit-order`。当指定`--skip-log-bin`或`--disable-log-bin`时，MySQL 默认禁用这些选项。如果同时指定`--log-slave-updates`或`--slave-preserve-commit-order`和`--skip-log-bin`或`--disable-log-bin`，将发出警告或错误消息。

    在 MySQL 5.7 中，启用二进制日志记录时必须指定服务器 ID，否则服务器将无法启动。在 MySQL 8.0 中，默认情况下将`server_id`系统变量设置为 1。当启用二进制日志记录时，现在可以使用此默认服务器 ID 启动服务器，但如果您没有通过设置`server_id`系统变量显式指定服务器 ID，则会发出信息消息。对于用于复制拓扑的服务器，必须为每个服务器指定唯一的非零服务器 ID。

    有关二进制日志的格式和管理信息，请参阅第 7.4.4 节，“二进制日志”。

+   [`--log-bin-index[=*`file_name`*]`](replication-options-binary-log.html#option_mysqld_log-bin-index)

    | 命令行格式 | `--log-bin-index=file_name` |
    | --- | --- |
    | 系统变量 | `log_bin_index` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 文件名 |

    二进制日志索引文件的名称，其中包含二进制日志文件的名称。默认情况下，它具有与使用`--log-bin`选项指定的二进制日志文件相同的位置和基本名称，再加上扩展名`.index`。如果您没有指定`--log-bin`，默认的二进制日志索引文件名为`binlog.index`。如果您指定了不带字符串或空字符串的`--log-bin`选项，则默认的二进制日志索引文件名为`*`host_name`*-bin.index`，使用主机机器的名称。

    有关二进制日志的格式和管理信息，请参见第 7.4.4 节，“二进制日志”。

**语句选择选项。** 下面列表中的选项影响写入二进制日志的语句，从而由复制源服务器发送到其副本。还有一些用于副本的选项，控制应该执行或忽略来自源的语句。有关详细信息，请参见第 19.1.6.3 节，“副本服务器选项和变量”。

+   `--binlog-do-db=*`db_name`*`

    | 命令行格式 | `--binlog-do-db=name` |
    | --- | --- |
    | 类型 | 字符串 |

    此选项对二进制日志记录的影响类似于`--replicate-do-db`对复制的影响。

    此选项的影响取决于是否使用基于语句或基于行的日志格式，就像`--replicate-do-db`的影响取决于是否使用基于语句或基于行的复制一样。您应该记住，用于记录给定语句的格式可能不一定与`binlog_format`的值所指示的格式相同。例如，DDL 语句如`CREATE TABLE`和`ALTER TABLE`始终作为语句记录，而不考虑生效的日志格式，因此对于`--binlog-do-db`的以下基于语句的规则始终适用于确定语句是否被记录。

    **基于语句的日志记录。** 只有那些默认数据库（即由`USE`选择的数据库）为*`db_name`*的语句才会被写入二进制日志。要指定多个数据库，请多次使用此选项，每个数据库使用一次；但是，这样做不会导致跨数据库语句（例如`UPDATE *`some_db.some_table`* SET foo='bar'`）在选择不同数据库（或没有数据库）时被记录。

    警告

    要指定多个数据库，必须多次使用此选项。因为数据库名称可以包含逗号，如果提供逗号分隔的列表，则该列表被视为单个数据库的名称。

    使用基于语句的日志记录时不像您期望的那样工作的示例：如果服务器使用`--binlog-do-db=sales`启动，并且您发出以下语句，则`UPDATE`语句不会被记录：

    ```sql
    USE prices;
    UPDATE sales.january SET amount=amount+1000;
    ```

    这种“只需检查默认数据库”行为的主要原因是，仅凭语句本身很难知道是否应该复制（例如，如果您使用多表`DELETE`语句或跨多个数据库操作的多表`UPDATE`语句）。如果没有必要，仅检查默认数据库而不是所有数据库会更快。

    另一个可能不明显的情况是，即使在设置选项时没有指定，给定的数据库也被复制。如果服务器使用`--binlog-do-db=sales`启动，即使在设置`--binlog-do-db`时未包括`prices`，以下`UPDATE`语句也会被记录：

    ```sql
    USE sales;
    UPDATE prices.discounts SET percentage = percentage + 10;
    ```

    因为`sales`是发出`UPDATE`语句时的默认数据库，所以`UPDATE`被记录。

    **基于行的日志记录。** 记录仅限于数据库*`db_name`*。只有属于*`db_name`*的表的更改才会被记录；默认数据库对此没有影响。假设服务器使用`--binlog-do-db=sales`启动，并且基于行的日志记录生效，然后执行以下语句：

    ```sql
    USE prices;
    UPDATE sales.february SET amount=amount+100;
    ```

    `sales`数据库中`february`表的更改将根据`UPDATE`语句记录；无论是否发出了`USE`语句都会发生。然而，当使用基于行的日志记录格式和`--binlog-do-db=sales`时，以下`UPDATE`所做的更改不会被记录：

    ```sql
    USE prices;
    UPDATE prices.march SET amount=amount-25;
    ```

    即使`USE prices`语句被更改为`USE sales`，`UPDATE`语句的效果仍不会被写入二进制日志。

    在基于语句的日志记录和基于行的日志记录中，关于涉及多个数据库的语句的`--binlog-do-db`处理存在另一个重要区别。假设服务器是使用`--binlog-do-db=db1`启动的，并且执行了以下语句：

    ```sql
    USE db1;
    UPDATE db1.table1, db2.table2 SET db1.table1.col1 = 10, db2.table2.col2 = 20;
    ```

    如果使用基于语句的日志记录，两个表的更新都会被写入二进制日志。然而，当使用基于行的格式时，只有`table1`的更改被记录；`table2`位于不同的数据库中，因此不会受到`UPDATE`的影响。现在假设，而不是使用`USE db1`语句，而是使用了`USE db4`语句：

    ```sql
    USE db4;
    UPDATE db1.table1, db2.table2 SET db1.table1.col1 = 10, db2.table2.col2 = 20;
    ```

    当使用基于语句的日志记录时，使用`UPDATE`语句时不会被写入二进制日志。然而，当使用基于行的日志记录时，`table1`的更改被记录，但`table2`的更改不会被记录——换句话说，只有在由`--binlog-do-db`指定的数据库中的表的更改才会被记录，并且默认数据库的选择对此行为没有影响。

+   `--binlog-ignore-db=*`db_name`*`

    | 命令行格式 | `--binlog-ignore-db=name` |
    | --- | --- |
    | 类型 | 字符串 |

    这个选项对二进制日志记录的影响类似于`--replicate-ignore-db`对复制的影响。

    此选项的效果取决于是否使用基于语句或基于行的日志记录格式，就像`--replicate-ignore-db`的效果取决于是否使用基于语句或基于行的复制一样。您应该记住，用于记录给定语句的格式可能不一定与`binlog_format`的值所指示的格式相同。例如，DDL 语句（如`CREATE TABLE`和`ALTER TABLE`）始终作为语句记录，而不考虑生效的日志记录格式，因此对于`--binlog-ignore-db`的以下基于语句的规则始终适用于确定该语句是否被记录。

    **基于语句的日志记录。** 告诉服务器不记录默认数据库（即由`USE`选择的数据库）为*`db_name`*的任何语句。

    当没有默认数据库时，不会应用`--binlog-ignore-db`选项，并且这些语句始终被记录。 (Bug #11829838, Bug #60188)

    **基于行的格式。** 告诉服务器不要记录对数据库*`db_name`*中任何表的更新。当前数据库不起作用。

    在使用基于语句的日志记录时，以下示例不会按您期望的方式工作。假设服务器是使用`--binlog-ignore-db=sales`启动的，并且您发出以下语句：

    ```sql
    USE prices;
    UPDATE sales.january SET amount=amount+1000;
    ```

    在这种情况下，`UPDATE`语句会被记录，因为`--binlog-ignore-db`仅适用于默认数据库（由`USE`语句确定）。由于`sales`数据库在语句中明确指定，因此该语句未被过滤。但是，在使用基于行的日志记录时，`UPDATE`语句的效果不会写入二进制日志，这意味着`sales.january`表的任何更改都不会被记录；在这种情况下，`--binlog-ignore-db=sales`导致忽略对源数据库副本中`sales`数据库中表所做的所有更改以进行二进制日志记录。

    要忽略多个数据库，请多次使用此选项，每个数据库使用一次。因为数据库名称可以包含逗号，如果提供逗号分隔的列表，则该列表将被视为单个数据库的名称。

    如果您正在使用跨数据库更新并且不希望将这些更新记录下来，则不应使用此选项。

**校验和选项。** MySQL 支持读取和写入二进制日志校验和。可以使用以下两个选项启用它们：

+   `--binlog-checksum={NONE|CRC32}`

    | 命令行格式 | `--binlog-checksum=type` |
    | --- | --- |
    | 类型 | 字符串 |
    | 默认值 | `CRC32` |
    | 有效值 | `NONE``CRC32` |

    启用此选项会导致源代码为写入二进制日志的事件生成校验和。设置为`NONE`以禁用，或者设置要用于生成校验和的算法的名称；目前仅支持 CRC32 校验和，CRC32 是默认值。您不能在事务内更改此选项的设置。

要控制副本（从中继日志）读取校验和，请使用`--slave-sql-verify-checksum`选项。

**测试和调试选项。** 以下二进制日志选项用于复制测试和调试。它们不适用于正常操作。

+   `--max-binlog-dump-events=*`N`*`

    | 命令行格式 | `--max-binlog-dump-events=#` |
    | --- | --- |
    | 类型 | 整数 |
    | 默认值 | `0` |

    此选项仅在 MySQL 测试套件内部用于复制测试和调试。

+   `--sporadic-binlog-dump-fail`

    | 命令行格式 | `--sporadic-binlog-dump-fail[={OFF&#124;ON}]` |
    | --- | --- |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    此选项仅在 MySQL 测试套件内部用于复制测试和调试。

##### 与二进制日志一起使用的系统变量

以下列表描述了用于控制二进制日志记录的系统变量。它们可以在服务器启动时设置，并且其中一些可以使用 `SET` 在运行时更改。用于控制二进制日志记录的服务器选项在本节的前面列出。

+   `binlog_cache_size`

    | 命令行格式 | `--binlog-cache-size=#` |
    | --- | --- |
    | 系统变量 | `binlog_cache_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709547520` |
    | 最大值（32 位平台） | `4294963200` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    在事务期间保存对二进制日志的更改的内存缓冲区的大小。

    当服务器启用二进制日志记录（使用 `log_bin` 系统变量设置为 ON）时，如果服务器支持任何事务性存储引擎，则为每个客户端分配一个二进制日志缓存。如果事务的数据超过内存缓冲区的空间，多余的数据将存储在临时文件中。当服务器上启用二进制日志加密时，内存缓冲区不加密，但（从 MySQL 8.0.17 开始）用于保存二进制日志缓存的任何临时文件是加密的。在每个事务提交后，通过清除内存缓冲区并截断使用的临时文件来重置二进���日志缓存。

    如果经常使用大型事务，可以增加此缓存大小以获得更好的性能，从而减少或消除写入临时文件的需求。`Binlog_cache_use` 和 `Binlog_cache_disk_use` 状态变量可用于调整此变量的大小。参见 第 7.4.4 节，“二进制日志”。

    `binlog_cache_size` 仅设置事务缓存的大小；语句缓存的大小由 `binlog_stmt_cache_size` 系统变量控制。

+   `binlog_checksum`

    | 命令行格式 | `--binlog-checksum=type` |
    | --- | --- |
    | 系统变量 | `binlog_checksum` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `CRC32` |
    | 有效值 | `NONE``CRC32` |

    启用此变量会导致源为二进制日志中的每个事件编写校验和。`binlog_checksum`支持值`NONE`（禁用校验和）和`CRC32`。默认值为`CRC32`。当禁用`binlog_checksum`（值为`NONE`）时，服务器通过为每个事件编写和检查事件长度（而不是校验和）来验证仅将完整事件写入二进制日志。

    在源上将此变量设置为副本不识别的值会导致副本将其自己的`binlog_checksum`值设置为`NONE`，并因错误而停止复制。如果与旧副本的向后兼容性是一个问题，您可能希望将值明确设置为`NONE`。

    直到 MySQL 8.0.20，Group Replication 无法使用校验和，也不支持二进制日志中的校验和，因此在配置服务器实例成为组成员时，必须设置`binlog_checksum=NONE`。从 MySQL 8.0.21 开始，Group Replication 支持校验和，因此组成员可以使用默认设置。

    更改`binlog_checksum`的值会导致二进制日志被旋转，因为必须为整个二进制日志文件编写校验和，而不能仅为部分文件编写。您不能在事务内更改`binlog_checksum`的值。

    当使用`binlog_transaction_compression`系统变量启用二进制日志事务压缩时，不会为压缩的事务负载中的每个事件编写校验和。相反，为 GTID 事件编写校验和，并为压缩的`Transaction_payload_event`编写校验和。

+   `binlog_direct_non_transactional_updates`

    | 命令行格式 | `--binlog-direct-non-transactional-updates[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `binlog_direct_non_transactional_updates` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    由于并发问题，当事务包含对事务表和非事务表的更新时，复制品可能会变得不一致。MySQL 试图通过将非事务语句写入事务缓存来保留这些语句之间的因果关系，该缓存在提交时被刷新。然而，当代表事务对非事务表进行的修改立即对其他连接可见时，问题就会出现，因为这些更改可能不会立即���入二进制日志。

    `binlog_direct_non_transactional_updates`变量提供了解决此问题的一种可能方法。默认情况下，此变量已禁用。启用`binlog_direct_non_transactional_updates`会导致对非事务表的更新直接写入二进制日志，而不是写入事务缓存。

    截至 MySQL 8.0.14 版本，设置此系统变量的会话值是一项受限操作。会话用户必须具有足够权限来设置受限会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    *`binlog_direct_non_transactional_updates`仅适用于使用基于语句的二进制日志格式复制的语句*；也就是说，仅当`binlog_format`的值为`STATEMENT`时，或者当`binlog_format`为`MIXED`且给定语句正在使用基于语句的格式进行复制时才起作用。当二进制日志格式为`ROW`时，或者当`binlog_format`设置为`MIXED`且给定语句使用基于行的格式进行复制时，此变量不起作用。

    重要提示

    在启用此变量之前，您必须确保事务表和非事务表之间没有依赖关系；这种依赖关系的一个示例是语句`INSERT INTO myisam_table SELECT * FROM innodb_table`。否则，这些语句很可能会导致复制品与源之间出现分歧。

    当二进制日志格式为`ROW`或`MIXED`时，此变量不起作用。

+   `binlog_encryption`

    | 命令行格式 | `--binlog-encryption[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `binlog_encryption` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用此服务器上二进制日志文件和中继日志文件的加密。`OFF`是默认设置。`ON`设置二进制日志文件和中继日志文件的加密。不需要在服务器上启用二进制日志记录以启用加密，因此可以在没有二进制日志的副本上加密中继日志文件。要使用加密，必须安装并配置一个密钥环插件以提供 MySQL 服务器的密钥环服务。有关如何执行此操作的说明，请参见第 8.4.4 节，“MySQL 密钥环”。任何受支持的密钥环插件都可以用于存储二进制日志加密密钥。

    当首次启用二进制日志加密启动服务器时，在初始化二进制日志和中继日志之前会生成一个新的二进制日志加密密钥。此密钥用于为每个二进制日志文件（如果服务器启用了二进制日志记录）和中继日志文件（如果服务器有复制通道）加密一个文件密码，并且从文件密码生成的进一步密钥用于加密文件中的数据。中继日志文件为所有通道加密，包括组复制应用程序通道和在激活加密后创建的新通道。二进制日志索引文件和中继日志索引文件永远不会被加密。

    如果在服务器运行时激活加密，那么此时会生成一个新的二进制日志加密密钥。例外情况是，如果以前在服务器上激活了加密，然后禁用了，那么之前使用的二进制日志加密密钥将再次被使用。二进制日志文件和中继日志文件立即进行轮换，并且新文件和所有后续二进制日志文件和中继日志文件的文件密码都使用此二进制日志加密密钥进行加密。仍然存在于服务器上的现有二进制日志文件和中继日志文件不会自动加密，但如果不再需要，可以将它们清除。

    如果通过将`binlog_encryption`系统变量更改为`OFF`来停用加密，则二进制日志文件和中继日志文件将立即进行轮换，并且所有后续记录都将是未加密的。以前加密的文件不会自动解密，但服务器仍然能够读取它们。激活或停用加密时需要`BINLOG_ENCRYPTION_ADMIN`权限（或已弃用的`SUPER`权限）。组复制应用程序通道不包括在中继日志轮换请求中，因此在正常使用中，这些通道的未加密记录直到它们的日志轮换后才开始。

    有关二进制日志文件和中继日志文件加密的更多信息，请参见第 19.3.2 节，“加密二进制日志文件和中继日志文件”。

+   `binlog_error_action`

    | 命令行格式 | `--binlog-error-action[=value]` |
    | --- | --- |
    | 系统变量 | `binlog_error_action` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ABORT_SERVER` |
    | 有效值 | `IGNORE_ERROR``ABORT_SERVER` |

    控制服务器遇到无法写入、刷新或同步二进制日志等错误时会发生什么，这可能导致源二进制日志不一致，副本失去同步。

    此变量默认为`ABORT_SERVER`，当二进制日志遇到此类错误时，服务器会停止记录并关闭。在重新启动时，恢复的过程与意外服务器停止的情况相同（参见第 19.4.2 节，“处理复制的意外停止”）。

    当`binlog_error_action`设置为`IGNORE_ERROR`时，如果服务器遇到此类错误，它会继续进行当前事务，记录错误然后停止记录，并继续执行更新。要恢复二进制日志记录，必须重新启用`log_bin`，这需要重新启动服务器。此设置与 MySQL 的旧版本向后兼容。

+   `binlog_expire_logs_seconds`

    | 命令行格式 | `--binlog-expire-logs-seconds=#` |
    | --- | --- |
    | 系统变量 | `binlog_expire_logs_seconds` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `2592000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 秒 |

    设置二进制日志过期时间（以秒为单位）。在过期后，二进制日志文件可以自动删除。可能的删除发生在启动时和二进制日志刷新时。日志刷新的发生如第 7.4 节，“MySQL 服务器日志”所示。

    默认的二进制日志过期时间为 2592000 秒，相当于 30 天（30*24*60*60 秒）。如果在启动时既没有为`binlog_expire_logs_seconds`设置值，也没有为废弃的系统变量`expire_logs_days`设置值，则使用默认值。如果在启动时为`binlog_expire_logs_seconds`或`expire_logs_days`中的一个变量设置了非零值，则该值将用作二进制日志的过期时间。如果在启动时为这两个变量都设置了非零值，则`binlog_expire_logs_seconds`的值将用作二进制日志的过期时间，而`expire_logs_days`的值将被忽略并显示警告消息。

    在运行时，如果另一个变量`binlog_expire_logs_seconds`或`expire_logs_days`当前已设置为非零值，则不能将`binlog_expire_logs_seconds`或`expire_logs_days`设置为非零值。由于`binlog_expire_logs_seconds`的默认值为非零，因此必须在设置或更改`expire_logs_days`的值之前明确将`binlog_expire_logs_seconds`设置为零。

    从 MySQL 8.0.29 开始，可以通过将系统变量`binlog_expire_logs_auto_purge`设置为`OFF`来禁用二进制日志的自动清理。这优先于`binlog_expire_logs_seconds`的任何设置。

    在 MySQL 8.0.28 及更早版本中，要禁用二进制日志的自动清理，必须明确为`binlog_expire_logs_seconds`指定值为 0，并且不为`expire_logs_days`指定值。为了与早期版本兼容，如果您明确为`expire_logs_days`指定值为 0，并且不为`binlog_expire_logs_seconds`指定值，则也会禁用自动清理。在这种情况下，不会应用`binlog_expire_logs_seconds`的默认值。

    要手动删除二进制日志文件，请使用`PURGE BINARY LOGS`语句。参见 Section 15.4.1.1, “PURGE BINARY LOGS Statement”。

+   `binlog_expire_logs_auto_purge`

    | 命令行格式 | `--binlog-expire-logs-auto-purge={ON&#124;OFF}` |
    | --- | --- |
    | 引入版本 | 8.0.29 |
    | 系统变量 | `binlog_expire_logs_auto_purge` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    启用或禁用二进制日志文件的自动清理。将此变量设置为`ON`（默认值）启用自动清理；将其设置为`OFF`禁用自动清理。等待清理的间隔由`binlog_expire_logs_seconds`和`expire_logs_days`控制。

    注意

    即使`binlog_expire_logs_auto_purge`为`ON`，将`binlog_expire_logs_seconds`和`expire_logs_days`都设置为`0`也会阻止自动清理操作的进行。

    此变量对`PURGE BINARY LOGS`没有影响。

+   `binlog_format`

    | 命令行格式 | `--binlog-format=format` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `binlog_format` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ROW` |
    | 有效值 | `MIXED``STATEMENT``ROW` |

    此系统变量设置了二进制日志记录格式，可以是`STATEMENT`、`ROW`或`MIXED`中的任何一种。（参见第 19.2.1 节，“复制格式”。）当服务器启用二进制日志记录时，即`log_bin`系统变量设置为`ON`时，设置生效。在 MySQL 8.0 中，默认情况下启用二进制日志记录，并且默认使用基于行的格式。

    注意

    `binlog_format`在 MySQL 8.0.34 中已被弃用，并可能在将来的 MySQL 版本中被移除。这意味着除基于行的日志记录格式外的其他格式的支持也可能在将来的版本中被移除。因此，任何新的 MySQL 复制设置应仅使用基于行的日志记录。

    `binlog_format`可以在启动时或运行时设置，但在某些情况下，在运行时更改此变量是不可能的，或会导致复制失败，如后文所述。

    默认值为`ROW`。*例外*：在 NDB Cluster 中，默认值为`MIXED`；不支持基于语句的复制。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参见第 7.1.9.1 节，“系统变量权限”。

    更改此变量生效的规则和效果持续时间与其他 MySQL 服务器系统变量相同。有关更多信息，请参见第 15.7.6.1 节，“变量赋值的 SET 语法”。

    当指定`MIXED`时，将使用基于语句的复制，但有例外情况，即仅当仅有基于行的复制保证会产生正确结果时才会使用基于行的复制。例如，当语句包含可加载函数或`UUID()`函数时会发生这种情况。

    有关在设置每个二进制日志格式时如何处理存储程序（存储过程和函数、触发器和事件）的详细信息，请参见 Section 27.7，“存储程序二进制日志记录”。

    在以下情况下无法在运行时切换复制格式：

    +   无法从存储函数或触发器内部更改复制格式。

    +   如果会话有打开的临时表，则无法为该会话更改复制格式（`SET @@SESSION.binlog_format`）。

    +   如果任何复制通道有打开的临时表，则无法全局更改复制格式（`SET @@GLOBAL.binlog_format`或`SET @@PERSIST.binlog_format`）。

    +   如果任何复制通道应用程序线程当前正在运行，则无法全局更改复制格式（`SET @@GLOBAL.binlog_format`或`SET @@PERSIST.binlog_format`）。

    在任何情况下尝试切换复制格式（或尝试设置当前复制格式）都会导致错误。但是，您可以随时使用`PERSIST_ONLY`（`SET @@PERSIST_ONLY.binlog_format`）来更改复制格式，因为此操作不会修改运行时全局系统变量的值，并且只在服务器重新启动后生效。

    当存在任何临时表时不建议在运行时切换复制格式，因为仅在使用基于语句的复制时才记录临时表，而在基于行的复制和混合复制中，它们不会被记录���

    在复制源服务器上更改日志格式不会导致副本更改其日志格式以匹配。在复制正在进行时切换复制格式可能会导致问题，如果副本启用了二进制日志记录，并且更改导致副本使用`STATEMENT`格式记录日志，而源使用`ROW`或`MIXED`格式记录日志。副本无法将以`ROW`记录格式接收的二进制日志条目转换为`STATEMENT`格式以在自己的二进制日志中使用，因此此情况可能导致复制失败。有关更多信息，请参见 Section 7.4.4.2，“设置二进制日志格式”。

    二进制日志格式会影响以下服务器选项的行为：

    +   `--replicate-do-db`

    +   `--replicate-ignore-db`

    +   `--binlog-do-db`

    +   `--binlog-ignore-db`

    这些效果在���个选项的描述中有详细讨论。

+   `binlog_group_commit_sync_delay`

    | 命令行格式 | `--binlog-group-commit-sync-delay=#` |
    | --- | --- |
    | 系统变量 | `binlog_group_commit_sync_delay` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `1000000` |
    | 单位 | 微秒 |

    控制二进制日志提交在将二进制日志文件同步到磁盘之前等待多少微秒。默认情况下，`binlog_group_commit_sync_delay`设置为 0，意味着没有延迟。将`binlog_group_commit_sync_delay`设置为微秒延迟可以使更多事务一起同步到磁盘，从而减少提交一组事务的总时间，因为较大的组需要更少的时间单位来完成。

    当设置`sync_binlog=0`或`sync_binlog=1`时，`binlog_group_commit_sync_delay`指定的延迟将应用于每个二进制日志提交组在同步之前（或在`sync_binlog=0`的情况下，在继续之前）。当将`sync_binlog`设置为大于 1 的值*n*时，延迟将应用于每*n*个二进制日志提交组之后。

    设置`binlog_group_commit_sync_delay`可以增加任何具有（或在故障转移后可能具有）副本的服务器上并因此可以增加副本的并行执行的提交事务数量。要从这种效果中受益，副本服务器必须设置`replica_parallel_type=LOGICAL_CLOCK`（从 MySQL 8.0.26 开始）或`slave_parallel_type=LOGICAL_CLOCK`，并且当`binlog_transaction_dependency_tracking=COMMIT_ORDER`也设置时效果更显著。在调整`binlog_group_commit_sync_delay`的设置时，重要的是考虑源的吞吐量和副本的吞吐量。

    设置`binlog_group_commit_sync_delay`也可以减少在任何具有二进制日志的服务器（源或副本）上对二进制日志执行`fsync()`调用的次数。

    请注意，设置`binlog_group_commit_sync_delay`会增加服务器上事务的延迟，可能会影响客户端应用程序。此外，在高并发工作负载下，延迟可能增加争用，从而降低吞吐量。通常，设置延迟的好处超过了缺点，但应始终进行调整以确定最佳设置。

+   `binlog_group_commit_sync_no_delay_count`

    | 命令行格式 | `--binlog-group-commit-sync-no-delay-count=#` |
    | --- | --- |
    | 系统变量 | `binlog_group_commit_sync_no_delay_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `100000` |

    在由`binlog_group_commit_sync_delay`指定的当前延迟中放弃之前等待的最大事务数。如果`binlog_group_commit_sync_delay`设置为 0，则此选项无效。

+   `binlog_max_flush_queue_time`

    | 命令行格式 | `--binlog-max-flush-queue-time=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `binlog_max_flush_queue_time` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `100000` |
    | 单位 | 微秒 |

    `binlog_max_flush_queue_time`已被弃用，并标记为在未来的 MySQL 版本中最终移除。以前，此系统变量控制了继续从刷新队列中读取事务的时间（以微秒为单位），然后进行组提交。它不再起作用。

+   `binlog_order_commits`

    | 命令行格式 | `--binlog-order-commits[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `binlog_order_commits` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    当在复制源服务器上启用此变量（默认情况下）时，发给存储引擎的事务提交指令在单个线程上串行化，因此事务总是按照写入二进制日志的顺序提交。禁用此变量允许使用多个线程发出事务提交指令。与二进制日志组提交结合使用，这可以防止单个事务的提交速率成为吞吐量的瓶颈，并可能产生性能改进。

    事务在所有涉及的存储引擎确认事务准备提交时被写入二进制日志。然后二进制日志组提交逻辑在其写入二进制日志后提交一组事务。当禁用`binlog_order_commits`时，由于使用多个线程进行此过程，提交组中的事务可能按照不同于二进制日志中的顺序提交。（来自单个客户端的事务总是按照时间顺序提交。）在许多情况下，这并不重要，因为在单独事务中执行的操作应该产生一致的结果，如果不是这种情况，则应该使用单个事务。

    如果您希望确保源端和多线程复制的事务历史保持一致，请在复制端设置`slave_preserve_commit_order=1`。

+   `binlog_rotate_encryption_master_key_at_startup`

    | 命令行格式 | `--binlog-rotate-encryption-master-key-at-startup[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.14 |
    | 系统变量 | `binlog_rotate_encryption_master_key_at_startup` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    指定在服务器启动时是否会轮换二进制日志主密钥。二进制日志主密钥是用于加密服务器上的二进制日志文件和中继日志文件的文件密码的二进制日志加密密钥。当启用二进制日志加密（`binlog_encryption=ON`）时，首次启动服务器时会生成一个新的二进制日志加密密钥，并用作二进制日志主密钥。如果 `binlog_rotate_encryption_master_key_at_startup` 系统变量也设置为 `ON`，每当服务器重新启动时，会生成一个进一步的二进制日志加密密钥，并用作所有后续二进制日志文件和中继日志文件的二进制日志主密钥。如果 `binlog_rotate_encryption_master_key_at_startup` 系统变量设置为 `OFF`，即默认情况下，服务器重新启动后会再次使用现有的二进制日志主密钥。有关二进制日志加密密钥和二进制日志主密钥的更多信息，请参见 Section 19.3.2, “Encrypting Binary Log Files and Relay Log Files”。

+   `binlog_row_event_max_size`

    | 命令行格式 | `--binlog-row-event-max-size=#` |
    | --- | --- |
    | 系统变量 (≥ 8.0.14) | `binlog_row_event_max_size` |
    | 范围 (≥ 8.0.14) | 全局 |
    | 动态 (≥ 8.0.14) | 否 |
    | `SET_VAR` 提示适用 (≥ 8.0.14) | 否 |
    | 类型 | 整数 |
    | 默认值 | `8192` |
    | 最小值 | `256` |
    | 最大值 (64 位平台) | `18446744073709551615` |
    | 最大值 (32 位平台) | `4294967295` |
    | 单位 | 字节 |

    当使用基于行的二进制日志记录时，此设置是行基二进制日志事件的最大大小的软限制，以字节为单位。在可能的情况下，存储在二进制日志中的行被分组为大小不超过此设置值的事件。如果无法拆分事件，则最大大小可能会超过。默认值为 8192 字节。

    此全局系统变量是只读的，只能在服务器启动时设置。因此，其值只能通过在 `SET` 语句中使用 `PERSIST_ONLY` 关键字或 `@@persist_only` 限定符来修改。

+   `binlog_row_image`

    | 命令行格式 | `--binlog-row-image=image_type` |
    | --- | --- |
    | 系统变量 | `binlog_row_image` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `full` |
    | 有效值 | `full`（记录所有列）`minimal`（仅记录更改的列和用于识别行的列）`noblob`（记录所有列，除了不需要的 BLOB 和 TEXT 列） |

    对于 MySQL 基于行的复制，此变量确定如何将行图像写入二进制日志。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅 第 7.1.9.1 节，“系统变量权限”。

    在 MySQL 基于行的复制中，每个行更改事件包含两个图像，一个“前”图像，其列与搜索要更新的行匹配，以及包含更改的“后”图像。通常，MySQL 记录完整行（即所有列）的前后图像。但是，并不严格要求在两个图像中都包含每一列，我们通常可以通过仅记录实际需要的列来节省磁盘、内存和网络使用。

    注意

    删除行时，仅记录前图像，因为删除后没有更改的值需要传播。插入行时，仅记录后图像，因为没有现有行需要匹配。仅在更新行时，才需要前后图像，并且两者都写入二进制日志。

    对于前图像，只需要记录一组最小列，以唯一标识行。如果包含行的表具有主键，则只会将主键列写入二进制日志。否则，如果表具有所有列均为 `NOT NULL` 的唯一键，则只需要记录唯一键中的列。 （如果表既没有主键也没有不带任何 `NULL` 列的唯一键，则必须在前图像中使用并记录所有列。）对于后图像，只需要记录实际更改的列。

    您可以使用 `binlog_row_image` 系统变量使服务器记录完整或最小行。实际上，该变量可以取以下三个可能的值，如下列表所示：

    +   `full`：记录前图像和后图像中的所有列。

    +   `minimal`：仅记录前图像中用于识别要更改的行的列；仅记录 SQL 语句指定的值或由自增生成的后图像中的列。

    +   `noblob`：记录所有列（与 `full` 相同），但不记录不需要识别行或未更改的 `BLOB` 和 `TEXT` 列。

    注意

    NDB Cluster 不支持此变量；设置它不会影响对`NDB`表的日志记录。

    默认值为`full`。

    当使用`minimal`或`noblob`时，仅当源表和目标表均满足以下条件时，删除和更新才能正确工作：

    +   所有列必须存在且顺序相同；每列必须使用与另一表中对应列相同的数据类型。

    +   表必须具有相同的主键定义。

    （换句话说，表必须相同，可能除了不是表主键的索引之外。）

    如果不满足这些条件，则可能目标表中的主键列值不足以为删除或更新提供唯一匹配。在这种情况下，不会发出警告或错误；源表和副本会默默地发生分歧，从而破坏一致性。

    当二进制日志格式为`STATEMENT`时，设置此变量不起作用。当`binlog_format`为`MIXED`时，`binlog_row_image`的设置将应用于使用基于行格式记录的更改，但此设置对作为语句记录的更改没有影响。

    在全局或会话级别设置`binlog_row_image`不会导致隐式提交；这意味着可以在事务进行中��改此变量而不影响事务。

+   `binlog_row_metadata`

    | 命令行格式 | `--binlog-row-metadata=metadata_type` |
    | --- | --- |
    | 系统变量 | `binlog_row_metadata` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `MINIMAL` |
    | 有效值 | `FULL`（包括所有元数据）`MINIMAL`（限制包含的元数据） |

    配置在使用基于行的日志记录时向二进制日志添加的表元数据的数量。当设置为`MINIMAL`时，默认情况下，只记录与`SIGNED`标志、列字符集和几何类型相关的元数据。当设置为`FULL`时，将记录完整的表元数据，例如列名、`ENUM`或`SET`字符串值、`PRIMARY KEY`信息等。

    扩展元数据具有以下目的：

    +   副本使用元数据在其表结构与源表不同的情况下传输数据。

    +   外部软件可以使用元数据解码行事件并将数据存储到外部数据库，例如数据仓库。

+   `binlog_row_value_options`

    | 命令行格式 | `--binlog-row-value-options=#` |
    | --- | --- |
    | 系统变量 | `binlog_row_value_options` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 设置 |
    | 默认值 |  |
    | 有效值 | `PARTIAL_JSON` |

    当设置为`PARTIAL_JSON`时，这将启用一种节省空间的二进制日志格式，用于修改 JSON 文档中仅有小部分内容的更新，这会导致基于行的复制仅将 JSON 文档的修改部分写入二进制日志中的更新后图像，而不是写入完整文档（参见 JSON 值的部分更新）。这适用于使用任何`JSON_SET()`、`JSON_REPLACE()`和`JSON_REMOVE()`序列修改 JSON 列的`UPDATE`语句。如果服务器无法生成部分更新，则使用完整文档。

    默认值是一个空字符串，这会禁用该格式的使用。要取消`binlog_row_value_options`并恢复写入完整 JSON 文档，将其值设置为空字符串。

    设置此系统变量的会话值是一项受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    `binlog_row_value_options=PARTIAL_JSON`仅在启用二进制日志记录且`binlog_format`设置为`ROW`或`MIXED`时生效。基于语句的复制*始终*仅记录 JSON 文档的修改部分，而不管为`binlog_row_value_options`设置了任何值。为了最大化节省的空间量，请与此选项一起使用`binlog_row_image=NOBLOB`或`binlog_row_image=MINIMAL`。`binlog_row_image=FULL`保存的空间比这两者少，因为完整的 JSON 文档存储在前图像中，而部分更新仅存储在后图像中。

    **mysqlbinlog**输出包括以使用`BINLOG`语句编码为 base-64 字符串的事件形式的部分 JSON 更新。如果指定了`--verbose`选项，**mysqlbinlog**将以可读的 JSON 形式使用伪 SQL 语句显示部分 JSON 更新。

    MySQL 复制在副本上无法应用修改到 JSON 文档时会生成错误。这包括找不到路径的情况。请注意，即使有此类和其他安全检查，如果副本上的 JSON 文档与源上的不同，并且应用了部分更新，仍然存在在副本上生成有效但意外的 JSON 文档的理论可能性。

+   `binlog_rows_query_log_events`

    | 命令行格式 | `--binlog-rows-query-log-events[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `binlog_rows_query_log_events` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此系统变量仅影响基于行的日志记录。启用时，会导致服务器将信息性日志事件（如行查询日志事件）写入其二进制日志中。此信息可用于调试和相关目的，例如在无法从行更新中重建时获取源上发出的原始查询。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    这些信息事件通常被 MySQL 程序忽略，读取二进制日志时不会引起任何问题，因此在复制或从备份中恢复时也不会有问题。要查看它们，请通过使用 mysqlbinlog 的`--verbose`选项两次增加详细程度，可以是`-vv`或`--verbose --verbose`。

+   `binlog_stmt_cache_size` |

    | 命令行格式 | `--binlog-stmt-cache-size=#` |
    | --- | --- |
    | 系统变量 | `binlog_stmt_cache_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32768` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709547520` |
    | 最大值（32 位平台） | `4294963200` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    用于二进制日志的内存缓冲区大小，用于保存在事务期间发出的非事务性语句。

    当服务器上启用了二进制日志记录（使用`log_bin`系统变量设置为 ON），如果服务器支持任何事务存储引擎，则为每个客户端分配单独的二进制日志事务和语句缓存。如果事务中使用的非事务性语句的数据超过内存缓冲区中的空间，则多余的数据将存储在临时文件中。当服务器上激活二进制日志加密时，内存缓冲区不加密，但（从 MySQL 8.0.17 开始）用于保存二进制日志缓存的任何临时文件是加密的。在每个事务提交后，通过清除内存缓冲区并截断使用的临时文件来重置二进制日志语句缓存。

    如果您在事务中经常使用大型非事务性语句，可以增加此缓存大小以通过减少或消除写入临时文件的需求来获得更好的性能。`Binlog_stmt_cache_use`和`Binlog_stmt_cache_disk_use`状态变量可用于调整此变量的大小。请参阅第 7.4.4 节，“二进制日志”。

    `binlog_cache_size`系统变量设置事务缓存的大小。

+   `binlog_transaction_compression`

    | 命令行格式 | `--binlog-transaction-compression[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.20 |
    | 系统变量 | `binlog_transaction_compression` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    启用对在此服务器上写入二进制日志文件的事务的压缩。`OFF`是默认设置。使用`binlog_transaction_compression_level_zstd`系统变量设置用于压缩的 zstd 算法级别。

    启用二进制日志事务压缩后，事务负载将被压缩，然后作为单个事件(`Transaction_payload_event`)写入二进制日志文件。压缩的事务负载在发送到副本、其他 Group Replication 组成员或诸如**mysqlbinlog**等客户端时仍保持压缩状态，并且在中继日志中仍以压缩状态写入。因此，二进制日志事务压缩既节省了事务发起者和接收者（以及它们的备份）的存储空间，又在服务器实例之间发送事务时节省了网络带宽。

    要使`binlog_transaction_compression=ON`直接生效，必须在服务器上启用二进制日志记录。当 MySQL 服务器实例没有二进制日志时，如果它是从 MySQL 8.0.20 版本开始的，它可以接收、处理和显示压缩的事务负载，而不管其对`binlog_transaction_compression`的值如何。这些服务器实例接收的压缩事务负载以压缩状态写入中继日志，因此它们间接受益于复制拓扑中其他服务器执行的压缩。

    此系统变量在事务上下文中无法更改。设置此系统变量的会话值是受限操作。会话用户必须具有足够权限来设置受限会话变量。请参见第 7.1.9.1 节，“系统变量权限”。

    有关二进制日志事务压缩的更多信息，包括哪些事件被压缩，哪些事件不被压缩，以及在使用事务压缩时行为的变化，请参见第 7.4.4.5 节，“二进制日志事务压缩”。

    *在 NDB 8.0.31 之前*：在服务器运行时设置此变量对`NDB`表的事务日志记录没有影响。可以通过在命令行或选项文件中启动 MySQL 时使用`--binlog-transaction-compression=ON`来为`NDB`表启用二进制日志事务压缩，但不能在服务器运行时启用或禁用。

    *在 NDB 8.0.31 及更高版本中*：您可以使用`ndb_log_transaction_compression`系统变量来为`NDB`启用此功能。此外，在命令行或`my.cnf`文件中设置`--binlog-transaction-compression=ON`会导致在服务器启动时启用`ndb_log_transaction_compression`。有关该变量的详细信息，请参阅变量的描述。

+   `binlog_transaction_compression_level_zstd`

    | 命令行格式 | `--binlog-transaction-compression-level-zstd=#` |
    | --- | --- |
    | 引入 | 8.0.20 |
    | 系统变量 | `binlog_transaction_compression_level_zstd` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `3` |
    | 最小值 | `1` |
    | 最大值 | `22` |

    设置此服务器上的二进制日志事务压缩级别，该功能由`binlog_transaction_compression`系统变量启用。该值是一个整数，确定压缩工作量，从 1（最低工作量）到 22（最高工作量）。如果不指定此系统变量，则压缩级别设置为 3。

    随着压缩级别的增加，数据压缩比例增加，从而减少了事务负载所需的存储空间和网络带宽。然而，数据压缩所需的工作量也会增加，消耗源服务器上的时间、CPU 和内存资源。压缩工作量的增加与数据压缩比例的增加之间并没有线性关系。

    此系统变量不能在事务上下文中更改。设置此系统变量的会话值是一项受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    此变量对`NDB`表的事务日志记录没有影响；在 NDB Cluster 8.0.31 及更高版本中，您可以改用`ndb_log_transaction_compression_level_zstd`。

+   `binlog_transaction_dependency_tracking`

    | 命令行格式 | `--binlog-transaction-dependency-tracking=value` |
    | --- | --- |
    | 已弃用 | 8.0.35 |
    | 系统变量 | `binlog_transaction_dependency_tracking` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `COMMIT_ORDER` |
    | 有效值 | `COMMIT_ORDER``WRITESET``WRITESET_SESSION` |

    对于具有多线程副本的复制源服务器（副本上的`replica_parallel_workers`或`slave_parallel_workers`大于 0），`binlog_transaction_dependency_tracking`指定源**mysqld**生成的依赖信息的方式，该信息写入二进制日志以帮助副本确定哪些事务可以并行执行。

    复制源写入的依赖信息使用逻辑时间戳表示。（因此，设置此变量要求`replica_parallel_type`或`slave_parallel_type`已设置为`LOGICAL_CLOCK`。）每个事务有两个逻辑时间戳，如下所示：

    +   `sequence_number`：对于给定二进制日志中的第一个事务，此值为 1，对于第二个事务为 2，依此类推。每个二进制日志文件中的编号重新从 1 开始。

    +   `last_committed`：这指的是与当前事务发生冲突的最近提交事务的`sequence_number`。此值始终小于`sequence_number`。

    `binlog_transaction_dependency_tracking`控制用于计算这些逻辑时间戳的方案的选择。可用选择如下：

    +   `COMMIT_ORDER`：如果第一个事务的提交时间窗口与第二个事务的提交时间窗口重叠，则认为两个事务是独立的。这是默认值。

        提交时间窗口从事务的最后一条语句执行后立即开始，并在存储引擎提交结束后立即结束。由于事务在这两个时间点之间持有所有行锁，我们知道它们不会更新相同的行。

    +   `WRITESET`：逻辑时间戳基于`COMMIT_ORDER`与基于事务的写入集的第二方案的组合计算。事务中的每一行都向事务的写入集添加一个或多个哈希集，每个唯一键在行中都有一个。 （如果没有唯一的非空键，则使用行的哈希。）这包括已删除和已插入的行；对于更新的行，旧行和新行也包括在内。

        如果两个事务的写入集重叠，即两个事务的写入集中存在相同的某个数字（哈希），则认为这两个事务存在冲突。此外，由于写入集的计算方式，存在周期性的序列化点，使得写入集计算过程将序列化点之后的每个事务视为与序列化点之前的每个事务存在冲突。序列化点仅影响`WRITESET`算法计算的依赖关系；序列化点两侧的事务可能具有重叠的提交时间窗口，因此尽管如此，在副本上可以并行化。DDL 语句、更新具有外键的表的事务以及会话值与全局值不同的`transaction_write_set_extraction`的事务会触发序列化点。如果自上一个序列化点以来提交的事务生成了至少`binlog_transaction_dependency_history_size`个唯一哈希，则还会强制执行序列化点。

        为了使多线程复制与 NDB 集群复制一起工作（NDB 8.0.33 及更高版本支持），源端必须将此变量设置为`WRITESET`。有关更多信息，请参见第 25.7.11 节，“使用多线程应用程序的 NDB 集群复制”。

    +   `WRITESET_SESSION`: 如果以下任一语句为真，则认为两个事务存在依赖关系：

        +   事务根据`WRITESET`存在依赖关系。

        +   事务在同一用户会话中提交。

    在`WRITESET`或`WRITESET_SESSION`模式下，源端使用`COMMIT_ORDER`为具有空或部分写入集的事务、更新没有主键或唯一键的表的事务以及更新外键关系中的父表的事务生成依赖信息。

    要将`binlog_transaction_dependency_tracking`设置为`WRITESET`或`WRITESET_SESSION`，必须将`transaction_write_set_extraction`设置为`OFF`之外的值；默认值（`XXHASH64`）已足够。只要`binlog_transaction_dependency_tracking`的值为`WRITESET`或`WRITESET_SESSION`，就无法更改`transaction_write_set_extraction`。对于复制事务，任何值的更改在副本停止并使用`STOP REPLICA`和`START REPLICA`重新启动后才会生效。

    保留并检查最新事务更改给定行的行哈希数由`binlog_transaction_dependency_history_size`的值确定。

    在从中继日志应用事务时，组复制在认证后执行自己的并行化，独立于为`binlog_transaction_dependency_tracking`设置的任何值，但是该变量确实影响事务如何写入组复制成员的二进制日志。这些日志中的依赖信息用于协助从捐赠者的二进制日志进行状态传输的过程，该过程在成员加入或重新加入组时发生。对于该过程，将`binlog_transaction_dependency_tracking`设置为`WRITESET`可以根据组的工作负载提高组成员的性能。

+   `binlog_transaction_dependency_history_size`

    | 命令行格式 | `--binlog-transaction-dependency-history-size=#` |
    | --- | --- |
    | 系统变量 | `binlog_transaction_dependency_history_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `25000` |
    | 最小值 | `1` |
    | 最大值 | `1000000` |

    设置在内存中保留并用于查找最后修改给定行的事务的行哈希数的上限。一旦达到这个哈希数，历史记录将被清除。

+   `expire_logs_days`

    | 命令行格式 | `--expire-logs-days=#` |
    | --- | --- |
    | 已弃用 | 是 |
    | 系统变量 | `expire_logs_days` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `99` |
    | 单位 | 天 |

    指定自动删除二进制日志文件之前的天数。`expire_logs_days`已弃用，您应该期望在将来的版本中将其删除。相反，请使用`binlog_expire_logs_seconds`，它以秒为单位设置二进制日志过期期限。如果您没有为任何系统变量设置值，则默认的过期期限为 30 天。可能的删除发生在启动时和二进制日志刷新时。如第 7.4 节，“MySQL 服务器日志”中所示，日志刷新发生。 

    如果在启动时为 `expire_logs_days` 指定了任何非零值，并且同时指定了 `binlog_expire_logs_seconds`，则忽略该值，并使用 `binlog_expire_logs_seconds` 作为二进制日志过期期限。在这种情况下会发出警告消息。只有在未指定或指定为 0 时，启动时为 `expire_logs_days` 指定的非零值才会作为二进制日志过期期限。

    在运行时，如果另一个值已经被设置为非零值，你不能将 `binlog_expire_logs_seconds` 或 `expire_logs_days` 设置为非零值。因为 `binlog_expire_logs_seconds` 的默认值为非零，所以你必须在设置或更改 `expire_logs_days` 的值之前，显式将 `binlog_expire_logs_seconds` 设置为零。

    要禁用二进制日志的自动清理，请明确为 `binlog_expire_logs_seconds` 指定值为 0，并且不为 `expire_logs_days` 指定值。为了与早期版本兼容，如果明确为 `expire_logs_days` 指定值为 0，并且不为 `binlog_expire_logs_seconds` 指定值，则自动清理也会被禁用。在这种情况下，不会应用 `binlog_expire_logs_seconds` 的默认值。

    要手动删除二进制日志文件，请使用 `PURGE BINARY LOGS` 语句。参见 Section 15.4.1.1, “PURGE BINARY LOGS 语句”。

+   `log_bin`

    | 系统变量 | `log_bin` |
    | --- | --- |
    | 作用范围 | 全局 |
    | 是否动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |

    显示服务器上二进制日志记录的状态，可以是启用（`ON`）或禁用（`OFF`）。启用二进制日志记录后，服务器会记录所有更改数据的语句到二进制日志中，用于备份和复制。`ON`表示二进制日志可用，`OFF`表示未使用。可以使用`--log-bin`选项指定二进制日志的基本名称和位置。

    在早期的 MySQL 版本中，默认情况下禁用了二进制日志记录，只有在指定`--log-bin`选项时才启用。从 MySQL 8.0 开始，默认情况下启用了二进制日志记录，无论是否指定了`--log-bin`选项，`log_bin`系统变量都设置为`ON`。唯一的例外是，如果您使用**mysqld**手动初始化数据目录，通过使用`--initialize`或`--initialize-insecure`选项，此时默认情况下禁用了二进制日志记录。在这种情况下，可以通过指定`--log-bin`选项来启用二进制日志记录。

    如果在启动时指定了`--skip-log-bin`或`--disable-log-bin`选项，则禁用二进制日志记录，`log_bin`系统变量设置为`OFF`。如果同时指定了其中任何一个选项和`--log-bin`选项，则后面指定的选项优先。

    有关二进制日志的格式和管理信息，请参阅第 7.4.4 节，“二进制日志”。

+   `log_bin_basename`

    | 系统变量 | `log_bin_basename` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    存储二进制日志文件的基本名称和路径，可以使用`--log-bin`服务器选项进行设置。变量的最大长度为 256。在 MySQL 8.0 中，如果未提供`--log-bin`选项，则默认基本名称为`binlog`。为了与 MySQL 5.7 兼容，如果提供了`--log-bin`选项但没有字符串或为空字符串，则默认基本名称为`*`host_name`*-bin`，使用主机机器的名称。默认位置是数据目录。

+   `log_bin_index`

    | 命令行格式 | `--log-bin-index=file_name` |
    | --- | --- |
    | 系统变量 | `log_bin_index` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |

    保存二进制日志索引文件的基本名称和路径，可以使用 `--log-bin-index` 服务器选项进行设置。变量长度最大为 256。

+   `log_bin_trust_function_creators`

    | 命令行格式 | `--log-bin-trust-function-creators[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `log_bin_trust_function_creators` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量在启用二进制日志记录时应用。它控制存储函数创建者是否可信，不会创建可能导致不安全事件写入二进制日志的存储函数。如果设置为 0（默认值），除了具有 `SUPER` 权限外，用户不允许创建或更改存储函数，还必须具有 `CREATE ROUTINE` 或 `ALTER ROUTINE` 权限。设置为 0 还强制要求函数必须声明具有 `DETERMINISTIC` 特性，或具有 `READS SQL DATA` 或 `NO SQL` 特性。如果将变量设置为 1，则 MySQL 不会强制执行这些限制以创建存储函数。此变量还适用于触发器创建。请参阅 第 27.7 节，“存储程序二进制日志记录”。

+   `log_bin_use_v1_row_events`

    | 命令行格式 | `--log-bin-use-v1-row-events[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.18 |
    | 系统变量 | `log_bin_use_v1_row_events` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此只读系统变量已弃用。在服务器启动时将系统变量设置为`ON`，通过使用 Version 1 二进制日志行事件而不是 MySQL 5.6 默认的 Version 2 二进制日志行事件，启用了与运行 MySQL Server 5.5 及更早版本的副本机器进行基于行的复制。

+   `log_replica_updates`

    | 命令行格式 | `--log-replica-updates[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `log_replica_updates` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    从 MySQL 8.0.26 开始，使用`log_replica_updates` 替代从该版本开始弃用的`log_slave_updates`。在 MySQL 8.0.26 之前的版本中，请使用`log_slave_updates`。

    `log_replica_updates` 指定了复制服务器从复制源服务器接收的更新是否应记录到复制服务器自己的二进制日志中。

    启用此变量会导致复制服务器将从源接收并由复制 SQL 线程执行的更新写入复制服务器自己的二进制日志中。二进制日志记录由`--log-bin`选项控制，默认情况下启用，复制服务器上也必须启用二进制日志记录才能记录更新。请参阅第 19.1.6 节，“复制和二进制日志选项和变量”。`log_replica_updates` 默认启用，除非您指定`--skip-log-bin`来禁用二进制日志记录，此时 MySQL 也会默认禁用复制更新记录。如果需要在启用二进制日志时禁用复制更新记录，请在复制服务器启动时指定`--log-replica-updates=OFF`。

    启用`log_replica_updates` 可以实现复制服务器的链式连接。例如，您可能希望使用以下安排设置复制服务器：

    ```sql
    A -> B -> C
    ```

    在这里，`A` 作为复制服务器 `B` 的源，`B` 作为复制服务器 `C` 的源。为使此工作正常，`B` 必须同时是源和复制服务器。启用二进制日志记录和`log_replica_updates`启用，这些是默认设置，从 `A` 接收的更新由 `B` 记录到其二进制日志中，因此可以传递给 `C`。

+   `log_slave_updates`

    | 命令行格式 | `--log-slave-updates[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.26 |
    | 系统变量 | `log_slave_updates` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    从 MySQL 8.0.26 开始，`log_slave_updates`已被弃用，应改用别名`log_replica_updates`。在 MySQL 8.0.26 之前的版本中，请使用`log_slave_updates`。

    `log_slave_updates`指定是否应将复制服务器从复制源服务器接收的更新记录到副本自己的二进制日志中。

    启用此变量会导致副本将从源接收并由复制 SQL 线程执行的更新写入副本自己的二进制日志中。必须还在副本上启用二进制日志记录，该记录由`--log-bin`选项控制，并且默认情况下已启用，才能记录更新。请参阅第 19.1.6 节，“复制和二进制日志选项和变量”。默认情况下启用`log_slave_updates`，除非您指定`--skip-log-bin`来禁用二进制日志记录，此时 MySQL 也默认禁用副本更新记录。如果需要在启用二进制日志时禁用副本更新记录，请在副本服务器启动时指定`--log-slave-updates=OFF`。

    启用`log_slave_updates`可以使复制服务器链接在一起。例如，您可能希望使用以下安排设置复制服务器：

    ```sql
    A -> B -> C
    ```

    在这里，`A`作为副本`B`的源，`B`作为副本`C`的源。为了使其工作，`B`必须同时是源*和*副本。启用二进制日志记录和`log_slave_updates`启用，这些是默认设置，从`A`接收的更新由`B`记录到其二进制日志中，因此可以传递给`C`。

+   `log_statements_unsafe_for_binlog`

    | 命令行格式 | `--log-statements-unsafe-for-binlog[={OFF&#124;ON}]` |
    | --- | --- |
    | 弃用 | 8.0.34 |
    | 系统变量 | `log_statements_unsafe_for_binlog` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    如果遇到错误 1592，控制生成的警告是否添加到错误日志中。

+   `master_verify_checksum`

    | 命令行格式 | `--master-verify-checksum[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `master_verify_checksum` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 版本开始，`master_verify_checksum` 已被弃用，应改用别名 `source_verify_checksum`。在 MySQL 8.0.26 版本之前的发布版本中，请使用 `master_verify_checksum`。

    启用 `master_verify_checksum` 会导致源通过检查校验和来验证从二进制日志中读取的事件，并在不匹配时停止并显示错误。`master_verify_checksum` 默认情况下是禁用的；在这种情况下，源使用二进制日志中的事件长度来验证事件，因此只有完整的事件才会从二进制日志中读取。

+   `max_binlog_cache_size`

    | 命令行格式 | `--max-binlog-cache-size=#` |
    | --- | --- |
    | 系统变量 | `max_binlog_cache_size` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（64 位平台） | `18446744073709547520` |
    | 默认值（32 位平台） | `4294967295` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709547520` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    如果一个事务需要的字节数超过这个值，服务器会生成一个“多语句事务需要超过 'max_binlog_cache_size' 字节的存储空间”错误。当 `gtid_mode` 不是 `ON` 时，最大推荐值为 4GB，因为在这种情况下，MySQL 无法处理大于 4GB 的二进制日志位置；当 `gtid_mode` 是 `ON` 时，这个限制不适用，服务器可以处理任意大小的二进制日志位置。

    如果由于 `gtid_mode` 不是 `ON`，或出于其他原因，您需要保证二进制日志不超过给定大小 *`maxsize`*，则应根据此处显示的公式设置此变量：

    ```sql
    max_binlog_cache_size < 
      (((*maxsize* - max_binlog_size) / max_connections) - 1000) / 1.2
    ```

    这个计算考虑了以下条件：

    +   只要在开始写入之前的大小小于`max_binlog_size`，服务器就会写入二进制日志。

    +   服务器不会写入单个事务，而是一组事务。一组中可能的最大事务数等于`max_connections`。

    +   服务器会写入未包含在缓存中的数据。这包括每个事件的 4 字节校验和；虽然这只会增加不到事务大小的 20%，但这个量是不可忽略的。此外，服务器会为每个事务写入一个`Gtid_log_event`；每个事件可能会使写入二进制日志的内容增加另外 1 KB。

    `max_binlog_cache_size`仅设置事务缓存的大小；语句缓存的上限由`max_binlog_stmt_cache_size`系统变量控制。

    `max_binlog_cache_size`对会话的可见性与`binlog_cache_size`系统变量的可见性相匹配；换句话说，改变其值只会影响在值更改后启动的新会话。

+   `max_binlog_size`

    | 命令行格式 | `--max-binlog-size=#` |
    | --- | --- |
    | 系统变量 | `max_binlog_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1073741824` |
    | 最小值 | `4096` |
    | 最大值 | `1073741824` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    如果写入二进制日志导致当前日志文件大小超过此变量的值，则服务器会旋转二进制日志（关闭当前文件并打开下一个文件）。最小值为 4096 字节。最大和默认值为 1GB。加密的二进制日志文件有额外的 512 字节头部，包含在`max_binlog_size`中。

    事务被一次性写入二进制日志，因此永远不会在多个二进制日志之间分割。因此，如果你有大事务，你可能会看到比`max_binlog_size`更大的二进制日志文件。

    如果`max_relay_log_size`为 0，则`max_binlog_size`的值也适用于中继日志。

    在服务器上使用 GTIDs 时，当达到`max_binlog_size`时，如果无法访问系统表`mysql.gtid_executed`以写入当前二进制日志文件中的 GTIDs，则无法旋转二进制日志。在这种情况下，服务器根据其`binlog_error_action`设置做出响应。如果设置为`IGNORE_ERROR`，服务器上会记录错误并停止二进制日志记录，或者如果设置为`ABORT_SERVER`，服务器将关闭。

+   `max_binlog_stmt_cache_size`

    | 命令行格式 | `--max-binlog-stmt-cache-size=#` |
    | --- | --- |
    | 系统变量 | `max_binlog_stmt_cache_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `18446744073709547520` |
    | 最小值 | `4096` |
    | 最大值 | `18446744073709547520` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    如果事务中的非事务性语句需要的内存超过这么多字节，服务器将生成错误。最小值为 4096。32 位平台上的最大和默认值为 4GB，在 64 位平台上为 16EB（艾字节）。

    `max_binlog_stmt_cache_size`仅设置语句缓存的大小；事务缓存的上限完全由`max_binlog_cache_size`系统变量控制。

+   `original_commit_timestamp`

    | 系统变量 | `original_commit_timestamp` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 数值 |

    用于复制的内部使用。在副本上重新执行事务时，此值设置为事务在原始源上提交时的时间，以自纪元以来的微秒为单位。这允许原始提交时间戳在整个复制拓扑中传播。

    设置此系统变量的会话值是受限操作。会话用户必须具有`REPLICATION_APPLIER`权限（参见第 19.3.3 节，“复制权限检查”），或具有足够权限设置受限会话变量（参见第 7.1.9.1 节，“系统变量权限”）。但是，请注意，该变量不是供用户设置的；它是由复制基础设施自动设置的。

+   `source_verify_checksum`

    | 命令行格式 | `--source-verify-checksum[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `source_verify_checksum` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，使用`source_verify_checksum`代替从该版本开始已弃用的`master_verify_checksum`。在 MySQL 8.0.26 之前的版本中，请使用`master_verify_checksum`。

    启用`source_verify_checksum`会导致源通过检查校验和来验证从二进制日志中读取的事件，并在不匹配时停止并显示错误。`source_verify_checksum`默认情况下是禁用的；在这种情况下，源使用二进制日志中的事件长度来验证事件，因此只有完整的事件才会从二进制日志中读取。

+   `sql_log_bin`

    | 系统变量 | `sql_log_bin` |
    | --- | --- |
    | 作用域 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量控制当前会话是否启用对二进制日志的记录（假设二进制日志本身已启用）。默认值为`ON`。要为当前会话禁用或启用二进制日志记录，请将会话`sql_log_bin`变量设置为`OFF`或`ON`。

    将此变量设置为`OFF`，以便在对源进行更改时暂时禁用二进制日志记录，这些更改不希望被复制到副本中。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有足够权限来设置受限制的会话变量。请参阅第 7.1.9.1 节，“系统变量权限”。

    不可能在事务或子查询中设置`sql_log_bin`的会话值。

    *将此变量设置为 `OFF` 可防止将 GTID 分配给二进制日志中的事务。* 如果您正在使用 GTID 进行复制，这意味着即使稍后重新启用二进制日志记录，从此时开始写入日志的 GTID 也不会考虑期间发生的任何事务，因此实际上这些事务会丢失。

+   `sync_binlog`

    | 命令行格式 | `--sync-binlog=#` |
    | --- | --- |
    | 系统变量 | `sync_binlog` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    控制 MySQL 服务器将二进制日志同步到磁盘的频率。

    +   `sync_binlog=0`：禁用 MySQL 服务器将二进制日志同步到磁盘。相反，MySQL 服务器依赖操作系统定期刷新二进制日志到磁盘，就像对任何其他文件一样。这个设置提供了最佳性能，但在断电或操作系统崩溃的情况下，可能会出现服务器已提交但尚未同步到二进制日志的事务。

    +   `sync_binlog=1`：在提交事务之前将二进制日志同步到磁盘。这是最安全的设置，但由于增加了磁盘写入次数，可能会对性能产生负面影响。在断电或操作系统崩溃的情况下，二进制日志中缺失的事务仅处于准备状态。这允许自动恢复程序回滚事务，确保没有事务丢失在二进制日志中。

    +   `sync_binlog=*`N`*`，其中 *`N`* 是除 0 或 1 之外的值：在收集了 `N` 个二进制日志提交组后，将二进制日志同步到磁盘。在断电或操作系统崩溃的情况下，可能会出现服务器已提交但尚未刷新到二进制日志的事务。这个设置可能会对性能产生负面影响，因为增加了磁盘写入次数。较高的值可以提高性能，但增加了数据丢失的风险。

    对于使用`InnoDB`和事务的复制设置，为了获得最大的耐久性和一致性，请使用以下设置：

    +   `sync_binlog=1`。

    +   `innodb_flush_log_at_trx_commit=1`。

    注意

    许多操作系统和一些磁盘硬件会欺骗刷新到磁盘的操作。它们可能告诉**mysqld**刷新已经完成，即使实际上并没有。在这种情况下，即使使用推荐的设置，事务的耐久性也无法得到保证，最坏的情况下，停电可能会损坏`InnoDB`数据。在 SCSI 磁盘控制器或磁盘本身中使用带电池后备的磁盘缓存可以加快文件刷新速度，并使操作更安全。您还可以尝试禁用硬件缓存中的磁盘写入缓存。

+   `transaction_write_set_extraction`

    | 命令行格式 | `--transaction-write-set-extraction[=value]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 系统变量 | `transaction_write_set_extraction` |
    | 作用域 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `XXHASH64` |
    | 有效值 | `OFF``MURMUR32``XXHASH64` |

    此系统变量指定在事务期间提取的写入哈希的算法。默认值为`XXHASH64`。`OFF`表示不收集写入集。

    `transaction_write_set_extraction`在 MySQL 8.0.26 中已弃用；预计将在未来的 MySQL 版本中删除。

    `XXHASH64`设置是 Group Replication 所必需的，用于从事务中提取写入以用于所有组成员的冲突检测和认证过程（参见 Section 20.3.1, “Group Replication Requirements”）。对于具有多线程副本的复制源服务器（在这些副本上，`replica_parallel_workers`或`slave_parallel_workers`设置为大于 0 的值），其中`binlog_transaction_dependency_tracking`设置为`WRITESET`或`WRITESET_SESSION`，`transaction_write_set_extraction`不能为`OFF`。当前`binlog_transaction_dependency_tracking`的值为`WRITESET`或`WRITESET_SESSION`时，不能更改`transaction_write_set_extraction`的值。

    截至 MySQL 8.0.14 版本，设置此系统变量的会话值是一项受限制的操作；会话用户必须具有足够的特权来设置受限制的会话变量（参见第 7.1.9.1 节，“系统变量特权”）。`binlog_format`必须设置为`ROW`才能更改`transaction_write_set_extraction`的值。如果更改了该值，则新值在复制事务上不会立即生效，直到副本使用`STOP REPLICA`和`START REPLICA`停止并重新启动后才会生效。
