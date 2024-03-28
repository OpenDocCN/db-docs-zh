> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-multi-source-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-multi-source-configuration.html)

#### 19.1.5.1 配置多源复制

多源复制拓扑至少需要配置两个源和一个副本。在这些教程中，我们假设您有两个源`source1`和`source2`，以及一个副本`replicahost`。副本从每个源复制一个数据库，从`source1`复制`db1`，从`source2`复制`db2`。

多源复制拓扑中的源可以配置为使用基于 GTID 的复制，或者基于二进制日志位置的复制。查看第 19.1.3.4 节，“使用 GTIDs 设置复制”了解如何配置使用基于 GTID 的复制的源。查看第 19.1.2.1 节，“设置复制源配置”了解如何配置使用基于文件位置的复制的源。

多源复制拓扑中的副本需要`TABLE`存储库用于副本的连接元数据存储库和应用程序元数据存储库，这在 MySQL 8.0 中是默认的。多源复制与已弃用的替代文件存储库不兼容。

在所有副本上创建一个适当的用户帐户，副本可以使用该帐户进行连接。您可以在所有源上使用相同的帐户，或者在每个源上使用不同的帐户。如果您仅为复制目的创建一个帐户，则该帐户只需要`REPLICATION SLAVE`权限。例如，要设置一个名为`ted`的新用户，该用户可以从副本`replicahost`连接，请使用**mysql**客户端在每个源上发出以下语句：

```sql
mysql> CREATE USER 'ted'@'replicahost' IDENTIFIED BY '*password*';
mysql> GRANT REPLICATION SLAVE ON *.* TO 'ted'@'replicahost';
```

有关更多详细信息，以及关于 MySQL 8.0 新用户默认身份验证插件的重要信息，请参阅第 19.1.2.3 节，“为复制创建用户”。
