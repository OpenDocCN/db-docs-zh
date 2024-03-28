> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-limitations-syntax.html)

#### 25.2.7.1 NDB Cluster 中与 SQL 语法不符

与某些 MySQL 功能相关的某些 SQL 语句在与`NDB`表一起使用时会产生错误，如下列表所述：

+   **临时表。** 不支持临时表。尝试创建使用`NDB`存储引擎的临时表或更改现有临时表以使用`NDB`都会失败，并显示错误消息表存储引擎'ndbcluster'不支持创建选项'TEMPORARY'。

+   **NDB 表中的索引和键。** NDB Cluster 表上的键和索引受以下限制：

    +   **列宽。** 尝试在宽度大于 3072 字节的`NDB`表列上创建索引会成功，但实际上只有前 3072 字节用于索引。在这种情况下，会发出警告 Specified key was too long; max key length is 3072 bytes，并且`SHOW CREATE TABLE`语句显示索引的长度为 3072。

    +   **TEXT 和 BLOB 列。** 你不能在使用`NDB`表列的`TEXT`或`BLOB`数据类型上创建索引。

    +   **FULLTEXT 索引。** `NDB`存储引擎不支持`FULLTEXT`索引，这仅适用于`MyISAM`和`InnoDB`表。

        但是，你可以在`NDB`表的`VARCHAR`列上创建索引。

    +   **使用 HASH 键和 NULL。** 在唯一键和主键中使用可空列意味着使用这些列的查询将被处理为全表扫描。要解决此问题，请使列`NOT NULL`，或重新创建索引而不使用`USING HASH`选项。

    +   **前缀。** 没有前缀索引；只能对整个列进行索引。（`NDB`列索引的大小始终与列的宽度相同，以字节为单位，最多为 3072 字节，如本节前面描述的那样。另请参阅 Section 25.2.7.6, “Unsupported or Missing Features in NDB Cluster”，获取更多信息。）

    +   **BIT 列。** `BIT`列不能作为主键、唯一键或索引，也不能成为复合主键、唯一键或索引的一部分。

    +   **AUTO_INCREMENT 列。** 与其他 MySQL 存储引擎一样，`NDB`存储引擎每个表最多只能处理一个`AUTO_INCREMENT`列，并且这个列必须被索引。然而，在没有显式主键的 NDB 表中，一个`AUTO_INCREMENT`列会被自动定义并用作“隐藏”主键。因此，你不能创建一个具有`AUTO_INCREMENT`列但没有显式主键的`NDB`表。

        下面的`CREATE TABLE`语句不起作用，如下所示：

        ```sql
        # No index on AUTO_INCREMENT column; table has no primary key
        # Raises ER_WRONG_AUTO_KEY
        mysql> CREATE TABLE n (
         ->     a INT,
         ->     b INT AUTO_INCREMENT
         ->     )
         -> ENGINE=NDB;
        ERROR 1075 (42000): Incorrect table definition; there can be only one auto
        column and it must be defined as a key 

        # Index on AUTO_INCREMENT column; table has no primary key
        # Raises NDB error 4335
        mysql> CREATE TABLE n (
         ->     a INT,
         ->     b INT AUTO_INCREMENT,
         ->     KEY k (b)
         ->     )
         -> ENGINE=NDB;
        ERROR 1296 (HY000): Got error 4335 'Only one autoincrement column allowed per
        table. Having a table without primary key uses an autoincr' from NDBCLUSTER
        ```

        下面的语句创建了一个具有主键、一个`AUTO_INCREMENT`列以及对该列的索引的表，并成功：

        ```sql
        # Index on AUTO_INCREMENT column; table has a primary key
        mysql> CREATE TABLE n (
         ->     a INT PRIMARY KEY,
         ->     b INT AUTO_INCREMENT,
         ->     KEY k (b)
         ->     )
         -> ENGINE=NDB;
        Query OK, 0 rows affected (0.38 sec)
        ```

