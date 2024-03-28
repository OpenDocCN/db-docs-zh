# 17.12.1 在线 DDL 操作

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-operations.html)

本节中提供了 DDL 操作的在线支持详细信息、语法示例和用法说明。

+   索引操作

+   主键操作

+   列操作

+   生成列操作

+   外键操作

+   表操作

+   表空间操作

+   分区操作

#### 索引操作

以下表格概述了索引操作的在线 DDL 支持。星号表示额外信息、异常或依赖关系。详情请参阅语法和用法说明。

**表 17.16 索引操作的在线 DDL 支持**

| 操作 | 立即 | 就地 | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 创建或添加二级索引 | 否 | 是 | 否 | 是 | 否 |
| 删除索引 | 否 | 是 | 否 | 是 | 是 |
| 重命名索引 | 否 | 是 | 否 | 是 | 是 |
| 添加`FULLTEXT`索引 | 否 | 是* | 否* | 否 | 否 |
| 添加`SPATIAL`索引 | 否 | 是 | 否 | 否 | 否 |
| 更改索引类型 | 是 | 是 | 否 | 是 | 是 |

##### 语法和用法说明

+   创建或添加二级索引

    ```sql
    CREATE INDEX *name* ON *table* (*col_list*);
    ```

    ```sql
    ALTER TABLE *tbl_name* ADD INDEX *name* (*col_list*);
    ```

    在创建索引时，表仍可用于读写操作。`CREATE INDEX`语句仅在访问表的所有事务完成后才完成，以便索引的初始状态反映表的最新内容。

    添加二级索引的在线 DDL 支持意味着您通常可以通过在加载数据后创建不带二级索引的表，然后在数据加载后添加二级索引，从而加快创建和加载表及相关索引的整个过程。

    新创建的二级索引仅包含在`CREATE INDEX`或`ALTER TABLE`语句执行完成时表中的已提交数据。它不包含任何未提交的值、旧版本的值或标记为删除但尚未从旧索引中删除的值。

    一些因素会影响此操作的性能、空间使用和语义。详情请参阅 Section 17.12.8, “在线 DDL 限制”。

+   删除索引

    ```sql
    DROP INDEX *name* ON *table*;
    ```

    ```sql
    ALTER TABLE *tbl_name* DROP INDEX *name*;
    ```

    当索引被删除时，表仍然可用于读写操作。`DROP INDEX`语句只有在访问表的所有事务完成后才会完成，以便索引的初始状态反映表的最新内容。

+   重命名索引

    ```sql
    ALTER TABLE *tbl_name* RENAME INDEX *old_index_name* TO *new_index_name*, ALGORITHM=INPLACE, LOCK=NONE;
    ```

+   添加`FULLTEXT`索引

    ```sql
    CREATE FULLTEXT INDEX *name* ON table(*column*);
    ```

    添加第一个`FULLTEXT`索引会在没有用户定义的`FTS_DOC_ID`列的情况下重建表。可以在不重建表的情况下添加额外的`FULLTEXT`索引。

+   添加`SPATIAL`索引

    ```sql
    CREATE TABLE geom (g GEOMETRY NOT NULL);
    ALTER TABLE geom ADD SPATIAL INDEX(g), ALGORITHM=INPLACE, LOCK=SHARED;
    ```

+   更改索引类型（`USING {BTREE | HASH}`）

    ```sql
    ALTER TABLE *tbl_name* DROP INDEX i1, ADD INDEX i1(*key_part,...*) USING BTREE, ALGORITHM=INSTANT;
    ```

#### 主键操作

下表概述了主键操作的在线 DDL 支持。星号表示额外信息、异常或依赖项。请参阅语法和用法说明。

**表 17.17 主键操作的在线 DDL 支持**

| 操作 | 立即 | 原地 | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 添加主键 | 否 | 是* | 是* | 是 | 否 |
| 删除主键 | 否 | 否 | 是 | 否 | 否 |
| 删除主键并添加另一个 | 否 | 是 | 是 | 是 | 否 |

##### 语法和用法说明

