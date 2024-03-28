> 原文：[`dev.mysql.com/doc/refman/8.0/en/alter-table-partition-operations.html`](https://dev.mysql.com/doc/refman/8.0/en/alter-table-partition-operations.html)

#### 15.1.9.1 修改表分区操作

与分区表相关的`ALTER TABLE`的分区相关子句可用于对分区表进行重新分区，添加、删除、丢弃、导入、合并和拆分分区，以及执行分区维护。

+   在分区表上简单地使用带有`ALTER TABLE`的*`partition_options`*子句会根据*`partition_options`*定义的分区方案重新分区表。该子句始终以`PARTITION BY`开头，并遵循与`CREATE TABLE`的*`partition_options`*子句相同的语法和其他规则（有关更详细信息，请参见第 15.1.20 节，“CREATE TABLE Statement”），也可用于对尚未分区的现有表进行分区。例如，考虑一个（非分区的）表定义如下：

    ```sql
    CREATE TABLE t1 (
        id INT,
        year_col INT
    );
    ```

    可以使用以下语句将此表按`HASH`进行分区，使用`id`列作为分区键，分为 8 个分区：

    ```sql
    ALTER TABLE t1
        PARTITION BY HASH(id)
        PARTITIONS 8;
    ```

    MySQL 支持与`[SUB]PARTITION BY [LINEAR] KEY`一起使用的`ALGORITHM`选项。`ALGORITHM=1`使服务器在计算行在分区中的放置时使用与 MySQL 5.1 相同的键哈希函数；`ALGORITHM=2`表示服务器使用 MySQL 5.5 及更高版本中新的`KEY`分区表默认实现和使用的键哈希函数。 （使用 MySQL 5.5 及更高版本中实施的键哈希函数创建的分区表不能被 MySQL 5.1 服务器使用。）不指定该选项与使用`ALGORITHM=2`具有相同效果。该选项主要用于在 MySQL 5.1 及更高版本之间升级或降级`[LINEAR] KEY`分区表，或者在 MySQL 5.5 或更高版本服务器上创建由`KEY`或`LINEAR KEY`分区的表，该表可在 MySQL 5.1 服务器上使用。

    使用`ALTER TABLE ... PARTITION BY`语句得到的表必须遵循与使用`CREATE TABLE ... PARTITION BY`创建的表相同的规则。这包括表可能具有的任何唯一键（包括任何主键）与用于分区表达式的列之间的关系规则，如第 26.6.1 节，“分区键、主键和唯一键”中所讨论的。指定分区数量的`CREATE TABLE ... PARTITION BY`规则也适用于`ALTER TABLE ... PARTITION BY`。

    `ALTER TABLE ADD PARTITION` 的 *`partition_definition`* 子句支持与`CREATE TABLE`语句中同名子句相同的选项。 （有关语法和描述，请参见 Section 15.1.20, “CREATE TABLE Statement”。）假设您已按此处所示创建了分区表：

    ```sql
    CREATE TABLE t1 (
        id INT,
        year_col INT
    )
    PARTITION BY RANGE (year_col) (
        PARTITION p0 VALUES LESS THAN (1991),
        PARTITION p1 VALUES LESS THAN (1995),
        PARTITION p2 VALUES LESS THAN (1999)
    );
    ```

    您可以按以下方式向此表添加一个新的分区 `p3`，用于存储小于 `2002` 的值：

    ```sql
    ALTER TABLE t1 ADD PARTITION (PARTITION p3 VALUES LESS THAN (2002));
    ```

    `DROP PARTITION` 可用于删除一个或多个 `RANGE` 或 `LIST` 分区。此语句不能与 `HASH` 或 `KEY` 分区一起使用；而应使用 `COALESCE PARTITION`（请参见本节后面的内容）。丢弃在 *`partition_names`* 列表中命名的已删除分区中存储的任何数据。例如，给定先前定义的表 `t1`，您可以按如下所示删除命名为 `p0` 和 `p1` 的分区：

    ```sql
    ALTER TABLE t1 DROP PARTITION p0, p1;
    ```

    注意

    `DROP PARTITION` 不能用于使用`NDB`存储引擎的表。请参阅 Section 26.3.1, “RANGE 和 LIST 分区的管理”，以及 Section 25.2.7, “NDB Cluster 的已知限制”。

    `ADD PARTITION` 和 `DROP PARTITION` 目前不支持 `IF [NOT] EXISTS`。

    `DISCARD PARTITION ... TABLESPACE` 和 `IMPORT PARTITION ... TABLESPACE` 选项将可传输表空间功能扩展到单个 `InnoDB` 表分区。每个 `InnoDB` 表分区都有自己的表空间文件（`.ibd` 文件）。可传输表空间功能使得从一个运行中的 MySQL 服务器实例复制表空间到另一个运行实例，或在同一实例上执行还原变得容易。这两个选项都接受一个逗号分隔的一个或多个分区名称列表。例如：

    ```sql
    ALTER TABLE t1 DISCARD PARTITION p2, p3 TABLESPACE;
    ```

    ```sql
    ALTER TABLE t1 IMPORT PARTITION p2, p3 TABLESPACE;
    ```

    在对子分区表运行`DISCARD PARTITION ... TABLESPACE`和`IMPORT PARTITION ... TABLESPACE`时，允许使用分区和子分区名称。当指定分区名称时，将包括该分区的子分区。

    可传输表空间功能还支持复制或还原分区的 `InnoDB` 表。有关更多信息，请参阅 Section 17.6.1.3, “导入 InnoDB 表”。

    支持对分区表进行重命名。您可以间接地使用`ALTER TABLE ... REORGANIZE PARTITION`重命名单个分区；但是，此操作会复制分区的数据。

    要从选定的分区中删除行，请使用`TRUNCATE PARTITION`选项。此选项接受一个或多个逗号分隔的分区名称列表。考虑通过此语句创建的表`t1`：

    ```sql
    CREATE TABLE t1 (
        id INT,
        year_col INT
    )
    PARTITION BY RANGE (year_col) (
        PARTITION p0 VALUES LESS THAN (1991),
        PARTITION p1 VALUES LESS THAN (1995),
        PARTITION p2 VALUES LESS THAN (1999),
        PARTITION p3 VALUES LESS THAN (2003),
        PARTITION p4 VALUES LESS THAN (2007)
    );
    ```

    要从分区`p0`中删除所有行，请使用以下语句：

    ```sql
    ALTER TABLE t1 TRUNCATE PARTITION p0;
    ```

    刚刚显示的语句具有与以下`DELETE`语句相同的效果：

    ```sql
    DELETE FROM t1 WHERE year_col < 1991;
    ```

    在截断多个分区时，分区不必是连续的：这可以极大简化对分区表的删除操作，否则如果使用`DELETE`语句进行操作，则需要非常复杂的`WHERE`条件。例如，此语句删除分区`p1`和`p3`中的所有行：

    ```sql
    ALTER TABLE t1 TRUNCATE PARTITION p1, p3;
    ```

    这里显示了一个等效的`DELETE`语句：

    ```sql
    DELETE FROM t1 WHERE
        (year_col >= 1991 AND year_col < 1995)
        OR
        (year_col >= 2003 AND year_col < 2007);
    ```

    如果在分区名单的位置使用`ALL`关键字，则该语句将作用于所有表分区。

    `TRUNCATE PARTITION`仅删除行；它不会改变表本身的定义，也不会改变任何分区的定义。

    要验证行是否已删除，请检查`INFORMATION_SCHEMA.PARTITIONS`表，使用类似于以下查询的查询：

    ```sql
    SELECT PARTITION_NAME, TABLE_ROWS
        FROM INFORMATION_SCHEMA.PARTITIONS
        WHERE TABLE_NAME = 't1';
    ```

    `COALESCE PARTITION`可用于由`HASH`或`KEY`分区的表，以减少*`number`*个分区的数量。假设您已创建表`t2`如下所示：

    ```sql
    CREATE TABLE t2 (
        name VARCHAR (30),
        started DATE
    )
    PARTITION BY HASH( YEAR(started) )
    PARTITIONS 6;
    ```

    要将`t2`使用的分区数量从 6 减少到 4，请使用以下语句：

    ```sql
    ALTER TABLE t2 COALESCE PARTITION 2;
    ```

    最后*`number`*个分区中包含的数据合并到剩余分区中。在这种情况下，分区 4 和 5 合并到前 4 个分区（编号为 0、1、2 和 3 的分区）。

    要更改分区表中使用的一些但不是所有分区，可以使用`REORGANIZE PARTITION`。此语句可以以几种方式使用：

    +   将一组分区合并为单个分区。通过在*`partition_names`*列表中命名多个分区并提供*`partition_definition`*的单个定义来完成此操作。

    +   将现有分区分割为多个分区。通过为*`partition_names`*命名单个分区并提供多个*`partition_definitions`*来实现此目的。

    +   更改使用`VALUES LESS THAN`定义的一部分分区的范围，或者更改使用`VALUES IN`定义的一部分分区的值列表。

    注意

    对于未明确命名的分区，MySQL 会自动提供默认名称`p0`、`p1`、`p2`等。关于子分区也是如此。

    有关`ALTER TABLE ... REORGANIZE PARTITION`语句的更详细信息和示例，请参见第 26.3.1 节，“RANGE 和 LIST 分区的管理”。

+   要与表交换分区或子分区，请使用`ALTER TABLE ... EXCHANGE PARTITION`语句——即将分区或子分区中的任何现有行移动到非分区表中，并将非分区表中的任何现有行移动到表分区或子分区中。

    一旦使用`ALGORITHM=INSTANT`向分区表添加了一个或多个列，就不再可能与该表交换分区。

    查看用法信息和示例，请参见第 26.3.3 节，“使用表交换分区和子分区”。

+   几个选项提供了类似于非分区表通过`CHECK TABLE`和`REPAIR TABLE`实现的分区维护和修复功能（这些语句也支持分区表；有关更多信息，请参见第 15.7.3 节，“表维护语句”）。这些包括`ANALYZE PARTITION`、`CHECK PARTITION`、`OPTIMIZE PARTITION`、`REBUILD PARTITION`和`REPAIR PARTITION`。每个选项都需要一个*`partition_names`*子句，由一个或多个分区名称组成，用逗号分隔。这些分区必须已经存在于目标表中。您还可以在*`partition_names`*的位置使用`ALL`关键字，此时该语句将作用于所有表分区。有关更多信息和示例，请参见第 26.3.4 节，“分区维护”。

    `InnoDB`目前不支持按分区进行优化；`ALTER TABLE ... OPTIMIZE PARTITION`会导致整个表被重建和分析，并发出适当的警告。（Bug #11751825，Bug #42822）为解决此问题，请改用`ALTER TABLE ... REBUILD PARTITION`和`ALTER TABLE ... ANALYZE PARTITION`。

    对于未分区的表，不支持`ANALYZE PARTITION`、`CHECK PARTITION`、`OPTIMIZE PARTITION`和`REPAIR PARTITION`选项。

+   `REMOVE PARTITIONING`使您可以删除表的分区，而不影响表或其数据。此选项可以与其他`ALTER TABLE`选项结合使用，例如用于添加、删除或重命名列或索引的选项。

+   使用`ENGINE`选项与`ALTER TABLE`一起，可以更改表使用的存储引擎，而不影响分区。目标存储引擎必须提供自己的分区处理程序。只有`InnoDB`和`NDB`存储引擎具有本机分区处理程序；`NDB`目前不受 MySQL 8.0 支持。

`ALTER TABLE` 语句可以包含 `PARTITION BY` 或 `REMOVE PARTITIONING` 子句以及其他修改规范，但 `PARTITION BY` 或 `REMOVE PARTITIONING` 子句必须在任何其他规范之后指定。

`ADD PARTITION`、`DROP PARTITION`、`COALESCE PARTITION`、`REORGANIZE PARTITION`、`ANALYZE PARTITION`、`CHECK PARTITION` 和 `REPAIR PARTITION` 选项不能与其他修改规范组合在单个 `ALTER TABLE` 中，因为上述列出的选项作用于单个分区。有关更多信息，请参见 Section 15.1.9.1, “ALTER TABLE Partition Operations”。

在给定的 `ALTER TABLE` 语句中，只能使用以下选项中的任何一个的单个实例：`PARTITION BY`、`ADD PARTITION`、`DROP PARTITION`、`TRUNCATE PARTITION`、`EXCHANGE PARTITION`、`REORGANIZE PARTITION`、`COALESCE PARTITION`、`ANALYZE PARTITION`、`CHECK PARTITION`、`OPTIMIZE PARTITION`、`REBUILD PARTITION` 或 `REMOVE PARTITIONING`。

例如，以下两个语句是无效的：

```sql
ALTER TABLE t1 ANALYZE PARTITION p1, ANALYZE PARTITION p2;

ALTER TABLE t1 ANALYZE PARTITION p1, CHECK PARTITION p2;
```

在第一种情况下，你可以使用单个语句并列出要分析的分区 `p1` 和 `p2` 的表 `t1`，同时并行分析这两个分区，如下所示：

```sql
ALTER TABLE t1 ANALYZE PARTITION p1, p2;
```

在第二种情况下，不可能同时对同一表的不同分区执行 `ANALYZE` 和 `CHECK` 操作。相反，你必须发出两个单独的语句，如下所示：

```sql
ALTER TABLE t1 ANALYZE PARTITION p1;
ALTER TABLE t1 CHECK PARTITION p2;
```

`REBUILD` 操作目前不支持子分区。`REBUILD` 关键字明确禁止与子分区一起使用，并且如果这样使用会导致 `ALTER TABLE` 失败并显示错误。

当要检查或修复的分区包含任何重复键错误时，`CHECK PARTITION` 和 `REPAIR PARTITION` 操作将失败。

有关这些语句的更多信息，请参见 Section 26.3.4, “Maintenance of Partitions”。
