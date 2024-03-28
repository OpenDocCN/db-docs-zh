# 10.2.3 优化 INFORMATION_SCHEMA 查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-optimization.html)

监视数据库的应用程序可能经常使用`INFORMATION_SCHEMA`表。为这些表编写查询最有效的方法是使用以下一般准则：

+   尽量只查询`INFORMATION_SCHEMA`表中作为数据字典表视图的表。

+   尽量只查询静态元数据。选择列或使用检索条件来获取动态元数据会增加处理动态元数据的开销。

注意

在`INFORMATION_SCHEMA`查询中，数据库和表名的比较行为可能与您期望的不同。详情请参见 Section 12.8.7, “Using Collation in INFORMATION_SCHEMA Searches”。

这些`INFORMATION_SCHEMA`表实际上是作为数据字典表的视图实现的，因此对它们的查询会从数据字典中检索信息：

```sql
CHARACTER_SETS
CHECK_CONSTRAINTS
COLLATIONS
COLLATION_CHARACTER_SET_APPLICABILITY
COLUMNS
EVENTS
FILES
INNODB_COLUMNS
INNODB_DATAFILES
INNODB_FIELDS
INNODB_FOREIGN
INNODB_FOREIGN_COLS
INNODB_INDEXES
INNODB_TABLES
INNODB_TABLESPACES
INNODB_TABLESPACES_BRIEF
INNODB_TABLESTATS
KEY_COLUMN_USAGE
PARAMETERS
PARTITIONS
REFERENTIAL_CONSTRAINTS
RESOURCE_GROUPS
ROUTINES
SCHEMATA
STATISTICS
TABLES
TABLE_CONSTRAINTS
TRIGGERS
VIEWS
VIEW_ROUTINE_USAGE
VIEW_TABLE_USAGE
```

一些类型的值，即使是非视图的`INFORMATION_SCHEMA`表，也是通过从数据字典中查找来检索的。这包括数据库和表名、表类型和存储引擎等值。

一些`INFORMATION_SCHEMA`表包含提供表统计信息的列：

```sql
STATISTICS.CARDINALITY
TABLES.AUTO_INCREMENT
TABLES.AVG_ROW_LENGTH
TABLES.CHECKSUM
TABLES.CHECK_TIME
TABLES.CREATE_TIME
TABLES.DATA_FREE
TABLES.DATA_LENGTH
TABLES.INDEX_LENGTH
TABLES.MAX_DATA_LENGTH
TABLES.TABLE_ROWS
TABLES.UPDATE_TIME
```

这些列代表动态表元数据；即，随着表内容变化而变化的信息。

默认情况下，MySQL 在查询这些列时从`mysql.index_stats`和`mysql.innodb_table_stats`数据字典表中检索缓存值，这比直接从存储引擎检索统计信息更有效。如果缓存的统计信息不可用或已过期，MySQL 会从存储引擎中检索最新的统计信息，并将其缓存在`mysql.index_stats`和`mysql.innodb_table_stats`数据字典表中。随后的查询将检索缓存的统计信息，直到缓存的统计信息过期。服务器重新启动或首次打开`mysql.index_stats`和`mysql.innodb_table_stats`表不会自动更新缓存的统计信息。

`information_schema_stats_expiry`会话变量定义了缓存统计信息过期之前的时间段。默认值为 86400 秒（24 小时），但时间段可以延长至一年。

要随时更新给定表的缓存值，请使用`ANALYZE TABLE`。

在以下情况下，查询统计列不会在`mysql.index_stats`和`mysql.innodb_table_stats`数据字典表中存储或更新统计信息：

+   当缓存的统计信息尚未过期时。

+   当`information_schema_stats_expiry`设置为 0 时。

+   当服务器处于`read_only`、`super_read_only`、`transaction_read_only`或`innodb_read_only`模式时。

+   当查询还检索性能模式数据时。

`information_schema_stats_expiry`是一个会话变量，每个客户端会话可以定义自己的过期值。一个会话检索并缓存的统计信息对其他会话可用。

注意

如果启用了`innodb_read_only`系统变量，则`分析表`可能会失败，因为它无法更新数据字典中使用`InnoDB`的统计表。对于更新键分布的`分析表`操作，即使操作更新表本身（例如，如果它是`MyISAM`表），也可能发生失败。要获取更新后的分布统计信息，请设置`information_schema_stats_expiry=0`。

对于在数据字典表上实现为视图的`INFORMATION_SCHEMA`表，基础数据字典表上的索引允许优化器构建高效的查询执行计划。要查看优化器所做的选择，请使用`解释`。要还查看服务器用于执行`INFORMATION_SCHEMA`查询的查询，请在`解释`之后立即使用`显示警告`。

考虑以下语句，用于识别`utf8mb4`字符集的排序规则：

```sql
mysql> SELECT COLLATION_NAME
       FROM INFORMATION_SCHEMA.COLLATION_CHARACTER_SET_APPLICABILITY
       WHERE CHARACTER_SET_NAME = 'utf8mb4';
+----------------------------+
| COLLATION_NAME             |
+----------------------------+
| utf8mb4_general_ci         |
| utf8mb4_bin                |
| utf8mb4_unicode_ci         |
| utf8mb4_icelandic_ci       |
| utf8mb4_latvian_ci         |
| utf8mb4_romanian_ci        |
| utf8mb4_slovenian_ci       |
...
```

服务器如何处理该语句？要找出，请使用`解释`：

```sql
mysql> EXPLAIN SELECT COLLATION_NAME
       FROM INFORMATION_SCHEMA.COLLATION_CHARACTER_SET_APPLICABILITY
       WHERE CHARACTER_SET_NAME = 'utf8mb4'\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: cs
   partitions: NULL
         type: const
possible_keys: PRIMARY,name
          key: name
      key_len: 194
          ref: const
         rows: 1
     filtered: 100.00
        Extra: Using index
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: col
   partitions: NULL
         type: ref
possible_keys: character_set_id
          key: character_set_id
      key_len: 8
          ref: const
         rows: 68
     filtered: 100.00
        Extra: NULL 2 rows in set, 1 warning (0.01 sec)
```

要查看用于满足该语句的查询，请使用`显示警告`：

```sql
mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `mysql`.`col`.`name` AS `COLLATION_NAME`
         from `mysql`.`character_sets` `cs`
         join `mysql`.`collations` `col`
         where ((`mysql`.`col`.`character_set_id` = '45')
         and ('utf8mb4' = 'utf8mb4'))
```

如`显示警告`所示，服务器将处理对`COLLATION_CHARACTER_SET_APPLICABILITY`的查询，就像对`mysql`系统数据库中的`character_sets`和`collations`数据字典表的查询一样。
