> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-persistent-stats.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-persistent-stats.html)

#### 17.8.10.1 配置持久性优化器统计参数

持久性优化器统计功能通过将统计数据存储到磁盘并使其在服务器重新启动时持久化，从而改善了计划稳定性，使得优化器更有可能为给定查询每次做出一致的选择。

当`innodb_stats_persistent=ON`或当单独的表被定义为`STATS_PERSISTENT=1`时，优化器统计数据会持久化到磁盘。`innodb_stats_persistent`默认启用。

以前，当重新启动服务器和执行其他某些类型的操作时，优化器统计数据会被清除，并在下一次访问表时重新计算。因此，在重新计算统计数据时可能会产生不同的估计值，导致查询执行计划中的不同选择和查询性能的变化。

持久性统计数据存储在`mysql.innodb_table_stats`和`mysql.innodb_index_stats`表中。请参阅第 17.8.10.1.5 节，“InnoDB 持久性统计表”。

如果您不希望将优化器统计数据持久化到磁盘，请参阅第 17.8.10.2 节，“配置非持久性优化器统计参数”

##### 17.8.10.1.1 配置持久性优化器统计的自动计算

默认启用的`innodb_stats_auto_recalc`变量控制当表的行数变化超过 10%时是否自动计算统计数据。您还可以通过在创建或更改表时指定`STATS_AUTO_RECALC`子句来为单独的表配置自动统计数据重新计算。

由于自动统计重新计算的异步性质，即使启用了`innodb_stats_auto_recalc`，在运行影响表超过 10%的 DML 操作后，统计数据可能不会立即重新计算。在某些情况下，统计数据重新计算可能会延迟几秒钟。如果需要立即更新的统计数据，请运行`ANALYZE TABLE`来启动同步（前台）重新计算统计数据。

如果禁用了`innodb_stats_auto_recalc`，您可以通过在对索引列进行重大更改后执行`ANALYZE TABLE`语句来确保优化器统计的准确性。您还可以考虑将`ANALYZE TABLE`添加到在加载数据后运行的设置脚本中，并在低活动时间定期运行`ANALYZE TABLE`。

当向现有表添加索引或添加或删除列时，无论`innodb_stats_auto_recalc`的值如何，都会计算索引统计信息并将其添加到`innodb_index_stats`表中。

##### 17.8.10.1.2 为单个表配置优化器统计参数

`innodb_stats_persistent`，`innodb_stats_auto_recalc`，和`innodb_stats_persistent_sample_pages`是全局变量。要覆盖这些系统范围的设置，并为单个表配置优化器统计参数，您可以在`CREATE TABLE`或`ALTER TABLE`语句中定义`STATS_PERSISTENT`，`STATS_AUTO_RECALC`和`STATS_SAMPLE_PAGES`子句。

+   `STATS_PERSISTENT`指定是否为`InnoDB`表启用持久统计信息。值`DEFAULT`导致表的持久统计设置由`innodb_stats_persistent`设置确定。值为`1`时，为表启用持久统计信息，值为`0`时禁用该功能。在为单个表启用持久统计信息后，使用`ANALYZE TABLE`在加载表数据后计算统计信息。

+   `STATS_AUTO_RECALC`指定是否自动重新计算持久统计信息。值`DEFAULT`导致表的持久统计设置由`innodb_stats_auto_recalc`设置确定。值为`1`时，当表数据变化了 10%时重新计算统计信息。值为`0`时，阻止对表进行自动重新计算。当使用值为 0 时，使用`ANALYZE TABLE`在对表进行重大更改后重新计算统计信息。

+   `STATS_SAMPLE_PAGES` 指定了在对索引列进行统计计算时，通过 `ANALYZE TABLE` 操作采样的索引页面数量，例如。

所有三个子句在以下 `CREATE TABLE` 示例中指定：

```sql
CREATE TABLE `t1` (
`id` int(8) NOT NULL auto_increment,
`data` varchar(255),
`date` datetime,
PRIMARY KEY  (`id`),
INDEX `DATE_IX` (`date`)
) ENGINE=InnoDB,
  STATS_PERSISTENT=1,
  STATS_AUTO_RECALC=1,
  STATS_SAMPLE_PAGES=25;
```

##### 17.8.10.1.3 配置 InnoDB 优化器统计信息的采样页面数

