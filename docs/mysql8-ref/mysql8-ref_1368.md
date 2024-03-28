> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-options-source.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-source.html)

#### 19.1.6.2 复制源选项和变量

本节描述了您可以在复制源服务器上使用的服务器选项和系统变量。您可以在命令行或选项文件中指定选项。您可以使用 `SET` 指定系统变量值。

在源和每个副本上，您必须设置 `server_id` 系统变量以建立唯一的复制 ID。对于每个服务器，您应该选择一个在 1 到 2³² − 1 范围内的唯一正整数，并且每个 ID 必须与复制拓扑中任何其他源或副本使用的任何其他 ID 不同。例如：`server-id=3`。

用于控制二进制日志记录的源上使用的选项，请参阅第 19.1.6.4 节，“二进制日志选项和变量”。

##### 复制源服务器的启动选项

以下列表描述了用于控制复制源服务器的启动选项。复制相关的系统变量将在本节后面讨论。

+   `--show-replica-auth-info`

    | 命令行格式 | `--show-replica-auth-info[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    从 MySQL 8.0.26 开始，请使用 `--show-replica-auth-info`，在 MySQL 8.0.26 之前，请使用 `--show-slave-auth-info`。这两个选项具有相同的效果。这些选项在源上显示复制用户名和密码在 `SHOW REPLICAS`（或在 MySQL 8.0.22 之前，`SHOW SLAVE HOSTS`）的输出中，用于使用 `--report-user` 和 `--report-password` 选项启动的副本。

+   `--show-slave-auth-info`

    | 命令行格式 | `--show-slave-auth-info[={OFF&#124;ON}]` |
    | --- | --- |
    | 已弃用 | 8.0.26 |
    | 类型 | 布尔 |
    | 默认值 | `OFF` |

    在 MySQL 8.0.26 之前，请使用此选项，而不是 `--show-replica-auth-info`。这两个选项具有相同的效果。

##### 复制源服务器上使用的系统变量

以下系统变量用于或由复制源服务器使用：

+   `auto_increment_increment`

    | 命令行格式 | `--auto-increment-increment=#` |
    | --- | --- |
    | 系统变量 | `auto_increment_increment` |
    | 范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    `auto_increment_increment` 和 `auto_increment_offset` 用于循环（源到源）复制，并可用于控制 `AUTO_INCREMENT` 列的操作。这两个变量都有全局和会话值，每个变量的值都可以是介于 1 到 65,535 之间的整数。将这两个变量中的任何一个的值设置为 0 会使其值改为 1。尝试将这两个变量中的任何一个的值设置为大于 65,535 或小于 0 的整数会使其值改为 65,535。尝试将 `auto_increment_increment` 或 `auto_increment_offset` 的值设置为非整数值会产生错误，并且变量的实际值保持不变。

    注意

    `auto_increment_increment` 也支持用于 `NDB` 表。

    截至 MySQL 8.0.18，设置此系统变量的会话值不再是受限制的操作。

    当在服务器上启动组复制时，`auto_increment_increment` 的值会更改为 `group_replication_auto_increment_increment` 的值，默认为 7，并且 `auto_increment_offset` 的值会更改为服务器 ID。当停止组复制时，这些更改会被还原。只有当 `auto_increment_increment` 和 `auto_increment_offset` 的默认值都为 1 时，才会进行这些更改和还原。如果它们的值已经从默认值修改过，则组复制不会更改它们。从 MySQL 8.0 开始，当组复制处于单主模式时，即只有一个服务器写入时，系统变量也不会被修改。

    `auto_increment_increment`和`auto_increment_offset`影响`AUTO_INCREMENT`列的行为如下：

    +   `auto_increment_increment`控制连续列值之间的间隔。例如：

        ```sql
        mysql> SHOW VARIABLES LIKE 'auto_inc%';
        +--------------------------+-------+
        | Variable_name            | Value |
        +--------------------------+-------+
        | auto_increment_increment | 1     |
        | auto_increment_offset    | 1     |
        +--------------------------+-------+
        2 rows in set (0.00 sec)

        mysql> CREATE TABLE autoinc1
         -> (col INT NOT NULL AUTO_INCREMENT PRIMARY KEY);
          Query OK, 0 rows affected (0.04 sec)

        mysql> SET @@auto_increment_increment=10;
        Query OK, 0 rows affected (0.00 sec)

        mysql> SHOW VARIABLES LIKE 'auto_inc%';
        +--------------------------+-------+
        | Variable_name            | Value |
        +--------------------------+-------+
        | auto_increment_increment | 10    |
        | auto_increment_offset    | 1     |
        +--------------------------+-------+
        2 rows in set (0.01 sec)

        mysql> INSERT INTO autoinc1 VALUES (NULL), (NULL), (NULL), (NULL);
        Query OK, 4 rows affected (0.00 sec)
        Records: 4  Duplicates: 0  Warnings: 0

        mysql> SELECT col FROM autoinc1;
        +-----+
        | col |
        +-----+
        |   1 |
        |  11 |
        |  21 |
        |  31 |
        +-----+
        4 rows in set (0.00 sec)
        ```

    +   `auto_increment_offset`确定`AUTO_INCREMENT`列值的起始点。考虑以下情况，假设这些语句在与`auto_increment_increment`描述中给出的示例相同的会话中执行：

        ```sql
        mysql> SET @@auto_increment_offset=5;
        Query OK, 0 rows affected (0.00 sec)

        mysql> SHOW VARIABLES LIKE 'auto_inc%';
        +--------------------------+-------+
        | Variable_name            | Value |
        +--------------------------+-------+
        | auto_increment_increment | 10    |
        | auto_increment_offset    | 5     |
        +--------------------------+-------+
        2 rows in set (0.00 sec)

        mysql> CREATE TABLE autoinc2
         -> (col INT NOT NULL AUTO_INCREMENT PRIMARY KEY);
        Query OK, 0 rows affected (0.06 sec)

        mysql> INSERT INTO autoinc2 VALUES (NULL), (NULL), (NULL), (NULL);
        Query OK, 4 rows affected (0.00 sec)
        Records: 4  Duplicates: 0  Warnings: 0

        mysql> SELECT col FROM autoinc2;
        +-----+
        | col |
        +-----+
        |   5 |
        |  15 |
        |  25 |
        |  35 |
        +-----+
        4 rows in set (0.02 sec)
        ```

        当`auto_increment_offset`的值大于`auto_increment_increment`的值时，将忽略`auto_increment_offset`的值。

    如果这两个变量中的任何一个被更改，然后新行被插入到包含`AUTO_INCREMENT`列的表中，结果可能看起来令人费解，因为`AUTO_INCREMENT`值的系列是在不考虑列中已经存在的任何值的情况下计算的，并且插入的下一个值是系列中大于`AUTO_INCREMENT`列中最大现有值的最小值。该系列的计算方式如下：

    `auto_increment_offset` + *`N`* × `auto_increment_increment`

    其中*`N`*是系列[1, 2, 3, ...]中的正整数值。例如：

    ```sql
    mysql> SHOW VARIABLES LIKE 'auto_inc%';
    +--------------------------+-------+
    | Variable_name            | Value |
    +--------------------------+-------+
    | auto_increment_increment | 10    |
    | auto_increment_offset    | 5     |
    +--------------------------+-------+
    2 rows in set (0.00 sec)

    mysql> SELECT col FROM autoinc1;
    +-----+
    | col |
    +-----+
    |   1 |
    |  11 |
    |  21 |
    |  31 |
    +-----+
    4 rows in set (0.00 sec)

    mysql> INSERT INTO autoinc1 VALUES (NULL), (NULL), (NULL), (NULL);
    Query OK, 4 rows affected (0.00 sec)
    Records: 4  Duplicates: 0  Warnings: 0

    mysql> SELECT col FROM autoinc1;
    +-----+
    | col |
    +-----+
    |   1 |
    |  11 |
    |  21 |
    |  31 |
    |  35 |
    |  45 |
    |  55 |
    |  65 |
    +-----+
    8 rows in set (0.00 sec)
    ```

    所示的`auto_increment_increment`和`auto_increment_offset`的值生成系列 5 + *`N`* × 10，即[5, 15, 25, 35, 45, ...]。在`INSERT`之前`col`列中存在的最高值为 31，`AUTO_INCREMENT`系列中的下一个可用值为 35，因此`col`的插入值从该点开始，并且查询的结果如`SELECT`中所示。

    不可能将这两个变量的影响限制在单个表中；这些变量控制 MySQL 服务器上*所有*表中的所有`AUTO_INCREMENT`列的行为。如果设置了任一变量的全局值，则其影响将持续，直到更改全局值或通过设置会话值覆盖，或者直到**mysqld**被重新启动。如果设置了本地值，则新值会影响当前用户在会话期间插入新行的所有表的`AUTO_INCREMENT`列，除非在该会话期间更改了这些值。

    `auto_increment_increment`的默认值为 1。请参阅第 19.5.1.1 节，“复制和 AUTO_INCREMENT”。

+   `auto_increment_offset`

    | 命令行格式 | `--auto-increment-offset=#` |
    | --- | --- |
    | 系统变量 | `auto_increment_offset` |
    | 作用范围 | 全局，会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 是 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    此变量的默认值为 1。如果将其保留为默认值，并且在多主模式下在服务器上启动组复制，则将其更改为服务器 ID。有关更多信息，请参阅`auto_increment_increment`的描述。

    注意

    `auto_increment_offset` 也支持用于与`NDB`表。

    截至 MySQL 8.0.18，设置此系统变量的会话值不再是受限制的操作。

+   `immediate_server_version`

    | 引入版本 | 8.0.14 |
    | --- | --- |
    | 系统变量 | `immediate_server_version` |
    | 作用范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `999999` |
    | 最小值 | `0` |
    | 最大值 | `999999` |

    用于复制的内部使用。此会话系统变量保存了复制拓扑中作为直接源的 MySQL 服务器发布号（例如，MySQL 8.0.14 服务器实例的`80014`）。如果此直接服务器处于不支持会话系统变量的发布中，则变量的值设置为 0（`UNKNOWN_SERVER_VERSION`）。

    此变量的值从源复制到副本。有了这些信息，副本可以正确处理源自较旧发布的源的数据，通过识别在涉及的发布之间发生的语法更改或语义更改，并适当处理这些更改。此信息还可用于组复制环境，其中一个或多个复制组成员的发布版本比其他成员新。变量的值可以在每个事务的二进制日志中查看（作为`Gtid_log_event`的一部分，或者如果服务器上未使用 GTID，则作为`Anonymous_gtid_log_event`），并且在调试跨版本复制问题时可能会有所帮助。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有`REPLICATION_APPLIER`权限（请参阅第 19.3.3 节，“复制权限检查”），或具有足够权限设置受限制会话变量的权限（请参阅第 7.1.9.1 节，“系统变量权限”）。但是，请注意，该变量不是供用户设置的；它是由复制基础设施自动设置的。

+   `original_server_version`

    | 引入版本 | 8.0.14 |
    | --- | --- |
    | 系统变量 | `original_server_version` |
    | 范围 | 会话 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `999999` |
    | 最小值 | `0` |
    | 最大值 | `999999` |

    用于复制的内部使用。此会话系统变量保存了事务最初提交的 MySQL Server 版本号（例如，对于 MySQL 8.0.14 服务器实例，为`80014`）。如果此原始服务器的版本不支持会话系统变量，则变量的值将设置为 0（`UNKNOWN_SERVER_VERSION`）。请注意，当原始服务器设置了版本号时，如果立即服务器或复制拓扑中的任何其他中介服务器不支持会话系统变量，则变量的值将重置为 0，并且不会复制其值。

    该变量的值设置和使用方式与`immediate_server_version`系统变量相同。如果变量的值与`immediate_server_version`系统变量的值相同，则只记录后者在二进制日志中，并指示原始服务器版本相同。

    在 Group Replication 环境中，查看更改日志事件，这些事件是每个组成员在新成员加入组时排队的特殊事务，都带有排队事务的组成员的服务器版本标记。这确保了加入成员知道原始捐赠者的服务器版本。因为特定视图更改的排队事件在所有成员上具有相同的 GTID，所以在这种情况下，相同的 GTID 实例可能具有不同的原始服务器版本。

    设置此系统变量的会话值是受限制的操作。会话用户必须具有`REPLICATION_APPLIER`权限（参见第 19.3.3 节，“复制权限检查”），或具有足够权限设置受限制会话变量的权限（参见第 7.1.9.1 节，“系统变量权限”）。但是，请注意，该变量不是供用户设置的；它是由复制基础设施自动设置的。

+   `rpl_semi_sync_master_enabled`

    | 命令行格式 | `--rpl-semi-sync-master-enabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_enabled` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    控制源服务器上是否启用半同步复制。要启用或禁用插件，请将此变量设置为`ON`或`OFF`（或分别为 1 或 0）。默认值为`OFF`。

    仅当源端半同步复制插件已安装��才可用。

+   `rpl_semi_sync_master_timeout`

    | 命令行格式 | `--rpl-semi-sync-master-timeout=#` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_timeout` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    以毫秒为单位的值，控制源在提交等待来自副本的确认之前超时并回滚到异步复制的时间。默认值为 10000（10 秒）。

    仅当源端半同步复制插件已安装时才可用。

+   `rpl_semi_sync_master_trace_level`

    | 命令行格式 | `--rpl-semi-sync-master-trace-level=#` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_trace_level` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    源服务器上的半同步复制调试跟踪级别。定义了四个级别：

    +   1 = 一般级别（例如，时间函数失败）

    +   16 = 详细级别（更详细的信息）

    +   32 = 网络等待级别（有关网络等待的更多信息）

    +   64 = 函数级别（有关函数进入和退出的信息）

    仅当源端半同步复制插件已安装时才可用此变量。

+   `rpl_semi_sync_master_wait_for_slave_count`

    | 命令行格式 | `--rpl-semi-sync-master-wait-for-slave-count=#` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_wait_for_slave_count` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    源必须在继续之前每个事务收到的副本确认数。默认情况下，`rpl_semi_sync_master_wait_for_slave_count`为`1`，这意味着在收到单个副本确认后，半同步复制会继续进行。对于此变量的小值性能最佳。

    例如，如果`rpl_semi_sync_master_wait_for_slave_count`为`2`，则在超时期限配置为`rpl_semi_sync_master_timeout`之前，必须有 2 个副本确认接收事务，半同步复制才能继续。如果在超时期间较少的副本确认接收事务，则源会恢复到正常复制。

    注意

    此行为还取决于`rpl_semi_sync_master_wait_no_slave`

    仅当源端半同步复制插件已安装时才可用此变量。

+   `rpl_semi_sync_master_wait_no_slave`

    | 命令行格式 | `--rpl-semi-sync-master-wait-no-slave[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_wait_no_slave` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    控制源是否等待由`rpl_semi_sync_master_timeout`配置的超时期限到期，即使在超时期间副本计数降至少于`rpl_semi_sync_master_wait_for_slave_count`配置的副本数。

    当`rpl_semi_sync_master_wait_no_slave`的值为`ON`（默认）时，在超时期间允许副本计数降至少于`rpl_semi_sync_master_wait_for_slave_count`。只要足够多的副本在超时期限到期之前确认事务，半同步复制就会继续。

    当`rpl_semi_sync_master_wait_no_slave`的值为`OFF`时，如果在任何时候副本计数降至少于`rpl_semi_sync_master_wait_for_slave_count`中配置的数量，在由`rpl_semi_sync_master_timeout`配置的超时期间内，源将恢复到正常复制。

    此变量仅在安装了源端半同步复制插件时才可用。

+   `rpl_semi_sync_master_wait_point`

    | 命令行格式 | `--rpl-semi-sync-master-wait-point=value` |
    | --- | --- |
    | 系统变量 | `rpl_semi_sync_master_wait_point` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认数值 | `AFTER_SYNC` |
    | 有效数值 | `AFTER_SYNC``AFTER_COMMIT` |

    此变量控制半同步复制源服务器在向提交事务的客户端返回状态之前等待副本确认接收事务的点。这些值是允许的：

    +   `AFTER_SYNC`（默认）：源将每个事务写入其二进制日志和副本，并将二进制日志同步到磁盘。源在同步后等待副本确认接收事务。在收到确认后，源将事务提交给存储引擎并向客户端返回结果，然后客户端可以继续。

    +   `AFTER_COMMIT`：源将每个事务写入其二进制日志和副本，同步二进制日志，并提交事务给存储引擎。源在提交后等待副本确认接收事务。在收到确认后，源向客户端返回结果，然后客户端可以继续。

    这些设置的复制特性有以下不同：

    +   使用`AFTER_SYNC`，所有客户端同时看到已提交的事务：在副本确认并在源上提交给存储引擎之后。因此，所有客户端在源上看到相同的数据。

        在源故障的情况下，所有在源上提交的事务都已复制到副本（保存到其中继日志）。源服务器意外退出并故障转移至副本是无损的，因为副本是最新的。但请注意，在这种情况下源服务器不能重新启动，必须被丢弃，因为其二进制日志可能包含未提交的事务，在二进制日志恢复后在外部化后会与副本发生冲突。

    +   使用 `AFTER_COMMIT`，发出事务的客户端只有在服务器提交给存储引擎并接收到副本确认后才会收到返回状态。在提交之后和副本确认之前，其他客户端可能会在提交客户端之前看到已提交的事务。

        如果出现问题导致副本未能处理事务，则在源服务器意外退出并故障转移至副本时，这些客户端可能会看到相对于在源服务器上看到的数据有所丢失。

    仅当源端半同步复制插件已安装时才可用此变量。

    在 MySQL 5.7 中添加了 `rpl_semi_sync_master_wait_point` 后，由于它增加了半同步接口版本，创建了一个版本兼容性约束：MySQL 5.7 及更高版本的服务器不与旧版本的半同步复制插件一起工作，反之亦然。

+   `rpl_semi_sync_source_enabled`

    | 命令行格式 | `--rpl-semi-sync-source-enabled[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_enabled` |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    当在副本上安装了 `rpl_semi_sync_source`（`semisync_source.so`库）插件以设置半同步复制时，`rpl_semi_sync_source_enabled` 可用。如果安装了 `rpl_semi_sync_master` 插件（`semisync_master.so`库），则可用 `rpl_semi_sync_master_enabled`。

    `rpl_semi_sync_source_enabled` 控制源服务器上是否启用半同步复制。要启用或禁用插件，请将此变量设置为 `ON` 或 `OFF`（或分别设置为 1 或 0）。默认值为 `OFF`。

+   `rpl_semi_sync_source_timeout`

    | 命令行格式 | `--rpl-semi-sync-source-timeout=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_timeout` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `10000` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 毫秒 |

    `rpl_semi_sync_source_timeout` 在安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件的复制品上设置半同步复制时可用。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可用`rpl_semi_sync_master_timeout`。

    `rpl_semi_sync_source_timeout` 控制源在提交等待来自复制品的确认之前等待的时间，超时后将回滚到异步复制。该值以毫秒为单位指定，默认值为 10000（10 秒）。

+   `rpl_semi_sync_source_trace_level`

    | 命令行格式 | `--rpl-semi-sync-source-trace-level=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_trace_level` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `32` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    `rpl_semi_sync_source_trace_level` 在安装了`rpl_semi_sync_source`（`semisync_source.so`库）插件的复制品上设置半同步复制时可用。如果安装了`rpl_semi_sync_master`插件（`semisync_master.so`库），则可用`rpl_semi_sync_master_trace_level`。

    `rpl_semi_sync_source_trace_level` 指定源服务器上半同步复制的调试跟踪级别。定义了四个级别：

    +   1 = 一般级别（例如，时间函数失败）

    +   16 = 详细级别（更详细的信息）

    +   32 = 网络等待级别（有关网络等待的更多信息）

    +   64 = 函数级别（有关函数进入和退出的信息）

+   `rpl_semi_sync_source_wait_for_replica_count`

    | 命令行格式 | `--rpl-semi-sync-source-wait-for-replica-count=#` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_wait_for_replica_count` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `1` |
    | 最大值 | `65535` |

    当在副本上安装了 `rpl_semi_sync_source` (`semisync_source.so` 库) 插件以设置半同步复制时，`rpl_semi_sync_source_wait_for_replica_count` 就可用。如果安装了 `rpl_semi_sync_master` 插件 (`semisync_master.so` 库)，则可以使用 `rpl_semi_sync_master_wait_for_slave_count`。

    `rpl_semi_sync_source_wait_for_replica_count` 指定源在继续之前必须收到每个事务的副本确认数。默认情况下，`rpl_semi_sync_source_wait_for_replica_count` 是 `1`，意味着在收到单个副本确认后，半同步复制会继续进行。对于此变量的小值，性能最佳。

    例如，如果 `rpl_semi_sync_source_wait_for_replica_count` 是 `2`，那么在半同步复制的超时期间内，必须有 2 个副本确认接收事务，才能继续进行半同步复制。如果在超时期间内有更少的副本确认接收事务，则源将恢复为正常复制。

    注意

    这种行为还取决于 `rpl_semi_sync_source_wait_no_replica`。

+   `rpl_semi_sync_source_wait_no_replica`

    | 命令行格式 | `--rpl-semi-sync-source-wait-no-replica[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_wait_no_replica` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    当在复制品上安装了 `rpl_semi_sync_source`（`semisync_source.so` 库）插件以设置半同步复制时，可以使用 `rpl_semi_sync_source_wait_no_replica`。如果安装了 `rpl_semi_sync_master` 插件（`semisync_master.so` 库），则可以使用 `rpl_semi_sync_source_wait_no_replica` 控制源端是否等待由 `rpl_semi_sync_source_timeout` 配置的超时期限到期，即使在超时期间复制品数量少于 `rpl_semi_sync_source_wait_for_replica_count` 配置的数量。

    当 `rpl_semi_sync_source_wait_no_replica` 的值为 `ON`（默认值）时，允许在超时期间复制品数量少于 `rpl_semi_sync_source_wait_for_replica_count`。只要足够多的复制品在超时期限到期前确认事务，半同步复制就会继续。

    当 `rpl_semi_sync_source_wait_no_replica` 的值为 `OFF` 时，如果在由 `rpl_semi_sync_source_timeout` 配置的超时期间内任何时候复制品数量少于 `rpl_semi_sync_source_wait_for_replica_count` 配置的数量，源端将恢复为正常复制。

+   `rpl_semi_sync_source_wait_point`

    | 命令行格式 | `--rpl-semi-sync-source-wait-point=value` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `rpl_semi_sync_source_wait_point` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认数值 | `AFTER_SYNC` |
    | 有效数值 | `AFTER_SYNC``AFTER_COMMIT` |

    当在复制品上安装了 `rpl_semi_sync_source`（`semisync_source.so` 库）插件以设置半同步复制时，可以使用 `rpl_semi_sync_source_wait_point`。如果安装了 `rpl_semi_sync_master` 插件（`semisync_master.so` 库），则可以使用 `rpl_semi_sync_master_wait_point`。

    `rpl_semi_sync_source_wait_point` 控制半同步复制源服务器在返回提交事务的客户端状态之前等待副本确认事务接收的点。允许这些值：

    +   `AFTER_SYNC`（默认）：源将每个事务写入其二进制日志和副本，并将二进制日志同步到磁盘。源在同步后等待副本确认事务接收。在收到确认后，源将事务提交到存储引擎并向客户端返回结果，然后客户端可以继续。

    +   `AFTER_COMMIT`：源将每个事务写入其二进制日志和副本，同步二进制日志，并提交事务到存储引擎。源在提交后等待副本确认事务接收。在收到确认后，源向客户端返回结果，然后客户端可以继续。

    这些设置的复制特性有以下不同：

    +   使用`AFTER_SYNC`，所有客户端同时看到已提交的事务：在副本确认并在源上提交到存储引擎之后。因此，所有客户端在源上看到相同的数据。

        在源故障的情况下，所有在源上提交的事务已经被复制到副本（保存在其中继日志中）。源服务器意外退出并故障转移到副本是无损失的，因为副本是最新的。但请注意，在这种情况下源不能重新启动，必须被丢弃，因为其二进制日志可能包含未提交的事务，在二进制日志恢复后会与副本在外部化时发生冲突。

    +   使用`AFTER_COMMIT`，发出事务的客户端只有在服务器提交到存储引擎并接收副本确认后才会收到返回状态。在提交后和副本确认前，其他客户端可以在提交客户端之前看到已提交的事务。

        如果副本未处理事务导致出现问题，那么在源服务器意外退出并故障转移到副本的情况下，这些客户端可能会看到相对于在源上看到的数据有所丢失。
