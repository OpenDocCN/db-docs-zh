# 1\. 介绍

> 原文：[`sqlite.com/vtab.html#tabfunc2`](https://sqlite.com/vtab.html#tabfunc2)

虚拟表是注册到打开的 SQLite 数据库连接中的对象。从 SQL 语句的角度来看，虚拟表对象看起来像任何其他表或视图。但在幕后，对虚拟表的查询和更新调用虚拟表对象的回调方法，而不是直接读取和写入数据库文件。

虚拟表机制允许应用程序发布可从 SQL 语句访问的接口，就像它们是表一样。SQL 语句对虚拟表几乎可以像对真实表一样做任何事情，但有以下例外：

+   无法在虚拟表上创建触发器。

+   无法在虚拟表上创建额外的索引。（虚拟表可以有索引，但必须构建到虚拟表实现中。不能使用 CREATE INDEX 语句单独添加索引。）

+   无法对虚拟表运行 ALTER TABLE ... ADD COLUMN 命令。

单独的虚拟表实现可能会施加额外的约束。例如，某些虚拟实现可能提供只读表。或者某些虚拟表实现可能允许 INSERT 或 DELETE，但不允许 UPDATE。或者某些虚拟表实现可能限制可以进行的 UPDATE 类型。

虚拟表可以表示内存中的数据结构。或者它可以表示磁盘上不是 SQLite 格式的数据的视图。或者应用程序可以按需计算虚拟表的内容。

以下是一些现有的和推测的虚拟表用途：

+   全文搜索接口

+   使用 R-Tree 进行空间索引

+   检查 SQLite 数据库文件的磁盘内容（dbstat 虚拟表）

+   读取和/或写入逗号分隔值（CSV）文件的内容

+   访问主机计算机的文件系统，就像它是数据库表一样

+   启用对像 R 这样的统计包中的数据进行 SQL 操作

请查看 virtual tables 列表页面，获取更长的实际虚拟表实现列表。

## 1.1\. 使用方法

使用 CREATE VIRTUAL TABLE 语句创建虚拟表。

**create-virtual-table-stmt:**

<svg class="pikchr" viewBox="0 0 624.096 259.848"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CREATE</text> <text x="183" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VIRTUAL</text> <text x="286" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">TABLE</text> <text x="372" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IF</text> <text x="435" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="521" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">EXISTS</text> <text x="95" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">schema-name</text> <text x="197" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="300" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">table-name</text> <text x="67" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">USING</text> <text x="187" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">module-name</text> <text x="300" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="432" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">module-argument</text> <text x="563" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="432" y="242" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

CREATE VIRTUAL TABLE 语句创建一个名为 table-name 的新表，该表派生自类模块名称。module-name 是由 sqlite3_create_module()接口注册为虚拟表的名称。

```sql
CREATE VIRTUAL TABLE tablename USING modulename;

```

在模块名称后面，还可以提供逗号分隔的参数。

```sql
CREATE VIRTUAL TABLE tablename USING modulename(arg1, arg2, ...);

```

模块参数的格式非常通用。每个模块参数可能包含关键字、字符串字面量、标识符、数字和标点符号。每个模块参数在创建虚拟表时作为文本传递给虚拟表实现的构造方法，构造方法负责解析和解释参数。参数语法足够通用，以至于虚拟表实现可以将其参数解释为普通 CREATE TABLE 语句中的列定义。实现也可以对参数施加其他解释。

一旦创建了虚拟表，它可以像任何其他表一样使用，但特定虚拟表实现可能会施加一些上述未提到的异常。使用普通 DROP TABLE 语法销毁虚拟表。

### 1.1.1\. 临时虚拟表

没有"CREATE TEMP VIRTUAL TABLE"语句。要创建临时虚拟表，请在虚拟表名称前加上"temp"模式。

```sql
CREATE VIRTUAL TABLE temp.tablename USING module(arg1, ...);

```

### 1.1.2\. 同名虚拟表

一些虚拟表自动存在于每个数据库连接的"main"模式中，即使没有 CREATE VIRTUAL TABLE 语句也是如此。这些虚拟表称为"同名虚拟表"。要使用同名虚拟表，只需将模块名称用作表名即可。同名虚拟表仅存在于"main"模式中，因此如果使用不同模式名称作为前缀，则无法正常工作。

一个同名虚拟表的例子是 dbstat 虚拟表。要将 dbstat 虚拟表作为同名虚拟表使用，只需针对"dbstat"模块名称查询，就像查询普通表一样。（请注意，SQLite 必须使用 SQLITE_ENABLE_DBSTAT_VTAB 选项编译，才能在构建中包括 dbstat 虚拟表。）

```sql
SELECT * FROM dbstat;

```

如果其 xCreate 方法与 xConnect 方法完全相同，或者 xCreate 方法为 NULL，则虚拟表称为同名虚拟表。在使用 CREATE VIRTUAL TABLE 语句首次创建虚拟表时，会调用 xCreate 方法。每当数据库连接附加到或重新解析模式时，都会调用 xConnect 方法。当这两个方法相同时，表明虚拟表没有需要创建和销毁的持久状态。

### 1.1.3\. 仅同名虚拟表

如果 xCreate 方法为 NULL，则禁止对该虚拟表执行 CREATE VIRTUAL TABLE 语句，并且该虚拟表为"仅同名虚拟表"。仅同名虚拟表可用作表值函数。

注意，在版本 3.9.0（2015-10-14）之前，SQLite 在调用 xCreate 方法之前未检查其是否为 NULL。因此，如果在 SQLite 版本 3.8.11.1（2015-07-29）或更早版本中注册了仅同名虚拟表，并尝试对该虚拟表模块执行 CREATE VIRTUAL TABLE 命令，则会发生跳转到 NULL 指针的情况，导致崩溃。

## 1.2\. 实现

虚拟表实现使用了几个新的 C 级对象：

```sql
typedef struct sqlite3_vtab sqlite3_vtab;
typedef struct sqlite3_index_info sqlite3_index_info;
typedef struct sqlite3_vtab_cursor sqlite3_vtab_cursor;
typedef struct sqlite3_module sqlite3_module;

```

sqlite3_module 结构定义了一个用于实现虚拟表的模块对象。将模块视为可以构建具有类似属性的多个虚拟表的类。例如，可以有一个模块提供对磁盘上逗号分隔值（CSV）文件的只读访问。然后可以使用该模块创建若干虚拟表，其中每个虚拟表都引用不同的 CSV 文件。

模块结构包含通过 SQLite 调用的方法，执行虚拟表上的各种操作，如创建虚拟表的新实例或销毁旧实例，读写数据，搜索和删除、更新或插入行。模块结构将在下文中详细解释。

每个虚拟表实例由 sqlite3_vtab 结构表示。sqlite3_vtab 结构的样子如下：

```sql
struct sqlite3_vtab {
  const sqlite3_module *pModule;
  int nRef;
  char *zErrMsg;
};

```

虚拟表实现通常会对此结构进行子类化，以添加额外的私有和特定实现的字段。nRef 字段在 SQLite 核心内部使用，不应由虚拟表实现更改。虚拟表实现可以通过在 zErrMsg 中放置错误消息字符串来将错误消息文本传递给核心。必须从 SQLite 内存分配函数（如 sqlite3_mprintf() 或 sqlite3_malloc()）获取空间来保存此错误消息字符串。在对 zErrMsg 赋予新值之前，虚拟表实现必须使用 sqlite3_free() 释放 zErrMsg 的任何现有内容。未能执行此操作将导致内存泄漏。当核心向客户应用程序传递错误消息文本或销毁虚拟表时，核心将释放并清零 zErrMsg 的内容。虚拟表实现只需要在使用新的、不同的错误消息覆盖内容时担心释放 zErrMsg 的内容。

sqlite3_vtab_cursor 结构表示指向虚拟表特定行的指针。这就是 sqlite3_vtab_cursor 的样子：

```sql
struct sqlite3_vtab_cursor {
  sqlite3_vtab *pVtab;
};

```

再一次，实际的实现很可能会对此结构进行子类化，以添加额外的私有字段。

sqlite3_index_info 结构用于在实现虚拟表的模块的 xBestIndex 方法中传递信息。

在运行 CREATE VIRTUAL TABLE 语句之前，必须使用该语句中指定的模块将其注册到数据库连接。可以使用 sqlite3_create_module() 或 sqlite3_create_module_v2() 接口来实现这一点：

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

sqlite3_create_module() 和 sqlite3_create_module_v2() 程序将模块名与 sqlite3_module 结构及专用于每个模块的客户端数据关联起来。 两种 create_module 方法之间唯一的区别在于 _v2 方法包含了一个额外的参数，用于指定客户端数据指针的析构函数。 模块结构定义了虚拟表的行为。 模块结构如下所示：

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

模块结构定义了每个虚拟表对象的所有方法。 模块结构还包含了 iVersion 字段，用于定义模块表结构的特定版本。 目前，iVersion 总是 4 或更低，但在 SQLite 的未来版本中，模块结构定义可能会扩展到包括额外的方法，在这种情况下，最大的 iVersion 值将会增加。

模块结构的其余部分包括用于实现虚拟表各种特性的方法。 关于每个方法的详细信息将在后续部分提供。

## 1.3\. 虚拟表与共享缓存

在 SQLite 版本 3.6.17（2009-08-10）之前，虚拟表机制假设每个 数据库连接 都保存其自己的数据库架构副本。 因此，启用 共享缓存模式 的数据库中不能使用虚拟表机制。 如果启用了 共享缓存模式，sqlite3_create_module() 接口将返回错误。 从 SQLite 版本 3.6.17 开始，该限制得到了放宽。

## 1.4\. 创建新的虚拟表实现

按照以下步骤创建您自己的虚拟表：

1.  编写所有必要的方法。

1.  创建一个包含指向步骤 1 所有方法的 sqlite3_module 结构的实例。

1.  使用其中一个接口 sqlite3_create_module() 或 sqlite3_create_module_v2() 注册您的 sqlite3_module 结构。

1.  运行一个指定了新模块的 CREATE VIRTUAL TABLE 命令，使用 USING 子句。

唯一真正困难的部分是步骤 1。您可能希望从现有的虚拟表实现开始，并修改以满足您的需求。 [SQLite 源代码树](https://sqlite.org/src/dir?ci=trunk&type=tree) 包含许多适合复制的虚拟表实现，包括：

+   **[templatevtab.c](https://sqlite.org/src/file/ext/misc/templatevtab.c)** → 一个专门用于作为其他自定义虚拟表模板的虚拟表。

+   **[series.c](https://sqlite.org/src/file/ext/misc/series.c)** → 生成 generate_series() 表值函数的实现。

+   **[json.c](https://sqlite.org/src/file/src/json.c)** → 包含 json_each() 和 json_tree() 表值函数的源代码。

+   **[csv.c](https://sqlite.org/src/file/ext/misc/csv.c)** → 一个读取 CSV 文件的虚拟表。

SQLite 源代码树中有许多其他虚拟表实现，可以作为示例使用。通过搜索 "sqlite3_create_module" 可以找到这些其他虚拟表实现。

您可能还希望将您的新虚拟表实现为可加载扩展。

# 2\. 虚拟表方法

## 2.1\. xCreate 方法

```sql
int (*xCreate)(sqlite3 *db, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

当 xCreate 方法与 xConnect 方法的指针相同时，将调用 xCreate 方法来响应 CREATE VIRTUAL TABLE 语句以创建虚拟表的新实例。 如果省略 xCreate 方法（如果它是一个 NULL 指针），那么虚拟表将是一个仅名义虚拟表。

db 参数是执行 CREATE VIRTUAL TABLE 语句的 SQLite 数据库连接 的指针。 pAux 参数是注册 虚拟表模块 的 sqlite3_create_module() 或 sqlite3_create_module_v2() 调用的第四个参数的客户端数据指针副本。 argv 参数是指向空终止字符串的 argc 指针数组。 第一个字符串，argv[0]，是调用的模块名称。 模块名称是作为 sqlite3_create_module() 的第二个参数提供的名称，并作为 CREATE VIRTUAL TABLE 语句中 USING 子句的参数运行。 数组的第二个元素，argv[1]，是正在创建新虚拟表的数据库的名称。 主数据库的名称为 "main"，TEMP 数据库为 "temp"，或者是 ATTACH 语句末尾指定的附加数据库的名称。 数组的第三个元素，argv[2]，是在 CREATE VIRTUAL TABLE 语句中 TABLE 关键字后指定的新虚拟表的名称。 如果存在，argv[] 数组中的第四个及后续字符串报告了 CREATE VIRTUAL TABLE 语句中模块名称的参数。

此方法的任务是构造新的虚拟表对象（一个 sqlite3_vtab 对象），并将指向它的指针返回给 *ppVTab。

作为创建新的 sqlite3_vtab 结构的一部分，此方法必须调用 sqlite3_declare_vtab() 来告知 SQLite 核心有关虚拟表中列和数据类型的信息。 sqlite3_declare_vtab() API 的原型如下：

```sql
int sqlite3_declare_vtab(sqlite3 *db, const char *zCreateTable)

```

sqlite3_declare_vtab() 的第一个参数必须与此方法的第一个参数相同，即 数据库连接 指针。sqlite3_declare_vtab() 的第二个参数必须是一个以零结尾的 UTF-8 字符串，其中包含一个格式良好的 CREATE TABLE 语句，定义虚拟表中的列及其数据类型。此 CREATE TABLE 语句中的表名和所有约束均会被忽略，只有列名和数据类型有关。CREATE TABLE 语句字符串不必保存在持久性内存中，该字符串可以在 sqlite3_declare_vtab() 函数返回后释放或重新使用。

xConnect 方法还可以选择通过对 sqlite3_vtab_config() 接口进行一次或多次调用来请求虚拟表的特殊功能：

```sql
int sqlite3_vtab_config(sqlite3 *db, int op, ...);

```

对 sqlite3_vtab_config() 的调用是可选的。但是为了最大安全性，建议虚拟表实现在虚拟表不会从触发器或视图内部使用时调用 "sqlite3_vtab_config(db, SQLITE_VTAB_DIRECTONLY)"。

xCreate 方法不需要初始化 sqlite3_vtab 对象的 pModule、nRef 和 zErrMsg 字段。SQLite 核心将处理这些琐事。

如果 xCreate 方法成功创建新的虚拟表，则应返回 SQLITE_OK，如果失败则返回 SQLITE_ERROR。如果失败，不应分配 sqlite3_vtab 结构。如果失败，可选择在 *pzErr 中返回错误消息。必须使用 SQLite 内存分配函数（如 sqlite3_malloc() 或 sqlite3_mprintf()）分配用于保存错误消息字符串的空间，因为 SQLite 核心在向应用程序报告错误后会尝试使用 sqlite3_free() 释放这些空间。

如果省略 xCreate 方法（保留为 NULL 指针），则该虚拟表是一个 仅拟名虚拟表。无法使用 CREATE VIRTUAL TABLE 创建虚拟表的新实例，且只能通过其模块名称使用该虚拟表。请注意，SQLite 版本 3.9.0（2015-10-14）之前的版本不理解仅拟名虚拟表，如果尝试在仅拟名虚拟表上使用 CREATE VIRTUAL TABLE 会导致段错误，因为未检查 xCreate 方法是否为 null。

如果 xCreate 方法与 xConnect 方法的确切指针相同，则表示该虚拟表不需要初始化支持存储。这样的虚拟表可以用作同名虚拟表，或者使用 CREATE VIRTUAL TABLE 创建具名虚拟表，或者两者都可以。

### 2.1.1\. 虚拟表中的隐藏列

如果列数据类型包含特殊关键字“HIDDEN”（以任何大小写字母的任意组合），则该关键字将从列数据类型名称中省略，并在内部标记为隐藏列。隐藏列与普通列在三个方面不同：

+   PRAGMA table_info 返回的数据集中未列出隐藏列。

+   在 SELECT 的结果集中，隐藏列不包括在"*"表达式的展开中，

+   在缺乏显式列列表的 INSERT 语句中，隐藏列不包括在隐含的列列表中。

例如，如果将以下 SQL 传递给 sqlite3_declare_vtab()：

```sql
CREATE TABLE x(a HIDDEN VARCHAR(12), b INTEGER, c INTEGER Hidden);

```

然后虚拟表将创建两个隐藏列，数据类型为"VARCHAR(12)"和"INTEGER"。

隐藏列的一个示例用法可以在 FTS3 虚拟表实现中看到，其中每个 FTS 虚拟表包含一个 FTS 隐藏列，用于将信息从虚拟表传递到 FTS 辅助函数和 FTS MATCH 运算符。

### 2.1.2\. 表值函数

一个包含隐藏列的 virtual table 可以在 SELECT 语句的 FROM 子句中像表值函数一样使用。表值函数的参数成为虚拟表的隐藏列的约束条件。

例如，“generate_series”扩展（位于[ext/misc/series.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/series.c)文件中的[源代码树](https://www.sqlite.org/src/tree?ci=trunk)）实现了一个同名虚拟表，其模式如下：

```sql
CREATE TABLE generate_series(
  value,
  start HIDDEN,
  stop HIDDEN,
  step HIDDEN
);

```

这个表的实现中的 sqlite3_module.xBestIndex 方法检查与隐藏列的相等约束，并将其用作确定生成的整数“值”输出范围的输入参数。对于任何无约束的列，使用合理的默认值。例如，要列出 5 到 50 之间的所有整数：

```sql
SELECT value FROM generate_series(5,50);

```

上述查询等同于以下内容：

```sql
SELECT value FROM generate_series WHERE start=5 AND stop=50;

```

虚拟表名称上的参数与隐藏列按顺序匹配。参数数量可以少于隐藏列的数量，此时后面的隐藏列不受限制。但如果虚拟表中的参数比隐藏列更多，将会导致错误。

### 2.1.3\. WITHOUT ROWID 虚拟表

从 SQLite 版本 3.14.0（2016-08-08）开始，传递给 sqlite3_declare_vtab()的 CREATE TABLE 语句可以包含 WITHOUT ROWID 子句。这对于虚拟表行不易映射为唯一整数的情况非常有用。包含 WITHOUT ROWID 的 CREATE TABLE 语句必须定义一个或多个列作为主键。主键的每一列都必须单独为 NOT NULL，并且每行的所有列必须集体唯一。

注意，SQLite 不会强制执行 WITHOUT ROWID 虚拟表的主键约束。执行是底层虚拟表实现的责任。但 SQLite 确实假设主键约束是有效的 - 即被标识的列确实是唯一的且非空 - 并且利用此假设来优化针对虚拟表的查询。

当然，在 WITHOUT ROWID 虚拟表上无法访问 rowid 列。

xUpdate 方法最初设计时围绕单个值作为 ROWID 展开。后来，xUpdate 方法已经扩展，以适应任意的主键替代 ROWID，但主键仍然必须只有一个列。因此，SQLite 将拒绝任何具有多个主键列和非空 xUpdate 方法的 WITHOUT ROWID 虚拟表。

## 2.2\. xConnect 方法

```sql
int (*xConnect)(sqlite3*, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

xConnect 方法与 xCreate 非常相似。它具有相同的参数，并像 xCreate 一样构造一个新的 sqlite3_vtab 结构。它也必须像 xCreate 一样调用 sqlite3_declare_vtab()。还应该进行与 xCreate 相同的所有 sqlite3_vtab_config()调用。

区别在于 xConnect 用于建立与现有虚拟表的新连接，而 xCreate 用于从头创建新虚拟表。

当虚拟表具有某种必须在创建时初始化的后备存储时，xCreate 和 xConnect 方法才有所不同。xCreate 方法创建并初始化后备存储。xConnect 方法仅连接到现有的后备存储。当 xCreate 和 xConnect 相同时，表是一个同名虚拟表。

例如，考虑一个虚拟表实现，它提供对磁盘上现有逗号分隔值（CSV）文件的只读访问。对于这样的虚拟表，不需要创建或初始化后端存储（因为 CSV 文件已经存在于磁盘上），因此该模块的 xCreate 和 xConnect 方法将是相同的。

另一个例子是实现全文索引的虚拟表。xCreate 方法必须创建和初始化用于保存该索引的字典和 posting 列表的数据结构。另一方面，xConnect 方法只需定位并使用之前 xCreate 调用创建的现有字典和 posting 列表。

如果 xConnect 方法成功创建新的虚拟表，必须返回 SQLITE_OK，如果不成功，则返回 SQLITE_ERROR。如果不成功，不得分配 sqlite3_vtab 结构。如果不成功，错误消息可以选择性地在 *pzErr 中返回。必须使用类似于 sqlite3_malloc() 或 sqlite3_mprintf() 的 SQLite 内存分配函数分配错误消息字符串的空间，因为 SQLite 核心在报告错误后将尝试使用 sqlite3_free() 释放该空间。

虚拟表实现中，每个虚拟表都需要 xConnect 方法，尽管 xCreate 和 sqlite3_module 对象的 xConnect 指针可能指向同一个函数，如果虚拟表不需要初始化后端存储。

## 2.3\. xBestIndex 方法

SQLite 使用虚拟表模块的 xBestIndex 方法来确定访问虚拟表的最佳方式。xBestIndex 方法的原型如下：

```sql
int (*xBestIndex)(sqlite3_vtab *pVTab, sqlite3_index_info*);

```

SQLite 核心通过填充 sqlite3_index_info 结构的某些字段，并将指向该结构的指针作为第二个参数传递给 xBestIndex 方法，与 xBestIndex 方法进行通信。xBestIndex 方法填充该结构的其他字段，形成回复。sqlite3_index_info 结构如下：

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

注意 "estimatedRows"、"idxFlags" 和 colUsed 字段上的警告。这些字段分别是在 SQLite 版本 3.8.2、3.9.0 和 3.10.0 中添加的。任何读取或写入这些字段的扩展必须首先检查正在使用的 SQLite 库版本是否大于或等于相应的版本，例如可以将 sqlite3_libversion_number() 返回的值与常量 3008002、3009000 和/或 3010000 进行比较。尝试访问由旧版 SQLite 创建的 sqlite3_index_info 结构中的这些字段的结果是未定义的。

此外，还有一些定义的常量：

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

使用 sqlite3_vtab_collation()接口查找在评估第 i 个约束时应使用的排序序列的名称：

```sql
const char *sqlite3_vtab_collation(sqlite3_index_info*, int i);

```

当 SQLite 核心编译涉及虚拟表的查询时，会调用 xBestIndex 方法。换句话说，当 SQLite 运行 sqlite3_prepare()或其等效方法时，会调用此方法。通过调用此方法，SQLite 核心向虚拟表表示需要访问虚拟表中某些行的子集，并希望了解执行该访问的最有效方法。xBestIndex 方法将回复 SQLite 核心可以用来进行虚拟表高效搜索的信息。

在编译单个 SQL 查询时，SQLite 核心可能会多次调用 xBestIndex，并在 sqlite3_index_info 中使用不同的设置。然后 SQLite 核心将选择似乎提供最佳性能的组合。

在调用此方法之前，SQLite 核心会用关于当前正在处理的查询的信息初始化一个 sqlite3_index_info 结构的实例。这些信息主要来自查询的 WHERE 子句和 ORDER BY 或 GROUP BY 子句，但如果查询是一个连接，则还会来自任何 ON 或 USING 子句。SQLite 核心提供给 xBestIndex 方法的信息保存在标记为“Inputs”的结构的一部分中。“Outputs”部分初始化为零。

sqlite3_index_info 结构中的信息是短暂的，并且可能在 xBestIndex 方法返回后被覆盖或释放。如果 xBestIndex 方法需要记住 sqlite3_index_info 结构的任何部分，它应该制作一份副本。必须小心地将副本存储在可能会被释放的位置，例如在 needToFreeIdxStr 设置为 1 的 idxStr 字段中。

注意，在调用 xFilter 之前始终会调用 xBestIndex，因为从 xBestIndex 中获取的 idxNum 和 idxStr 输出是 xFilter 的必需输入。但是，并不保证在成功的 xBestIndex 之后一定会调用 xFilter。

每个虚拟表实现都需要 xBestIndex 方法。

### 2.3.1\. 输入

SQLite 核心试图向虚拟表传达的主要信息是可用于限制需要搜索的行数的约束条件。aConstraint[]数组包含每个约束的条目。该数组中将恰好有 nConstraint 个条目。

每个约束通常对应于 WHERE 子句中的一个项或形如 USING 或 ON 子句中的一个项

> 列 OP 表达式

在“column”是虚拟表中的列，“OP”是像“=”或“<”这样的运算符，而“EXPR”是任意表达式。因此，例如，如果 WHERE 子句包含如下项：

```sql
a = 5

```

其中一个约束可能是在 "a" 列上，操作符为 "="，表达式为 "5"。约束不需要在 WHERE 子句中具有文字表示。查询优化器可能会对 WHERE 子句进行转换，以提取尽可能多的约束。例如，如果 WHERE 子句包含类似以下内容：

```sql
x BETWEEN 10 AND 100 AND 999>y

```

查询优化器可能会将此转换为三个单独的约束：

```sql
x >= 10
x <= 100
y < 999

```

对于每个这样的约束，aConstraint[].iColumn 字段指示约束左侧的列。虚拟表的第一列是列 0。虚拟表的 rowid 是列 -1。aConstraint[].op 字段指示使用的操作符。SQLITE_INDEX_CONSTRAINT_* 常量将整数常量映射到操作符值。列按照在 sqlite3_declare_vtab() 中调用时定义的顺序出现。在确定列索引时，隐藏列也会被计数。

如果虚拟表的 xFindFunction() 方法已定义，并且 xFindFunction() 有时返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值，则约束也可能是以下形式之一：

> FUNCTION(column, EXPR)

在这种情况下，aConstraint[].op 的值与 xFindFunction() 返回的 FUNCTION 的值相同。

数组 aConstraint[] 包含有关适用于虚拟表的所有约束的信息。但是由于表在连接中的排序方式，某些约束可能无法使用。因此，xBestIndex 方法只需考虑具有 aConstraint[].usable 标志为 true 的约束。

除了 WHERE 子句约束外，SQLite 核心还告知 xBestIndex 方法关于 ORDER BY 子句的信息。（在聚合查询中，SQLite 核心可能会用 GROUP BY 子句信息替代 ORDER BY 子句信息，但这个事实不应该对 xBestIndex 方法造成任何影响。）如果 ORDER BY 子句的所有项都是虚拟表中的列，则 nOrderBy 将是 ORDER BY 子句中项的数量，aOrderBy[] 数组将标识每个 ORDER BY 子句项的列，以及该列是 ASC 还是 DESC。

在 SQLite version 3.10.0（2016-01-06）及更高版本中，colUsed 字段可用于指示被准备的语句实际使用的虚拟表的哪些字段。如果 colUsed 的最低位设置了，这意味着使用了第一列。第二低位对应第二列，依此类推。如果 colUsed 的最高位设置了，这意味着除了前 63 列之外还使用了一列或多列。如果 xFilter 方法需要列使用信息，则必须将所需的位编码到输出的 idxNum 字段或 idxStr 内容中。

#### 2.3.1.1\. LIKE、GLOB、REGEXP 和 MATCH 函数

对于 LIKE、GLOB、REGEXP 和 MATCH 操作符，aConstraint[].iColumn 值是虚拟表列，即操作符的左操作数。但是，如果这些操作符表示为函数调用而不是操作符，则 aConstraint[].iColumn 值引用的是作为该函数第二个参数的虚拟表列：

> LIKE(*EXPR*, *column*)
> 
> GLOB(*EXPR*, *column*)
> 
> REGEXP(*EXPR*, *column*)
> 
> MATCH(*EXPR*, *column*)

因此，就 xBestIndex()方法而言，以下两种形式是等效的：

> *column* LIKE *EXPR*
> 
> LIKE(*EXPR*,*column*)

对于 LIKE、GLOB、REGEXP 和 MATCH 函数，仅在查看函数的第二个参数时才会出现这种特殊行为。对于所有其他函数，aConstraint[].iColumn 值引用的是函数的第一个参数。

然而，LIKE、GLOB、REGEXP 和 MATCH 的这种特殊功能不适用于 xFindFunction()方法。xFindFunction()方法始终基于 LIKE、GLOB、REGEXP 或 MATCH 操作符的左操作数，而不是这些操作符的函数调用等效的第一个参数。

#### 2.3.1.2\. LIMIT 和 OFFSET

当 aConstraint[].op 是 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，表示在使用虚拟表的 SQL 查询语句中存在 LIMIT 或 OFFSET 子句。LIMIT 和 OFFSET 操作符没有左操作数，因此当 aConstraint[].op 是 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，aConstraint[].iColumn 值是无意义的，不应使用。

#### 2.3.1.3\. 约束的右手边值

sqlite3_vtab_rhs_value() 接口可用于尝试访问约束的右操作数。但是，在运行 xBestIndex 方法时可能并不知道右操作数的值，因此 sqlite3_vtab_rhs_value() 调用可能不成功。通常情况下，约束的右操作数仅在输入 SQL 中编码为文字值时才可用于 xBestIndex。如果右操作数被编码为表达式或 主机参数，则 xBestIndex 可能无法访问。某些操作符，如 SQLITE_INDEX_CONSTRAINT_ISNULL 和 SQLITE_INDEX_CONSTRAINT_ISNOTNULL，没有右操作数。sqlite3_vtab_rhs_value() 接口对于这类操作符总是返回 SQLITE_NOTFOUND。

### 2.3.2\. 输出

根据上述所有信息，xBestIndex 方法的工作是找出搜索虚拟表的最佳方法。

xBestIndex 方法通过 idxNum 和 idxStr 字段向 xFilter 方法传递索引策略。就 SQLite 核心而言，idxNum 值和 idxStr 字符串内容是任意的，并且只要 xBestIndex 和 xFilter 在它们的含义上达成一致，可以有任何含义。SQLite 核心只是简单地将信息从 xBestIndex 复制到 xFilter 方法，假定 idxStr 引用的字符序列是以 NUL 结尾的。

idxStr 值可以是从 SQLite 内存分配函数（如 sqlite3_mprintf()）获得的字符串。如果是这种情况，则 needToFreeIdxStr 标志必须设置为 true，以便 SQLite 核心在使用完后调用 sqlite3_free() 释放该字符串，避免内存泄漏。idxStr 值也可以是静态常量字符串，在这种情况下，needToFreeIdxStr 布尔值应保持 false。

estimatedCost 字段应设置为执行该查询所需的估计磁盘访问操作数。SQLite 核心通常会多次调用 xBestIndex，使用不同的约束获取多个成本估算，然后选择给出最低估算的查询计划。SQLite 核心在调用 xBestIndex 之前将 estimatedCost 初始化为一个非常大的值，因此如果 xBestIndex 确定当前参数组合不理想，它可以保持 estimatedCost 字段不变，以阻止其使用。

如果当前的 SQLite 版本为 3.8.2 或更高，则 estimatedRows 字段可以设置为建议查询计划返回的行数的估算值。如果未显式设置此值，则使用默认的 25 行估算。

如果当前 SQLite 版本为 3.9.0 或更高，则 idxFlags 字段可以设置为 SQLITE_INDEX_SCAN_UNIQUE，表示虚拟表在给定输入约束条件时将仅返回零行或一行。在后续 SQLite 版本中，可能会理解 idxFlags 字段的附加位。

aConstraintUsage[]数组包含 sqlite3_index_info 结构中输入部分 nConstraint 个约束条件的每一个元素。aConstraintUsage[]数组被 xBestIndex 用来告诉核心它如何使用这些约束条件。

xBestIndex 方法可以将 aConstraintUsage[].argvIndex 条目设置为大于零的值。应该将一个条目设置为 1，另一个设置为 2，再另一个设置为 3，依此类推，直到 xBestIndex 方法想要的数量为止。相应约束条件的 EXPR 将作为 argv[]参数传递给 xFilter。

例如，如果 aConstraint[3].argvIndex 设置为 1，则在调用 xFilter 时，传递给 xFilter 的 argv[0]将具有 aConstraint[3]约束的 EXPR 值。

#### 2.3.2.1\. 在字节码中省略约束检查

默认情况下，SQLite 生成的字节码会对虚拟表的每一行进行双重检查，以验证它们是否满足所有约束条件。如果虚拟表能够保证某个约束条件始终得到满足，它可以尝试通过设置 aConstraintUsage[].omit 来抑制这种双重检查。然而，除了一些例外情况外，这只是一个提示，并不能保证会抑制约束条件的多余检查。重点如下：

+   只有当约束条件的 argvIndex 值大于 0 且小于或等于 16 时，omit 标志才会被尊重。对于未将右操作数传递到 xFilter 方法的约束条件，永远不会抑制约束检查。当前实现只能抑制传递给 xFilter 的前 16 个值的冗余约束检查，尽管这个限制可能会在未来的版本中增加。

+   只要 argvIndex 大于 0，就始终尊重 SQLITE_INDEX_CONSTRAINT_OFFSET 约束的 omit 标志。在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置 omit 标志表示 SQLite 将自行抑制输出的前 N 行，其中 N 是 OFFSET 运算符的右操作数。如果虚拟表实现在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置 omit，但未能抑制输出的前 N 行，则整体查询将产生不正确的结果。

#### 2.3.2.2\. ORDER BY 和 orderByConsumed

如果虚拟表将按 ORDER BY 子句指定的顺序输出行，则可以将 orderByConsumed 标志设置为 true。如果输出不自动按正确顺序，则必须将 orderByConsumed 保留在其默认 false 设置中。这将告知 SQLite 核心在虚拟表输出后需要对数据进行单独的排序处理。设置 orderByConsumed 是一种优化。如果将 orderByConsumed 设置为其默认值（0），则查询将始终获得正确答案。如果设置 orderByConsumed 不正确，则可能导致错误答案。建议新的虚拟表实现在最初未设置 orderByConsumed 的值，然后在确认一切正常工作后，根据需要返回并尝试优化设置 orderByConsumed。

有时即使虚拟表的输出与 nOrderBy 和 aOrderBy 指定的顺序不完全相符，也可以安全地设置 orderByConsumed 标志。如果 sqlite3_vtab_distinct() 接口返回 1 或 2，则表示可以放宽排序。有关详细信息，请参阅关于 sqlite3_vtab_distinct() 的文档。

### 2.3.3\. 返回值

xBestIndex 方法应在成功时返回 SQLITE_OK。如果发生任何致命错误，则应返回适当的错误代码（例如：SQLITE_NOMEM）。

如果 xBestIndex 返回 SQLITE_CONSTRAINT，这并不表示错误。相反，SQLITE_CONSTRAINT 表示指定的输入参数组合对虚拟表执行其工作是不足的。从逻辑上讲，这与将 estimatedCost 设置为无穷大是相同的。如果针对特定查询计划的每次 xBestIndex 调用都返回 SQLITE_CONSTRAINT，则表示无法安全使用虚拟表，并且 sqlite3_prepare() 调用将因“无查询解决方案”错误而失败。

### 2.3.4\. 强制对表值函数进行必需参数的执行

xBestIndex 返回的 SQLITE_CONSTRAINT 对于具有必需参数的 表值函数 非常有用。如果一个必需参数的 aConstraint[].usable 字段为 false，则 xBestIndex 方法应返回 SQLITE_CONSTRAINT。如果某个必需字段在 aConstraint[] 数组中根本不存在，则意味着输入 SQL 中省略了相应的参数。在这种情况下，xBestIndex 应在 pVTab->zErrMsg 中设置错误消息并返回 SQLITE_ERROR。总结一下：

1.  对于必需的参数，aConstraint[].usable 值为 false → 返回 SQLITE_CONSTRAINT。

1.  如果必需的参数在 aConstraint[] 数组中根本没有出现 → 在 pVTab->zErrMsg 中设置错误消息并返回 SQLITE_ERROR。

下面的示例将更好地说明从 xBestIndex 返回 SQLITE_CONSTRAINT 作为返回值的用法：

```sql
SELECT * FROM realtab, tablevaluedfunc(realtab.x);

```

假设"tablevaluedfunc"的第一个隐藏列是"param1"，则上面的查询在语义上等同于这个查询：

```sql
SELECT * FROM realtab, tablevaluedfunc
 WHERE tablevaluedfunc.param1 = realtab.x;

```

查询规划器必须在此查询的多种可能实现之间做出选择，但特别注意的是有两个计划：

1.  扫描 realtab 的所有行，并对每一行，在 tablevaluedfunc 中找到 param1 等于 realtab.x

1.  扫描 tablevaluedfunc 的所有行，并对每一行在 realtab 中找到 x 等于 tablevaluedfunc.param1 的行。

xBestIndex 方法将为上述每个潜在的计划调用一次。对于计划 1，对 param1 列的 SQLITE_CONSTRAINT_EQ 约束条件的 aConstraint[].usable 标志将为 true，因为"param1 = ?"约束条件的右侧值将是已知的，因为它由外部 realtab 循环确定。但对于计划 2，"param1 = ?"的 aConstraint[].usable 标志将为 false，因为右侧值是由内部循环确定的，因此是一个未知量。因为 param1 是表值函数的必需输入，所以 xBestIndex 方法在提供计划 2 时应返回 SQLITE_CONSTRAINT，表示缺少必需的输入。这迫使查询规划器选择计划 1。

## 2.4\. xDisconnect 方法

```sql
int (*xDisconnect)(sqlite3_vtab *pVTab);

```

这种方法释放了对虚拟表的连接。只有 sqlite3_vtab 对象被销毁。虚拟表不会被销毁，与虚拟表关联的任何后备存储仍然存在。这种方法撤销了 xConnect 的工作。

这种方法是对虚拟表连接的析构函数。与 xDestroy 方法相比，这种方法有所不同。xDestroy 是整个虚拟表的析构函数。

每个虚拟表实现都需要 xDisconnect 方法，尽管 xDisconnect 和 xDestroy 方法可以是同一个函数，如果对特定的虚拟表有意义的话。

## 2.5\. xDestroy 方法

```sql
int (*xDestroy)(sqlite3_vtab *pVTab);

```

这种方法释放了对虚拟表的连接，就像 xDisconnect 方法一样，并销毁了底层表的实现。这种方法撤销了 xCreate 的工作。

当使用虚拟表的数据库连接关闭时，会调用 xDisconnect 方法。只有在执行 DROP TABLE 语句来删除虚拟表时才会调用 xDestroy 方法。

每个虚拟表实现都需要 xDestroy 方法，尽管 xDisconnect 和 xDestroy 方法可以是同一个函数，如果对特定的虚拟表有意义的话。

## 2.6\. xOpen 方法

```sql
int (*xOpen)(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor);

```

方法 xOpen 创建一个用于访问（读和/或写）虚拟表的新游标。此方法的成功调用将为 sqlite3_vtab_cursor（或其子类）分配内存，初始化新对象，并使 *ppCursor 指向新对象。成功的调用然后返回 SQLITE_OK。

对于此方法的每次成功调用，SQLite 核心随后将调用 xClose 方法来销毁分配的游标。

方法 xOpen 不需要初始化 sqlite3_vtab_cursor 结构体的 pVtab 字段。SQLite 核心会自动处理这一任务。

虚拟表实现必须能够支持任意数量的同时打开的游标。

初始打开时，游标处于未定义状态。SQLite 核心将在尝试定位或读取游标之前立即调用游标上的 xFilter 方法。

方法 xOpen 是每个虚拟表实现所必需的。

## 2.7\. 方法 xClose

```sql
int (*xClose)(sqlite3_vtab_cursor*);

```

方法 xClose 关闭先前由 xOpen 打开的游标。SQLite 核心将始终对使用 xOpen 打开的每个游标调用一次 xClose。

此方法必须释放由相应 xOpen 调用分配的所有资源。即使返回错误，也不会再次调用该例程。SQLite 核心在关闭后不会再次使用 sqlite3_vtab_cursor。

方法 xClose 是每个虚拟表实现所必需的。

## 2.8\. 方法 xEof

```sql
int (*xEof)(sqlite3_vtab_cursor*);

```

方法 xEof 必须在指定的游标当前指向有效数据行时返回 false（零），否则返回 true（非零）。此方法在每次 xFilter 和 xNext 调用后由 SQL 引擎立即调用。

方法 xEof 是每个虚拟表实现所必需的。

## 2.9\. 方法 xFilter

```sql
int (*xFilter)(sqlite3_vtab_cursor*, int idxNum, const char *idxStr,
              int argc, sqlite3_value **argv);

```

此方法开始对虚拟表的搜索。第一个参数是由 xOpen 打开的游标。接下来的两个参数定义了先前由 xBestIndex 选择的特定搜索索引。idxNum 和 idxStr 的具体含义不重要，只要 xFilter 和 xBestIndex 对此含义达成一致即可。

函数 xBestIndex 可能已经使用 sqlite3_index_info 结构体的 aConstraintUsage[].argvIndex 值请求了某些表达式的值。这些值通过 argc 和 argv 参数传递给 xFilter。

如果虚拟表包含一个或多个符合搜索条件的行，则游标必须指向第一行。后续对 xEof 的调用必须返回 false（零）。如果没有匹配的行，则游标必须处于使得 xEof 返回 true（非零）的状态。SQLite 引擎将使用 xColumn 和 xRowid 方法访问该行内容。xNext 方法用于前进到下一行。

如果成功，此方法必须返回 SQLITE_OK，如果发生错误，则返回 sqlite error code。

xFilter 方法是每个虚拟表实现所必需的。

## 2.10\. xNext 方法

```sql
int (*xNext)(sqlite3_vtab_cursor*);

```

xNext 方法将 虚拟表游标 前进到由 xFilter 初始化的结果集的下一行。如果在调用此例程时游标已经指向最后一行，则游标不再指向有效数据，后续对 xEof 方法的调用必须返回 true（非零）。如果成功将游标前进到另一行内容，则后续对 xEof 的调用必须返回 false（零）。

如果成功，此方法必须返回 SQLITE_OK，如果发生错误，则返回 sqlite error code。

xNext 方法是每个虚拟表实现所必需的。

## 2.11\. xColumn 方法

```sql
int (*xColumn)(sqlite3_vtab_cursor*, sqlite3_context*, int N);

```

SQLite 核心调用此方法以查找当前行第 N 列的值。N 是从零开始的，因此第一列编号为 0。xColumn 方法可以使用以下接口将其结果返回给 SQLite：

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

如果 xColumn 方法实现未调用上述任何函数，则列的值默认为 SQL NULL。

要引发错误，xColumn 方法应使用 result_text() 方法之一设置错误消息文本，然后返回适当的 error code。xColumn 方法在成功时必须返回 SQLITE_OK。

xColumn 方法是每个虚拟表实现所必需的。

## 2.12\. xRowid 方法

```sql
int (*xRowid)(sqlite3_vtab_cursor *pCur, sqlite_int64 *pRowid);

```

成功调用此方法将导致*pRowid 被填充为虚拟表游标`pCur`当前指向的行的 rowid。此方法在成功时返回 SQLITE_OK，在失败时返回适当的错误代码。

每个虚拟表实现都需要`xRowid`方法。

## 2.13\. `xUpdate`方法

```sql
int (*xUpdate)(
  sqlite3_vtab *pVTab,
  int argc,
  sqlite3_value **argv,
  sqlite_int64 *pRowid
);

```

所有对虚拟表的更改都使用`xUpdate`方法进行。这个方法可以用来插入、删除或更新。

argc 参数指定 argv 数组中的条目数。对于纯删除操作，argc 的值为 1，对于插入或替换或更新操作，argc 的值为 N+2，其中 N 是表中的列数。在前一句中，N 包括任何隐藏列。

每个 argv 条目在 C 中都将具有非 NULL 值，但可能包含 SQL 值 NULL。换句话说，对于所有`i`在 0 到`argc-1`之间，`argv[i]!=0`总是成立。但是，可能会出现`sqlite3_value_type(argv[i])==SQLITE_NULL`的情况。

`argv[0]`参数是要删除的虚拟表中行的 rowid。如果`argv[0]`是 SQL NULL，则不会进行删除操作。

`argv[1]`参数是要插入到虚拟表中的新行的 rowid。如果`argv[1]`是 SQL NULL，则实现必须为新插入的行选择一个 rowid。随后的 argv[]条目包含虚拟表列的值，按照声明这些列的顺序进行排列。列数将与使用 sqlite3_declare_vtab()调用的表声明匹配。所有隐藏列都包括在内。

在没有 rowid 插入（argc>1，argv[1]是 SQL NULL）的情况下，在使用 ROWID 的虚拟表上（但不是在 WITHOUT ROWID 虚拟表上），实现必须将*pRowid 设置为新插入行的 rowid；这将成为 sqlite3_last_insert_rowid()函数返回的值。在所有其他情况下设置此值都是无害的操作；如果 argc==1 或 argv[1]不是 SQL NULL，则 SQLite 引擎会忽略*pRowid 返回值。

每次调用 xUpdate 都将落入下面显示的一种情况。请注意，对**argv[i]**的引用意味着 argv[i]对象内包含的 SQL 值，而不是 argv[i]对象本身。

> **`argc = 1`
> 
> `argv[0] ≠ NULL**`
> 
> DELETE：删除具有与 argv[0]相等的 rowid 或主键的单行。不进行插入。
> 
> **`argc > 1`
> 
> `argv[0] = NULL**`
> 
> 插入：使用从 argv[2] 和之后获取的列值插入新行。在 rowid 虚拟表中，如果 argv[1] 是 SQL NULL，则会自动生成新的唯一 rowid。对于无 rowid 虚拟表，argv[1] 将为 NULL，在这种情况下，实现应从 argv[2] 和之后的适当列中获取主键值。
> 
> **argc > 1
> 
> `argv[0] ≠ NULL
> 
> `argv[0] = argv[1]**
> 
> 更新：使用 argv[0] 中的 rowid 或主键更新为 argv[2] 和之后的新值参数。
> 
> **argc > 1
> 
> `argv[0] ≠ NULL
> 
> `argv[0] ≠ argv[1]**
> 
> 更新带有 rowid 或主键更改：使用 argv[0] 中的 rowid 或主键更新为 argv[1] 中的 rowid 或主键以及 argv[2] 和之后的新值参数。这将发生在 SQL 语句更新行 id 的情况下，如下所示：
> 
> > 更新 表 SET rowid=rowid+1 WHERE ...;

如果 xUpdate 方法成功，则必须返回 SQLITE_OK。如果发生故障，则 xUpdate 必须返回适当的 错误代码。在失败时，可以选择将 pVTab->zErrMsg 元素替换为使用类似 sqlite3_mprintf() 或 sqlite3_malloc() 从 SQLite 分配的内存中存储的错误消息文本。

如果 xUpdate 方法违反虚拟表的某些约束（包括但不限于尝试存储错误数据类型的值、尝试存储过大或过小的值、或尝试更改只读值），则 xUpdate 必须使用适当的 错误代码 失败。

如果 xUpdate 方法正在执行更新操作，则可以使用 sqlite3_value_nochange(X) 来发现虚拟表中哪些列实际上被更新语句修改了。 sqlite3_value_nochange(X) 接口对于不变的列返回 true。在每次更新时，SQLite 首先为表中每个未更改的列分别调用 xColumn 以获取该列的值。 xColumn 方法可以通过调用 sqlite3_vtab_nochange() 来检查 SQL 级别上是否未更改该列。如果 xColumn 发现列未被修改，则应返回而不使用 sqlite3_result_xxxxx() 接口设置结果。只有在这种情况下，xUpdate 方法内部的 sqlite3_value_nochange() 才会返回 true。如果 xColumn 调用了一个或多个 sqlite3_result_xxxxx() 接口，则 SQLite 会理解为列值的更改，而 xUpdate 方法内部对该列的 sqlite3_value_nochange() 调用将返回 false。

在虚拟表实例上和在虚拟表的行上可能会有一个或多个 sqlite3_vtab_cursor 对象处于打开和使用状态，当调用 xUpdate 方法时。在实现 xUpdate 方法时，必须准备好处理来自其他现有游标的删除或修改表行的尝试。如果虚拟表无法适应这些更改，则 xUpdate 方法必须返回一个错误代码。

xUpdate 方法是可选的。如果虚拟表在 sqlite3_module 中的 xUpdate 指针是空指针，则该虚拟表是只读的。

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

此方法在 sqlite3_prepare()期间调用，以使虚拟表实现有机会重载函数。如果此方法被设置为 NULL，则不会进行重载。

当一个函数将虚拟表的列作为其第一个参数时，将调用此方法以查看虚拟表是否希望重载该函数。前三个参数是输入：虚拟表、函数的参数数目和函数的名称。如果不希望进行重载，则此方法返回 0。要重载函数，此方法将新函数实现写入*pxFunc，并将用户数据写入*ppArg，并返回 1 或介于 SQLITE_INDEX_CONSTRAINT_FUNCTION 和 255 之间的数字。

历史上，xFindFunction()的返回值只能是零或一。零表示函数未重载，一表示已重载。从版本 3.25.0（2018-09-15）开始，可以返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值。如果 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值，则表示该函数接受两个参数，并且该函数可以在查询的 WHERE 子句中用作布尔值，并且虚拟表能够利用该函数加速查询结果。当 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值时，返回的值将成为传递给 xBestIndex()中的一个约束的 sqlite3_index_info.aConstraint.op 值。函数的第一个参数是由约束的 aConstraint[].iColumn 字段标识的列，第二个参数是将传递给 xFilter()（如果设置了 aConstraintUsage[].argvIndex 值）或从 sqlite3_vtab_rhs_value()返回的值。

Geopoly 模块 是利用 SQLITE_INDEX_CONSTRAINT_FUNCTION 来提升性能的虚拟表的一个示例。Geopoly 的 xFindFunction() 方法返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 以用于 geopoly_overlap() SQL 函数，并且返回 SQLITE_INDEX_CONSTRAINT_FUNCTION+1 以用于 geopoly_within() SQL 函数。这允许对查询进行优化，例如：

```sql
SELECT * FROM geopolytab WHERE geopoly_overlap(_shape, $query_polygon);
SELECT * FROM geopolytab WHERE geopoly_within(_shape, $query_polygon);

```

注意中缀函数（LIKE、GLOB、REGEXP 和 MATCH）反转其参数的顺序。因此，“like(A,B)”通常与“B like A”相同。但是，xFindFunction() 总是查看最左边的参数，而不是第一个逻辑参数。因此，对于形式“B like A”，SQLite 查看左操作数“B”，如果该操作数是虚拟表列，则在该虚拟表上调用 xFindFunction() 方法。但如果使用形式“like(A,B)”，则 SQLite 检查 A 项是否为虚拟表列，如果是，则为列 A 的虚拟表调用 xFindFunction() 方法。

此例程返回的函数指针必须对于给定的 sqlite3_vtab 对象的生命周期有效。

## 2.15\. xBegin 方法

```sql
int (*xBegin)(sqlite3_vtab *pVTab);

```

这个方法在虚拟表上开始一个事务。这个方法是可选的。sqlite3_module 的 xBegin 指针可能为 NULL。

这个方法总是在调用 xCommit 或 xRollback 方法之一后跟随。虚拟表事务不嵌套，因此在单个虚拟表上不会连续调用 xBegin 方法而没有调用 xCommit 或 xRollback。在 xBegin 和对应的 xCommit 或 xRollback 之间可能会多次调用其他方法。

## 2.16\. xSync 方法

```sql
int (*xSync)(sqlite3_vtab *pVTab);

```

这个方法在虚拟表上发出两阶段提交的开始信号。这个方法是可选的。sqlite3_module 的 xSync 指针可能为 NULL。

此方法仅在调用 xBegin 方法后且在 xCommit 或 xRollback 之前调用。为了实现两阶段提交，将在任何虚拟表上的 xCommit 方法调用之前调用所有虚拟表的 xSync 方法。如果任何 xSync 方法失败，则整个事务将回滚。

## 2.17\. xCommit 方法

```sql
int (*xCommit)(sqlite3_vtab *pVTab);

```

这个方法导致虚拟表事务提交。这个方法是可选的。sqlite3_module 的 xCommit 指针可能为 NULL。

在此方法调用之前总是先调用 xBegin 和 xSync。

## 2.18\. xRollback 方法

```sql
int (*xRollback)(sqlite3_vtab *pVTab);

```

此方法导致虚拟表事务回滚。此方法是可选的。sqlite3_module 的 xRollback 指针可能为 NULL。

对此方法的调用始终紧随对 xBegin 的先前调用。

## 2.19\. xRename 方法

```sql
int (*xRename)(sqlite3_vtab *pVtab, const char *zNew);

```

此方法提供了一个通知，告知虚拟表实现将给虚拟表一个新名称。如果此方法返回 SQLITE_OK，那么 SQLite 将重命名表。如果此方法返回错误代码，则会阻止重命名操作。

xRename 方法是可选的。如果省略，则不能使用 ALTER TABLE RENAME 命令重命名虚拟表。

在调用此方法之前，会先启用 PRAGMA legacy_alter_table 设置，并在此方法结束后恢复 legacy_alter_table 的值。这对于使用 shadow tables 的虚拟表的正确操作是必需的，其中 shadow tables 必须重命名以匹配新的虚拟表名称。如果 legacy_alter_format 为关闭状态，则每次 xRename 方法尝试更改 shadow table 的名称时都将为虚拟表调用 xConnect 方法。

## 2.20\. xSavepoint、xRelease 和 xRollbackTo 方法

```sql
int (*xSavepoint)(sqlite3_vtab *pVtab, int);
int (*xRelease)(sqlite3_vtab *pVtab, int);
int (*xRollbackTo)(sqlite3_vtab *pVtab, int);

```

这些方法为虚拟表实现提供了实现嵌套事务的机会。它们始终是可选的，并且仅在 SQLite 版本 3.7.7（2011-06-23）及更高版本中才会被调用。

当调用 xSavepoint(X,N) 时，这是对虚拟表 X 的信号，它应将当前状态保存为 savepoint N。随后调用 xRollbackTo(X,R) 意味着虚拟表的状态应返回到上次调用 xSavepoint(X,R) 时的状态。调用 xRollbackTo(X,R) 将使所有 N>R 的 savepoint 失效；在未通过调用 xSavepoint() 重新初始化之前，这些失效的 savepoint 都不会被回滚或释放。调用 xRelease(X,M) 会使所有 N>=M 的 savepoint 失效。

除了在 xBegin() 和 xCommit() 或 xRollback() 之间的调用之外，将永远不会调用 xSavepoint()、xRelease() 或 xRollbackTo() 方法。

## 2.21\. xShadowName 方法

一些虚拟表实现（例如：FTS3、FTS5 和 RTREE）利用真实（非虚拟）数据库表来存储内容。例如，当内容插入 FTS3 虚拟表时，数据最终存储在名为 "%_content"、"%_segdir"、"%_segments"、"%_stat" 和 "%_docsize" 的真实表中，其中 "%" 是原始虚拟表的名称。用于存储虚拟表内容的这些辅助真实表称为 "shadow tables"。详细信息请参见 (1)、(2) 和 (3)。

xShadowName 方法存在，允许 SQLite 确定某个实际表是否确实是虚拟表的影子表。

如果 SQLite 满足以下所有条件，则认为一个实际表是一个影子表：

+   表名包含一个或多个"_"字符。

+   名称中最后一个"_"字符之前的部分与使用 CREATE VIRTUAL TABLE 创建的虚拟表的名称完全匹配。（影子表不适用于同名虚拟表和表值函数。）

+   虚拟表包含一个 xShadowName 方法。

+   当 xShadowName 方法的输入是表名中最后一个"_"字符后面的部分时，xShadowName 方法返回 true。

如果 SQLite 识别一个表为影子表，并且设置了 SQLITE_DBCONFIG_DEFENSIVE 标志，那么对于普通 SQL 语句来说，影子表是只读的。影子表仍然可以被写入，但只能通过某个虚拟表实现的方法内部调用的 SQL 来写入。

xShadowName 方法的整个目的是保护影子表内容免受恶意 SQL 的破坏。每个使用影子表的虚拟表实现都应该能够检测和处理被损坏的影子表内容。但是，特定虚拟表实现中的错误可能允许故意损坏的影子表导致崩溃或其他故障。xShadowName 机制旨在通过防止普通 SQL 语句故意损坏影子表来避免零日攻击。

影子表默认是可读写的。只有在使用 sqlite3_db_config()设置了 SQLITE_DBCONFIG_DEFENSIVE 标志之后，影子表才会变成只读。影子表默认需要是可读写的，以保持向后兼容性。例如，CLI 的.dump 命令生成的 SQL 文本直接写入影子表中。

## 2.22\. xIntegrity 方法

如果一个 sqlite3_module 的 iVersion 为 4 或更高，并且 xIntegrity 方法不为 NULL，则 PRAGMA integrity_check 和 PRAGMA quick_check 命令将在处理过程中调用 xIntegrity。如果 xIntegrity 方法将错误消息字符串写入第五个参数，则 PRAGMA integrity_check 将将该错误作为其输出的一部分报告。因此，换句话说，xIntegrity 方法允许 PRAGMA integrity_check 命令验证存储在虚拟表中的内容的完整性。

xIntegrity 方法被调用时有五个参数：

+   **pVTab** → 指向正在检查的 sqlite3_vtab 对象的指针。

+   **zSchema** → 虚拟表定义所在的模式（"main"、"temp"等）的名称。

+   **zTabName** → 虚拟表的名称。

+   **mFlags** → 一个标志，指示这是一个"integrity_check"还是"quick_check"。当前，该参数总是 0 或 1，但未来的 SQLite 版本可能会使用整数的其他位来指示额外的处理选项。

+   **pzErr** → 此参数指向一个初始化为 NULL 的"char*"。如果 xIntegrity()实现发现任何问题，应该从 sqlite3_malloc()或等效方法获取错误字符串，并让*pzErr 指向该字符串。

xIntegrity 方法通常应返回 SQLITE_OK，即使在虚拟表内容中发现问题。任何其他错误代码意味着 xIntegrity 方法本身在评估虚拟表内容时遇到问题。例如，如果发现 FTS5 的反向索引内部不一致，则 xIntegrity 方法应将适当的错误消息写入 pzErr 参数并返回 SQLITE_OK。但如果 xIntegrity 方法由于内存耗尽而无法完成对虚拟表内容的评估，则应返回 SQLITE_NOMEM。

如果生成了错误消息，则应从 sqlite3_malloc64()或等效方法获取空间以保存错误消息字符串。当 xIntegrity 返回时，错误消息字符串的所有权将传递给 SQLite 核心。核心会确保在使用完错误消息后调用 sqlite3_free()来回收内存。调用 xIntegrity 方法的 PRAGMA integrity_check 命令不会更改返回的错误消息。xIntegrity 方法本身应包含虚拟表名称作为消息的一部分。zSchema 和 zName 参数提供了便利。

mFlags 参数目前是一个布尔值（0 或 1），指示 xIntegrity 方法是由于 PRAGMA integrity_check（mFlags==0）还是由于 PRAGMA quick_check（mFlags==1）而调用。一般来说，xIntegrity 方法应该以线性时间完成所有可能的有效性检查，但仅在`(mFlags&1)==0`时才执行需要超线性时间的检查。未来的 SQLite 版本可能会使用 mFlags 参数的高阶位来指示额外的处理选项。

支持 xIntegrity 方法是在 SQLite 版本 3.44.0（2023-11-01）中添加的。在同一个发布版本中，xIntegrity 方法还被添加到许多内置虚拟表中，例如 FTS3，FTS5 和 RTREE，因此当运行 PRAGMA integrity_check 时，这些表的内容将自动进行一致性检查。