优化器使用关于键分布的估计 统计信息 来选择执行计划的索引，基于索引的相对 选择性。诸如 `ANALYZE TABLE` 等操作会导致 `InnoDB` 从表中的每个索引中随机采样页面，以估算索引的 基数。这种采样技术被称为 随机潜水。

`innodb_stats_persistent_sample_pages` 控制了采样页面的数量。您可以在运行时调整此设置以管理优化器使用的统计估算的质量。默认值为 20。在遇到以下问题时考虑修改此设置：

1.  *统计数据不够准确且优化器选择了次优执行计划*，如 `EXPLAIN` 输出所示。您可以通过比较索引的实际基数（通过在索引列上运行 `SELECT DISTINCT` 确定）与 `mysql.innodb_index_stats` 表中的估算值来检查统计数据的准确性。

    如果确定统计数据不够准确，则应增加 `innodb_stats_persistent_sample_pages` 的值，直到统计估算足够准确。然而，增加 `innodb_stats_persistent_sample_pages` 太多可能会导致 `ANALYZE TABLE` 运行缓慢。

1.  *`ANALYZE TABLE` 运行太慢*。在这种情况下，应将 `innodb_stats_persistent_sample_pages` 减少，直到 `ANALYZE TABLE` 的执行时间可接受。然而，将值减少太多可能会导致统计不准确和查询执行计划不佳的第一个问题。

    如果无法在准确统计和`ANALYZE TABLE`执行时间之间取得平衡，请考虑减少表中索引列的数量或限制分区数量以减少`ANALYZE TABLE`的复杂性。还需要考虑表主键中的列数，因为主键列会附加到每个非唯一索引上。

    有关相关信息，请参阅第 17.8.10.3 节，“InnoDB 表的 ANALYZE TABLE 复杂性估算”。

##### 17.8.10.1.4 在持久统计计算中包含已标记删除的记录

默认情况下，`InnoDB`在计算统计信息时会读取未提交的数据。在未提交事务删除表中的行的情况下，计算行估计和索引统计时会排除已标记删除的记录，这可能导致使用除`READ UNCOMMITTED`之外的事务隔离级别并行操作表的其他事务出现非最佳执行计划。为避免这种情况，可以启用`innodb_stats_include_delete_marked`以确保在计算持久性优化器统计信息时包含已标记删除的记录。

当启用`innodb_stats_include_delete_marked`时，`ANALYZE TABLE`在重新计算统计信息时会考虑已标记删除的记录。

`innodb_stats_include_delete_marked`是一个影响所有`InnoDB`表的全局设置，仅适用于持久性优化器统计信息。

##### 17.8.10.1.5 InnoDB 持久性统计表

持久性统计功能依赖于`mysql`数据库中的内部管理表，名称为`innodb_table_stats`和`innodb_index_stats`。这些表会在所有安装、升级和源代码构建过程中自动设置。

**表 17.6 innodb_table_stats 的列**

| 列名 | 描述 |
| --- | --- |
| `database_name` | 数据库名称 |
| `table_name` | 表名、分区名或子分区名 |
| `last_update` | 指示`InnoDB`上次更新此行的时间戳 |
| `n_rows` | 表中的行数 |
| `clustered_index_size` | 主索引的大小，以页为单位 |
| `sum_of_other_index_sizes` | 其他（非主键）索引的总大小，以页为单位 |

**表 17.7 innodb_index_stats 的列**

| 列名 | 描述 |
| --- | --- |
| `database_name` | 数据库名称 |
| `table_name` | 表名、分区名或子分区名 |
| `index_name` | 索引名称 |
| `last_update` | 指示上次更新行的时间戳 |
| `stat_name` | 统计信息的名称，其值在`stat_value`列中报告 |
| `stat_value` | 在`stat_name`列中命名的统计信息的值 |
| `sample_size` | 用于`stat_value`列中提供的估计值的页面样本数 |
| `stat_description` | 在`stat_name`列中命名的统计信息的描述 |

`innodb_table_stats`和`innodb_index_stats`表包括一个`last_update`列，显示索引统计信息上次更新的时间：

```sql
mysql> SELECT * FROM innodb_table_stats \G
*************************** 1\. row ***************************
           database_name: sakila
              table_name: actor
             last_update: 2014-05-28 16:16:44
                  n_rows: 200
    clustered_index_size: 1
sum_of_other_index_sizes: 1
...
```

