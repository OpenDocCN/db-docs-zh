# 25.6.13 权限同步和 NDB_STORED_USER

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-privilege-synchronization.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-privilege-synchronization.html)

NDB 8.0 引入了一个新的机制，用于在连接到 NDB Cluster 的 SQL 节点之间共享和同步用户、角色和权限。这可以通过授予 `NDB_STORED_USER` 权限来启用。查看权限的描述以获取使用信息。

`NDB_STORED_USER` 在 `SHOW GRANTS` 的输出中与其他权限一样被打印出来，如下所示：

```sql
mysql> SHOW GRANTS for 'jon'@'localhost';
+---------------------------------------------------+
| Grants for jon@localhost                          |
+---------------------------------------------------+
| GRANT USAGE ON *.* TO `jon`@`localhost`           |
| GRANT NDB_STORED_USER ON *.* TO `jon`@`localhost` |
+---------------------------------------------------+
```

您还可以使用 NDB Cluster 提供的 **ndb_select_all** 实用程序验证此帐户的权限是否已共享，如下所示（一些输出已换行以保持格式）：

```sql
$> ndb_select_all -d mysql ndb_sql_metadata | grep '`jon`@`localhost`'
12      "'jon'@'localhost'"     0       [NULL]  "GRANT USAGE ON *.* TO `jon`@`localhost`"
11      "'jon'@'localhost'"     0       2       "CREATE USER `jon`@`localhost`
IDENTIFIED WITH 'caching_sha2_password' AS
0x2441243030352466014340225A107D590E6E653B5D587922306102716D752E6656772F3038512F
6C5072776D30376D37347A384B557A4C564F70495158656A31382E45324E33
REQUIRE NONE PASSWORD EXPIRE DEFAULT ACCOUNT UNLOCK PASSWORD HISTORY DEFAULT
PASSWORD REUSE INTERVAL DEFAULT PASSWORD REQUIRE CURRENT DEFAULT"
12      "'jon'@'localhost'"     1       [NULL]  "GRANT NDB_STORED_USER ON *.* TO `jon`@`localhost`"
```

`ndb_sql_metadata` 是一个特殊的 `NDB` 表，不能使用 **mysql** 或其他 MySQL 客户端看到。

授予 `NDB_STORED_USER` 权限的语句，例如 `GRANT NDB_STORED_USER ON *.* TO 'cluster_app_user'@'localhost'`，通过指示 `NDB` 使用查询 `SHOW CREATE USER cluster_app_user@localhost` 和 `SHOW GRANTS FOR cluster_app_user@localhost` 来创建一个快照，然后将结果存储在 `ndb_sql_metadata` 中。然后请求任何其他 SQL 节点读取和应用该快照。每当 MySQL 服务器启动并作为 SQL 节点加入集群时，它会执行这些存储的 `CREATE USER` 和 `GRANT` 语句作为集群模式同步过程的一部分。

每当在非原始 SQL 节点上执行 SQL 语句时，该语句将在 `NDBCLUSTER` 存储引擎的实用线程中运行；这是在与 MySQL 复制副本应用程序线程相同的安全环境中完成的。

从 NDB 8.0.27 开始，执行对用户权限的更改的 SQL 节点在执行之前会获取全局锁，这可以防止不同 SQL 节点上的并发 ACL 操作导致死锁。在 NDB 8.0.27 之前，对具有 `NDB_STORED_USER` 的用户的更改是完全异步更新的，没有任何锁被获取。

请记住，由于共享模式更改操作是同步执行的，因此在更改任何共享用户或用户后，下一个共享模式更改将作为同步点。在模式更改分发开始之前，任何待处理的用户更改都会完成；之后模式更改本身会同步运行。例如，如果一个`DROP DATABASE`语句跟随一个`DROP USER`对分布式用户的操作，那么在所有 SQL 节点上用户的删除完成之前，数据库的删除无法进行。

如果来自多个 SQL 节点的多个`GRANT`、`REVOKE`或其他用户管理语句导致给定用户在不同 SQL 节点上的权限不一致，您可以通过在已知权限正确的 SQL 节点上为该用户发出`GRANT NDB_STORED_USER`来解决此问题；这将导致对权限的新快照被获取并同步到其他 SQL 节点。

NDB Cluster 8.0 不支持通过更改 MySQL 权限表使其使用`NDB`存储引擎来在 NDB Cluster 中的 SQL 节点之间分发 MySQL 用户和权限，就像在 NDB 7.6 和之前的版本中一样（参见 Distributed Privileges Using Shared Grant Tables）。有关此更改对从先前版本升级到 NDB 8.0 的影响的信息，请参见 Section 25.3.7, “Upgrading and Downgrading NDB Cluster”。
