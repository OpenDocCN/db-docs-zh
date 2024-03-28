# 17.10 InnoDB 行格式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-row-format.html)

表的行格式决定了其行是如何物理存储的，进而影响查询和 DML 操作的性能。当更多行适合存储在单个磁盘页中时，查询和索引查找可以更快地工作，缓冲池中需要更少的缓存内存，并且写出更新值所需的 I/O 也更少。

每个表中的数据被分成页。构成每个表的页被排列在一种称为 B 树索引的树数据结构中。表数据和辅助索引都使用这种类型的结构。代表整个表的 B 树索引被称为聚簇索引，根据主键列组织。聚簇索引数据结构的节点包含行中所有列的值。辅助索引结构的节点包含索引列和主键列的值。

变长列是存储在 B 树索引节点中的列值的一个例外。太长以至于无法适应 B 树页的变长列存储在称为溢出页的单独分配的磁盘页上。这样的列被称为离页列。离页列的值存储在溢出页的单链列表中，每个这样的列都有自己的一个或多个溢出页列表。根据列长度，变长列值的全部或前缀存储在 B 树中，以避免浪费存储空间并且需要读取一个单独的页。

`InnoDB` 存储引擎支持四种行格式：`冗余`，`紧凑`，`动态` 和 `压缩`。

**表 17.15 InnoDB 行格式概述**

| 行格式 | 紧凑存储特性 | 增强的可变长度列存储 | 大索引键前缀支持 | 压缩支持 | 支持的表空间类型 |
| --- | --- | --- | --- | --- | --- |
| `冗余` | 否 | 否 | 否 | 否 | 系统，每表一个文件，通用 |
| `紧凑` | 是 | 否 | 否 | 否 | 系统，每表一个文件，通用 |
| `动态` | 是 | 是 | 是 | 否 | 系统，每表一个文件，通用 |
| `压缩` | 是 | 是 | 是 | 是 | 每表一个文件，通用 |

接下来的主题描述了行格式存储特性以及如何定义和确定表的行格式。

+   冗余行格式

+   紧凑行格式

+   动态行格式

+   压缩行格式

+   定义表的行格式

+   确定表的行格式

### REDUNDANT 行格式

`REDUNDANT`格式与 MySQL 的旧版本兼容。

使用`REDUNDANT`行格式的表将变长列值（`VARCHAR`、`VARBINARY`和`BLOB`和`TEXT`类型）的前 768 字节存储在 B 树节点内的索引记录中，其余部分存储在溢出页上。大于或等于 768 字节的固定长度列被编码为变长列，可以存储在页外。例如，如果字符集的最大字节长度大于 3，`CHAR(255)`列可能超过 768 字节，就像`utf8mb4`一样。

如果列的值为 768 字节或更少，则不使用溢出页，可能会节省一些 I/O，因为该值完全存储在 B 树节点中。这对于相对较短的`BLOB`列值效果很好，但可能导致 B 树节点填满数据而不是键值，降低其效率。具有许多`BLOB`列的表可能导致 B 树节点变得过满，并包含太少行，使整个索引比行较短或列值存储在页外时效率低。

#### REDUNDANT 行格式存储特性

`REDUNDANT`行格式具有以下存储特性：

+   每个索引记录包含一个 6 字节的头部。头部用于链接连续的记录，并用于行级锁定。

+   聚簇索引中的记录包含所有用户定义的列字段。此外，还有一个 6 字节的事务 ID 字段和一个 7 字节的回滚指针字段。

+   如果表没有定义主键，则每个聚簇索引记录还包含一个 6 字节的行 ID 字段。

+   每个二级索引记录包含所有未在二级索引中的聚簇索引键定义的主键列。

+   记录包含对记录的每个字段的指针。如果记录中字段的总长度小于 128 字节，则指针为一个字节；否则为两个字节。指针数组称为记录目录。指针指向的区域是记录的数据部分。

+   在内部，像`CHAR(10)`这样的固定长度字符列以固定长度格式存储。在`VARCHAR`列中不会截断尾随空格。

+   大于或等于 768 字节的固定长度列被编码为变长列，可以存储在页外。例如，如果字符集的最大字节长度大于 3，`CHAR(255)`列可能超过 768 字节，就像`utf8mb4`一样。

+   SQL 中的`NULL`值在记录目录中保留一到两个字节。如果存储在可变长度列中，则 SQL 中的`NULL`值在记录的数据部分中不保留任何字节。对于固定长度列，列的固定长度在记录的数据部分中保留。为`NULL`值保留固定空间允许列在位更新为非`NULL`值而不会导致索引页碎片化。