```sql
mysql> SELECT * FROM innodb_index_stats \G
*************************** 1\. row ***************************
   database_name: sakila
      table_name: actor
      index_name: PRIMARY
     last_update: 2014-05-28 16:16:44
       stat_name: n_diff_pfx01
      stat_value: 200
     sample_size: 1
     ...
```

`innodb_table_stats`和`innodb_index_stats`表可以手动更新，这样可以强制执行特定的查询优化计划或测试替代计划而不修改数据库。如果手动更新统计信息，请使用`FLUSH TABLE *tbl_name*`语句加载更新后的统计信息。

持久统计被视为本地信息，因为它们与服务器实例相关。因此，在自动统计信息重新计算时，`innodb_table_stats`和`innodb_index_stats`表不会被复制。如果运行`ANALYZE TABLE`来启动统计信息的同步重新计算，该语句会被复制（除非你禁止了它的日志记录），并且重新计算会在副本上进行。

##### 17.8.10.1.6 InnoDB 持久统计表示例

`innodb_table_stats`表中包含每个表的一行。以下示例演示了收集的数据类型。

表`t1`包含一个主索引（列`a`，`b`），一个次要索引（列`c`，`d`），和一个唯一索引（列`e`，`f`）：

```sql
CREATE TABLE t1 (
a INT, b INT, c INT, d INT, e INT, f INT,
PRIMARY KEY (a, b), KEY i1 (c, d), UNIQUE KEY i2uniq (e, f)
) ENGINE=INNODB;
```

插入五行样本数据后，表`t1`如下所示：

```sql
mysql> SELECT * FROM t1;
+---+---+------+------+------+------+
| a | b | c    | d    | e    | f    |
+---+---+------+------+------+------+
| 1 | 1 |   10 |   11 |  100 |  101 |
| 1 | 2 |   10 |   11 |  200 |  102 |
| 1 | 3 |   10 |   11 |  100 |  103 |
| 1 | 4 |   10 |   12 |  200 |  104 |
| 1 | 5 |   10 |   12 |  100 |  105 |
+---+---+------+------+------+------+
```

要立即更新统计信息，运行`ANALYZE TABLE`（如果`innodb_stats_auto_recalc`已启用，则假定在几秒钟内自动更新统计信息，假设已达到更改表行的 10%阈值）：

```sql
mysql> ANALYZE TABLE t1;
+---------+---------+----------+----------+
| Table   | Op      | Msg_type | Msg_text |
+---------+---------+----------+----------+
| test.t1 | analyze | status   | OK       |
+---------+---------+----------+----------+
```

表`t1`的表统计信息显示了`InnoDB`上次更新表统计信息的时间（`2014-03-14 14:36:34`），表中的行数（`5`），聚集索引大小（`1`页），以及其他索引的组合大小（`2`页）。

```sql
mysql> SELECT * FROM mysql.innodb_table_stats WHERE table_name like 't1'\G
*************************** 1\. row ***************************
           database_name: test
              table_name: t1
             last_update: 2014-03-14 14:36:34
                  n_rows: 5
    clustered_index_size: 1
sum_of_other_index_sizes: 2
```

`innodb_index_stats`表对每个索引包含多行。`innodb_index_stats`表中的每一行提供与`stat_name`列中命名的特定索引统计信息相关的数据，并在`stat_description`列中描述。例如：

```sql
mysql> SELECT index_name, stat_name, stat_value, stat_description
       FROM mysql.innodb_index_stats WHERE table_name like 't1';
+------------+--------------+------------+-----------------------------------+
| index_name | stat_name    | stat_value | stat_description                  |
+------------+--------------+------------+-----------------------------------+
| PRIMARY    | n_diff_pfx01 |          1 | a                                 |
| PRIMARY    | n_diff_pfx02 |          5 | a,b                               |
| PRIMARY    | n_leaf_pages |          1 | Number of leaf pages in the index |
| PRIMARY    | size         |          1 | Number of pages in the index      |
| i1         | n_diff_pfx01 |          1 | c                                 |
| i1         | n_diff_pfx02 |          2 | c,d                               |
| i1         | n_diff_pfx03 |          2 | c,d,a                             |
| i1         | n_diff_pfx04 |          5 | c,d,a,b                           |
| i1         | n_leaf_pages |          1 | Number of leaf pages in the index |
| i1         | size         |          1 | Number of pages in the index      |
| i2uniq     | n_diff_pfx01 |          2 | e                                 |
| i2uniq     | n_diff_pfx02 |          5 | e,f                               |
| i2uniq     | n_leaf_pages |          1 | Number of leaf pages in the index |
| i2uniq     | size         |          1 | Number of pages in the index      |
+------------+--------------+------------+-----------------------------------+
```