+   **外键的限制。** NDB 8.0 中对外键约束的支持与`InnoDB`提供的相似，但受以下限制：

    +   每个作为外键引用的列都需要一个显式的唯一键，如果它不是表的主键。

    +   当引用是指向父表的主键时，不支持`ON UPDATE CASCADE`。

        这是因为对主键的更新被实现为删除旧行（包含旧主键的行）以及插入新行（带有新主键）。这对于`NDB`内核来说是不可见的，它将这两行视为相同，因此无法知道应该级联执行此更新。

    +   当子表包含一个或多个`TEXT`或`BLOB`类型的列时，也不支持`ON DELETE CASCADE`。（Bug #89511, Bug #27484882）

    +   不支持`SET DEFAULT`。（`InnoDB`也不支持。）

    +   `NO ACTION`关键字被接受但被视为`RESTRICT`。`NO ACTION`是标准 SQL 关键字，在 MySQL 8.0 中是默认的。（与`InnoDB`相同。）

    +   在早期版本的 NDB Cluster 中，当创建一个具有外键引用另一张表中索引的表时，有时似乎可以创建外键，即使索引中列的顺序不匹配，这是因为并不总是返回适当的错误。对于这个问题的部分修复改进了内部使用的错误以在大多数情况下工作；然而，在父索引是唯一索引的情况下，仍然可能发生这种情况。（Bug #18094360）

    有关更多信息，请参见第 15.1.20.5 节，“外键约束”和第 1.6.3.2 节，“外键约束”。

+   **NDB 集群和几何数据类型。** 几何数据类型（`WKT`和`WKB`）支持`NDB`表。但是，不支持空间索引。

+   **字符集和二进制日志文件。** 目前，`ndb_apply_status`和`ndb_binlog_index`表使用`latin1`（ASCII）字符集创建。由于二进制日志的名称记录在此表中，因此使用非拉丁字符命名的二进制日志文件在这些表中无法正确引用。这是一个已知问题，我们正在努力解决。（Bug #50226）

    要解决此问题，请在命名二进制日志文件或设置`--basedir`、`--log-bin`或`--log-bin-index`选项时仅使用 Latin-1 字符。

+   **使用用户定义的分区创建 NDB 表。** NDB 集群中对用户定义的分区的支持仅限于[`LINEAR`] `KEY`分区。在`CREATE TABLE`语句中使用`ENGINE=NDB`或`ENGINE=NDBCLUSTER`与任何其他分区类型会导致错误。

    可以覆盖此限制，但不支持在生产环境中使用。有关详细信息，请参见用户定义的分区和 NDB 存储引擎（NDB 集群）")。

    **默认分区方案。** 所有 NDB 集群表默认通过使用表的主键作为分区键进行`KEY`分区。如果未为表明确设置主键，则由`NDB`存储引擎自动创建的“隐藏”主键将被使用。有关这些和相关问题的更多讨论，请参见第 26.2.5 节，“KEY 分区”。

    `CREATE TABLE`和`ALTER TABLE`语句如果导致用户分区的`NDBCLUSTER`表不满足以下两个要求中的任何一个或两个都不允许，并将失败并显示错误：

    1.  表必须有显式的主键。

    1.  表中分区表达式中列出的所有列必须是主键的一部分。

    **例外。** 如果使用空列列表（即使用`PARTITION BY [LINEAR] KEY()`）创建用户分区的`NDBCLUSTER`表，则不需要显式主键。

    **NDBCLUSTER 表的最大分区数。** 当使用用户定义的分区时，`NDBCLUSTER`表可以定义的最大分区数为每个节点组 8 个。（有关 NDB Cluster 节点组的更多信息，请参见第 25.2.2 节，“NDB Cluster 节点、节点组、片段副本和分区”）。

    **不支持 DROP PARTITION。** 无法使用`ALTER TABLE ... DROP PARTITION`从`NDB`表中删除分区。对于 NDB 表，支持其他分区扩展——`ADD PARTITION`、`REORGANIZE PARTITION`和`COALESCE PARTITION`，但使用复制，因此不是优化的。请参见第 26.3.1 节，“RANGE 和 LIST 分区的管理”和第 15.1.9 节，“ALTER TABLE 语句”。

    **分区选择。** 不支持为`NDB`表选择分区。有关更多信息，请参见第 26.5 节，“分区选择”。

+   **JSON 数据类型。** MySQL 中的`JSON`数据类型在 NDB 8.0 中的**mysqld**中支持`NDB`表。

    一个`NDB`表最多可以有 3 个`JSON`列。

    NDB API 没有专门用于处理`JSON`数据的功能，它将其简单地视为`BLOB`数据。处理数据为`JSON`必须由应用程序执行。
