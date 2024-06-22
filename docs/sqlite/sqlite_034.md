# 1\. 概述

> 原文：[`sqlite.com/dbstat.html`](https://sqlite.com/dbstat.html)

DBSTAT 虚拟表是一个只读的 eponymous 虚拟表，返回关于存储 SQLite 数据库内容所用磁盘空间量的信息。DBSTAT 虚拟表的示例用途包括 sqlite3_analyzer.exe 实用程序程序以及[Fossil 实现的](https://www.fossil-scm.org/)SQLite 版本控制系统中的[table size 饼图](https://www.sqlite.org/src/repo-tabsize)。

当 SQLite 使用 SQLITE_ENABLE_DBSTAT_VTAB 编译时选项构建时，DBSTAT 虚拟表在所有数据库连接上都可用。

DBSTAT 虚拟表是一个 eponymous 虚拟表，这意味着在使用之前不需要运行 CREATE VIRTUAL TABLE 创建 dbstat 虚拟表的实例。可以将 "dbstat" 模块名称用作查询 dbstat 虚拟表直接的表名。例如：

```sql
SELECT * FROM dbstat;

```

如果希望使用 dbstat 模块的命名虚拟表，则创建 dbstat 虚拟表实例的推荐方法如下：

```sql
CREATE VIRTUAL TABLE temp.stat USING dbstat(main);

```

注意在虚拟表名（"stat"）之前的 "temp." 限定词。此限定词使虚拟表在当前数据库连接的持续时间内为临时存在。这是推荐的方法。

dbstat 的 "main" 参数是提供信息的默认模式。默认为 "main"，因此上述示例中使用 "main" 是多余的。对于任何特定查询，可以通过在查询的 FROM 子句中的虚拟表名称的函数参数中指定替代模式来更改模式（有关更多详细信息，请参见 FROM 子句中的表值函数的进一步讨论）。

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

DBSTAT 表仅报告数据库文件中 B 树的内容。自由列表页面、指针映射页面和锁页面不包括在分析中。

默认情况下，数据库文件中每个 B 树页面的 DBSTAT 表中有一行。每行提供有关数据库的该页面的空间利用信息。但是，如果隐藏列 "aggregate" 为 TRUE，则结果将被聚合，DBSTAT 表中每个数据库中的每个 B 树仅有一行，提供跨整个 B 树的空间利用信息。

# 2\. dbstat 虚拟表的 "path" 列

“path”列描述了从 b 树结构的根节点到每个页面的路径。根节点本身的“path”是“/”。当“aggregate”为 TRUE 时，“path”为 NULL。“path”对于 b 树页的根节点的最左侧子页面是“/000/”。（B 树按从左到右的顺序存储内容，因此左边的页面的键比右边的页面的键小。）根页面的下一个页面是“/001”，依此类推，每个兄弟页面由 3 位十六进制值标识。第 451 个最左侧兄弟的子页面的路径如“/1c2/000/”，“/1c2/001/”等。溢出页通过将“+”字符和六位十六进制值附加到它们连接的单元格的路径来指定。例如，从根页面的第 450 个子页面的最左边单元格链接的三个溢出页由以下路径标识：

```sql
'/1c2/000+000000'         // First page in overflow chain
'/1c2/000+000001'         // Second page in overflow chain
'/1c2/000+000002'         // Third page in overflow chain

```

如果路径使用二进制排序规则排序，则与单元格相对应的溢出页将出现在其子页面的排序顺序之前：

```sql
'/1c2/000/'               // Left-most child of 451st child of root

```

# 3\. 聚合数据

从 SQLite 版本 3.31.0（2020-01-22）开始，DBSTAT 表新增了一个名为“aggregate”的隐藏列，如果约束为 TRUE，则 DBSTAT 将针对数据库中的每个 b 数生成一行，而不是每页生成一行。在聚合模式下运行时，“path”、“pagetype”和“pgoffset”列始终为 NULL，“pageno”列保存的是整个 b 树中的页面数，而不是对应行的页面数。

下表显示了 DBSTAT（非隐藏）列在正常模式和聚合模式下的含义：

> | 列 | 正常含义 | 聚合模式含义 |
> | --- | --- | --- |
> | name | 当前行对应的 b 树实现的表或索引的名称 |
> | path | 参见上面的描述 | 始终为 NULL |
> | pageno | 当前行所对应的数据库页面的页码 | 当前行所对应的 b 树中的页面总数 |
> | pagetype | ‘leaf’或‘interior’ | 始终为 NULL |
> | ncell | 当前页面或 b 树上的单元格数 |
> | payload | 当前页面或 b 树上的有用有效载荷字节数 |
> | unused | 当前页面或 b 树上未使用的字节 |
> | mx_payload | 当前页面或 b 树中找到的最大有效负载。 |
> | pgoffset | 页面开始的字节偏移 | 始终为 NULL |
> | pgsize | 当前页面或 b 树使用的总存储空间。 |

# 4\. DBSTAT 虚拟表的用例

要查找模式“aux1”中表“xyz”所使用的页面总数，请使用以下两个查询中的任一个（第一个是传统方法，第二个展示了聚合功能的使用方式）：

```sql
SELECT count(*) FROM dbstat('aux1') WHERE name='xyz';
SELECT pageno FROM dbstat('aux1',1) WHERE name='xyz';

```

要查看表格内容在磁盘上的存储效率，计算实际内容占用的空间与总磁盘空间的比值。这个比值越接近 100%，打包效率越高。（在此示例中，假定'xyz'表位于'main'模式中。再次提醒，这里有两个不同版本展示了使用 DBSTAT 的情况，分别是不带和带有新聚合特性。）

```sql
SELECT sum(pgsize-unused)*100.0/sum(pgsize) FROM dbstat WHERE name='xyz';
SELECT (pgsize-unused)*100.0/pgsize FROM dbstat
 WHERE name='xyz' AND aggregate=TRUE;

```

要找到表的平均扇出，运行：

```sql
SELECT avg(ncell) FROM dbstat WHERE name='xyz' AND pagetype='internal';

```

当磁盘访问是顺序时，现代文件系统的操作速度更快。因此，如果数据库文件的内容位于顺序页面上，SQLite 将运行更快。要找出数据库中顺序页面的比例（因此可以得到一个在决定何时执行 VACUUM 时可能有用的测量），运行如下查询：

```sql
CREATE TEMP TABLE s(rowid INTEGER PRIMARY KEY, pageno INT);
INSERT INTO s(pageno) SELECT pageno FROM dbstat ORDER BY path;
SELECT sum(s1.pageno+1==s2.pageno)*1.0/count(*)
  FROM s AS s1, s AS s2
 WHERE s1.rowid+1=s2.rowid;
DROP TABLE s;

```
