# 15.1.20 CREATE TABLE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table.html)

15.1.20.1 CREATE TABLE 创建的文件

15.1.20.2 CREATE TEMPORARY TABLE 语句

15.1.20.3 CREATE TABLE ... LIKE 语句

15.1.20.4 CREATE TABLE ... SELECT 语句

15.1.20.5 外键约束

15.1.20.6 CHECK 约束

15.1.20.7 隐式列规范更改

15.1.20.8 CREATE TABLE 和生成列

15.1.20.9 二级索引和生成列

15.1.20.10 隐式列

15.1.20.11 生成的隐式主键

15.1.20.12 设置 NDB 注释选项

```sql
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] *tbl_name*
    (*create_definition*,...)
    [*table_options*]
    [*partition_options*]

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] *tbl_name*
    [(*create_definition*,...)]
    [*table_options*]
    [*partition_options*]
    [IGNORE | REPLACE]
    [AS] *query_expression*

CREATE [TEMPORARY] TABLE [IF NOT EXISTS] *tbl_name*
    { LIKE *old_tbl_name* | (LIKE *old_tbl_name*) }

*create_definition*: {
    *col_name* *column_definition*
  | {INDEX | KEY} [*index_name*] [*index_type*] (*key_part*,...)
      [*index_option*] ...
  | {FULLTEXT | SPATIAL} [INDEX | KEY] [*index_name*] (*key_part*,...)
      [*index_option*] ...
  | [CONSTRAINT [*symbol*]] PRIMARY KEY
      [*index_type*] (*key_part*,...)
      [*index_option*] ...
  | [CONSTRAINT [*symbol*]] UNIQUE [INDEX | KEY]
      [*index_name*] [*index_type*] (*key_part*,...)
      [*index_option*] ...
  | [CONSTRAINT [*symbol*]] FOREIGN KEY
      [*index_name*] (*col_name*,...)
      *reference_definition*
  | *check_constraint_definition*
}

*column_definition*: {
    *data_type* [NOT NULL | NULL] [DEFAULT {*literal* | (*expr*)} ]
      [VISIBLE | INVISIBLE]
      [AUTO_INCREMENT] [UNIQUE [KEY]] [[PRIMARY] KEY]
      [COMMENT '*string*']
      [COLLATE *collation_name*]
      [COLUMN_FORMAT {FIXED | DYNAMIC | DEFAULT}]
      [ENGINE_ATTRIBUTE [=] '*string*']
      [SECONDARY_ENGINE_ATTRIBUTE [=] '*string*']
      [STORAGE {DISK | MEMORY}]
      [*reference_definition*]
      [*check_constraint_definition*]
  | *data_type*
      [COLLATE *collation_name*]
      [GENERATED ALWAYS] AS (*expr*)
      [VIRTUAL | STORED] [NOT NULL | NULL]
      [VISIBLE | INVISIBLE]
      [UNIQUE [KEY]] [[PRIMARY] KEY]
      [COMMENT '*string*']
      [*reference_definition*]
      [*check_constraint_definition*]
}

*data_type*:
    (see Chapter 13, Data Types)

*key_part*: {*col_name* [(*length*)] | (*expr*)} [ASC | DESC]

*index_type*:
    USING {BTREE | HASH}

*index_option*: {
    KEY_BLOCK_SIZE [=] *value*
  | *index_type*
  | WITH PARSER *parser_name*
  | COMMENT '*string*'
  | {VISIBLE | INVISIBLE}
  |ENGINE_ATTRIBUTE [=] '*string*'
  |SECONDARY_ENGINE_ATTRIBUTE [=] '*string*'
}

*check_constraint_definition*:
    [CONSTRAINT [*symbol*]] CHECK (*expr*) [[NOT] ENFORCED]

*reference_definition*:
    REFERENCES *tbl_name* (*key_part*,...)
      [MATCH FULL | MATCH PARTIAL | MATCH SIMPLE]
      [ON DELETE *reference_option*]
      [ON UPDATE *reference_option*]

*reference_option*:
    RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT

*table_options*:
    *table_option* [[,] *table_option*] ...

*table_option*: {
    AUTOEXTEND_SIZE [=] *value*
  | AUTO_INCREMENT [=] *value*
  | AVG_ROW_LENGTH [=] *value*
  | [DEFAULT] CHARACTER SET [=] *charset_name*
  | CHECKSUM [=] {0 | 1}
  | [DEFAULT] COLLATE [=] *collation_name*
  | COMMENT [=] '*string*'
  | COMPRESSION [=] {'ZLIB' | 'LZ4' | 'NONE'}
  | CONNECTION [=] '*connect_string*'
  | {DATA | INDEX} DIRECTORY [=] '*absolute path to directory*'
  | DELAY_KEY_WRITE [=] {0 | 1}
  | ENCRYPTION [=] {'Y' | 'N'}
  | ENGINE [=] *engine_name*
  | ENGINE_ATTRIBUTE [=] '*string*'
  | INSERT_METHOD [=] { NO | FIRST | LAST }
  | KEY_BLOCK_SIZE [=] *value*
  | MAX_ROWS [=] *value*
  | MIN_ROWS [=] *value*
  | PACK_KEYS [=] {0 | 1 | DEFAULT}
  | PASSWORD [=] '*string*'
  | ROW_FORMAT [=] {DEFAULT | DYNAMIC | FIXED | COMPRESSED | REDUNDANT | COMPACT}
  | START TRANSACTION 
  | SECONDARY_ENGINE_ATTRIBUTE [=] '*string*'
  | STATS_AUTO_RECALC [=] {DEFAULT | 0 | 1}
  | STATS_PERSISTENT [=] {DEFAULT | 0 | 1}
  | STATS_SAMPLE_PAGES [=] *value*
  | *tablespace_option*
  | UNION [=] (*tbl_name*[,*tbl_name*]...)
}

*partition_options*:
    PARTITION BY
        { [LINEAR] HASH(*expr*)
        | [LINEAR] KEY [ALGORITHM={1 | 2}] (*column_list*)
        | RANGE{(*expr*) | COLUMNS(*column_list*)}
        | LIST{(*expr*) | COLUMNS(*column_list*)} }
    [PARTITIONS *num*]
    [SUBPARTITION BY
        { [LINEAR] HASH(*expr*)
        | [LINEAR] KEY [ALGORITHM={1 | 2}] (*column_list*) }
      [SUBPARTITIONS *num*]
    ]
    [(*partition_definition* [, *partition_definition*] ...)]

*partition_definition*:
    PARTITION *partition_name*
        [VALUES
            {LESS THAN {(*expr* | *value_list*) | MAXVALUE}
            |
            IN (*value_list*)}]
        [[STORAGE] ENGINE [=] *engine_name*]
        [COMMENT [=] '*string*' ]
        [DATA DIRECTORY [=] '*data_dir*']
        [INDEX DIRECTORY [=] '*index_dir*']
        [MAX_ROWS [=] *max_number_of_rows*]
        [MIN_ROWS [=] *min_number_of_rows*]
        [TABLESPACE [=] tablespace_name]
        [(*subpartition_definition* [, *subpartition_definition*] ...)]

*subpartition_definition*:
    SUBPARTITION *logical_name*
        [[STORAGE] ENGINE [=] *engine_name*]
        [COMMENT [=] '*string*' ]
        [DATA DIRECTORY [=] '*data_dir*']
        [INDEX DIRECTORY [=] '*index_dir*']
        [MAX_ROWS [=] *max_number_of_rows*]
        [MIN_ROWS [=] *min_number_of_rows*]
        [TABLESPACE [=] tablespace_name]

*tablespace_option*:
    TABLESPACE *tablespace_name* [STORAGE DISK]
  | [TABLESPACE *tablespace_name*] STORAGE MEMORY

*query_expression:*
    SELECT ...   (*Some valid select or union statement*)
```

`CREATE TABLE` 创建具有给定名称的表。您必须对表具有 `CREATE` 权限。

默认情况下，表在默认数据库中使用 `InnoDB` 存储引擎创建。如果表已存在、没有默认数据库或数据库不存在，则会出现错误。

MySQL 对表的数量没有限制。底层文件系统可能对代表表的文件数量有限制。各个存储引擎可能会施加特定于引擎的约束。`InnoDB` 允许最多 40 亿个表。

有关表的物理表示信息，请参阅 第 15.1.20.1 节“CREATE TABLE 创建的文件”.

`CREATE TABLE` 语句有几个方面，在本节的以下主题中描述：

+   表名

+   临时表

+   表克隆和复制

+   列数据类型和属性

+   索引、外键和 CHECK 约束

+   表选项

+   表分区

#### 表名

+   `*`tbl_name`*`

    表名可以指定为*`db_name.tbl_name`*，以在特定数据库中创建表。无论是否存在默认数据库，都可以使用此方法，假设数据库存在。如果使用带引号的标识符，请分别引用数据库和表名。例如，写成``mydb`.`mytbl``，而不是``mydb.mytbl``。

    可接受的表名规则在第 11.2 节，“模式对象名称”中给出。

+   `IF NOT EXISTS`

    如果表存在，则防止出现错误。但是，并没有验证现有表的结构是否与 `CREATE TABLE` 语句指示的结构完全相同。

#### 临时表

在创建表时，可以使用 `TEMPORARY` 关键字。`TEMPORARY` 表仅在当前会话中可见，并在会话关闭时自动删除。有关更多信息，请参见第 15.1.20.2 节，“CREATE TEMPORARY TABLE 语句”。

#### 表克隆和复制

