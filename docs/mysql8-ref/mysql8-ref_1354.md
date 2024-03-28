> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-enable-gtids.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-mode-change-online-enable-gtids.html)

#### 19.1.4.2 启用 GTID 事务在线

本节描述了如何在已在线并使用匿名事务的服务器上启用 GTID 事务，以及可选的自动定位。此过程不需要将服务器脱机，并适用于生产环境。但是，如果在启用 GTID 事务时有可能将服务器脱机，则该过程更容易。

从 MySQL 8.0.23 开始，您可以设置复制通道，为尚未具有 GTID 的复制事务分配 GTID。此功能使得可以从不使用基于 GTID 的复制的源服务器复制到使用该功能的副本服务器。如果可能在复制源服务器上启用 GTID，如本过程中所述，请使用此方法。分配 GTID 适用于无法启用 GTID 的复制源服务器。有关此选项的更多信息，请参阅第 19.1.3.6 节，“从不具有 GTID 的源复制到具有 GTID 的副本”。

在开始之前，请确保服务器满足以下先决条件：

+   您的拓扑中的*所有*服务器必须使用 MySQL 5.7.6 或更高版本。除非*所有*拓扑中的服务器都使用此版本，否则无法在任何单个服务器上在线启用 GTID 事务。

+   所有服务器的`gtid_mode`设置为默认值`OFF`。

以下过程可以随时暂停，并在原地恢复，或通过跳转到第 19.1.4.3 节，“在线禁用 GTID 事务”的相应步骤来撤销。这使得该过程具有容错性，因为在过程中可能出现的任何不相关问题都可以像往常一样处理，然后继续在离开的地方继续进行。

注意

在继续下一步之前，您必须完成每个步骤。

要启用 GTID 事务：

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY = WARN;
    ```

    让服务器在正常工作负载下运行一段时间，并监视日志。如果此步骤在日志中引发任何警告，请调整应用程序，使其仅使用 GTID 兼容功能，并且不生成任何警告。

    重要

    这是第一个重要的步骤。在继续下一步之前，您必须确保错误日志中没有生成任何警告。

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.ENFORCE_GTID_CONSISTENCY = ON;
    ```

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = OFF_PERMISSIVE;
    ```

    不重要哪个服务器首先执行此语句，但重要的是所有服务器在任何服务器开始下一步之前完成此步骤。

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = ON_PERMISSIVE;
    ```

    这个语句首先由哪台服务器执行并不重要。

1.  在每台服务器上，等待直到状态变量 `ONGOING_ANONYMOUS_TRANSACTION_COUNT` 为零。可以使用以下方式进行检查：

    ```sql
    SHOW STATUS LIKE 'ONGOING_ANONYMOUS_TRANSACTION_COUNT';
    ```

    注意

    在副本上，理论上可能会显示零，然后再次显示非零。这不是问题，只要它显示零一次即可。

1.  等待生成到第 5 步的所有事务复制到所有服务器。您可以在不停止更新的情况下执行此操作：唯一重要的是所有匿名事务都得到复制。

    参见 Section 19.1.4.4, “验证匿名事务的复制” 以了解检查所有匿名事务是否已复制到所有服务器的一种方法。

1.  如果您将二进制日志用于除复制之外的任何其他用途，例如时间点备份和恢复，请等到您不再需要具有不带 GTIDs 事务的旧二进制日志。

    例如，在第 6 步完成后，您可以在正在进行备份的服务器上执行 `FLUSH LOGS`。然后要么明确地进行备份，要么等待您设置的任何定期备份例程的下一次迭代。

    理想情况下，等待服务器清除在完成第 6 步时存在的所有二进制日志。还要等待在第 6 步之前进行的任何备份过期。

    重要

    这是第二个重要的要点。理解二进制日志中包含的匿名事务，如果没有 GTIDs，在下一步之后将无法使用。在此步骤之后，您必须确保在拓扑结构中不存在不带 GTIDs 的事务。

1.  在每台服务器上执行：

    ```sql
    SET @@GLOBAL.GTID_MODE = ON;
    ```

1.  在每台服务器上，将 `gtid_mode=ON` 和 `enforce_gtid_consistency=ON` 添加到 `my.cnf`。

    现在，您可以确保所有事务都具有 GTID（除了在第 5 步或更早生成的事务已经被处理）。为了开始使用 GTID 协议，以便稍后执行自动故障转移，在每个副本上执行以下操作。可选地，如果使用多源复制，请为每个通道执行此操作，并包括 `FOR CHANNEL *`channel`*` 子句：

    ```sql
    STOP SLAVE [FOR CHANNEL 'channel'];
    CHANGE MASTER TO MASTER_AUTO_POSITION = 1 [FOR CHANNEL 'channel'];
    START SLAVE [FOR CHANNEL 'channel'];

    Or from MySQL 8.0.22 / 8.0.23:
    STOP REPLICA [FOR CHANNEL 'channel'];
    CHANGE REPLICATION SOURCE TO SOURCE_AUTO_POSITION = 1 [FOR CHANNEL 'channel'];
    START REPLICA [FOR CHANNEL 'channel'];
    ```
