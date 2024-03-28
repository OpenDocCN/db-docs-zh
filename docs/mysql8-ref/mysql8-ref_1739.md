# 25.6.9 将数据导入到 MySQL 集群

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-importing-data.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-importing-data.html)

在设置新的 NDB 集群实例时，通常需要从现有 NDB 集群、MySQL 实例或其他来源导入数据。这些数据通常以以下一种或多种格式提供：

+   由**mysqldump**或**mysqlpump**生成的 SQL 转储文件。可以使用**mysql**客户端导入此文件，如本节后面所示。

+   由**mysqldump**或其他导出程序生成的 CSV 文件。这些文件可以使用**mysql**客户端中的`LOAD DATA INFILE`导入到`NDB`中，或者使用 NDB 集群分发的**ndb_import**实用程序。有关后者的更多信息，请参见第 25.5.13 节，“ndb_import — 将 CSV 数据导入 NDB”。

+   使用`START BACKUP`在`NDB`管理客户端中生成的本机`NDB`备份。要导入本机备份，必须使用作为 NDB 集群一部分提供的**ndb_restore**程序。有关使用此程序的更多信息，请参见第 25.5.23 节，“ndb_restore — 恢复 NDB 集群备份”。

从 SQL 文件导入数据时，通常不需要强制执行事务或外键，并临时禁用这些功能可以极大加快导入过程。可以使用**mysql**客户端来执行此操作，可以从客户端会话中执行，也可以在命令行上调用。在**mysql**客户端会话中，可以使用以下 SQL 语句执行导入：

```sql
SET ndb_use_transactions=0;
SET foreign_key_checks=0;

source *path/to/dumpfile*;

SET ndb_use_transactions=1;
SET foreign_key_checks=1;
```

以这种方式执行导入时，*必须*在执行**mysql**客户端的`source`命令后再次启用`ndb_use_transaction`和`foreign_key_checks`。否则，同一会话中后续语句可能也会在不执行事务或外键约束的情况下执行，这可能导致数据不一致。

从系统 shell 中，您可以使用 **mysql** 客户端的 `--init-command` 选项，在禁用事务和外键强制执行的情况下导入 SQL 文件，就像这样：

```sql
$> mysql --init-command='SET ndb_use_transactions=0; SET foreign_key_checks=0' < *path/to/dumpfile*
```

还可以将数据加载到一个 `InnoDB` 表中，然后使用 ALTER TABLE ... ENGINE NDB) 将其转换为使用 NDB 存储引擎。特别是对于许多表，您应该考虑到这可能需要多次操作；此外，如果使用外键，您必须仔细注意 `ALTER TABLE` 语句的顺序，因为外键在使用不同 MySQL 存储引擎的表之间不起作用。

您应该意识到，在本节中之前描述的方法并不针对非常大的数据集或大型事务进行优化。如果一个应用程序确实需要大型事务或许多并发事务作为正常操作的一部分，您可能希望增加 `MaxNoOfConcurrentOperations` 数据节点配置参数的值，这将保留更多内存以允许数据节点在其事务协调器意外停止时接管事务。

在执行 NDB Cluster 表的批量 `DELETE` 或 `UPDATE` 操作时，您可能也希望这样做。如果可能的话，尝试让应用程序以块的方式执行这些操作，例如，通过在这些语句中添加 `LIMIT`。

如果数据导入操作由于任何原因未能成功完成，您应该准备好执行任何必要的清理工作，包括可能一个或多个 `DROP TABLE` 语句，`DROP DATABASE` 语句，或两者都有。如果不这样做，可能会导致数据库处于不一致状态。
