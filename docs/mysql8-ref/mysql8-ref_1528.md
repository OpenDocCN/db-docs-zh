> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-secure-user.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-secure-user.html)

#### 20.6.3.1 为分布式恢复安全用户凭据

从二进制日志进行状态传输需要具有正确权限的复制用户，以便 Group Replication 可以建立直接成员之间的复制通道。相同的复制用户用于所有组成员的分布式恢复。如果组成员已设置为支持作为分布式恢复的一部分的远程克隆操作（从 MySQL 8.0.17 开始提供），则此复制用户也用作捐赠者上的克隆用户，并且对于此角色也需要正确的权限。有关设置此用户的详细说明，请参阅第 20.2.1.3 节，“分布式恢复的用户凭据”。

为了保护用户凭据，您可以要求用户帐户的连接使用 SSL，并且（从 MySQL 8.0.21 开始）可以在启动 Group Replication 时提供用户凭据，而不是将它们存储在副本状态表中。此外，如果您使用缓存 SHA-2 认证，您必须在组成员上设置 RSA 密钥对。

重要

当使用 MySQL 通信堆栈（`group_replication_communication_stack=MYSQL`）和成员之间的安全连接（`group_replication_ssl_mode`未设置为`DISABLED`）时，恢复用户必须正确设置，因为他们也是组通信的用户。请按照第 20.6.3.1.2 节，“具有 SSL 的复制用户”和第 20.6.3.1.3 节，“安全提供复制用户凭据”中的说明进行操作。

##### 20.6.3.1.1 具有缓存 SHA-2 认证插件的复制用户

默认情况下，在 MySQL 8 中创建的用户使用第 8.4.1.2 节，“缓存 SHA-2 可插拔认证”。如果您为分布式恢复配置的复制用户使用缓存 SHA-2 认证插件，并且*不*在分布式恢复连接中使用 SSL，那么 RSA 密钥对将用于密码交换。有关 RSA 密钥对的更多信息，请参阅第 8.3.3 节，“创建 SSL 和 RSA 证书和密钥”。

在这种情况下，您可以将`rpl_user`的公钥复制到加入成员，或者配置捐赠者在请求时提供公钥。更安全的方法是将复制用户帐户的公钥复制到加入成员。然后，您需要在加入成员上配置`group_replication_recovery_public_key_path`系统变量，指定复制用户帐户的公钥路径。

较不安全的方法是在捐赠者上设置`group_replication_recovery_get_public_key=ON`，以便它们向加入成员提供复制用户帐户的公钥。没有办法验证服务器的身份，因此只有在确定没有服务器身份被破坏的风险时才设置`group_replication_recovery_get_public_key=ON`，例如通过中间人攻击。

##### 20.6.3.1.2 带 SSL 的复制用户

需要 SSL 连接的复制用户必须在服务器加入组（加入成员）连接到捐赠者之前创建。通常，在为服务器加入组进行配置时设置。要为需要 SSL 连接的分布式恢复创建一个复制用户，请在所有将参与组的服务器上发出以下语句：

```sql
mysql> SET SQL_LOG_BIN=0;
mysql> CREATE USER '*rec_ssl_user*'@'%' IDENTIFIED BY '*password*' REQUIRE SSL;
mysql> GRANT REPLICATION SLAVE ON *.* TO '*rec_ssl_user*'@'%';
mysql> GRANT CONNECTION_ADMIN ON *.* TO '*rec_ssl_user*'@'%';
mysql> GRANT BACKUP_ADMIN ON *.* TO '*rec_ssl_user*'@'%';
mysql> GRANT GROUP_REPLICATION_STREAM ON *.* TO *rec_ssl_user*@'%';
mysql> FLUSH PRIVILEGES;
mysql> SET SQL_LOG_BIN=1;
```

注意

当使用 MySQL 通信堆栈（`group_replication_communication_stack=MYSQL`）和成员之间的安全连接（`group_replication_ssl_mode`未设置为`DISABLED`）时，需要`GROUP_REPLICATION_STREAM`权限。参见第 20.6.1 节，“连接安全管理的通信堆栈”。

##### 20.6.3.1.3 安全提供复制用户凭据

要为复制用户提供用户凭据，您可以将它们永久设置为`group_replication_recovery`通道的凭据，使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句。另外，从 MySQL 8.0.21 开始，您可以在每次启动群组复制时在`START GROUP_REPLICATION`语句中指定它们。在`START GROUP_REPLICATION`上指定的用户凭据优先于使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句设置的任何用户凭据。

使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`设置的用户凭据以明文形式存储在服务器上的复制元数据存储库中，但在`START GROUP_REPLICATION`中指定的用户凭据仅保存在内存中，并且通过`STOP GROUP_REPLICATION`语句或服务器关闭时会被删除。因此，使用`START GROUP_REPLICATION`指定用户凭据有助于保护群组复制服务器免受未经授权的访问。然而，这种方法与通过`group_replication_start_on_boot`系统变量指定自动启动群组复制不兼容。

如果要永久设置用户凭据，请在准备加入群组的成员上发出`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句：

```sql
mysql> CHANGE MASTER TO MASTER_USER='*rec_ssl_user*', MASTER_PASSWORD='*password*' 
            FOR CHANNEL 'group_replication_recovery';

Or from MySQL 8.0.23:
mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='*rec_ssl_user*', SOURCE_PASSWORD='*password*' 
            FOR CHANNEL 'group_replication_recovery';
```

在首次启动群组复制或服务器重新启动时，要在`START GROUP_REPLICATION`上提供用户凭据，请发出以下语句：

```sql
mysql> START GROUP_REPLICATION USER='*rec_ssl_user*', PASSWORD='*password*';
```

重要提示

如果您切换到使用`START GROUP_REPLICATION`在以前使用`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`提供凭据的服务器上指定用户凭据，您必须完成以下步骤以获得此更改的安全性好处。

1.  使用`STOP GROUP_REPLICATION`语句在组成员上停止组复制。虽然在组复制运行时可以执行以下两个步骤，但需要重新启动组复制以实施更改。

1.  将`group_replication_start_on_boot`系统变量的值设置为`OFF`（默认为`ON`）。

1.  通过发出以下语句从副本状态表中删除分布式恢复凭据：

    ```sql
    mysql> CHANGE MASTER TO MASTER_USER='', MASTER_PASSWORD='' 
                FOR CHANNEL 'group_replication_recovery';

    Or from MySQL 8.0.23:
    mysql> CHANGE REPLICATION SOURCE TO SOURCE_USER='', SOURCE_PASSWORD='' 
                FOR CHANNEL 'group_replication_recovery';
    ```

1.  使用指定分布式恢复用户凭据的`START GROUP_REPLICATION`语句在组成员上重新启动组复制。

如果不执行这些步骤，凭据将继续存储在副本状态表中，并且在远程克隆操作期间也可以传输到其他组成员，用于分布式恢复。然后，`group_replication_recovery`通道可能会意外地使用存储的凭据在原始成员或从原始成员克隆的成员上启动组复制。服务器启动时（包括远程克隆操作后）自动启动组复制将使用存储的用户凭据，如果操作员未在`START GROUP_REPLICATION`中指定分布式恢复凭据，则也会使用这些凭据。
