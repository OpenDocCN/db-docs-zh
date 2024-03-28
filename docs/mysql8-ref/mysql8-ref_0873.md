# 14.21 Performance Schema Functions

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html)

截至 MySQL 8.0.16，MySQL 包含内置的 SQL 函数，用于格式化或检索性能模式数据，并可用作对应的`sys`模式存储函数的等效函数。内置函数可以在任何模式中调用，无需限定符，不像`sys`函数，后者要求使用`sys.`模式限定符或`sys`为当前模式。

**表 14.31 Performance Schema Functions**

| 名称 | 描述 | 引入版本 |
| --- | --- | --- |
| [`FORMAT_BYTES()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_format-bytes) | 将字节计数转换为带单位的值 | 8.0.16 |
| [`FORMAT_PICO_TIME()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_format-pico-time) | 将皮秒时间转换为带单位的值 | 8.0.16 |
| [`PS_CURRENT_THREAD_ID()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_ps-current-thread-id) | 当前线程的性能模式线程 ID | 8.0.16 |
| [`PS_THREAD_ID()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_ps-thread-id) | 给定线程的性能模式线程 ID | 8.0.16 |

内置函数取代了相应的`sys`函数，后者已被��用；预计它们将在未来的 MySQL 版本中被移除。使用`sys`函数的应用程序应调整为使用内置函数，需要注意`sys`函数与内置函数之间的一些细微差异。有关这些差异的详细信息，请参阅本节中的函数描述。 