+   `LIKE`

    使用 `CREATE TABLE ... LIKE` 根据另一个表的定义创建一个空表，包括原始表中定义的任何列属性和索引：

    ```sql
    CREATE TABLE *new_tbl* LIKE *orig_tbl*;
    ```

    更多信息，请参见第 15.1.20.3 节，“CREATE TABLE ... LIKE 语句”。

+   `[AS] *`query_expression`*`

    要从一个表创建另一个表，请在 `CREATE TABLE` 语句末尾添加一个 `SELECT` 语句：

    ```sql
    CREATE TABLE *new_tbl* AS SELECT * FROM *orig_tbl*;
    ```

    更多信息，请参见第 15.1.20.4 节，“CREATE TABLE ... SELECT 语句”。

+   `IGNORE | REPLACE`

    `IGNORE` 和 `REPLACE` 选项指示在使用 `SELECT` 语句复制表时如何处理重复唯一键值的行。

    更多信息，请参见第 15.1.20.4 节，“CREATE TABLE ... SELECT 语句”。

#### 列数据类型和属性

每个表的列有一个硬限制为 4096 列，但对于给定表，有效最大值可能会更少，并取决于第 10.4.7 节，“表列计数和行大小限制”中讨论的因素。

+   `*`data_type`*`

    *`data_type`* 代表列定义中的数据类型。有关指定列数据类型的语法以及每种类型属性的详细描述，请参见第十三章，*数据类型*。

    +   一些属性不适用于所有数据类型。`AUTO_INCREMENT` 仅适用于整数和浮点类型。在 MySQL 8.0.17 中，使用 `AUTO_INCREMENT` 与 `FLOAT` 或 `DOUBLE` 列已被弃用；预计在未来的 MySQL 版本中将删除对其的支持。

        在 MySQL 8.0.13 之前，`DEFAULT` 不适用于 `BLOB`, `TEXT`, `GEOMETRY` 和 `JSON` 类型。

    +   字符数据类型（`CHAR`, `VARCHAR`, `TEXT`, `ENUM`, `SET` 和任何同义词）可以包括 `CHARACTER SET` 来指定列的字符集。`CHARSET` 是 `CHARACTER SET` 的同义词。可以使用 `COLLATE` 属性指定字符集的排序规则，以及其他任何属性。有关详细信息，请参见 第十二章，*字符集、排序规则、Unicode*。示例：

        ```sql
        CREATE TABLE t (c CHAR(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_bin);
        ```

        MySQL 8.0 解释字符列定义中的长度规范为字符。`BINARY` 和 `VARBINARY` 的长度以字节为单位。

    +   对于 `CHAR`、`VARCHAR`、`BINARY` 和 `VARBINARY` 列，可以创建仅使用列值前导部分的索引，使用 `*`col_name`*(*`length`*)` 语法指定索引前缀长度。`BLOB` 和 `TEXT` 列也可以被索引，但必须给出前缀长度。对于非二进制字符串类型，前缀长度以字符为单位，对于二进制字符串类型，前缀长度以字节为单位。也就是说，对于 `CHAR`、`VARCHAR` 和 `TEXT` 列，索引条目由每个列值的前 *`length`* 个字符组成，对于 `BINARY`、`VARBINARY` 和 `BLOB` 列，索引条目由每个列值的前 *`length`* 个字节组成。像这样仅对列值前缀进行索引可以使索引文件变得更小。有关索引前缀的更多信息，请参见 第 15.1.15 节，“CREATE INDEX 语句”。

        仅 `InnoDB` 和 `MyISAM` 存储引擎支持对 `BLOB` 和 `TEXT` 列进行索引。例如：

        ```sql
        CREATE TABLE test (blob_col BLOB, INDEX(blob_col(10)));
        ```

        如果指定的索引前缀超过最大列数据类型大小，`CREATE TABLE` 将处理索引如下：

        +   对于非唯一索引，如果启用了严格的 SQL 模式，则会发生错误，或者索引长度会减小以符合最大列数据类型大小，并产生警告（如果未启用严格的 SQL 模��）。

        +   对于唯一索引，无论 SQL 模式如何，都会发生错误，因为减少索引长度可能会导致插入不符合指定唯一性要求的非唯一条目。

    +   `JSON` 列无法被索引。您可以通过在生成的列上创建索引来绕过此限制，该生成的列从 `JSON` 列中提取标量值。详细示例请参见 在生成的列上创建索引以提供 JSON 列索引。

+   `NOT NULL | NULL`

    如果未指定 `NULL` 或 `NOT NULL`，则该列被视为已指定 `NULL`。

    在 MySQL 8.0 中，只有`InnoDB`、`MyISAM`和`MEMORY`存储引擎支持可以具有`NULL`值的列上的索引。在其他情况下，必须将索引列声明为`NOT NULL`，否则会产生错误。

+   `DEFAULT`

    为列指定默认值。有关默认值处理的更多信息，包括列定义不包含显式`DEFAULT`值的情况，请参见第 13.6 节，“数据类型默认值”。

    如果启用了`NO_ZERO_DATE`或`NO_ZERO_IN_DATE` SQL 模式，并且日期值默认值不符合该模式，则`CREATE TABLE`在严格 SQL 模式未启用时会产生警告，在启用严格模式时会产生错误。例如，启用`NO_ZERO_IN_DATE`，`c1 DATE DEFAULT '2010-00-00'`会产生警告。

+   `VISIBLE`、`INVISIBLE`

    指定列的可见性。如果两个关键字都不存在，则默认为`VISIBLE`。表必须至少有一列可见。尝试使所有列不可见会产生错误。有关更多信息，请参见第 15.1.20.10 节，“不可见列”。

    `VISIBLE` 和 `INVISIBLE` 关键字从 MySQL 8.0.23 开始可用。在 MySQL 8.0.23 之前，所有列都是可见的。

+   `AUTO_INCREMENT`

    整数或浮点列可以具有附加属性`AUTO_INCREMENT`。当您将`NULL`（推荐）或`0`值插入到索引的`AUTO_INCREMENT`列中时，该列将设置为下一个序列值。通常这是`*`value`*+1`，其中*`value`*是当前表中列的最大值。`AUTO_INCREMENT`序列从`1`开始。

    在插入行后检索`AUTO_INCREMENT`值，请使用`LAST_INSERT_ID()` SQL 函数或`mysql_insert_id()` C API 函数。请参见第 14.15 节，“信息函数”，以及 mysql_insert_id()。

    如果启用了`NO_AUTO_VALUE_ON_ZERO` SQL 模式，则可以将`0`存储在`AUTO_INCREMENT`列中，而不生成新的序列值。参见第 7.1.11 节，“服务器 SQL 模式”。

    每个表只能有一个`AUTO_INCREMENT`列，它必须被索引，并且不能有默认值。`AUTO_INCREMENT`列只有在包含正值时才能正常工作。插入负数被视为插入一个非常大的正数。这样做是为了避免当数字从正数“环绕”到负数时出现精度问题，并确保您不会意外地获得一个包含`0`的`AUTO_INCREMENT`列。

    对于`MyISAM`表，您可以在多列键中指定一个`AUTO_INCREMENT`次要列。参见 Section 5.6.9, “Using AUTO_INCREMENT”。

    要使 MySQL 与某些 ODBC 应用程序兼容，您可以使用以下查询找到最后插入行的`AUTO_INCREMENT`值：

    ```sql
    SELECT * FROM *tbl_name* WHERE *auto_col* IS NULL
    ```

    此方法要求`sql_auto_is_null`变量未设置为 0。请参见 Section 7.1.8, “Server System Variables”。

    有关`InnoDB`和`AUTO_INCREMENT`的信息，请参见 Section 17.6.1.6, “AUTO_INCREMENT Handling in InnoDB”。有关`AUTO_INCREMENT`和 MySQL 复制的信息，请参见 Section 19.5.1.1, “Replication and AUTO_INCREMENT”。

+   `COMMENT`

    可以使用`COMMENT`选项为列指定长达 1024 个字符的注释。该注释将显示在`SHOW CREATE TABLE`和`SHOW FULL COLUMNS`语句中。它还显示在信息模式`COLUMNS`表的`COLUMN_COMMENT`列中。

+   `COLUMN_FORMAT`

    在 NDB Cluster 中，还可以使用`COLUMN_FORMAT`为`NDB`表的各个列指定数据存储格式。可接受的列格式包括`FIXED`、`DYNAMIC`和`DEFAULT`。`FIXED`用于指定固定宽度存储，`DYNAMIC`允许列为可变宽度，`DEFAULT`使列使用由列的数据类型确定的固定宽度或可变宽度存储（可能被`ROW_FORMAT`修饰符覆盖）。

    对于`NDB`表，`COLUMN_FORMAT`的默认值为`FIXED`。

    在 NDB Cluster 中，使用`COLUMN_FORMAT=FIXED`定义的列的最大可能偏移量为 8188 字节。有关更多信息和可能的解决方法，请参见 Section 25.2.7.5, “Limits Associated with Database Objects in NDB Cluster”。

    `COLUMN_FORMAT` 目前对使用除 `NDB` 之外的存储引擎的表的列没有影响。MySQL 8.0 会默默忽略 `COLUMN_FORMAT`。

+   `ENGINE_ATTRIBUTE` 和 `SECONDARY_ENGINE_ATTRIBUTE` 选项（自 MySQL 8.0.21 起可用）用于指定主存储引擎和辅助存储引擎的列属性。这些选项保留供将来使用。

    允许的值是包含有效 `JSON` 文档或空字符串（''）的字符串文字。无效的 `JSON` 将被拒绝。

    ```sql
    CREATE TABLE t1 (c1 INT ENGINE_ATTRIBUTE='{"*key*":"*value*"}');
    ```

    `ENGINE_ATTRIBUTE` 和 `SECONDARY_ENGINE_ATTRIBUTE` 的值可以重复使用而不会出错。在这种情况下，将使用最后指定的值。

    `ENGINE_ATTRIBUTE` 和 `SECONDARY_ENGINE_ATTRIBUTE` 的值不会被服务器检查，也不会在表的存储引擎更改时被清除。

