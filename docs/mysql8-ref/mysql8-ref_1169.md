> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-fulltext-index.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-fulltext-index.html)

#### 17.6.2.4 InnoDB 全文索引

全文索引是在基于文本的列（`CHAR`、`VARCHAR`或`TEXT`列）上创建的，以加快对这些列中包含的数据的查询和 DML 操作。

全文索引被定义为`CREATE TABLE`语句的一部分，或者通过`ALTER TABLE`或`CREATE INDEX`添加到现有表中。

使用`MATCH() ... AGAINST`语法执行全文搜索。有关使用信息，请参见第 14.9 节，“全文搜索函数”。

`InnoDB`全文索引在本节中以下主题下进行描述：

+   InnoDB 全文索引设计

+   InnoDB 全文索引表

+   InnoDB 全文索引缓存

+   InnoDB 全文索引 DOC_ID 和 FTS_DOC_ID 列

+   InnoDB 全文索引删除处理

+   InnoDB 全文索引事务处理

+   监控 InnoDB 全文索引

##### InnoDB 全文索引设计

`InnoDB`全文索引采用倒排索引设计。倒排索引存储单词列表，对于每个单词，还存储该单词出现在的文档列表。为了支持位置搜索，还存储了每个单词的位置信息，作为字节偏移量。

##### InnoDB 全文索引表

创建`InnoDB`全文索引时，将创建一组索引表，如下例所示：

```sql
mysql> CREATE TABLE opening_lines (
       id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200),
       FULLTEXT idx (opening_line)
       ) ENGINE=InnoDB;

mysql> SELECT table_id, name, space from INFORMATION_SCHEMA.INNODB_TABLES
       WHERE name LIKE 'test/%';
+----------+----------------------------------------------------+-------+
| table_id | name                                               | space |
+----------+----------------------------------------------------+-------+
|      333 | test/fts_0000000000000147_00000000000001c9_index_1 |   289 |
|      334 | test/fts_0000000000000147_00000000000001c9_index_2 |   290 |
|      335 | test/fts_0000000000000147_00000000000001c9_index_3 |   291 |
|      336 | test/fts_0000000000000147_00000000000001c9_index_4 |   292 |
|      337 | test/fts_0000000000000147_00000000000001c9_index_5 |   293 |
|      338 | test/fts_0000000000000147_00000000000001c9_index_6 |   294 |
|      330 | test/fts_0000000000000147_being_deleted            |   286 |
|      331 | test/fts_0000000000000147_being_deleted_cache      |   287 |
|      332 | test/fts_0000000000000147_config                   |   288 |
|      328 | test/fts_0000000000000147_deleted                  |   284 |
|      329 | test/fts_0000000000000147_deleted_cache            |   285 |
|      327 | test/opening_lines                                 |   283 |
+----------+----------------------------------------------------+-------+
```

前六个索引表包括倒排索引，并被称为辅助索引表。当传入文档被标记化时，单词（也称为“标记”）与位置信息和相关的`DOC_ID`一起插入到索引表中。这些单词根据单词的第一个字符的字符集排序权重完全排序并分区在六个索引表中。

倒排索引被分成六个辅助索引表，以支持并行索引创建。默认情况下，两个线程对单词和相关数据进行标记化、排序和插入到索引表中。执行此工作的线程数量可通过 `innodb_ft_sort_pll_degree` 变量进行配置。在创建大表的全文索引时，考虑增加线程数量。

辅助索引表名以 `fts_` 为前缀，并以 `index_*`#`*` 为后缀。每个辅助索引表通过辅助索引表名中的十六进制值与索引表相关联，该值与索引表的 `table_id` 匹配。例如，`test/opening_lines` 表的 `table_id` 是 `327`，其十六进制值为 0x147。如前面的示例所示，与 `test/opening_lines` 表相关联的辅助索引表的名称中出现了“147”十六进制值。

代表全文索引的 `index_id` 的十六进制值也出现在辅助索引表名中。例如，在辅助表名 `test/fts_0000000000000147_00000000000001c9_index_1` 中，十六进制值 `1c9` 的十进制值为 457。可以通过查询信息模式 `INNODB_INDEXES` 表来识别在 `opening_lines` 表上定义的索引（idx）的此值（457）。

```sql
mysql> SELECT index_id, name, table_id, space from INFORMATION_SCHEMA.INNODB_INDEXES
       WHERE index_id=457;
+----------+------+----------+-------+
| index_id | name | table_id | space |
+----------+------+----------+-------+
|      457 | idx  |      327 |   283 |
+----------+------+----------+-------+
```

如果主表是在 每表一个文件 表空间中创建的，则索引表存储在自己的表空间中。否则，索引表存储在索引表所在的表空间中。

前面示例中显示的其他索引表称为常见索引表，用于处理删除和存储全文索引的内部状态。与为每个全文索引创建的倒排索引表不同，这组表对于在特定表上创建的所有全文索引都是通用的。

