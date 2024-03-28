> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-howto-repuser.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-repuser.html)

#### 19.1.2.3 为复制创建用户

每个副本使用 MySQL 用户名和密码连接到源端，因此必须在源端上有一个副本可以使用的用户帐户。当您设置副本时，通过`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前）指定用户名称。任何帐户都可以用于此操作，只要已授予`REPLICATION SLAVE`权限。您可以选择为每个副本创建不同的帐户，或者使用相同的帐户连接到源端以供每个副本使用。

尽管您不必专门为复制创建帐户，但您应该知道复制用户名和密码以明文形式存储在复制连接元数据存储库`mysql.slave_master_info`中（参见 Section 19.2.4.2，“复制元数据存储库”）。因此，您可能希望创建一个仅具有复制过程权限的单独帐户，以最大程度地减少其他帐户遭受威胁的可能性。

要创建一个新帐户，请使用`CREATE USER`。要为此帐户授予复制所需的权限，请使用`GRANT`语句。如果您仅为复制目的创建一个帐户，则该帐户只需要`REPLICATION SLAVE`权限。例如，要设置一个名为`repl`的新用户，该用户可以从`example.com`域内的任何主机连接进行复制，请在源端执行以下语句：

```sql
mysql> CREATE USER 'repl'@'%.example.com' IDENTIFIED BY '*password*';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%.example.com';
```

有关用户帐户操作语句的更多信息，请参见 Section 15.7.1，“帐户管理语句”。

重要提示

要使用使用`caching_sha2_password`插件进行身份验证的用户帐户连接到源，您必须按照 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”中描述的设置安装安全连接，或者启用不加密的连接以支持使用 RSA 密钥对进行密码交换。`caching_sha2_password`认证插件是从 MySQL 8.0 开始新用户的默认选项（有关详细信息，请参见 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”）。如果您创建或用于复制的用户帐户（由`MASTER_USER`选项指定）使用此认证插件，并且您没有使用安全连接，则必须启用基于 RSA 密钥对的密码交换以成功连接。