+   添加主键

    ```sql
    ALTER TABLE *tbl_name* ADD PRIMARY KEY (*column*), ALGORITHM=INPLACE, LOCK=NONE;
    ```

    在原地重建表。数据被大幅重新组织，使其成为一项昂贵的操作。如果必须将列转换为`NOT NULL`，则在某些条件下不允许使用`ALGORITHM=INPLACE`。

    重构聚簇索引总是需要复制表数据。因此，最好在创建表时定义主键，而不是稍后发出`ALTER TABLE ... ADD PRIMARY KEY`。

    当创建`UNIQUE`或`PRIMARY KEY`索引时，MySQL 必须做一些额外的工作。对于`UNIQUE`索引，MySQL 检查表中是否没有重复键值。对于`PRIMARY KEY`索引，MySQL 还检查是否没有`PRIMARY KEY`列包含`NULL`。

    当使用`ALGORITHM=COPY`子句添加主键时，MySQL 会将相关列中的`NULL`值转换为默认值：数字为 0，基于字符的列和 BLOB 为空字符串，`DATETIME`为 0000-00-00 00:00:00。这是一种非标准行为，Oracle 建议您不要依赖于此。只有在`SQL_MODE`设置包括`strict_trans_tables`或`strict_all_tables`标志时，才允许使用`ALGORITHM=INPLACE`添加主键；当`SQL_MODE`设置为 strict 时，允许使用`ALGORITHM=INPLACE`，但如果请求的主键列包含`NULL`值，则语句仍可能失败。`ALGORITHM=INPLACE`行为更符合标准。

    如果创建一个没有主键的表，`InnoDB`会为您选择一个主键，可以是第一个在`NOT NULL`列上定义的`UNIQUE`键，或者是系统生成的键。为了避免不确定性和额外隐藏列的潜在空间需求，请在`CREATE TABLE`语句中指定`PRIMARY KEY`子句。

    MySQL 通过将现有数据从原始表复制到具有所需索引结构的临时表来创建新的聚集索引。一旦数据完全复制到临时表，原始表将以不同的临时表名称重命名。包含新聚集索引的临时表将以原始表的名称重命名，并且原始表将从数据库中删除。

    适用于次要索引操作的在线性能增强不适用于主键索引。InnoDB 表的行存储在基于主键组织的聚集索引中，形成一些数据库系统称为“索引组织表”的结构。由于表结构与主键紧密相关，重新定义主键仍然需要复制数据。

    当对主键使用`ALGORITHM=INPLACE`时，即使数据仍在复制，也比使用`ALGORITHM=COPY`更有效，因为：

    +   对于`ALGORITHM=INPLACE`不需要撤消日志或相关的重做日志。这些操作会给使用`ALGORITHM=COPY`的 DDL 语句增加开销。

    +   次要索引条目已经预先排序，因此可以按顺序加载。

    +   由于没有随机访问插入到次要索引中，因此不使用更改缓冲区。

+   删除主键

    ```sql
    ALTER TABLE *tbl_name* DROP PRIMARY KEY, ALGORITHM=COPY;
    ```

    只有`ALGORITHM=COPY`支持在同一`ALTER TABLE`语句中删除主键而不添加新主键。

+   删除一个主键并添加另一个

    ```sql
    ALTER TABLE *tbl_name* DROP PRIMARY KEY, ADD PRIMARY KEY (*column*), ALGORITHM=INPLACE, LOCK=NONE;
    ```

    数据进行了大幅重组，这是一项昂贵的操作。

#### 列操作

下表提供了关于列操作的在线 DDL 支持的概述。星号表示额外信息、异常或依赖关系。详情请参见语法和用法说明。

**表 17.18 列操作的在线 DDL 支持**

| 操作 | Instant | In Place | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 添加列 | 是* | 是 | 否* | 是* | 是 |
| 删除列 | 是* | 是 | 是 | 是 | 是 |
| 重命名列 | 是* | 是 | 否 | 是* | 是 |
| 重新排序列 | 否 | 是 | 是 | 是 | 否 |
| 设置列默认值 | 是 | 是 | 否 | 是 | 是 |
| 更改列数据类型 | 否 | 否 | 是 | 否 | 否 |
| 扩展`VARCHAR`列大小 | 否 | 是 | 否 | 是 | 是 |
| 删除列默认值 | 是 | 是 | 否 | 是 | 是 |
| 更改自增值 | 否 | 是 | 否 | 是 | 否* |
| 使列为`NULL` | 否 | 是 | 是* | 是 | 否 |
| 使列为`NOT NULL` | 否 | 是* | 是* | 是 | 否 |
| 修改`ENUM`或`SET`列的定义 | 是 | 是 | 否 | 是 | 是 |
| 操作 | Instant | In Place | 重建表 | 允许并发 DML | 仅修改元数据 |

##### 语法和用法说明

