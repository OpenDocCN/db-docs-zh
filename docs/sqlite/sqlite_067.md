# 1\. RBU 扩展

> 原文：[`sqlite.com/rbu.html`](https://sqlite.com/rbu.html)

RBU 扩展是专为在网络边缘的低功耗设备上使用的大型 SQLite 数据库文件而设计的附加模块。RBU 可以用于两个独立的任务：

+   **RBU 更新操作**。RBU 更新 是对数据库文件的批量更新，可以包括一个或多个表上的许多插入、更新和删除操作。

+   **RBU 真空操作**。RBU 真空 优化并重建整个数据库文件，其效果类似于 SQLite 的原生 VACUUM 命令。

RBU 的缩写代表 "Resumable Bulk Update"。

SQLite 内置的 SQL 命令可以完成这两个 RBU 功能 - RBU 更新通过在单个事务中执行一系列 INSERT、DELETE 和 UPDATE 命令，而 RBU 真空则通过单个 VACUUM 命令。RBU 模块相比这些更简单的方法提供了以下优势：

1.  **RBU 可能更高效**

    SQLite 使用的数据结构 B 树（用于在磁盘上存储每个表和索引）最有效的修改方法是按键顺序进行修改。但是，如果 SQL 表具有一个或多个索引，那么每个索引的键顺序可能与主表和其他辅助索引不同。因此，在执行一系列 INSERT、UPDATE 和 DELETE 语句时，通常无法按键顺序对所有 B 树进行更新。RBU 更新过程通过在一个通道中对主表应用所有修改，然后在单独的通道中对每个索引应用修改，确保每个 B 树都得到了最优更新。对于大型数据库文件（即不适合于操作系统磁盘缓存的文件），这一过程可以使更新速度提高两个数量级。

    RBU 真空操作所需的临时磁盘空间较少，写入磁盘的数据也较少，相较于 SQLite 的 VACUUM 操作。SQLite 的 VACUUM 操作需要大约是最终数据库文件大小的两倍的临时磁盘空间来运行，并且总写入数据量大约是最终数据库文件大小的三倍。相比之下，RBU 真空操作需要大约是最终数据库文件大小的临时磁盘空间，并且总写入数据量是两倍于此。

    另一方面，RBU 真空操作比常规 SQLite VACUUM 使用更多 CPU - 在某个测试中多达五倍。因此，在相同条件下，RBU 真空操作通常比 SQLite VACUUM 操作慢得多。

1.  **RBU 在后台运行**

    正在进行的 RBU 操作（更新或真空操作）不会干扰对数据库文件的读访问。

1.  **RBU 以增量方式运行**

    RBU 操作可以暂停，然后稍后恢复，期间可能发生断电和/或系统重启。对于 RBU 更新，原始数据库内容对所有数据库读取者可见，直到整个更新被应用完成 - 即使更新被暂停然后稍后恢复。

RBU 扩展默认未启用。要启用它，请使用编译时选项 SQLITE_ENABLE_RBU 编译 amalgamation。

# 2\. RBU 更新

## 2.1\. RBU 更新限制

以下限制适用于 RBU 更新：

+   更改必须仅包括 INSERT、UPDATE 和 DELETE 操作。不支持 CREATE 和 DROP 操作。

+   INSERT 语句不得使用默认值。

+   UPDATE 和 DELETE 语句必须通过 rowid 或非 NULL 主键值来识别目标行。

+   UPDATE 语句不得修改 PRIMARY KEY 或 rowid 值。

+   RBU 更新不能应用于包含名为"rbu_control"的列的任何表。

+   RBU 更新不会触发任何触发器。

+   RBU 更新不会检测或阻止外键或 CHECK 约束的违反。

+   所有 RBU 更新使用“OR ROLLBACK”约束处理机制。

+   目标数据库可能不处于 WAL 模式。

+   ~~目标数据库上可能没有表达式索引。~~ 表达式索引从 SQLite 3.30.0 版本（2019-10-04）开始支持。

+   在应用 RBU 更新时，目标数据库上不得进行其他写操作。为防止此类操作，目标数据库上会持有读锁。

## 2.2\. 准备 RBU 更新文件

所有 RBU 要应用的更改都存储在一个名为“RBU 数据库”的单独 SQLite 数据库中。要修改的数据库称为“目标数据库”。

每个目标数据库中将通过更新修改的表都会在 RBU 数据库中创建一个对应的表。RBU 数据库表的架构与目标数据库不同，但来源于目标数据库，具体描述请参见下文。

RBU 数据库表包含每个被更新的目标数据库行的单行记录。填充 RBU 数据库表的详细过程请参阅下一节。

### 2.2.1\. RBU 数据库架构

对于目标数据库中的每个表，RBU 数据库应包含一个名为"data<*integer*>_<*target-table-name*>"的表，其中<*target-table-name*>是目标数据库中表的名称，<*integer*>是任意长度的数字字符序列（0-9）。RBU 数据库中的表按名称顺序处理（按照二进制排序顺序从小到大），因此更新目标表的顺序受到数据 _% 表名称中<*integer*>部分选择的影响。在使用 RBU 更新 某些类型的虚拟表 时，这可能是有用的，但通常没有理由使用任何东西来代替<*integer*>部分除了空字符串。

数据 _% 表必须具有与目标表相同的所有列，另外还需要一个名为"rbu_control"的额外列。数据 _% 表不应具有主键或唯一约束，但是每列的类型应与目标数据库中相应列的类型相同。rbu_control 列不应具有任何类型。例如，如果目标数据库包含：

```sql
CREATE TABLE t1(a INTEGER PRIMARY KEY, b TEXT, c UNIQUE);

```

那么 RBU 数据库应包含：

```sql
CREATE TABLE data_t1(a INTEGER, b TEXT, c, rbu_control);

```

数据 _% 表中列的顺序无关紧要。

如果目标数据库表是虚拟表或没有主键声明的表，则数据 _% 表还必须包含一个名为"rbu_rowid"的列。rbu_rowid 列被映射到表的 ROWID。例如，如果目标数据库包含以下内容之一：

```sql
CREATE VIRTUAL TABLE x1 USING fts3(a, b);
CREATE TABLE x1(a, b);

```

那么 RBU 数据库应包含：

```sql
CREATE TABLE data_x1(a, b, rbu_rowid, rbu_control);

```

无法使用 RBU 更新“rowid”列不像主键值那样运行的虚拟表。

目标表的所有非隐藏列（即由 "SELECT *" 匹配的所有列）必须存在于输入表中。对于虚拟表，隐藏列是可选的 - 如果在输入表中存在，则由 RBU 更新，否则不会更新。例如，要向具有隐藏的 languageid 列的 fts4 表写入：

```sql
CREATE VIRTUAL TABLE ft1 USING fts4(a, b, languageid='langid');

```

可以使用以下任一输入表模式：

```sql
CREATE TABLE data_ft1(a, b, langid, rbu_rowid, rbu_control);
CREATE TABLE data_ft1(a, b, rbu_rowid, rbu_control);

```

### 2.2.2\. RBU 数据库内容

每行在 RBU 更新的一部分插入目标数据库时，对应的 data_% 表应包含一条 "rbu_control" 列设置为整数值 0 的记录。其他列应设置为组成要插入的新记录的值。

"rbu_control" 列的整数值也可以设为 2 用于插入操作。在这种情况下，新行会悄悄地替换任何具有相同主键值的现有行。这相当于先删除再插入具有相同主键值的行。这不同于 SQL 的 REPLACE 命令，在那种情况下，新行可能会替换任何冲突行（例如由于唯一约束或索引而产生的冲突行），而不仅仅是具有冲突主键的行。

如果目标数据库表具有 INTEGER PRIMARY KEY，则不可能向 IPK 列插入 NULL 值。试图这样做会导致 SQLITE_MISMATCH 错误。

每行在 RBU 更新中从目标数据库中删除时，对应的 data_% 表应包含一条 "rbu_control" 列设置为整数值 1 的记录。要删除的行的实际主键值应存储在 data_% 表的相应列中。其他列中存储的值不会被使用。

对于目标数据库中每行要作为 RBU 更新的一部分进行 UPDATE，相应的 data_% 表应包含一个 rbu_control 列设置为包含文本类型值。用于更新的真实主键值应存储在 data_% 表行的相应列中，所有更新的列的新值也应存储在其中。rbu_control 列中的文本值必须与目标数据库表中的列数相同，并且必须完全由 'x' 和 '.' 字符组成（或在某些特殊情况下为 'd' - 见下文）。对于要更新的每一列，相应的字符设置为 'x'。对于保持原样的列，rbu_control 值的相应字符应设置为 '.'。例如，给定上述表格，更新语句为：

```sql
UPDATE t1 SET c = 'usa' WHERE a = 4;

```

由 data_t1 行创建的数据 t1 列表示。

```sql
INSERT INTO data_t1(a, b, c, rbu_control) VALUES(4, NULL, 'usa', '..x');

```

如果使用 RBU 来更新目标数据库中的大型 BLOB 值，则将存储一个补丁或增量，以修改现有的 BLOB 而不是在 RBU 数据库中存储全新的值可能更有效。RBU 允许以两种方式指定增量：

+   在 "石油增量" 格式中 - 这是 [Fossil 源代码管理系统](http://fossil-scm.org) 使用的 BLOB 增量格式，或者

+   在 RBU 应用程序定义的自定义格式中。

石油增量格式仅用于更新 BLOB 值。在 data_% 表中，不存储新的 BLOB，而是存储石油增量。而不是在 rbu_control 字符串的列作为更新的一部分中指定 'x'，而是存储一个 'f' 字符。处理 'f' 更新时，RBU 从磁盘加载原始 BLOB 数据，将石油增量应用于其上，并将结果存储回数据库文件中。由 sqldiff --rbu 生成的 RBU 数据库在 RBU 数据库中保存空间时使用石油增量。

要使用自定义增量格式，RBU 应用程序必须在开始处理更新之前注册名为 "rbu_delta" 的用户定义 SQL 函数。rbu_delta() 将被调用两个参数 - 存储在目标表列中的原始值和作为 RBU 更新的一部分提供的增量值。它应返回将增量应用到原始值后的结果。要使用自定义增量函数，更新目标列时 rbu_control 值对应的字符必须设置为 'd' 而不是 'x'。然后，RBU 不是使用存储在对应数据 _% 列中的值来更新目标表，而是调用用户定义的 SQL 函数 "rbu_delta()" 并将结果存储在目标表列中。

例如，此行：

```sql
INSERT INTO data_t1(a, b, c, rbu_control) VALUES(4, NULL, 'usa', '..d');

```

导致 RBU 以类似方式更新目标数据库表：

```sql
UPDATE t1 SET c = rbu_delta(c, 'usa') WHERE a = 4;

```

如果目标数据库表是虚拟表或者没有主键的表，则 rbu_control 值不应包含与 rbu_rowid 值对应的字符。例如，这样：

```sql
INSERT INTO data_ft1(a, b, rbu_rowid, rbu_control) 
  VALUES(NULL, 'usa', 12, '.x');

```

导致类似如下结果：

```sql
UPDATE ft1 SET b = 'usa' WHERE rowid = 12;

```

数据 _% 表本身不应有主键声明。然而，如果从每个数据 _% 表中按 "rowid" 排序的行顺序与按对应目标数据库表的主键排序它们的顺序大致相同，那么 RBU 就更有效率。换句话说，在插入到数据 _% 表之前，应该使用目标表主键字段对行进行排序。

### 2.2.3\. 使用 RBU 与 FTS3/4 表

通常，[FTS3 或 FTS4](https://fts3.html) 表是具有类似主键功能的虚拟表的示例。因此，对于以下 FTS4 表：

```sql
CREATE VIRTUAL TABLE ft1 USING fts4(addr, text);
CREATE VIRTUAL TABLE ft2 USING fts4;             -- implicit "content" column

```

数据 _% 表可以按以下方式创建：

```sql
CREATE TABLE data_ft1 USING fts4(addr, text, rbu_rowid, rbu_control);
CREATE TABLE data_ft2 USING fts4(content, rbu_rowid, rbu_control);

```

并填充，就好像目标表是普通的 SQLite 表且没有显式的主键列。

[无内容的 FTS4 表](https://fts3.html#_contentless_fts4_tables_) 处理方式类似，除非尝试更新或删除行会在应用更新时导致错误。

外部内容 FTS4 表 也可以使用 RBU 进行更新。在这种情况下，用户需要配置 RBU 数据库，以便将相同的 UPDATE、DELETE 和 INSERT 操作应用到 FTS4 索引和底层内容表。对于所有外部内容 FTS4 表的更新，用户还需要确保任何 UPDATE 或 DELETE 操作先应用到 FTS4 索引，然后再应用到底层内容表（详细说明请参阅 FTS4 文档）。在 RBU 中，通过确保用于向 FTS4 表写入的 data_% 表的名称按照 BINARY 校对序列排序在用于更新底层内容表的 data_% 表的名称之前来完成这一操作。为了避免在 RBU 数据库中重复数据，可以使用 SQL 视图代替其中一个 data_% 表。例如，对于目标数据库模式：

```sql
CREATE TABLE ccc(addr, text);
CREATE VIRTUAL TABLE ccc_fts USING fts4(addr, text, content=ccc);

```

可以使用以下 RBU 数据库模式：

```sql
CREATE TABLE data_ccc(addr, text, rbu_rowid, rbu_control);
CREATE VIEW data0_ccc_fts AS SELECT * FROM data_ccc;

```

数据表 data_ccc 可以像往常一样填充，以进行对目标数据库表 ccc 的更新。RBU 将从 data0_ccc_fts 视图读取相同的更新，并将其应用于 FTS 表 ccc_fts。因为 "data0_ccc_fts" 比 "data_ccc" 小，所以按照要求先更新 FTS 表。

对于底层内容表具有显式 INTEGER PRIMARY KEY 列的情况稍微复杂，因为存储在 rbu_control 列中的文本值对于 FTS 索引及其底层内容表略有不同。对于底层内容表，rbu_control 文本值中必须包含字符以表示显式 IPK，但对于具有隐式 rowid 的 FTS 表本身，则不应该包含。尽管这很不便，但可以通过更复杂的视图来解决，如下所示：

```sql
-- Target database schema
CREATE TABLE ddd(i INTEGER PRIMARY KEY, k TEXT);
CREATE VIRTUAL TABLE ddd_fts USING fts4(k, content=ddd);

-- RBU database schema
CREATE TABLE data_ccc(i, k, rbu_control);
CREATE VIEW data0_ccc_fts AS SELECT i AS rbu_rowid, k, CASE 
  WHEN rbu_control IN (0,1) THEN rbu_control ELSE substr(rbu_control, 2) END
FROM data_ccc;

```

在上述的 SQL 视图中，substr()函数返回 rbu_control 参数的文本，去除第一个字符（对应于不需要在 FTS 表中的列“i”）。

### 2.2.4\. 使用 sqldiff 自动生成 RBU 更新

自 SQLite 版本 3.9.0（2015-10-14）起，sqldiff 实用工具能够生成代表两个具有相同模式的数据库之间差异的 RBU 数据库。例如，以下命令：

```sql
sqldiff --rbu t1.db t2.db

```

输出一个 SQL 脚本，用于创建一个 RBU 数据库，如果用于更新数据库 t1.db，则修补它以使其内容与数据库 t2.db 完全相同。

默认情况下，sqldiff 尝试处理两个提供给它的数据库中的所有非虚拟表。如果任何表在一个数据库中出现而在另一个数据库中不存在，或者如果任何表在一个数据库中的模式稍有不同，将会报错。如果这造成问题，“--table”选项可能会有帮助。

默认情况下，sqldiff 会忽略虚拟表。但是，可以通过类似以下命令显式创建一个虚拟表的 RBU 数据 _%表，其中包含类似主键功能的 rowid：

```sql
sqldiff --rbu --table <*virtual-table-name*> t1.db t2.db

```

不幸的是，尽管默认情况下忽略虚拟表，但是它们创建的任何底层数据库表以在数据库中存储数据的方式并不被忽略，并且 sqldiff 会将这些表添加到任何 RBU 数据库中。因此，试图使用 sqldiff 创建 RBU 更新以应用到具有一个或多个虚拟表的目标数据库的用户可能需要单独使用--table 选项运行 sqldiff 来更新每个要更新的目标数据库中的表。

## 2.3\. RBU 更新 C/C++编程

RBU 扩展接口允许应用将存储在 RBU 数据库中的 RBU 更新应用到现有的目标数据库。具体步骤如下：

1.  使用 sqlite3rbu_open(T,A,S)函数打开一个 RBU 句柄。

    T 参数是目标数据库文件的名称。A 参数是 RBU 数据库文件的名称。S 参数是用于存储在中断后恢复更新所需的状态信息的“状态数据库”的名称。如果 S 参数为 NULL，则状态信息存储在 RBU 数据库中，其表名均以“rbu_”开头。

    sqlite3rbu_open(T,A,S)函数返回一个指向“sqlite3rbu”对象的指针，然后将其传递给后续接口。

1.  使用由 sqlite3rbu_open()返回的数据库句柄来注册任何所需的虚拟表模块（其中参数 X 是从 sqlite3rbu_open()返回的 sqlite3rbu 指针）。此外，如有必要，使用 sqlite3_create_function_v2()注册 rbu_delta() SQL 函数。

1.  在 sqlite3rbu 对象指针 X 上调用 sqlite3rbu_step(X)函数一次或多次。每次调用 sqlite3rbu_step()执行单个 B 树操作，因此可能需要数千次调用才能应用完整的更新。当更新完全应用时，sqlite3rbu_step()接口将返回 SQLITE_DONE。

1.  调用 sqlite3rbu_close(X)以销毁 sqlite3rbu 对象指针。如果已调用 sqlite3rbu_step(X)足够次数以完全应用更新到目标数据库，则将 RBU 数据库标记为完全应用。否则，RBU 更新应用的状态将保存在状态数据库中（或者如果在 sqlite3rbu_open()中指定的状态数据库文件名称为 NULL，则保存在 RBU 数据库中），以便稍后恢复更新。

如果在调用 sqlite3rbu_close() 时目标数据库仅部分应用了更新，则状态信息将保存在状态数据库（如果存在）或者 RBU 数据库中。这允许后续的进程从中断的位置自动恢复 RBU 更新。如果状态信息存储在 RBU 数据库中，可以通过删除所有表名以 "rbu_" 开头的表来移除它。

欲了解更多详细信息，请参阅 [头文件 sqlite3rbu.h](http://sqlite.org/src/doc/trunk/ext/rbu/sqlite3rbu.h) 中的注释。

# 3\. RBU 清理

## 3.1\. RBU 清理限制

与 SQLite 内置的 VACUUM 命令相比，RBU 清理存在以下限制：

+   它不能用于包含 表达式索引 的数据库。

+   被清理的数据库可能不处于 WAL 模式。

## 3.2\. RBU 清理 C/C++ 编程

本节提供了 RBU 清理集成到应用程序中的概述和示例代码。有关详细信息，请参阅 [头文件 sqlite3rbu.h](http://sqlite.org/src/doc/trunk/ext/rbu/sqlite3rbu.h) 中的注释。

所有的 RBU 清理应用程序都实现了以下过程的某种变体：

1.  通过调用 sqlite3rbu_vacuum(T, S) 创建 RBU 句柄。

    参数 T 是要清理的数据库文件的名称。参数 S 是如果暂停清理操作，RBU 模块将保存其状态的数据库的名称。

    当调用 sqlite3rbu_vacuum()时，如果状态数据库 S 不存在，则会自动创建并填充，用于存储 RBU 真空状态的单个表 - "rbu_state"。如果正在进行的 RBU 真空被暂停，则此表会填充状态数据。下次使用相同的 S 参数调用 sqlite3rbu_vacuum()时，它会检测到这些数据，并尝试恢复暂停的真空操作。当 RBU 真空操作完成或遇到错误时，RBU 会自动删除 rbu_state 表的内容。在这种情况下，下一次调用 sqlite3rbu_vacuum()将从头开始进行全新的真空操作。

    建议根据目标数据库名称设定 RBU 真空状态数据库名称的约定。以下示例代码使用"<target>-vacuum"，其中<target>是正在进行真空操作的数据库名称。

1.  用于真空操作的数据库中使用的任何自定义排序序列都会在由 sqlite3rbu_db()函数返回的两个数据库句柄中注册。

1.  在 RBU 句柄上调用 sqlite3rbu_step()函数，直到 RBU 真空完成、出现错误或应用程序希望暂停为止。

    每次调用 sqlite3rbu_step()都会完成一部分真空操作的工作。根据数据库的大小，单次真空操作可能需要数千次 sqlite3rbu_step()调用。如果真空操作完成，sqlite3rbu_step()返回 SQLITE_DONE；如果真空操作尚未完成但没有错误发生，则返回 SQLITE_OK；如果遇到错误，则返回 SQLite 错误代码。如果发生错误，所有后续对 sqlite3rbu_step()的调用都会立即返回相同的错误代码。

1.  最后，调用 sqlite3rbu_close()关闭 RBU 句柄。如果应用在真空操作完成之前停止调用 sqlite3rbu_step()，则真空的状态会保存在状态数据库中，以便稍后恢复。

    与 sqlite3rbu_step()类似，如果真空操作已完成，sqlite3rbu_close()返回 SQLITE_DONE。如果真空尚未完成但未发生错误，则返回 SQLITE_OK。或者，如果发生错误，则返回 SQLite 错误代码。如果错误发生在先前的 sqlite3rbu_step()调用中，sqlite3rbu_close()将返回相同的错误代码。

以下示例代码演示了上述描述的技术。

```sql
*/**
*** Either start a new RBU vacuum or resume a suspended RBU vacuum on* 
*** database zTarget. Return when either an error occurs, the RBU* 
*** vacuum is finished or when the application signals an interrupt*
*** (code not shown).*
****
*** If the RBU vacuum is completed successfully, return SQLITE_DONE.*
*** If an error occurs, return SQLite error code. Or, if the application*
*** signals an interrupt, suspend the RBU vacuum operation so that it*
*** may be resumed by a subsequent call to this function and return*
*** SQLITE_OK.*
****
*** This function uses the database named "<zTarget>-vacuum" for*
*** the state database, where <zTarget> is the name of the database* 
*** being vacuumed.*
**/*
int do_rbu_vacuum(const char *zTarget){
  int rc;
  char *zState;                   */* Name of state database */*
  sqlite3rbu *pRbu;               */* RBU vacuum handle */*

  zState = sqlite3_mprintf("%s-vacuum", zTarget);
  if( zState==0 ) return SQLITE_NOMEM;
  pRbu = sqlite3rbu_vacuum(zTarget, zState);
  sqlite3_free(zState);

  if( pRbu ){
    sqlite3 *dbTarget = sqlite3rbu_db(pRbu, 0);
    sqlite3 *dbState = sqlite3rbu_db(pRbu, 1);

    */* Any custom collation sequences used by the target database must*
    *** be registered with both database handles here.  */*

    while( sqlite3rbu_step(pRbu)==SQLITE_OK ){
      if( *<application has signaled interrupt>* ) break;
    }
  }
  rc = sqlite3rbu_close(pRbu);
  return rc;
}

```
