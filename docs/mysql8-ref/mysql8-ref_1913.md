# 28.3.18 信息模式 ndb_transid_mysql_connection_map 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-ndb-transid-mysql-connection-map-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-ndb-transid-mysql-connection-map-table.html)

`ndb_transid_mysql_connection_map`表提供了`NDB`事务、`NDB`事务协调器和连接到 NDB 集群作为 API 节点的 MySQL 服务器之间的映射。在填充`server_operations`和`server_transactions`表时，将使用此信息`ndbinfo` NDB 集群信息数据库。

| `INFORMATION_SCHEMA`名称 | `SHOW`名称 | 备注 |
| --- | --- | --- |
| `mysql_connection_id` |  | MySQL 服务器连接 ID |
| `node_id` |  | 事务协调器节点 ID |
| `ndb_transid` |  | `NDB`事务 ID |

`mysql_connection_id`与`SHOW PROCESSLIST`输出中显示的连接或会话 ID 相同。

与此表相关联的`SHOW`语句不存在。

这是一个非标准表，特定于 NDB 集群。它作为一个`INFORMATION_SCHEMA`插件实现；您可以通过检查`SHOW PLUGINS`的输出来验证它是否受支持。如果启用了`ndb_transid_mysql_connection_map`支持，此语句的输出将包括一个具有此名称的插件，类型为`INFORMATION SCHEMA`，并且状态为`ACTIVE`，如下所示（使用强调文本）：

```sql
mysql> SHOW PLUGINS;
+----------------------------------+--------+--------------------+---------+---------+
| Name                             | Status | Type               | Library | License |
+----------------------------------+--------+--------------------+---------+---------+
| binlog                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| mysql_native_password            | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| sha256_password                  | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| caching_sha2_password            | ACTIVE | AUTHENTICATION     | NULL    | GPL     |
| sha2_cache_cleaner               | ACTIVE | AUDIT              | NULL    | GPL     |
| daemon_keyring_proxy_plugin      | ACTIVE | DAEMON             | NULL    | GPL     |
| CSV                              | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| MEMORY                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| InnoDB                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| INNODB_TRX                       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| INNODB_CMP                       | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |

...

| INNODB_SESSION_TEMP_TABLESPACES  | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |
| MyISAM                           | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| MRG_MYISAM                       | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| PERFORMANCE_SCHEMA               | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| TempTable                        | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| ARCHIVE                          | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| BLACKHOLE                        | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| ndbcluster                       | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
| ndbinfo                          | ACTIVE | STORAGE ENGINE     | NULL    | GPL     |
*| ndb_transid_mysql_connection_map | ACTIVE | INFORMATION SCHEMA | NULL    | GPL     |*
| ngram                            | ACTIVE | FTPARSER           | NULL    | GPL     |
| mysqlx_cache_cleaner             | ACTIVE | AUDIT              | NULL    | GPL     |
| mysqlx                           | ACTIVE | DAEMON             | NULL    | GPL     |
+----------------------------------+--------+--------------------+---------+---------+
47 rows in set (0.01 sec)
```

插件默认启用。您可以通过使用`--ndb-transid-mysql-connection-map`选项启动服务器来禁用它（或者强制服务器不运行，除非插件启动）。如果插件被禁用，状态将显示为`SHOW PLUGINS`中的`DISABLED`。插件无法在运行时启用或禁用。

尽管此表及其列的名称以小写显示，但在 SQL 语句中引用它们时，可以使用大写或小写。

要创建此表，MySQL 服务器必须是与 NDB 集群分发一起提供的二进制文件，或者是从启用了`NDB`存储引擎支持的 NDB 集群源代码构建的。它在标准 MySQL 8.0 服务器中不可用。