+   添加列

    ```sql
    ALTER TABLE *tbl_name* ADD COLUMN *column_name* *column_definition*, ALGORITHM=INSTANT;
    ```

    `INSTANT`是 MySQL 8.0.12 之后的默认算法，之前是`INPLACE`。

    当`INSTANT`算法添加列时，以下限制适用：

    +   一条语句不能将添加列与不支持`INSTANT`算法的其他`ALTER TABLE`操作结合在一起。

    +   `INSTANT`算法可以在表中的任何位置添加列。在 MySQL 8.0.29 之前，`INSTANT`算法只能将列添加为表的最后一列。

    +   不能向使用`ROW_FORMAT=COMPRESSED`、具有`FULLTEXT`索引、位于数据字典表空间中的表或临时表添加列。临时表仅支持`ALGORITHM=COPY`。

    +   当`INSTANT`算法添加列时，MySQL 会检查行大小，如果添加超过限制，则会抛出以下错误。

        错误 4092（HY000）：无法使用 ALGORITHM=INSTANT 添加列，因为此后最大可能行大小超过了最大允许行大小。请尝试 ALGORITHM=INPLACE/COPY。

        在 MySQL 8.0.29 之前，MySQL 在`INSTANT`算法添加列时不会检查行大小。但是，在插入和更新表中的行的 DML 操作期间，MySQL 会检查行大小。

    +   在使用`INSTANT`算法添加列后，表的内部表示中列的最大数量不能超过 1022。错误消息为：

        错误 4158（HY000）：无法使用 ALGORITHM=INSTANT 将列添加到*`tbl_name`*。请尝试 ALGORITHM=INPLACE/COPY

    +   `INSTANT`算法无法向系统模式表（如内部`mysql`表）添加或删除列。此限制是在 MySQL 8.0.29 中添加的。

    可以在同一 `ALTER TABLE` 语句中添加多个列。例如：

    ```sql
    ALTER TABLE t1 ADD COLUMN c2 INT, ADD COLUMN c3 INT, ALGORITHM=INSTANT;
    ```

    每次执行`ALTER TABLE ... ALGORITHM=INSTANT` 操作添加一个或多个列、删除一个或多个列，或者在同一操作中添加和删除一个或多个列后，都会创建一个新的行版本。`INFORMATION_SCHEMA.INNODB_TABLES.TOTAL_ROW_VERSIONS` 列跟踪表的行版本数量。每次立即添加或删除列时，该值都会递增。初始值为 0。

    ```sql
    mysql>  SELECT NAME, TOTAL_ROW_VERSIONS FROM INFORMATION_SCHEMA.INNODB_TABLES 
            WHERE NAME LIKE 'test/t1';
    +---------+--------------------+
    | NAME    | TOTAL_ROW_VERSIONS |
    +---------+--------------------+
    | test/t1 |                  0 |
    +---------+--------------------+
    ```

    当具有立即添加或删除列的表通过表重建的 `ALTER TABLE` 或 `OPTIMIZE TABLE` 操作重建时，`TOTAL_ROW_VERSIONS` 值将重置为 0。允许的最大行版本数为 64，因为每个行版本都需要额外的表元数据空间。当达到行版本限制时，使用`ALGORITHM=INSTANT`的`ADD COLUMN`和`DROP COLUMN`操作将被拒绝，并显示错误消息建议使用`COPY`或`INPLACE`算法重建表。

    错误 4080 (HY000): 表 test/t1 的最大行版本已达到。无法立即添加或删除更多列。请使用 COPY/INPLACE。

    以下 `INFORMATION_SCHEMA` 列提供了立即添加列的附加元数据。有关这些列的描述，请参考这些列的描述以获取更多信息。参见 Section 28.4.9, “The INFORMATION_SCHEMA INNODB_COLUMNS Table” 和 Section 28.4.23, “The INFORMATION_SCHEMA INNODB_TABLES Table”。

    +   `INNODB_COLUMNS.DEFAULT_VALUE`

    +   `INNODB_COLUMNS.HAS_DEFAULT`

    +   `INNODB_TABLES.INSTANT_COLS`

    在添加自增列时不允许并发 DML。数据会被大幅重组，使其成为一项昂贵的操作。至少需要`ALGORITHM=INPLACE, LOCK=SHARED`。

    如果使用`ALGORITHM=INPLACE`添加列，则表将被重建。

