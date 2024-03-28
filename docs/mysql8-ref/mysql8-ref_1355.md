> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-disable-gtids.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-disable-gtids.html)

#### 19.1.4.3 在线禁用 GTID 事务

本节描述了如何在已经在线的服务器上禁用 GTID 事务。此过程无需将服务器脱机，并适用于在生产环境中使用。但是，如果您有可能在禁用 GTID 模式时将服务器脱机，那么该过程会更加简单。

该过程类似于在服务器在线时启用 GTID 事务，但是步骤相反。唯一不同的是等待已记录事务复制的时间点。

在开始之前，请确保服务器满足以下先决条件：

+   您的拓扑中的*所有*服务器必须使用 MySQL 5.7.6 或更高版本。除非拓扑中的*所有*服务器都使用此版本，否则无法在线禁用 GTID 事务。

+   所有服务器的`gtid_mode`设置为`ON`。

+   任何服务器上都未设置`--replicate-same-server-id`选项。如果此选项与`--log-slave-updates`选项（默认情况下）和启用了二进制日志记录（也是默认情况）一起设置，则无法禁用 GTID 事务。在没有 GTID 的情况下，这些选项的组合会在循环复制中导致无限循环。

1.  在每个副本上执行以下操作，如果您正在使用多源复制，请为每个通道执行，并包括`FOR CHANNEL`通道子句：

    ```sql
    STOP SLAVE [FOR CHANNEL 'channel'];
    CHANGE MASTER TO MASTER_AUTO_POSITION = 0, MASTER_LOG_FILE = file, \
    MASTER_LOG_POS = position [FOR CHANNEL 'channel'];
    START SLAVE [FOR CHANNEL 'channel'];

    Or from MySQL 8.0.22 / 8.0.23:
    STOP REPLICA [FOR CHANNEL 'channel'];
    CHANGE REPLICATION SOURCE TO SOURCE_AUTO_POSITION = 0, SOURCE_LOG_FILE = file, \
    SOURCE_LOG_POS = position [FOR CHANNEL 'channel'];
    START REPLICA [FOR CHANNEL 'channel'];
    ```

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = ON_PERMISSIVE;
    ```

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = OFF_PERMISSIVE;
    ```

1.  在每台服务器上，等待变量@@GLOBAL.GTID_OWNED 等于空字符串。可以使用以下方式进行检查：

    ```sql
    SELECT @@GLOBAL.GTID_OWNED;
    ```

    理论上，在副本上，这可能为空，然后再次变为非空。这不是问题，只要它曾经为空就足够了。

1.  等待当前存在于任何二进制日志中的所有事务在所有副本中复制。参见 Section 19.1.4.4, “验证匿名事务的复制”，了解检查所有匿名事务是否已复制到所有服务器的一种方法。

1.  如果您将二进制日志用于除复制之外的任何其他用途，例如进行时间点备份或还原：请等待您不再需要具有 GTID 事务的旧二进制日志。

    例如，在完成第 5 步之后，您可以在正在进行备份的服务器上执行`FLUSH LOGS`。然后，要么明确进行备份，要么等待您设置的任何定期备份例程的下一次迭代。

    理想情况下，等待服务器清除在步骤 5 完成时存在的所有二进制日志。还要等待在步骤 5 之前备份的任何备份过期。

    重要提示

    这是整个过程中的一个重要点。重要的是要理解，包含 GTID 事务的日志在下一步之后不能再使用。在继续之前，您必须确保拓扑结构中不存在任何 GTID 事务。

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = OFF;
    ```

1.  在每台服务器上，在 `my.cnf` 中设置 `gtid_mode=OFF`。

    如果您想设置 `enforce_gtid_consistency=OFF`，现在可以这样做。设置后，您应该将 `enforce_gtid_consistency=OFF` 添加到您的配置文件中。

如果您想降级到 MySQL 的早期版本，现在可以使用正常的降级过程进行操作。
