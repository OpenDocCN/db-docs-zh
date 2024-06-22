# 1\. 外键约束介绍

> 原文：[`sqlite.com/foreignkeys.html`](https://sqlite.com/foreignkeys.html)

## 概述

本文描述了 SQLite 中引入的 SQL 外键约束的支持 版本 3.6.19（2009-10-14）。

第一部分通过示例介绍了 SQL 外键的概念，并定义了文档其余部分使用的术语。第二部分描述了应用程序必须执行的步骤，以便在 SQLite 中启用外键约束（默认情况下是禁用的）。接下来的第三部分描述了用户必须创建的索引，以便使用外键约束，并描述了应为了使外键约束有效而创建的索引。第四部分描述了 SQLite 支持的高级外键相关功能，第五部分描述了增强以支持外键约束的 ALTER 和 DROP TABLE 命令的方式。最后，第六部分列举了当前实现的缺失功能和限制。

本文不包含有关在 SQLite 中创建外键约束所使用的语法的完整描述。此可在 CREATE TABLE 语句的文档中找到。

SQL 外键约束用于在表之间强制“存在”关系。例如，考虑使用以下 SQL 命令创建的数据库模式：

```sql
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
CREATE TABLE track(
  trackid     INTEGER,
  trackname   TEXT, 
  trackartist INTEGER     -- Must map to an artist.artistid!
);

```

使用此数据库的应用程序有权假定，在 *track* 表中的每一行都存在对应的 *artist* 表中的一行。毕竟，声明中的注释就这么说的。不幸的是，如果用户使用外部工具编辑数据库，或者应用程序中存在错误，可能会插入到 *track* 表中不对应于 *artist* 表中任何行的行。或者，从 *artist* 表中删除行，使 *track* 表中存在孤立行，这些行不对应于 *artist* 中剩余行的任何行。这可能会导致应用程序或稍后出现故障，或者至少使编写应用程序更加困难。

一种解决方案是在数据库模式中添加 SQL 外键约束，以强制 *artist* 表和 *track* 表之间的关系。为此，可以通过修改 *track* 表的声明来添加外键定义，如下所示：

```sql
CREATE TABLE track(
  trackid     INTEGER, 
  trackname   TEXT, 
  trackartist INTEGER,
  FOREIGN KEY(trackartist) REFERENCES artist(artistid)
);

```

这样，SQLite 强制执行约束。尝试向 *track* 表中插入不对应于 *artist* 表中任何行的行将失败，同样，尝试在 *artist* 表中存在依赖行时删除行也将失败。有一个例外：如果 *track* 表中的外键列为 NULL，则不需要 *artist* 表中的对应条目。用 SQL 表达，这意味着对于 *track* 表中的每一行，以下表达式都为真：

```sql
trackartist IS NULL OR EXISTS(SELECT 1 FROM artist WHERE artistid=trackartist)

```

提示：如果应用程序要求*艺术家*和*曲目*之间有更严格的关系，不允许在*trackartist*列中出现 NULL 值，只需在架构中添加适当的“NOT NULL”约束即可。

还有其他几种方法可以向 CREATE TABLE 语句添加等效的外键声明。有关详细信息，请参阅 CREATE TABLE 文档。

下面的 SQLite 命令行会话说明了添加到*track*表的外键约束的效果：

```sql
sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
1         Dean Martin      
2         Frank Sinatra    

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
11       That's Amore       1  
12       Christmas Blues    1  
13       My Way             2  

sqlite> *-- This fails because the value inserted into the trackartist column (3)*
sqlite> *-- does not correspond to row in the artist table.*
sqlite> INSERT INTO track VALUES(14, 'Mr. Bojangles', 3);
SQL error: foreign key constraint failed

sqlite> *-- This succeeds because a NULL is inserted into trackartist. A*
sqlite> *-- corresponding row in the artist table is not required in this case.*
sqlite> INSERT INTO track VALUES(14, 'Mr. Bojangles', NULL);

sqlite> *-- Trying to modify the trackartist field of the record after it has* 
sqlite> *-- been inserted does not work either, since the new value of trackartist (3)*
sqlite> *-- Still does not correspond to any row in the artist table.*
sqlite> UPDATE track SET trackartist = 3 WHERE trackname = 'Mr. Bojangles';
SQL error: foreign key constraint failed

sqlite> *-- Insert the required row into the artist table. It is then possible to*
sqlite> *-- update the inserted row to set trackartist to 3 (since a corresponding*
sqlite> *-- row in the artist table now exists).*
sqlite> INSERT INTO artist VALUES(3, 'Sammy Davis Jr.');
sqlite> UPDATE track SET trackartist = 3 WHERE trackname = 'Mr. Bojangles';

sqlite> *-- Now that "Sammy Davis Jr." (artistid = 3) has been added to the database,*
sqlite> *-- it is possible to INSERT new tracks using this artist without violating*
sqlite> *-- the foreign key constraint:*
sqlite> INSERT INTO track VALUES(15, 'Boogie Woogie', 3);

```

正如您所预期的那样，通过删除或更新*artist*表中的行来操纵数据库，使其违反外键约束状态是不可能的：

```sql
sqlite> *-- Attempting to delete the artist record for "Frank Sinatra" fails, since*
sqlite> *-- the track table contains a row that refer to it.*
sqlite> DELETE FROM artist WHERE artistname = 'Frank Sinatra';
SQL error: foreign key constraint failed

sqlite> *-- Delete all the records from the track table that refer to the artist*
sqlite> *-- "Frank Sinatra". Only then is it possible to delete the artist.*
sqlite> DELETE FROM track WHERE trackname = 'My Way';
sqlite> DELETE FROM artist WHERE artistname = 'Frank Sinatra';

sqlite> *-- Try to update the artistid of a row in the artist table while there*
sqlite> *-- exists records in the track table that refer to it.* 
sqlite> UPDATE artist SET artistid=4 WHERE artistname = 'Dean Martin';
SQL error: foreign key constraint failed

sqlite> *-- Once all the records that refer to a row in the artist table have*
sqlite> *-- been deleted, it is possible to modify the artistid of the row.*
sqlite> DELETE FROM track WHERE trackname IN('That''s Amore', 'Christmas Blues');
sqlite> UPDATE artist SET artistid=4 WHERE artistname = 'Dean Martin';

```

SQLite 使用以下术语：

+   **父表**是外键约束所引用的表。本节示例中的父表是*artist*表。一些书籍和文章将其称为*被引用表*，这可能更加正确，但往往会导致混淆。

+   **子表**是应用外键约束的表，也是包含 REFERENCES 子句的表。本节示例中使用*track*表作为子表。其他书籍和文章将其称为*引用表*。

+   **父键**是外键约束所引用的父表中的列或列集。这通常是父表的主键，但并非总是如此。父键必须是父表中的命名列或列，而不是 rowid。

+   **子键**是受外键约束限制的子表中的列或列集，并保存 REFERENCES 子句。

外键约束在子表中的每一行，要么一个或多个子键列为空，要么存在父表中的一行，其中每个父键列包含与其关联的子键列中的值相等的值。

在上述段落中，“相等”的术语意味着在使用此处指定的规则进行比较时相等。以下澄清适用：

+   在比较文本值时，始终使用与父键列关联的排序序列。

+   在比较值时，如果父键列具有亲和性，则在执行比较之前将该亲和性应用于子键值。

# 2\. 启用外键支持

要在 SQLite 中使用外键约束，库必须未定义 SQLITE_OMIT_FOREIGN_KEY 和 SQLITE_OMIT_TRIGGER。如果定义了 SQLITE_OMIT_TRIGGER 但未定义 SQLITE_OMIT_FOREIGN_KEY，则 SQLite 的行为与版本 3.6.19（2009-10-14）之前的版本相同 - 解析外键定义，并可以使用 PRAGMA foreign_key_list 进行查询，但不强制执行外键约束。在此配置中，PRAGMA foreign_keys 命令是无效的。如果定义了 OMIT_FOREIGN_KEY，则甚至不能解析外键定义（尝试指定外键定义将导致语法错误）。

假设库已经编译启用了外键约束，但仍需在运行时由应用程序使用 PRAGMA foreign_keys 命令启用它。例如：

```sql
sqlite> PRAGMA foreign_keys = ON;

```

外键约束默认情况下处于禁用状态（为了向后兼容性），因此必须为每个数据库连接单独启用它们。（然而，请注意，SQLite 的未来版本可能会更改，以使外键约束默认启用。小心的开发人员不会假设外键默认启用或禁用，而是根据需要启用或禁用它们。）应用程序还可以使用 PRAGMA foreign_keys 语句来确定当前是否已启用外键。以下命令行会话演示了这一点：

```sql
sqlite> PRAGMA foreign_keys;
0
sqlite> PRAGMA foreign_keys = ON;
sqlite> PRAGMA foreign_keys;
1
sqlite> PRAGMA foreign_keys = OFF;
sqlite> PRAGMA foreign_keys;
0

```

提示：如果命令"PRAGMA foreign_keys"返回的不是包含"0"或"1"的单行数据，那么您正在使用的 SQLite 版本不支持外键（可能是因为版本低于 3.6.19 或编译时定义了 SQLITE_OMIT_FOREIGN_KEY 或 SQLITE_OMIT_TRIGGER）。

不可能在多语句事务（SQLite 不处于自动提交模式时）中间启用或禁用外键约束。尝试这样做不会返回错误；它只是无效果。

# 3\. 必需和建议的数据库索引

通常情况下，外键约束的父键是父表的主键。如果它们不是主键，则父键列必须共同受到唯一约束或具有唯一索引的约束。如果父键列具有唯一索引，则该索引必须使用在父表的 CREATE TABLE 语句中指定的排序序列。例如，

```sql
CREATE TABLE parent(a PRIMARY KEY, b UNIQUE, c, d, e, f);
CREATE UNIQUE INDEX i1 ON parent(c, d);
CREATE INDEX i2 ON parent(e);
CREATE UNIQUE INDEX i3 ON parent(f COLLATE nocase);

CREATE TABLE child1(f, g REFERENCES parent(a));                        *-- Ok*
CREATE TABLE child2(h, i REFERENCES parent(b));                        *-- Ok*
CREATE TABLE child3(j, k, FOREIGN KEY(j, k) REFERENCES parent(c, d));  *-- Ok*
CREATE TABLE child4(l, m REFERENCES parent(e));                        *-- Error!*
CREATE TABLE child5(n, o REFERENCES parent(f));                        *-- Error!*
CREATE TABLE child6(p, q, FOREIGN KEY(p, q) REFERENCES parent(b, c));  *-- Error!*
CREATE TABLE child7(r REFERENCES parent(c));                           *-- Error!*

```

表 *child1*、*child2* 和 *child3* 中作为表的一部分创建的外键约束都是正确的。表 *child4* 中作为表的一部分创建的外键是错误的，因为即使父键列被索引，该索引也不是唯一的。表 *child5* 中的外键是错误的，因为即使父键列有唯一索引，该索引使用了不同的排序序列。表 *child6* 和 *child7* 是不正确的，因为虽然两者在其父键上有唯一索引，但这些键不完全匹配单个唯一索引的列。

如果数据库模式包含外键错误，需要查看超过一个表定义才能识别这些错误，则在创建表时不会检测到这些错误。相反，这些错误会阻止应用程序准备使用外键修改子或父表内容的 SQL 语句。在更改内容时报告的错误是“DML 错误”，在更改模式时报告的错误是“DDL 错误”。换句话说，需要查看子表和父表的配置错误外键约束是 DML 错误。外键 DML 错误的英语语言错误消息通常是“外键不匹配”，但如果父表不存在，则也可以是“没有这样的表”。如果：

+   父表不存在，或者

+   在外键约束中命名的父键列不存在，或者

+   在外键约束中命名的父键列不是父表的主键，并且不受使用在 CREATE TABLE 中指定的排序序列的唯一约束的约束，或者

+   子表引用父表的主键，但未指定主键列，并且父表中的主键列数与子键列数不匹配。

上述最后一个项目如下所示：

```sql
CREATE TABLE parent2(a, b, PRIMARY KEY(a,b));

CREATE TABLE child8(x, y, FOREIGN KEY(x,y) REFERENCES parent2);        *-- Ok*
CREATE TABLE child9(x REFERENCES parent2);                             *-- Error!*
CREATE TABLE child10(x,y,z, FOREIGN KEY(x,y,z) REFERENCES parent2);    *-- Error!*

```

相比之下，如果外键错误可以通过简单查看子表的定义而不必查看父表定义就可以识别，则子表的 CREATE TABLE 语句失败。因为错误发生在模式更改期间，这是 DDL 错误。无论在创建表时是否启用外键约束，都会报告外键 DDL 错误。

索引不是必需的子键列，但几乎总是有益的。回到 第一部分 中的示例，每当应用程序从 *artist* 表（父表）中删除一行时，它执行以下 SELECT 语句来搜索 *track* 表（子表）中的引用行的等效内容。

```sql
SELECT rowid FROM track WHERE trackartist = ?

```

上述的“？”被替换为从 *artist* 表中删除的记录的 *artistid* 列的值（请记住 *trackartist* 列是子键，*artistid* 列是父键）。或者更一般地说：

```sql
SELECT rowid FROM <child-table> WHERE <child-key> = :parent_key_value

```

如果此 SELECT 返回任何行，则 SQLite 认为从父表中删除行将违反外键约束并返回错误。如果修改了父键的内容或在父表中插入了新行，则可能运行类似的查询。如果这些查询不能使用索引，则被迫对整个子表进行线性扫描。在一个非平凡的数据库中，这可能成本太高。

因此，在大多数实际系统中，应在每个外键约束的子键列上创建索引。子键索引不必是（通常也不会是）唯一索引。再次回到第一部分中的示例，为了有效实现外键约束，完整的数据库模式可能是：

```sql
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
CREATE TABLE track(
  trackid     INTEGER,
  trackname   TEXT, 
  trackartist INTEGER REFERENCES artist
);
CREATE INDEX trackindex ON track(trackartist);

```

上面的代码块使用了一种简写形式来创建外键约束。在列定义中附加一个"REFERENCES *<parent-table>*"子句将创建一个将该列映射到*<parent-table>*主键的外键约束。更多详细信息请参阅 CREATE TABLE 文档。

# 4\. 高级外键约束特性

## 4.1\. 复合外键约束

复合外键约束是指子键和父键都是复合键的约束。例如，考虑以下数据库模式：

```sql
CREATE TABLE album(
  albumartist TEXT,
  albumname TEXT,
  albumcover BINARY,
  PRIMARY KEY(albumartist, albumname)
);

CREATE TABLE song(
  songid     INTEGER,
  songartist TEXT,
  songalbum TEXT,
  songname   TEXT,
  FOREIGN KEY(songartist, songalbum) REFERENCES album(albumartist, albumname)
);

```

在这个系统中，歌曲表中的每个条目都必须映射到具有相同艺术家和专辑组合的专辑表中的条目。

父键和子键必须具有相同的基数。在 SQLite 中，如果任何子键列（在本例中为 songartist 和 songalbum）为空，则不需要在父表中有相应的行。

## 4.2\. 延迟外键约束

每个 SQLite 中的外键约束都被分为立即生效和延迟生效。外键约束默认为立即生效。到目前为止，所展示的所有外键示例都是立即生效的外键约束。

如果一条语句修改了数据库内容，导致在语句结束时立即外键约束被违反，则会抛出异常并回滚语句的效果。相反，如果一条语句修改了数据库内容，导致延迟外键约束被违反，则不会立即报告违反情况。延迟外键约束直到事务尝试 COMMIT 才会检查。只要用户有一个打开的事务，数据库允许存在违反任意数量的延迟外键约束的状态。但是，只要外键约束仍然违反，COMMIT 将会失败。

如果当前语句不在显式事务（BEGIN/COMMIT/ROLLBACK 块）中，则隐式事务在语句执行完毕后立即提交。在这种情况下，延迟约束的行为与即时约束相同。

要将外键约束标记为延迟，其声明必须包括以下子句：

```sql
DEFERRABLE INITIALLY DEFERRED                *-- A deferred foreign key constraint*

```

指定外键约束的完整语法作为 CREATE TABLE 文档的一部分提供。用以下任何短语替换上述短语将创建一个即时外键约束。

```sql
NOT DEFERRABLE INITIALLY DEFERRED            *-- An immediate foreign key constraint*
NOT DEFERRABLE INITIALLY IMMEDIATE           *-- An immediate foreign key constraint*
NOT DEFERRABLE                               *-- An immediate foreign key constraint*
DEFERRABLE INITIALLY IMMEDIATE               *-- An immediate foreign key constraint*
DEFERRABLE                                   *-- An immediate foreign key constraint*

```

defer_foreign_keys pragma 可用于临时更改所有外键约束为延迟约束，而不管它们的声明方式如何。

以下示例说明了使用延迟外键约束的效果。

```sql
*-- Database schema. Both tables are initially empty.* 
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
CREATE TABLE track(
  trackid     INTEGER,
  trackname   TEXT, 
  trackartist INTEGER REFERENCES artist(artistid) DEFERRABLE INITIALLY DEFERRED
);

sqlite3> *-- If the foreign key constraint were immediate, this INSERT would*
sqlite3> *-- cause an error (since as there is no row in table artist with*
sqlite3> *-- artistid=5). But as the constraint is deferred and there is an*
sqlite3> *-- open transaction, no error occurs.*
sqlite3> BEGIN;
sqlite3>   INSERT INTO track VALUES(1, 'White Christmas', 5);

sqlite3> *-- The following COMMIT fails, as the database is in a state that*
sqlite3> *-- does not satisfy the deferred foreign key constraint. The*
sqlite3> *-- transaction remains open.*
sqlite3> COMMIT;
SQL error: foreign key constraint failed

sqlite3> *-- After inserting a row into the artist table with artistid=5, the*
sqlite3> *-- deferred foreign key constraint is satisfied. It is then possible*
sqlite3> *-- to commit the transaction without error.*
sqlite3>   INSERT INTO artist VALUES(5, 'Bing Crosby');
sqlite3> COMMIT;

```

可以在数据库不满足延迟外键约束的状态下 RELEASE nested savepoint 事务。然而，事务保存点（在当前没有打开事务的情况下打开的非嵌套保存点）受到与 COMMIT 相同的限制 - 在数据库处于这种状态时尝试 RELEASE 它将失败。

如果 COMMIT 语句（或事务 SAVEPOINT 的 RELEASE）因为数据库当前处于违反延迟外键约束的状态而失败，并且当前存在 nested savepoints，则嵌套保存点保持打开状态。

## 4.3\. ON DELETE 和 ON UPDATE 操作

外键的 ON DELETE 和 ON UPDATE 子句用于配置删除父表行时的操作（ON DELETE），或修改现有行的父键值时的操作（ON UPDATE）。单个外键约束可以为 ON DELETE 和 ON UPDATE 配置不同的操作。外键操作在许多方面类似于触发器。

在 SQLite 数据库中，每个外键的 ON DELETE 和 ON UPDATE 操作可以是 "NO ACTION"、"RESTRICT"、"SET NULL"、"SET DEFAULT" 或 "CASCADE" 中的一个。如果未明确指定操作，则默认为 "NO ACTION"。

+   **NO ACTION**: 配置为 "NO ACTION" 意味着：当数据库中的父键被修改或删除时，不会执行任何特殊操作。

+   **RESTRICT**: “RESTRICT”操作意味着当存在一个或多个映射到其上的子键时，应用程序被禁止删除（对于 ON DELETE RESTRICT）或修改（对于 ON UPDATE RESTRICT）父键。 RESTRICT 操作与正常的外键约束执行的区别在于，RESTRICT 操作处理发生在更新字段时 - 而不是像即时约束一样在当前语句结束时，或者像延迟约束一样在当前事务结束时。即使外键约束是延迟的，配置 RESTRICT 操作也会导致 SQLite 在删除或修改具有依赖子键的父键时立即返回错误。

+   **SET NULL**: 如果配置的操作是“SET NULL”，那么当删除父键（对于 ON DELETE SET NULL）或修改父键（对于 ON UPDATE SET NULL）时，子表中映射到父键的所有行的子键列将设置为 SQL NULL 值。

+   **SET DEFAULT**: “SET DEFAULT”操作与“SET NULL”类似，不同之处在于每个子键列设置为包含列的默认值而不是 NULL。有关如何为表列分配默认值的详细信息，请参阅 CREATE TABLE 文档。

+   **CASCADE**: “CASCADE”操作将删除或更新父键的操作传播到每个依赖子键。对于“ON DELETE CASCADE”操作，这意味着与已删除的父行相关联的子表中的每行也将被删除。对于“ON UPDATE CASCADE”操作，这意味着将修改每个依赖子键中存储的值以匹配新的父键值。

例如，向外键添加如下所示的“ON UPDATE CASCADE”子句，可以增强第一部分中示例模式，允许用户更新 artistid（外键约束的父键）列而不会破坏引用完整性：

```sql
*-- Database schema*
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
CREATE TABLE track(
  trackid     INTEGER,
  trackname   TEXT, 
  trackartist INTEGER REFERENCES artist(artistid) ON UPDATE CASCADE
);

sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
1         Dean Martin      
2         Frank Sinatra    

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
11       That's Amore       1
12       Christmas Blues    1
13       My Way             2  

sqlite> *-- Update the artistid column of the artist record for "Dean Martin".*
sqlite> *-- Normally, this would raise a constraint, as it would orphan the two*
sqlite> *-- dependent records in the track table. However, the ON UPDATE CASCADE clause*
sqlite> *-- attached to the foreign key definition causes the update to "cascade"*
sqlite> *-- to the child table, preventing the foreign key constraint violation.*
sqlite> UPDATE artist SET artistid = 100 WHERE artistname = 'Dean Martin';

sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
2         Frank Sinatra    
100       Dean Martin      

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
11       That's Amore       100
12       Christmas Blues    100  
13       My Way             2  

```

配置 ON UPDATE 或 ON DELETE 操作并不意味着不需要满足外键约束。例如，如果配置了“ON DELETE SET DEFAULT”操作，但父表中不存在与子键列的默认值对应的行，则在存在依赖子键的情况下删除父键仍会导致外键违规。例如：

```sql
*-- Database schema*
CREATE TABLE artist(
  artistid    INTEGER PRIMARY KEY, 
  artistname  TEXT
);
CREATE TABLE track(
  trackid     INTEGER,
  trackname   TEXT, 
  trackartist INTEGER DEFAULT 0 REFERENCES artist(artistid) ON DELETE SET DEFAULT
);

sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
3         Sammy Davis Jr.

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
14       Mr. Bojangles      3

sqlite> *-- Deleting the row from the parent table causes the child key*
sqlite> *-- value of the dependent row to be set to integer value 0\. However, this*
sqlite> *-- value does not correspond to any row in the parent table. Therefore*
sqlite> *-- the foreign key constraint is violated and an is exception thrown.*
sqlite> DELETE FROM artist WHERE artistname = 'Sammy Davis Jr.';
SQL error: foreign key constraint failed

sqlite> *-- This time, the value 0 does correspond to a parent table row. And*
sqlite> *-- so the DELETE statement does not violate the foreign key constraint*
sqlite> *-- and no exception is thrown.*
sqlite> INSERT INTO artist VALUES(0, 'Unknown Artist');
sqlite> DELETE FROM artist WHERE artistname = 'Sammy Davis Jr.';

sqlite> SELECT * FROM artist;
artistid  artistname       
--------  -----------------
0         Unknown Artist

sqlite> SELECT * FROM track;
trackid  trackname          trackartist
-------  -----------------  -----------
14       Mr. Bojangles      0

```

熟悉 SQLite 触发器的人会注意到，在上面示例中演示的“ON DELETE SET DEFAULT”操作与以下 AFTER DELETE 触发器的效果类似：

```sql
CREATE TRIGGER on_delete_set_default AFTER DELETE ON artist BEGIN
  UPDATE child SET trackartist = 0 WHERE trackartist = old.artistid;
END;

```

每当删除外键约束的父表中的行，或者修改存储在父键列或列中的值时，逻辑事件的序列是：

1.  执行适用的 BEFORE 触发程序，

1.  检查本地（非外键）约束，

1.  更新或删除父表中的行，

1.  执行任何必需的外键操作，

1.  执行适用的 AFTER 触发程序。

ON UPDATE 外键操作和 SQL 触发器之间有一个重要的区别。只有当父键的值被修改，使新的父键值不等于旧值时，才会执行 ON UPDATE 操作。例如：

```sql
*-- Database schema*
CREATE TABLE parent(x PRIMARY KEY);
CREATE TABLE child(y REFERENCES parent ON UPDATE SET NULL);

sqlite> SELECT * FROM parent;
x
----
key

sqlite> SELECT * FROM child;
y
----
key

sqlite> *-- Since the following UPDATE statement does not actually modify*
sqlite> *-- the parent key value, the ON UPDATE action is not performed and*
sqlite> *-- the child key value is not set to NULL.*
sqlite> UPDATE parent SET x = 'key';
sqlite> SELECT IFNULL(y, 'null') FROM child;
y
----
key

sqlite> *-- This time, since the UPDATE statement does modify the parent key*
sqlite> *-- value, the ON UPDATE action is performed and the child key is set*
sqlite> *-- to NULL.*
sqlite> UPDATE parent SET x = 'key2';
sqlite> SELECT IFNULL(y, 'null') FROM child;
y
----
null

```

# 5\. CREATE、ALTER 和 DROP TABLE 命令

本节描述了 CREATE TABLE、ALTER TABLE 和 DROP TABLE 命令与 SQLite 的外键交互方式。

CREATE TABLE 命令在启用或禁用 外键约束 时操作相同。在创建表时不会检查外键约束的父键定义。用户可以创建引用不存在的父表或者不存在或者不被主键或唯一约束共同绑定的父键列的外键定义。

ALTER TABLE 命令在启用外键约束时，在两个方面的工作方式有所不同：

+   不可能使用 "ALTER TABLE ... ADD COLUMN" 语法添加包含 REFERENCES 子句的列，除非新列的默认值为 NULL。尝试这样做会返回错误。

+   如果使用 "ALTER TABLE ... RENAME TO" 命令重命名一个包含一个或多个外键约束的父表，那么外键约束的定义将被修改，以引用父表的新名称。存储在 sqlite_schema 表 中的子 CREATE TABLE 语句的文本将被修改，以反映新的父表名称。

如果在准备时启用了外键约束，那么 DROP TABLE 命令在删除表之前执行隐式的 DELETE 操作来删除表中的所有行。隐式的 DELETE 操作不会触发任何 SQL 触发器，但可能会调用外键操作或约束违规。如果立即外键约束违反，则 DROP TABLE 语句失败，表不会被删除。如果延迟外键约束违反，则在用户尝试提交事务时报告错误，如果在那时仍然存在外键约束违反。隐式 DELETE 过程中遇到的任何 "foreign key mismatch" 错误都将被忽略。

对于 ALTER TABLE 和 DROP TABLE 命令的这些增强功能的目的是确保它们在启用外键约束时无法用于创建包含外键违规的数据库，至少是在外键约束启用时。但是，这个规则有一个例外。如果父键不受作为父表定义的一部分创建的主键或唯一约束的约束，而是通过使用 CREATE INDEX 命令创建的索引受唯一约束的约束，则可以在不引起“外键不匹配”错误的情况下填充子表。如果从数据库模式中删除唯一索引，则将删除父表本身，不会报错。但是数据库可能处于一个状态，其中外键约束的子表包含不引用任何父表行的行。如果数据库模式中的所有父键都受到作为父表定义的一部分添加的主键或唯一约束的约束，而不是外部唯一索引的约束，则可以避免这种情况。

DROP TABLE 和 ALTER TABLE 命令的属性如上所述仅在启用外键时适用。如果用户认为它们不可取，那么解决方法是在执行 DROP 或 ALTER TABLE 命令之前使用 PRAGMA foreign_keys 来禁用外键约束。当然，在禁用外键约束时，用户可以违反外键约束，从而创建内部不一致的数据库。

# 6\. 限制和不支持的功能

本节列出了一些在其他地方未提及的限制和省略功能。

1.  **不支持 MATCH 子句。** 根据 SQL92，MATCH 子句可以附加到复合外键定义，以修改处理子键中出现的 NULL 值的方式。如果指定了 "MATCH SIMPLE"，则不需要子键对应于父表的任何行，如果一个或多个子键值为 NULL。如果指定了 "MATCH FULL"，则如果任何子键值为 NULL，则不需要父表中的相应行，但是所有子键值必须为 NULL。最后，如果外键约束声明为 "MATCH PARTIAL" 并且一个子键值为 NULL，则必须存在至少一行父表，其中非 NULL 子键值与父键值匹配。

    SQLite 解析 MATCH 子句（即，如果指定了 MATCH 子句，不会报告语法错误），但不强制执行它们。在 SQLite 中，所有外键约束都被处理为如果指定了 MATCH SIMPLE。

1.  **不支持在延迟模式和立即模式之间切换约束。** 许多系统允许用户在运行时切换单个外键约束的延迟和立即模式（例如使用 Oracle 的 "SET CONSTRAINT" 命令）。SQLite 不支持这一功能。在 SQLite 中，创建外键约束时会永久标记为延迟或立即模式。

1.  **外键操作的递归限制。** SQLITE_MAX_TRIGGER_DEPTH 和 SQLITE_LIMIT_TRIGGER_DEPTH 设置确定触发器程序递归的最大允许深度。对于这些限制，外键操作被视为触发器程序。PRAGMA recursive_triggers 设置不影响外键操作的运行。不可能禁用递归外键操作。

1.  **外键可能不会跨越模式边界。** 也就是说，在`REFERENCES (X.Y)`表中，表`X`只会在包含`REFERENCES`子句的模式内解析。
