> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-user-credentials.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-user-credentials.html)

#### 20.2.1.3 分布式恢复的用户凭据

组复制使用分布式恢复过程在加入组时同步组成员。分布式恢复涉及将捐赠者的二进制日志中的事务传输到加入成员，使用名为`group_replication_recovery`的复制通道。因此，您必须设置具有正确权限的复制用户，以便组复制可以建立直接成员之间的复制通道。如果组成员已设置为支持远程克隆操作作为分布式恢复的一部分，该操作从 MySQL 8.0.17 开始提供，那么此复制用户也用作捐赠者上的克隆用户，并且对于此角色也需要正确的权限。有关分布式恢复的完整描述，请参见第 20.5.4 节，“分布式恢复”。

每个组成员上的分布式恢复必须使用相同的复制用户。可以在二进制日志中记录为分布式恢复创建复制用户的过程，然后依靠分布式恢复来复制用于创建用户的语句。或者，您可以在创建复制用户之前禁用二进制日志记录，然后在每个成员上手动创建用户，例如，如果您想避免更改传播到其他服务器实例。如果这样做，请确保在配置用户后重新启用二进制日志记录。

重要提示

如果您的组的分布式恢复连接使用 SSL，那么在加入成员连接到捐赠者之前，必须在每台服务器上创建复制用户。有关为分布式恢复连接设置 SSL 并创建需要 SSL 的复制用户的说明，请参见第 20.6.3 节，“保护分布式恢复连接”

重要提示

默认情况下，在 MySQL 8 中创建的用户使用第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。如果用于分布式恢复的复制用户使用缓存 SHA-2 认证插件，并且*不*在分布式恢复连接中使用 SSL，那么 RSA 密钥对用于密码交换。您可以将复制用户的公钥复制到加入成员，或者在请求时配置捐赠者提供公钥。有关如何执行此操作的说明，请参见第 20.6.3.1 节，“分布式恢复的安全用户凭据”。

要为分布式恢复创建复制用户，请按照以下步骤进行：

1.  启动 MySQL 服务器实例，然后连接客户端。

1.  如果要禁用二进制日志记录，以便在每个实例上单独创建复制用户，请执行以下语句：

    ```sql
    mysql> SET SQL_LOG_BIN=0;
    ```

1.  创建一个具有以下权限的 MySQL 用户：

    +   `REPLICATION SLAVE`，用于与捐赠者建立分布式恢复连接以检索数据。

    +   `CONNECTION_ADMIN`，确保如果涉及的服务器之一被置于脱机模式，则不会终止 Group Replication 连接。

    +   `BACKUP_ADMIN`，如果复制组中的服务器已设置为支持克隆（参见 Section 20.5.4.2, “Cloning for Distributed Recovery”）。此权限在分布式恢复的克隆操作中，需要成员作为捐赠者时才需要。

    +   `GROUP_REPLICATION_STREAM`，如果 MySQL 通信堆栈用于复制组（参见 Section 20.6.1, “Communication Stack for Connection Security Management”）。此权限要求用户帐户能够使用 MySQL 通信堆栈建立和维护 Group Replication 的连接。

    在此示例中显示了用户*`rpl_user`*和密码*`password`*。在配置服务器时，请使用适当的用户名和密码：

    ```sql
    mysql> CREATE USER *rpl_user*@'%' IDENTIFIED BY '*password*';
    mysql> GRANT REPLICATION SLAVE ON *.* TO *rpl_user*@'%';
    mysql> GRANT CONNECTION_ADMIN ON *.* TO *rpl_user*@'%';
    mysql> GRANT BACKUP_ADMIN ON *.* TO *rpl_user*@'%';
    mysql> GRANT GROUP_REPLICATION_STREAM ON *.* TO *rpl_user*@'%';
    mysql> FLUSH PRIVILEGES;
    ```

1.  如果您禁用了二进制日志记录，请在创建用户后立即通过以下语句启用它：

    ```sql
    mysql> SET SQL_LOG_BIN=1;
    ```

1.  创建复制用户后，必须向服务器提供用户凭据，以供分布式恢复使用。您可以通过将用户凭据设置为`group_replication_recovery`通道的凭据，使用`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（MySQL 8.0.23 之前）。或者，从 MySQL 8.0.21 开始，您可以在`START GROUP_REPLICATION`语句中指定分布式恢复的用户凭据。

    +   使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`设置的用户凭据以明文形式存储在服务器上的复制元数据存储库中。它们在启动 Group Replication 时应用，包括如果`group_replication_start_on_boot`系统变量设置为`ON`时的自动启动。

    +   在`START GROUP_REPLICATION`中指定的用户凭据仅保存在内存中，并且通过`STOP GROUP_REPLICATION`语句或服务器关闭时会被删除。您必须发出`START GROUP_REPLICATION`语句再次提供凭据，因此不能使用这些凭据自动启动 Group Replication。通过指定用户凭据的这种方法有助于保护 Group Replication 服务器免受未经授权的访问。

    有关使用每种提供用户凭据方法的安全影响的更多信息，请参阅 Section 20.6.3.1.3, “提供复制用户凭据的安全性”。如果选择使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句提供用户凭据，请立即在服务器实例上发出以下语句，将*`rpl_user`*和*`password`*替换为创建用户时使用的值：

    ```sql
    mysql> CHANGE MASTER TO MASTER_USER='*rpl_user*', MASTER_PASSWORD='*password*' \\
    		      FOR CHANNEL 'group_replication_recovery';

    Or from MySQL 8.0.23:
    mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='*rpl_user*', SOURCE_PASSWORD='*password*' \\
    		      FOR CHANNEL 'group_replication_recovery';
    ```