即使删除全文索引，常见索引表也会保留。删除全文索引时，为索引创建的 `FTS_DOC_ID` 列将被保留，因为删除 `FTS_DOC_ID` 列将需要重建先前索引的表。常见索引表用于管理 `FTS_DOC_ID` 列。

+   `fts_*_deleted` 和 `fts_*_deleted_cache`

    包含已删除但数据尚未从全文索引中删除的文档的文档 ID（DOC_ID）。`fts_*_deleted_cache` 是 `fts_*_deleted` 表的内存版本。

+   `fts_*_being_deleted` 和 `fts_*_being_deleted_cache`

    包含已删除并且数据目前正在从全文索引中删除的文档的文档 ID（DOC_ID）。`fts_*_being_deleted_cache` 表是 `fts_*_being_deleted` 表的内存版本。

+   `fts_*_config`

    存储有关全文索引内部状态的信息。最重要的是，它存储了`FTS_SYNCED_DOC_ID`，用于标识已解析并刷新到磁盘的文档。在崩溃恢复的情况下，`FTS_SYNCED_DOC_ID` 值用于标识尚未刷新到磁盘的文档，以便重新解析这些文档并添加回全文索引缓存。要查看此表中的数据，请查询信息模式 `INNODB_FT_CONFIG` 表。

##### InnoDB 全文索引缓存

当插入文档时，文档会被标记化，单词和相关数据会被插入到全文索引中。即使是对于小型文档，这个过程也可能导致大量小的插入到辅助索引表中，使得对这些表的并发访问成为一个争议点。为了避免这个问题，`InnoDB` 使用全文索引缓存来临时缓存最近插入行的索引表插入。这个内存中的缓存结构会保存插入，直到缓存满了，然后批量将它们刷新到磁盘（到辅助索引表）。您可以查询信息模式 `INNODB_FT_INDEX_CACHE` 表来查看最近插入行的标记化数据。

缓存和批量刷新行为避免了对辅助索引表的频繁更新，这可能导致在繁忙的插入和更新时间发生并发访问问题。批处理技术还避免了对同一个单词的多次插入，并最小化了重复条目。与单独刷新每个单词不同，相同单词的插入会合并并作为单个条目刷新到磁盘，提高了插入效率，同时保持辅助索引表尽可能小。

`innodb_ft_cache_size` 变量用于配置全文索引缓存大小（每个表的基础），这会影响全文索引缓存刷新的频率。您还可以使用 `innodb_ft_total_cache_size` 变量为给定实例中的所有表定义一个全局全文索引缓存大小限制。

全文索引缓存存储与辅助索引表相同的信息。然而，全文索引缓存仅缓存最近插入行的标记化数据。当查询时，已经刷新到磁盘（到辅助索引表）的数据不会被带回到全文索引缓存中。辅助索引表中的数据直接查询，然后与全文索引缓存中的结果合并后返回。

##### InnoDB 全文索引 DOC_ID 和 FTS_DOC_ID 列

`InnoDB` 使用一个称为`DOC_ID`的唯一文档标识符，将全文索引中的单词映射到单词出现的文档记录上。这种映射需要在索引表上有一个`FTS_DOC_ID`列。如果没有定义`FTS_DOC_ID`列，`InnoDB`在创建全文索引时会自动添加一个隐藏的`FTS_DOC_ID`列。下面的示例演示了这种行为。

以下表定义不包括`FTS_DOC_ID`列：

```sql
mysql> CREATE TABLE opening_lines (
       id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200)
       ) ENGINE=InnoDB;
```

当使用`CREATE FULLTEXT INDEX`语法在表上创建全文索引时，会返回一个警告，报告`InnoDB`正在重建表以添加`FTS_DOC_ID`列。

```sql
mysql> CREATE FULLTEXT INDEX idx ON opening_lines(opening_line);
Query OK, 0 rows affected, 1 warning (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 1

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------+
| Level   | Code | Message                                          |
+---------+------+--------------------------------------------------+
| Warning |  124 | InnoDB rebuilding table to add column FTS_DOC_ID |
+---------+------+--------------------------------------------------+
```

当使用`ALTER TABLE`向没有`FTS_DOC_ID`列的表添加全文索引时，会返回相同的警告。如果在`CREATE TABLE`时创建全文索引并且没有指定`FTS_DOC_ID`列，则`InnoDB`会自动添加一个隐藏的`FTS_DOC_ID`列，而不会有警告。

在`CREATE TABLE`时定义`FTS_DOC_ID`列比在已加载数据的表上创建全文索引要便宜。如果在加载数据之前在表上定义了`FTS_DOC_ID`列，则不需要重建表及其索引即可添加新列。如果不关心`CREATE FULLTEXT INDEX`的性能，请省略`FTS_DOC_ID`列，让`InnoDB`为您创建。`InnoDB`会在`FTS_DOC_ID`列上创建一个隐藏的`FTS_DOC_ID`列以及一个唯一索引（`FTS_DOC_ID_INDEX`）。如果要创建自己的`FTS_DOC_ID`列，则该列必须定义为`BIGINT UNSIGNED NOT NULL`，并命名为`FTS_DOC_ID`（全大写），如下面的示例所示：