+   `STORAGE`

    对于 `NDB` 表，可以通过使用 `STORAGE` 子句来指定列是存储在磁盘上还是内存中。`STORAGE DISK` 导致列存储在磁盘上，而 `STORAGE MEMORY` 导致使用内存存储。仍然必须在使用的 `CREATE TABLE` 语句中包含 `TABLESPACE` 子句：

    ```sql
    mysql> CREATE TABLE t1 (
     ->     c1 INT STORAGE DISK,
     ->     c2 INT STORAGE MEMORY
     -> ) ENGINE NDB;
    ERROR 1005 (HY000): Can't create table 'c.t1' (errno: 140) 
    mysql> CREATE TABLE t1 (
     ->     c1 INT STORAGE DISK,
     ->     c2 INT STORAGE MEMORY
     -> ) TABLESPACE ts_1 ENGINE NDB;
    Query OK, 0 rows affected (1.06 sec)
    ```

    对于 `NDB` 表，`STORAGE DEFAULT` 等同于 `STORAGE MEMORY`。

    `STORAGE` 子句对使用除 `NDB` 之外的存储引擎的表没有影响。`STORAGE` 关键字仅在随 NDB Cluster 一起提供的 **mysqld** 构建中受支持；在 MySQL 的任何其他版本中都不被识别，任何尝试使用 `STORAGE` 关键字都会导致语法错误。

+   `GENERATED ALWAYS`

    用于指定生成列表达式。有关生成列的信息，请参阅第 15.1.20.8 节，“CREATE TABLE 和 Generated Columns”。

    存储生成列 可以被索引。`InnoDB` 支持对虚拟生成列进行二级索引。请参阅第 15.1.20.9 节，“Secondary Indexes and Generated Columns”。

#### 索引、外键和检查约束

创建索引、外键和`CHECK`约束时适用几个关键字。除了以下描述外，有关一般背景，请参阅 Section 15.1.15, “CREATE INDEX Statement”，Section 15.1.20.5, “FOREIGN KEY Constraints”和 Section 15.1.20.6, “CHECK Constraints”。

+   `CONSTRAINT *`symbol`*`

    可以使用`CONSTRAINT *`symbol`*`子句来命名约束。如果未给出该子句，或者在`CONSTRAINT`关键字后未包含*`symbol`*，MySQL 会自动生成约束名称，以下列出的情况除外。如果使用了*`symbol`*值，必须对每个模式（数据库）的每种约束类型保持唯一性。重复的*`symbol`*会导致错误。另请参阅有关生成约束标识符长度限制的讨论，详见 Section 11.2.1, “Identifier Length Limits”。

    注意

    如果在外键定义中未给出`CONSTRAINT *`symbol`*`子句，或者在`CONSTRAINT`关键字后未包含*`symbol`*，MySQL 在 MySQL 8.0.15 之前使用外键索引名称，并在此后自动生成约束名称。

    SQL 标准规定所有类型的约束（主键、唯一索引、外键、检查）属于同一命名空间。在 MySQL 中，每种约束类型在每个模式中都有自己的命名空间。因此，每种约束类型的名称必须在每个模式中保持唯一，但不同类型的约束可以具有相同的名称。

+   `PRIMARY KEY`

    唯一索引，所有关键列必须定义为`NOT NULL`。如果它们没有明确声明为`NOT NULL`，MySQL 会隐式（并悄无声息地）声明它们。一个表只能有一个`PRIMARY KEY`。`PRIMARY KEY`的名称始终为`PRIMARY`，因此不能用作任何其他类型索引的名称。

    如果没有`PRIMARY KEY`，应用程序要求表中的`PRIMARY KEY`，MySQL 将第一个没有`NULL`列的`UNIQUE`索引作为`PRIMARY KEY`返回。

    在`InnoDB`表中，保持`PRIMARY KEY`简短，以减少次要索引的存储开销。每个次要索引条目都包含相应行的主键列的副本。（参见 Section 17.6.2.1, “Clustered and Secondary Indexes”。）

    在创建的表中，首先放置`PRIMARY KEY`，然后是所有`UNIQUE`索引，然后是非唯一索引。这有助于 MySQL 优化器优先考虑使用哪个索引，并更快地检测重复的`UNIQUE`键。

    `PRIMARY KEY`可以是多列索引。但是，您不能在列规范中使用`PRIMARY KEY`关键属性创建多列索引。这样做只会将该单列标记为主键。您必须使用单独的`PRIMARY KEY(*`key_part`*, ...)`子句。

    如果表具有由整数类型组成的单列`PRIMARY KEY`或`UNIQUE NOT NULL`索引，您可以在`SELECT`语句中使用`_rowid`来引用索引列，如 Unique Indexes 中所述。

    在 MySQL 中，`PRIMARY KEY`的名称是`PRIMARY`。对于其他索引，如果您没有分配名称，则该索引将被分配与第一个索引列相同的名称，并带有可选后缀（`_2`，`_3`，`...`）以使其唯一。您可以使用`SHOW INDEX FROM *`tbl_name`*`查看表的索引名称。参见 Section 15.7.7.22, “SHOW INDEX Statement”。

+   `KEY | INDEX`

    `KEY`通常是`INDEX`的同义词。在列定义中给出时，`PRIMARY KEY`关键属性也可以简单地指定为`KEY`。这是为了与其他数据库系统兼容而实现的。

+   `UNIQUE`

    `UNIQUE`索引创建一个约束，使索引中的所有值必须是不同的。如果尝试添加具有与现有行匹配的键值的新行，则会发生错误。对于所有引擎，`UNIQUE`索引允许对可以包含`NULL`的列进行多个`NULL`值。如果为`UNIQUE`索引的列指定前缀值，则列值必须在前缀长度内是唯一的。

    如果表具有由整数类型组成的单列`PRIMARY KEY`或`UNIQUE NOT NULL`索引，您可以在`SELECT`语句中使用`_rowid`来引用索引列，如 Unique Indexes 中所述。

+   `FULLTEXT`

    `FULLTEXT`索引是用于全文搜索的特殊类型的索引。只有`InnoDB`和`MyISAM`存储引擎支持`FULLTEXT`索引。它们只能从`CHAR`、`VARCHAR`和`TEXT`列创建。索引始终在整个列上进行；不支持列前缀索引，如果指定了任何前缀长度，则会被忽略。有关操作的详细信息，请参见第 14.9 节，“全文搜索函数”。可以指定`WITH PARSER`子句作为*`index_option`*值，以将解析器插件与索引关联，如果全文索引和搜索操作需要特殊处理。此子句仅适用于`FULLTEXT`索引。`InnoDB`和`MyISAM`支持全文解析器插件。有关更多信息，请参见全文解析器插件和编写全文解析器插件。

+   `空间`

    您可以在空间数据类型上创建`SPATIAL`索引。空间类型仅支持`InnoDB`和`MyISAM`表，并且索引列必须声明为`NOT NULL`。请参见第 13.4 节，“空间数据类型”。

+   `外键`

    MySQL 支持外键，允许您在表之间交叉引用相关数据，并支持外键约束，有助于保持这些分散数据的一致性。有关定义和选项信息，请参见*`reference_definition`*，以及*`reference_option`*。

    使用`InnoDB`存储引擎的分区表不支持外键。有关更多信息，请参见第 26.6 节，“分区的限制和限制”。

+   `CHECK`

    `CHECK`子句使得可以创建用于检查表行中数据值的约束。请参见第 15.1.20.6 节，“CHECK 约束”。

+   `*`key_part`*`

    +   *`key_part`*规范可以以`ASC`或`DESC`结尾，以指定索引值是按升序还是降序存储。如果没有给出顺序说明符，则默认为升序。

    +   前缀，由*`length`*属性定义，对于使用`REDUNDANT`或`COMPACT`行格式的`InnoDB`表最多可以达到 767 字节长。对于使用`DYNAMIC`或`COMPRESSED`行格式的`InnoDB`表，前缀长度限制为 3072 字节。对于`MyISAM`表，前缀长度限制为 1000 字节。

        前缀*限制*以字节为单位。但是，在`CREATE TABLE`、`ALTER TABLE`和`CREATE INDEX`语句中的索引规范中，对于非二进制字符串类型（`CHAR`、`VARCHAR`、`TEXT`），前缀*长度*被解释为字符数，对于二进制字符串类型（`BINARY`、`VARBINARY`、`BLOB`），前缀*长度*被解释为字节数。在为使用多字节字符集的非二进制字符串列指定前缀长度时，请考虑这一点。

    +   从 MySQL 8.0.17 开始，*`key_part`*规范的*`expr`*可以采用形式`(CAST *`json_path`* AS *`type`* ARRAY)`，以在`JSON`列上创建多值索引。多值索引提供了有关创建、使用以及多值索引的限制和限制的详细信息。

+   `*`index_type`*`

    一些存储引擎允许在创建索引时指定索引类型。*`index_type`*指定符的语法是`USING *`type_name`*`。

    例子：

    ```sql
    CREATE TABLE lookup
      (id INT, INDEX USING BTREE (id))
      ENGINE = MEMORY;
    ```

    `USING`的首选位置是在索引列列表之后。它可以在列列表之前给出，但是在该位置使用该选项的支持已被弃用，您应该期望在未来的 MySQL 版本中将其移除。

