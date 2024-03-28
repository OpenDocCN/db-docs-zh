> 原文：[`dev.mysql.com/doc/refman/8.0/en/clone-plugin-replication.html`](https://dev.mysql.com/doc/refman/8.0/en/clone-plugin-replication.html)

#### 7.6.7.7 复制用的克隆

克隆插件支持复制。除了克隆数据外，克隆操作还会从捐赠者提取复制坐标并将其传输给接收者，这使得可以使用克隆插件为配置组复制成员和副本提供服务。使用克隆插件进行配置比复制大量事务要快得多且更有效率。

配置组复制成员也可以配置为使用克隆插件作为分布式恢复的选项，这样加入成员会自动选择从现有组成员检索组数据的最有效方式。有关更多信息，请参见 Section 20.5.4.2, “Cloning for Distributed Recovery”。

在克隆操作期间，二进制日志位置（文件名、偏移量）和`gtid_executed` GTID 集都会从捐赠者的 MySQL 服务器实例中提取并传输到接收者。这些数据允许在复制流中的一致位置启动复制。二进制日志和中继日志（保存在文件中）不会从捐赠者复制到接收者。为了启动复制，接收者需要的二进制日志必须在数据克隆和启动复制之间不被清除。如果所需的二进制日志不可用，则会报告复制握手错误。因此，克隆实例应尽快添加到复制组中，以避免所需的二进制日志被清除或新成员明显滞后，需要更多的恢复时间。

+   在克隆的 MySQL 服务器实例上执行此查询，以检查已传输给接收者的二进制日志位置：

    ```sql
    mysql> SELECT BINLOG_FILE, BINLOG_POSITION FROM performance_schema.clone_status;
    ```

+   在克隆的 MySQL 服务器实例上执行此查询，以检查已传输给接收者的`gtid_executed` GTID 集：

    ```sql
    mysql> SELECT @@GLOBAL.GTID_EXECUTED;
    ```

在 MySQL 8.0 中，默认情况下，复制元数据存储库保存在在克隆操作期间从捐赠者复制到接收者的表中。复制元数据存储库保存了可以在克隆操作后正确恢复复制的与复制相关的配置设置。

+   在 MySQL 8.0.17 和 8.0.18 中，只会复制表 `mysql.slave_master_info`（连接元数据存储库）。

+   从 MySQL 8.0.19 开始，表 `mysql.slave_relay_log_info`（应用程序元数据存储库）和 `mysql.slave_worker_info`（应用程序工作程序元数据存储库）也会被复制。

要查看每个表中包含的内容列表，请参阅 Section 19.2.4.2, “Replication Metadata Repositories”。请注意，如果服务器上使用了设置 `master_info_repository=FILE` 和 `relay_log_info_repository=FILE`（这在 MySQL 8.0 中不是默认设置且已被弃用），则不会克隆复制元数据存储库；只有在设置为 `TABLE` 时才会克隆。

要进行复制克隆，请执行以下步骤：

1.  对于 Group Replication 的新成员，首先按照 Section 20.2.1.6, “Adding Instances to the Group” 中的说明配置 MySQL Server 实例以进行 Group Replication。同时，设置克隆的先决条件，详见 Section 20.5.4.2, “Cloning for Distributed Recovery”。当在加入成员上发出 `START GROUP_REPLICATION` 命令时，克隆操作将由 Group Replication 自动管理，因此您无需手动执行操作，也无需在加入成员上执行任何进一步的设置步骤。

1.  对于源/复制 MySQL 复制拓扑中的副本，首先手动将数据从捐赠方 MySQL 服务器实例克隆到接收方。捐赠方必须是复制拓扑中的源或副本。有关克隆说明，请参阅 Section 7.6.7.3, “Cloning Remote Data”。

1.  克隆操作成功完成后，如果您希望在接收方 MySQL 服务器实例上使用与捐赠方相同的复制通道，请验证哪些通道可以在源/复制 MySQL 复制拓扑中自动恢复复制，哪些需要手动设置。

    +   对于基于 GTID 的复制，如果接收方配置为 `gtid_mode=ON` 并且从配置为 `gtid_mode=ON`、`ON_PERMISSIVE` 或 `OFF_PERMISSIVE` 的捐赠方克隆，那么从捐赠方应用 `gtid_executed` GTID 集到接收方。如果接收方是从已在拓扑中的副本克隆而来，那么在克隆操作后，使用 GTID 自动定位的复制通道可以在启动通道后自动恢复复制。如果您只想使用这些相同通道，则无需执行任何手动设置。

    +   对于基于二进制日志文件位置的复制，如果接收端是 MySQL 8.0.17 或 8.0.18，则从提供端的二进制日志位置不会应用到接收端，只会记录在性能模式 `clone_status` 表中。因此，接收端上使用基于二进制日志文件位置的复制通道必须手动设置以在克隆操作后恢复复制。确保这些通道未配置为在服务器启动时自动开始复制，因为它们尚未具有二进制日志位置，并尝试从头开始复制。

    +   对于基于二进制日志文件位置的复制，如果接收端是 MySQL 8.0.19 或更高版本，则从提供端应用二进制日志位置到接收端。接收端上使用基于二进制日志文件位置的复制通道会自动尝试执行中继日志恢复过程，使用克隆的中继日志信息，在重新启动复制之前。对于单线程副本（`replica_parallel_workers` 或 `slave_parallel_workers` 设置为 0），在没有其他问题的情况下，中继日志恢复应该成功，使通道能够在没有进一步设置的情况下恢复复制。对于多线程副本（`replica_parallel_workers` 或 `slave_parallel_workers` 大于 0），中继日志恢复可能会失败，因为通常无法自动完成。在这种情况下，会发出错误消息，您必须手动设置通道。

1.  如果您需要手动设置克隆复制通道，或者希望在接收端使用不同的复制通道，以下说明提供了一个摘要和简化示例，用于将接收端 MySQL 服务器实例添加到复制拓扑中。还请参考适用于您的复制设置的详细说明。

    +   要将一个接收方 MySQL 服务器实例添加到使用基于 GTID 的事务作为复制数据源的 MySQL 复制拓扑中，请根据需要配置实例，并按照 Section 19.1.3.4，“使用 GTID 设置复制”中的说明操作。按照以下简化示例为实例添加复制通道。`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）必须定义源的主机地址和端口号，并且应启用`SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION`选项，如下所示：

        ```sql
        mysql> CHANGE MASTER TO MASTER_HOST = '*source_host_name*', MASTER_PORT = *source_port_num*,
               ...
               MASTER_AUTO_POSITION = 1,
               FOR CHANNEL '*setup_channel*';
        mysql> START SLAVE USER = '*user_name*' PASSWORD = '*password*' FOR CHANNEL '*setup_channel*';

        Or from MySQL 8.0.22 and 8.0.23:

        mysql> CHANGE SOURCE TO SOURCE_HOST = '*source_host_name*', SOURCE_PORT = *source_port_num*,
               ...
               SOURCE_AUTO_POSITION = 1,
               FOR CHANNEL '*setup_channel*';
        mysql> START REPLICA USER = '*user_name*' PASSWORD = '*password*' FOR CHANNEL '*setup_channel*';
        ```

    +   要将一个接收方 MySQL 服务器实例添加到使用基于二进制日志文件位置的复制的 MySQL 复制拓扑中，请根据需要配置实例，并按照 Section 19.1.2，“设置基于二进制日志文件位置的复制”中的说明操作。按照以下简化示例为实例添加复制通道，使用在克隆操作期间传输给接收方的二进制日志位置：

        ```sql
        mysql> SELECT BINLOG_FILE, BINLOG_POSITION FROM performance_schema.clone_status;
        mysql> CHANGE MASTER TO MASTER_HOST = '*source_host_name*', MASTER_PORT = *source_port_num*,
               ...
               MASTER_LOG_FILE = '*source_log_name*',
               MASTER_LOG_POS = *source_log_pos*,
               FOR CHANNEL '*setup_channel*';
        mysql> START SLAVE USER = '*user_name*' PASSWORD = '*password*' FOR CHANNEL '*setup_channel*';

        Or from MySQL 8.0.22 and 8.0.23:

        mysql> SELECT BINLOG_FILE, BINLOG_POSITION FROM performance_schema.clone_status;
        mysql> CHANGE SOURCE TO SOURCE_HOST = '*source_host_name*', SOURCE_PORT = *source_port_num*,
               ...
               SOURCE_LOG_FILE = '*source_log_name*',
               SOURCE_LOG_POS = *source_log_pos*,
               FOR CHANNEL '*setup_channel*';
        mysql> START REPLICA USER = '*user_name*' PASSWORD = '*password*' FOR CHANNEL '*setup_channel*';
        ```