### 紧凑行格式

与`REDUNDANT`行格式相比，`COMPACT`行格式将行存储空间减少约 20%，但会增加某些操作的 CPU 使用率。如果您的工作负载是受缓存命中率和磁盘速度限制的典型工作负载，`COMPACT`格式可能更快。如果工作负载受 CPU 速度限制，则紧凑格式可能较慢。

使用`COMPACT`行格式的表将变长列值（`VARCHAR`、`VARBINARY`和`BLOB`和`TEXT`类型）的前 768 字节存储在 B 树节点内的索引记录中，其余部分存储在溢出页上。大于或等于 768 字节的固定长度列被编码为可变长度列，可以存储在页外。例如，如果字符集的最大字节长度大于 3，那么`CHAR(255)`列可能超过 768 字节，就像`utf8mb4`一样。

如果列的值为 768 字节或更少，则不使用溢出页，可能会节省一些 I/O，因为该值完全存储在 B 树节点中。这对于相对较短的`BLOB`列值效果很好，但可能导致 B 树节点填满数据而不是键值，降低其效率。具有许多`BLOB`列的表可能导致 B 树节点变得过满，并包含太少行，使整个索引比行较短或列值存储在页外时效率低。

#### 紧凑行格式存储特性

`COMPACT`行格式具有以下存储特性：

+   每个索引记录包含一个 5 字节的头部，可能在可变长度头部之前。头部用于链接连续记录，并用于行级锁定。

+   记录头部的可变长度部分包含一个位向量，用于指示`NULL`列。如果索引中可以为`NULL`的列数为*`N`*，则位向量占用`CEILING(*`N`*/8)`字节。（例如，如果可以为`NULL`的列数为 9 到 16 列，位向量将使用两个字节。）`NULL`列不占用除此向量中的位之外的空间。头部的可变长度部分还包含可变长度列的长度。每个长度占用一到两个字节，取决于列的最大长度。如果索引中的所有列都是`NOT NULL`且具有固定长度，则记录头部没有可变长度部分。

+   对于每个非`NULL`可变长度字段，记录头部包含列的长度，使用一到两个字节。只有当列的一部分存储在溢出页中或最大长度超过 255 字节且实际长度超过 127 字节时，才需要两个字节。对于外部存储的列，2 字节长度表示内部存储部分的长度加上指向外部存储部分的 20 字节指针。内部部分为 768 字节，因此长度为 768+20。20 字节指针存储列的真实长度。

+   记录头部后跟非`NULL`列的数据内容。

+   聚集索引中的记录包含所有用户定义的列。此外，还有一个 6 字节的事务 ID 字段和一个 7 字节的回滚指针字段。

+   如果表没有定义主键，则每个聚集索引记录还包含一个 6 字节的行 ID 字段。

+   每个二级索引记录包含所有聚集索引键中定义的主键列，这些列不在二级索引中。如果任何主键列是可变长度的，则每个二级索引的记录头部都有一个可变长度部分来记录它们的长度，即使二级索引是在固定长度列上定义的。

+   在内部，对于非可变长度字符集，如`CHAR(10)`这样的固定长度字符列以固定长度格式存储。

    `VARCHAR`列不会截断尾随空格。

+   在内部，对于可变长度字符集，如`utf8mb3`和`utf8mb4`，`InnoDB`尝试将`CHAR(*`N`*)`存储为*`N`*字节，通过修剪尾随空格。如果`CHAR(*`N`*)`列值的字节长度超过*`N`*字节，则尾随空格将被修剪至列值字节长度的最大值。`CHAR(*`N`*)`列的最大长度为最大字符字节长度×*`N`*。

    为`CHAR(*`N`*)`保留了最小的*`N`*字节。在许多情况下，保留最小空间*`N`*可以使列更新就地完成，而不会导致索引页碎片化。相比之下，使用`REDUNDANT`行格式时，`CHAR(*`N`*)`列占用最大字符字节长度×*`N`*。

    长度大于或等于 768 字节的固定长度列被编码为变长字段，可以存储在页外。例如，如果字符集的最大字节长度大于 3，那么`CHAR(255)`列可以超过 768 字节，就像`utf8mb4`一样。

### 动态行格式

`DYNAMIC`行格式提供了与`COMPACT`行格式相同的存储特性，但增加了对长变长列的增强存储能力，并支持大型索引键前缀。