+   `*`index_option`*`

    *`index_option`*值指定索引的附加选项。

    +   `KEY_BLOCK_SIZE`

        对于`MyISAM`表，`KEY_BLOCK_SIZE`可选地指定用于索引键块的字节大小。该值被视为提示；如果需要，可以使用不同的大小。为单个索引定义指定的`KEY_BLOCK_SIZE`值会覆盖表级别的`KEY_BLOCK_SIZE`值。

        有关表级别`KEY_BLOCK_SIZE`属性的信息，请参阅表选项。

    +   `WITH PARSER`

        `WITH PARSER`选项只能与`FULLTEXT`索引一起使用。如果全文索引和搜索操作需要特殊处理，则将解析器插件与索引关联起来。`InnoDB`和`MyISAM`支持全文解析器插件。如果您有一个带有关联全文解析器插件的`MyISAM`表，您可以使用`ALTER TABLE`将表转换为`InnoDB`。

    +   `COMMENT`

        索引定义可以包括最多 1024 个字符的可选注释。

        你可以使用`*`index_option`*` `COMMENT`子句为单个索引设置`InnoDB` `MERGE_THRESHOLD`值。参见第 17.8.11 节，“配置索引页合并阈值”。

    +   `VISIBLE`，`INVISIBLE`

        指定索引可见性。索引默认可见。不可见索引不会被优化器使用。索引可见性的规范适用于主键之外的索引（显式或隐式）。更多信息，请参见第 10.3.12 节，“不可见索引”。

    +   `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`选项（自 MySQL 8.0.21 起可用）用于指定主要和次要存储引擎的索引属性。这些选项保留供将来使用。

    有关可接受的*`index_option`*值的更多信息，请参见第 15.1.15 节，“CREATE INDEX 语句”。有关索引的更多信息，请参见第 10.3.1 节，“MySQL 如何使用索引”。

+   `*`reference_definition`*`

    有关*`reference_definition`*语法详细信息和示例，请参见第 15.1.20.5 节，“外键约束”。

    `InnoDB`和`NDB`表支持检查外键约束。引用表的列必须始终明确命名。外键支持`ON DELETE`和`ON UPDATE`操作。有关更详细的信息和示例，请参见第 15.1.20.5 节，“外键约束”。

    对于其他存储引擎，MySQL 服务器会解析并忽略`CREATE TABLE`语句中的`FOREIGN KEY`语法。

    重要

    对于熟悉 ANSI/ISO SQL 标准的用户，请注意，包括`InnoDB`在内的任何存储引擎都不识别或强制执行用于参照完整性约束定义的`MATCH`子句。明确指定`MATCH`子句不会产生指定的效果，并且还会导致`ON DELETE`和`ON UPDATE`子句被忽略。因此，应避免指定`MATCH`。

    SQL 标准中的`MATCH`子句控制如何处理复合（多列）外键中的`NULL`值与主键进行比较。`InnoDB`基本上实现了`MATCH SIMPLE`定义的语义，允许外键全部或部分为`NULL`。在这种情况下，包含这种外键的（子表）行允许被插入，并且不匹配参考（父表）中的任何行。可以使用触发器实现其他语义。

    此外，MySQL 要求被引用的列需要建立索引以提高性能。然而，`InnoDB`不强制要求被引用的列声明为`UNIQUE`或`NOT NULL`。对于引用非唯一键或包含`NULL`值的键的外键引用的处理在`UPDATE`或`DELETE CASCADE`等操作中没有明确定义。建议使用只引用既`UNIQUE`（或`PRIMARY`）又`NOT NULL`键的外键。

    MySQL 解析但忽略“内联`REFERENCES`规范”（如 SQL 标准中定义的）中的引用，其中引用被定义为列规范的一部分。只有在作为单独的`FOREIGN KEY`规范的一部分指定时，MySQL 才接受`REFERENCES`子句。有关更多信息，请参见 Section 1.6.2.3, “FOREIGN KEY Constraint Differences”。

+   `*`reference_option`*`

    有关`RESTRICT`、`CASCADE`、`SET NULL`、`NO ACTION`和`SET DEFAULT`选项的信息，请参见 Section 15.1.20.5, “FOREIGN KEY Constraints”。

#### 表选项

表选项用于优化表的行为。在大多数情况下，您不必指定任何选项。除非另有说明，这些选项适用于所有存储引擎。不适用于给定存储引擎的选项可能会被接受并记入表定义中。如果以后使用`ALTER TABLE`将表转换为使用不同的存储引擎，则这些选项将适用。

+   `ENGINE`

    指定表的存储引擎，可以使用以下表中显示的名称之一。引擎名称可以是带引号或不带引号的。带引号的名称`'DEFAULT'`会被识别但会被忽略。

    | 存储引擎 | 描述 |
    | --- | --- |
    | `InnoDB` | 具有行锁定和外键的事务安全表。新表的默认存储引擎。参见第十七章，*InnoDB 存储引擎*，特别是如果您有 MySQL 经验但是对`InnoDB`不熟悉，请参见第 17.1 节，“InnoDB 简介”。 |
    | `MyISAM` | 主要用于只读或读取频繁工作负载的二进制便携式存储引擎。参见第 18.2 节，“MyISAM 存储引擎”。 |
    | `MEMORY` | 这种存储引擎的数据仅存储在内存中。参见第 18.3 节，“MEMORY 存储引擎”。 |
    | `CSV` | 以逗号分隔值格式存储行的表。参见第 18.4 节，“CSV 存储引擎”。 |
    | `ARCHIVE` | 存档存储引擎。参见第 18.5 节，“ARCHIVE 存储引擎”。 |
    | `EXAMPLE` | 一个示例引擎。参见第 18.9 节，“EXAMPLE 存储引擎”。 |
    | `FEDERATED` | 访问远程表的存储引擎。参见第 18.8 节，“FEDERATED 存储引擎”。 |
    | `HEAP` | 这是`MEMORY`的同义词。 |
    | `MERGE` | 一组作为一个表使用的`MyISAM`表。也称为`MRG_MyISAM`。参见第 18.7 节，“MERGE 存储引擎”。 |
    | `NDB` | 集群化、容错、基于内存的表，支持事务和外键。也称为`NDBCLUSTER`。参见第二十五章，*MySQL NDB Cluster 8.0*。 |
    | 存储引擎 | 描述 |

    默认情况下，如果指定了不可用的存储引擎，语句将失败并显示错误。您可以通过从服务器 SQL 模式中删除`NO_ENGINE_SUBSTITUTION`来覆盖此行为，以便 MySQL 允许将指定的引擎替换为默认存储引擎。通常在这种情况下，默认值为`default_storage_engine`系统变量的`InnoDB`。当禁用`NO_ENGINE_SUBSTITUTION`时，如果未遵守存储引擎规范，则会发出警告。

+   `AUTOEXTEND_SIZE`

    定义`InnoDB`在表空间满时扩展的量。在 MySQL 8.0.23 中引入。设置必须是 4MB 的倍数。默认设置为 0，这将导致表空间根据隐式默认行为进行扩展。有关更多信息，请参见第 17.6.3.9 节，“表空间 AUTOEXTEND_SIZE 配置”。

+   `AUTO_INCREMENT`

    表的初始`AUTO_INCREMENT`值。在 MySQL 8.0 中，这适用于`MyISAM`、`MEMORY`、`InnoDB`和`ARCHIVE`表。对于不支持`AUTO_INCREMENT`表选项的引擎，要设置第一个自动增量值，需要在创建表后插入一个值比所需值小 1 的“虚拟”行，然后删除虚拟行。

    对于支持在`CREATE TABLE`语句中使用`AUTO_INCREMENT`表选项的引擎，您也可以使用`ALTER TABLE *`tbl_name`* AUTO_INCREMENT = *`N`*`来重置`AUTO_INCREMENT`值。该值不能低于当前列中的最大值。

+   `AVG_ROW_LENGTH`

    表的平均行长度的近似值。只需为具有可变大小行的大表设置此值。

    当您创建一个`MyISAM`表时，MySQL 使用`MAX_ROWS`和`AVG_ROW_LENGTH`选项的乘积来决定生成的表有多大。如果您没有指定任何选项，`MyISAM`数据和索引文件的最大大小默认为 256TB。（如果您的操作系统不支持那么大的文件，表大小将受文件大小限制。）如果您想要减小指针大小以使索引更小更快，并且您实际上不需要大文件，您可以通过设置`myisam_data_pointer_size`系统变量来减小默认指针大小。（参见第 7.1.8 节，“服务器系统变量”。）如果您希望所有表都能超过默认限制增长，并且愿意让表比必要的稍慢和稍大，您可以通过设置此变量来增加默认指针大小。将值设置为 7 允许表大小达到 65,536TB。

+   `[DEFAULT] 字符集`

    为表指定默认字符集。`CHARSET`是`CHARACTER SET`的同义词。如果字符集名称为`DEFAULT`，则使用数据库字符集。

+   `校验和`

    如果您希望 MySQL 为所有行维护一个实时校验和（即 MySQL 在表更改时自动更新的校验和），请将此设置为 1。这使得表更新稍慢，但也更容易找到损坏的表。`CHECKSUM TABLE`语句报告校验和。（仅适用于`MyISAM`。）

+   `[DEFAULT] 校对规则`

    为表指定默认排序规则。

