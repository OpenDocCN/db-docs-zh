# 28.3.21 INFORMATION_SCHEMA PARTITIONS 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-partitions-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-partitions-table.html)

`PARTITIONS` 表提供有关表分区的信息。该表中的每一行对应��分区表的单个分区或子分区。有关分区表的更多信息，请参阅 第二十六章，“分区”。

`PARTITIONS` 表包含以下列：

+   `TABLE_CATALOG`

    表所属的目录的名称。该值始终为 `def`。

+   `TABLE_SCHEMA`

    表所属模式（数据库）的名称。

+   `TABLE_NAME`

    包含分区的表的名称。

+   `PARTITION_NAME`

    分区的名称。

+   `SUBPARTITION_NAME`

    如果 `PARTITIONS` 表行表示子分区，则为子分区的名称；否则为 `NULL`。

    对于 `NDB`：该值始终为 `NULL`。

+   `PARTITION_ORDINAL_POSITION`

    所有分区按照定义的顺序进行索引，其中 `1` 是分配给第一个分区的编号。随着分区的添加、删除和重新组织，索引可能会发生变化；此列中显示的编号反映了当前顺序，考虑到任何索引更改。

+   `SUBPARTITION_ORDINAL_POSITION`

    在给定分区内的子分区也按照表内分区的方式进行索引和重新索引。

+   `PARTITION_METHOD`

    值为 `RANGE`、`LIST`、`HASH`、`LINEAR HASH`、`KEY` 或 `LINEAR KEY` 中的一个；即 26.2 节，“分区类型” 中讨论的可用分区类型之一。

+   `SUBPARTITION_METHOD`

    值为 `HASH`、`LINEAR HASH`、`KEY` 或 `LINEAR KEY` 中的一个；即 26.2.6 节，“子分区” 中讨论的可用子分区类型之一。

+   `PARTITION_EXPRESSION`

    在创建表的 `CREATE TABLE` 或 `ALTER TABLE` 语句中使用的分区函数的表达式，该表达式创建了表的当前分区方案。

    例如，考虑在 `test` 数据库中使用以下语句创建的分区表：

    ```sql
    CREATE TABLE tp (
        c1 INT,
        c2 INT,
        c3 VARCHAR(25)
    )
    PARTITION BY HASH(c1 + c2)
    PARTITIONS 4;
    ```

    `PARTITIONS` 表中来自此表分区的 `PARTITIONS` 表行中的 `PARTITION_EXPRESSION` 列显示 `c1 + c2`，如下所示：

    ```sql
    mysql> SELECT DISTINCT PARTITION_EXPRESSION
           FROM INFORMATION_SCHEMA.PARTITIONS
           WHERE TABLE_NAME='tp' AND TABLE_SCHEMA='test';
    +----------------------+
    | PARTITION_EXPRESSION |
    +----------------------+
    | c1 + c2              |
    +----------------------+
    ```

    对于未明确分区的表，无论存储引擎如何，此列始终为 `NULL`。

+   `SUBPARTITION_EXPRESSION`

    这与定义表的分区表达式的子分区表达式的工作方式相同，就像`PARTITION_EXPRESSION`为定义表的分区而使用的分区表达式一样。

    如果表没有子分区，这一列为`NULL`。

+   `PARTITION_DESCRIPTION`

    此列用于`RANGE`和`LIST`分区。对于`RANGE`分区，它包含在分区的`VALUES LESS THAN`子句中设置的值，可以是整数或`MAXVALUE`。对于`LIST`分区，此列包含在分区的`VALUES IN`子句中定义的值，这是一个逗号分隔的整数值列表。

    对于`PARTITION_METHOD`不是`RANGE`或`LIST`的分区，此列始终为`NULL`。

+   `TABLE_ROWS`

    分区中的表行数。

    对于分区的`InnoDB`表，`TABLE_ROWS`列中给出的行数仅是 SQL 优化中使用的估计值，可能并不总是准确。

    对于`NDB`表，您还可以使用**ndb_desc**实用程序获取此信息。

+   `AVG_ROW_LENGTH`

    存储在此分区或子分区中的行的平均长度，以字节为单位。这与`DATA_LENGTH`除以`TABLE_ROWS`相同。

    对于`NDB`表，您还可以使用**ndb_desc**实用程序获取此信息。

+   `DATA_LENGTH`

    存储在此分区或子分区中的所有行的总长度，以字节为单位；即存储在分区或子分区中的字节总数。

    对于`NDB`表，您还可以使用**ndb_desc**实用程序获取此信息。

+   `MAX_DATA_LENGTH`

    可以存储在此分区或子分区中的最大字节数。

    对于`NDB`表，您还可以使用**ndb_desc**实用程序获取此信息。

+   `INDEX_LENGTH`

    此分区或子分区的索引文件长度，以字节为单位。

    对于`NDB`表的分区，无论表是否使用隐式或显式分区，`INDEX_LENGTH`列的值始终为 0。但是，您可以使用**ndb_desc**实用程序获取等效信息。

+   `DATA_FREE`

    分配给分区或子分区但未使用的字节数。

    对于`NDB`表，您还可以使用**ndb_desc**实用程序获取此信息。

+   `CREATE_TIME`

    分区或子分区创建的时间。

+   `UPDATE_TIME`

    分区或子分区上次修改的时间。

+   `CHECK_TIME`

    此分区或子分区所属表上次检查的时间。

    对于分区的`InnoDB`表，该值始终为`NULL`。

+   `CHECKSUM`

    校验和值（如果有）；否则为`NULL`。

+   `PARTITION_COMMENT`

    如果分区有评论，则评论的文本。如果没有，则该值为空。

    分区评论的最大长度定义为 1024 个字符，`PARTITION_COMMENT`列的显示宽度也是 1024 个字符，以匹配此限制。

+   `NODEGROUP`

    分区所属的节点组。对于 NDB 集群表，始终为`default`。对于使用除`NDB`之外的存储引擎的分区表，该值也为`default`。否则，此列为空。

+   `TABLESPACE_NAME`

    分区所属表空间的名称。该值始终为`DEFAULT`，除非表使用`NDB`存储引擎（请参阅本节末尾的*注释*）。

#### 注释

+   `PARTITIONS`是一个非标准的`INFORMATION_SCHEMA`表。

+   使用除`NDB`之外的任何存储引擎并且未分区的表在`PARTITIONS`表中有一行。但是，`PARTITION_NAME`、`SUBPARTITION_NAME`、`PARTITION_ORDINAL_POSITION`、`SUBPARTITION_ORDINAL_POSITION`、`PARTITION_METHOD`、`SUBPARTITION_METHOD`、`PARTITION_EXPRESSION`、`SUBPARTITION_EXPRESSION`和`PARTITION_DESCRIPTION`列的值均为`NULL`。此外，在这种情况下，`PARTITION_COMMENT`列为空。

+   在`NDB`集群中，未明确分区的`NDB`表在`PARTITIONS`表中为每个数据节点有一行。对于每一行：

    +   `SUBPARTITION_NAME`、`SUBPARTITION_ORDINAL_POSITION`、`SUBPARTITION_METHOD`、`PARTITION_EXPRESSION`、`SUBPARTITION_EXPRESSION`、`CREATE_TIME`、`UPDATE_TIME`、`CHECK_TIME`、`CHECKSUM`和`TABLESPACE_NAME`列均为`NULL`。

    +   `PARTITION_METHOD`始终为`AUTO`。

    +   `NODEGROUP`列为`default`。

    +   `PARTITION_COMMENT`列为空。