当使用`ROW_FORMAT=DYNAMIC`创建表时，`InnoDB`可以将长变长列值（如`VARCHAR`、`VARBINARY`和`BLOB`以及`TEXT`类型）完全存储在页外，聚簇索引记录仅包含指向溢出页的 20 字节指针。长度大于或等于 768 字节的固定长度字段被编码为变长字段。例如，如果字符集的最大字节长度大于 3，那么`CHAR(255)`列可以超过 768 字节，就像`utf8mb4`一样。

列是否存储在页外取决于页大小和行的总大小。当行过长时，会选择最长的列进行页外存储，直到聚簇索引记录适合于 B 树页。长度小于或等于 40 字节的`TEXT`和`BLOB`列存储在行内。

`DYNAMIC`行格式保持了在索引节点中存储整个行的效率（与`COMPACT`和`REDUNDANT`格式一样），但`DYNAMIC`行格式避免了将 B 树节点填充大量长列数据字节的问题。`DYNAMIC`行格式基于这样一个思想，即如果长数据值的一部分存储在页外，通常最有效的方式是将整个值存储在页外。使用`DYNAMIC`格式，较短的列可能会保留在 B 树节点中，最大程度地减少给定行所需的溢出页数量。

`DYNAMIC`行格式支持索引键前缀长达 3072 字节。

使用 `DYNAMIC` 行格式的表可以存储在系统表空间、每表一个表空间和通用表空间中。要将 `DYNAMIC` 表存储在系统表空间中，要么禁用`innodb_file_per_table`并使用常规的 `CREATE TABLE` 或 `ALTER TABLE` 语句，要么在 `CREATE TABLE` 或 `ALTER TABLE` 中使用 `TABLESPACE [=] innodb_system` 表选项。`innodb_file_per_table` 变量不适用于通用表空间，也不适用于使用 `TABLESPACE [=] innodb_system` 表选项将 `DYNAMIC` 表存储在系统表空间中。

#### `DYNAMIC` 行格式存储特性

`DYNAMIC` 行格式是 `COMPACT` 行格式的一种变体。有关存储特性，请参阅 COMPACT 行格式存储特性。

### 压缩行格式

`COMPRESSED` 行格式提供了与 `DYNAMIC` 行格式相同的存储特性和功能，但增加了对表和索引数据压缩的支持。

`COMPRESSED` 行格式使用与 `DYNAMIC` 行格式相似的内部细节进行页外存储，同时由于表和索引数据被压缩并使用较小的页大小，还有额外的存储和性能考虑。对于 `COMPRESSED` 行格式，`KEY_BLOCK_SIZE` 选项控制存储在聚簇索引中的列数据量，以及放置在溢出页上的数据量。有关 `COMPRESSED` 行格式的更多信息，请参阅第 17.9 节，“InnoDB 表和页压缩”。

`COMPRESSED` 行格式支持索引键前缀长达 3072 字节。

使用 `COMPRESSED` 行格式的表可以在每表一个表空间或通用表空间中创建。系统表空间不支持 `COMPRESSED` 行格式。要将 `COMPRESSED` 表存储在每表一个表空间中，必须启用`innodb_file_per_table`变量。`innodb_file_per_table`变量不适用于通用表空间。通用表空间支持所有行格式，但由于不同的物理页大小，压缩和非压缩表不能共存于同一通用表空间中。有关更多信息，请参阅第 17.6.3.3 节，“通用表空间”。

#### 压缩行格式存储特性

`COMPRESSED` 行格式是 `COMPACT` 行格式的一种变体。有关存储特性，请参阅 COMPACT 行格式存储特性。

### 定义表的行格式

`InnoDB`表的默认行格式由`innodb_default_row_format`变量定义，默认值为`DYNAMIC`。当未明确定义`ROW_FORMAT`表选项或指定`ROW_FORMAT=DEFAULT`时，将使用默认行格式。

表格的行格式可以在`CREATE TABLE`或`ALTER TABLE`语句中通过`ROW_FORMAT`表选项明确定义。例如：

```sql
CREATE TABLE t1 (c1 INT) ROW_FORMAT=DYNAMIC;
```

明确定义的`ROW_FORMAT`设置会覆盖默认行格式。指定`ROW_FORMAT=DEFAULT`等同于使用隐式默认值。

`innodb_default_row_format`变量可以动态设置：

```sql
mysql> SET GLOBAL innodb_default_row_format=DYNAMIC;
```

有效的`innodb_default_row_format`选项包括`DYNAMIC`、`COMPACT`和`REDUNDANT`。不支持在系统表空间中使用的`COMPRESSED`行格式不能定义为默认值。只能在`CREATE TABLE`或`ALTER TABLE`语句中明确定义。尝试将`innodb_default_row_format`变量设置为`COMPRESSED`会返回错误：

