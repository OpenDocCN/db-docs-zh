# 20.6.1 连接安全管理的通信堆栈

> 译文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-connection-security.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-connection-security.html)

从 MySQL 8.0.27 开始，Group Replication 可以通过以下方法之一保护成员之间的组通信连接：

+   使用其自身的安全协议实现，包括 TLS/SSL 和用于传入组通信系统（GCS）连接的白名单。这是 MySQL 8.0.26 及更早版本的唯一选项。

+   使用 MySQL 服务器自身的连接安全代替 Group Replication 的实现。使用 MySQL 协议意味着可以使用标准的用户认证方法来授予（或撤销）对组的访问权限，而不是使用白名单，并且服务器协议的最新功能始终在发布时可用。此选项从 MySQL 8.0.27 开始提供。

通过设置系统变量`group_replication_communication_stack`为`XCOM`来选择使用 Group Replication 的自身实现（这是默认选择），或者设置为`MYSQL`以使用 MySQL 服务器的连接安全。

复制组要使用 MySQL 通信堆栈，需要进行以下额外配置。当您从使用 XCom 通信堆栈切换到 MySQL 通信堆栈时，特别重要的是确保所有这些要求都得到满足。

**MySQL 通信堆栈的 Group Replication 要求**

+   由`group_replication_local_address`系统变量配置的网络地址必须设置为 MySQL 服务器正在侦听的 IP 地址和端口之一，如服务器的`bind_address`系统变量所指定的那样。每个成员的 IP 地址和端口组合在组内必须是唯一的。建议为每个组成员配置`group_replication_group_seeds`系统变量，其中包含所有组成员的本地地址。

+   MySQL 通信栈支持网络命名空间，而 XCom 通信栈不支持。如果使用网络命名空间与组复制的本地地址（`group_replication_local_address`）一起使用，必须为每个组成员使用`CHANGE REPLICATION SOURCE TO`语句进行配置。此外，每个组成员的`report_host`服务器系统变量必须设置为报告命名空间。所有组成员必须使用相同的命名空间，以避免在分布式恢复期间可能出现的地址解析问题。

+   `group_replication_ssl_mode`系统变量必须设置为组通信所需的设置。此系统变量控制组通信是否启用或禁用 TLS/SSL。对于 MySQL 8.0.26 及更早版本，TLS/SSL 配置始终从服务器的 SSL 设置中获取；对于 MySQL 8.0.27 及更高版本，在使用 MySQL 通信栈时，TLS/SSL 配置将从组复制的分布式恢复设置中获取。为避免潜在冲突，此设置应在所有组成员上相同。

+   `--ssl`或`--skip-ssl`服务器选项和`require_secure_transport`服务器系统变量的设置应在所有组成员上相同，以避免潜在冲突。如果`group_replication_ssl_mode`设置为`REQUIRED`、`VERIFY_CA`或`VERIFY_IDENTITY`，请使用`--ssl`和`require_secure_transport=ON`。如果`group_replication_ssl_mode`设置为`DISABLED`，请使用`require_secure_transport=OFF`。

+   如果为组通信启用了 TLS/SSL，则必须配置 Group Replication 的用于保护分布式恢复的设置，如果尚未配置，则必须配置，或者如果已经配置，则必须进行验证。MySQL 通信堆栈不仅在成员之间的分布式恢复连接中使用这些设置，还在一般组通信中使用 TLS/SSL 配置。`group_replication_recovery_use_ssl`和其他`group_replication_recovery_*`系统变量在第 20.6.3.2 节，“用于分布式恢复的安全套接字层（SSL）连接”中有解释。

+   当组使用 MySQL 通信堆栈时，不使用 Group Replication 白名单，因此`group_replication_ip_allowlist`和`group_replication_ip_whitelist`系统变量将被忽略，无需配置。

+   Group Replication 用于分布式恢复的复制用户帐户，如使用`CHANGE REPLICATION SOURCE TO`语句配置的，用于在设置 Group Replication 连接时由 MySQL 通信堆栈进行身份验证。这个用户帐户在所有组成员上都是相同的，必须具有以下权限：

    +   `GROUP_REPLICATION_STREAM`。此权限是用户帐户能够使用 MySQL 通信堆栈建立 Group Replication 连接所必需的。

    +   `CONNECTION_ADMIN`。此权限是为了确保 Group Replication 连接在涉及的服务器之一处于离线模式时不会被终止。如果使用 MySQL 通信堆栈而没有此权限，则被放置在离线模式的成员将被从组中驱逐。

    这些权限是所有复制用户帐户必须具有的权限`REPLICATION SLAVE`和`BACKUP_ADMIN`的附加权限（请参阅第 20.2.1.3 节，“用于分布式恢复的用户凭据”）。在添加新权限时，请确保在发出`GRANT`语句之前通过发出`SET SQL_LOG_BIN=0`跳过每个组成员上的二进制日志记录，并在之后通过`SET SQL_LOG_BIN=1`，以便本地事务不会干扰 Group Replication 的重新启动。

