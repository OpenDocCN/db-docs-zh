# 1\. 简介

> 原文：[`sqlite.com/vtab.html`](https://sqlite.com/vtab.html)

虚拟表是在打开的 SQLite 数据库连接 中注册的对象。从 SQL 语句的角度看，虚拟表对象看起来像任何其他表或视图。但在幕后，对虚拟表的查询和更新会调用虚拟表对象的回调方法，而不是直接读取和写入数据库文件。

虚拟表机制允许应用程序发布从 SQL 语句访问的接口，就像它们是表一样。SQL 语句对虚拟表几乎可以做任何对真实表可以做的事情，但以下几点除外：

+   不能在虚拟表上创建触发器。

+   不能在虚拟表上创建额外的索引。（虚拟表可以有索引，但必须内置到虚拟表实现中。不能使用 CREATE INDEX 语句单独添加索引。）

+   不能对虚拟表运行 ALTER TABLE ... ADD COLUMN 命令。

单个虚拟表实现可能会施加额外的约束。例如，某些虚拟实现可能提供只读表，或者某些虚拟表实现允许插入或删除，但不允许更新。或者某些虚拟表实现可能限制可以进行的更新类型。

虚拟表可能代表内存中的数据结构。或者它可能代表不在 SQLite 格式中的磁盘上数据的视图。或者应用程序可能根据需求计算虚拟表的内容。

下面列出了一些虚拟表的现有和假设用途：

+   全文搜索接口（fts3.html）

+   使用 R-Trees 的空间索引

+   检视 SQLite 数据库文件的磁盘内容（dbstat 虚拟表）

+   读取和/或写入逗号分隔值（CSV）文件的内容

+   访问主机计算机的文件系统，就像它是一个数据库表一样

+   允许在统计软件包（如 R）中进行 SQL 数据操作

查看 虚拟表列表 页面获取更多实际虚拟表实现的列表。

## 1.1\. 使用

使用 CREATE VIRTUAL TABLE 语句创建虚拟表。

**create-virtual-table-stmt:**

<svg class="pikchr" viewBox="0 0 624.096 259.848"><text x="74" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">CREATE</text> <text x="183" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">VIRTUAL</text> <text x="286" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">TABLE</text> <text x="372" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">IF</text> <text x="435" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">NOT</text> <text x="521" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">EXISTS</text> <text x="95" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模式名称</text> <text x="197" y="92" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="300" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表名</text> <text x="67" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">使用</text> <text x="187" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模块名称</text> <text x="300" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="432" y="204" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模块参数</text> <text x="563" y="204" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="432" y="242" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text></svg>

CREATE VIRTUAL TABLE 语句创建一个名为 table-name 的新表，该表派生自类 module-name。 module-name 是通过 sqlite3_create_module()接口为虚拟表注册的名称。

```sql
CREATE VIRTUAL TABLE tablename USING modulename;

```

也可以在模块名称后面提供逗号分隔的参数。

```sql
CREATE VIRTUAL TABLE tablename USING modulename(arg1, arg2, ...);

```

模块参数的格式非常通用。每个模块参数可能包含关键字、字符串文字、标识符、数字和标点符号。每个模块参数在创建虚拟表时作为文本传递给虚拟表实现的构造方法，构造方法负责解析和解释这些参数。参数语法足够通用，使得虚拟表实现可以根据需要将其参数解释为列定义在普通的 CREATE TABLE 语句中。实现也可以对参数施加其他解释。

一旦创建了虚拟表，它可以像任何其他表一样使用，但要注意特定虚拟表实现施加的例外情况。使用普通的 DROP TABLE 语法来销毁虚拟表。

### 1.1.1\. 临时虚拟表

没有"CREATE TEMP VIRTUAL TABLE"语句。要创建临时虚拟表，需在虚拟表名称前加上"temp"模式。

```sql
CREATE VIRTUAL TABLE temp.tablename USING module(arg1, ...);

```

### 1.1.2\. 同名虚拟表

某些虚拟表会自动存在于每个数据库连接的"main"模式中，即使没有 CREATE VIRTUAL TABLE 语句也是如此。这些虚拟表称为"同名虚拟表"。要使用同名虚拟表，只需像使用表一样使用模块名称即可。同名虚拟表仅存在于"main"模式中，因此如果前缀为不同的模式名称，则无法正常工作。

一个同名虚拟表的示例是 dbstat 虚拟表。要将 dbstat 虚拟表用作同名虚拟表，只需针对"dbstat"模块名称进行查询，就像它是普通表一样。（请注意，SQLite 必须使用 SQLITE_ENABLE_DBSTAT_VTAB 选项编译，才能在构建中包含 dbstat 虚拟表。）

```sql
SELECT * FROM dbstat;

```

如果其 xCreate 方法与 xConnect 方法完全相同，或者 xCreate 方法为空，则虚拟表被称为同名。xCreate 方法在使用 CREATE VIRTUAL TABLE 语句首次创建虚拟表时被调用。xConnect 方法在数据库连接附加或重新解析模式时被调用。当这两个方法相同时，表示虚拟表没有需要创建和销毁的持久状态。

### 1.1.3\. 仅同名虚拟表

如果 xCreate 方法为空，则禁止对该虚拟表使用 CREATE VIRTUAL TABLE 语句，并且该虚拟表称为“仅同名虚拟表”。仅同名虚拟表可用作表值函数。

请注意，在 3.9.0 版本（2015-10-14）之前，SQLite 在调用 xCreate 方法之前不会检查其是否为空。因此，如果在 SQLite3.8.11.1 版本（2015-07-29）或更早版本中注册了仅同名虚拟表，并尝试对该虚拟表模块使用 CREATE VIRTUAL TABLE 命令，则会发生对空指针的跳转，导致崩溃。

## 1.2\. 实现

几个新的 C 级对象被虚拟表实现所使用：

```sql
typedef struct sqlite3_vtab sqlite3_vtab;
typedef struct sqlite3_index_info sqlite3_index_info;
typedef struct sqlite3_vtab_cursor sqlite3_vtab_cursor;
typedef struct sqlite3_module sqlite3_module;

```

sqlite3_module 结构定义了一个模块对象，用于实现虚拟表。可以将模块看作是一个类，从中可以构造具有类似属性的多个虚拟表。例如，可以有一个模块提供对磁盘上逗号分隔值（CSV）文件的只读访问。然后可以使用这一个模块来创建多个虚拟表，其中每个虚拟表引用不同的 CSV 文件。

模块结构包含 SQLite 调用的方法，用于在虚拟表上执行各种操作，如创建虚拟表的新实例或销毁旧实例，读取和写入数据，搜索和删除、更新或插入行。模块结构将在下文中详细说明。

每个虚拟表实例由一个 sqlite3_vtab 结构表示。sqlite3_vtab 结构如下所示：

```sql
struct sqlite3_vtab {
  const sqlite3_module *pModule;
  int nRef;
  char *zErrMsg;
};

```

虚拟表实现通常会对此结构进行子类化，以添加额外的私有和特定于实现的字段。nRef 字段由 SQLite 核心内部使用，虚拟表实现不应更改它。虚拟表实现可以通过将错误消息字符串放入 zErrMsg 中向核心传递错误消息文本。必须从 SQLite 内存分配函数（例如 sqlite3_mprintf() 或 sqlite3_malloc()）获取用于保存此错误消息字符串的空间。在向 zErrMsg 分配新值之前，虚拟表实现必须使用 sqlite3_free() 释放 zErrMsg 的任何预先存在的内容。如果未执行此操作，将导致内存泄漏。当 SQLite 核心将错误消息文本传递给客户端应用程序或销毁虚拟表时，将释放并清零 zErrMsg 的内容。只有当虚拟表实现用新的、不同的错误消息覆盖内容时，才需要担心释放 zErrMsg 内容。

sqlite3_vtab_cursor 结构表示指向虚拟表特定行的指针。一个 sqlite3_vtab_cursor 看起来像这样：

```sql
struct sqlite3_vtab_cursor {
  sqlite3_vtab *pVtab;
};

```

再次强调，实际实现很可能会对此结构进行子类化，以添加额外的私有字段。

sqlite3_index_info 结构用于在实现虚拟表的模块的 xBestIndex 方法中传递信息。

在运行 CREATE VIRTUAL TABLE 语句之前，必须向数据库连接注册该语句中指定的模块。这可以通过 sqlite3_create_module() 或 sqlite3_create_module_v2() 接口来实现：

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

sqlite3_create_module() 和 sqlite3_create_module_v2() 程序将模块名称与一个 sqlite3_module 结构及特定于每个模块的单独客户端数据关联起来。这两种 create_module 方法之间唯一的区别在于 _v2 方法包括一个额外的参数，用于指定客户端数据指针的析构函数。模块结构定义了虚拟表的行为。模块结构如下所示：

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

模块结构定义了每个虚拟表对象的所有方法。模块结构还包含 iVersion 字段，该字段定义了模块表结构的特定版本。当前，iVersion 始终为 4 或更低版本，但在 SQLite 的未来版本中，模块结构的定义可能会扩展以包含额外的方法，此时最大的 iVersion 值将会增加。

模块结构的其余部分包括用于实现虚拟表各种功能的方法。每个方法的详细信息将在续篇中提供。

## 1.3\. 虚拟表和共享缓存

在 SQLite 版本 3.6.17（2009-08-10）之前，虚拟表机制假定每个数据库连接都保留其自己的数据库模式副本。因此，在启用共享缓存模式的数据库中无法使用虚拟表机制。如果启用了共享缓存模式，sqlite3_create_module()接口会返回错误。从 SQLite 版本 3.6.17 开始，这一限制得到了放宽。

## 1.4\. 创建新的虚拟表实现

按照以下步骤创建自己的虚拟表：

1.  编写所有必要的方法。

1.  创建一个包含从步骤 1 中所有方法指针的 sqlite3_module 结构。

1.  使用 sqlite3_create_module()或 sqlite3_create_module_v2()接口注册您的 sqlite3_module 结构。

1.  运行一个指定新模块的 CREATE VIRTUAL TABLE 命令中的 USING 子句。

唯一真正困难的部分是步骤 1。您可能希望从现有的虚拟表实现开始，并对其进行修改以适应您的需求。[SQLite 源码树](https://sqlite.org/src/dir?ci=trunk&type=tree)包含许多适合复制的虚拟表实现，包括：

+   **[templatevtab.c](https://sqlite.org/src/file/ext/misc/templatevtab.c)** → 一个专门用于作为其他自定义虚拟表模板的虚拟表。

+   **[series.c](https://sqlite.org/src/file/ext/misc/series.c)** → 实现了 generate_series()表值函数。

+   **[json.c](https://sqlite.org/src/file/src/json.c)** → 包含了 json_each()和 json_tree()表值函数的源码。

+   **[csv.c](https://sqlite.org/src/file/ext/misc/csv.c)** → 读取 CSV 文件的虚拟表。

SQLite 源码树中有许多其他虚拟表实现，可以作为示例。通过搜索"sqlite3_create_module"定位这些其他虚拟表实现。

你可能还想将新的虚拟表实现为可加载扩展。

# 2\. 虚拟表方法

## 2.1\. xCreate 方法

```sql
int (*xCreate)(sqlite3 *db, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

当响应于 CREATE VIRTUAL TABLE 语句时，将调用 xCreate 方法来创建虚拟表的新实例。如果 xCreate 方法与 xConnect 方法指向相同的指针，则该虚拟表是一个同名虚拟表。如果省略了 xCreate 方法（如果它是一个空指针），则该虚拟表是一个仅同名虚拟表。

db 参数是指向执行 CREATE VIRTUAL TABLE 语句的 SQLite 数据库连接 的指针。pAux 参数是客户端数据指针的副本，该指针是传递给 sqlite3_create_module() 或 sqlite3_create_module_v2() 调用的第四个参数，用于注册 虚拟表模块。argv 参数是一个包含 argc 个指向以空字符结尾的字符串的数组。第一个字符串，argv[0]，是正在调用的模块的名称。模块名称是作为第二个参数提供给 sqlite3_create_module()，并作为 CREATE VIRTUAL TABLE 语句的 USING 子句参数提供的名称。第二个字符串，argv[1]，是正在创建新虚拟表的数据库的名称。主数据库的数据库名称为"main"，TEMP 数据库的名称为"temp"，或者附加数据库语句末尾指定的名称。数组的第三个元素，argv[2]，是在 CREATE VIRTUAL TABLE 语句中 TABLE 关键字后指定的新虚拟表的名称。如果存在，argv[] 数组中的第四个及其后续字符串报告了在 CREATE VIRTUAL TABLE 语句中模块名称的参数。

此方法的作用是构造新的虚拟表对象（一个 sqlite3_vtab 对象），并在 *ppVTab 中返回指向它的指针。

作为创建新 sqlite3_vtab 结构的一部分，此方法必须调用 sqlite3_declare_vtab() 来告知 SQLite 核心关于虚拟表中的列和数据类型。sqlite3_declare_vtab() API 具有以下原型：

```sql
int sqlite3_declare_vtab(sqlite3 *db, const char *zCreateTable)

```

sqlite3_declare_vtab()方法的第一个参数必须与此方法的第一个参数相同，即数据库连接指针。sqlite3_declare_vtab()方法的第二个参数必须是一个以零结尾的 UTF-8 字符串，其中包含一个良好格式的 CREATE TABLE 语句，用于定义虚拟表中的列及其数据类型。在此 CREATE TABLE 语句中的表名会被忽略，以及所有的约束条件。只有列名和数据类型是重要的。CREATE TABLE 语句的字符串不需要持久保存在内存中。该字符串可以在 sqlite3_declare_vtab()例程返回后立即释放和/或重用。

xConnect 方法还可以通过调用 sqlite3_vtab_config()接口来选择性地请求虚拟表的特殊功能：

```sql
int sqlite3_vtab_config(sqlite3 *db, int op, ...);

```

调用 sqlite3_vtab_config()是可选的。但为了最大的安全性，建议虚拟表实现在虚拟表不会从触发器或视图中使用时调用"sqlite3_vtab_config(db, SQLITE_VTAB_DIRECTONLY)"。

xCreate 方法不需要初始化 sqlite3_vtab 对象的 pModule、nRef 和 zErrMsg 字段。SQLite 核心将处理这些琐事。

如果`xCreate`成功创建了新的虚拟表，则应返回 SQLITE_OK，否则返回 SQLITE_ERROR。如果未成功，则不能分配 sqlite3_vtab 结构。如果不成功，可以选择在*pzErr 中返回错误消息。必须使用 SQLite 内存分配函数（如 sqlite3_malloc()或 sqlite3_mprintf()）分配用于保存错误消息字符串的空间，因为 SQLite 核心将尝试使用 sqlite3_free()释放此空间，此操作在将错误报告给应用程序之后进行。

如果`xCreate`方法被省略（作为 NULL 指针），那么虚拟表就是一个仅同名虚拟表。不能使用 CREATE VIRTUAL TABLE 创建虚拟表的新实例，只能通过其模块名称使用虚拟表。请注意，SQLite 3.9.0（2015-10-14）之前的版本不理解仅同名虚拟表，如果尝试在仅同名虚拟表上使用 CREATE VIRTUAL TABLE，因为`xCreate`方法未检查为 null，将导致段错误。

如果`xCreate`方法与 xConnect 方法的指针完全相同，则表明虚拟表不需要初始化支持存储。这样的虚拟表可以用作同名虚拟表，也可以使用 CREATE VIRTUAL TABLE 作为具名虚拟表或两者兼而有之。

### 2.1.1\. 虚拟表中的隐藏列

如果列数据类型包含特殊关键字"HIDDEN"（大小写字母的任何组合），则该关键字将从列数据类型名称中省略，并在内部标记列为隐藏列。隐藏列在三个方面与普通列不同：

+   隐藏列未列在由"PRAGMA table_info"返回的数据集中，

+   隐藏列不包括在 SELECT 的结果集中"*"表达式的展开中，

+   隐藏列不包括在缺少显式列列表的 INSERT 语句的隐含列列表中。

例如，如果以下 SQL 传递给 sqlite3_declare_vtab()：

```sql
CREATE TABLE x(a HIDDEN VARCHAR(12), b INTEGER, c INTEGER Hidden);

```

然后虚拟表将以两个隐藏列创建，并且数据类型分别为"VARCHAR(12)"和"INTEGER"。

隐藏列的示例用法可以在 FTS3 虚拟表实现中看到，其中每个 FTS 虚拟表包含一个 FTS 隐藏列，用于从虚拟表传递信息到 FTS 辅助函数和 FTS MATCH 运算符。

### 2.1.2\. 表值函数

可以像 SELECT 语句的 FROM 子句中的表值函数一样使用包含隐藏列的 virtual table。表值函数的参数成为虚拟表的 HIDDEN 列的约束条件。

例如，“generate_series”扩展（位于[ext/misc/series.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/series.c)文件中的[源代码树](https://www.sqlite.org/src/tree?ci=trunk)）实现了一个 eponymous 虚拟表具有以下模式：

```sql
CREATE TABLE generate_series(
  value,
  start HIDDEN,
  stop HIDDEN,
  step HIDDEN
);

```

在此表的实现中，sqlite3_module.xBestIndex 方法检查针对 HIDDEN 列的等式约束，并将其用作确定生成整数“值”输出范围的输入参数。对于任何无约束列，使用合理的默认值。例如，要列出 5 到 50 之间的所有整数：

```sql
SELECT value FROM generate_series(5,50);

```

前面的查询等同于以下内容：

```sql
SELECT value FROM generate_series WHERE start=5 AND stop=50;

```

虚拟表名称上的参数与 hidden columns 按顺序匹配。参数数量可以少于隐藏列的数量，在这种情况下，后续的隐藏列是无约束的。但是，如果参数多于虚拟表中隐藏列的数量，则会导致错误。

### 2.1.3\. WITHOUT ROWID 虚拟表

从 SQLite 版本 3.14.0（2016-08-08）开始，传递给 sqlite3_declare_vtab() 的 CREATE TABLE 语句可以包含 WITHOUT ROWID 子句。这对于虚拟表行不能轻松映射为唯一整数的情况非常有用。包含 WITHOUT ROWID 的 CREATE TABLE 语句必须定义一个或多个列作为主键。主键的每一列必须单独为 NOT NULL，并且每行的所有列在集体上必须是唯一的。

注意，SQLite 对于 WITHOUT ROWID 虚拟表不强制执行主键约束。执行主键约束是底层虚拟表实现的责任。但 SQLite 假定主键约束有效 - 即所标识的列确实是唯一的且非空的 - 并利用此假设来优化针对虚拟表的查询。

在 WITHOUT ROWID 虚拟表上，rowid 列不可访问（当然）。

最初设计 xUpdate 方法时，它围绕着将 ROWID 作为单个值。 xUpdate 方法已扩展以适应任意的主键替代 ROWID，但主键仍必须仅为一列。因此，SQLite 将拒绝具有多个主键列和非空 xUpdate 方法的 WITHOUT ROWID 虚拟表。

## 2.2\. xConnect 方法

```sql
int (*xConnect)(sqlite3*, void *pAux,
             int argc, char *const*argv,
             sqlite3_vtab **ppVTab,
             char **pzErr);

```

xConnect 方法与 xCreate 非常相似。它具有相同的参数，并像 xCreate 一样构造一个新的 sqlite3_vtab 结构。并且它还必须像 xCreate 一样调用 sqlite3_declare_vtab()。它还应该像 xCreate 一样进行所有相同的 sqlite3_vtab_config()调用。

区别在于，xConnect 用于建立到现有虚拟表的新连接，而 xCreate 则用于从头开始创建新的虚拟表。

xCreate 和 xConnect 方法只有在虚拟表的虚拟表格存在某种必须在首次创建虚拟表格时初始化的后备存储时才有所不同。xCreate 方法创建并初始化后备存储。xConnect 方法只是连接到现有的后备存储。当 xCreate 和 xConnect 相同时，表格是一个 eponymous virtual table。

例如，考虑一个虚拟表实现，它提供对现有磁盘上逗号分隔值（CSV）文件的只读访问。对于这种虚拟表，不需要创建或初始化后备存储（因为 CSV 文件已经存在于磁盘上），因此 xCreate 和 xConnect 方法将在该模块中完全相同。

另一个例子是实现全文索引的虚拟表。xCreate 方法必须创建和初始化用于保存该索引的字典和倒排列表的数据结构。另一方面，xConnect 方法只需定位并使用先前 xCreate 调用创建的现有字典和倒排列表。

如果 xConnect 方法成功创建了新的虚拟表，必须返回 SQLITE_OK，否则返回 SQLITE_ERROR。如果不成功，不得分配 sqlite3_vtab 结构。如果不成功，可选地在 *pzErr 中返回错误消息。必须使用 SQLite 内存分配函数（如 sqlite3_malloc() 或 sqlite3_mprintf()）分配用于保存错误消息字符串的空间，因为 SQLite 核心会在向应用程序报告错误后尝试使用 sqlite3_free() 释放该空间。

xConnect 方法对于每个虚拟表实现都是必需的，尽管 xCreate 和 sqlite3_module 对象的 xConnect 指针可能指向同一个函数，如果虚拟表不需要初始化后备存储。

## 2.3\. xBestIndex 方法

SQLite 使用虚拟表模块的 xBestIndex 方法来确定访问虚拟表的最佳方式。xBestIndex 方法的原型如下：

```sql
int (*xBestIndex)(sqlite3_vtab *pVTab, sqlite3_index_info*);

```

SQLite 核心通过填充 sqlite3_index_info 结构的某些字段，并将指向该结构的指针作为第二个参数传递给 xBestIndex 方法，与 xBestIndex 方法进行通信。xBestIndex 方法填充该结构的其他字段，形成回复。sqlite3_index_info 结构如下所示：

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

注意 "estimatedRows"、"idxFlags" 和 colUsed 字段上的警告。这些字段分别是在 SQLite 版本 3.8.2、3.9.0 和 3.10.0 中添加的。任何读取或写入这些字段的扩展必须首先检查正在使用的 SQLite 库的版本是否大于或等于适当的版本，也许可以通过比较从 sqlite3_libversion_number() 返回的值与常量 3008002、3009000 和/或 3010000 来实现。在由旧版本 SQLite 创建的 sqlite3_index_info 结构中尝试访问这些字段的结果是未定义的。

此外，还有一些定义好的常量：

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

使用 sqlite3_vtab_collation() 接口来查找在评估第 i 个约束时应该使用的 排序序列 的名称。

```sql
const char *sqlite3_vtab_collation(sqlite3_index_info*, int i);

```

当 SQLite 核心编译涉及虚拟表的查询时，它会调用 xBestIndex 方法。换句话说，SQLite 在运行 sqlite3_prepare() 或其等效方法时调用此方法。通过调用此方法，SQLite 核心告诉虚拟表，它需要访问虚拟表中的一些行的子集，并且希望知道执行此访问的最有效方式。xBestIndex 方法会提供信息，SQLite 核心随后可以使用这些信息来进行对虚拟表的高效搜索。

在编译单个 SQL 查询时，SQLite 核心可能会多次调用 xBestIndex，使用不同的设置在 sqlite3_index_info 中。然后，SQLite 核心将选择似乎提供最佳性能的组合。

在调用此方法之前，SQLite 核心会使用关于当前正在尝试处理的查询的信息来初始化 sqlite3_index_info 结构的一个实例。这些信息主要来自查询的 WHERE 子句和 ORDER BY 或 GROUP BY 子句，但如果查询是一个连接，还来自任何 ON 或 USING 子句。SQLite 核心提供给 `xBestIndex` 方法的信息存储在标记为 "Inputs" 的结构部分中。"Outputs" 部分被初始化为零。

sqlite3_index_info 结构中的信息是临时的，一旦 `xBestIndex` 方法返回，它可能会被覆盖或释放。如果 `xBestIndex` 方法需要记住 sqlite3_index_info 结构的任何部分，应该进行复制。必须小心地将副本存储在一个会被释放的地方，比如在设置 `needToFreeIdxStr` 为 1 的 `idxStr` 字段中。

注意 `xBestIndex` 方法在调用 xFilter 方法之前，因为 `xBestIndex` 方法的 `idxNum` 和 `idxStr` 输出是 `xFilter` 方法所需的输入。但是，并不能保证 `xFilter` 方法在成功调用 `xBestIndex` 方法后一定会被调用。

对于每个虚拟表实现，`xBestIndex` 方法是必需的。

### 2.3.1\. 输入

SQLite 核心试图向虚拟表传达的主要内容是可用于限制需要搜索的行数的约束条件。`aConstraint[]` 数组包含每个约束的一个条目。该数组中将恰好有 nConstraint 个条目。

每个约束通常对应于 WHERE 子句或 USING 或 ON 子句中形式为的一个术语

> `column OP EXPR`

这里的 "column" 是虚拟表中的列，"OP" 是像 "=" 或 "<" 这样的操作符，"EXPR" 是任意表达式。因此，例如，如果 WHERE 子句包含这样的一个术语：

```sql
a = 5

```

然后，其中一个约束将在“a”列上，操作符为“=”，表达式为“5”。约束不需要具有 WHERE 子句的文字表示。查询优化器可能会对 WHERE 子句进行转换，以提取尽可能多的约束。因此，例如，如果 WHERE 子句包含类似以下内容：

```sql
x BETWEEN 10 AND 100 AND 999>y

```

查询优化器可能会将此转换为三个单独的约束：

```sql
x >= 10
x <= 100
y < 999

```

对于每个这样的约束条件，aConstraint[].iColumn 字段指示约束条件左侧显示的列。虚拟表的第一列是列 0。虚拟表的行 id 是列 -1。aConstraint[].op 字段指示使用的操作符。SQLITE_INDEX_CONSTRAINT_* 常量将整数常量映射为操作符值。列按照调用 sqlite3_declare_vtab() 中定义的顺序出现在 xCreate 或 xConnect 方法中。隐藏列在确定列索引时计算在内。

如果虚拟表的 xFindFunction() 方法已定义，并且如果 xFindFunction() 有时返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大，则约束也可以是以下形式之一：

> FUNCTION( column, EXPR)

在这种情况下，aConstraint[].op 值与 xFindFunction() 为 FUNCTION 返回的值相同。

aConstraint[] 数组包含关于适用于虚拟表的所有约束的信息。但是由于表在连接中的顺序，某些约束可能无法使用。因此，xBestIndex 方法只能考虑具有 aConstraint[].usable 标志为 true 的约束。

除了 WHERE 子句约束之外，SQLite 核心还告诉 xBestIndex 方法 ORDER BY 子句的信息。（在聚合查询中，SQLite 核心可能会放置 GROUP BY 子句信息以替代 ORDER BY 子句信息，但这个事实不应影响 xBestIndex 方法。）如果 ORDER BY 子句的所有项都是虚拟表中的列，则 nOrderBy 将是 ORDER BY 子句中项的数量，并且 aOrderBy[] 数组将标识每个 ORDER BY 子句项的列以及该列是否为 ASC 或 DESC。

在 SQLite 版本 3.10.0（2016-01-06）及以后的版本中，colUsed 字段可用于指示语句准备过程中实际使用的虚拟表字段。如果 colUsed 的最低位被设置，这意味着使用了第一列。次低位对应第二列，依此类推。如果 colUsed 的最高位被设置，这意味着使用了除了前 63 列之外的一列或多列。如果 xFilter 方法需要列使用信息，则必须将所需的位编码为输出 idxNum 字段或 idxStr 内容中。

#### 2.3.1.1\. LIKE、GLOB、REGEXP 和 MATCH 函数

对于 LIKE、GLOB、REGEXP 和 MATCH 运算符，aConstraint[].iColumn 的值是左操作数的虚拟表列。但是，如果这些运算符表示为函数调用而不是操作符，则 aConstraint[].iColumn 的值将引用作为该函数第二个参数的虚拟表列：

> LIKE(*EXPR*, *column*)
> 
> GLOB(*EXPR*, *column*)
> 
> REGEXP(*EXPR*, *column*)
> 
> MATCH(*EXPR*, *column*)

因此，就 xBestIndex() 方法而言，以下两种形式是等效的：

> *column* LIKE *EXPR*
> 
> LIKE(*EXPR*, *column*)

仅对于 LIKE、GLOB、REGEXP 和 MATCH 函数，这种查看函数第二个参数的特殊行为才会发生。对于所有其他函数，aConstraint[].iColumn 值引用函数的第一个参数。

LIKE、GLOB、REGEXP 和 MATCH 的这种特殊特性不适用于 xFindFunction()方法。然而，xFindFunction()方法总是依赖于 LIKE、GLOB、REGEXP 或 MATCH 操作符的左操作数，但依赖于这些操作符的函数调用等价物的第一个参数。

#### 2.3.1.2\. LIMIT 和 OFFSET

当 aConstraint[].op 为 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，这表明在使用虚拟表的 SQL 查询语句中存在 LIMIT 或 OFFSET 子句。LIMIT 和 OFFSET 操作符没有左操作数，因此当 aConstraint[].op 为 SQLITE_INDEX_CONSTRAINT_LIMIT 或 SQLITE_INDEX_CONSTRAINT_OFFSET 之一时，aConstraint[].iColumn 值是无意义的，不应使用。

#### 2.3.1.3\. 约束的右操作数值

sqlite3_vtab_rhs_value() 接口可用于尝试访问约束的右操作数。但是，在运行 xBestIndex 方法时，右操作符的值可能未知，因此 sqlite3_vtab_rhs_value()调用可能不成功。通常，在输入 SQL 中将右操作数编码为字面值时，xBestIndex 才能访问约束的右操作数。如果将右操作数编码为表达式或主机参数，则 xBestIndex 可能无法访问它。某些操作符，如 SQLITE_INDEX_CONSTRAINT_ISNULL 和 SQLITE_INDEX_CONSTRAINT_ISNOTNULL，没有右操作数。sqlite3_vtab_rhs_value()接口对此类操作符始终返回 SQLITE_NOTFOUND。

### 2.3.2\. 输出

综上所述，xBestIndex 方法的工作是找出搜索虚拟表的最佳方法。

xBestIndex 方法通过 idxNum 和 idxStr 字段向 xFilter 方法传递索引策略。在 SQLite 核心的视角下，idxNum 值和 idxStr 字符串内容是任意的，并且只要 xBestIndex 和 xFilter 就这些内容达成一致意见，它们就可以具有任何意义。SQLite 核心仅仅是将 xBestIndex 中的信息复制到 xFilter 方法中，假设 idxStr 引用的字符序列以 NUL 结尾。

idxStr 值可以是从 SQLite 内存分配函数（如 sqlite3_mprintf()）获取的字符串。如果是这种情况，则必须将 needToFreeIdxStr 标志设置为 true，以便 SQLite 核心在完成后调用 sqlite3_free() 释放该字符串，从而避免内存泄漏。idxStr 值也可以是静态常量字符串，此时 needToFreeIdxStr 布尔值应保持为 false。

estimatedCost 字段应设置为执行针对虚拟表的此查询所需的估计磁盘访问操作次数。SQLite 核心通常会多次调用 xBestIndex，使用不同的约束条件获取多个成本估算，然后选择提供最低估算的查询计划。SQLite 核心在调用 xBestIndex 前将 estimatedCost 初始化为非常大的值，因此如果 xBestIndex 确定当前参数组合不理想，可以保持 estimatedCost 字段不变，以避免使用该计划。

如果当前版本的 SQLite 是 3.8.2 或更高版本，则 estimatedRows 字段可以设置为建议查询计划返回的行数的估计值。如果没有明确设置此值，则使用默认估计值 25 行。

如果当前版本的 SQLite 是 3.9.0 或更高版本，则 idxFlags 字段可以设置为 SQLITE_INDEX_SCAN_UNIQUE，以指示虚拟表在给定输入约束时仅返回零行或一行。在后续版本的 SQLite 中，可能会理解 idxFlags 字段的其他位。

aConstraintUsage[] 数组包含 sqlite3_index_info 结构中输入部分的 nConstraint 约束的每个元素。aConstraintUsage[] 数组由 xBestIndex 使用，用于告诉核心它如何使用这些约束。

方法 xBestIndex 可能会将 aConstraintUsage[].argvIndex 的条目设置为大于零的值。应该设置一个条目为 1，另一个为 2，再另一个为 3，依此类推，直到 xBestIndex 方法想要的数目为止。然后，相应约束的 EXPR 将作为 argv[] 参数传递给 xFilter。

例如，如果 aConstraint[3].argvIndex 被设置为 1，那么当调用 xFilter 时，传递给 xFilter 的 argv[0] 将具有 aConstraint[3] 约束的 EXPR 值。

#### 2.3.2.1\. 在字节码中省略约束检查

默认情况下，SQLite 生成的 字节码 将在虚拟表的每一行上双重检查所有约束，以验证它们是否满足。如果虚拟表能够保证某个约束始终被满足，它可以尝试通过设置 aConstraintUsage[].omit 来抑制该双重检查。然而，除了某些例外情况外，这只是一个提示，并不能保证约束的冗余检查将被抑制。关键点：

+   仅当约束的 argvIndex 值大于 0 且小于或等于 16 时，才会遵循省略标志。对于未将右操作数传递给 xFilter 方法的约束，永远不会抑制约束检查。当前实现只能抑制对前 16 个传递给 xFilter 的值的冗余约束检查，尽管该限制在未来版本中可能会增加。

+   只要 argvIndex 大于 0，对于 SQLITE_INDEX_CONSTRAINT_OFFSET 约束，总是尊重省略标志。在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置省略标志，表示虚拟表本身将抑制输出的前 N 行，其中 N 是 OFFSET 操作符的右操作数。如果虚拟表实现在 SQLITE_INDEX_CONSTRAINT_OFFSET 约束上设置省略标志，但未能抑制输出的前 N 行，则整体查询结果将不正确。

#### 2.3.2.2\. ORDER BY 和 orderByConsumed

如果虚拟表将按照 ORDER BY 子句指定的顺序输出行，则 orderByConsumed 标志可以设置为 true。如果输出不是自动以正确顺序，则必须保持 orderByConsumed 在其默认 false 设置状态。这将告知 SQLite 核心，在虚拟表输出后，需要对数据进行单独的排序处理。设置 orderByConsumed 是一种优化。如果将 orderByConsumed 设置为其默认值 (0)，查询将始终获得正确答案。如果错误地设置 orderByConsumed，则可能导致答案错误。建议新的虚拟表实现最初不设置 orderByConsumed 值，然后在确保一切正常工作之后，返回并尝试通过适当设置 orderByConsumed 进行优化。

有时，即使虚拟表的输出严格不按 nOrderBy 和 aOrderBy 指定的顺序，orderByConsumed 标志也可以安全设置。如果 sqlite3_vtab_distinct() 接口返回 1 或 2，则表示可以放宽排序要求。有关详细信息，请参阅关于 sqlite3_vtab_distinct() 的文档。

### 2.3.3\. 返回值

xBestIndex 方法在成功时应返回 SQLITE_OK。如果发生任何致命错误，则应返回适当的错误代码（例如：SQLITE_NOMEM）。

如果 xBestIndex 返回 SQLITE_CONSTRAINT，这并不表示错误。相反，SQLITE_CONSTRAINT 表示指定的输入参数组合对于虚拟表来说是不足的。这在逻辑上与将 estimatedCost 设置为无穷大相同。如果针对特定查询计划的每次 xBestIndex 调用都返回 SQLITE_CONSTRAINT，则表示无法安全使用虚拟表，并且 sqlite3_prepare() 调用将失败并显示“no query solution”错误。

### 2.3.4\. 强制表值函数的必需参数

xBestIndex 返回的 SQLITE_CONSTRAINT 对于有必需参数的表值函数很有用。如果一个必需参数在 aConstraint[].usable 字段为 false，则 xBestIndex 方法应返回 SQLITE_CONSTRAINT。如果一个必需字段根本不出现在 aConstraint[] 数组中，这意味着输入的 SQL 中省略了相应的参数。在这种情况下，xBestIndex 应该在 pVTab->zErrMsg 中设置错误消息并返回 SQLITE_ERROR。总结一下：

1.  对于一个必需参数，aConstraint[].usable 的值为 false → 返回 SQLITE_CONSTRAINT。

1.  一个必需参数在 aConstraint[] 数组中根本没有出现 → 在 pVTab->zErrMsg 中设置错误消息并返回 SQLITE_ERROR。

下面的例子将更好地说明从 xBestIndex 返回 SQLITE_CONSTRAINT 作为返回值的用法：

```sql
SELECT * FROM realtab, tablevaluedfunc(realtab.x);

```

假设“tablevaluedfunc”的第一个隐藏列是“param1”，则上述查询在语义上等同于这个：

```sql
SELECT * FROM realtab, tablevaluedfunc
 WHERE tablevaluedfunc.param1 = realtab.x;

```

查询规划器必须在此查询的许多可能实现之间做出决策，但特别注意两个计划：

1.  扫描 realtab 的所有行，并对于每一行，在 tablevaluedfunc 中查找 param1 等于 realtab.x 的行。

1.  扫描 tablevaluedfunc 的所有行，并对于每一行，在 realtab 中查找 x 等于 tablevaluedfunc.param1 的行。

方法 xBestIndex 将针对上述每个潜在计划调用一次。对于计划 1，对于 param1 列的 SQLITE_CONSTRAINT_EQ 约束的 aConstraint[].usable 标志将为 true，因为“param1 =？”约束的右侧值将是已知的，这是由外部 realtab 循环确定的。但对于计划 2，"param1 = ?" 的 aConstraint[].usable 标志将为 false，因为右侧值由内部循环确定，因此是一个未知量。由于 param1 是表值函数的必需输入，所以当 xBestIndex 方法被提供计划 2 时，应返回 SQLITE_CONSTRAINT，表示缺少必需的输入。这将强制查询规划器选择计划 1。

## 2.4\. xDisconnect 方法

```sql
int (*xDisconnect)(sqlite3_vtab *pVTab);

```

此方法释放虚拟表的连接。只有 sqlite3_vtab 对象被销毁。虚拟表不会被销毁，并且与虚拟表关联的任何后备存储将保持存在。此方法撤消了 xConnect 的工作。

此方法是虚拟表连接的析构函数。与 xDestroy 相比，此方法是整个虚拟表的析构函数。

对于每个虚拟表实现，xDisconnect 方法是必需的，尽管如果对于特定虚拟表而言 xDisconnect 和 xDestroy 方法是相同的函数，则可以接受。

## 2.5\. xDestroy 方法

```sql
int (*xDestroy)(sqlite3_vtab *pVTab);

```

此方法释放对虚拟表的连接，就像 xDisconnect 方法一样，并且还销毁了底层表实现。此方法撤消了 xCreate 的工作。

每当使用虚拟表的数据库连接关闭时，都会调用 xDisconnect 方法。仅当执行 DROP TABLE 语句针对虚拟表时，才会调用 xDestroy 方法。

对于每个虚拟表实现，xDestroy 方法是必需的，尽管如果对于特定虚拟表而言 xDisconnect 和 xDestroy 方法是相同的函数，则可以接受。

## 2.6\. xOpen 方法

```sql
int (*xOpen)(sqlite3_vtab *pVTab, sqlite3_vtab_cursor **ppCursor);

```

xOpen 方法创建一个用于访问（读取和/或写入）虚拟表的新游标。成功调用此方法将为 sqlite3_vtab_cursor（或其子类）分配内存，初始化新对象，并使 *ppCursor 指向新对象。成功调用然后返回 SQLITE_OK。

对于每次成功调用此方法，SQLite 核心将稍后调用 xClose 方法来销毁分配的游标。

xOpen 方法不需要初始化 sqlite3_vtab_cursor 结构体的 pVtab 字段。SQLite 核心会自动处理这个任务。

一个虚拟表实现必须能够支持任意数量的同时打开的游标。

当初始打开时，光标处于未定义状态。SQLite 核心会在任何尝试定位或从光标读取之前调用游标上的 xFilter 方法。

每个虚拟表实现都需要 xOpen 方法。

## 2.7\. xClose 方法

```sql
int (*xClose)(sqlite3_vtab_cursor*);

```

xClose 方法会关闭先前由 xOpen 打开的游标。SQLite 核心总会对每个使用 xOpen 打开的游标调用一次 xClose。

该方法必须释放由相应 xOpen 调用分配的所有资源。即使返回错误，也不会再次调用该例程。SQLite 核心在关闭后不会再使用 sqlite3_vtab_cursor。

每个虚拟表实现都需要 xClose 方法。

## 2.8\. xEof 方法

```sql
int (*xEof)(sqlite3_vtab_cursor*);

```

如果指定的游标当前指向有效的数据行，则 xEof 方法必须返回 false（零），否则返回 true（非零）。这个方法会在每次 xFilter 和 xNext 调用后立即由 SQL 引擎调用。

每个虚拟表实现都需要 xEof 方法。

## 2.9\. xFilter 方法

```sql
int (*xFilter)(sqlite3_vtab_cursor*, int idxNum, const char *idxStr,
              int argc, sqlite3_value **argv);

```

该方法开始对虚拟表进行搜索。第一个参数是由 xOpen 打开的游标。接下来的两个参数定义了先前由 xBestIndex 选择的特定搜索索引。只要 xFilter 和 xBestIndex 就其含义达成一致，idxNum 和 idxStr 的具体含义就不重要。

xBestIndex 函数可能已经使用了 sqlite3_index_info 结构的 aConstraintUsage[].argvIndex 值来请求某些表达式的值。这些值通过 argc 和 argv 参数传递给 xFilter。

如果虚拟表包含一个或多个符合搜索条件的行，则游标必须指向第一行。后续对 xEof 的调用必须返回 false（零）。如果没有匹配的行，则游标必须处于一种状态，将导致 xEof 返回 true（非零）。SQLite 引擎将使用 xColumn 和 xRowid 方法访问该行内容。xNext 方法将用于前进到下一行。

如果成功，此方法必须返回 SQLITE_OK，或者如果发生错误，则返回 sqlite 错误代码。

每个虚拟表实现都需要 xFilter 方法。

## 2.10\. The xNext Method

```sql
int (*xNext)(sqlite3_vtab_cursor*);

```

xNext 方法将一个 虚拟表游标 前进到由 xFilter 启动的结果集的下一行。如果在调用此例程时游标已经指向最后一行，则游标不再指向有效数据，后续对 xEof 方法的调用必须返回 true（非零）。如果成功将游标前进到另一行内容，则后续对 xEof 的调用必须返回 false（零）。

如果成功，此方法必须返回 SQLITE_OK，或者如果发生错误，则返回 sqlite 错误代码。

每个虚拟表实现都需要 xNext 方法。

## 2.11\. The xColumn Method

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

如果 xColumn 方法实现不调用上述任何函数，则列的值默认为 SQL NULL。

要引发错误，xColumn 方法应使用其中一种 result_text() 方法设置错误消息文本，然后返回适当的 错误代码。xColumn 方法在成功时必须返回 SQLITE_OK。

每个虚拟表实现都需要 xColumn 方法。

## 2.12\. **xRowid 方法**

```sql
int (*xRowid)(sqlite3_vtab_cursor *pCur, sqlite_int64 *pRowid);

```

如果成功调用此方法，将使 *pRowid* 填充为当前虚拟表游标 *pCur* 所指向的行的 rowid。此方法在成功时返回 SQLITE_OK，在失败时返回适当的 错误代码。

每个虚拟表实现都需要 xRowid 方法。

## 2.13\. **xUpdate 方法**

```sql
int (*xUpdate)(
  sqlite3_vtab *pVTab,
  int argc,
  sqlite3_value **argv,
  sqlite_int64 *pRowid
);

```

所有对虚拟表的更改都使用 xUpdate 方法进行。此方法可以用于插入、删除或更新。

argc 参数指定了 argv 数组中的条目数。对于纯删除操作，argc 的值为 1；对于插入、替换或更新操作，argc 的值为 N+2，其中 N 是表中的列数。在前一句中，N 包括任何隐藏列。

每个 argv 条目在 C 中都有非 NULL 值，但可能包含 SQL 值 NULL。换句话说，对于 **i** 在 0 到 `argc-1` 之间的 **i**，总是成立 `argv[i]!=0`。但有可能 `sqlite3_value_type(argv[i])==SQLITE_NULL`。

argv[0] 参数是要删除的虚拟表中行的 rowid。如果 argv[0] 是 SQL NULL，则不执行删除操作。

argv[1] 参数是要插入到虚拟表中的新行的 rowid。如果 argv[1] 是 SQL NULL，则实现必须为新插入的行选择一个 rowid。随后的 argv[] 条目包含按声明的列顺序的虚拟表的列的值。列数将与 xConnect 或 xCreate 方法使用 sqlite3_declare_vtab() 调用所做的表声明匹配。所有隐藏列都包括在内。

在执行无 rowid 插入（argc > 1，argv[1] 是 SQL NULL）时，对使用 ROWID 的虚拟表（但不适用于 WITHOUT ROWID 虚拟表），实现必须将 *pRowid 设置为新插入行的 rowid；这将成为 sqlite3_last_insert_rowid() 函数返回的值。在所有其他情况下设置这个值是无害的无操作；如果 argc==1 或 argv[1] 不是 SQL NULL，则 SQLite 引擎会忽略 *pRowid 返回值。

每次调用 xUpdate 都会落入下面显示的一个案例中。注意到对 **argv[i]** 的引用意味着 argv[i] 对象中保存的 SQL 值，而不是 argv[i] 对象本身。

> **argc = 1**
> 
> argv[0] ≠ NULL**
> 
> DELETE: 删除 rowid 或 PRIMARY KEY 等于 argv[0] 的单行记录。不进行插入操作。
> 
> **argc > 1**
> 
> argv[0] = NULL**
> 
> INSERT: 插入一行新记录，列值来自于 argv[2] 及其后的参数。在一个 rowid 虚拟表中，如果 argv[1] 是 SQL NULL，则会自动生成一个新的唯一 rowid。对于 WITHOUT ROWID 虚拟表，argv[1] 将为 NULL，此时实现应从 argv[2] 及其后的参数中适当的列中获取 PRIMARY KEY 值。
> 
> **argc > 1**
> 
> argv[0] ≠ NULL
> 
> argv[0] = argv[1]**
> 
> UPDATE: 使用 argv[2] 及其后参数中的新值更新具有 rowid 或 PRIMARY KEY argv[0] 的行。
> 
> **argc > 1**
> 
> argv[0] ≠ NULL
> 
> argv[0] ≠ argv[1]**
> 
> 使用 rowid 或 PRIMARY KEY 更改：当 SQL 语句更新一个 rowid 时，例如语句：
> 
> > 更新表 SET rowid=rowid+1 WHERE ...;

只有在 xUpdate 方法成功时，xUpdate 方法必须返回 SQLITE_OK。如果发生失败，则 xUpdate 必须返回适当的错误代码。在失败时，pVTab->zErrMsg 元素可以选择性地被从 SQLite 中分配内存存储的错误消息文本替换，使用诸如 sqlite3_mprintf()或 sqlite3_malloc()等函数。

如果 xUpdate 方法违反虚拟表的某些约束（包括但不限于尝试存储错误数据类型的值、尝试存储过大或过小的值，或尝试更改只读值），则 xUpdate 必须以适当的错误代码失败。

如果 xUpdate 方法正在执行 UPDATE 操作，那么可以使用 sqlite3_value_nochange(X) 来查找实际上被 UPDATE 语句修改的虚拟表的哪些列。sqlite3_value_nochange(X) 接口对于不变的列返回 true。在每次 UPDATE 操作时，SQLite 首先分别调用 xColumn 来获取表中每个不变列的值。xColumn 方法可以通过调用 sqlite3_vtab_nochange() 来检查 SQL 级别上列是否未改变。如果 xColumn 发现该列未被修改，则应该在不使用 sqlite3_result_xxxxx() 接口设置结果的情况下返回。只有在这种情况下，xUpdate 方法内部的 sqlite3_value_nochange() 才会返回 true。如果 xColumn 调用了一个或多个 sqlite3_result_xxxxx() 接口，则 SQLite 将理解为列值的更改，并且 xUpdate 方法内部对该列的 sqlite3_value_nochange() 调用将返回 false。

当 xUpdate 方法被调用时，在虚拟表实例上可能会打开并使用一个或多个 sqlite3_vtab_cursor 对象，甚至在虚拟表的行上也可能如此。xUpdate 的实现必须准备好处理从其他现有游标中删除或修改表行的尝试。如果虚拟表不能容纳这些更改，则 xUpdate 方法必须返回一个 错误代码。

xUpdate 方法是可选的。如果 sqlite3_module 中虚拟表的 xUpdate 指针是 NULL 指针，那么该虚拟表是只读的。

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

在调用 sqlite3_prepare()期间，会给虚拟表实现方法一个重载函数的机会。如果将此方法设置为 NULL，则不会发生重载。

当函数使用虚拟表列作为其第一个参数时，会调用此方法以查看虚拟表是否希望重载函数。前三个参数是输入：虚拟表、函数的参数数量和函数的名称。如果不需要重载，则此方法返回 0。若要重载函数，则此方法将新的函数实现写入*pxFunc 并将用户数据写入*ppArg，然后返回 1 或介于 SQLITE_INDEX_CONSTRAINT_FUNCTION 和 255 之间的数字。

历史上，xFindFunction() 的返回值为零或一。零表示函数未重载，而一表示已重载。在版本 3.25.0 (2018-09-15) 中添加了返回值为 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更高的能力。如果 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更高的值，则意味着该函数接受两个参数，并且该函数可以在查询的 WHERE 子句中用作布尔值，虚拟表能够利用该函数加速查询结果。当 xFindFunction 返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 或更大的值时，返回值将成为传递给 xBestIndex() 的约束之一的 sqlite3_index_info.aConstraint.op 值。函数的第一个参数是由约束的 aConstraint[].iColumn 字段标识的列，而第二个参数是将传递给 xFilter()（如果设置了 aConstraintUsage[].argvIndex 值）或从 sqlite3_vtab_rhs_value() 返回的值。

Geopoly 模块 是一个使用 SQLITE_INDEX_CONSTRAINT_FUNCTION 来提高性能的虚拟表的示例。Geopoly 的 xFindFunction() 方法返回 SQLITE_INDEX_CONSTRAINT_FUNCTION 以支持 geopoly_overlap() SQL 函数，并且返回 SQLITE_INDEX_CONSTRAINT_FUNCTION+1 以支持 geopoly_within() SQL 函数。这允许优化查询，例如：

```sql
SELECT * FROM geopolytab WHERE geopoly_overlap(_shape, $query_polygon);
SELECT * FROM geopolytab WHERE geopoly_within(_shape, $query_polygon);

```

注意，中缀函数（LIKE、GLOB、REGEXP 和 MATCH）颠倒了它们参数的顺序。因此，"like(A,B)"通常与"B like A"相同。但是，xFindFunction()始终查看最左边的参数，而不是第一个逻辑参数。因此，对于形式"B like A"，SQLite 查看左操作数"B"，如果该操作数是虚拟表列，则调用该虚拟表的 xFindFunction()方法。但是，如果使用了形式"like(A,B)"，则 SQLite 会检查 A 项，看它是否是虚拟表的列，如果是，则为列 A 的虚拟表调用 xFindFunction()方法。

此例程返回的函数指针必须在第一个参数中给出的 sqlite3_vtab 对象的生命周期内有效。

## 2.15\. xBegin 方法

```sql
int (*xBegin)(sqlite3_vtab *pVTab);

```

此方法在虚拟表上启动事务。此方法是可选的。sqlite3_module 的 xBegin 指针可能为 NULL。

此方法始终在调用 xCommit 或 xRollback 方法之一之后调用。虚拟表事务不会嵌套，因此在单个虚拟表上不会多次调用 xBegin 方法，而不在 xCommit 或 xRollback 之间进行调用。在 xBegin 和相应的 xCommit 或 xRollback 之间可能会多次调用其他方法。

## 2.16\. xSync 方法

```sql
int (*xSync)(sqlite3_vtab *pVTab);

```

此方法表示虚拟表上的两阶段提交的开始。此方法是可选的。sqlite3_module 的 xSync 指针可能为 NULL。

此方法仅在调用 xBegin 方法之后并在 xCommit 或 xRollback 之前调用。为了实现两阶段提交，在任何虚拟表上调用 xCommit 方法之前，会先调用所有虚拟表上的 xSync 方法。如果任何 xSync 方法失败，则整个事务将回滚。

## 2.17\. xCommit 方法

```sql
int (*xCommit)(sqlite3_vtab *pVTab);

```

此方法导致虚拟表事务提交。此方法是可选的。sqlite3_module 的 xCommit 指针可能为 NULL。

调用此方法总是在之前调用了 xBegin 和 xSync 方法。

## 2.18\. xRollback 方法

```sql
int (*xRollback)(sqlite3_vtab *pVTab);

```

此方法导致虚拟表事务回滚。此方法是可选的。sqlite3_module 的 xRollback 指针可能为 NULL。

调用此方法总是在之前调用了 xBegin 方法之后。

## 2.19\. xRename 方法

```sql
int (*xRename)(sqlite3_vtab *pVtab, const char *zNew);

```

此方法通知虚拟表实现，虚拟表将被赋予一个新名称。如果此方法返回 SQLITE_OK，则 SQLite 将重命名表。如果此方法返回错误代码，则阻止重命名。

xRename 方法是可选的。如果省略，则无法使用 ALTER TABLE RENAME 命令重命名虚拟表。

在调用此方法之前启用了 PRAGMA legacy_alter_table 设置，并且在此方法完成后恢复了 legacy_alter_table 的值。对于使用影子表的虚拟表的正确操作是必需的，其中影子表必须重命名以匹配新的虚拟表名称。如果 legacy_alter_format 关闭，则每次 xRename 方法尝试更改影子表名称时，都会为虚拟表调用 xConnect 方法。

## 2.20\. xSavepoint、xRelease 和 xRollbackTo 方法

```sql
int (*xSavepoint)(sqlite3_vtab *pVtab, int);
int (*xRelease)(sqlite3_vtab *pVtab, int);
int (*xRollbackTo)(sqlite3_vtab *pVtab, int);

```

这些方法为虚拟表实现提供了实现嵌套事务的机会。它们始终是可选的，仅在 SQLite 版本 3.7.7（2011-06-23）及更高版本中才会调用。

当调用 xSavepoint(X,N) 时，这表示虚拟表 X 应将其当前状态保存为保存点 N。随后调用 xRollbackTo(X,R) 表示虚拟表的状态应返回到最后一次调用 xSavepoint(X,R) 时的状态。调用 xRollbackTo(X,R) 将使所有 N>R 的保存点失效；在重新初始化之前通过调用 xSavepoint()，这些失效的保存点都不会回滚或释放。调用 xRelease(X,M) 会使所有 N>=M 的保存点失效。

除了在调用 xBegin() 和 xCommit() 或 xRollback() 之间，xSavepoint()、xRelease() 或 xRollbackTo() 方法将永远不会被调用。

## 2.21\. xShadowName 方法

一些虚拟表的实现（例如：FTS3、FTS5 和 RTREE）利用真实（非虚拟）数据库表存储内容。例如，当内容插入到 FTS3 虚拟表时，数据最终存储在名为 "%_content"、"%_segdir"、"%_segments"、"%_stat" 和 "%_docsize" 的真实表中，其中 "%" 是原始虚拟表的名称。存储虚拟表内容的这些辅助真实表称为 "影子表"。有关更多信息，请参见 (1)、(2) 和 (3)。

xShadowName 方法存在是为了允许 SQLite 确定某个真实表是否实际上是虚拟表的影子表。

如果以下所有条件都满足，SQLite 将理解真实表为影子表：

+   表名包含一个或多个 "_" 字符。

+   在最后一个 "_" 之前的部分与使用 [CREATE VIRTUAL TABLE](https://sqlite.org/lang_createvtab.html) 创建的虚拟表的名称完全匹配。（不识别影子表为 [同名虚拟表](https://sqlite.org/vtab.html#epovtab) 和 [表值函数](https://sqlite.org/vtab.html#tabfunc2)。）

+   虚拟表包含一个 xShadowName 方法。

+   当其输入是表名中最后一个 "_" 字符后的部分时，xShadowName 方法返回 true。

如果 SQLite 将表识别为影子表，并设置了 [SQLITE_DBCONFIG_DEFENSIVE](https://sqlite.org/c3ref/c_dbconfig_defensive.html#sqlitedbconfigdefensive) 标志，则普通 SQL 语句对于影子表是只读的。影子表仍然可以被写入，但只能通过某个虚拟表实现的方法中调用的 SQL 进行。

xShadowName 方法的整个目的是保护影子表的内容免受恶意 SQL 的破坏。每个使用影子表的虚拟表实现都应能检测和处理被破坏的影子表内容。然而，特定虚拟表实现中的错误可能会允许故意破坏的影子表导致崩溃或其他故障。xShadowName 机制旨在通过防止普通 SQL 语句故意破坏影子表来避免零日攻击。

影子表默认为读/写。只有在使用 [sqlite3_db_config()](https://sqlite.org/c3ref/db_config.html) 设置了 [SQLITE_DBCONFIG_DEFENSIVE](https://sqlite.org/c3ref/c_dbconfig_defensive.html#sqlitedbconfigdefensive) 标志后，影子表才会变为只读。影子表需要默认为读/写，以保持向后兼容性。例如，[CLI](https://sqlite.org/cli.html) 的 [.dump](https://sqlite.org/cli.html#dump) 命令生成的 SQL 文本直接写入影子表中。

## 2.22\. xIntegrity 方法

如果 sqlite3_module 的 iVersion 为 4 或更高，并且 xIntegrity 方法不为 NULL，则 PRAGMA integrity_check 和 PRAGMA quick_check 命令将在其处理中调用 xIntegrity。如果 xIntegrity 方法将错误消息字符串写入第五个参数，则 PRAGMA integrity_check 将将该错误作为其输出的一部分报告。换句话说，xIntegrity 方法允许 PRAGMA integrity_check 命令验证存储在虚拟表中的内容的完整性。

xIntegrity 方法调用时带有五个参数：

+   **pVTab** → 指向虚拟表的对象 sqlite3_vtab。

+   **zSchema** → 定义虚拟表所在模式（"main"、"temp"等）的名称。

+   **zTabName** → 虚拟表的名称。

+   **mFlags** → 一个标志，指示这是一个“integrity_check”还是“quick_check”。目前，该参数始终为 0 或 1，尽管 SQLite 的未来版本可能会使用整数的其他位来指示额外的处理选项。

+   **pzErr** → 此参数指向一个初始化为 NULL 的"char*"。如果 xIntegrity()实现发现任何问题，应使*pzErr 指向从 sqlite3_malloc()或等效函数获取的错误字符串。

xIntegrity 方法通常应返回 SQLITE_OK，即使在虚拟表内容中发现问题。任何其他错误代码意味着 xIntegrity 方法在尝试评估虚拟表内容时遇到问题。例如，如果发现 FTS5 的倒排索引在内部不一致，则 xIntegrity 方法应将适当的错误消息写入 pzErr 参数并返回 SQLITE_OK。但如果 xIntegrity 方法由于内存耗尽而无法完成对虚拟表内容的评估，则应返回 SQLITE_NOMEM。

如果生成了错误消息，应该从 sqlite3_malloc64()或等效函数中获取空间来保存错误消息字符串。当 xIntegrity 返回时，错误消息字符串的所有权将传递给 SQLite 核心。核心将确保在完成对错误消息的处理后调用 sqlite3_free()以释放内存。调用 xIntegrity 方法的 PRAGMA integrity_check 命令不会改变返回的错误消息。xIntegrity 方法本身应包含虚拟表名称作为消息的一部分。zSchema 和 zName 参数的提供旨在简化此过程。

mFlags 参数当前是布尔值（0 或 1），指示 xIntegrity 方法是由于 PRAGMA integrity_check（mFlags==0）还是由于 PRAGMA quick_check（mFlags==1）而被调用。一般来说，xIntegrity 方法应该尽可能在线性时间内完成其有效性检查，但只有在 `(mFlags&1)==0` 时才执行需要超线性时间的检查。未来的 SQLite 版本可能会使用 mFlags 参数的高阶位来指示额外的处理选项。

在 SQLite 版本 3.44.0（2023-11-01）中增加了对 xIntegrity 方法的支持。在同一版本中，xIntegrity 方法被添加到许多内置虚拟表中，例如全文搜索 3（FTS3），全文搜索 5（FTS5）和 RTREE，因此当运行 PRAGMA integrity_check 时，这些表的内容将自动进行一致性检查。
