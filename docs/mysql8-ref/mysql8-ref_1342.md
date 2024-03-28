> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-howto-additionalslaves.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-additionalslaves.html)

#### 19.1.2.8 向复制环境添加复制实例

您可以在不停止源服务器的情况下向现有复制配置中添加另一个复制实例。为此，您可以通过复制现有复制实例的数据目录来设置新的复制实例，并为新的复制实例指定不同的服务器 ID（由用户指定）和服务器 UUID（在启动时生成）。

注意

如果您要复制以创建新的复制实例的复制源服务器或现有复制实例具有任何计划事件，请确保在启动新的复制实例之前将这些事件禁用。如果新的复制实例上运行的事件已经在源上运行过，则重复的操作会导致错误。事件调度程序由`event_scheduler`系统变量控制，默认情况下从 MySQL 8.0 开始为`ON`，因此在新的复制实例启动时默认情况下会运行在原始服务器上活动的事件。要停止新的复制实例上的所有事件运行，请在新的复制实例上将`event_scheduler`系统变量设置为`OFF`或`DISABLED`。或者，您可以使用`ALTER EVENT`语句将单个事件设置为`DISABLE`或`DISABLE ON SLAVE`以防止它们在新的复制实例上运行。您可以使用`SHOW`语句或信息模式`EVENTS`表列出服务器上的事件。有关更多信息，请参见 Section 19.5.1.16, “Replication of Invoked Features”。

作为以这种方式创建新的复制实例的替代方法，MySQL 服务器的克隆插件可用于将所有数据和复制设置从现有复制实例传输到克隆实例。有关使用此方法的说明，请参见 Section 7.6.7.7, “Cloning for Replication”。

要复制现有的复制实例而不进行克隆，请按照以下步骤操作：

1.  停止现有的复制实例并记录复制实例状态信息，特别是源二进制日志文件和中继日志文件位置。您可以通过性能模式复制表（请参阅 Section 29.12.11, “Performance Schema Replication Tables”）查看复制实例状态，或通过以下方式发出`SHOW REPLICA STATUS`：

    ```sql
    mysql> STOP SLAVE;
    mysql> SHOW SLAVE STATUS\G
    Or from MySQL 8.0.22:
    mysql> STOP REPLICA;
    mysql> SHOW REPLICA STATUS\G
    ```

1.  关闭现有的复制实例：

    ```sql
    $> mysqladmin shutdown
    ```

1.  将现有副本的数据目录复制到新副本，包括日志文件和中继日志文件。您可以通过使用**tar**或`WinZip`创建存档，或者通过使用**cp**或**rsync**等工具执行直接复制来完成此操作。

    重要

    +   在复制之前，请验证所有与现有副本相关的文件实际上是否存储在数据目录中。例如，`InnoDB`系统表空间、撤消表空间和重做日志可能存储在其他位置。`InnoDB`表空间文件和文件表空间可能已在其他目录中创建。副本的二进制日志和中继日志可能在数据目录之外的自己的目录中。检查为现有副本设置的系统变量，并查找是否已指定任何替代路径。如果找到任何内容，请将这些目录一并复制过去。

    +   在复制过程中，如果文件用于复制元数据存储库（请参阅第 19.2.4 节，“中继日志和复制元数据存储库”），请确保还将这些文件从现有副本复制到新副本。如果表用于存储库，这是从 MySQL 8.0 开始的默认设置，则这些表位于数据目录中。

    +   复制后，从新副本的数据目录副本中删除`auto.cnf`文件，以便新副本使用不同生成的服务器 UUID 启动。服务器 UUID 必须是唯一的。

    添加新副本时遇到的常见问题是，新副本失败，并显示一系列警告和错误消息，例如：

    ```sql
    071118 16:44:10 [Warning] Neither --relay-log nor --relay-log-index were used; so
    replication may break when this MySQL server acts as a replica and has his hostname
    changed!! Please use '--relay-log=*new_replica_hostname*-relay-bin' to avoid this problem.
    071118 16:44:10 [ERROR] Failed to open the relay log './*old_replica_hostname*-relay-bin.003525'
    (relay_log_pos 22940879)
    071118 16:44:10 [ERROR] Could not find target log during relay log initialization
    071118 16:44:10 [ERROR] Failed to initialize the master info structure
    ```

    如果未指定`relay_log`系统变量，则可能会出现此情况，因为中继日志文件的文件名中包含主机名。如果未使用`relay_log_index`系统变量，则中继日志索引文件也是如此。有关这些变量的更多信息，请参见第 19.1.6 节，“复制和二进制日志选项和变量”。

    为避免此问题，在新的复制品上使用与现有复制品上使用的`relay_log`相同的值。如果在现有复制品上未显式设置此选项，请使用`*`existing_replica_hostname`*-relay-bin`。如果不可能，请将现有复制品的中继日志索引文件复制到新的复制品，并将新的复制品上的`relay_log_index`系统变量设置为与现有复制品上使用的相匹配。如果在现有复制品上未显式设置此选项，请使用`*`existing_replica_hostname`*-relay-bin.index`。或者，如果您在按照本节中的其余步骤后已尝试启动新的复制品并遇到类似先前描述的错误，则执行以下步骤：

    1.  如果您尚未这样做，请在新的复制品上发出`STOP REPLICA`。

        如果您已经重新启动了现有的复制品，请在现有的复制品上也发出`STOP REPLICA`。

    1.  将现有复制品的中继日志索引文件的内容复制到新复制品的中继日志索引文件中，确保覆盖文件中已有的任何内容。

    1.  继续执行本节中的其余步骤。

1.  复制完成后，重新启动现有复制品。

1.  在新的复制品上，编辑配置并为新的复制品分配一个不被源或任何现有复制品使用的唯一服务器 ID（使用`server_id`系统变量）。

1.  启动新的复制服务器，确保通过指定`--skip-slave-start`选项或从 MySQL 8.0.24 开始，使用`skip_slave_start`系统变量，确保复制尚未开始。使用性能模式复制表或发出`SHOW REPLICA STATUS`来确认新的复制品与现有复制品相比是否具有正确的设置。还要显示服务器 ID 和服务器 UUID，并验证这些对于新的复制品是正确且唯一的。

1.  通过发出`START REPLICA`语句来启动复制线程。新的复制品现在使用其连接元数据存储库中的信息来启动复制过程。