`stat_name`列显示以下类型的统计信息：

+   `size`：当`stat_name`=`size`时，`stat_value`列显示索引中的总页数。

+   `n_leaf_pages`：当`stat_name`=`n_leaf_pages`时，`stat_value`列显示索引中叶子页的数量。

+   `n_diff_pfx*`NN`*`：当`stat_name`=`n_diff_pfx01`时，`stat_value`列显示索引的第一列中不同值的数量。当`stat_name`=`n_diff_pfx02`时，`stat_value`列显示索引的前两列中不同值的数量，依此类推。当`stat_name`=`n_diff_pfx*`NN`*`时，`stat_description`列显示一个逗号分隔的计数索引列的列表。

为了进一步说明提供基数数据的`n_diff_pfx*`NN`*`统计信息，再次考虑之前介绍的`t1`表示例。如下所示，`t1`表创建了一个主索引（列`a`、`b`）、一个次要索引（列`c`、`d`）和一个唯一索引（列`e`、`f`）：

```sql
CREATE TABLE t1 (
  a INT, b INT, c INT, d INT, e INT, f INT,
  PRIMARY KEY (a, b), KEY i1 (c, d), UNIQUE KEY i2uniq (e, f)
) ENGINE=INNODB;
```

插入五行样本数据后，表`t1`如下所示：

```sql
mysql> SELECT * FROM t1;
+---+---+------+------+------+------+
| a | b | c    | d    | e    | f    |
+---+---+------+------+------+------+
| 1 | 1 |   10 |   11 |  100 |  101 |
| 1 | 2 |   10 |   11 |  200 |  102 |
| 1 | 3 |   10 |   11 |  100 |  103 |
| 1 | 4 |   10 |   12 |  200 |  104 |
| 1 | 5 |   10 |   12 |  100 |  105 |
+---+---+------+------+------+------+
```

当查询`index_name`、`stat_name`、`stat_value`和`stat_description`，其中`stat_name LIKE 'n_diff%'`时，将返回以下结果集：

```sql
mysql> SELECT index_name, stat_name, stat_value, stat_description
       FROM mysql.innodb_index_stats
       WHERE table_name like 't1' AND stat_name LIKE 'n_diff%';
+------------+--------------+------------+------------------+
| index_name | stat_name    | stat_value | stat_description |
+------------+--------------+------------+------------------+
| PRIMARY    | n_diff_pfx01 |          1 | a                |
| PRIMARY    | n_diff_pfx02 |          5 | a,b              |
| i1         | n_diff_pfx01 |          1 | c                |
| i1         | n_diff_pfx02 |          2 | c,d              |
| i1         | n_diff_pfx03 |          2 | c,d,a            |
| i1         | n_diff_pfx04 |          5 | c,d,a,b          |
| i2uniq     | n_diff_pfx01 |          2 | e                |
| i2uniq     | n_diff_pfx02 |          5 | e,f              |
+------------+--------------+------------+------------------+
```

对于`PRIMARY`索引，有两个`n_diff%`行。行数等于索引中的列数。

注意

对于非唯一索引，`InnoDB`会附加主键的列。

+   当`index_name`=`PRIMARY`且`stat_name`=`n_diff_pfx01`时，`stat_value`为`1`，表示索引的第一列（列`a`）中有一个单独的值。通过查看表`t1`中列`a`中的数据，确认列`a`中的不同值的数量，其中有一个单独的值（`1`）。计数的列（`a`）显示在结果集的`stat_description`列中。

+   当`index_name`=`PRIMARY`且`stat_name`=`n_diff_pfx02`时，`stat_value`为`5`，表示索引的两列（`a,b`）中有五个不同的值。通过查看表`t1`中列`a`和`b`中的数据，确认列`a`和`b`中的不同值的数量，其中有五个不同的值：(`1,1`)、(`1,2`)、(`1,3`)、(`1,4`)和(`1,5`)。计数的列（`a,b`）显示在结果集的`stat_description`列中。