+   删除列

    ```sql
    ALTER TABLE *tbl_name* DROP COLUMN *column_name*, ALGORITHM=INSTANT;
    ```

    截至 MySQL 8.0.29，`INSTANT`是默认算法，之前是`INPLACE`。

    使用`INSTANT`算法删除列时会出现以下限制：

    +   不能将删除列与不支持`ALGORITHM=INSTANT`的其他 `ALTER TABLE` 操作结合在同一语句中。

    +   不能从使用`ROW_FORMAT=COMPRESSED`、具有`FULLTEXT`索引、位于数据字典表空间中的表或临时表中删除列。临时表仅支持`ALGORITHM=COPY`。

    可以在同一`ALTER TABLE`语句中删除多个列；例如：

    ```sql
    ALTER TABLE t1 DROP COLUMN c4, DROP COLUMN c5, ALGORITHM=INSTANT;
    ```

    每次使用`ALGORITHM=INSTANT`添加或删除列时，都会创建一个新的行版本。`INFORMATION_SCHEMA.INNODB_TABLES.TOTAL_ROW_VERSIONS`列跟踪表的行版本数。每次立即添加或删除列时，该值都会递增。初始值为 0。

    ```sql
    mysql>  SELECT NAME, TOTAL_ROW_VERSIONS FROM INFORMATION_SCHEMA.INNODB_TABLES 
            WHERE NAME LIKE 'test/t1';
    +---------+--------------------+
    | NAME    | TOTAL_ROW_VERSIONS |
    +---------+--------------------+
    | test/t1 |                  0 |
    +---------+--------------------+
    ```

    当具有立即添加或删除列的表通过表重建`ALTER TABLE`或`OPTIMIZE TABLE`操作重建时，`TOTAL_ROW_VERSIONS`值将重置为 0。允许的最大行版本数为 64，因为每个行版本都需要额外的空间用于表元数据。当达到行版本限制时，使用`ALGORITHM=INSTANT`的`ADD COLUMN`和`DROP COLUMN`操作将被拒绝，并显示错误消息建议使用`COPY`或`INPLACE`算法重建表。

    错误 4080 (HY000)：表 test/t1 的最大行版本已达到。无法立即添加或删除更多列。请使用 COPY/INPLACE。

    如果使用`ALGORITHM=INSTANT`之外的算法，数据会被大幅重组，使其成为一项昂贵的操作。

+   重命名列

    ```sql
    ALTER TABLE *tbl* CHANGE *old_col_name* *new_col_name* *data_type*, ALGORITHM=INSTANT;
    ```

    MySQL 8.0.28 中添加了对重命名列的`ALGORITHM=INSTANT`支持。在较早的 MySQL Server 版本中，重命名列仅支持`ALGORITHM=INPLACE`和`ALGORITHM=COPY`。

    为了允许并发 DML，请保持相同的数据类型，只更改列名。

    当保持相同的数据类型和`[NOT] NULL`属性，仅更改列名时，该操作始终可以在线执行。

    仅允许使用`ALGORITHM=INPLACE`重命名另一个表引用的列。如果使用`ALGORITHM=INSTANT`、`ALGORITHM=COPY`或导致操作使用这些算法的其他条件，则`ALTER TABLE`语句将失败。

    `ALGORITHM=INSTANT`支持重命名虚拟列；`ALGORITHM=INPLACE`不支持。

    在相同语句中添加或删除虚拟列时，`ALGORITHM=INSTANT`和`ALGORITHM=INPLACE`不支持重命名列。在这种情况下，只支持`ALGORITHM=COPY`。

