> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-snapshot-method.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-snapshot-method.html)

#### 19.1.2.5 选择数据快照方法

如果源数据库包含现有数据，则需要将这些数据复制到每个副本中。有不同的方法可以从源数据库中导出数据。以下部分描述了可能的选项。

要选择适当的数据库转储方法，请在以下选项之间进行选择：

+   使用**mysqldump**工具创建要复制的所有数据库的转储。这是推荐的方法，特别是在使用`InnoDB`时。

+   如果您的数据库存储在二进制可移植文件中，则可以将原始数据文件复制到副本中。这可能比使用**mysqldump**并在每个副本上导入文件更有效，因为在重放`INSERT`语句时跳过更新索引的开销。对于诸如`InnoDB`之类的存储引擎，不建议这样做。

+   使用 MySQL Server 的克隆插件将所有数据从现有副本传输到克隆副本。有关使用此方法的说明，请参见 Section 7.6.7.7, “Cloning for Replication”。

提示

要部署多个 MySQL 实例，可以使用 InnoDB Cluster，它使您能够轻松管理一组 MySQL 服务器实例在 MySQL Shell 中。InnoDB Cluster 在一个编程环境中包装了 MySQL Group Replication，使您可以轻松部署一组 MySQL 实例以实现高可用性。此外，InnoDB Cluster 与 MySQL Router 无缝接口，使您的应用程序可以连接到集群而无需编写自己的故障转移过程。然而，对于不需要高可用性的类似用例，您可以使用 InnoDB ReplicaSet。有关 MySQL Shell 的安装说明，请参见此处。

##### 19.1.2.5.1 使用 mysqldump 创建数据快照

要在现有源数据库中创建数据快照，请使用**mysqldump**工具。完成数据转储后，在启动复制过程之前将这些数据导入副本。

以下示例将所有数据库转储到名为`dbdump.db`的文件中，并包括`--master-data`选项，该选项会自动附加在副本上启动复制过程所需的`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句：

```sql
$> mysqldump --all-databases --master-data > dbdump.db
```

注意

如果不使用`--master-data`，则需要在单独的会话中手动锁定所有表。参见 Section 19.1.2.4, “Obtaining the Replication Source Binary Log Coordinates”。

可以使用**mysqldump**工具排除转储中的某些数据库。如果要选择要包含在转储中的数据库，请不要使用`--all-databases`。选择以下选项之一：

+   使用`--ignore-table`选项排除数据库中的所有表。

+   只命名要转储的数据库，使用`--databases`选项。

注意

默认情况下，如果源端使用 GTIDs（`gtid_mode=ON`），**mysqldump** 在转储输出中包含源端`gtid_executed`集合中的 GTIDs，以将其添加到副本端的`gtid_purged`集合中。如果只转储特定数据库或表，重要的是要注意，**mysqldump** 包含的值包括源端`gtid_executed`集合中的所有事务的 GTIDs，即使这些事务更改了数据库的被抑制部分，或者服务器上的其他未包含在部分转储中的数据库。查看 mysqldump 的`--set-gtid-purged`选项的描述，以找到您正在使用的 MySQL 服务器版本的默认行为结果，以及如何更改行为，如果此结果不适合您的情况。

有关更多信息，请参阅 Section 6.5.4, “mysqldump — A Database Backup Program”。

要导入数据，可以将转储文件复制到副本，或者在远程连接到副本时从源文件访问。

##### 19.1.2.5.2 使用原始数据文件创建数据快照

本节描述了如何使用组成数据库的原始文件创建数据快照。使用这种方法时，对于使用具有复杂缓存或日志算法的存储引擎的表，需要额外的步骤来生成完美的“时间点”快照：初始复制命令可能会遗漏缓存信息和日志更新，即使你已经获得了全局读锁。存储引擎对此的响应取决于其崩溃恢复能力。

如果使用`InnoDB`表，可以使用 MySQL Enterprise Backup 组件中的**mysqlbackup**命令生成一致的快照。该命令记录了与要在副本上使用的快照对应的日志名称和偏移量。MySQL Enterprise Backup 是作为 MySQL Enterprise 订阅的一部分包含的商业产品。详细信息请参见 Section 32.1, “MySQL Enterprise Backup Overview”。

这种方法在源和副本具有不同值`ft_stopword_file`、`ft_min_word_len`或`ft_max_word_len`时也无法可靠工作，如果你要复制具有全文索引的表。

假设上述例外情况不适用于你的数据库，使用冷备份技术获取`InnoDB`表的可靠二进制快照：对 MySQL Server 进行慢关闭，然后手动复制数据文件。

当你的 MySQL 数据文件存在于单个文件系统上时，可以使用标准文件复制工具如**cp**或**copy**，远程复制工具如**scp**或**rsync**，压缩工具如**zip**或**tar**，或文件系统快照工具如**dump**来创建`MyISAM`表的原始数据快照。如果只复制特定数据库，只复制与这些表相关的文件。对于`InnoDB`，除非启用了`innodb_file_per_table`选项，否则所有数据库中的所有表都存储在系统表空间文件中。

以下文件不需要用于复制：

+   与`mysql`数据库相关的文件。

+   副本的连接元数据存储库文件`master.info`，如果使用；现在已弃用此文件的使用（参见 Section 19.2.4, “Relay Log and Replication Metadata Repositories”）。

+   源的二进制日志文件，除非您打算使用它来定位副本的源二进制日志坐标时，不包括二进制日志索引文件。

+   任何中继日志文件。

根据您是否使用`InnoDB`表，选择以下操作之一：

如果您正在使用`InnoDB`表，并且为了获得最一致的原始数据快照结果，请在过程中关闭源服务器，具体操作如下：

1.  获取读锁并获取源的状态。参见第 19.1.2.4 节，“获取复制源二进制日志坐标”。

1.  在单独的会话中，关闭源服务器：

    ```sql
    $> mysqladmin shutdown
    ```

1.  复制 MySQL 数据文件。以下示例展示了常见的操作方式。您只需选择其中一种：

    ```sql
    $> tar cf */tmp/db.tar* *./data*
    $> zip -r */tmp/db.zip* *./data*
    $> rsync --recursive *./data* */tmp/dbdata*
    ```

1.  重新启动源服务器。

如果您没有使用`InnoDB`表，您可以从源获取系统快照而无需关闭服务器，具体步骤如下：

1.  获取读锁并获取源的状态。参见第 19.1.2.4 节，“获取复制源二进制日志坐标”。

1.  复制 MySQL 数据文件。以下示例展示了常见的操作方式。您只需选择其中一种：

    ```sql
    $> tar cf */tmp/db.tar* *./data*
    $> zip -r */tmp/db.zip* *./data*
    $> rsync --recursive *./data* */tmp/dbdata*
    ```

1.  在获取读锁的客户端中，释放锁：

    ```sql
    mysql> UNLOCK TABLES;
    ```

创建数据库的归档或副本后，在启动复制过程之前将文件复制到每个副本。