`group_replication_communication_stack`实际上是一个群组范围的配置设置，所有群组成员必须设置相同的值。然而，Group Replication 自身并不检查群组范围配置设置的一致性。一个值与其他群组成员不同的成员无法与其他成员通信，因为通信协议不兼容，所以无法交换关于其配置设置的信息。

这意味着虽然在 Group Replication 运行时可以更改系统变量的值，并在重新启动群组成员上的 Group Replication 后生效，但成员仍然无法重新加入群组，直到所有成员的设置都已更改。因此，您必须在所有成员上停止 Group Replication 并更改系统变量的值，然后才能重新启动群组。因为所有成员都已停止，所以需要通过一个具有`group_replication_bootstrap_group=ON`的服务器进行完整重启群组（引导）。在值更改生效之前，您可以在停止的群组成员上进行其他必要的设置更改。

对于一个运行中的群组，按照以下步骤更改`group_replication_communication_stack`的值和其他必要的设置，以将一个群组从 XCom 通信栈迁移到 MySQL 通信栈，或者从 MySQL 通信栈迁移到 XCom 通信栈：

1.  在每个群组成员上停止 Group Replication，使用`STOP GROUP_REPLICATION`语句。最后停止主要成员，以免触发新的主要选举并等待其完成。

1.  在每个群组成员上，将系统变量`group_replication_communication_stack`设置为新的通信栈，`MYSQL`或`XCOM`，具体取决于情况。您可以通过编辑 MySQL 服务器配置文件（在 Linux 和 Unix 系统上通常命名为`my.cnf`，在 Windows 系统上为`my.ini`），或使用`SET`语句来完成。例如：

    ```sql
    SET PERSIST group_replication_communication_stack="MYSQL";
    ```

1.  如果您正在将复制组从 XCom 通信堆栈（默认）迁移到 MySQL 通信堆栈，请在每个组成员上配置或重新配置所需的系统变量为适当的设置，如上述清单中所述。例如，`group_replication_local_address`系统变量必须设置为 MySQL 服务器正在侦听的 IP 地址和端口之一。还要使用`CHANGE REPLICATION SOURCE TO`语句配置任何网络命名空间。

1.  如果您正在将复制组从 XCom 通信堆栈（默认）迁移到 MySQL 通信堆栈，请在每个组成员上发出`GRANT`语句，以赋予复制用户帐户`GROUP_REPLICATION_STREAM`和`CONNECTION_ADMIN`权限。您需要将组成员从应用 Group Replication 停止时应用的只读状态中取出。还要确保在发出`GRANT`语句之前在每个组成员上跳过二进制日志记录，通过在发出语句之前执行`SET SQL_LOG_BIN=0`，并在之后执行`SET SQL_LOG_BIN=1`，以确保本地事务不会干扰重新启动 Group Replication。例如：

    ```sql
    SET GLOBAL SUPER_READ_ONLY=OFF;
    SET SQL_LOG_BIN=0; 
    GRANT GROUP_REPLICATION_STREAM ON *.* TO rpl_user@'%';
    GRANT CONNECTION_ADMIN ON *.* TO rpl_user@'%';
    SET SQL_LOG_BIN=1;
    ```

1.  如果您正在将复制组从 MySQL 通信堆栈迁移回 XCom 通信堆栈，请在每个组成员上重新配置上述要求中的系统变量，以适合 XCom 通信堆栈的设置。第 20.9 节，“组复制变量”列出了系统变量及其默认值以及 XCom 通信堆栈的要求。

    注意

    +   XCom 通信堆栈不支持网络命名空间，因此组复制本地地址（`group_replication_local_address`系统变量）不能使用这些。通过发出`CHANGE REPLICATION SOURCE TO`语句取消设置。

    +   当切换回 XCom 通信堆栈时，由`group_replication_recovery_use_ssl`和其他`group_replication_recovery_*`系统变量指定的设置不用于保护组通信。相反，组复制系统变量`group_replication_ssl_mode`用于激活 SSL 用于组通信连接并指定连接的安全模式，其余配置取自服务器的 SSL 配置。有关详细信息，请参见第 20.6.2 节“使用安全套接字层（SSL）保护组通信连接”。

1.  要重新启动组，请按照第 20.5.2 节“重新启动组”中的过程进行操作，该节解释了如何安全地引导已执行和认证事务的组。由具有`group_replication_bootstrap_group=ON`的服务器引导是必要的，以更改通信堆栈，因为所有成员必须关闭。

1.  成员现在使用新的通信堆栈相互连接。任何将`group_replication_communication_stack`设置为先前通信堆栈（或在 XCom 的情况下默认设置）的服务器将无法加入该组。重要的是要注意，由于组复制甚至无法看到加入尝试，因此它不会检查并用错误消息拒绝加入成员。相反，当先前的通信堆栈放弃尝试联系新的通信堆栈时，尝试加入会悄无声息地失败。