+   重新排序列

    要重新排序列，请在`CHANGE`或`MODIFY`操作中使用`FIRST`或`AFTER`。

    ```sql
    ALTER TABLE *tbl_name* MODIFY COLUMN *col_name* *column_definition* FIRST, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    数据会被大幅重组，使其成为一项昂贵的操作。

+   更改列数据类型

    ```sql
    ALTER TABLE *tbl_name* CHANGE c1 c1 BIGINT, ALGORITHM=COPY;
    ```

    只有使用`ALGORITHM=COPY`才支持更改列数据类型。

+   扩展`VARCHAR`列大小

    ```sql
    ALTER TABLE *tbl_name* CHANGE COLUMN c1 c1 VARCHAR(255), ALGORITHM=INPLACE, LOCK=NONE;
    ```

    `VARCHAR`列所需的长度字节数必须保持不变。对于大小为 0 到 255 字节的`VARCHAR`列，需要一个长度字节来编码值。对于大小为 256 字节或更大的`VARCHAR`列，需要两个长度字节。因此，原地`ALTER TABLE`仅支持将`VARCHAR`列大小从 0 到 255 字节增加，或从 256 字节增加到更大的大小。原地`ALTER TABLE`不支持将`VARCHAR`列大小从小于 256 字节增加到等于或大于 256 字节。在这种情况下，所需的长度字节数从 1 变为 2，只能通过表复制（`ALGORITHM=COPY`）支持。例如，尝试使用原地`ALTER TABLE`将单字节字符集的`VARCHAR`列大小从 VARCHAR(255)更改为 VARCHAR(256)会返回此错误：

    ```sql
    ALTER TABLE *tbl_name* ALGORITHM=INPLACE, CHANGE COLUMN c1 c1 VARCHAR(256);
    ERROR 0A000: ALGORITHM=INPLACE is not supported. Reason: Cannot change
    column type INPLACE. Try ALGORITHM=COPY.
    ```

    注意

    `VARCHAR`列的字节长度取决于字符集的字节长度。

    使用原地`ALTER TABLE`不支持减小`VARCHAR`大小。减小`VARCHAR`大小需要表复制（`ALGORITHM=COPY`）。

+   设置列默认值

    ```sql
    ALTER TABLE *tbl_name* ALTER COLUMN *col* SET DEFAULT *literal*, ALGORITHM=INSTANT;
    ```

    仅修改表元数据。默认列值存储在数据字典中。

+   删除列默认值

    ```sql
    ALTER TABLE *tbl* ALTER COLUMN *col* DROP DEFAULT, ALGORITHM=INSTANT;
    ```

+   更改自增值

    ```sql
    ALTER TABLE *table* AUTO_INCREMENT=*next_value*, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    修改存储在内存中的值，而不是数据文件中的值。

    在使用复制或分片的分布式系统中，有时会将表的自增计数器重置为特定值。表中插入的下一行将使用其自增列的指定值。您也可以在数据仓库环境中使用此技术，定期清空所有表并重新加载它们，并从 1 重新启动自增序列。

+   使列`NULL`

    ```sql
    ALTER TABLE tbl_name MODIFY COLUMN *column_name* *data_type* NULL, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    在原地重建表。数据被大幅重新组织，使其成为昂贵的操作。

+   使列`NOT NULL`

    ```sql
    ALTER TABLE *tbl_name* MODIFY COLUMN *column_name* *data_type* NOT NULL, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    在原地重建表格。操作需要`STRICT_ALL_TABLES`或`STRICT_TRANS_TABLES` `SQL_MODE` 的支持才能成功。如果列包含 NULL 值，则操作将失败。服务器禁止对可能导致引用完整性丢失的外键列进行更改。请参阅第 15.1.9 节，“ALTER TABLE Statement”。数据将被大幅重新组织，这是一个昂贵的操作。

+   修改`ENUM`或`SET`列的定义

    ```sql
    CREATE TABLE t1 (c1 ENUM('a', 'b', 'c'));
    ALTER TABLE t1 MODIFY COLUMN c1 ENUM('a', 'b', 'c', 'd'), ALGORITHM=INSTANT;
    ```

    通过在有效成员值列表的*末尾*添加新的枚举或集合成员来修改`ENUM`或`SET`列的定义可以立即执行或原地执行，只要数据类型的存储大小不变。例如，向具有 8 个成员的`SET`列添加一个成员会将每个值所需的存储从 1 字节更改为 2 字节；这需要复制表格。在列表中间添加成员会导致现有成员的重新编号，这需要复制表格。

#### 生成列操作

下表提供了生成列操作的在线 DDL 支持概述。详情请参阅语法和用法注意事项。

**表 17.19 生成列操作的在线 DDL 支持**

| 操作 | 立即执行 | 原地执行 | 重建表格 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 添加`STORED`列 | 否 | 否 | 是 | 否 | 否 |
| 修改`STORED`列顺序 | 否 | 否 | 是 | 否 | 否 |
| 删除`STORED`列 | 否 | 是 | 是 | 是 | 否 |
| 添加`VIRTUAL`列 | 是 | 是 | 否 | 是 | 是 |
| 修改`VIRTUAL`列顺序 | 否 | 否 | 是 | 否 | 否 |
| 删除`VIRTUAL`列 | 是 | 是 | 否 | 是 | 是 |

##### 语法和用法注意事项

+   添加`STORED`列

    ```sql
    ALTER TABLE t1 ADD COLUMN (c2 INT GENERATED ALWAYS AS (c1 + 1) STORED), ALGORITHM=COPY;
    ```

    `ADD COLUMN`对于存储列（不使用临时表）不是一个原地操作，因为表达式必须由服务器评估。

+   修改`STORED`列顺序

    ```sql
    ALTER TABLE t1 MODIFY COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) STORED FIRST, ALGORITHM=COPY;
    ```

    在原地重建表格。

