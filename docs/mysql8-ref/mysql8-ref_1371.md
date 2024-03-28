> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-options-gtids.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-gtids.html)

#### 19.1.6.5 全局事务 ID 系统变量

本节描述的 MySQL 服务器系统变量用于监视和控制全局事务标识符（GTIDs）。有关更多信息，请参见第 19.1.3 节，“具有全局事务标识符的复制”。

+   `binlog_gtid_simple_recovery`

    | 命令行格式 | `--binlog-gtid-simple-recovery[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `binlog_gtid_simple_recovery` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示��用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    此变量控制 MySQL 启动或重新启动时在搜索 GTID 时如何迭代二进制日志文件。

    当`binlog_gtid_simple_recovery=TRUE`（这是 MySQL 8.0 中的默认值）时，`gtid_executed`和`gtid_purged`的值在启动时基于最新和最旧的二进制日志文件中的`Previous_gtids_log_event`的值进行计算。有关计算的描述，请参见 gtid_purged 系统变量。此设置在服务器重新启动时仅访问两个二进制日志文件。如果服务器上的所有二进制日志都是使用 MySQL 5.7.8 或更高版本生成的，则始终可以安全地使用`binlog_gtid_simple_recovery=TRUE`。

    如果服务器上存在来自 MySQL 5.7.7 或更旧版本的任何二进制日志（例如，在将较旧服务器升级到 MySQL 8.0 后），并且使用`binlog_gtid_simple_recovery=TRUE`，`gtid_executed`和`gtid_purged` 可能会在以下两种情况下被错误初始化：

    +   最新的二进制日志是由 MySQL 5.7.5 或更早版本生成的，并且对于某些二进制日志，`gtid_mode`为`ON`，但对于最新的二进制日志为`OFF`。

    +   在 MySQL 5.7.7 或更早版本上发出了`SET @@GLOBAL.gtid_purged`语句，并且在`SET @@GLOBAL.gtid_purged`语句时处于活动状态的二进制日志尚未被清除。

    如果在任何情况下计算出了不正确的 GTID 集合，则即使稍后使用`binlog_gtid_simple_recovery=FALSE`重新启动服务器，它仍然是不正确的。如果服务器存在任何这些情况或可能存在这些情况，请在启动或重新启动服务器之前设置`binlog_gtid_simple_recovery=FALSE`。

    当设置`binlog_gtid_simple_recovery=FALSE`时，计算`gtid_executed`和`gtid_purged`的方法如 gtid_purged 系统变量中所述，将更改为按以下方式迭代二进制日志文件：

    +   不再使用最新的二进制日志文件中的`Previous_gtids_log_event`和 GTID 日志事件的值，而是从最新的二进制日志文件开始计算`gtid_executed`，并使用从第一个二进制日志文件中找到的`Previous_gtids_log_event`值。如果服务器的最近的二进制日志文件没有 GTID 日志事件，例如如果服务器最初使用了`gtid_mode=ON`，但稍后将服务器更改为`gtid_mode=OFF`，这个过程可能需要很长时间。

    +   不再使用最旧的二进制日志文件中的`Previous_gtids_log_event`的值，而是从最旧的二进制日志文件开始计算`gtid_purged`，并使用从第一个二进制日志文件中找到的非空`Previous_gtids_log_event`值或至少一个 GTID 日志事件的值（表示从那一点开始使用 GTID）。如果服务器的旧二进制日志文件没有 GTID 日志事件，例如如果服务器最近才设置了`gtid_mode=ON`，这个过程可能需要很长时间。

+   `enforce_gtid_consistency`

    | 命令行格式 | `--enforce-gtid-consistency[=value]` |
    | --- | --- |
    | 系统变量 | `enforce_gtid_consistency` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``ON``WARN` |

    根据此变量的值，服务器通过仅允许执行可以安全使用 GTID 记录的语句来强制执行 GTID 一致性。在启用基于 GTID 的复制之前，*必须*将此变量设置为 `ON`。

    `enforce_gtid_consistency` 可配置的值为：

    +   `OFF`：允许所有事务违反 GTID 一致性。

    +   `ON`：不允许任何事务违反 GTID 一致性。

    +   `WARN`：允许所有事务违反 GTID 一致性，但在这种情况下会生成警告。

    `--enforce-gtid-consistency` 仅在对语句进行二进制日志记录时生效。如果服务器上禁用了二进制日志记录，或者如果语句未写入二进制日志因为它们被过滤器移除，那么对于未记录的语句，GTID 一致性不会被检查或强制执行。

    当 `enforce_gtid_consistency` 设置为 `ON` 时，只有可以使用 GTID 安全语句记录的语句才能被记录，因此以下列出的操作不能与此选项一起使用：

    +   在事务中的 `CREATE TEMPORARY TABLE` 或 `DROP TEMPORARY TABLE` 语句。

    +   更新既包含事务表又包含非事务表的事务或语句。有一个例外，即如果所有*非事务*表都是临时表，则允许在同一事务或同一语句中使用非事务 DML。

    +   `CREATE TABLE ... SELECT` 语句，在 MySQL 8.0.21 之前。从 MySQL 8.0.21 开始，支持原子 DDL 的存储引擎允许 `CREATE TABLE ... SELECT` 语句。

    更多信息，请参阅 Section 19.1.3.7, “使用 GTID 限制复制”。

    在 MySQL 5.7 之前以及在该版本系列的早期版本中，布尔值`enforce_gtid_consistency`默认为`OFF`。为了与这些早期版本保持兼容性，枚举默认为`OFF`，并且设置`--enforce-gtid-consistency`而不带值被解释为将值设置为`ON`。该变量还具有多个文本别名用于表示值：`0=OFF=FALSE`，`1=ON=TRUE`，`2=WARN`。这与其他枚举类型不同，但保持了与先前版本中使用的布尔类型的兼容性。这些更改影响了变量返回的内容。使用`SELECT @@ENFORCE_GTID_CONSISTENCY`，`SHOW VARIABLES LIKE 'ENFORCE_GTID_CONSISTENCY'`和`SELECT * FROM INFORMATION_SCHEMA.VARIABLES WHERE 'VARIABLE_NAME' = 'ENFORCE_GTID_CONSISTENCY'`，都返回文本形式，而不是数字形式。这是一个不兼容的更改，因为`@@ENFORCE_GTID_CONSISTENCY`对于布尔值返回数字形式，但对于`SHOW`和信息模式返回文本形式。

