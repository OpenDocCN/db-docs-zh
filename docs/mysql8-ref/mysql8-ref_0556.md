# 10.3.5 列索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/column-indexes.html`](https://dev.mysql.com/doc/refman/8.0/en/column-indexes.html)

最常见的索引类型涉及单个列，在数据结构中存储该列的值的副本，允许快速查找具有相应列值的行。B 树数据结构让索引可以快速找到特定值、一组值或一系列值，对应于`=`、`>`、`≤`、`BETWEEN`、`IN`等操作符，在`WHERE`子句中使用。

每个表的最大索引数和最大索引长度由存储引擎定义。请参见第十七章，“InnoDB 存储引擎”和第十八章，“替代存储引擎”。所有存储引擎至少支持每个表 16 个索引和至少 256 字节的总索引长度。大多数存储引擎具有更高的限制。

有关列索引的更多信息，请参见第 15.1.15 节，“CREATE INDEX 语句”。

+   索引前缀

+   全文索引

+   空间索引

+   MEMORY 存储引擎中的索引

#### 索引前缀

使用`*`col_name`*(*`N`*)`语法在字符串列的索引规范中，您可以创建一个仅使用列的前 *`N`* 个字符的索引。以这种方式仅对列值的前缀进行索引可以使索引文件变得更小。当您对`BLOB`或`TEXT`列进行索引时，*必须*为索引指定前缀长度。例如：

```sql
CREATE TABLE test (blob_col BLOB, INDEX(blob_col(10)));
```

对于使用`REDUNDANT`或`COMPACT`行格式的`InnoDB`表，前缀最长可达 767 字节。对于使用`DYNAMIC`或`COMPRESSED`行格式的`InnoDB`表，前缀长度限制为 3072 字节。对于 MyISAM 表，前缀长度限制为 1000 字节。

注意

前缀限制以字节为单位，而在`CREATE TABLE`、`ALTER TABLE`和`CREATE INDEX`语句中，前缀长度被解释为非二进制字符串类型（`CHAR`、`VARCHAR`、`TEXT`）的字符数和二进制字符串类型的字节数（`BINARY`、`VARBINARY`、`BLOB`）。在为使用多字节字符集的非二进制字符串列指定前缀长度时，请考虑这一点。

如果搜索词超过索引前缀长度，则使用索引来排除不匹配的行，然后检查剩余行以查找可能的匹配项。

有关索引前缀的更多信息，请参见 Section 15.1.15, “CREATE INDEX Statement”。

#### 全文索引

`FULLTEXT`索引用于全文搜索。只有`InnoDB`和`MyISAM`存储引擎支持`FULLTEXT`索引，且仅适用于`CHAR`、`VARCHAR`和`TEXT`列。索引始终在整个列上进行，不支持列前缀索引。有关详细信息，请参见 Section 14.9, “Full-Text Search Functions”。

优化应用于针对单个`InnoDB`表的某些类型的`FULLTEXT`查询。具有以下特征的查询特别高效：

+   仅返回文档 ID 或文档 ID 和搜索排名的`FULLTEXT`查询。

+   将匹配行按得分降序排序并应用`LIMIT`子句以获取前 N 个匹配行的`FULLTEXT`查询。要应用此优化，`WHERE`子句中不能有`WHERE`子句，且仅有一个按降序排序的`ORDER BY`子句。

+   仅检索与搜索词匹配的行的`COUNT(*)`值的`FULLTEXT`查询，没有额外的`WHERE`子句。将`WHERE`子句编码为`WHERE MATCH(*`text`*) AGAINST ('*`other_text`*')`，不包含任何`> 0`比较运算符。

对于包含全文表达式的查询，MySQL 在查询执行的优化阶段评估这些表达式。优化器不仅查看全文表达式并进行估计，而且在制定执行计划的过程中实际评估它们。

这种行为的一个影响是，对于全文查询，`EXPLAIN` 通常比对于在优化阶段不进行表达式评估的非全文查询慢。

对于全文查询，`EXPLAIN` 可能会在 `Extra` 列中显示 `选择表已优化`，这是由于匹配发生在优化期间；在这种情况下，后续执行时不需要访问表。

#### 空间索引

您可以在空间数据类型上创建索引。`MyISAM` 和 `InnoDB` 支持空间类型的 R 树索引。其他存储引擎使用 B 树来索引空间类型（除了 `ARCHIVE` 不支持空间类型索引）。

#### 内存存储引擎中的索引

`MEMORY` 存储引擎默认使用 `HASH` 索引，但也支持 `BTREE` 索引。
