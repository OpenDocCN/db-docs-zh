# 10.9.6 优化器统计

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizer-statistics.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-statistics.html)

`column_statistics`数据字典表存储关于列值的直方图统计信息，供优化器在构建查询执行计划时使用。要执行直方图管理，请使用`ANALYZE TABLE`语句。

`column_statistics`表具有以下特征：

+   该表包含所有数据类型的列的统计信息，除了几何类型（空间数据）和`JSON`。

+   该表是持久的，因此每次服务器启动时不需要创建列统计信息。

+   服务器对表执行更新；用户不执行。

`column_statistics`表不直接可被用户访问，因为它是数据字典的一部分。直方图信息可通过`INFORMATION_SCHEMA.COLUMN_STATISTICS`获得，它被实现为数据字典表上的视图。`COLUMN_STATISTICS`包含以下列：

+   `SCHEMA_NAME`、`TABLE_NAME`、`COLUMN_NAME`：适用于统计的模式、表和列的名称。

+   `HISTOGRAM`：描述以直方图形式存储的列统计信息的`JSON`值。

列直方图包含存储在列中值范围的部分的桶。直方图是`JSON`对象，以允许在列统计的表示中灵活性。这是一个示例直方图对象：

```sql
{
  "buckets": [
    [
      1,
      0.3333333333333333
    ],
    [
      2,
      0.6666666666666666
    ],
    [
      3,
      1
    ]
  ],
  "null-values": 0,
  "last-updated": "2017-03-24 13:32:40.000000",
  "sampling-rate": 1,
  "histogram-type": "singleton",
  "number-of-buckets-specified": 128,
  "data-type": "int",
  "collation-id": 8
}
```

直方图对象具有以下键：

+   `buckets`：直方图桶。桶结构取决于直方图类型。

    对于`singleton`直方图，桶包含两个值：

    +   值 1：桶的值。类型取决于列数据类型。

    +   值 2：表示该值的累积频率的双精度值。例如，.25 和.75 表示列中值的 25%和 75%小于或等于桶值。

    对于`equi-height`直方图，桶包含四个值：

    +   值 1、2：桶的下限和上限值（包括）。类型取决于列数据类型。

    +   值 3：表示该值的累积频率的双精度值。例如，.25 和.75 表示列中值的 25%和 75%小于或等于桶的上限值。

    +   值 4：从桶下限值到上限值的范围内的不同值的数量。

+   `null-values`：介于 0.0 和 1.0 之间的数字，表示列值中为 SQL `NULL`值的比例。如果为 0，则列不包含`NULL`值。

+   `last-updated`：生成直方图的时间，以*`YYYY-MM-DD hh:mm:ss.uuuuuu`*格式的 UTC 值。

+   `sampling-rate`: 介于 0.0 和 1.0 之间的数字，表示用于创建直方图的数据比例。值为 1 表示读取了所有数据（无抽样）。

+   `histogram-type`: 直方图类型：

    +   `singleton`: 一个桶代表列中的一个单个值。当列中不同数值的数量小于或等于`ANALYZE TABLE`语句生成直方图时指定的桶数时，将创建此直方图类型。

    +   `equi-height`: 一个桶代表一系列数值。当列中不同数值的数量大于`ANALYZE TABLE`语句生成直方图时指定的桶数时，将创建此直方图类型。

+   `number-of-buckets-specified`: `ANALYZE TABLE`语句生成直方图时指定的桶数。

+   `data-type`: 此直方图包含的数据类型。在将直方图从持久存储读取和解析到内存时需要。其值为`int`、`uint`（无符号整数）、`double`、`decimal`、`datetime`或`string`（包括字符和二进制字符串）之一。

+   `collation-id`: 直方图数据的排序规则 ID。当`data-type`值为`string`时，这个值大多有意义。这些值对应于信息模式`COLLATIONS`表中的`ID`列值。

要从直方图对象中提取特定值，可以使用`JSON`操作。例如：

```sql
mysql> SELECT
         TABLE_NAME, COLUMN_NAME,
         HISTOGRAM->>'$."data-type"' AS 'data-type',
         JSON_LENGTH(HISTOGRAM->>'$."buckets"') AS 'bucket-count'
       FROM INFORMATION_SCHEMA.COLUMN_STATISTICS;
+-----------------+-------------+-----------+--------------+
| TABLE_NAME      | COLUMN_NAME | data-type | bucket-count |
+-----------------+-------------+-----------+--------------+
| country         | Population  | int       |          226 |
| city            | Population  | int       |         1024 |
| countrylanguage | Language    | string    |          457 |
+-----------------+-------------+-----------+--------------+
```

优化器使用直方图统计信息（如果适用）来处理收集了统计信息的任何数据类型的列。优化器应用直方图统计信息来根据列值与常量值的比较的选择性（过滤效果）确定行估计。以下形式的谓词符合直方图使用的条件：

```sql
*col_name* = *constant*
*col_name* <> *constant*
*col_name* != *constant*
*col_name* > *constant*
*col_name* < *constant*
*col_name* >= *constant*
*col_name* <= *constant*
*col_name* IS NULL
*col_name* IS NOT NULL
*col_name* BETWEEN *constant* AND *constant*
*col_name* NOT BETWEEN *constant* AND *constant*
*col_name* IN (*constant*[, *constant*] ...)
*col_name* NOT IN (*constant*[, *constant*] ...)
```

例如，以下语句包含符合直方图使用条件的谓词：

```sql
SELECT * FROM orders WHERE amount BETWEEN 100.0 AND 300.0;
SELECT * FROM tbl WHERE col1 = 15 AND col2 > 100;
```

对常量值的比较要求包括常量函数，如`ABS()`和`FLOOR()`：

```sql
SELECT * FROM tbl WHERE col1 < ABS(-34);
```

直方图统计信息主要适用于非索引列。为适用直方图统计信息的列添加索引也可能帮助优化器进行行估计。权衡如下：

+   当表数据被修改时，索引必须更新。

+   直方图仅在需要时创建或更新，因此在修改表数据时不会增加额外开销。另一方面，当表发生修改时，统计信息会逐渐变得过时，直到下次更新为止。

优化器更喜欢使用范围优化器的行估计值，而不是直方图统计数据。如果优化器确定范围优化器适用，则不使用直方图统计数据。

对于已建立索引的列，可以通过索引潜入来获取相等比较的行估计值（参见 Section 10.2.1.2, “Range Optimization”）。在这种情况下，直方图统计数据未必有用，因为索引潜入可以提供更好的估计值。

在某些情况下，使用直方图统计数据可能不会改善查询执行（例如，如果统计数据已过时）。要检查是否是这种情况，请使用`ANALYZE TABLE`重新生成直方图统计数据，然后再次运行查询。

或者，要禁用直方图统计数据，可以使用`ANALYZE TABLE`来删除它们。另一种禁用直方图统计数据的方法是关闭`optimizer_switch`系统变量的`condition_fanout_filter`标志（尽管这可能会禁用其他优化）：

```sql
SET optimizer_switch='condition_fanout_filter=off';
```

如果使用直方图统计数据，可以通过`EXPLAIN`查看结果。考虑以下查询，其中`col1`列没有可用的索引：

```sql
SELECT * FROM t1 WHERE col1 < 24;
```

如果直方图统计数据表明`t1`中有 57%的行满足`col1 < 24`的条件，即使没有索引，也可以进行过滤，并且`EXPLAIN`中的`filtered`列显示 57.00。