+   `注释`

    表的注释，最长可达 2048 个字符。

    您可以使用 `*`table_option`*` `COMMENT` 子句为表设置 `InnoDB` `MERGE_THRESHOLD` 值。请参阅 Section 17.8.11, “Configuring the Merge Threshold for Index Pages”。

    **设置 NDB_TABLE 选项。** 在创建 `NDB` 表的 `CREATE TABLE` 或修改一个的 `ALTER TABLE` 语句中，表注释也可用于指定 `NDB_TABLE` 选项 `NOLOGGING`、`READ_BACKUP`、`PARTITION_BALANCE` 或 `FULLY_REPLICATED` 中的一个到四个作为一组名称-值对，如果需要，用逗号分隔，紧随以 `NDB_TABLE=` 开头的引用注释文本。以下是使用此语法的示例语句（强调文本）：

    ```sql
    CREATE TABLE t1 (
        c1 INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
        c2 VARCHAR(100),
        c3 VARCHAR(100) )
    ENGINE=NDB
    *COMMENT="NDB_TABLE=READ_BACKUP=0,PARTITION_BALANCE=FOR_RP_BY_NODE"*;
    ```

    引号字符串内不允许有空格。字符串不区分大小写。

    该注释将显示在 `SHOW CREATE TABLE` 的输出中。注释的文本也作为 MySQL 信息模式 `TABLES` 表的 TABLE_COMMENT 列中可用。

    这种注释语法也适用于 `NDB` 表的 `ALTER TABLE` 语句。请注意，与 `ALTER TABLE` 一起使用的表注释会替换表先前可能具有的任何现有注释。

    不支持在表注释中设置 `MERGE_THRESHOLD` 选项用于 `NDB` 表（将被忽略）。

    有关完整的语法信息和示例，请参阅 Section 15.1.20.12, “Setting NDB Comment Options”。

+   `COMPRESSION`

    用于 `InnoDB` 表的页面级压缩的压缩算法。支持的值包括 `Zlib`、`LZ4` 和 `None`。`COMPRESSION` 属性是通过透明页面压缩功能引入的。页面压缩仅支持驻留在 file-per-table 表空间中的 `InnoDB` 表，并且仅在支持稀疏文件和孔打孔的 Linux 和 Windows 平台上可用。有关更多信息，请参阅 Section 17.9.2, “InnoDB Page Compression”。

+   `CONNECTION`

    `FEDERATED` 表的连接字符串。

    注意

    旧版本的 MySQL 使用 `COMMENT` 选项作为连接字符串的注释。

+   `DATA DIRECTORY`、`INDEX DIRECTORY`

    对于`InnoDB`，`DATA DIRECTORY='*`目录`*'`子句允许在数据目录之外创建表。必须启用`innodb_file_per_table`变量才能使用`DATA DIRECTORY`子句。必须指定完整的目录路径。截至 MySQL 8.0.21，指定的目录必须为`InnoDB`所知。有关更多信息，请参阅第 17.6.1.2 节，“外部创建表”。

    在创建`MyISAM`表时，可以使用`DATA DIRECTORY='*`目录`*'`子句，`INDEX DIRECTORY='*`目录`*'`子句，或两者。它们分别指定`MyISAM`表的数据文件和索引文件放置的位置。与`InnoDB`表不同，当使用`DATA DIRECTORY`或`INDEX DIRECTORY`选项创建`MyISAM`表时，MySQL 不会创建与数据库名称对应的子目录。文件将在指定的目录中创建。

    您必须具有`FILE`权限才能使用`DATA DIRECTORY`或`INDEX DIRECTORY`表选项。

    重要

    对于分区表，将忽略表级`DATA DIRECTORY`和`INDEX DIRECTORY`选项。（Bug #32091）

    仅当您未使用`--skip-symbolic-links`选项时，这些选项才有效。您的操作系统还必须具有可用的、线程安全的`realpath()`调用。有关更完整的信息，请参阅第 10.12.2.2 节，“在 Unix 上为 MyISAM 表使用符号链接”。

    如果使用没有`DATA DIRECTORY`选项创建`MyISAM`表，则`.MYD`文件将在数据库目录中创建。默认情况下，如果`MyISAM`在这种情况下找到现有的`.MYD`文件，则会覆盖它。对于使用没有`INDEX DIRECTORY`选项创建的表的`.MYI`文件也适用相同规则。要抑制此行为，请使用`--keep_files_on_create`选项启动服务器，在这种情况下，`MyISAM`不会覆盖现有文件，而是返回错误。

    如果使用`DATA DIRECTORY`或`INDEX DIRECTORY`选项创建`MyISAM`表，并且找到现有的`.MYD`或`.MYI`文件，`MyISAM`将始终返回错误，并且不会覆盖指定目录中的文件。

    重要

    您不能使用包含 MySQL 数据目录的路径名与`DATA DIRECTORY`或`INDEX DIRECTORY`一起使用。这包括分区表和单个表分区。（参见 Bug #32167。）

+   `DELAY_KEY_WRITE`

    如果您希望延迟表的键更新直到表关闭，请将此设置为 1。请参阅第 7.1.8 节，“服务器系统变量”中的`delay_key_write`系统变量的描述。（仅适用于`MyISAM`。）

+   `ENCRYPTION`

    `ENCRYPTION`子句用于启用或禁用`InnoDB`表的页面级数据加密。必须在启用加密之前安装和配置密钥环插件。在 MySQL 8.0.16 之前，只能在每个表的表空间中创建表时指定`ENCRYPTION`子句。从 MySQL 8.0.16 开始，也可以在通用表空间中创建表时指定`ENCRYPTION`子句。

    截至 MySQL 8.0.16，如果未指定`ENCRYPTION`子句，表将继承默认的模式加密。如果启用了`table_encryption_privilege_check`变量，则需要`TABLE_ENCRYPTION_ADMIN`权限才能创建具有与默认模式加密不同的`ENCRYPTION`子句设置的表。在通用表空间中创建表时，表和表空间加密必须匹配。

    从 MySQL 8.0.16 开始，在使用不支持加密的存储引擎时，不允许指定值为`'N'`或`''`以外的`ENCRYPTION`子句。以前，该子句被接受。

    有关更多信息，请参见第 17.13 节，“InnoDB 数据静态加密”。

+   `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`选项（自 MySQL 8.0.21 起可用）用于指定主存储引擎和次要存储引擎的表属性。这些选项保留供将来使用。

    允许的值是包含有效`JSON`文档的字符串文字或空字符串（''）。无效的`JSON`会被拒绝。

    ```sql
    CREATE TABLE t1 (c1 INT) ENGINE_ATTRIBUTE='{"*key*":"*value*"}';
    ```

    `ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`值可以重复而不会出错。在这种情况下，将使用最后指定的值。

    服务器不会检查`ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`值，也不会在更改表的存储引擎时清除这些值。

+   `INSERT_METHOD`

    如果要向`MERGE`表插入数据，必须使用`INSERT_METHOD`指定要将行插入的表。`INSERT_METHOD`仅适用于`MERGE`表的选项。使用`FIRST`或`LAST`的值将插入到第一个或最后一个表中，使用`NO`的值将阻止插入。请参见第 18.7 节，“MERGE 存储引擎”。

+   `KEY_BLOCK_SIZE`

    对于`MyISAM`表，`KEY_BLOCK_SIZE`可选地指定用于索引键块的字节大小。该值被视为提示；如果需要，可以使用不同的大小。为单个索引定义指定的`KEY_BLOCK_SIZE`值会覆盖表级别的`KEY_BLOCK_SIZE`值。

    对于`InnoDB`表，`KEY_BLOCK_SIZE`指定了用于压缩`InnoDB`表的页大小（以千字节为单位）。`KEY_BLOCK_SIZE`值被视为提示；如果必要，`InnoDB`可能会使用不同的大小。`KEY_BLOCK_SIZE`只能小于或等于`innodb_page_size`值。值为 0 表示默认的压缩页大小，即`innodb_page_size`值的一半。根据`innodb_page_size`，可能的`KEY_BLOCK_SIZE`值包括 0、1、2、4、8 和 16。更多信息请参见 Section 17.9.1, “InnoDB Table Compression”。

    Oracle 建议在为`InnoDB`表指定`KEY_BLOCK_SIZE`时启用`innodb_strict_mode`。当启用`innodb_strict_mode`时，指定无效的`KEY_BLOCK_SIZE`值会返回错误。如果禁用`innodb_strict_mode`，无效的`KEY_BLOCK_SIZE`值会导致警告，并且`KEY_BLOCK_SIZE`选项会被忽略。

    对于`SHOW TABLE STATUS`响应中的`Create_options`列报告表实际使用的`KEY_BLOCK_SIZE`，`SHOW CREATE TABLE`也是如此。

    `InnoDB`仅支持表级别的`KEY_BLOCK_SIZE`。

    `KEY_BLOCK_SIZE`不支持 32KB 和 64KB 的`innodb_page_size`值。`InnoDB`表压缩不支持这些页大小。

    在创建临时表时，`InnoDB`不支持`KEY_BLOCK_SIZE`选项。

+   `MAX_ROWS`

    您计划存储在表中的最大行数。这不是一个硬限制，而是一个提示，告诉存储引擎表必须至少能够存储这么多行。

    重要

    使用`MAX_ROWS`来控制表分区数量的`NDB`表已被弃用。为了向后兼容，它在后续版本中仍然受支持，但可能在未来的版本中被移除。请改用 PARTITION_BALANCE；参见设置 NDB_TABLE 选项。

    `NDB`存储引擎将此值视为最大值。如果计划创建非常大的 NDB Cluster 表（包含数百万行），应使用此选项确保`NDB`为存储表主键哈希的哈希表分配足够数量的索引槽，方法是设置`MAX_ROWS = 2 * *`rows`*`，其中*`rows`*是您预计插入表中的行数。

    最大的`MAX_ROWS`值为 4294967295；更大的值将被截断为此限制。

