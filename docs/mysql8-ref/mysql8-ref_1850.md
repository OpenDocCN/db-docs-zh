# 26.2.5 键分区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-key.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-key.html)

按键分区类似于按哈希分区，不同之处在于哈希分区使用用户定义的表达式，而键分区的哈希函数由 MySQL 服务器提供。NDB Cluster 使用`MD5()`来实现这一目的；对于使用其他存储引擎的表，服务器使用自己的内部哈希函数。

`CREATE TABLE ... PARTITION BY KEY`的语法规则与创建哈希分区的表的规则类似。主要区别如下：

+   使用`KEY`而不是`HASH`。

+   `KEY`只接受零个或多个列名的列表。用作分区键的列必须包含表的主键的一部分或全部，如果表有主键的话。如果未指定列名作为分区键，则使用表的主键，如果有的话。例如，以下`CREATE TABLE`语句在 MySQL 8.0 中是有效的：

    ```sql
    CREATE TABLE k1 (
        id INT NOT NULL PRIMARY KEY,
        name VARCHAR(20)
    )
    PARTITION BY KEY()
    PARTITIONS 2;
    ```

    如果没有主键但有唯一键，则唯一键将用作分区键：

    ```sql
    CREATE TABLE k1 (
        id INT NOT NULL,
        name VARCHAR(20),
        UNIQUE KEY (id)
    )
    PARTITION BY KEY()
    PARTITIONS 2;
    ```

    但是，如果唯一键列未定义为`NOT NULL`，则前面的语句将失败。

    在这两种情况下，分区键是`id`列，即使在`SHOW CREATE TABLE`的输出中或者在信息模式`PARTITIONS`表的`PARTITION_EXPRESSION`列中没有显示。

    与其他分区类型不同，`KEY`分区的列不限于整数或`NULL`值。例如，以下`CREATE TABLE`语句是有效的：

    ```sql
    CREATE TABLE tm1 (
        s1 CHAR(32) PRIMARY KEY
    )
    PARTITION BY KEY(s1)
    PARTITIONS 10;
    ```

    如果指定了不同的分区类型，则前面的语句将*无效*。（在这种情况下，仅使用`PARTITION BY KEY()`也是有效的，并且与`PARTITION BY KEY(s1)`具有相同的效果，因为`s1`是表的主键。）

    关于这个问题的更多信息，请参见第 26.6 节，“分区的限制和限制”。

    不支持具有索引前缀的列用作分区键。这意味着`CHAR`、`VARCHAR`、`BINARY`和`VARBINARY`列可以用作分区键，只要它们不使用前缀；因为在索引定义中必须指定前缀，所以无法在分区键中使用`BLOB`和`TEXT`列。在 MySQL 8.0.21 之前，创建、修改或升级分区表时允许使用前缀的列，即使它们未包含在表的分区键中；在 MySQL 8.0.21 及更高版本中，这种宽容行为已被弃用，并且当使用一个或多个这样的列时，服务器会显示适当的警告或错误。有关更多信息和示例，请参见不支持键分区的列索引前缀。

    注意

    使用`NDB`存储引擎的表隐式地通过`KEY`进行分区，使用表的主键作为分区键（与其他 MySQL 存储引擎一样）。如果 NDB Cluster 表没有显式主键，则由`NDB`存储引擎为每个 NDB Cluster 表生成的“隐藏”主键将用作分区键。

    如果为`NDB`表定义了显式分区方案，则表必须具有显式主键，并且分区表达式中使用的任何列必须是该键的一部分。但是，如果表使用“空”分区表达式——即`PARTITION BY KEY()`而没有列引用，则不需要显式主键。

    您可以使用**ndb_desc**实用程序（使用`-p`选项）观察到这种分区。

    重要

    对于使用键分区的表，您不能执行`ALTER TABLE DROP PRIMARY KEY`，因为这样做会生成错误 ERROR 1466 (HY000): Field in list of fields for partition function not found in table。对于使用`KEY`进行分区的 NDB Cluster 表，这不是问题；在这种情况下，表将使用“隐藏”的主键重新组织为表的新分区键。参见第二十五章，*MySQL NDB Cluster 8.0*。

也可以通过线性键对表进行分区。这里是一个简单的例子：

```sql
CREATE TABLE tk (
    col1 INT NOT NULL,
    col2 CHAR(5),
    col3 DATE
)
PARTITION BY LINEAR KEY (col1)
PARTITIONS 3;
```

`LINEAR` 关键字在 `KEY` 分区上具有与 `HASH` 分区相同的效果，分区号是使用二的幂算法而不是模算术推导出来的。参见第 26.2.4.1 节，“线性哈希分区”，了解该算法及其影响的描述。
