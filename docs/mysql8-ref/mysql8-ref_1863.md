# 26.6.2 关于存储引擎的分区限制

> 译文链接：[`dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-storage-engines.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-limitations-storage-engines.html)

在 MySQL 8.0 中，分区支持实际上并非由 MySQL Server 提供，而是由表存储引擎自身或本地分区处理程序提供。在 MySQL 8.0 中，只有`InnoDB`和`NDB`存储引擎提供本地分区处理程序。这意味着分区表不能使用除这些之外的任何其他存储引擎来创建。（您必须使用带有`NDB`存储引擎的 MySQL NDB Cluster 来创建`NDB`表。）

**InnoDB 存储引擎。** `InnoDB`外键和 MySQL 分区不兼容。分区的`InnoDB`表不能具有外键引用，也不能具有被外键引用的列。具有或被外键引用的`InnoDB`表不能进行分区。

对于使用`InnoDB`的分区表，`ALTER TABLE ... OPTIMIZE PARTITION`无法正确工作。对于这些表，请改用`ALTER TABLE ... REBUILD PARTITION`和`ALTER TABLE ... ANALYZE PARTITION`。有关更多信息，请参见第 15.1.9.1 节，“ALTER TABLE Partition Operations”。

**用户定义的分区和 NDB 存储引擎（NDB Cluster）。** `KEY`（包括`LINEAR KEY`）分区是唯一支持的`NDB`存储引擎的分区类型。在 NDB Cluster 中，通常情况下不可能使用除[`LINEAR`] `KEY`之外的任何分区类型来创建 NDB Cluster 表，尝试这样做会导致错误。

*异常（不适用于生产）*：可以通过在 NDB Cluster SQL 节点上设置`new`系统变量为`ON`来覆盖此限制。如果您选择这样做，您应该意识到不支持使用除`[LINEAR] KEY`之外的其他分区类型的表进行生产。*在这种情况下，您可以创建和使用除`KEY`或`LINEAR KEY`之外的其他分区类型的表，但这完全是自担风险的。*您还应该意识到，此功能现已被弃用，并可能在将来的 NDB Cluster 版本中不经通知地被移除。

可以为`NDB` 表定义的最大分区数取决于集群中的数据节点和节点组数量、正在使用的 NDB Cluster 软件版本以及其他因素。有关更多信息，请参见 NDB 和用户定义的分区。

在`NDB`表中每个分区中可以存储的固定大小数据的最大量为 128 TB。以前，这个值是 16 GB。

会导致用户分区的`NDB` 表不满足以下两个要求之一或两者的`CREATE TABLE`和`ALTER TABLE`语句不被允许，并且会因错误而失败：

1.  表必须具有显式主键。

1.  表的分区表达式中列出的所有列必须是主键的一部分。

**异常。** 如果使用空列列表（即使用`PARTITION BY KEY()`或`PARTITION BY LINEAR KEY()`）创建用户分区的`NDB` 表，则不需要显式主键。

**分区选择。** `NDB` 表不支持分区选择。有关更多信息，请参见第 26.5 节“分区选择”。

**升级分区表。** 在执行升级时，使用`KEY`进行分区的表必须进行转储和重新加载。使用除`InnoDB`之外的存储引擎进行分区的表无法从 MySQL 5.7 或更早版本升级到 MySQL 8.0 或更高版本；您必须在升级之前使用`ALTER TABLE ... REMOVE PARTITIONING`删除这些表的分区，或者使用`ALTER TABLE ... ENGINE=INNODB`将其转换为`InnoDB`。

有关将`MyISAM`表转换为`InnoDB`的信息，请参见第 17.6.1.5 节“从 MyISAM 转换表到 InnoDB”。