+   `gtid_executed`

    | 系统变量 | `gtid_executed` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 单位 | GTID 集合 |

    当与全局范围一起使用时，此变量包含服务器上执行的所有事务和由`SET` `gtid_purged`语句设置的 GTIDs 的表示。这与`SHOW MASTER STATUS`和`SHOW REPLICA STATUS`输出中的`Executed_Gtid_Set`列的值相同。此变量的值是一个 GTID 集合，请参阅 GTID Sets 获取更多信息。

    当服务器启动时，`@@GLOBAL.gtid_executed`被初始化。有关如何迭代二进制日志以填充`gtid_executed`的更多信息，请参阅`binlog_gtid_simple_recovery`。随着事务的执行或执行任何`SET` `gtid_purged`语句，GTIDs 将被添加到集合中。

    在任何给定时间可以在二进制日志中找到的事务集合等于 `GTID_SUBTRACT(@@GLOBAL.gtid_executed, @@GLOBAL.gtid_purged)`；也就是说，所有尚未清除的二进制日志中的事务。

    执行 `RESET MASTER` 会导致这个变量的全局值（但不包括会话值）被重置为空字符串。除非由于 `RESET MASTER` 而清除了该集合，否则 GTID 不会从该集合中删除。

+   `gtid_executed_compression_period`

    | 命令行格式 | `--gtid-executed-compression-period=#` |
    | --- | --- |
    | 系统变量 | `gtid_executed_compression_period` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值（≥ 8.0.23） | `0` |
    | 默认值（≤ 8.0.22） | `1000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    每处理这么多事务时，压缩 `mysql.gtid_executed` 表。当服务器启用二进制日志记录时，不会使用这种压缩方法，而是在每次二进制日志轮换时压缩 `mysql.gtid_executed` 表。当服务器禁用二进制日志记录时，压缩线程会休眠，直到执行了指定数量的事务，然后唤醒以执行 `mysql.gtid_executed` 表的压缩。将此系统变量的值设置为 0 表示线程永远不会唤醒，因此不会使用这种显式压缩方法。相反，压缩会根据需要隐式发生。

    从 MySQL 8.0.17 开始，`InnoDB` 事务由一个单独的进程写入到 `mysql.gtid_executed` 表中，而非 `InnoDB` 事务。如果服务器同时存在 `InnoDB` 事务和非 `InnoDB` 事务，那么由这个系统变量控制的压缩会干扰这个进程的工作，并且可能显著减慢速度。因此，从那个版本开始，建议将 `gtid_executed_compression_period` 设置为 0。

    从 MySQL 8.0.23 开始，`InnoDB` 和非 `InnoDB` 事务由同一进程写入到 `mysql.gtid_executed` 表中，并且 `gtid_executed_compression_period` 的默认值为 0。

    更多信息请参见 mysql.gtid_executed 表压缩。

+   `gtid_mode`

    | 命令行格式 | `--gtid-mode=MODE` |
    | --- | --- |
    | 系统变量 | `gtid_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `OFF` |
    | 有效值 | `OFF``OFF_PERMISSIVE``ON_PERMISSIVE``ON` |

    控制 GTID 基础日志记录是否启用以及日志可以包含什么类型的事务。您必须具有足够的权限来设置全局系统变量。请参阅第 7.1.9.1 节，“系统变量权限”。在设置`gtid_mode=ON`之前，必须将`enforce_gtid_consistency`设置为`ON`。在修改此变量之前，请参阅第 19.1.4 节，“在线服务器上更改 GTID 模式”。

    记录的事务可以是匿名的或者使用 GTIDs。匿名事务依赖于二进制日志文件和位置来识别特定事务。GTID 事务具有用于引用事务的唯一标识符。不同的模式包括：

    +   `OFF`: 新事务和复制事务都必须是匿名的。

    +   `OFF_PERMISSIVE`: 新事务是匿名的。复制事务可以是匿名的或者 GTID 事务。

    +   `ON_PERMISSIVE`: 新事务是 GTID 事务。复制事务可以是匿名的或���GTID 事务。

    +   `ON`: 新事务和复制事务都必须是 GTID 事务。

    从一个值更改为另一个值只能一次一步。例如，如果`gtid_mode`当前设置为`OFF_PERMISSIVE`，则可以更改为`OFF`或`ON_PERMISSIVE`，但不能更改为`ON`。

    `gtid_purged`和`gtid_executed`的值是持久的，无论`gtid_mode`的值如何。因此，即使更改了`gtid_mode`的值，这些变量仍包含正确的值。

