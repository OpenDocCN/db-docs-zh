> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dictionary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-dictionary-tables.html)

#### 25.6.16.23 ndbinfo dictionary_tables 表

此表为`NDB`表提供`NDB`字典信息。`dictionary_tables`包含以下列：

+   `table_id`

    表的唯一 ID

+   `database_name`

    包含表的数据库的名称

+   `table_name`

    表的名称

+   `status`

    表状态；其中之一为`New`、`Changed`、`Retrieved`、`Invalid`或`Altered`。（有关对象状态值的更多信息，请参阅 Object::Status。）

+   `attributes`

    表属性数

+   `primary_key_cols`

    表主键中的列数

+   `primary_key`

    表主键中列的逗号分隔列表

+   `storage`

    表使用的存储类型；其中之一为`memory`、`disk`或`default`

+   `logging`

    是否为此表启用了日志记录

+   `dynamic`

    如果表是动态的，则为`1`；否则为`0`；如果`*`table`*->``getForceVarPart()`为真，或者至少一个表列是动态的，则表被视为动态的

+   `read_backup`

    如果从任何副本读取（为此表启用了`READ_BACKUP`选项，则为`1`，否则为`0`；请参阅 Section 15.1.20.12, “Setting NDB Comment Options”)

+   `fully_replicated`

    如果为此表启用了`FULLY_REPLICATED`（集群中的每个数据节点都有表的完整副本），则为`1`；否则为`0`；请参阅 Section 15.1.20.12, “Setting NDB Comment Options”

+   `checksum`

    如果此表使用校验和，则此列中的值为`1`；否则为`0`

+   `row_size`

    可以存储在一行中的数据量（以字节为单位），不包括单独存���在 blob 表中的任何 blob 数据；请参阅 Table::getRowSizeInBytes()，API 文档中获取更多信息

+   `min_rows`

    用于计算分区的最小行数；请参阅 Table::getMinRows()，API 文档中获取更多信息

+   `max_rows`

    用于计算分区的最大行数；请参阅 Table::getMaxRows()，API 文档中获取更多信息

+   `tablespace`

    表所属的表空间的 ID，如果表不使用磁盘上的数据，则为`0`

+   `fragment_type`

    表的片段类型；其中之一为`Single`、`AllSmall`、`AllMedium`、`AllLarge`、`DistrKeyHash`、`DistrKeyLin`、`UserDefined`、`unused`或`HashMapPartition`；有关更多信息，请参阅 Object::FragmentType，NDB API 文档中

+   `hash_map`

    表使用的哈希映射

+   `fragments`

    表片段数

+   `partitions`

    表使用的分区数

+   `partition_balance`

    使用的分区平衡类型，如果有的话；可以是`FOR_RP_BY_NODE`、`FOR_RA_BY_NODE`、`FOR_RP_BY_LDM`、`FOR_RA_BY_LDM`、`FOR_RA_BY_LDM_X_2`、`FOR_RA_BY_LDM_X_3`或`FOR_RA_BY_LDM_X_4`中的一个；参见第 15.1.20.12 节，“设置 NDB 注释选项”

+   `contains_GCI`

    如果表包含全局检查点索引，则为`1`，否则为`0`

+   `single_user_mode`

    单用户模式生效时允许对表的访问类型；可以是`locked`、`read_only`或`read_write`中的一个；这些等同于 NDB API 中的`Table::SingleUserMode`类型的`SingleUserModeLocked`、`SingleUserModeReadOnly`和`SingleUserModeReadWrite`值

+   `force_var_part`

    如果`*`table`*->``getForceVarPart()`对于此表为真，则为`1`，否则为`0`

+   `GCI_bits`

    用于测试

+   `author_bits`

    用于测试

`dictionary_tables`表在 NDB 8.0.29 中添加。
