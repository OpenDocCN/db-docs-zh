# 1\. 引言

> 原文：[`sqlite.com/vtab.html#tabfunc2`](https://sqlite.com/vtab.html#tabfunc2)

虚拟表是一个注册到开放 SQLite 数据库连接的对象。从 SQL 语句的角度看，虚拟表对象看起来像任何其他表或视图。但在幕后，对虚拟表的查询和更新会调用虚拟表对象的回调方法，而不是直接读取和写入数据库文件。

虚拟表机制允许应用程序发布可以从 SQL 语句访问的接口，就像它们是表一样。SQL 语句可以对虚拟表做几乎任何与真实表相同的操作，但以下操作除外：

+   不能在虚拟表上创建触发器。

+   不能在虚拟表上创建额外的索引。（虚拟表可以有索引，但必须内建到虚拟表的实现中。不能通过 CREATE INDEX 语句单独添加索引。）

+   不能对虚拟表运行 ALTER TABLE ... ADD COLUMN 命令。

各个虚拟表实现可能施加额外的约束。例如，一些虚拟实现可能提供只读表。或者一些虚拟表实现可能允许 INSERT 或 DELETE，但不允许 UPDATE。或者一些虚拟表实现可能限制可以进行的 UPDATE 类型。

虚拟表可以代表内存中的数据结构。或者它可以表示磁盘上不在 SQLite 格式中的数据的视图。或者应用程序可能按需计算虚拟表的内容。

下面是一些现有和假设的虚拟表用途：

+   全文搜索接口

+   使用 R-Tree 进行空间索引

+   检查 SQLite 数据库文件的磁盘内容（dbstat 虚拟表）

+   读取和/或写入逗号分隔值（CSV）文件的内容

+   访问主机计算机的文件系统，就像它是一个数据库表一样。

+   启用 SQL 在统计软件包如 R 中对数据的操作

查看虚拟表列表页面，了解更多实际虚拟表实现的列表。

## 1.1\. 使用方法

使用 CREATE VIRTUAL TABLE 语句创建虚拟表。

**create-virtual-table-stmt:**

<svg class="pikchr" viewBox="0 0 624.096 259.848"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CREATE</text> <text x="183" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VIRTUAL</text> <text x="286" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">TABLE</text> <text x="372" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IF</text> <text x="435" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="521" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">EXISTS</text> <text x="95" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">schema-name</text> <text x="197" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="300" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-name</text> <text x="67" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">USING</text> <text x="187" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">module-name</text> <text x="300" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="432" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">module-argument</text> <text x="563" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="432" y="242" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

CREATE VIRTUAL TABLE 语句创建一个名为 table-name 的新表，其来源于类 module-name。module-name 是通过 sqlite3_create_module() 接口为虚拟表注册的名称。

```sql
CREATE VIRTUAL TABLE tablename USING modulename;

```

模块的名称后面还可以提供逗号分隔的参数。

```sql
CREATE VIRTUAL TABLE tablename USING modulename(arg1, arg2, ...);

```

模块参数的格式非常通用。每个模块参数可以包含关键字、字符串字面值、标识符、数字和标点符号。在创建虚拟表时，每个模块参数将按照原文（作为文本）传递给虚拟表实现的 构造方法，该构造方法负责解析和解释这些参数。参数的语法足够通用，以至于虚拟表实现如果希望的话，可以将其参数解释为普通 CREATE TABLE 语句中的 列定义。实现也可以对参数施加其他解释。

一旦创建了虚拟表，它可以像任何其他表一样使用，但具体虚拟表实现所规定的例外情况除外。销毁虚拟表时，需使用普通的 DROP TABLE 语法。

### 1.1.1\. 临时虚拟表

没有 "CREATE TEMP VIRTUAL TABLE" 语句。要创建临时虚拟表，请在虚拟表名称前添加 "temp" 模式。

```sql
CREATE VIRTUAL TABLE temp.tablename USING module(arg1, ...);

```

### 1.1.2\. 同名虚拟表

某些虚拟表会自动存在于每个数据库连接的 "main" 模式中，即使没有 CREATE VIRTUAL TABLE 语句。此类虚拟表称为 "同名虚拟表"。要使用同名虚拟表，只需将模块名称用作普通表即可。同名虚拟表仅存在于 "main" 模式中，因此如果使用不同模式名称前缀，它们将无法工作。

一个例子是 dbstat 虚拟表。要将 dbstat 虚拟表用作同名虚拟表，只需针对 "dbstat" 模块名称进行查询，就像它是普通表一样。（请注意，SQLite 必须使用 SQLITE_ENABLE_DBSTAT_VTAB 选项编译，以在构建中包含 dbstat 虚拟表。）

```sql
SELECT * FROM dbstat;

```

如果其 xCreate 方法与 xConnect 方法完全相同，或者 xCreate 方法为 NULL，则虚拟表称为同名的。当使用 CREATE VIRTUAL TABLE 语句首次创建虚拟表时，将调用 xCreate 方法。当数据库连接附加到或重新解析模式时，将调用 xConnect 方法。当这两个方法相同时，表示虚拟表没有需要创建和销毁的持久状态。

### 1.1.3\. 仅同名虚拟表

如果 xCreate 方法为 NULL，则禁止为该虚拟表使用 CREATE VIRTUAL TABLE 语句，该虚拟表称为 "仅同名虚拟表"。仅同名虚拟表可用作表值函数。

注意，在 version 3.9.0 (2015-10-14) 之前，SQLite 在调用 xCreate 方法之前不会检查其是否为 NULL。因此，如果仅具有同名的虚拟表，并且尝试对该虚拟表模块使用 CREATE VIRTUAL TABLE 命令，则会发生跳转到 NULL 指针，导致崩溃。

## 1.2\. 实现

虚拟表实现使用了几个新的 C 级对象：

```sql
typedef struct sqlite3_vtab sqlite3_vtab;
typedef struct sqlite3_index_info sqlite3_index_info;
typedef struct sqlite3_vtab_cursor sqlite3_vtab_cursor;
typedef struct sqlite3_module sqlite3_module;

```

sqlite3_module 结构定义了一个模块对象，用于实现虚拟表。可以将模块想象为可以构建具有类似属性的多个虚拟表的类。例如，可以有一个模块提供对磁盘上逗号分隔值（CSV）文件的只读访问。然后可以使用该单个模块创建多个虚拟表，其中每个虚拟表引用不同的 CSV 文件。

模块结构包含由 SQLite 调用的方法，执行虚拟表的各种操作，如创建新实例、销毁旧实例、读取和写入数据、搜索和删除、更新或插入行。模块结构将在下文详细解释。

每个虚拟表实例都由一个 sqlite3_vtab 结构表示。sqlite3_vtab 结构如下所示：

```sql
struct sqlite3_vtab {
  const sqlite3_module *pModule;
  int nRef;
  char *zErrMsg;
};

```

虚拟表实现通常会子类化这个结构以添加额外的私有和特定于实现的字段。nRef 字段由 SQLite 核心内部使用，不应该被虚拟表实现修改。虚拟表实现可以通过在 zErrMsg 中放置错误消息字符串来向核心传递错误消息文本，这是从 SQLite 内存分配函数（如 sqlite3_mprintf()或 sqlite3_malloc()）中获取空间来存储该错误消息字符串。在给 zErrMsg 分配新值之前，虚拟表实现必须使用 sqlite3_free()释放 zErrMsg 的任何现有内容。如果未能执行此操作，将导致内存泄漏。当核心向客户端应用程序交付错误消息文本或销毁虚拟表时，核心将释放并清零 zErrMsg 的内容。虚拟表实现只需要在将内容覆盖为新的不同错误消息时担心释放 zErrMsg 的内容。

sqlite3_vtab_cursor 结构表示虚拟表特定行的指针。sqlite3_vtab_cursor 的结构如下所示：

```sql
struct sqlite3_vtab_cursor {
  sqlite3_vtab *pVtab;
};

```

再次强调，实际实现很可能会子类化这个结构以添加额外的私有字段。

sqlite3_index_info 结构用于在实现虚拟表的模块的 xBestIndex 方法中传入和传出信息。

在运行 CREATE VIRTUAL TABLE 语句之前，必须注册该语句中指定的模块到数据库连接。可以使用 sqlite3_create_module()或 sqlite3_create_module_v2()接口来完成这一操作：

```sql
int sqlite3_create_module(
  sqlite3 *db,               /* SQLite connection to register module with */
  const char *zName,         /* Name of the module */
  const sqlite3_module *,    /* Methods for the module */
  void *                     /* Client data for xCreate/xConnect */
);
int sqlite3_create_module_v2(
  sqlite3 *db,               /* SQLite connection to register module with */
  const char *zName,         /* Name of the module */
  const sqlite3_module *,    /* Methods for the module */
  void *,                    /* Client data for xCreate/xConnect */
  void(*xDestroy)(void*)     /* Client data destructor function */
);

```

sqlite3_create_module() 和 sqlite3_create_module_v2() 程序将模块名称与一个 sqlite3_module 结构及一个特定于每个模块的客户端数据关联起来。这两种 create_module 方法的唯一区别是 _v2 方法包含一个额外参数，指定客户端数据指针的析构函数。模块结构定义了虚拟表的行为。模块结构看起来像这样：

```sql
struct sqlite3_module {
  int iVersion;
  int (*xCreate)(sqlite3*, void *pAux,
               int argc, char *const*argv,
               sqlite3_vtab **ppVTab,
               char **pzErr);
  int (*xConnect)(sqlite3*, void *pAux,
               int argc, char *const*argv,
               sqlite3_vtab **ppVTab,
               char **pzErr);
  int (*xBestIndex)(sqlite3_vtab *pVTab, sqlite3_index_info*);
  int (*xDisconnect)(sqlite3_vtab *pVTab);
  int (*xDestroy)(sqlite3_vtab *pVTab);
  int (*xOpen)(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor);
  int (*xClose)(sqlite3_vtab_cursor*);
  int (*xFilter)(sqlite3_vtab_cursor*, int idxNum, const char *idxStr,
                int argc, sqlite3_value **argv);
  int (*xNext)(sqlite3_vtab_cursor*);
  int (*xEof)(sqlite3_vtab_cursor*);
  int (*xColumn)(sqlite3_vtab_cursor*, sqlite3_context*, int);
  int (*xRowid)(sqlite3_vtab_cursor*, sqlite_int64 *pRowid);
  int (*xUpdate)(sqlite3_vtab *, int, sqlite3_value **, sqlite_int64 *);
  int (*xBegin)(sqlite3_vtab *pVTab);
  int (*xSync)(sqlite3_vtab *pVTab);
  int (*xCommit)(sqlite3_vtab *pVTab);
  int (*xRollback)(sqlite3_vtab *pVTab);
  int (*xFindFunction)(sqlite3_vtab *pVtab, int nArg, const char *zName,
                     void (**pxFunc)(sqlite3_context*,int,sqlite3_value**),
                     void **ppArg);
  int (*xRename)(sqlite3_vtab *pVtab, const char *zNew);
  /* The methods above are in version 1 of the sqlite_module object. Those 
  ** below are for version 2 and greater. */
  int (*xSavepoint)(sqlite3_vtab *pVTab, int);
  int (*xRelease)(sqlite3_vtab *pVTab, int);
  int (*xRollbackTo)(sqlite3_vtab *pVTab, int);
  /* The methods above are in versions 1 and 2 of the sqlite_module object.
  ** Those below are for version 3 and greater. */
  int (*xShadowName)(const char*);
  /* The methods above are in versions 1 through 3 of the sqlite_module object.
  ** Those below are for version 4 and greater. */
  int (*xIntegrity)(sqlite3_vtab *pVTab, const char *zSchema,
                    const char *zTabName, int mFlags, char **pzErr);
};

```

模块结构定义了每个虚拟表对象的所有方法。模块结构还包含 iVersion 字段，该字段定义了模块表结构的特定版本。目前，iVersion 总是 4 或更低，但在 SQLite 的未来版本中，模块结构定义可能会扩展以包含额外的方法，届时最大的 iVersion 值将会增加。

模块结构的其余部分由用于实现虚拟表各种功能的方法组成。关于每个方法的详细信息在下文中提供。

## 1.3\. 虚拟表与共享缓存

在 SQLite 版本 3.6.17（2009-08-10）之前，虚拟表机制假设每个数据库连接保留自己的数据库模式副本。因此，在启用了共享缓存模式的数据库中无法使用虚拟表机制。如果启用了共享缓存模式，sqlite3_create_module()接口会返回错误。从 SQLite 版本 3.6.17 开始放宽了该限制。

## 1.4\. 创建新的虚拟表实现

按照以下步骤创建您自己的虚拟表：

1.  编写所有必要的方法。

1.  创建一个包含步骤 1 所有方法指针的 sqlite3_module 结构的实例。

1.  使用其中一个 sqlite3_create_module() 或 sqlite3_create_module_v2() 接口注册您的 sqlite3_module 结构。

1.  运行一个创建虚拟表的命令，在`USING`子句中指定新模块。

唯一真正困难的部分是步骤 1。你可能希望从现有的虚拟表实现开始，并根据需要进行修改。[SQLite 源代码树](https://sqlite.org/src/dir?ci=trunk&type=tree)包含许多适合复制的虚拟表实现，包括：

+   **[templatevtab.c](https://sqlite.org/src/file/ext/misc/templatevtab.c)** → 专门创建的虚拟表，作为其他自定义虚拟表的模板。

+   **[series.c](https://sqlite.org/src/file/ext/misc/series.c)** → 生成 generate_series() 表值函数的实现。

+   **[json.c](https://sqlite.org/src/file/src/json.c)** → 包含了 json_each() 和 json_tree() 表值函数的源代码。

+   **[csv.c](https://sqlite.org/src/file/ext/misc/csv.c)** → 一个读取 CSV 文件的虚拟表。

在 SQLite 源树中有 许多其他虚拟表实现，可用作示例。通过搜索 "sqlite3_create_module" 可找到这些其他虚拟表实现。

您可能还希望将新的虚拟表实现为一个 可加载扩展。

# 2\. 虚拟表方法

## 2.1\. xCreate 方法

```sql
int (*xCreate)(sqlite3 *db, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

xCreate 方法是在响应 CREATE VIRTUAL TABLE 语句时调用的，用于创建虚拟表的新实例。如果 xCreate 方法与 xConnect 方法相同，则该虚拟表是一个 同名虚拟表。如果省略了 xCreate 方法（如果它是一个空指针），那么该虚拟表是一个 仅同名虚拟表。

db 参数是指向执行 CREATE VIRTUAL TABLE 语句的 SQLite 数据库连接 的指针。pAux 参数是在注册 虚拟表模块 时作为第四个参数传递给 sqlite3_create_module() 或 sqlite3_create_module_v2() 调用的客户端数据指针的副本。argv 参数是一个包含 argc 个指向空终止字符串的指针数组。数组的第一个字符串，argv[0]，是被调用模块的名称。模块名称是作为 sqlite3_create_module() 的第二个参数提供的名称，也是 CREATE VIRTUAL TABLE 语句的 USING 子句中提供的参数。数组的第二个字符串，argv[1]，是正在创建新虚拟表的数据库的名称。对于主数据库，数据库名称是 "main"，对于 TEMP 数据库，名称是 "temp"，对于附加数据库，名称是在 ATTACH 语句的末尾指定的名称。数组的第三个元素，argv[2]，是在 CREATE VIRTUAL TABLE 语句中的 TABLE 关键字后指定的新虚拟表的名称。如果存在，argv[] 中的第四个及其后的字符串报告了 CREATE VIRTUAL TABLE 语句中模块名称的参数。

该方法的作用是构造新的虚拟表对象（一个 sqlite3_vtab 对象），并在 *ppVTab 中返回指向该对象的指针。

作为创建新的 sqlite3_vtab 结构的任务的一部分，该方法必须调用 sqlite3_declare_vtab() 来告知 SQLite 核心虚拟表中的列和数据类型。sqlite3_declare_vtab() API 具有以下原型：

```sql
int sqlite3_declare_vtab(sqlite3 *db, const char *zCreateTable)

```

调用 sqlite3_declare_vtab() 的第一个参数必须与此方法的第一个参数相同，即 数据库连接 指针。调用 sqlite3_declare_vtab() 的第二个参数必须是以零结尾的 UTF-8 字符串，包含一个形式良好的 CREATE TABLE 语句，定义虚拟表中的列和它们的数据类型。在此 CREATE TABLE 语句中表的名称被忽略，以及所有约束。只有列名和数据类型才重要。CREATE TABLE 语句的字符串不需要持久内存。一旦 sqlite3_declare_vtab() 程序返回，该字符串就可以被释放和/或重用。

通过调用 sqlite3_vtab_config() 接口，xConnect 方法还可以选择性地为虚拟表请求特殊功能：

```sql
int sqlite3_vtab_config(sqlite3 *db, int op, ...);

```

对 sqlite3_vtab_config() 的调用是可选的。但为了最大的安全性，建议虚拟表实现在虚拟表不会从触发器或视图内部使用时调用 "sqlite3_vtab_config(db, SQLITE_VTAB_DIRECTONLY)"。

xCreate 方法不需要初始化 sqlite3_vtab 对象的 pModule、nRef 和 zErrMsg 字段。SQLite 核心将负责处理此任务。

如果成功创建新的虚拟表，xCreate 应返回 SQLITE_OK，否则返回 SQLITE_ERROR。如果失败，必须不分配 sqlite3_vtab 结构。如果失败，可以选择在 *pzErr 中返回错误消息。用于存储错误消息字符串的空间必须使用 SQLite 内存分配函数（如 sqlite3_malloc() 或 sqlite3_mprintf()）分配，因为 SQLite 核心将尝试在向应用程序报告错误后使用 sqlite3_free() 释放该空间。

如果省略 xCreate 方法（作为 NULL 指针），那么该虚拟表就是一个 仅同名虚拟表。不能使用 CREATE VIRTUAL TABLE 创建虚拟表的新实例，虚拟表只能通过其模块名称使用。请注意，SQLite 3.9.0（2015-10-14）之前的版本不理解仅同名虚拟表，如果尝试在仅同名虚拟表上使用 CREATE VIRTUAL TABLE 会导致 segfault，因为未检查 xCreate 方法是否为 null。

如果 xCreate 方法与 xConnect 方法的确切指针相同，则表明虚拟表不需要初始化后备存储。这样的虚拟表可以用作 eponymous virtual table，或者使用 CREATE VIRTUAL TABLE 命名虚拟表，或者两者都可以。

### 2.1.1\. 虚拟表中的隐藏列

如果列数据类型包含特殊关键字"HIDDEN"（无论大小写组合如何），则该关键字将从列数据类型名称中省略，并且该列在内部标记为隐藏列。隐藏列与普通列有三个不同之处：

+   隐藏列不在由"PRAGMA table_info"返回的数据集中列出，

+   隐藏列不包括在 SELECT 结果集中"*"表达式的展开中，

+   隐藏列不包括在 INSERT 语句的隐式列列表中，该语句缺乏显式列列表。

例如，如果将以下 SQL 传递给 sqlite3_declare_vtab()：

```sql
CREATE TABLE x(a HIDDEN VARCHAR(12), b INTEGER, c INTEGER Hidden);

```

那么虚拟表将创建两个隐藏列，并且数据类型分别为"VARCHAR(12)"和"INTEGER"。

隐藏列的一个示例用法可以在 FTS3 虚拟表实现中看到，其中每个 FTS 虚拟表包含一个 FTS 隐藏列，用于将信息从虚拟表传递到 FTS 辅助函数和 FTS MATCH 操作符。

### 2.1.2\. 表值函数

一个包含隐藏列的虚拟表，可以像在 SELECT 语句的 FROM 子句中使用表值函数一样使用。表值函数的参数成为虚拟表的隐藏列的约束条件。

例如，"generate_series"扩展（位于[ext/misc/series.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/series.c)文件中的[source tree](https://www.sqlite.org/src/tree?ci=trunk)）实现了一个 eponymous virtual table，其架构如下：

```sql
CREATE TABLE generate_series(
  value,
  start HIDDEN,
  stop HIDDEN,
  step HIDDEN
);

```

在此表的实现中，sqlite3_module.xBestIndex 方法检查对隐藏列的等式约束，并将其作为输入参数，以确定生成的整数"value"输出的范围。对于任何未约束的列，使用合理的默认值。例如，要列出介于 5 和 50 之间的所有整数：

```sql
SELECT value FROM generate_series(5,50);

```

前面的查询等效于以下内容：

```sql
SELECT value FROM generate_series WHERE start=5 AND stop=50;

```

虚拟表名称上的参数与 hidden columns 中的隐藏列按顺序匹配。参数数量可以少于虚拟表中隐藏列的数量，此时后面的隐藏列是无约束的。但如果参数比虚拟表中的隐藏列更多，则会导致错误。

### 2.1.3\. WITHOUT ROWID 虚拟表

从 SQLite version 3.14.0（2016-08-08）开始，传递给 sqlite3_declare_vtab() 的 CREATE TABLE 语句可以包含 WITHOUT ROWID 子句。在虚拟表行不能轻松映射到唯一整数的情况下，这非常有用。包含 WITHOUT ROWID 的 CREATE TABLE 语句必须将一个或多个列定义为主键。主键的每一列都必须单独为 NOT NULL，并且每行的所有列必须集体唯一。

请注意，SQLite 不会强制执行 WITHOUT ROWID 虚拟表的主键。强制执行是底层虚拟表实现的责任。但 SQLite 确实假定主键约束有效 - 即已识别的列确实是唯一的和非 NULL 的 - 并且它使用该假设来优化针对虚拟表的查询。

在 WITHOUT ROWID 虚拟表上不可访问 rowid 列（当然）。

xUpdate 方法最初设计时围绕一个 ROWID 作为单个值。xUpdate 方法已扩展以适应任意主键替代 ROWID，但主键仍然必须仅为一列。因此，SQLite 将拒绝具有多于一个主键列和非 NULL xUpdate 方法的 WITHOUT ROWID 虚拟表。

## 2.2\. xConnect 方法

```sql
int (*xConnect)(sqlite3*, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

xConnect 方法与 xCreate 非常相似。它具有相同的参数，并像 xCreate 一样构造一个新的 sqlite3_vtab 结构。它还必须像 xCreate 一样调用 sqlite3_declare_vtab()。它还应该像 xCreate 一样进行所有相同的 sqlite3_vtab_config() 调用。

区别在于 xConnect 被调用以建立对现有虚拟表的新连接，而 xCreate 被调用以从头创建一个新的虚拟表。

当虚拟表具有必须在创建虚拟表时初始化的某种后备存储时，xCreate 和 xConnect 方法才会有所不同。xCreate 方法创建和初始化后备存储。xConnect 方法只是连接到现有的后备存储。当 xCreate 和 xConnect 相同时，表是一个 eponymous virtual table。

例如，考虑一个虚拟表实现，该实现提供对磁盘上现有的逗号分隔值（CSV）文件的只读访问。对于这样一个虚拟表，不需要创建或初始化后备存储（因为 CSV 文件已经存在于磁盘上），因此对于该模块，xCreate 和 xConnect 方法将是相同的。

另一个例子是实现全文索引的虚拟表。xCreate 方法必须创建和初始化用于保存该索引的词典和 posting lists 的数据结构。另一方面，xConnect 方法只需查找并使用之前由 xCreate 调用创建的现有词典和 posting lists。

如果 xConnect 方法成功创建新的虚拟表，则必须返回 SQLITE_OK，如果不成功，则返回 SQLITE_ERROR。如果不成功，则不得分配 sqlite3_vtab 结构。如果不成功，可以选择在 *pzErr 中返回错误消息。必须使用 SQLite 内存分配函数（如 sqlite3_malloc() 或 sqlite3_mprintf()）分配用于保存错误消息字符串的空间，因为 SQLite 核心将尝试使用 sqlite3_free() 释放该空间，在将错误报告到应用程序后。

xConnect 方法对于每个虚拟表实现都是必需的，尽管 xCreate 和 sqlite3_module 对象的 xConnect 指针可能指向同一个函数，如果虚拟表不需要初始化后备存储。

## 2.3\. xBestIndex 方法

SQLite 使用虚拟表模块的 xBestIndex 方法来确定访问虚拟表的最佳方式。xBestIndex 方法的原型如下：

```sql
int (*xBestIndex)(sqlite3_vtab *pVTab, sqlite3_index_info*);

```

SQLite 核心通过填充 sqlite3_index_info 结构的某些字段并将指向该结构的指针作为第二个参数传递给 xBestIndex 方法，与 xBestIndex 方法进行通信。xBestIndex 方法填写此结构的其他字段，形成回复。

```sql
struct sqlite3_index_info {
  /* Inputs */
  const int nConstraint;     /* Number of entries in aConstraint */
  const struct sqlite3_index_constraint {
     int iColumn;              /* Column constrained.  -1 for ROWID */
     unsigned char op;         /* Constraint operator */
     unsigned char usable;     /* True if this constraint is usable */
     int iTermOffset;          /* Used internally - xBestIndex should ignore */
  } *const aConstraint;      /* Table of WHERE clause constraints */
  const int nOrderBy;        /* Number of terms in the ORDER BY clause */
  const struct sqlite3_index_orderby {
     int iColumn;              /* Column number */
     unsigned char desc;       /* True for DESC.  False for ASC. */
  } *const aOrderBy;         /* The ORDER BY clause */

  /* Outputs */
  struct sqlite3_index_constraint_usage {
    int argvIndex;           /* if >0, constraint is part of argv to xFilter */
    unsigned char omit;      /* Do not code a test for this constraint */
  } *const aConstraintUsage;
  int idxNum;                /* Number used to identify the index */
  char *idxStr;              /* String, possibly obtained from sqlite3_malloc */
  int needToFreeIdxStr;      /* Free idxStr using sqlite3_free() if true */
  int orderByConsumed;       /* True if output is already ordered */
  double estimatedCost;      /* Estimated cost of using this index */
  /* Fields below are only available in SQLite 3.8.2 and later */
  sqlite3_int64 estimatedRows;    /* Estimated number of rows returned */
  /* Fields below are only available in SQLite 3.9.0 and later */
  int idxFlags;              /* Mask of SQLITE_INDEX_SCAN_* flags */
  /* Fields below are only available in SQLite 3.10.0 and later */
  sqlite3_uint64 colUsed;    /* Input: Mask of columns used by statement */
};

```

注意 "estimatedRows"、"idxFlags" 和 colUsed 字段上的警告。这些字段分别在 SQLite 版本 3.8.2、3.9.0 和 3.10.0 中添加。任何读取或写入这些字段的扩展必须首先检查正在使用的 SQLite 库版本是否大于或等于适当的版本 — 可能将从 sqlite3_libversion_number() 返回的值与常量 3008002、3009000 和/或 3010000 进行比较。在由旧版本 SQLite 创建的 sqlite3_index_info 结构中尝试访问这些字段的结果是未定义的。

另外，还有一些定义的常量：

```sql
#define SQLITE_INDEX_CONSTRAINT_EQ         2
#define SQLITE_INDEX_CONSTRAINT_GT         4
#define SQLITE_INDEX_CONSTRAINT_LE         8
#define SQLITE_INDEX_CONSTRAINT_LT        16
#define SQLITE_INDEX_CONSTRAINT_GE        32
#define SQLITE_INDEX_CONSTRAINT_MATCH     64
#define SQLITE_INDEX_CONSTRAINT_LIKE      65  /* 3.10.0 and later */
#define SQLITE_INDEX_CONSTRAINT_GLOB      66  /* 3.10.0 and later */
#define SQLITE_INDEX_CONSTRAINT_REGEXP    67  /* 3.10.0 and later */
#define SQLITE_INDEX_CONSTRAINT_NE        68  /* 3.21.0 and later */
#define SQLITE_INDEX_CONSTRAINT_ISNOT     69  /* 3.21.0 and later */
#define SQLITE_INDEX_CONSTRAINT_ISNOTNULL 70  /* 3.21.0 and later */
#define SQLITE_INDEX_CONSTRAINT_ISNULL    71  /* 3.21.0 and later */
#define SQLITE_INDEX_CONSTRAINT_IS        72  /* 3.21.0 and later */
#define SQLITE_INDEX_CONSTRAINT_LIMIT     73  /* 3.38.0 and later */
#define SQLITE_INDEX_CONSTRAINT_OFFSET    74  /* 3.38.0 and later */
#define SQLITE_INDEX_CONSTRAINT_FUNCTION 150  /* 3.25.0 and later */
#define SQLITE_INDEX_SCAN_UNIQUE           1  /* Scan visits at most 1 row */

```

使用 sqlite3_vtab_collation()接口来查找在评估第 i 个约束时应该使用的排序序列的名称：

```sql
const char *sqlite3_vtab_collation(sqlite3_index_info*, int i);

```

当编译涉及虚拟表的 SQL 查询时，SQLite 核心调用 xBestIndex 方法。换句话说，SQLite 在运行 sqlite3_prepare()或等效方法时调用此方法。通过调用此方法，SQLite 核心告诉虚拟表它需要访问虚拟表中的某些行的子集，并且想要知道最有效的访问方式。xBestIndex 方法会回复一些信息，SQLite 核心随后可以用来执行对虚拟表的高效搜索。

在编译单个 SQL 查询时，SQLite 核心可能会以不同的设置多次调用 xBestIndex 方法，来处理 sqlite3_index_info 中的组合。然后 SQLite 核心将选择看起来性能最佳的组合。

在调用此方法之前，SQLite 核心会初始化一个 sqlite3_index_info 结构的实例，其中包含关于当前正在处理的查询的信息。这些信息主要来源于查询的 WHERE 子句以及 ORDER BY 或 GROUP BY 子句，如果查询是一个连接，则还来自任何 ON 或 USING 子句。SQLite 核心提供给 xBestIndex 方法的信息保存在标记为“Inputs”的结构的部分中。“Outputs”部分被初始化为零。

sqlite3_index_info 结构中的信息是短暂的，并且可能在 xBestIndex 方法返回后被覆盖或释放。如果 xBestIndex 方法需要记住 sqlite3_index_info 结构的任何部分，它应该创建一个副本。必须小心地将副本存储在会被释放的地方，例如将 needToFreeIdxStr 设置为 1 的 idxStr 字段中。

注意 xBestIndex 总是会在 xFilter 之前调用，因为 xBestIndex 的 idxNum 和 idxStr 输出是 xFilter 所需的输入。但是，并不保证 xBestIndex 成功后一定会调用 xFilter。

对于每个虚拟表实现，xBestIndex 方法都是必需的。

### 2.3.1\. 输入

SQLite 核心试图传达给虚拟表的主要内容是可用于限制需要搜索的行数的约束。aConstraint[]数组包含每个约束的一个条目。在该数组中会有确切的 nConstraint 条目。

每个约束通常对应于 WHERE 子句中的一个术语，或者形式为 USING 或 ON 子句中的术语

> column OP EXPR

其中"column"是虚拟表中的一列，OP 是像"="或"<"这样的操作符，EXPR 是任意表达式。例如，如果 WHERE 子句包含以下术语：

```sql
a = 5

```

然后，“a”列的一个约束条件是用运算符“=”和表达式“5”定义的。约束条件不需要字面上的 WHERE 子句表示。查询优化器可能会对 WHERE 子句进行转换，以提取尽可能多的约束条件。例如，如果 WHERE 子句包含类似以下内容：

```sql
x BETWEEN 10 AND 100 AND 999>y

```

查询优化器可能会将此转换为三个单独的约束条件：

```sql
x >= 10
x <= 100
y < 999

```

对于每个这样的约束条件，aConstraint[]中的 iColumn 字段指示约束条件左侧出现的列。虚拟表的第一列是列 0。虚拟表的 rowid 是列-1。aConstraint[].op 字段指示使用的操作符。SQLITE_INDEX_CONSTRAINT_*常量将整数常量映射到操作符值。列按照在 xCreate 或 xConnect 方法中对 sqlite3_declare_vtab()的调用定义的顺序出现。在确定列索引时，隐藏列也要计算在内。

如果虚拟表的 xFindFunction()方法已定义，并且如果 xFindFunction()有时返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值，则约束条件也可能采用以下形式：

> FUNCTION( column, EXPR)

在这种情况下，aConstraint[].op 值与 xFindFunction()方法返回的 FUNCTION 相同。

aConstraint[]数组包含有关适用于虚拟表的所有约束条件的信息。但是由于表在连接中的排序方式，某些约束条件可能无法使用。因此，xBestIndex 方法必须仅考虑 aConstraint[].usable 标志为 true 的约束条件。

除了 WHERE 子句约束条件外，SQLite 核心还告诉 xBestIndex 方法关于 ORDER BY 子句的信息。（在聚合查询中，SQLite 核心可能会放入 GROUP BY 子句信息以替换 ORDER BY 子句信息，但这个事实不应影响 xBestIndex 方法。）如果 ORDER BY 子句的所有术语都是虚拟表中的列，则 nOrderBy 将是 ORDER BY 子句中术语的数量，并且 aOrderBy[]数组将标识 ORDER BY 子句中每个术语的列以及该列是否为 ASC 或 DESC。

在 SQLite 3.10.0 版（2016-01-06）及以后，colUsed 字段可用于指示被正在准备的语句实际使用的虚拟表的字段。如果 colUsed 的最低位被设置，这意味着使用了第一列。次低位对应第二列。以此类推。如果 colUsed 的最高位被设置，这意味着使用了除了前 63 列之外的一列或多列。如果 xFilter 方法需要使用列使用信息，则必须将必需的位编码为输出 idxNum 字段或 idxStr 内容中的哪些位。

#### 2.3.1.1\. LIKE、GLOB、REGEXP 和 MATCH 函数

对于 LIKE、GLOB、REGEXP 和 MATCH 运算符，aConstraint[].iColumn 值是运算符的左操作数的虚拟表列。然而，如果这些运算符被表达为函数调用而不是运算符，那么 aConstraint[].iColumn 值引用的是该函数的第二个参数的虚拟表列：

> LIKE(*EXPR*, *column*)
> 
> GLOB(*EXPR*, *column*)
> 
> REGEXP(*EXPR*, *column*)
> 
> MATCH(*EXPR*, *column*)

因此，对于 xBestIndex()方法来说，以下两种形式是等效的：

> *column* 像 *EXPR*
> 
> LIKE(*EXPR*,*column*)

查看函数的第二个参数的这种特殊行为仅适用于 LIKE、GLOB、REGEXP 和 MATCH 函数。对于所有其他函数，aConstraint[].iColumn 值引用的是函数的第一个参数。 

但是这种对 LIKE、GLOB、REGEXP 和 MATCH 的特殊特性并不适用于 xFindFunction()方法。然而，xFindFunction()方法始终基于 LIKE、GLOB、REGEXP 或 MATCH 运算符的左操作数，而不是基于这些运算符的函数调用等效的第一个参数。

#### 2.3.1.2\. LIMIT 和 OFFSET

当 aConstraint[].op 是 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，表示 SQL 查询语句使用了虚拟表的 LIMIT 或 OFFSET 子句。LIMIT 和 OFFSET 运算符没有左操作数，因此当 aConstraint[].op 是 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，aConstraint[].iColumn 的值是无意义的，不应该使用。

#### 2.3.1.3\. 约束的右操作数值

sqlite3_vtab_rhs_value()接口可用于尝试访问约束的右操作数。但是，在执行 xBestIndex 方法时，右操作数的值可能尚不可知，因此 sqlite3_vtab_rhs_value()调用可能不会成功。通常，如果右操作数在输入 SQL 中以文字值编码，则 xBestIndex 仅能访问约束的右操作数。如果右操作数编码为表达式或主机参数，它可能对 xBestIndex 不可用。一些操作符，如 SQLITE_INDEX_CONSTRAINT_ISNULL 和 SQLITE_INDEX_CONSTRAINT_ISNOTNULL，没有右操作数。对于这类操作符，sqlite3_vtab_rhs_value()接口总是返回 SQLITE_NOTFOUND。

### 2.3.2\. 输出

综合以上所有信息，xBestIndex 方法的工作是找出搜索虚拟表的最佳方式。

xBestIndex 方法通过 idxNum 和 idxStr 字段向 xFilter 方法传达索引策略。就 SQLite 核心而言，idxNum 值和 idxStr 字符串内容可以是任意的，并且只要 xBestIndex 和 xFilter 就它们的含义达成一致即可。SQLite 核心只是将信息从 xBestIndex 复制到 xFilter 方法，假设 idxStr 引用的字符序列以 NUL 结尾。

idxStr 值可以是从 SQLite 内存分配函数（如 sqlite3_mprintf()）获得的字符串。如果是这种情况，则必须将 needToFreeIdxStr 标志设置为 true，以便 SQLite 核心在完成使用后调用 sqlite3_free()释放该字符串，从而避免内存泄漏。idxStr 值也可能是静态常量字符串，此时 needToFreeIdxStr 布尔值应保持 false。

estimatedCost 字段应设置为执行该查询对虚拟表所需的估计磁盘访问操作数。SQLite 核心通常会多次调用 xBestIndex，使用不同的约束获取多个成本估计值，然后选择提供最低估计的查询计划。SQLite 核心在调用 xBestIndex 之前将 estimatedCost 初始化为一个非常大的值，因此如果 xBestIndex 确定当前参数组合不理想，它可以保持 estimatedCost 字段不变以阻止其使用。

如果当前版本的 SQLite 是 3.8.2 或更高，则 estimatedRows 字段可以设置为拟议查询计划返回的行数估计值。如果没有显式设置此值，则使用默认的 25 行估计值。

如果当前的 SQLite 版本是 3.9.0 或更高，则可以将 idxFlags 字段设置为 SQLITE_INDEX_SCAN_UNIQUE，以指示虚拟表在给定输入约束条件时仅返回零行或一行。在 SQLite 的后续版本中可能会理解 idxFlags 字段的其他位。

aConstraintUsage[] 数组包含输入部分的 nConstraint 约束的每个元素。aConstraintUsage[] 数组由 xBestIndex 使用，告诉核心它如何使用这些约束。

xBestIndex 方法可以将 aConstraintUsage[].argvIndex 条目设置为大于零的值。应该将精确地一个条目设置为 1，另一个设置为 2，另一个设置为 3，以此类推，直到 xBestIndex 方法想要的条目数。相应约束的 EXPR 将作为 argv[] 参数传递给 xFilter。

例如，如果 aConstraint[3].argvIndex 设置为 1，则在调用 xFilter 时，传递给 xFilter 的 argv[0] 将具有 aConstraint[3] 约束的 EXPR 值。

#### 2.3.2.1\. 在字节码中省略约束检查

默认情况下，SQLite 生成的字节码将在虚拟表的每一行上双重检查所有约束，以验证它们是否满足。如果虚拟表能够保证约束始终满足，它可以尝试通过设置 aConstraintUsage[].omit 来抑制该冗余检查。然而，除了一些例外情况外，这只是一个提示，不能保证冗余约束检查将被抑制。关键点：

+   只有在约束的 argvIndex 值大于 0 且小于或等于 16 时，才会遵循 omit 标志。对于不将其右操作数传递到 xFilter 方法的约束，永远不会抑制约束检查。当前实现只能抑制传递给 xFilter 的前 16 个值的冗余约束检查，尽管该限制可能会在未来的版本中增加。

+   只要 argvIndex 大于 0，对于 SQLITE_INDEX_CONSTRAINT_OFFSET 约束，始终会遵循 omit 标志。在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置 omit 标志表示虚拟表本身将抑制输出的前 N 行，其中 N 是 OFFSET 运算符的右操作数。如果虚拟表实现在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置了 omit，但未能抑制输出的前 N 行，则整体查询将得到错误的答案。

#### 2.3.2.2\. ORDER BY 和 orderByConsumed

如果虚拟表将按照 ORDER BY 子句指定的顺序输出行，则 orderByConsumed 标志可以设置为 true。如果输出不自动按正确顺序，则 orderByConsumed 必须保持其默认 false 设置。这将告知 SQLite 核心在虚拟表输出后需要对数据进行单独的排序传递。设置 orderByConsumed 是一种优化。如果 orderByConsumed 保持其默认值 (0)，则查询将始终获得正确答案。如果设置了 orderByConsumed，可能可以避免不必要的排序操作，从而加快查询速度，但如果错误地设置 orderByConsumed 可能导致错误答案。建议新的虚拟表实现在初始状态下不设置 orderByConsumed 值，然后在确保其他一切正常工作后，返回并尝试通过适当设置 orderByConsumed 来进行优化。

有时候，即使虚拟表的输出不严格按照 nOrderBy 和 aOrderBy 指定的顺序，orderByConsumed 标志也可以安全地设置。如果 sqlite3_vtab_distinct() 接口返回 1 或 2，则表示可以放宽排序要求。有关详细信息，请参阅 sqlite3_vtab_distinct() 的文档。

### 2.3.3\. 返回值

xBestIndex 方法在成功时应返回 SQLITE_OK。如果发生任何致命错误，则应返回适当的错误代码（例如 SQLITE_NOMEM）。

如果 xBestIndex 返回 SQLITE_CONSTRAINT，这并不表示错误。相反，SQLITE_CONSTRAINT 表示指定的输入参数组合对于虚拟表执行其工作是不足够的。逻辑上，这与将 estimatedCost 设置为无穷大是相同的。如果对于特定查询计划的每次 xBestIndex 调用都返回 SQLITE_CONSTRAINT，这意味着虚拟表无法安全使用，而且 sqlite3_prepare() 调用将因“无查询解决方案”错误而失败。

### 2.3.4\. 强制表值函数上的必需参数

来自 xBestIndex 的 SQLITE_CONSTRAINT 返回对于具有必需参数的 表值函数 是有用的。如果对于所需参数之一，aConstraint[].usable 字段为 false，则 xBestIndex 方法应返回 SQLITE_CONSTRAINT。如果一个必需字段在 aConstraint[] 数组中根本不出现，则表示相应参数在输入 SQL 中被省略。在这种情况下，xBestIndex 应在 pVTab->zErrMsg 中设置错误消息，并返回 SQLITE_ERROR。总结如下：

1.  对于所需参数，aConstraint[] 中的 usable 值为 false → 返回 SQLITE_CONSTRAINT。

1.  对于在 aConstraint[] 数组中根本未出现的必需参数 → 在 pVTab->zErrMsg 中设置错误消息，并返回 SQLITE_ERROR。

下面的示例将更好地说明了在 xBestIndex 中使用 SQLITE_CONSTRAINT 作为返回值的情况：

```sql
SELECT * FROM realtab, tablevaluedfunc(realtab.x);

```

假设"tablevaluedfunc"的第一个隐藏列是"param1"，上述查询在语义上等同于此查询：

```sql
SELECT * FROM realtab, tablevaluedfunc
 WHERE tablevaluedfunc.param1 = realtab.x;

```

查询规划器必须在此查询的许多可能实现之间做出决策，但特别值得注意的是两个计划：

1.  扫描 realtab 的所有行，并且对于每一行，在 tablevaluedfunc 中查找 param1 等于 realtab.x 的行。

1.  扫描 tablevaluedfunc 的所有行，并且对于每一行，在 realtab 中查找 x 等于 tablevaluedfunc.param1 的行。

对于上述每个潜在的计划，xBestIndex 方法将被调用一次。对于计划 1，因为"param1 = ?"约束条件的右侧值是已知的，所以 aConstraint[].usable 标志为 SQLITE_CONSTRAINT_EQ 约束在 param1 列上为 true，因为它由外部的 realtab 循环确定。但是对于计划 2，"param1 = ?"的 aConstraint[].usable 标志为 false，因为右侧值由内部循环确定，因此是一个未知量。由于 param1 是表值函数的必需输入，所以当 xBestIndex 方法接收到计划 2 时，应返回 SQLITE_CONSTRAINT，表示缺少必需的输入。这迫使查询规划器选择计划 1。

## 2.4\. xDisconnect 方法

```sql
int (*xDisconnect)(sqlite3_vtab *pVTab);

```

此方法释放对虚拟表的连接。只销毁 sqlite3_vtab 对象。虚拟表不会被销毁，并且与虚拟表关联的任何后备存储都将保持。此方法撤消了 xConnect 的工作。

此方法是连接到虚拟表的析构函数。将此方法与 xDestroy 进行对比。xDestroy 是整个虚拟表的析构函数。

对于每个虚拟表实现，都需要 xDisconnect 方法，尽管 xDisconnect 和 xDestroy 方法可以是同一个函数，如果对于特定的虚拟表而言是有意义的。

## 2.5\. xDestroy 方法

```sql
int (*xDestroy)(sqlite3_vtab *pVTab);

```

此方法释放对虚拟表的连接，就像 xDisconnect 方法一样，并且还销毁底层表实现。此方法撤消了 xCreate 的工作。

每当使用虚拟表的数据库连接关闭时，将调用 xDisconnect 方法。仅在执行 DROP TABLE 语句针对虚拟表时，才会调用 xDestroy 方法。

对于每个虚拟表实现，都需要 xDestroy 方法，尽管 xDisconnect 和 xDestroy 方法可以是同一个函数，如果对于特定的虚拟表而言是有意义的。

## 2.6\. xOpen 方法

```sql
int (*xOpen)(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor);

```

xOpen 方法创建用于访问（读取和/或写入）虚拟表的新游标。成功调用此方法将为 sqlite3_vtab_cursor（或其子类）分配内存，初始化新对象，并使 *ppCursor 指向新对象。成功调用将返回 SQLITE_OK。

每次成功调用此方法后，SQLite 核心稍后将调用 xClose 方法来销毁分配的游标。

xOpen 方法不需要初始化 sqlite3_vtab_cursor 结构的 pVtab 字段。SQLite 核心将自动处理这一步骤。

虚拟表实现必须能够支持任意数量的同时打开游标。

初始打开时，游标处于未定义状态。SQLite 核心将在尝试定位或读取游标之前调用游标上的 xFilter 方法。

每个虚拟表实现都需要 xOpen 方法。

## 2.7\. xClose 方法

```sql
int (*xClose)(sqlite3_vtab_cursor*);

```

xClose 方法关闭由 xOpen 打开的游标。SQLite 核心将始终为每个使用 xOpen 打开的游标调用一次 xClose。

此方法必须释放由相应 xOpen 调用分配的所有资源。即使返回错误，该例程也不会再次被调用。SQLite 核心在关闭后不会再使用 sqlite3_vtab_cursor。

每个虚拟表实现都需要 xClose 方法。

## 2.8\. xEof 方法

```sql
int (*xEof)(sqlite3_vtab_cursor*);

```

如果指定的游标当前指向有效数据行，则 xEof 方法必须返回 false（零），否则返回 true（非零）。SQL 引擎在每次 xFilter 和 xNext 调用之后立即调用此方法。

每个虚拟表实现都需要 xEof 方法。

## 2.9\. xFilter 方法

```sql
int (*xFilter)(sqlite3_vtab_cursor*, int idxNum, const char *idxStr,
              int argc, sqlite3_value **argv);

```

此方法开始对虚拟表进行搜索。第一个参数是由 xOpen 打开的游标。接下来的两个参数定义了先前由 xBestIndex 选择的特定搜索索引。idxNum 和 idxStr 的具体含义并不重要，只要 xFilter 和 xBestIndex 对其含义达成一致即可。

函数 xBestIndex 可能已经使用了 sqlite3_index_info 结构中的 aConstraintUsage[].argvIndex 值来请求某些表达式的值。这些值将通过 argc 和 argv 参数传递给 xFilter 方法。

如果虚拟表包含与搜索条件匹配的一个或多个行，则游标必须停在第一行。后续对 xEof 的调用必须返回 false（零）。如果没有匹配的行，则游标必须处于导致 xEof 返回 true（非零）的状态。SQLite 引擎将使用 xColumn 和 xRowid 方法访问该行内容。将使用 xNext 方法前进到下一行。

如果成功，此方法必须返回 SQLITE_OK，否则返回一个 sqlite 错误码。

每个虚拟表实现都需要 xFilter 方法。

## 2.10\. The xNext Method

```sql
int (*xNext)(sqlite3_vtab_cursor*);

```

xNext 方法将虚拟表游标前进到由 xFilter 启动的结果集的下一行。如果此例程调用时游标已指向最后一行，则游标不再指向有效数据，后续对 xEof 方法的调用必须返回 true（非零）。如果成功将游标前进到另一行内容，则后续对 xEof 的调用必须返回 false（零）。

如果成功，此方法必须返回 SQLITE_OK，否则返回一个 sqlite 错误码。

每个虚拟表实现都需要 xNext 方法。

## 2.11\. The xColumn Method

```sql
int (*xColumn)(sqlite3_vtab_cursor*, sqlite3_context*, int N);

```

SQLite 核心调用此方法以查找当前行的第 N 列的值。N 从零开始，因此第一列编号为 0。xColumn 方法可以使用以下接口将其结果返回给 SQLite：

+   sqlite3_result_blob()

+   sqlite3_result_double()

+   sqlite3_result_int()

+   sqlite3_result_int64()

+   sqlite3_result_null()

+   sqlite3_result_text()

+   sqlite3_result_text16()

+   sqlite3_result_text16le()

+   sqlite3_result_text16be()

+   sqlite3_result_zeroblob()

如果 xColumn 方法的实现不调用上述任何函数，则列的值默认为 SQL NULL。

要引发错误，xColumn 方法应使用 result_text()方法之一设置错误消息文本，然后返回适当的错误码。xColumn 方法在成功时必须返回 SQLITE_OK。

每个虚拟表实现都需要 xColumn 方法。

## 2.12\. The xRowid Method

```sql
int (*xRowid)(sqlite3_vtab_cursor *pCur, sqlite_int64 *pRowid);

```

此方法的成功调用将导致 *pRowid 填充为虚拟表光标 pCur 当前指向的行的 rowid。此方法在成功时返回 SQLITE_OK。在失败时返回适当的 错误代码。

每个虚拟表实现都需要 xRowid 方法。

## 2.13\. xUpdate 方法

```sql
int (*xUpdate)(
  sqlite3_vtab *pVTab,
  int argc,
  sqlite3_value **argv,
  sqlite_int64 *pRowid
);

```

所有对虚拟表的更改都使用 xUpdate 方法进行。这一方法可用于插入、删除或更新。

argc 参数指定 argv 数组中的条目数。对于纯删除操作，argc 的值为 1；对于插入、替换或更新操作，argc 的值为 N+2，其中 N 是表中的列数。在前述句子中，N 包括任何隐藏列。

每个 argv 条目在 C 中都有非 NULL 值，但可能包含 SQL 值 NULL。换句话说，对于 **i** 在 0 到 `argc-1` 之间，始终成立 `argv[i]!=0`。但可能存在 `sqlite3_value_type(argv[i])==SQLITE_NULL` 的情况。

argv[0] 参数是要删除的虚拟表中的一行的 rowid。如果 argv[0] 是 SQL NULL，则不进行删除。

argv[1] 参数是要插入到虚拟表中的新行的 rowid。如果 argv[1] 是 SQL NULL，则实现必须为新插入的行选择一个 rowid。随后的 argv[] 条目按照声明列的顺序包含虚拟表的列的值。列数将与 xConnect 或 xCreate 方法使用的 sqlite3_declare_vtab() 调用声明的表声明匹配。所有隐藏列都包括在内。

在进行没有行 id 的插入操作（argc>1 且 argv[1] 是 SQL NULL）时，在使用 ROWID 的虚拟表上（但不适用于 WITHOUT ROWID 虚拟表），实现必须将 *pRowid 设置为新插入行的行 id；这将成为 sqlite3_last_insert_rowid() 函数返回的值。在其他情况下设置此值是无害的空操作；如果 argc==1 或 argv[1] 不是 SQL NULL，则 SQLite 引擎会忽略 *pRowid 返回值。

每次调用 xUpdate 都会落入下面显示的一个情况中。注意，对 **argv[i]** 的引用指的是 argv[i] 对象中保存的 SQL 值，而不是 argv[i] 对象本身。

> **argc = 1
> 
> argv[0] ≠ NULL**
> 
> DELETE：删除具有与 argv[0] 相等的 rowid 或主键的单行。不进行插入操作。
> 
> **argc > 1
> 
> argv[0] = NULL**
> 
> 插入：将从 argv[2]和后续参数中获取的列值插入新行。在 rowid 虚拟表中，如果 argv[1]是 SQL NULL，则会自动生成一个新的唯一 rowid。对于 WITHOUT ROWID 虚拟表，argv[1]将为空，此时实现应该从 argv[2]和后续参数中的适当列获取主键值。
> 
> **argc > 1
> 
> argv[0] ≠ NULL
> 
> argv[0] = argv[1]**
> 
> 更新：带有 rowid 或主键 argv[0]的行使用 argv[2]和后续参数中的新值进行更新。
> 
> **argc > 1
> 
> argv[0] ≠ NULL
> 
> argv[0] ≠ argv[1]**
> 
> 更新行 id 或主键更新：带有 rowid 或主键 argv[0]的行将使用 argv[1]中的 rowid 或主键以及 argv[2]和后续参数中的新值进行更新。当 SQL 语句更新 rowid，如下 SQL 语句：
> 
> > 更新 表设置 rowid=rowid+1 WHERE ...;

如果 xUpdate 方法成功，则必须返回 SQLITE_OK。如果发生失败，xUpdate 必须返回适当的 错误代码。失败时，pVTab->zErrMsg 元素可以选择性地替换为从 SQLite 使用 sqlite3_mprintf() 或 sqlite3_malloc() 分配的内存中存储的错误消息文本。

如果 xUpdate 方法违反了虚拟表的某些约束（包括但不限于尝试存储错误数据类型的值，尝试存储太大或太小的值，或尝试更改只读值），那么 xUpdate 必须以适当的错误代码失败。

如果 xUpdate 方法执行 UPDATE，那么可以使用 sqlite3_value_nochange(X) 来发现虚拟表的哪些列实际上被 UPDATE 语句修改。sqlite3_value_nochange(X) 接口返回对于没有改变的列为真。在每次 UPDATE 时，SQLite 首先会分别为表中每个未改变的列调用 xColumn 方法以获取该列的值。xColumn 方法可以通过调用 sqlite3_vtab_nochange() 来检查该列在 SQL 级别上是否未改变。如果 xColumn 看到列没有被修改，它应该在不使用 sqlite3_result_xxxxx() 接口设置结果的情况下返回。只有在这种情况下，xUpdate 方法内的 sqlite3_value_nochange() 才会为真。如果 xColumn 调用一个或多个 sqlite3_result_xxxxx() 接口，则 SQLite 会将其视为列值的更改，xUpdate 内的 sqlite3_value_nochange() 对该列的调用将返回假。

当 xUpdate 方法被调用时，虚拟表实例上可能有一个或多个正在使用的 sqlite3_vtab_cursor 对象，甚至可能是虚拟表行上的行。xUpdate 的实现必须准备好处理试图从其他现有游标删除或修改表行的尝试。如果虚拟表无法容纳此类更改，则 xUpdate 方法必须返回一个错误代码。

xUpdate 方法是可选的。如果虚拟表的 sqlite3_module 中的 xUpdate 指针是空指针，则该虚拟表是只读的。

## 2.14\. xFindFunction 方法

```sql
int (*xFindFunction)(
  sqlite3_vtab *pVtab,
  int nArg,
  const char *zName,
  void (**pxFunc)(sqlite3_context*,int,sqlite3_value**),
  void **ppArg
);

```

此方法在 sqlite3_prepare()期间调用，以便虚拟表实现有机会重载函数。如果此方法被设置为 NULL，则不进行重载操作。

当函数使用虚拟表的列作为其第一个参数时，将调用此方法以查看虚拟表是否希望重载该函数。前三个参数是输入：虚拟表、函数的参数数量以及函数的名称。如果不希望进行重载，则此方法返回 0。要重载函数，此方法将新的函数实现写入*pxFunc，并将用户数据写入*ppArg，并返回 1 或介于 SQLITE_INDEX_CONSTRAINT_FUNCTION 和 255 之间的数字。

历史上，xFindFunction() 的返回值通常是零或一。零表示函数未重载，而一表示函数已重载。在 3.25.0 版本（2018-09-15）中，新增了返回值可以是 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的能力。如果 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值，则表示该函数接受两个参数，并且可以在查询的 WHERE 子句中作为布尔值使用，虚拟表能够利用该函数来加速查询结果。当 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大时，返回的值将成为传递给 xBestIndex()的一个约束的 sqlite3_index_info.aConstraint.op 值。函数的第一个参数是约束的 aConstraint[].iColumn 字段标识的列，第二个参数是将传递给 xFilter()（如果 aConstraintUsage[].argvIndex 值已设置）或从 sqlite3_vtab_rhs_value()返回的值。

Geopoly 模块 是一个使用 SQLITE_INDEX_CONSTRAINT_FUNCTION 来提高性能的虚拟表的示例。Geopoly 的 xFindFunction() 方法返回 SQLITE_INDEX_CONSTRAINT_FUNCTION，用于 geopoly_overlap() SQL 函数，并为 geopoly_within() SQL 函数返回 SQLITE_INDEX_CONSTRAINT_FUNCTION+1。这允许针对如下查询进行搜索优化：

```sql
SELECT * FROM geopolytab WHERE geopoly_overlap(_shape, $query_polygon);
SELECT * FROM geopolytab WHERE geopoly_within(_shape, $query_polygon);

```

注意中缀函数（LIKE、GLOB、REGEXP 和 MATCH）颠倒了参数的顺序。因此，“like(A,B)”通常与“B like A”相同。但是，xFindFunction() 总是查看最左边的参数，而不是第一个逻辑参数。因此，对于形式“B like A”，SQLite 查看左操作数“B”，如果该操作数是虚拟表列，则在该虚拟表上调用 xFindFunction() 方法。但如果使用的是形式“like(A,B)”，那么 SQLite 将检查 A 项，看看它是否是虚拟表的列，并在是的情况下为列 A 的虚拟表调用 xFindFunction() 方法。

该例程返回的函数指针必须对给定的 sqlite3_vtab 对象的生命周期有效。

## 2.15\. **xBegin** 方法

```sql
int (*xBegin)(sqlite3_vtab *pVTab);

```

该方法在虚拟表上开始一个事务。此方法是可选的。sqlite3_module 的 xBegin 指针可能为 NULL。

该方法始终在调用 xCommit 或 xRollback 方法之一后紧随其后。虚拟表事务不嵌套，因此在单个虚拟表上不会多次调用 xBegin 方法，除非在 xCommit 或 xRollback 之间进行了调用。可能会在 xBegin 和相应的 xCommit 或 xRollback 之间发生多次对其他方法的调用。

## 2.16\. **xSync** 方法

```sql
int (*xSync)(sqlite3_vtab *pVTab);

```

该方法标志着在虚拟表上进行两阶段提交的开始。此方法是可选的。sqlite3_module 的 xSync 指针可能为 NULL。

此方法仅在调用 xBegin 方法后，并在 xCommit 或 xRollback 之前调用。为了实现两阶段提交，将在调用任何虚拟表的 xCommit 方法之前调用所有虚拟表的 xSync 方法。如果任何 xSync 方法失败，则整个事务将被回滚。

## 2.17\. **xCommit** 方法

```sql
int (*xCommit)(sqlite3_vtab *pVTab);

```

该方法导致虚拟表事务提交。此方法是可选的。sqlite3_module 的 xCommit 指针可能为 NULL。

对该方法的调用始终在之前调用 xBegin 和 xSync 方法之后。

## 2.18\. **xRollback** 方法

```sql
int (*xRollback)(sqlite3_vtab *pVTab);

```

这种方法导致虚拟表事务回滚。这种方法是可选的。sqlite3_module 的 xRollback 指针可能为 NULL。

对这种方法的调用总是在先前调用 xBegin 之后进行。

## 2.19\. xRename 方法

```sql
int (*xRename)(sqlite3_vtab *pVtab, const char *zNew);

```

这种方法提供了通知虚拟表实现虚拟表将获得新名称的功能。如果此方法返回 SQLITE_OK，则 SQLite 将重命名表。如果此方法返回错误代码，则将阻止重命名。

xRename 方法是可选的。如果省略，则无法使用 ALTER TABLE RENAME 命令重命名虚拟表。

在调用此方法之前启用了 PRAGMA legacy_alter_table 设置，并在此方法完成后恢复了 legacy_alter_table 的值。这对于那些使用影子表的虚拟表的正确操作是必要的，其中影子表必须被重命名以匹配新的虚拟表名称。如果 legacy_alter_format 关闭，则每次 xRename 方法尝试更改影子表名称时都会调用 xConnect 方法来为虚拟表建立连接。

## 2.20\. xSavepoint, xRelease 和 xRollbackTo 方法

```sql
int (*xSavepoint)(sqlite3_vtab *pVtab, int);
int (*xRelease)(sqlite3_vtab *pVtab, int);
int (*xRollbackTo)(sqlite3_vtab *pVtab, int);

```

这些方法为虚拟表实现提供了实现嵌套事务的机会。它们始终是可选的，并且只会在 SQLite 版本 3.7.7（2011-06-23）及更高版本中调用。

当调用 xSavepoint(X,N)时，这是一个信号，告诉虚拟表 X 应将其当前状态保存为保存点 N。随后对 xRollbackTo(X,R)的调用意味着虚拟表的状态应返回到上次调用 xSavepoint(X,R)时的状态。对 xRollbackTo(X,R)的调用将使所有 N>R 的保存点无效；没有无效的保存点会在首先通过调用 xSavepoint()重新初始化之前回滚或释放。对 xRelease(X,M)的调用使所有 N>=M 的保存点无效。

xSavepoint()，xRelease()，或 xRollbackTo()方法不会在调用 xBegin()和 xCommit()或 xRollback()之间以外的任何时候调用。

## 2.21\. xShadowName 方法

一些虚拟表实现（例如：FTS3，FTS5，和 RTREE）利用真实（非虚拟）数据库表来存储内容。例如，当内容插入到 FTS3 虚拟表时，数据最终存储在名为"%_content"，"%_segdir"，"%_segments"，"%_stat"和"%_docsize"的真实表中，其中"%"是原始虚拟表的名称。存储虚拟表内容的这些辅助真实表被称为"影子表"。参见（1），（2），和（3）以获取更多信息。

xShadowName 方法存在的目的是允许 SQLite 确定某个真实表是否确实是虚拟表的阴影表。

如果以下所有条件都满足，SQLite 将理解真实表为阴影表：

+   表的名称中包含一个或多个"_"字符。

+   名称中最后一个"_"字符之前的部分与使用 CREATE VIRTUAL TABLE 创建的虚拟表的名称完全匹配。（对于 eponymous virtual tables 和 table-valued functions，不会识别阴影表。）

+   虚拟表包含 xShadowName 方法。

+   当 xShadowName 方法的输入为表名称最后一个"_"字符之后的部分时，xShadowName 方法返回 true。

如果 SQLite 识别某个表为阴影表，并且设置了 SQLITE_DBCONFIG_DEFENSIVE 标志，那么该阴影表对普通 SQL 语句而言是只读的。阴影表仍然可以被写入，但只能由某个虚拟表实现的方法中调用的 SQL 进行。

xShadowName 方法的主要目的是防止敌对 SQL 损坏阴影表的内容。每个使用阴影表的虚拟表实现都应能够检测和处理已损坏的阴影表内容。然而，特定虚拟表实现中的错误可能允许有意损坏的阴影表导致崩溃或其他故障。xShadowName 机制旨在通过防止普通 SQL 语句有意损坏阴影表来避免零日漏洞利用。

阴影表默认为读/写模式。只有在使用 sqlite3_db_config()设置了 SQLITE_DBCONFIG_DEFENSIVE 标志后，阴影表才会变为只读。为了保持向后兼容性，阴影表需要默认为读/写模式。例如，CLI 的.dump 命令生成的 SQL 文本直接写入阴影表。

## 2.22\. xIntegrity 方法

如果一个 sqlite3_module 的 iVersion 为 4 或更高，并且 xIntegrity 方法不为 NULL，则 PRAGMA integrity_check 和 PRAGMA quick_check 命令将在处理过程中调用 xIntegrity。如果 xIntegrity 方法将错误消息字符串写入第五个参数，则 PRAGMA integrity_check 将将该错误作为其输出的一部分报告。换句话说，xIntegrity 方法允许 PRAGMA integrity_check 命令验证存储在虚拟表中的内容的完整性。

xIntegrity 方法被调用时带有五个参数：

+   **pVTab** → 指向被检查的虚拟表对象 sqlite3_vtab 的指针。

+   **zSchema** → 定义虚拟表的模式的名称（"main"、"temp"等）。

+   **zTabName** → 虚拟表的名称。

+   **mFlags** → 一个标志，指示这是一个 "integrity_check" 还是 "quick_check"。当前，该参数始终是 0 或 1，尽管 SQLite 的未来版本可能会使用整数的其他位来指示额外的处理选项。

+   **pzErr** → 此参数指向一个初始化为 NULL 的 "char*"。如果 xIntegrity() 发现任何问题，则实现应使 *pzErr 指向从 sqlite3_malloc() 或其等效函数获取的错误字符串。

xIntegrity 方法通常应返回 SQLITE_OK — 即使它在虚拟表内容中发现问题。任何其他错误代码意味着 xIntegrity 方法本身在尝试评估虚拟表内容时遇到问题。例如，如果发现 FTS5 的倒排索引内部不一致，则 xIntegrity 方法应向 pzErr 参数写入适当的错误消息并返回 SQLITE_OK。但是，如果 xIntegrity 方法由于内存耗尽而无法完成对虚拟表内容的评估，则应返回 SQLITE_NOMEM。

如果生成错误消息，则应从 sqlite3_malloc64()或其等效函数获取空间以保存错误消息字符串。当 xIntegrity 返回时，错误消息字符串的所有权将传递给 SQLite 核心。核心会确保在完成对错误消息的使用后调用 sqlite3_free()以回收内存。调用 xIntegrity 方法的 PRAGMA integrity_check 命令不会改变返回的错误消息。xIntegrity 方法本身应该将虚拟表的名称作为消息的一部分。为了更容易做到这一点，提供了 zSchema 和 zName 参数。

mFlags 参数当前是一个布尔值（0 或 1），指示 xIntegrity 方法是由于 PRAGMA integrity_check（mFlags==0）还是由于 PRAGMA quick_check（mFlags==1）而被调用。一般来说，xIntegrity 方法应尽可能在线性时间内完成所有有效性检查，但仅在 `(mFlags&1)==0` 时才执行需要超线性时间的检查。SQLite 的未来版本可能会使用 mFlags 参数的高阶位来指示额外的处理选项。

在 SQLite 版本 3.44.0 (2023-11-01) 中添加了对 xIntegrity 方法的支持。在同一版本中，xIntegrity 方法还添加到了许多内置虚拟表中，如 FTS3，FTS5 和 RTREE，因此从现在开始，在运行 PRAGMA integrity_check 时将自动检查这些表的内容是否一致。