+   `gtid_next`

    | 系统变量 | `gtid_next` |
    | --- | --- |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `AUTOMATIC` |
    | 有效值 | `AUTOMATIC``ANONYMOUS``<UUID>:<NUMBER>` |

    此变量用于指定下一个 GTID 是否以及如何获取。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有`REPLICATION_APPLIER`权限（参见 Section 19.3.3, “Replication Privilege Checks”），或具有足够权限设置受限制的会话变量（参见 Section 7.1.9.1, “System Variable Privileges”）。

    `gtid_next` 可以取以下任一值：

    +   `AUTOMATIC`: 使用下一个自动生成的全局事务 ID。

    +   `ANONYMOUS`: 事务没有全局标识符，仅通过文件和位置进行标识。

    +   以 *`UUID`*:*`NUMBER`* 格式的全局事务 ID。

    以上哪些选项有效取决于`gtid_mode`的设置，更多信息请参见 Section 19.1.4.1, “Replication Mode Concepts”。如果`gtid_mode`为`OFF`，设置此变量不会产生任何效果。

    在将此变量设置为 *`UUID`*:*`NUMBER`* 后，事务已经提交或回滚之后，必须再次发出明确的 `SET GTID_NEXT` 语句，然后才能执行其他语句。

    `DROP TABLE` 或 `DROP TEMPORARY TABLE` 在将非临时表与临时表组合使用，或者使用事务存储引擎的临时表与使用非事务存储引擎的临时表时会失败，并显示明确的错误。

+   `gtid_owned`

    | 系统变量 | `gtid_owned` |
    | --- | --- |
    | 范围 | 全局，会话 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 单元 | GTID 集合 |

    这个只读变量主要用于内部使用。其内容取决于其范围。

    +   当在全局范围内使用时，`gtid_owned` 保存着服务器当前正在使用的所有 GTID 的列表，以及拥有它们的线程的 ID。这个变量主要用于多线程复制，以检查事务是否已经在另一个线程上应用。应用程序线程在处理事务时始终拥有事务的 GTID，因此`@@global.gtid_owned`在处理过程中显示 GTID 和所有者。当事务已经提交（或回滚）时，应用程序线程释放 GTID 的所有权。

    +   当与会话范围一起使用时，`gtid_owned`保存当前由此会话使用并拥有的单个 GTID。此变量主要用于测试和调试在客户端通过设置`gtid_next`显式分配事务的 GTID 时 GTID 的使用情况。在这种情况下，`@@session.gtid_owned`在客户端处理事务时始终显示 GTID，直到事务已提交（或回滚）。当客户端完成处理事务时，变量将被清除。如果会话使用`gtid_next=AUTOMATIC`，则`gtid_owned`仅在事务的提交语句执行期间短暂填充，因此无法从相关会话中观察到，尽管在正确时刻读取`@@global.gtid_owned`时会列出。如果您有跟踪客户端在会话中处理的 GTID 的要求，可以启用由`session_track_gtids`系统变量控制的会话状态跟踪器。

