# 1\. 简介

> 原文：[`sqlite.com/withoutrowid.html`](https://sqlite.com/withoutrowid.html)

默认情况下，SQLite 中的每一行都有一个特殊列，通常称为"rowid"，用于在表内唯一标识该行。然而，如果在 CREATE TABLE 语句的末尾添加了"WITHOUT ROWID"短语，则特殊的"rowid"列将被省略。省略 rowid 有时会带来空间和性能上的优势。

一个无 ROWID 表是使用[聚簇索引](https://en.wikipedia.org/wiki/Database_index#Clustered)作为主键的表。

## 1.1\. 语法

要创建一个无 ROWID 表，只需在 CREATE TABLE 语句的末尾添加关键字"WITHOUT ROWID"。例如：

> ```sql
> CREATE TABLE IF NOT EXISTS wordcount(
>   word TEXT PRIMARY KEY,
>   cnt INTEGER
> ) WITHOUT ROWID;
> 
> ```

与所有 SQL 语法一样，关键字的大小写不重要。可以写成"WITHOUT rowid"、"without rowid"或者"WiThOuT rOwId"，它们的意思是相同的。

每个无 ROWID 表必须有一个主键。如果没有主键的 CREATE TABLE 语句中缺少 WITHOUT ROWID 子句，则会引发错误。

在大多数情况下，普通表的特殊"rowid"列也可以称为"oid"或"_rowid_"。然而，在 CREATE TABLE 语句中只有"rowid"可以作为关键字。

## 1.2\. 兼容性

需要使用 SQLite 版本 3.8.2 (2013-12-06) 或更新版本才能使用无 ROWID 表。如果尝试使用较早版本的 SQLite 打开包含一个或多个无 ROWID 表的数据库，将导致"malformed database schema"错误。

## 1.3\. 怪癖

无 ROWID 仅在 SQLite 中发现，并且与任何其他 SQL 数据库引擎不兼容，据我们所知。在一个优雅的系统中，所有表都将像无 ROWID 表一样工作，即使没有无 ROWID 关键字。然而，当 SQLite 首次设计时，它仅使用整数 rowids 作为行键，以简化实现。这种方法多年来运行良好。但随着对 SQLite 的需求增加，需要确保主键确实对应于底层行键的表的需求也更加迫切。为了在不破坏向后兼容性的前提下满足这一需求，于 2013 年左右添加了无 ROWID 概念。

# 2\. 与普通 ROWID 表的区别

无 ROWID 语法是一种优化。它并不提供新的功能。使用无 ROWID 表可以完成的任何事情，在普通 ROWID 表中也可以以完全相同的方式、使用完全相同的语法来完成。无 ROWID 表的唯一优势是它有时可以比普通 ROWID 表占用更少的磁盘空间和/或执行速度更快。

就大部分而言，普通 rowid 表格和 WITHOUT ROWID 表格可以互换使用。但是 WITHOUT ROWID 表格有一些额外的限制，这些限制不适用于普通 rowid 表格：

1.  **每个 WITHOUT ROWID 表格必须有一个主键。** 尝试创建一个没有主键的 WITHOUT ROWID 表格将导致错误。

1.  **与“INTEGER PRIMARY KEY”关联的特殊行为不适用于 WITHOUT ROWID 表格。** 在普通表格中，“INTEGER PRIMARY KEY”意味着该列是 rowid 的别名。但是在 WITHOUT ROWID 表格中没有 rowid，因此该特殊含义不再适用。在 WITHOUT ROWID 表格中的“INTEGER PRIMARY KEY”列的行为类似于普通表格中的“INT PRIMARY KEY”列：它是一个具有整数关联性的主键。

1.  **AUTOINCREMENT 在 WITHOUT ROWID 表格上不起作用。** AUTOINCREMENT 机制假设存在 rowid，因此在 WITHOUT ROWID 表格上不起作用。如果在创建 WITHOUT ROWID 表格的 CREATE TABLE 语句中使用了“AUTOINCREMENT”关键字，则会引发错误。

1.  **在 WITHOUT ROWID 表格中，主键的每一列都强制为 NOT NULL。** 这符合 SQL 标准。每个主键列应该是独立的 NOT NULL。然而，由于一个 Bug，早期版本的 SQLite 没有强制主键列为 NOT NULL。发现这个 Bug 时，已经有太多 SQLite 数据库在使用，因此决定不修复这个 Bug，以免破坏兼容性。因此，SQLite 中的普通 rowid 表格违反了 SQL 标准，并允许在主键字段中插入 NULL 值。但是 WITHOUT ROWID 表格遵循标准，如果尝试向主键列插入 NULL 值，则会抛出错误。

1.  **sqlite3_last_insert_rowid()函数不适用于 WITHOUT ROWID 表格。** 插入 WITHOUT ROWID 表格不会改变 sqlite3_last_insert_rowid()函数返回的值。last_insert_rowid() SQL 函数也不受影响，因为它只是 sqlite3_last_insert_rowid()的包装。

1.  **增量 BLOB I/O](c3ref/blob_open.html)机制不适用于 WITHOUT ROWID 表格。** 增量 BLOB I/O 使用 rowid 创建 sqlite3_blob 对象来进行直接 I/O。然而，WITHOUT ROWID 表格没有 rowid，因此无法为 WITHOUT ROWID 表格创建 sqlite3_blob 对象。

1.  **sqlite3_update_hook() 接口不会对 WITHOUT ROWID 表的更改触发回调。** 来自 sqlite3_update_hook() 的回调的一部分是已更改的表行的 rowid。然而，WITHOUT ROWID 表没有 rowid。因此，在 WITHOUT ROWID 表更改时不会调用更新钩子。

# 3\. WITHOUT ROWID 表的好处

WITHOUT ROWID 表是一种可以减少存储和处理需求的优化。

在普通的 SQLite 表中，PRIMARY KEY 实际上只是一个 UNIQUE 索引。用于在磁盘上查找记录的关键是 rowid。普通 SQLite 表中的特殊“INTEGER PRIMARY KEY”列类型会导致该列成为 rowid 的别名，因此 INTEGER PRIMARY KEY 是真正的 PRIMARY KEY。但包括“INT PRIMARY KEY”在内的任何其他类型的 PRIMARY KEY 都只是普通 rowid 表中的唯一索引。

考虑一个用于存储词汇及其在某个文本语料库中每个单词出现次数的表（如下所示）：

> ```sql
> CREATE TABLE IF NOT EXISTS wordcount(
>   word TEXT PRIMARY KEY,
>   cnt INTEGER
> );
> 
> ```

作为普通的 SQLite 表，“wordcount”被实现为两个单独的 B 树。主表使用隐藏的 rowid 值作为键，并将“word”和“cnt”列存储为数据。CREATE TABLE 语句中的“TEXT PRIMARY KEY”短语导致在“word”列上创建一个 unique index。此索引是一个单独的 B 树，使用“word”和“rowid”作为键，并根本不存储数据。请注意，每个“word”的完整文本存储了两次：一次在主表中，一次在索引中。

考虑查询此表以查找单词“xsync”的出现次数：

> ```sql
> SELECT cnt FROM wordcount WHERE word='xsync';
> 
> ```

此查询首先必须搜索索引 B 树，查找包含“word”匹配值的任何条目。在索引中找到条目后，提取 rowid 并用于搜索主表。然后读取主表中的“cnt”值并返回。因此，需要执行两次单独的二进制搜索来完成请求。

WITHOUT ROWID 表使用等效表的不同数据设计。

> ```sql
> CREATE TABLE IF NOT EXISTS wordcount(
>   word TEXT PRIMARY KEY,
>   cnt INTEGER
> ) WITHOUT ROWID;
> 
> ```

在这个后表中，只有一个 B 树使用“word”列作为其键和“cnt”列作为其数据。（技术性说明：低级实现实际上将“word”和“cnt”都存储在 B 树的“key”区域。但除非您查看数据库文件的低级字节编码，否则这个事实并不重要。）由于只有一个 B 树，数据库中“word”列的文本只存储一次。此外，查询特定“word”的“cnt”值只涉及对主 B 树进行一次二进制搜索，因为可以直接从该第一次搜索找到的记录中检索“cnt”值，而无需对 rowid 进行第二次二进制搜索。

因此，在某些情况下，**WITHOUT ROWID** 表可以使用大约一半的磁盘空间，并且运行速度几乎可以快两倍。当然，在真实的架构中，通常会有次要索引和/或唯一约束条件，情况会更加复杂。但即使在这种情况下，对于具有非整数或复合主键的表来说，使用 **WITHOUT ROWID** 通常也可以带来空间和性能优势。

# 4\. 何时使用 **WITHOUT ROWID**

**WITHOUT ROWID** 优化可能对具有非整数或复合（多列）主键且不存储大字符串或 BLOB 的表有帮助。

对于只有单个 INTEGER 主键的表，**WITHOUT ROWID** 表将能够正确工作（即提供正确答案）。然而，在这种情况下，普通的 rowid 表将会更快。因此，避免创建具有 INTEGER 类型单列主键的 **WITHOUT ROWID** 表是一个良好的设计选择。

**WITHOUT ROWID** 表在单个行不太大的情况下效果最佳。一个良好的经验法则是，**WITHOUT ROWID** 表中单行的平均大小应该小于数据库页面大小的约 1/20。这意味着每行不应包含超过每个页面大小约 50 字节（对于 1KiB 页面大小）或约 200 字节（对于 4KiB 页面大小）。**WITHOUT ROWID** 表将能够处理（即得到正确答案）任意大的行 - 长达 2GB - 但是对于大行大小，传统的 rowid 表通常会更快。这是因为 rowid 表是作为 B*-Trees 实现的，其中所有内容存储在树的叶子节点中，而 **WITHOUT ROWID** 表则是使用普通的 B-Tree 实现，内容存储在叶子节点和中间节点上。在中间节点存储内容会导致每个中间节点条目在页面上占用更多空间，从而减少扇出，并增加搜索成本。

"sqlite3_analyzer.exe" 实用程序程序，作为 SQLite 源码树中的源代码或者作为预编译二进制文件出现在 [SQLite Download 页面](https://www.sqlite.org/download.html)，可用于测量现有 SQLite 数据库中表行的平均大小。

注意除了上面详细列出的一些特殊情况之外，**WITHOUT ROWID** 表和 rowid 表的工作方式是相同的。只要给定相同的 SQL 语句，它们都会生成相同的结果。因此，在开发周期的后期对应用程序进行实验来测试是否使用 **WITHOUT ROWID** 表会有帮助是一件简单的事情。一个好的策略是在产品开发接近尾声时，才回过头来运行测试，看看在具有非整数主键的表中添加 **WITHOUT ROWID** 是否有助于提升性能或者是否有害，并仅在有利的情况下保留 **WITHOUT ROWID**。

# 5\. 确定现有表是否为 **WITHOUT ROWID**

一个 WITHOUT ROWID 表对于 PRAGMA table_info 和 PRAGMA table_xinfo 命令返回与普通表相同的内容。但与普通表不同的是，WITHOUT ROWID 表还会响应 PRAGMA index_info 命令。对 WITHOUT ROWID 表使用 PRAGMA index_info 命令会返回关于表的主键的信息。通过这种方式，PRAGMA index_info 命令可以明确确定特定表是 WITHOUT ROWID 表还是普通表 - 普通表始终不返回行，而 WITHOUT ROWID 表始终返回一行或多行。
