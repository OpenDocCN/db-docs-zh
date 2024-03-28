# 10.2.4 优化性能模式查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-optimization.html)

监视数据库的应用程序可能经常使用性能模式表。为了最有效地为这些表编写查询，请利用它们的索引。例如，包括一个`WHERE`子句，根据索引列中的特定值进行检索行的限制。

大多数性能模式表都有索引。不具有索引的表通常包含少量行或不太可能经常查询。性能模式索引使优化器可以访问除全表扫描之外的执行计划。这些索引还提高了相关对象的性能，例如使用这些表的`sys`模式视图。

要查看给定性能模式表是否具有索引以及它们是什么，请使用`SHOW INDEX`或`SHOW CREATE TABLE`：

```sql
mysql> SHOW INDEX FROM performance_schema.accounts\G
*************************** 1\. row ***************************
        Table: accounts
   Non_unique: 0
     Key_name: ACCOUNT
 Seq_in_index: 1
  Column_name: USER
    Collation: NULL
  Cardinality: NULL
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: HASH
      Comment:
Index_comment:
      Visible: YES
*************************** 2\. row ***************************
        Table: accounts
   Non_unique: 0
     Key_name: ACCOUNT
 Seq_in_index: 2
  Column_name: HOST
    Collation: NULL
  Cardinality: NULL
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: HASH
      Comment:
Index_comment:
      Visible: YES 
mysql> SHOW CREATE TABLE performance_schema.rwlock_instances\G
*************************** 1\. row ***************************
       Table: rwlock_instances
Create Table: CREATE TABLE `rwlock_instances` (
  `NAME` varchar(128) NOT NULL,
  `OBJECT_INSTANCE_BEGIN` bigint(20) unsigned NOT NULL,
  `WRITE_LOCKED_BY_THREAD_ID` bigint(20) unsigned DEFAULT NULL,
  `READ_LOCKED_BY_COUNT` int(10) unsigned NOT NULL,
  PRIMARY KEY (`OBJECT_INSTANCE_BEGIN`),
  KEY `NAME` (`NAME`),
  KEY `WRITE_LOCKED_BY_THREAD_ID` (`WRITE_LOCKED_BY_THREAD_ID`)
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

要查看性能模式查询的执行计划以及是否使用任何索引，请使用`EXPLAIN`：

```sql
mysql> EXPLAIN SELECT * FROM performance_schema.accounts
       WHERE (USER,HOST) = ('root','localhost')\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: accounts
   partitions: NULL
         type: const
possible_keys: ACCOUNT
          key: ACCOUNT
      key_len: 278
          ref: const,const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

`EXPLAIN`输出表明优化器使用包含`USER`和`HOST`列的`ACCOUNT`索引的`accounts`表。

性能模式索引是虚拟的：它们是性能模式存储引擎的构造，不使用内存或磁盘存储。性能模式向优化器报告索引信息，以便它可以构建高效的执行计划。性能模式反过来使用优化器关于要查找的信息（例如，特定键值），以便它可以执行高效的查找而不构建实际的索引结构。这种实现提供了两个重要的好处：

+   它完全避免了通常发生在经常更新的表上的维护成本。

+   它在查询执行的早期阶段减少了检索的数据量。对于索引列上的条件，性能模式有效地只返回满足查询条件的表行。没有索引，性能模式将返回表中的所有行，要求优化器稍后评估每行的条件以生成最终结果。

性能模式索引是预定义的，不能删除、添加或更改。

性能模式索引类似于哈希索引。例如：

+   它们仅用于使用`=`或`<=>`运算符进行相等比较。

+   它们是无序的。如果查询结果必须具有特定的行排序特性，请包含`ORDER BY`子句。

关于哈希索引的更多信息，请参见第 10.3.9 节，“B-Tree 和哈希索引的比较”。
