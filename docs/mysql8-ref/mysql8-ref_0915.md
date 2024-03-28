> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-secondary-indexes.html)

#### 15.1.20.9 二级索引和生成列

`InnoDB`支持虚拟生成列上的二级索引。不支持其他索引类型。在虚拟列上定义的二级索引有时被称为“虚拟索引”。

可以在一个或多个虚拟列上或在虚拟列和常规列或存储生成列的组合上创建二级索引。包含虚拟列的二级索引可以被定义为`UNIQUE`。

当在虚拟生成列上创建二级索引时，生成列的值会实现在索引记录中。如果索引是一个覆盖索引（包含查询检索的所有列），生成列的值将从索引结构中的实现值中检索，而不是实时计算。

使用虚拟列上的二级索引时需要考虑额外的写入成本，因为在`INSERT`和`UPDATE`操作期间，在二级索引记录中实现虚拟列值时执行计算。即使有额外的写入成本，虚拟列上的二级索引可能比生成*存储*列更可取，后者实现在聚簇索引中，导致需要更多磁盘空间和内存的更大表。如果在虚拟列上未定义二级索引，则读取时会有额外的成本，因为每次检查列的行时都必须计算虚拟列值。

虚拟列的索引列值被 MVCC 记录，以避免在回滚或清除操作期间重新计算生成列值。对于`COMPACT`和`REDUNDANT`行格式，记录值的数据长度受索引键限制的限制为 767 字节，对于`DYNAMIC`和`COMPRESSED`行格式，为 3072 字节。

在虚拟列上添加或删除二级索引是一个原地操作。

##### 为提供 JSON 列索引而对生成列建立索引

如其他地方所述，`JSON`列不能直接建立索引。要创建引用此类列的索引，可以定义一个生成列，提取应该建立索引的信息，然后在生成列上创建索引，如下例所示：

```sql
mysql> CREATE TABLE jemp (
 ->     c JSON,
 ->     g INT GENERATED ALWAYS AS (c->"$.id"),
 ->     INDEX i (g)
 -> );
Query OK, 0 rows affected (0.28 sec)

mysql> INSERT INTO jemp (c) VALUES
     >   ('{"id": "1", "name": "Fred"}'), ('{"id": "2", "name": "Wilma"}'),
     >   ('{"id": "3", "name": "Barney"}'), ('{"id": "4", "name": "Betty"}');
Query OK, 4 rows affected (0.04 sec)
Records: 4  Duplicates: 0  Warnings: 0

mysql> SELECT c->>"$.name" AS name
     >     FROM jemp WHERE g > 2;
+--------+
| name   |
+--------+
| Barney |
| Betty  |
+--------+
2 rows in set (0.00 sec)

mysql> EXPLAIN SELECT c->>"$.name" AS name
     >    FROM jemp WHERE g > 2\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: jemp
   partitions: NULL
         type: range
possible_keys: i
          key: i
      key_len: 5
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using where 1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select json_unquote(json_extract(`test`.`jemp`.`c`,'$.name'))
AS `name` from `test`.`jemp` where (`test`.`jemp`.`g` > 2) 1 row in set (0.00 sec)
```

（我们已经将此示例中最后一条语句的输出包装起来以适应查看区域。）

当您在包含使用`->`或`->>`运算符的一个或多个表达式的`SELECT`或其他 SQL 语句上使用`EXPLAIN`时，这些表达式将被转换为使用`JSON_EXTRACT()`和（如果需要）`JSON_UNQUOTE()`的等效形式，如下所示，在`EXPLAIN`语句后立即显示的`SHOW WARNINGS`输出中：

```sql
mysql> EXPLAIN SELECT c->>"$.name"
     > FROM jemp WHERE g > 2 ORDER BY c->"$.name"\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: jemp
   partitions: NULL
         type: range
possible_keys: i
          key: i
      key_len: 5
          ref: NULL
         rows: 2
     filtered: 100.00
        Extra: Using where; Using filesort 1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select json_unquote(json_extract(`test`.`jemp`.`c`,'$.name')) AS
`c->>"$.name"` from `test`.`jemp` where (`test`.`jemp`.`g` > 2) order by
json_extract(`test`.`jemp`.`c`,'$.name') 1 row in set (0.00 sec)
```

请参阅`->`和`->>`运算符的描述，以及`JSON_EXTRACT()`和`JSON_UNQUOTE()`函数的描述，获取更多信息和示例。

这种技术还可以用于提供间接引用其他类型列的索引，这些列不能直接进行索引，例如`GEOMETRY`列。

在 MySQL 8.0.21 及更高版本中，还可以使用`JSON_VALUE()`函数在`JSON`列上创建索引，使用可以优化查询的表达式。有关该函数的更多信息和示例，请参阅该函数的描述。

###### NDB Cluster 中的 JSON 列和间接索引

在 MySQL NDB Cluster 中，也可以使用 JSON 列的间接索引，但需要符合以下条件：

1.  `NDB`将`JSON`列值在内部处理为`BLOB`。这意味着任何具有一个或多个 JSON 列的 NDB 表必须具有主键，否则无法记录在二进制日志中。

1.  `NDB`存储引擎不支持虚拟列的索引。由于生成列的默认值是`VIRTUAL`，因此必须明确指定要应用间接索引的生成列为`STORED`。

用于创建此处显示的`jempn`表的**`CREATE TABLE`**语句是先前显示的`jemp`表的版本，经过修改以使其与`NDB`兼容：

```sql
CREATE TABLE jempn (
  a BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  c JSON DEFAULT NULL,
  g INT GENERATED ALWAYS AS (c->"$.id") STORED,
  INDEX i (g)
) ENGINE=NDB;
```

我们可以使用以下`INSERT`语句填充这个表：

```sql
INSERT INTO jempn (c) VALUES
  ('{"id": "1", "name": "Fred"}'),
  ('{"id": "2", "name": "Wilma"}'),
  ('{"id": "3", "name": "Barney"}'),
  ('{"id": "4", "name": "Betty"}');
```

现在`NDB`可以使用索引`i`，如下所示：

```sql
mysql> EXPLAIN SELECT c->>"$.name" AS name
 ->           FROM jempn WHERE g > 2\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: jempn
   partitions: p0,p1,p2,p3
         type: range
possible_keys: i
          key: i
      key_len: 5
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using pushed condition (`test`.`jempn`.`g` > 2) 1 row in set, 1 warning (0.01 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select
json_unquote(json_extract(`test`.`jempn`.`c`,'$.name')) AS `name` from
`test`.`jempn` where (`test`.`jempn`.`g` > 2) 1 row in set (0.00 sec)
```

你应该记住，存储的生成列以及该列上的任何索引都使用`DataMemory`。
