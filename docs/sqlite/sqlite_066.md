# 1\. 概述

> 原文：[`sqlite.com/dbstat.html`](https://sqlite.com/dbstat.html)

DBSTAT 虚拟表是一个只读的 同名虚拟表，用于返回有关存储 SQLite 数据库内容所使用的磁盘空间量的信息。DBSTAT 虚拟表的示例用途包括 sqlite3_analyzer.exe 实用程序程序以及 SQLite 的版本控制系统 [Fossil 实现](https://www.fossil-scm.org/) 中的 [表大小饼图](https://www.sqlite.org/src/repo-tabsize)。

当 SQLite 使用 SQLITE_ENABLE_DBSTAT_VTAB 编译时选项，DBSTAT 虚拟表可在所有 数据库连接 上使用。

DBSTAT 虚拟表是一个 同名虚拟表，意味着在使用它之前不需要运行 CREATE VIRTUAL TABLE 来创建 dbstat 虚拟表的实例。可以像使用表名一样使用 "dbstat" 模块名直接查询 dbstat 虚拟表。例如：

```sql
SELECT * FROM dbstat;

```

如果需要使用 dbstat 模块的命名虚拟表，则创建 dbstat 虚拟表实例的推荐方法如下：

```sql
CREATE VIRTUAL TABLE temp.stat USING dbstat(main);

```

注意虚拟表名称 ("stat") 前的 "temp." 限定符。该限定符使虚拟表成为临时表 - 仅在当前数据库连接的持续时间内存在。这是推荐的方法。

dbstat 的 "main" 参数是提供信息的默认模式。默认值为 "main"，因此上述示例中使用 "main" 是多余的。对于任何特定查询，可以通过在查询的 FROM 子句中将替代模式指定为虚拟表名称的函数参数来更改模式。 (有关更多详细信息，请参见 FROM 子句中的表值函数 进行进一步讨论。)

DBSTAT 虚拟表的模式如下：

```sql
CREATE TABLE dbstat(
  name       TEXT,        -- Name of table or index
  path       TEXT,        -- Path to page from root
  pageno     INTEGER,     -- Page number, or page count
  pagetype   TEXT,        -- 'internal', 'leaf', 'overflow', or NULL
  ncell      INTEGER,     -- Cells on page (0 for overflow pages)
  payload    INTEGER,     -- Bytes of payload on this page or btree
  unused     INTEGER,     -- Bytes of unused space on this page or btree
  mx_payload INTEGER,     -- Largest payload size of all cells on this row
  pgoffset   INTEGER,     -- Byte offset of the page in the database file
  pgsize     INTEGER,     -- Size of the page, in bytes
  schema     TEXT HIDDEN, -- Database schema being analyzed
  aggregate  BOOL HIDDEN  -- True to enable aggregate mode
);

```

DBSTAT 表仅报告数据库文件中 B 树的内容。空闲列表页、指针映射页和锁页不包含在分析中。

默认情况下，DBSTAT 表中对数据库文件中每个 B 树页有一行。每行提供有关数据库中该页空间利用率的信息。但是，如果隐藏列 "aggregate" 为 TRUE，则结果将被聚合，并且 DBSTAT 表中对数据库中每个 B 树只有一行，提供整个 B 树的空间利用率信息。

# 2\. dbstat 虚拟表的 "path" 列

"path" 列描述了从 B 树结构的根节点到每个页的路径。根节点本身的 "path" 为 '/'。当 "aggregate" 为 TRUE 时，"path" 为 NULL。对于 B 树页根的最左侧子页，"path" 为 '/000/'。（B 树按从左到右的顺序存储内容，因此左侧的页具有比右侧页更小的键。）根页的次左侧子页为 '/001'，依此类推，每个兄弟页由三位十六进制值标识。根页的第 451 个最左侧兄弟的子页路径如 '/1c2/000/'，'/1c2/001/' 等。溢出页通过在链接自的单元路径后附加 '+' 字符和六位十六进制值来指定。例如，从根页的第 450 个子页最左侧单元链中链接的三个溢出页由以下路径标识：

```sql
'/1c2/000+000000'         // First page in overflow chain
'/1c2/000+000001'         // Second page in overflow chain
'/1c2/000+000002'         // Third page in overflow chain

```

如果使用 BINARY 校对顺序对路径进行排序，则与单元相关的溢出页将在排序顺序中出现在其子页之前：

```sql
'/1c2/000/'               // Left-most child of 451st child of root

```

# 3\. 聚合数据

自 SQLite 版本 3.31.0（2020-01-22）起，DBSTAT 表新增了一个名为“聚合”（hidden column）的列，如果约束为 TRUE，则 DBSTAT 将在数据库中每个 B 树生成一行，而不是每个页面生成一行。在聚合模式下运行时，“path”、“pagetype”和“pgoffset”列始终为 NULL，“pageno”列保存整个 B 树中的页面数，而不是与行对应的页面数。

下表显示了 DBSTAT 的（非隐藏）列在普通模式和聚合模式下的含义：

> | Column | 普通含义 | 聚合模式含义 |
> | --- | --- | --- |
> | name | 当前行的 B 树实现的表或索引的名称 |
> | path | 参见上述描述 | 始终为 NULL |
> | pageno | 当前行的数据库页面号 | 当前行的 B 树中的页面总数 |
> | pagetype | 'leaf'或'interior' | 始终为 NULL |
> | ncell | 当前页或 B 树上的单元格数 |
> | payload | 当前页或 B 树上有用载荷的字节数 |
> | unused | 当前页或 B 树上未使用的字节数 |
> | mx_payload | 当前页或 B 树中找到的最大载荷。 |
> | pgoffset | 页面起始处的字节偏移量 | 始终为 NULL |
> | pgsize | 当前页或 B 树使用的总存储空间。 |

# 4\. dbstat 虚拟表的示例用法

要查找架构“aux1”中表“xyz”存储的页面总数，请使用以下两个查询之一（第一个是传统方式，第二个显示聚合特性的使用）：

```sql
SELECT count(*) FROM dbstat('aux1') WHERE name='xyz';
SELECT pageno FROM dbstat('aux1',1) WHERE name='xyz';

```

要查看表内容在磁盘上存储的效率如何，计算用于容纳实际内容的空间与总磁盘空间的比率。这个比率越接近 100%，打包效率越高。（在这个例子中，假设‘xyz’表位于‘main’模式中。同样，有两个不同的版本展示了使用 DBSTAT，分别是没有和带有新聚合特性的。）

```sql
SELECT sum(pgsize-unused)*100.0/sum(pgsize) FROM dbstat WHERE name='xyz';
SELECT (pgsize-unused)*100.0/pgsize FROM dbstat
 WHERE name='xyz' AND aggregate=TRUE;

```

要找出表的平均扇出，运行：

```sql
SELECT avg(ncell) FROM dbstat WHERE name='xyz' AND pagetype='internal';

```

现代文件系统在磁盘访问是顺序时运行速度更快。因此，如果数据库文件的内容在顺序页面上，SQLite 将运行得更快。要找出数据库中多少页面是顺序的（从而获取一个在决定何时执行 VACUUM 时可能有用的测量），可以运行如下查询：

```sql
CREATE TEMP TABLE s(rowid INTEGER PRIMARY KEY, pageno INT);
INSERT INTO s(pageno) SELECT pageno FROM dbstat ORDER BY path;
SELECT sum(s1.pageno+1==s2.pageno)*1.0/count(*)
  FROM s AS s1, s AS s2
 WHERE s1.rowid+1=s2.rowid;
DROP TABLE s;

```