+   删除`STORED`列

    ```sql
    ALTER TABLE t1 DROP COLUMN c2, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    在原地重建表格。

+   添加`VIRTUAL`列

    ```sql
    ALTER TABLE t1 ADD COLUMN (c2 INT GENERATED ALWAYS AS (c1 + 1) VIRTUAL), ALGORITHM=INSTANT;
    ```

    对于非分区表，添加虚拟列可以立即执行或原地执行。

    对分区表来说，添加`VIRTUAL`列不是一个原地操作。

+   修改`VIRTUAL`列顺序

    ```sql
    ALTER TABLE t1 MODIFY COLUMN c2 INT GENERATED ALWAYS AS (c1 + 1) VIRTUAL FIRST, ALGORITHM=COPY;
    ```

+   删除`VIRTUAL`列

    ```sql
    ALTER TABLE t1 DROP COLUMN c2, ALGORITHM=INSTANT;
    ```

    对于非分区表，删除`VIRTUAL`列可以立即执行或原地执行。

#### 外键操作

下表提供了外键操作的在线 DDL 支持概述。星号表示额外信息、异常或依赖关系。详情请参阅语法和用法注意事项。

**表 17.20 外键操作的在线 DDL 支持**

| 操作 | 立即 | 就地 | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 添加外键约束 | 否 | 是* | 否 | 是 | 是 |
| 删除外键约束 | 否 | 是 | 否 | 是 | 是 |

##### 语法和用法说明

+   添加外键约束

    当`foreign_key_checks`被禁用时，支持`INPLACE`算法。否则，只支持`COPY`算法。

    ```sql
    ALTER TABLE *tbl1* ADD CONSTRAINT *fk_name* FOREIGN KEY *index* (*col1*)
      REFERENCES *tbl2*(*col2*) *referential_actions*;
    ```

+   删除外键约束

    ```sql
    ALTER TABLE *tbl* DROP FOREIGN KEY *fk_name*;
    ```

    可以在启用或禁用`foreign_key_checks`选项的情况下在线执行删除外键操作。

    如果不知道特定表上外键约束的名称，请发出以下语句，并在每个外键的`CONSTRAINT`子句中找到约束名称：

    ```sql
    SHOW CREATE TABLE *table*\G
    ```

    或者，查询信息模式`TABLE_CONSTRAINTS`表，并使用`CONSTRAINT_NAME`和`CONSTRAINT_TYPE`列来识别外键名称。

    您还可以在单个语句中删除外键及其关联的索引：

    ```sql
    ALTER TABLE *table* DROP FOREIGN KEY *constraint*, DROP INDEX *index*;
    ```

注意

如果表中已经存在外键（即，它是包含`FOREIGN KEY ... REFERENCE`子句的子表），则对在线 DDL 操作会有额外限制，即使这些操作并不直接涉及外键列：

+   如果对父表的更改通过`ON UPDATE`或`ON DELETE`子句使用`CASCADE`或`SET NULL`参数导致子表中的关联更改，那么对子表进行的`ALTER TABLE`可能需要等待另一个事务提交。

+   同样，如果一张表是外键关系中的父表，即使它不包含任何`FOREIGN KEY`子句，如果`INSERT`、`UPDATE`或`DELETE`语句导致子表中的`ON UPDATE`或`ON DELETE`操作，它也可能需要等待`ALTER TABLE`完成。

#### 表操作

以下表格提供了表操作的在线 DDL 支持概述。星号表示额外信息、异常或依赖关系。有关详细信息，请参阅语法和用法说明。

**表 17.21 表操作的在线 DDL 支持**

| 操作 | 立即 | 就地 | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 更改 `ROW_FORMAT` | 否 | 是 | 是 | 是 | 否 |
| 更改 `KEY_BLOCK_SIZE` | 否 | 是 | 是 | 是 | 否 |
| 设置持久表统计信息 | 否 | 是 | 否 | 是 | 是 |
| 指定字符集 | 否 | 是 | 是* | 是 | 否 |
| 转换字符集 | 否 | 否 | 是* | 否 | 否 |
| 优化表 | 否 | 是* | 是 | 是 | 否 |
| 使用 `FORCE` 选项重建 | 否 | 是* | 是 | 是 | 否 |
| 执行空重建 | 否 | 是* | 是 | 是 | 否 |
| 重命名表 | 是 | 是 | 否 | 是 | 是 |

##### 语法和用法说明

+   更改 `ROW_FORMAT`

    ```sql
    ALTER TABLE *tbl_name* ROW_FORMAT = *row_format*, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    数据进行了大幅重组，这是一个昂贵的操作。

    有关 `ROW_FORMAT` 选项的更多信息，请参见表选项。