+   `MIN_ROWS`

    您计划在表中存储的最小行数。`MEMORY`存储引擎将此选项用作有关内存使用的提示。

+   `PACK_KEYS`

    仅对`MyISAM`表生效。如果想要更小的索引，请将此选项设置为 1。通常这会使更新变慢，读取变快。将选项设置为 0 会禁用所有键的压缩。将其设置为`DEFAULT`会告诉存储引擎只压缩长的`CHAR`、`VARCHAR`、`BINARY`或`VARBINARY`列。

    如果不使用`PACK_KEYS`，默认情况下会压缩字符串，但不会压缩数字。如果使用`PACK_KEYS=1`，数字也会被压缩。

    在压缩二进制数字键时，MySQL 使用前缀压缩：

    +   每个键需要额外一个字节来指示前一个键的多少个字节与下一个键相同。

    +   指向行的指针以高字节优先顺序直接存储在键后面，以提高压缩效果。

    这意味着如果在两个连续行上有许多相同的键，通常后续的“相同”键只需要两个字节（包括指向行的指针）。与普通情况下后续键需要`storage_size_for_key + pointer_size`（其中指针大小通常为 4）相比，如果有许多相同的数字，则通过前缀压缩可以获得显著的好处。如果所有键都完全不同，每个键会多使用一个字节，如果键不是可以有`NULL`值的键。在这种情况下，压缩的键长度存储在用于标记键是否为`NULL`的同一个字节中。

+   `PASSWORD`

    此选项未使用。

+   `ROW_FORMAT`

    定义了行存储的物理格式。

    在禁用严格模式时创建表时，如果指定的行格式不受支持，则使用存储引擎的默认行格式。表的实际行格式将在响应`SHOW TABLE STATUS`时的`Row_format`列中报告。`Create_options`列显示了在`CREATE TABLE`语句中指定的行格式，`SHOW CREATE TABLE`也是如此。

    表使用的行格式取决于用于表的存储引擎。

    对于`InnoDB`表：

    +   默认行格式由`innodb_default_row_format`定义，其默认设置为`DYNAMIC`。当未定义`ROW_FORMAT`选项或使用`ROW_FORMAT=DEFAULT`时，将使用默认行格式。

        如果未定义`ROW_FORMAT`选项，或者使用`ROW_FORMAT=DEFAULT`，重建表的操作也会将表的行格式悄悄地更改为由`innodb_default_row_format`定义的默认值。有关更多信息，请参见定义表的行格式。

    +   为了更有效地存储数据类型，特别是`BLOB`类型，使用`DYNAMIC`。有关与`DYNAMIC`行格式相关的要求，请参见 DYNAMIC 行格式。

    +   要为`InnoDB`表启用压缩，请指定`ROW_FORMAT=COMPRESSED`。在创建临时表时，不支持`ROW_FORMAT=COMPRESSED`选项。有关与`COMPRESSED`行格式相关的要求，请参见第 17.9 节，“InnoDB 表和页面压缩”。

    +   旧版本的 MySQL 中使用的行格式仍可通过指定`REDUNDANT`行格式来请求。

    +   当指定非默认的`ROW_FORMAT`子句时，考虑同时启用`innodb_strict_mode`配置选项。

    +   不支持`ROW_FORMAT=FIXED`。如果在禁用`innodb_strict_mode`的情况下指定了`ROW_FORMAT=FIXED`，`InnoDB`会发出警告并假定`ROW_FORMAT=DYNAMIC`。如果在启用`innodb_strict_mode`（默认情况下）的情况下指定了`ROW_FORMAT=FIXED`，`InnoDB`会返回错误。

    +   有关`InnoDB`行格式的更多信息，请参见第 17.10 节，“InnoDB 行格式”。

    对于`MyISAM`表，选项值可以是`FIXED`或`DYNAMIC`，用于静态或变长行格式。**myisampack**将类型设置为`COMPRESSED`。请参见 Section 18.2.3, “MyISAM Table Storage Formats”。

    对于`NDB`表，默认的`ROW_FORMAT`是`DYNAMIC`。

+   `START TRANSACTION`

    这是一个内部使用的表选项。它在 MySQL 8.0.21 中引入，允许将`CREATE TABLE ... SELECT`作为单个原子事务记录在二进制日志中，当使用支持原子 DDL 的存储引擎进行基于行的复制时。在`CREATE TABLE ... START TRANSACTION`之后只允许`BINLOG`、`COMMIT`和`ROLLBACK`语句。有关相关信息，请参见 Section 15.1.1, “Atomic Data Definition Statement Support”。

+   `STATS_AUTO_RECALC`

    指定是否自动重新计算`InnoDB`表的持久性统计信息。值`DEFAULT`会导致表的持久性统计设置由`innodb_stats_auto_recalc`配置选项确定。值`1`会导致当表中的数据变化了 10%时重新计算统计信息。值`0`会阻止对该表进行自动重新计算；在这种设置下，在对表进行重大更改后，发出`ANALYZE TABLE`语句重新计算统计信息。有关持久性统计功能的更多信息，请参见 Section 17.8.10.1, “Configuring Persistent Optimizer Statistics Parameters”。

+   `STATS_PERSISTENT`

    指定是否为`InnoDB`表启用持久性统计信息。值`DEFAULT`导致表的持久性统计信息设置由`innodb_stats_persistent`配置选项确定。值`1`为表启用持久性统计信息，而值`0`关闭此功能。通过`CREATE TABLE`或`ALTER TABLE`语句启用持久性统计信息后，需发出`ANALYZE TABLE`语句来计算统计信息，在将代表性数据加载到表中后。有关持久性统计信息功能的更多信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”。

+   `STATS_SAMPLE_PAGES`

    在估算索引列的基数和其他统计信息时要采样的索引页数，例如由`ANALYZE TABLE`计算的那些。有关更多信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”。

+   `TABLESPACE`

    `TABLESPACE`子句可用于在现有的通用表空间、每个表的文件表空间或系统表空间中创建`InnoDB`表。

    ```sql
    CREATE TABLE *tbl_name* ... TABLESPACE [=] *tablespace_name*
    ```

    您指定的通用表空间必须在使用`TABLESPACE`子句之前存在。有关通用表空间的信息，请参见第 17.6.3.3 节，“通用表空间”。

    `*`tablespace_name`*`是区分大小写的标识符。它可以带引号也可以不带。不允许使用斜杠字符（“/”）。以“innodb_”开头的名称保留供特殊用途。

    要在系统表空间中创建表，请将`innodb_system`指定为表空间名称。

    ```sql
    CREATE TABLE *tbl_name* ... TABLESPACE [=] innodb_system
    ```

    使用`TABLESPACE [=] innodb_system`，您可以将任何未压缩行格式的表放置在系统表空间中，而不受`innodb_file_per_table`设置的影响。例如，您可以使用`TABLESPACE [=] innodb_system`将具有`ROW_FORMAT=DYNAMIC`的表添加到系统表空间中。

    要在每个表的文件表空间中创建表，请将`innodb_file_per_table`指定为表空间名称。

    ```sql
    CREATE TABLE *tbl_name* ... TABLESPACE [=] innodb_file_per_table
    ```

    注意

    如果启用了`innodb_file_per_table`，则无需指定`TABLESPACE=innodb_file_per_table`来创建`InnoDB`每个表的文件表空间。当启用`innodb_file_per_table`时，默认情况下在每个表的文件表空间中创建`InnoDB`表。

    `DATA DIRECTORY`子句允许与`CREATE TABLE ... TABLESPACE=innodb_file_per_table`一起使用，但在与`TABLESPACE`子句结合使用时不受支持。从 MySQL 8.0.21 开始，`DATA DIRECTORY`子句中指定的目录必须为`InnoDB`所知。更多信息，请参阅使用 DATA DIRECTORY 子句。

    注意

    从 MySQL 8.0.13 开始，使用`TABLESPACE = innodb_file_per_table`和`TABLESPACE = innodb_temporary`子句与`CREATE TEMPORARY TABLE`已不再受支持；预计将在将来的 MySQL 版本中删除。

    `STORAGE`表选项仅用于`NDB`表。`STORAGE`确定所使用的存储类型，可以是`DISK`或`MEMORY`中的任一种。

    `TABLESPACE ... STORAGE DISK`将表分配给 NDB Cluster Disk Data 表空间。除非在`TABLESPACE` *`tablespace_name`*之前，否则不能在`CREATE TABLE`中使用`STORAGE DISK`。

    对于`STORAGE MEMORY`，表空间名称是可选的，因此，您可以使用`TABLESPACE *`tablespace_name`* STORAGE MEMORY`或简单地使用`STORAGE MEMORY`来明确指定表位于内存中。

    有关更多信息，请参阅第 25.6.11 节，“NDB Cluster Disk Data Tables”。

+   `UNION`

    用于将一组相同的`MyISAM`表作为一个表访问。这仅适用于`MERGE`表。请参阅第 18.7 节，“MERGE 存储引擎”。

    您必须对映射到`MERGE`表的表具有`SELECT`、`UPDATE`和`DELETE`权限。

    注意

    以前，所有使用的表都必须与`MERGE`表本身位于同一个数据库中。这个限制不再适用。

#### 表分区

*`partition_options`*可用于控制使用`CREATE TABLE`创建的表的分区。

此部分开头显示的*`partition_options`*语法中并非所有选项适用于所有分区类型。有关每种类型的特定信息，请参阅以下各个类型的列表，并参阅第二十六章，*分区*，以获取有关 MySQL 中分区工作原理和用途的更完整信息，以及有关表创建和其他与 MySQL 分区相关的语句的其他示例。