+   [`FORMAT_BYTES(*`count`*)`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_format-bytes)

    给定一个数字字节计数，将其转换为人类可读格式，并返回一个由值和单位指示器组成的字符串。该字符串包含四舍五入到 2 位小数和至少 3 个有效数字的字节数。小于 1024 字节的数字表示为整数，不进行四舍五入。如果*`count`*为`NULL`，则返回`NULL`。

    单位指示器取决于字节计数参数的大小，如下表所示。

    | 参数值 | 结果单位 | 结果单位指示器 |
    | --- | --- | --- |
    | 最多 1023 | 字节 | 字节 |
    | 最多 1024² − 1 | kibibytes | KiB |
    | 最多 1024³ − 1 | mebibytes | MiB |
    | 最多 1024⁴ − 1 | gibibytes | GiB |
    | 最多 1024⁵ − 1 | tebibytes | TiB |
    | 最多 1024⁶ − 1 | pebibytes | PiB |
    | 1024⁶及以上 | exbibytes | EiB |

    ```sql
    mysql> SELECT FORMAT_BYTES(512), FORMAT_BYTES(18446644073709551615);
    +-------------------+------------------------------------+
    | FORMAT_BYTES(512) | FORMAT_BYTES(18446644073709551615) |
    +-------------------+------------------------------------+
    |  512 bytes        | 16.00 EiB                          |
    +-------------------+------------------------------------+
    ```

    [`FORMAT_BYTES()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_format-bytes) 在 MySQL 8.0.16 中添加。它可以用来替代`sys`模式中的`format_bytes()` Function")函数，需要注意以下区别：

    +   [`FORMAT_BYTES()`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-functions.html#function_format-bytes) 使用`EiB`单位指示器。`sys.format_bytes()` Function")则不使用。

+   `FORMAT_PICO_TIME(*`time_val`*)`

    给定一个数值型 Performance Schema 潜伏时间或等待时间（以皮秒为单位），将其转换为人类可读格式，并返回一个由值和单位指示符组成的字符串。字符串包含四舍五入到 2 位小数的十进制时间和至少 3 个有效数字。小于 1 纳秒的时间表示为整数，不进行四舍五入。

    如果 *`time_val`* 为 `NULL`，此函数返回 `NULL`。

    单位指示符取决于时间值参数的大小，如下表所示。

    | 参数值 | 结果单位 | 结果单位指示符 |
    | --- | --- | --- |
    | 最大为 10³ − 1 | 皮秒 | ps |
    | 最大为 10⁶ − 1 | 纳秒 | ns |
    | 最大为 10⁹ − 1 | 微秒 | us |
    | 最大为 10¹² − 1 | 毫秒 | ms |
    | 最大为 60×10¹² − 1 | 秒 | s |
    | 最大为 3.6×10¹⁵ − 1 | 分钟 | min |
    | 最大为 8.64×10¹⁶ − 1 | 小时 | h |
    | 大于等于 8.64×10¹⁶ | 天 | d |

    ```sql
    mysql> SELECT FORMAT_PICO_TIME(3501), FORMAT_PICO_TIME(188732396662000);
    +------------------------+-----------------------------------+
    | FORMAT_PICO_TIME(3501) | FORMAT_PICO_TIME(188732396662000) |
    +------------------------+-----------------------------------+
    | 3.50 ns                | 3.15 min                          |
    +------------------------+-----------------------------------+
    ```

    `FORMAT_PICO_TIME()` 在 MySQL 8.0.16 版本中添加。可以用来替代 `sys` 模式中的 `format_time()` Function") 函数，需要注意以下区别：

    +   为了表示分钟，`sys.format_time()` Function") 使用 `m` 单位指示符，而 `FORMAT_PICO_TIME()` 使用 `min`。

    +   `sys.format_time()` Function") 使用 `w`（周）单位指示符。`FORMAT_PICO_TIME()` 不使用。

+   `PS_CURRENT_THREAD_ID()`

    返回一个表示当前连接分配的 Performance Schema 线程 ID 的 `BIGINT UNSIGNED` 值。

    线程 ID 返回值是 Performance Schema 表中 `THREAD_ID` 列中给定类型的值。

    Performance Schema 配置对 `PS_CURRENT_THREAD_ID()` 的影响与对 `PS_THREAD_ID()` 的影响相同。详情请参阅该函数的描述。

    ```sql
    mysql> SELECT PS_CURRENT_THREAD_ID();
    +------------------------+
    | PS_CURRENT_THREAD_ID() |
    +------------------------+
    |                     52 |
    +------------------------+
    mysql> SELECT PS_THREAD_ID(CONNECTION_ID());
    +-------------------------------+
    | PS_THREAD_ID(CONNECTION_ID()) |
    +-------------------------------+
    |                            52 |
    +-------------------------------+
    ```

    `PS_CURRENT_THREAD_ID()` 在 MySQL 8.0.16 版本中添加。可以用作调用 `sys` 模式中的 `ps_thread_id()` Function") 函数的快捷方式，参数为 `NULL` 或 `CONNECTION_ID()`。

+   `PS_THREAD_ID(*`connection_id`*)`

    给定连接 ID，返回一个表示分配给连接 ID 的性能模式线程 ID 的`BIGINT UNSIGNED`值，如果连接 ID 没有线程 ID 存在，则返回`NULL`。后者可能发生在未被检测的线程上，或者如果*`connection_id`*为`NULL`。

    连接 ID 参数是性能模式`threads`表中`PROCESSLIST_ID`列或`SHOW PROCESSLIST`输出中的`Id`列的值。

    线程 ID 返回值是性能模式表中`THREAD_ID`列中给定类型的值。

    性能模式配置会影响`PS_THREAD_ID()`的操作。 (这些备注也适用于`PS_CURRENT_THREAD_ID()`.)

    +   禁用`thread_instrumentation`消费者会导致无法在线程级别收集和聚合统计数据，但不会影响`PS_THREAD_ID()`。

    +   如果`performance_schema_max_thread_instances`不为 0，则性能模式为线程统计数据分配内存，并为每个可用实例内存的线程分配一个内部 ID。如果有线程没有可用实例内存，`PS_THREAD_ID()`返回`NULL`；在这种情况下，`Performance_schema_thread_instances_lost`不为零。

    +   如果`performance_schema_max_thread_instances`为 0，则性能模式不分配线程内存，`PS_THREAD_ID()`返回`NULL`。

    +   如果性能模式本身被禁用，`PS_THREAD_ID()`会产生错误。

    ```sql
    mysql> SELECT PS_THREAD_ID(6);
    +-----------------+
    | PS_THREAD_ID(6) |
    +-----------------+
    |              45 |
    +-----------------+
    ```

    `PS_THREAD_ID()`在 MySQL 8.0.16 中添加。它可以代替`sys`模式的`ps_thread_id()` Function")函数，但要注意以下差异：

    +   使用`NULL`作为参数，`sys.ps_thread_id()` Function")函数返回当前连接的线程 ID，而`PS_THREAD_ID()`返回`NULL`。要获取当前连接的线程 ID，请使用`PS_CURRENT_THREAD_ID()`。