+   更改 `KEY_BLOCK_SIZE`

    ```sql
    ALTER TABLE *tbl_name* KEY_BLOCK_SIZE = *value*, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    数据进行了大幅重组，这是一个昂贵的操作。

    有关 `KEY_BLOCK_SIZE` 选项的更多信息，请参见表选项。

+   设置持久表统计信息选项

    ```sql
    ALTER TABLE *tbl_name* STATS_PERSISTENT=0, STATS_SAMPLE_PAGES=20, STATS_AUTO_RECALC=1, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    仅修改表元数据。

    持久统计信息包括 `STATS_PERSISTENT`、`STATS_AUTO_RECALC` 和 `STATS_SAMPLE_PAGES`。有关更多信息，请参见 Section 17.8.10.1, “Configuring Persistent Optimizer Statistics Parameters”。

+   指定字符集

    ```sql
    ALTER TABLE *tbl_name* CHARACTER SET = *charset_name*, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    如果新的字符编码不同，则重新构建表。

+   转换字符集

    ```sql
    ALTER TABLE *tbl_name* CONVERT TO CHARACTER SET *charset_name*, ALGORITHM=COPY;
    ```

    如果新的字符编码不同，则重新构建表。

+   优化表

    ```sql
    OPTIMIZE TABLE *tbl_name*;
    ```

    不支持具有 `FULLTEXT` 索引的表的就地操作。该操作使用 `INPLACE` 算法，但不允��使用 `ALGORITHM` 和 `LOCK` 语法。

+   使用 `FORCE` 选项重建表

    ```sql
    ALTER TABLE *tbl_name* FORCE, ALGORITHM=INPLACE, LOCK=NONE;
    ```

    在 MySQL 5.6.17 中使用 `ALGORITHM=INPLACE`。对于具有 `FULLTEXT` 索引的表，不支持 `ALGORITHM=INPLACE`。

+   `执行“null”重建`

    ```sql
    `ALTER TABLE *tbl_name* ENGINE=InnoDB, ALGORITHM=INPLACE, LOCK=NONE;`
    ```

    在 MySQL 5.6.17 中使用 `ALGORITHM=INPLACE`。对于具有 `FULLTEXT` 索引的表，不支持 `ALGORITHM=INPLACE`。

+   `重命名表`

    ```sql
    `ALTER TABLE *old_tbl_name* RENAME TO *new_tbl_name*, ALGORITHM=INSTANT;`
    ```

    `重命名表可以立即执行或就地执行。MySQL 重命名与表 *`tbl_name`* 对应的文件，而不进行复制。（您也可以使用 `RENAME TABLE` 语句来重命名表。请参见 Section 15.1.36, “RENAME TABLE Statement”。）专门授予重命名表的权限不会迁移到新名称。必须手动更改。`

`#### 表空间操作

以下表格概述了表空间操作的在线 DDL 支持。有关详细信息，请参见语法和用法说明。

**表 17.22 表空间操作的在线 DDL 支持**

| 操作 | 立即 | 就地 | 重建表 | 允许并发 DML | 仅修改元数据 |
| --- | --- | --- | --- | --- | --- |
| 重命名通用表空间 | 否 | 是 | 否 | 是 | 是 |
| 启用或禁用通用表空间加密 | 否 | 是 | 否 | 是 | 否 |
| 启用或禁用按表加密的表空间 | 否 | 否 | 是 | 否 | 否 |

##### 语法和使用说明

+   重命名通用表空间

    ```sql
    ALTER TABLESPACE *tablespace_name* RENAME TO *new_tablespace_name*;
    ```

    `ALTER TABLESPACE ... RENAME TO`使用`INPLACE`算法，但不支持`ALGORITHM`子句。

+   启用或禁用通用表空间加密

    ```sql
    ALTER TABLESPACE *tablespace_name* ENCRYPTION='Y';
    ```

    `ALTER TABLESPACE ... ENCRYPTION`使用`INPLACE`算法，但不支持`ALGORITHM`子句。

    有关相关信息，请参阅第 17.13 节，“InnoDB 数据静态加密”。

+   启用或禁用按表加密的表空间

    ```sql
    ALTER TABLE *tbl_name* ENCRYPTION='Y', ALGORITHM=COPY;
    ```

    有关相关信息，请参阅第 17.13 节，“InnoDB 数据静态加密”。

#### 分区操作

除了一些`ALTER TABLE`分区子句外，针对分区`InnoDB`表的在线 DDL 操作遵循适用于常规`InnoDB`表的相同规则。

一些`ALTER TABLE`分区子句不会像常规非分区`InnoDB`表一样通过相同的内部在线 DDL API。因此，对于`ALTER TABLE`分区子句的在线支持有所不同。

