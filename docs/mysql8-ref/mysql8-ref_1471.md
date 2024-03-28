# 19.5.4 复制故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-problems.html)

如果您已按照说明操作，但复制设置无法正常工作，首先要做的是*检查错误日志中的消息*。许多用户在遇到问题后没有及时这样做而浪费了时间。

如果您无法从错误日志中确定问题所在，请尝试以下技术：

+   验证源是否启用了二进制日志记录，通过发出`SHOW MASTER STATUS`语句进行验证。二进制日志记录默认启用。如果启用了二进制日志记录，则`Position`不为零。如果未启用二进制日志记录，请验证您是否未使用任何禁用二进制日志记录的设置运行源，例如`--skip-log-bin`选项。

+   验证`server_id`系统变量在源和副本上启动时是否已设置，并且 ID 值在每台服务器上是唯一的。

+   验证副本是否正在运行。使用`SHOW REPLICA STATUS`检查`Replica_IO_Running`和`Replica_SQL_Running`的值是否都为`Yes`。如果不是，请验证启动副本服务器时使用的选项。例如，`--skip-slave-start`命令行选项，或者从 MySQL 8.0.24 开始，`skip_slave_start`系统变量，阻止复制线程启动，直到您发出`START REPLICA`语句。

+   如果副本正在运行，请检查它是否已经与源建立连接。使用`SHOW PROCESSLIST`，找到 I/O（接收器）和 SQL（应用程序）线程，并检查它们的`State`列以查看显示的内容。参见 Section 19.2.3, “Replication Threads”。如果接收器线程状态显示`Connecting to master`，请检查以下内容：

    +   验证源上复制用户的权限。

    +   检查源的主机名是否正确，并确保您使用正确的端口连接到源。用于复制的端口与用于客户端网络通信的端口相同（默认为`3306`）。对于主机名，请确保名称解析为正确的 IP 地址。

    +   检查配置文件，查看源或副本上是否启用了`skip_networking`系统变量以禁用网络。如果是，请注释该设置或将其删除。

    +   如果源有防火墙或 IP 过滤配置，请确保用于 MySQL 的网络端口未被过滤。

    +   通过使用`ping`或`traceroute`/`tracert`到达主机来检查是否可以访问源。

+   如果副本以前正在运行但已停止，则原因通常是在源上成功运行的某个语句在副本上失败。如果您已经正确地对源进行了快照，并且从未在复制线程之外修改副本上的数据，则不应该发生这种情况。如果副本意外停止，则是一个错误，或者您遇到了 Section 19.5.1, “Replication Features and Issues”中描述的已知复制限制之一。如果是错误，请参阅 Section 19.5.5, “How to Report Replication Bugs or Problems”，了解如何报告。

+   如果在源上成功运行的语句在副本上拒绝运行，请尝试以下步骤，如果不可行，则无法通过删除副本的数据库并从源复制新快照进行完整数据库重新同步：

    1.  确定副本上受影响的表是否与源表不同。尝试理解是如何发生的。然后使副本的表与源的表相同，并运行`START REPLICA`。

    1.  如果前面的步骤不起作用或不适用，请尝试理解是否可以安全地手动进行更新（如果需要），然后忽略源的下一个语句。

    1.  如果您决定副本可以跳过源的下一个语句，请发出以下语句：

        ```sql
        mysql> SET GLOBAL sql_slave_skip_counter = *N*;
        mysql> START SLAVE;

        Or from MySQL 8.0.26:
        mysql> SET GLOBAL sql_replica_skip_counter = *N*;
        mysql> START REPLICA;
        ```

        如果下一个来自源的语句不使用`AUTO_INCREMENT`或`LAST_INSERT_ID()`，则*`N`*的值应为 1。否则，该值应为 2。对于使用`AUTO_INCREMENT`或`LAST_INSERT_ID()`的语句使用值 2 的原因是它们在源的二进制日志中占据两个事件。

        参见 SET GLOBAL sql_slave_skip_counter 语法。

    1.  如果您确定副本最初与源完全同步，并且没有人在复制线程之外更新涉及的表，则差异可能是由错误引起的。如果您正在运行最新版本的 MySQL，请报告问题。如果您正在运行旧版本，请尝试升级到最新的生产版本以确定问题是否仍然存在。