注意

`FTS_DOC_ID`列不需要定义为`AUTO_INCREMENT`列，但这样做可以使数据加载更容易。

```sql
mysql> CREATE TABLE opening_lines (
       FTS_DOC_ID BIGINT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200)
       ) ENGINE=InnoDB;
```

如果选择自己定义`FTS_DOC_ID`列，则需要负责管理该列，以避免空值或重复值。`FTS_DOC_ID`值不能被重复使用，这意味着`FTS_DOC_ID`值必须是递增的。

可选地，您可以在`FTS_DOC_ID`列上创建所需的唯一`FTS_DOC_ID_INDEX`（全大写）。

```sql
mysql> CREATE UNIQUE INDEX FTS_DOC_ID_INDEX on opening_lines(FTS_DOC_ID);
```

如果不创建`FTS_DOC_ID_INDEX`，`InnoDB`会自动创建它。

注意

由于`InnoDB` SQL 解析器不使用降序索引，因此无法将`FTS_DOC_ID_INDEX`定义为降序索引。

最大使用的`FTS_DOC_ID`值和新的`FTS_DOC_ID`值之间允许的间隔为 65535。

为避免重建表，删除全文索引时会保留`FTS_DOC_ID`列。

##### InnoDB 全文索引删除处理

删除具有全文索引列的记录可能导致辅助索引表中的大量小删除，使得对这些表的并发访问成为争议点。为了避免这个问题，每当从索引表中删除记录时，删除文档的`DOC_ID`会被记录在特殊的`FTS_*_DELETED`表中，并且索引记录仍然保留在全文索引中。在返回查询结果之前，`FTS_*_DELETED`表中的信息用于过滤已删除的`DOC_ID`。这种设计的好处是删除快速且廉价。缺点是删除记录后索引的大小不会立即减小。要删除已删除记录的全文索引条目，请在具有`innodb_optimize_fulltext_only=ON`的索引表上运行`OPTIMIZE TABLE`以重建全文索引。有关更多信息，请参阅优化 InnoDB 全文索引。

##### InnoDB 全文索引事务处理

`InnoDB`全文索引由于其缓存和批处理行为具有特殊的事务处理特性。具体来说，在事务提交时处理全文索引的更新和插入操作，这意味着全文搜索只能看到已提交的数据。以下示例演示了这种行为。只有在插入的行被提交后，全文搜索才会返回结果。

```sql
mysql> CREATE TABLE opening_lines (
       id INT UNSIGNED AUTO_INCREMENT NOT NULL PRIMARY KEY,
       opening_line TEXT(500),
       author VARCHAR(200),
       title VARCHAR(200),
       FULLTEXT idx (opening_line)
       ) ENGINE=InnoDB;

mysql> BEGIN;

mysql> INSERT INTO opening_lines(opening_line,author,title) VALUES
       ('Call me Ishmael.','Herman Melville','Moby-Dick'),
       ('A screaming comes across the sky.','Thomas Pynchon','Gravity\'s Rainbow'),
       ('I am an invisible man.','Ralph Ellison','Invisible Man'),
       ('Where now? Who now? When now?','Samuel Beckett','The Unnamable'),
       ('It was love at first sight.','Joseph Heller','Catch-22'),
       ('All this happened, more or less.','Kurt Vonnegut','Slaughterhouse-Five'),
       ('Mrs. Dalloway said she would buy the flowers herself.','Virginia Woolf','Mrs. Dalloway'),
       ('It was a pleasure to burn.','Ray Bradbury','Fahrenheit 451');

mysql> SELECT COUNT(*) FROM opening_lines WHERE MATCH(opening_line) AGAINST('Ishmael');
+----------+
| COUNT(*) |
+----------+
|        0 |
+----------+

mysql> COMMIT;

mysql> SELECT COUNT(*) FROM opening_lines WHERE MATCH(opening_line) AGAINST('Ishmael');
+----------+
| COUNT(*) |
+----------+
|        1 |
+----------+
```

##### 监控 InnoDB 全文索引

您可以通过查询以下`INFORMATION_SCHEMA`表来监视和检查`InnoDB`全文索引的特殊文本处理方面：

+   `INNODB_FT_CONFIG`

+   `INNODB_FT_INDEX_TABLE`

+   `INNODB_FT_INDEX_CACHE`

+   `INNODB_FT_DEFAULT_STOPWORD`

+   `INNODB_FT_DELETED`

+   `INNODB_FT_BEING_DELETED`

通过查询`INNODB_INDEXES`和`INNODB_TABLES`，您还可以查看全文索引和表的基本信息。

欲了解更多信息，请参阅第 17.15.4 节，“InnoDB INFORMATION_SCHEMA FULLTEXT Index Tables”。