以下表格显示了每个`ALTER TABLE`分区语句的在线状态。无论使用哪种在线 DDL API，MySQL 都会尽可能减少数据复制和锁定。

使用`ALGORITHM=COPY`的`ALTER TABLE`分区选项或仅允许“`ALGORITHM=DEFAULT, LOCK=DEFAULT`”的选项，使用`COPY`算法重新分区表。换句话说，使用新的分区方案创建了一个新的分区表。新创建的表包括`ALTER TABLE`语句应用的任何更改，并且表数据被复制到新表结构中。

**表 17.23 分区操作的在线 DDL 支持**

| 分区子句 | 立即 | 就地 | 允许 DML | 备注 |
| --- | --- | --- | --- | --- |
| `PARTITION BY` | 否 | 否 | 否 | 允许`ALGORITHM=COPY`，`LOCK={DEFAULT&#124;SHARED&#124;EXCLUSIVE}` |
| `ADD PARTITION` | 否 | 是* | 是* | 对于 `RANGE` 和 `LIST` 分区，支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;NONE&#124;SHARED&#124;EXCLUSISVE}`，对于 `HASH` 和 `KEY` 分区，支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;SHARED&#124;EXCLUSISVE}`，对于所有分区类型，支持 `ALGORITHM=COPY, LOCK={SHARED&#124;EXCLUSIVE}`。对于使用 `RANGE` 或 `LIST` 进行分区的表，不会复制现有数据。对于使用 `HASH` 或 `LIST` 进行分区的表，允许使用 `ALGORITHM=COPY` 进行并发查询，因为 MySQL 在持有共享锁的同时复制数据。 |
| `DROP PARTITION` | 否 | 是* | 是* | 支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;NONE&#124;SHARED&#124;EXCLUSIVE}`。对于使用 `RANGE` 或 `LIST` 进行分区的表，不会复制数据。使用 `ALGORITHM=INPLACE` 的 `DROP PARTITION` 删除存储在分区中的数据并删除该分区。然而，使用 `ALGORITHM=COPY` 或 `old_alter_table=ON` 的 `DROP PARTITION` 会重建分区表，并尝试将数据从已删除的分区移动到具有兼容的 `PARTITION ... VALUES` 定义的另一个分区。无法移动到另一个分区的数据将被删除。 |
| `DISCARD PARTITION` | 否 | 否 | 否 | 仅允许 `ALGORITHM=DEFAULT`, `LOCK=DEFAULT` |
| `IMPORT PARTITION` | 否 | 否 | 否 | 仅允许 `ALGORITHM=DEFAULT`, `LOCK=DEFAULT` |
| `TRUNCATE PARTITION` | 否 | 是 | 是 | 不复制现有数据。它仅删除行；不会改变表本身或任何分区的定义。 |
| `COALESCE PARTITION` | 否 | 是* | 否 | 支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;SHARED&#124;EXCLUSIVE}`。 |
| `REORGANIZE PARTITION` | 否 | 是* | 否 | 支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;SHARED&#124;EXCLUSIVE}`。 |
| `EXCHANGE PARTITION` | 否 | 是 | 是 |  |
| `ANALYZE PARTITION` | 否 | 是 | 是 |  |
| `CHECK PARTITION` | 否 | 是 | 是 |  |
| `OPTIMIZE PARTITION` | 否 | 否 | 否 | `ALGORITHM` 和 `LOCK` 子句被忽略。重建整个表。参见 Section 26.3.4, “Maintenance of Partitions”。 |
| `REBUILD PARTITION` | 否 | 是* | 否 | 支持 `ALGORITHM=INPLACE, LOCK={DEFAULT&#124;SHARED&#124;EXCLUSIVE}`。 |
| `REPAIR PARTITION` | 否 | 是 | 是 |  |
| `REMOVE PARTITIONING` | 否 | 否 | 否 | 允许 `ALGORITHM=COPY`，`LOCK={DEFAULT&#124;SHARED&#124;EXCLUSIVE}` |
| 分区子句 | 瞬时 | 就地 | 允许 DML | 备注 |

针对分区表的非分区在线`ALTER TABLE`操作遵循适用于常规表的相同规则。然而，`ALTER TABLE`对每个表分区执行在线操作，这会导致系统资源需求增加，因为操作在多个分区上执行。

有关`ALTER TABLE`分区子句的更多信息，请参阅分区选项，以及第 15.1.9.1 节，“ALTER TABLE 分区操作”。有关分区的一般信息，请参阅第二十六章，“*分区*”。