对于次要索引（`i1`），有四个`n_diff%`行。次要索引（`c,d`）仅定义了两列，但有四个`n_diff%`行，因为`InnoDB`会在所有非唯一索引后缀主键。因此，有四个`n_diff%`行，而不是两个，以考虑次要索引列（`c,d`）和主键列（`a,b`）。

+   当`index_name`=`i1`且`stat_name`=`n_diff_pfx01`时，`stat_value`为`1`，表示索引的第一列（列`c`）中有一个单独的值。通过查看表`t1`中列`c`中的数据，确认列`c`中的不同值的数量，其中有一个单独的值：(`10`)。计数的列（`c`）显示在结果集的`stat_description`列中。

+   当 `index_name`=`i1` 且 `stat_name`=`n_diff_pfx02` 时，`stat_value` 为 `2`，表示索引（`c,d`）的前两列中有两个不同的值。在表 `t1` 中查看列 `c` 和 `d` 的数据，可以确认有两个不同的值：(`10,11`) 和 (`10,12`)。计数的列（`c,d`）显示在结果集的 `stat_description` 列中。

+   当 `index_name`=`i1` 且 `stat_name`=`n_diff_pfx03` 时，`stat_value` 为 `2`，表示索引（`c,d,a`）的前三列中有两个不同的值。在表 `t1` 中查看列 `c`、`d` 和 `a` 的数据，可以确认有两个不同的值：(`10,11,1`) 和 (`10,12,1`)。计数的列（`c,d,a`）显示在结果集的 `stat_description` 列中。

+   当 `index_name`=`i1` 且 `stat_name`=`n_diff_pfx04` 时，`stat_value` 为 `5`，表示索引（`c,d,a,b`）的四列中有五个不同的值。在表 `t1` 中查看列 `c`、`d`、`a` 和 `b` 的数据，可以确认有五个不同的值：(`10,11,1,1`)、(`10,11,1,2`)、(`10,11,1,3`)、(`10,12,1,4`) 和 (`10,12,1,5`)。计数的列（`c,d,a,b`）显示在结果集的 `stat_description` 列中。

对于唯一索引（`i2uniq`），有两行 `n_diff%`。

+   当 `index_name`=`i2uniq` 且 `stat_name`=`n_diff_pfx01` 时，`stat_value` 为 `2`，表示索引（列 `e`）的第一列中有两个不同的值。在表 `t1` 中查看列 `e` 的数据，可以确认有两个不同的值：(`100`) 和 (`200`)。计数的列（`e`）显示在结果集的 `stat_description` 列中。

+   当 `index_name`=`i2uniq` 且 `stat_name`=`n_diff_pfx02` 时，`stat_value` 为 `5`，表示索引（`e,f`）的两列中有五个不同的值。在表 `t1` 中查看列 `e` 和 `f` 的数据，可以确认有五个不同的值：(`100,101`)、(`200,102`)、(`100,103`)、(`200,104`) 和 (`100,105`)。计数的列（`e,f`）显示在结果集的 `stat_description` 列中。

##### 17.8.10.1.7 使用 `innodb_index_stats` 表检索索引大小

您可以使用 `innodb_index_stats` 表检索表、分区或子分区的索引大小。在下面的示例中，检索了表 `t1` 的索引大小。有关表 `t1` 和相应索引统计信息的定义，请参见第 17.8.10.1.6 节，“InnoDB 持久性统计表示例”。

```sql
mysql> SELECT SUM(stat_value) pages, index_name,
       SUM(stat_value)*@@innodb_page_size size
       FROM mysql.innodb_index_stats WHERE table_name='t1'
       AND stat_name = 'size' GROUP BY index_name;
+-------+------------+-------+
| pages | index_name | size  |
+-------+------------+-------+
|     1 | PRIMARY    | 16384 |
|     1 | i1         | 16384 |
|     1 | i2uniq     | 16384 |
+-------+------------+-------+
```

对于分区或子分区，您可以使用相同的查询，只需修改`WHERE`子句以检索索引大小。例如，以下查询检索表`t1`的分区索引大小：

```sql
mysql> SELECT SUM(stat_value) pages, index_name,
       SUM(stat_value)*@@innodb_page_size size
       FROM mysql.innodb_index_stats WHERE table_name like 't1#P%'
       AND stat_name = 'size' GROUP BY index_name;
```