+   `gtid_purged`

    | 系统变量 | `gtid_purged` |
    | --- | --- |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 单位 | GTID 集合 |

    全局值`gtid_purged`系统变量（`@@GLOBAL.gtid_purged`）是一个 GTID 集，包含在服务器上已提交但不在服务器上任何二进制日志文件中存在的所有事务的 GTID。`gtid_purged`是`gtid_executed`的子集。`gtid_purged`中包含以下类别的 GTID：

    +   已在副本上以二进制日志记录禁用的事务的复制 GTID。

    +   现在已清除的二进制日志文件中写入的事务的 GTID。

    +   明确通过语句`SET @@GLOBAL.gtid_purged`添加到集合中的 GTID。

    当服务器启动时，全局的`gtid_purged`的值会被初始化为一组 GTID。有关如何计算此 GTID 集合的信息，请参阅 gtid_purged 系统变量。如果服务器上存在来自 MySQL 5.7.7 或更早版本的二进制日志，则可能需要在服务器的配置文件中设置`binlog_gtid_simple_recovery=FALSE`以生成正确的计算。有关需要此设置的情况的详细信息，请参阅`binlog_gtid_simple_recovery`的描述。

    执行`RESET MASTER`会导致`gtid_purged`的值被重置为空字符串。

    您可以设置`gtid_purged`的值，以记录服务器上已应用某个 GTID 集合中的事务，尽管这些事务在服务器上的任何二进制日志中都不存在。此操作的一个示例用例是在服务器上恢复一个或多个数据库的备份时，但您没有包含这些事务的相关二进制日志。

    重要提示

    GTID 仅在服务器实例上可用，直到有符号 64 位整数的非负值的数量（2 的 63 次方减 1）。如果您将`gtid_purged`的值设置为接近此限制的数字，后续的提交可能会导致服务器耗尽 GTID 并执行`binlog_error_action`指定的操作。从 MySQL 8.0.23 开始，当服务器实例接近限制时会发出警告消息。

    从 MySQL 8.0 开始，有两种方法可以设置`gtid_purged`的值。您可以用您指定的 GTID 集替换`gtid_purged`的值，或者将您指定的 GTID 集附加到已由`gtid_purged`持有的 GTID 集中。如果服务器没有现有的 GTIDs，例如您正在使用现有数据库的备份进行配置的空服务器，两种方法都会产生相同的结果。如果您正在恢复与服务器上已有的事务重叠的备份，例如用**mysqldump**（即使备份是部分的，也包括服务器上所有事务的 GTIDs）从源创建的部分转储替换受损表，使用第一种方法替换`gtid_purged`的值。如果您正在恢复与服务器上已有的事务不重叠的备份，例如使用来自两个不同服务器的转储进行多源复制的配置，使用第二种方法添加到`gtid_purged`的值。

    +   要用您指定的 GTID 集替换`gtid_purged`的值，请使用以下语句：

        ```sql
        SET @@GLOBAL.gtid_purged = 'gtid_set'
        ```

        `gtid_set`必须是当前`gtid_purged`值的超集，并且不能与`gtid_subtract(gtid_executed,gtid_purged)`相交。换句话说，新的 GTID 集合**必须**包括已经在`gtid_purged`中的任何 GTIDs，并且**不能**包括尚未被清除的`gtid_executed`中的任何 GTIDs。`gtid_set`也不能包括任何在`@@global.gtid_owned`中的 GTIDs，即当前正在服务器上处理的事务的 GTIDs。

        结果是全局值`gtid_purged`被设置为`gtid_set`，而`gtid_executed`的值变为`gtid_set`和先前`gtid_executed`的值的并集。

    +   要将您指定的 GTID 集附加到`gtid_purged`，请使用以下带有加号(+)的语句：

        ```sql
        SET @@GLOBAL.gtid_purged = '+gtid_set'
        ```

        `gtid_set` **绝对不能**与当前值`gtid_executed`相交。换句话说，新的 GTID 集不能包含`gtid_executed`中的任何 GTID，包括已经在`gtid_purged`中的事务。`gtid_set`也不能包含任何在`@@global.gtid_owned`中的 GTID，即当前正在服务器上处理的事务的 GTID。

        结果是`gtid_set`被添加到`gtid_executed`和`gtid_purged`中。

注意

如果服务器上存在来自 MySQL 5.7.7 或更早版本的任何二进制日志（例如，在将旧服务器升级到 MySQL 8.0 后），在发出`SET @@GLOBAL.gtid_purged`语句后，您可能需要在重新启动服务器之前在服务器的配置文件中设置`binlog_gtid_simple_recovery=FALSE`，否则`gtid_purged`可能会计算错误。有关需要此设置的情况的详细信息，请参阅`binlog_gtid_simple_recovery`的描述。