分区可以被修改、合并、添加到表中，也可以从表中删除。有关完成这些任务的 MySQL 语句的基本信息，请参见 Section 15.1.9, “ALTER TABLE Statement”。有关更详细的描述和示例，请参见 Section 26.3, “Partition Management”。

+   `PARTITION BY`

    如果使用了 *`partition_options`* 子句，它以 `PARTITION BY` 开头。这个子句包含用于确定分区的函数；该函数返回一个从 1 到 *`num`* 的整数值，其中 *`num`* 是分区的数量。（一个表中可以包含的用户定义分区的最大数量是 1024；本节稍后讨论的子分区数量也包括在这个最大值中。）

    注意

    在 `PARTITION BY` 子句中使用的表达式 (*`expr`*) 不能引用不在被创建的表中的任何列；这样的引用是明确不允许的，并会导致语句失败并出现错误。 (Bug #29444)

+   `HASH(*`expr`*)`

    对一个或多个列进行哈希运算，创建一个用于定位和查找行的键。*`expr`* 是使用一个或多个表列的表达式。这可以是任何有效的 MySQL 表达式（包括 MySQL 函数），产生一个单个整数值。例如，以下是使用 `PARTITION BY HASH` 的两个有效的 `CREATE TABLE` 语句：

    ```sql
    CREATE TABLE t1 (col1 INT, col2 CHAR(5))
        PARTITION BY HASH(col1);

    CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATETIME)
        PARTITION BY HASH ( YEAR(col3) );
    ```

    不能在 `PARTITION BY HASH` 中使用 `VALUES LESS THAN` 或 `VALUES IN` 子句。

    `PARTITION BY HASH` 使用 *`expr`* 除以分区数后的余数（即模数）。有关示例和额外信息，请参见 Section 26.2.4, “HASH Partitioning”。

    `LINEAR` 关键字涉及一个略有不同的算法。在这种情况下，存储行的分区号是通过一个或多个逻辑 `AND` 操作的结果计算得出的。有关线性哈希的讨论和示例，请参见 Section 26.2.4.1, “LINEAR HASH Partitioning”。

+   `KEY(*`column_list`*)`

    这类似于 `HASH`，不同之处在于 MySQL 提供了哈希函数，以保证数据均匀分布。*`column_list`* 参数只是一个包含 1 个或多个表列（最多：16）的列表。这个示例展示了一个简单的按键分区的表，有 4 个分区：

    ```sql
    CREATE TABLE tk (col1 INT, col2 CHAR(5), col3 DATE)
        PARTITION BY KEY(col3)
        PARTITIONS 4;
    ```

    对于按键分区的表，您可以使用`LINEAR`关键字来采用线性分区。这与按`HASH`分区的表具有相同的效果。也就是说，分区号是使用`&`运算符而不是取模来确定的（详见第 26.2.4.1 节，“线性 HASH 分区”和第 26.2.5 节，“KEY 分区”）。此示例使用按键进行线性分区以在 5 个分区之间分发数据：

    ```sql
    CREATE TABLE tk (col1 INT, col2 CHAR(5), col3 DATE)
        PARTITION BY LINEAR KEY(col3)
        PARTITIONS 5;
    ```

    `ALGORITHM={1 | 2}`选项支持`[SUB]PARTITION BY [LINEAR] KEY`。`ALGORITHM=1`使服务器使用与 MySQL 5.1 相同的键哈希函数；`ALGORITHM=2`表示服务器使用 MySQL 5.5 及更高版本中默认实现和使用的键哈希函数。 （使用 MySQL 5.5 及更高版本中实施的键哈希函数创建的分区表不能被 MySQL 5.1 服务器使用。）不指定该选项与使用`ALGORITHM=2`具有相同的效果。该选项主要用于在 MySQL 5.1 和后续 MySQL 版本之间升级或降级`[LINEAR] KEY`分区表时，或者在 MySQL 5.5 或更高版本服务器上创建按`KEY`或`LINEAR KEY`分区的表，该表可以在 MySQL 5.1 服务器上使用。有关更多信息，请参见第 15.1.9.1 节，“ALTER TABLE 分区操作”。

    **mysqldump**将此选项写入版本化注释中。

    在必要时，`ALGORITHM=1`将显示在使用版本化注释的`SHOW CREATE TABLE`输出中，方式与**mysqldump**相同。即使在创建原始表时指定了此选项，`SHOW CREATE TABLE`输出中也总是省略`ALGORITHM=2`。

    您不能在`PARTITION BY KEY`中使用`VALUES LESS THAN`或`VALUES IN`子句。

+   `RANGE(*`表达式`*)`

    在这种情况下，*表达式*使用一组`VALUES LESS THAN`运算符显示一系列值的范围。在使用范围分区时，您必须至少定义一个使用`VALUES LESS THAN`的分区。您不能在范围分区中使用`VALUES IN`。

    注意

    对于按`RANGE`分区的表，`VALUES LESS THAN`必须与整数文字值或评估为单个整数值的表达式一起使用。在 MySQL 8.0 中，您可以在使用`PARTITION BY RANGE COLUMNS`定义的表中克服这一限制，如本节后面所述。

    假设您有一个希望根据以下方案对包含年份值的列进行分区的表。

    | 分区号： | 年份范围： |
    | --- | --- |
    | 0 | 1990 年及之前 |
    | 1 | 1991 至 1994 年 |
    | 2 | 1995 to 1998 |
    | 3 | 1999 to 2002 |
    | 4 | 2003 to 2005 |
    | 5 | 2006 and later |

    实现这种分区方案的表可以通过以下`CREATE TABLE` 语句实现：

    ```sql
    CREATE TABLE t1 (
        year_col  INT,
        some_data INT
    )
    PARTITION BY RANGE (year_col) (
        PARTITION p0 VALUES LESS THAN (1991),
        PARTITION p1 VALUES LESS THAN (1995),
        PARTITION p2 VALUES LESS THAN (1999),
        PARTITION p3 VALUES LESS THAN (2002),
        PARTITION p4 VALUES LESS THAN (2006),
        PARTITION p5 VALUES LESS THAN MAXVALUE
    );
    ```

    `PARTITION ... VALUES LESS THAN ...` 语句按顺序工作。`VALUES LESS THAN MAXVALUE` 用于指定大于其他指定最大值的“剩余”值。

    `VALUES LESS THAN`子句按顺序工作，类似于`switch ... case`块的`case`部分（在许多编程语言中如 C、Java 和 PHP 中找到）。也就是说，这些子句必须按照每个连续`VALUES LESS THAN`中指定的上限大于前一个的方式排列，其中引用`MAXVALUE`的子句在列表中最后出现。

+   `RANGE COLUMNS(*`column_list`*)`

    这种`RANGE`变体便于对使用多列范围条件的查询进行分区修剪（即，具有诸如`WHERE a = 1 AND b < 10`或`WHERE a = 1 AND b = 10 AND c < 10`条件的查询）。它允许您通过在`COLUMNS`子句中列出的列列表和在每个`PARTITION ... VALUES LESS THAN (*`value_list`*)`分区定义子句中设置的列值集来指定多列中的值范围。（在最简单的情况下，此集合由单个列组成。）在`column_list`和`value_list`中引用的列的最大数量为 16。

    在`COLUMNS`子句中使用的`column_list`可能只包含列名；列表中的每个列必须是以下 MySQL 数据类型之一：整数类型；字符串类型；时间或日期列类型。不允许使用`BLOB`、`TEXT`、`SET`、`ENUM`、`BIT`或空间数据类型的列；也不允许使用浮点数类型的列。您也不能在`COLUMNS`子句中使用函数或算术表达式。

    在分区定义中使用的`VALUES LESS THAN`子句必须为`COLUMNS()`子句中出现的每个列指定一个文字值；也就是说，用于每个`VALUES LESS THAN`子句的值列表必须包含与`COLUMNS`子句中列出的列数相同的值。尝试在`VALUES LESS THAN`子句中使用比`COLUMNS`子句中列出的列数更多或更少的值会导致出现错误 Inconsistency in usage of column lists for partitioning...。您不能对出现在`VALUES LESS THAN`中的任何值使用`NULL`。可以多次使用`MAXVALUE`来表示给定列，如下例所示：

    ```sql
    CREATE TABLE rc (
        a INT NOT NULL,
        b INT NOT NULL
    )
    PARTITION BY RANGE COLUMNS(a,b) (
        PARTITION p0 VALUES LESS THAN (10,5),
        PARTITION p1 VALUES LESS THAN (20,10),
        PARTITION p2 VALUES LESS THAN (50,MAXVALUE),
        PARTITION p3 VALUES LESS THAN (65,MAXVALUE),
        PARTITION p4 VALUES LESS THAN (MAXVALUE,MAXVALUE)
    );
    ```

    在`VALUES LESS THAN`值列表中使用的每个值必须与相应列的类型完全匹配；不会进行任何转换。例如，您不能为与使用整数类型的列匹配的值使用字符串`'1'`（您必须使用数字`1`），也不能为与使用字符串类型的列匹配的值使用数字`1`（在这种情况下，您必须使用带引号的字符串：`'1'`）。

    有关更多信息，请参见第 26.2.1 节，“RANGE 分区”和第 26.4 节，“分区修剪”。

+   `LIST(*`expr`*)`

    当基于具有受限可能值集的表列分配分区时，这是很有用的，例如州或国家代码。在这种情况下，所有属于某个州或国家的行可以分配到一个分区，或者可以为某个州或国家集保留一个分区。它类似于`RANGE`，只是每个分区只能使用`VALUES IN`来指定可允许的值。

    `VALUES IN`用于匹配的值列表。例如，您可以创建以下分区方案：

    ```sql
    CREATE TABLE client_firms (
        id   INT,
        name VARCHAR(35)
    )
    PARTITION BY LIST (id) (
        PARTITION r0 VALUES IN (1, 5, 9, 13, 17, 21),
        PARTITION r1 VALUES IN (2, 6, 10, 14, 18, 22),
        PARTITION r2 VALUES IN (3, 7, 11, 15, 19, 23),
        PARTITION r3 VALUES IN (4, 8, 12, 16, 20, 24)
    );
    ```

    在使用列表分区时，您必须至少定义一个使用`VALUES IN`的分区。您不能在`PARTITION BY LIST`中使用`VALUES LESS THAN`。

    注意

    对于通过`LIST`分区的表，与`VALUES IN`一起使用的值列表必须仅包含整数值。在 MySQL 8.0 中，您可以通过后面在本节中描述的`LIST COLUMNS`进行分区来克服这种限制。

+   `LIST COLUMNS(*`column_list`*)`

    这种`LIST`的变体为使用多列比较条件的查询提供了分区修剪的便利（即，具有诸如`WHERE a = 5 AND b = 5`或`WHERE a = 1 AND b = 10 AND c = 5`等条件）。它允许您通过在`COLUMNS`子句中使用列列表和在每个`PARTITION ... VALUES IN (*`value_list`*)`分区定义子句中使用一组列值来指定多列的值。

    用于`LIST COLUMNS(*`column_list`*)`中的列列表和`VALUES IN(*`value_list`*)`中使用的值列表的数据类型规则与用于`RANGE COLUMNS(*`column_list`*)`中的列列表和`VALUES LESS THAN(*`value_list`*)`中使用的值列表的规则相同，只是在`VALUES IN`子句中，不允许使用`MAXVALUE`，您可以使用`NULL`。

    在使用`PARTITION BY LIST COLUMNS`时，与在使用`PARTITION BY LIST`时使用`VALUES IN`的值列表之间有一个重要区别。在与`PARTITION BY LIST COLUMNS`一起使用时，`VALUES IN`子句中的每个元素必须是一组列值；每组中的值数量必须与`COLUMNS`子句中使用的列数相同，并且这些值的数据类型必须与列的数据类型匹配（并且以相同顺序出现）。在最简单的情况下，该集合由单个列组成。在*`column_list`*和组成*`value_list`*的元素中可以使用的最大列数为 16。

    以下`CREATE TABLE`语句定义的表提供了使用`LIST COLUMNS`分区的示例：

    ```sql
    CREATE TABLE lc (
        a INT NULL,
        b INT NULL
    )
    PARTITION BY LIST COLUMNS(a,b) (
        PARTITION p0 VALUES IN( (0,0), (NULL,NULL) ),
        PARTITION p1 VALUES IN( (0,1), (0,2), (0,3), (1,1), (1,2) ),
        PARTITION p2 VALUES IN( (1,0), (2,0), (2,1), (3,0), (3,1) ),
        PARTITION p3 VALUES IN( (1,3), (2,2), (2,3), (3,2), (3,3) )
    );
    ```

+   `PARTITIONS *`num`*`

    可以选择使用`PARTITIONS *`num`*`子句指定分区的数量，其中*`num`*是分区的数量。如果同时使用此子句和任何使用`PARTITION`子句声明的分区，则*`num`*必须等于使用`PARTITION`子句声明的所有分区的总数。

    注意

    在创建按`RANGE`或`LIST`分区的表时，无论是否使用`PARTITIONS`子句，仍必须在表定义中至少包含一个`PARTITION VALUES`子句（见下文）。

+   `SUBPARTITION BY`

    分区可以选择地分成多个子分区。这可以通过使用可选的`SUBPARTITION BY`子句来指示。子分区可以通过`HASH`或`KEY`进行。其中任何一个都可以是`LINEAR`。这与先前描述的等效分区类型的工作方式相同。（不可能通过`LIST`或`RANGE`进行子分区。）

    可以使用`SUBPARTITIONS`关键字后跟一个整数值来指示子分区的数量。

+   对`PARTITIONS`或`SUBPARTITIONS`子句中使用的值进行严格检查，并且此值必须遵守以下规则：

    +   值必须是正的、非零整数。

    +   不允许前导零。

    +   值必须是整数文字，并且不能是表达式。例如，`PARTITIONS 0.2E+01`是不允许的，即使`0.2E+01`评估为`2`。（Bug #15890）

+   `*`partition_definition`*`

    可以使用*`partition_definition`*子句分别定义每个分区。构成此子句的各个部分如下：

    +   `PARTITION *`partition_name`*`

        为分区指定一个逻辑名称。

    +   `VALUES`

        对于范围分区，每个分区必须包括一个`VALUES LESS THAN`子句；对于列表分区，您必须为每个分区指定一个`VALUES IN`子句。这用于确定哪些行将存储在此分区中。有关分区类型的讨论，请参见第二十六章，*分区*中的语法示例。

    +   `[STORAGE] ENGINE`

        MySQL 接受 `PARTITION` 和 `SUBPARTITION` 的 `[STORAGE] ENGINE` 选项。目前，此选项可用的唯一方式是将所有分区或所有子分区设置为相同的存储引擎，尝试为同一表中的分区或子分区设置不同的存储引擎会引发错误 ERROR 1469 (HY000): The mix of handlers in the partitions is not permitted in this version of MySQL。

    +   `COMMENT`

        可以使用可选的 `COMMENT` 子句来指定描述分区的字符串。示例：

        ```sql
        COMMENT = 'Data for the years previous to 1999'
        ```

        分区注释的最大长度为 1024 个字符。

    +   `DATA DIRECTORY` 和 `INDEX DIRECTORY`

        `DATA DIRECTORY` 和 `INDEX DIRECTORY` 可用于指示存储此分区数据和索引的目录。`*`data_dir`*` 和 `*`index_dir`*` 必须是绝对系统路径名。

        截至 MySQL 8.0.21 版本，`DATA DIRECTORY` 子句中指定的目录必须为 `InnoDB` 所知。更多信息，请参阅 使用 DATA DIRECTORY 子句。

        您必须具有 `FILE` 权限才能使用 `DATA DIRECTORY` 或 `INDEX DIRECTORY` 分区选项。

        示例：

        ```sql
        CREATE TABLE th (id INT, name VARCHAR(30), adate DATE)
        PARTITION BY LIST(YEAR(adate))
        (
          PARTITION p1999 VALUES IN (1995, 1999, 2003)
            DATA DIRECTORY = '/var/appdata/95/data'
            INDEX DIRECTORY = '/var/appdata/95/idx',
          PARTITION p2000 VALUES IN (1996, 2000, 2004)
            DATA DIRECTORY = '/var/appdata/96/data'
            INDEX DIRECTORY = '/var/appdata/96/idx',
          PARTITION p2001 VALUES IN (1997, 2001, 2005)
            DATA DIRECTORY = '/var/appdata/97/data'
            INDEX DIRECTORY = '/var/appdata/97/idx',
          PARTITION p2002 VALUES IN (1998, 2002, 2006)
            DATA DIRECTORY = '/var/appdata/98/data'
            INDEX DIRECTORY = '/var/appdata/98/idx'
        );
        ```

        `DATA DIRECTORY` 和 `INDEX DIRECTORY` 的行为与用于 `MyISAM` 表的 `CREATE TABLE` 语句的 *`table_option`* 子句中的行为相同。

        每个分区可以指定一个数据目录和一个索引目录。如果未指定，数据和索引默认存储在表的数据库目录中。

        如果 `NO_DIR_IN_CREATE` 生效，则在创建分区表时将忽略 `DATA DIRECTORY` 和 `INDEX DIRECTORY` 选项。

    +   `MAX_ROWS` 和 `MIN_ROWS`

        可以分别指定要存储在分区中的最大和最小行数。*`max_number_of_rows`* 和 *`min_number_of_rows`* 的值必须是正整数。与具有相同名称的表级选项一样，这些值仅作为服务器的“建议”，而不是硬限制。

    +   `TABLESPACE`

        可以通过指定 `TABLESPACE `innodb_file_per_table`` 为分区指定一个 `InnoDB` 每表表空间。所有分区必须属于相同的存储引擎。

        不支持将 `InnoDB` 表分区放置在共享的 `InnoDB` 表空间中。共享表空间包括 `InnoDB` 系统表空间和通用表空间。

+   `*`subpartition_definition`*`

    分区定义可以选择性地包含一个或多个 *`subpartition_definition`* 子句。每个子句至少包含 `SUBPARTITION *`name`*`，其中 *`name`* 是子分区的标识符。除了将 `PARTITION` 关键字替换为 `SUBPARTITION` 外，子分区定义的语法与分区定义完全相同。

    子分区必须通过`HASH`或`KEY`完成，并且只能在`RANGE`或`LIST`分区上完成。参见第 26.2.6 节，“子分区”。

**通过生成列进行分区**

允许通过生成的列进行分区。例如：

```sql
CREATE TABLE t1 (
  s1 INT,
  s2 INT AS (EXP(s1)) STORED
)
PARTITION BY LIST (s2) (
  PARTITION p1 VALUES IN (1)
);
```

分区将生成的列视为常规列，这使得可以解决对于分区不允许的函数的限制（参见第 26.6.3 节，“与函数相关的分区限制”）。前面的示例演示了这种技术：`EXP()`不能直接在`PARTITION BY`子句中使用，但可以使用使用`EXP()`定义的生成列。
