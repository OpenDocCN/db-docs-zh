> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-gtids-howto.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-gtids-howto.html)

#### 19.1.3.4 使用 GTID 设置复制

本节描述了在 MySQL 8.0 中配置和启动基于 GTID 的复制的过程。这是一个“冷启动”过程，假设您要么是第一次启动源服务器，要么可以停止它；有关从运行中的源服务器使用 GTID 为副本提供服务的信息，请参阅第 19.1.3.5 节，“使用 GTID 进行故障转移和扩展”。有关在线更改服务器上的 GTID 模式的信息，请参阅第 19.1.4 节，“在线更改 GTID 模式”。

最简单的 GTID 复制拓扑结构的启动过程中的关键步骤，包括一个源和一个副本，如下所示：

1.  如果复制已经在运行，请使两个服务器变为只读状态以进行同步。

1.  停止两个服务器。

1.  使用启用了 GTID 并配置了正确选项的正确选项重新启动两个服务器。

    启动服务器所需的**mysqld**选项将在本节稍后的示例中讨论。

1.  指示副本使用源作为复制数据源并使用自动定位。完成此步骤所需的 SQL 语句在本节稍后的示例中描述。

1.  进行新的备份。包含没有 GTID 的事务的二进制日志不能在启用 GTID 的服务器上使用，因此在此点之前进行的备份不能与新配置一起使用。

1.  启动副本，然后在两个服务器上禁用只读模式，以便它们可以接受更新。

在以下示例中，两个服务器已经作为源和副本运行，使用 MySQL 的基于二进制日志位置的复制协议。如果您要使用新服务器，请参阅第 19.1.2.3 节，“为复制创建用户”以获取有关为复制连接添加特定用户的信息，以及第 19.1.2.1 节，“设置复制源配置”以获取有关设置`server_id`变量的信息。以下示例显示了如何在服务器的选项文件中存储**mysqld**启动选项，请参阅第 6.2.2.2 节，“使用选项文件”以获取更多信息。或者，您可以在运行**mysqld**时使用启动选项。

后续大部分步骤需要使用 MySQL `root` 帐户或具有`SUPER`权限的其他 MySQL 用户帐户。**mysqladmin** `shutdown` 需要`SUPER`权限或`SHUTDOWN`权限。

**第一步：同步服务器。** 仅在使用不使用 GTID 复制的服务器时才需要此步骤。对于新服务器，请继续到第三步。通过在每个服务器上将`read_only`系统变量设置为`ON`来使服务器只读，发出以下命令：

```sql
mysql> SET @@GLOBAL.read_only = ON;
```

等待所有正在进行的事务提交或回滚。然后，允许复制品赶上源。*确保复制品处理了所有更新非常重要*。

如果您使用二进制日志来进行除了复制之外的任何操作，例如进行时点备份和恢复，请等到您不再需要包含没有 GTID 的旧二进制日志。理想情况下，等待服务器清除所有二进制日志，并等待任何现有备份过期。

重要提示

重要提示：必须了解不包含 GTID 的事务的日志不能在启用 GTID 的服务器上使用。在继续之前，您必须确保在拓扑结构中任何地方都不存在不带 GTID 的事务。

**第二步：停止两个服务器。** 使用**mysqladmin**停止每个服务器，如下所示，其中*`username`*是具有足够权限关闭服务器的 MySQL 用户的用户名：

```sql
$> mysqladmin -u*username* -p shutdown
```

然后在提示处提供此用户的密码。

**第三步：启用两个启用 GTID 的服务器。** 要启用基于 GTID 的复制，必须通过将`gtid_mode`变量设置为`ON`来启用每个服务器的 GTID 模式，并通过启用`enforce_gtid_consistency`变量来确保仅记录对于基于 GTID 的复制安全的语句。例如：

```sql
gtid_mode=ON
enforce-gtid-consistency=ON
```

使用 `--skip-slave-start` 选项或从 MySQL 8.0.24 开始，使用 `skip_slave_start` 系统变量，确保在配置副本设置之前不会启动复制。从 MySQL 8.0.26 开始，改用 `--skip-replica-start` 或 `skip_replica_start`。有关 GTID 相关选项和变量的更多信息，请参见 第 19.1.6.5 节，“全局事务 ID 系统变量”。

在使用 mysql.gtid_executed 表 时，不需要启用二进制日志记录才能使用 GTIDs 是强制性的。源服务器必须始终启用二进制日志记录才能进行复制。但是，副本服务器可以使用 GTIDs，但不启用二进制日志记录。如果需要在副本服务器上禁用二进制日志记录，可以通过为副本指定 `--skip-log-bin` 和 `--log-replica-updates=OFF` 或 `--log-slave-updates=OFF` 选项来实现。

**第 4 步：配置副本以使用基于 GTID 的自动定位。** 告诉副本使用具有基于 GTID 的事务的源作为复制数据源，并使用基于 GTID 的自动定位而不是基于文件的定位。在副本上发出 `CHANGE REPLICATION SOURCE TO` 语句（从 MySQL 8.0.23 开始）或 `CHANGE MASTER TO` 语句（在 MySQL 8.0.23 之前），在语句中包含 `SOURCE_AUTO_POSITION` | `MASTER_AUTO_POSITION` 选项，告诉副本源的事务由 GTIDs 标识。

您可能还需要为源主机名和端口号以及用于副本连接到源的复制用户帐户的用户名和密码提供适当的值；如果这些值在第 1 步之前已经设置并且不需要进行进一步更改，则可以安全地从此处显示的语句中省略相应的选项。

```sql
mysql> CHANGE MASTER TO
     >     MASTER_HOST = *host*,
     >     MASTER_PORT = *port*,
     >     MASTER_USER = *user*,
     >     MASTER_PASSWORD = *password*,
     >     MASTER_AUTO_POSITION = 1;

Or from MySQL 8.0.23:

mysql> CHANGE REPLICATION SOURCE TO
     >     SOURCE_HOST = *host*,
     >     SOURCE_PORT = *port*,
     >     SOURCE_USER = *user*,
     >     SOURCE_PASSWORD = *password*,
     >     SOURCE_AUTO_POSITION = 1;
```

**第 5 步：进行新的备份。** 在启用 GTIDs 之前制作的现有备份现在无法在这些服务器上使用，因为您已经启用了 GTIDs。在这一点上进行新的备份，这样您就不会没有可用的备份了。

例如，您可以在执行备份的服务器上执行`FLUSH LOGS`。然后要么明确地进行备份，要么等待您设置的任何定期备份例程的下一次迭代。

**步骤 6：启动副本并禁用只读模式。** 像这样启动副本：

```sql
mysql> START SLAVE;
Or from MySQL 8.0.22:
mysql> START REPLICA;
```

如果您在第 1 步中将服务器配置为只读，则需要执行以下步骤才能使服务器再次接受更新。发出以下语句：

```sql
mysql> SET @@GLOBAL.read_only = OFF;
```

基于 GTID 的复制现在应该正在运行，您可以像以前一样开始（或恢复）源上的活动。第 19.1.3.5 节，“使用 GTIDs 进行故障转移和扩展”，讨论了在使用 GTIDs 时创建新副本。
