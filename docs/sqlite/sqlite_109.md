# 数据库速度比较

> 原文：[`sqlite.com/speed.html`](https://sqlite.com/speed.html)

**注意：这份文档非常非常古老。它描述了 SQLite、MySQL 和 PostgreSQL 的过时版本的速度比较。**

这里的数据已经变得毫无意义。此页面仅作为历史文献保留。

### 执行摘要

进行了一系列测试，以衡量 SQLite 2.7.6、PostgreSQL 7.1.3 和 MySQL 3.23.41 的相对性能。以下是从这些实验中得出的一般结论：

+   SQLite 2.7.6 在大多数常见操作中显著快速（有时快 10 到 20 倍），比 RedHat 7.2 上默认的 PostgreSQL 7.1.3 安装要快。

+   对于大多数常见操作，SQLite 2.7.6 通常比 MySQL 3.23.41 快（有时快两倍以上）。

+   SQLite 执行 CREATE INDEX 或 DROP TABLE 操作的速度不如其他数据库快。但这并不被视为问题，因为这些操作不频繁。

+   如果将多个操作组合到单个事务中，SQLite 将表现最佳。

这里呈现的结果有以下注意事项：

+   这些测试未尝试测量多用户性能或涉及多个连接和子查询的复杂查询的优化。

+   这些测试针对一个相对较小的（约 14 兆字节）数据库进行。它们不衡量数据库引擎在解决更大问题时的扩展能力。

### 测试环境

这些测试所使用的平台是一台 1.6GHz 的 Athlon 处理器，配备 1GB 内存和一块 IDE 硬盘驱动器。操作系统是 RedHat Linux 7.2，带有一个原始内核。

使用的 PostgreSQL 和 MySQL 服务器是在 RedHat 7.2 上默认提供的。（PostgreSQL 版本为 7.1.3，MySQL 版本为 3.23.41。）没有对这些引擎进行调优。特别注意，RedHat 7.2 上默认的 MySQL 配置不支持事务。虽然不支持事务给 MySQL 带来了较大的速度优势，但 SQLite 在大多数测试中仍能自己站稳脚跟。

我被告知，RedHat 7.3 中默认的 PostgreSQL 配置过于保守（它设计用于在有 8MB RAM 的机器上运行），PostgreSQL 可以通过一些熟练的配置调整来显著提高性能。Matt Sergeant 报告说他已调整了他的 PostgreSQL 安装，并重新运行了下面展示的测试。他的结果显示 PostgreSQL 和 MySQL 以大约相同的速度运行。有关 Matt 的结果，请访问

> 废弃的网址：http://www.sergeant.org/sqlite_vs_pgsync.html

SQLite 在网站上出现的相同配置下进行了测试。它使用了-O6 优化并使用了-DNDEBUG=1 开关，该开关禁用了 SQLite 代码中的许多“assert()”语句。编译器选项-DNDEBUG=1 大致会使 SQLite 的速度提高一倍。

所有测试均在一个静止的机器上进行。一个简单的 Tcl 脚本被用来生成和运行所有的测试。你可以在 SQLite 源代码树中的文件**tools/speedtest.tcl**中找到这个 Tcl 脚本的副本。

所有测试报告中的时间均表示以秒为单位的挂钟时间。SQLite 报告了两个不同的时间值。第一个值是 SQLite 在默认配置下，打开完整磁盘同步时的表现。打开同步时，SQLite 在关键点执行**fsync()**系统调用（或等效操作），确保关键数据实际写入磁盘驱动器表面。如果操作系统崩溃或计算机在数据库更新中断时意外关机，同步操作是确保数据库完整性的必要条件。第二个时间值是 SQLite 在关闭同步时的表现。关闭同步时，SQLite 有时会更快，但存在操作系统崩溃或意外断电可能损坏数据库的风险。一般而言，同步 SQLite 的时间用于与同步的 PostgreSQL 进行比较，而异步 SQLite 的时间用于与异步 MySQL 引擎进行比较。

### Test 1: 1000 插入操作

> CREATE TABLE t1(a INTEGER, b INTEGER, c VARCHAR(100));
> 
> INSERT INTO t1 VALUES(1,13153,'一万三千一百五十三');
> 
> INSERT INTO t1 VALUES(2,75560,'七万五千五百六十');
> 
> *... 995 行被省略*
> 
> INSERT INTO t1 VALUES(998,66289,'六万六千二百八十九');
> 
> INSERT INTO t1 VALUES(999,24322,'二万四千三百二十二');
> 
> INSERT INTO t1 VALUES(1000,94142,'九万四千一百四十二');

| PostgreSQL: |    4.373 |
| --- | --- |
| MySQL: |    0.114 |
| SQLite 2.7.6: |    13.061 |
| SQLite 2.7.6 (无同步): |    0.223 |

因为它没有一个中央服务器来协调访问，所以 SQLite 必须在每个事务中关闭和重新打开数据库文件，因此使其缓存失效。在这个测试中，每个 SQL 语句都是一个单独的事务，所以数据库文件必须被打开和关闭，并且缓存必须被刷新 1000 次。尽管如此，SQLite 的异步版本仍然几乎和 MySQL 一样快。然而，请注意同步版本有多慢。在同步测试的大部分 13 秒中，SQLite 都是处于等待磁盘 I/O 完成的空闲状态。

### 测试 2：在一个事务中插入 25000 条记录

> BEGIN;
> 
> CREATE TABLE t2(a INTEGER, b INTEGER, c VARCHAR(100));
> 
> INSERT INTO t2 VALUES(1,59672,'五万九千六百七十二');
> 
> *... 24997 lines omitted*
> 
> INSERT INTO t2 VALUES(24999,89569,'八万九千五百六十九');
> 
> INSERT INTO t2 VALUES(25000,94666,'九万四千六百六十六');
> 
> COMMIT;

| PostgreSQL: |    4.900 |
| --- | --- |
| MySQL: |    2.184 |
| SQLite 2.7.6: |    0.914 |
| SQLite 2.7.6 (nosync): |    0.757 |

当所有的 INSERT 语句都放在一个事务中时，SQLite 不再需要在每个语句之间关闭和重新打开数据库，或者在每个语句之间使缓存失效。它也不必在最后之前执行任何 fsync()操作。在这种情况下，SQLite 的性能比 PostgreSQL 和 MySQL 都要快得多。

### 测试 3：向一个带索引的表中插入 25000 条记录

> BEGIN;
> 
> CREATE TABLE t3(a INTEGER, b INTEGER, c VARCHAR(100));
> 
> CREATE INDEX i3 ON t3(c);
> 
> *... 24998 lines omitted*
> 
> INSERT INTO t3 VALUES(24999,88509,'八万八千五百零九');
> 
> INSERT INTO t3 VALUES(25000,84791,'八万四千七百九十一');
> 
> COMMIT;

| PostgreSQL: |    8.175 |
| --- | --- |
| MySQL: |    3.197 |
| SQLite 2.7.6: |    1.555 |
| SQLite 2.7.6 (nosync): |    1.402 |

有报道称 SQLite 在索引表上的表现不佳。最近增加了这个测试来证明这些谣言是不正确的。确实，SQLite 在创建新的索引条目方面不如其他引擎快（见下面的第 6 个测试），但其总体速度仍然更快。

### Test 4: 100 个没有索引的 SELECT 查询

> BEGIN;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=0 AND b<1000;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=100 AND b<1100;
> 
> *... 省略 96 行*
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=9800 AND b<10800;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=9900 AND b<10900;
> 
> COMMIT;

| PostgreSQL: |    3.629 |
| --- | --- |
| MySQL: |    2.760 |
| SQLite 2.7.6: |    2.494 |
| SQLite 2.7.6 (nosync): |    2.526 |

这个测试在一个 25000 条目的表上执行 100 次查询，没有索引，因此需要完整表扫描。SQLite 的先前版本在这个测试上比 PostgreSQL 和 MySQL 慢，但是最近的性能增强提高了其速度，使其成为这组中最快的。

### Test 5: 100 个基于字符串比较的 SELECT 查询

> BEGIN;
> 
> SELECT count(*), avg(b) FROM t2 WHERE c LIKE '%one%';
> 
> SELECT count(*), avg(b) FROM t2 WHERE c LIKE '%two%';
> 
> *... 省略 96 行*
> 
> SELECT count(*), avg(b) FROM t2 WHERE c LIKE '%ninety nine%';
> 
> SELECT count(*), avg(b) FROM t2 WHERE c LIKE '%one hundred%';
> 
> COMMIT;

| PostgreSQL: |    13.409 |
| --- | --- |
| MySQL: |    4.640 |
| SQLite 2.7.6: |    3.362 |
| SQLite 2.7.6 (nosync): |    3.372 |

这个测试仍然执行 100 次完整表扫描，但使用字符串比较而不是数值比较。SQLite 在这里比 PostgreSQL 快三倍以上，比 MySQL 快约 30%。

### Test 6: 创建索引

> CREATE INDEX i2a ON t2(a);
> 
> CREATE INDEX i2b ON t2(b);

| PostgreSQL: |    0.381 |
| --- | --- |
| MySQL: |    0.318 |
| SQLite 2.7.6: |    0.777 |
| SQLite 2.7.6 (nosync): |    0.659 |

SQLite 在创建新索引时较慢。这不是一个很大的问题（因为新索引并不经常创建），但正在解决中。希望 SQLite 的未来版本能在这方面做得更好。

### 测试 7：带索引的 5000 次 SELECT 查询

> SELECT count(*), avg(b) FROM t2 WHERE b>=0 AND b<100;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=100 AND b<200;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=200 AND b<300;
> 
> *... 4994 lines omitted*
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=499700 AND b<499800;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=499800 AND b<499900;
> 
> SELECT count(*), avg(b) FROM t2 WHERE b>=499900 AND b<500000;

| PostgreSQL: |    4.614 |
| --- | --- |
| MySQL: |    1.270 |
| SQLite 2.7.6: |    1.121 |
| SQLite 2.7.6 (nosync): |    1.162 |

三种数据库引擎在有索引时运行速度更快。但是 SQLite 仍然是最快的。

### 测试 8：不带索引的 1000 次 UPDATE 操作

> BEGIN;
> 
> UPDATE t1 SET b=b*2 WHERE a>=0 AND a<10;
> 
> UPDATE t1 SET b=b*2 WHERE a>=10 AND a<20;
> 
> *... 996 lines omitted*
> 
> UPDATE t1 SET b=b*2 WHERE a>=9980 AND a<9990;
> 
> UPDATE t1 SET b=b*2 WHERE a>=9990 AND a<10000;
> 
> COMMIT;

| PostgreSQL: |    1.739 |
| --- | --- |
| MySQL: |    8.410 |
| SQLite 2.7.6: |    0.637 |
| SQLite 2.7.6 (nosync): |    0.638 |

对于这个特定的 UPDATE 测试，MySQL 在速度上始终比 PostgreSQL 和 SQLite 慢五到十倍。我不知道为什么。MySQL 通常是一个非常快速的引擎。也许这个问题已经在 MySQL 的后续版本中得到了解决。

### 测试 9：带索引的 25000 次 UPDATE 操作

> BEGIN;
> 
> UPDATE t2 SET b=468026 WHERE a=1;
> 
> UPDATE t2 SET b=121928 WHERE a=2;
> 
> *... 24996 lines omitted*
> 
> UPDATE t2 SET b=35065 WHERE a=24999;
> 
> UPDATE t2 SET b=347393 WHERE a=25000;
> 
> COMMIT;

| PostgreSQL: |    18.797 |
| --- | --- |
| MySQL: |    8.134 |
| SQLite 2.7.6: |    3.520 |
| SQLite 2.7.6 (nosync): |    3.104 |

就在最近的 2.7.0 版本中，SQLite 在这个测试中的速度与 MySQL 相近。但是最近对 SQLite 的优化使得 UPDATE 操作的速度翻了一番还多。

### 测试 10：带索引的 25000 次文本 UPDATE 操作

> 开始;
> 
> 更新 t2 设置 c='一十四万八千三百八十二' WHERE a=1;
> 
> 更新 t2 设置 c='三十六万五千二' WHERE a=2;
> 
> *... 24996 行省略*
> 
> 更新 t2 设置 c='三十八万三千九十九' WHERE a=24999;
> 
> 更新 t2 设置 c='二十五万六千八百三十' WHERE a=25000;
> 
> 提交;

| PostgreSQL: |    48.133 |
| --- | --- |
| MySQL: |    6.982 |
| SQLite 2.7.6: |    2.408 |
| SQLite 2.7.6 (nosync): |    1.725 |

在这里，SQLite 2.7.0 版本运行速度与 MySQL 大致相同。但现在 2.7.6 版本比 MySQL 快两倍以上，比 PostgreSQL 快二十倍以上。

公平地对待 PostgreSQL，在这个测试中开始抛锚。一个有经验的管理员可能通过微调服务器来使 PostgreSQL 在这里运行得更快。

### 测试 11: INSERTs from a SELECT

> 开始;
> 
> 插入到 t1 从 t2 选择 b,a,c;
> 
> 插入到 t2 从 t1 选择 b,a,c;
> 
> 提交;

| PostgreSQL: |    61.364 |
| --- | --- |
| MySQL: |    1.537 |
| SQLite 2.7.6: |    2.787 |
| SQLite 2.7.6 (nosync): |    1.599 |

在这个测试中，SQLite 的异步版本比 MySQL 稍慢一点。（MySQL 在执行 INSERT...SELECT 语句时似乎特别擅长。）PostgreSQL 引擎仍在抛锚 - 它使用的 61 秒大部分时间都花在等待磁盘 I/O 上。

### 测试 12: DELETE without an index

> 删除从 t2 WHERE c LIKE '%fifty%';

| PostgreSQL: |    1.509 |
| --- | --- |
| MySQL: |    0.975 |
| SQLite 2.7.6: |    4.004 |
| SQLite 2.7.6 (nosync): |    0.560 |

SQLite 的同步版本在这个测试中是最慢的，而异步版本是最快的。差异在于执行 fsync() 需要额外的时间。

### 测试 13: DELETE with an index

> 删除从 t2 WHERE a>10 AND a<20000;

| PostgreSQL: |    1.316 |
| --- | --- |
| MySQL: |    2.262 |
| SQLite 2.7.6: |    2.068 |
| SQLite 2.7.6 (nosync): |    0.752 |

此测试非常重要，因为它是 PostgreSQL 比 MySQL 更快的少数几个测试之一。异步 SQLite 在这一点上，然而，比其他两个都要快。

### 测试 14: 大 DELETE 后的大 INSERT

> INSERT INTO t2 SELECT * FROM t1;

| PostgreSQL: |    13.168 |
| --- | --- |
| MySQL: |    1.815 |
| SQLite 2.7.6: |    3.210 |
| SQLite 2.7.6 (nosync): |    1.485 |

一些旧版 SQLite（在 2.4.0 版本之前）在大量 DELETE 后的新 INSERT 操作中表现出性能下降。如此测试显示，此问题现在已得到解决。

### 测试 15: 大量小的 INSERT 后的大 DELETE

> BEGIN;
> 
> DELETE FROM t1;
> 
> INSERT INTO t1 VALUES(1,10719,'一万零七百一十九');
> 
> *... 省略了 11997 行*
> 
> INSERT INTO t1 VALUES(11999,72836,'七万二千八百三十六');
> 
> INSERT INTO t1 VALUES(12000,64231,'六万四千二百三十一');
> 
> COMMIT;

| PostgreSQL: |    4.556 |
| --- | --- |
| MySQL: |    1.704 |
| SQLite 2.7.6: |    0.618 |
| SQLite 2.7.6 (nosync): |    0.406 |

SQLite 在事务内执行 INSERT 操作非常出色，这可能解释了为什么在这个测试中它比其他数据库快得多。

### 测试 16: DROP TABLE

> DROP TABLE t1;
> 
> DROP TABLE t2;
> 
> DROP TABLE t3;

| PostgreSQL: |    0.135 |
| --- | --- |
| MySQL: |    0.015 |
| SQLite 2.7.6: |    0.939 |
| SQLite 2.7.6 (nosync): |    0.254 |

当涉及到删除表时，SQLite 比其他数据库慢。这可能是因为当 SQLite 删除表时，它必须遍历并擦除数据库文件中处理该表的记录。而 MySQL 和 PostgreSQL 则使用单独的文件来表示每个表，因此它们可以通过删除文件来简单地删除表，这样做要快得多。

另一方面，删除表不是一个非常常见的操作，因此如果 SQLite 花费更长时间，这并不被视为一个大问题。