```sql
mysql> SET GLOBAL innodb_default_row_format=COMPRESSED;
ERROR 1231 (42000): Variable 'innodb_default_row_format'
can't be set to the value of 'COMPRESSED'
```

新创建的表在未明确指定`ROW_FORMAT`选项或使用`ROW_FORMAT=DEFAULT`时，将使用由`innodb_default_row_format`变量定义的行格式。例如，以下`CREATE TABLE`语句使用由`innodb_default_row_format`变量定义的行格式。

```sql
CREATE TABLE t1 (c1 INT);
```

```sql
CREATE TABLE t2 (c1 INT) ROW_FORMAT=DEFAULT;
```

当未明确指定`ROW_FORMAT`选项，或使用`ROW_FORMAT=DEFAULT`时，重建表的操作会静默更改表的行格式为由`innodb_default_row_format`变量定义的格式。

表重建操作包括使用`ALGORITHM=COPY`或`ALGORITHM=INPLACE`的`ALTER TABLE`操作，其中需要重建表。有关更多信息，请参见第 17.12.1 节，“在线 DDL 操作”。`OPTIMIZE TABLE`也是一种表重建操作。

以下示例演示了一种表重建操作，静默更改了未明确定义行格式的表的行格式。

```sql
mysql> SELECT @@innodb_default_row_format;
+-----------------------------+
| @@innodb_default_row_format |
+-----------------------------+
| dynamic                     |
+-----------------------------+

mysql> CREATE TABLE t1 (c1 INT);

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME LIKE 'test/t1' \G
*************************** 1\. row ***************************
     TABLE_ID: 54
         NAME: test/t1
         FLAG: 33
       N_COLS: 4
        SPACE: 35
   ROW_FORMAT: Dynamic
ZIP_PAGE_SIZE: 0
   SPACE_TYPE: Single 
mysql> SET GLOBAL innodb_default_row_format=COMPACT;

mysql> ALTER TABLE t1 ADD COLUMN (c2 INT);

mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME LIKE 'test/t1' \G
*************************** 1\. row ***************************
     TABLE_ID: 55
         NAME: test/t1
         FLAG: 1
       N_COLS: 5
        SPACE: 36
   ROW_FORMAT: Compact
ZIP_PAGE_SIZE: 0
   SPACE_TYPE: Single
```

在将现有表格的行格式从`REDUNDANT`或`COMPACT`更改为`DYNAMIC`之前，请考虑以下潜在问题。

+   `REDUNDANT`和`COMPACT`行格式支持最大索引键前缀长度为 767 字节，而`DYNAMIC`和`COMPRESSED`行格式支持索引键前缀长度为 3072 字节。在复制环境中，如果源上的`innodb_default_row_format`变量设置为`DYNAMIC`，并在副本上设置为`COMPACT`，则以下 DDL 语句，在源上成功但在副本上失败，因为未明确定义行格式：

    ```sql
    CREATE TABLE t1 (c1 INT PRIMARY KEY, c2 VARCHAR(5000), KEY i1(c2(3070)));
    ```

    有关相关信息，请参见第 17.22 节，“InnoDB 限制”。

+   导入未明确定义行格式的表，如果源服务器上的`innodb_default_row_format`设置与目标服务器上的设置不同，则会导致模式不匹配错误。有关更多信息，请参见第 17.6.1.3 节，“导入 InnoDB 表”。

### 确定表的行格式

要确定表的行格式，请使用`SHOW TABLE STATUS`：

```sql
mysql> SHOW TABLE STATUS IN test1\G
*************************** 1\. row ***************************
           Name: t1
         Engine: InnoDB
        Version: 10
     Row_format: Dynamic
           Rows: 0
 Avg_row_length: 0
    Data_length: 16384
Max_data_length: 0
   Index_length: 16384
      Data_free: 0
 Auto_increment: 1
    Create_time: 2016-09-14 16:29:38
    Update_time: NULL
     Check_time: NULL
      Collation: utf8mb4_0900_ai_ci
       Checksum: NULL
 Create_options:
        Comment:
```

或者，查询信息模式`INNODB_TABLES`表：

```sql
mysql> SELECT NAME, ROW_FORMAT FROM INFORMATION_SCHEMA.INNODB_TABLES WHERE NAME='test1/t1';
+----------+------------+
| NAME     | ROW_FORMAT |
+----------+------------+
| test1/t1 | Dynamic    |
+----------+------------+
```
