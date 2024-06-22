# 1\. 介绍

> 原文：[`sqlite.com/sessionintro.html`](https://sqlite.com/sessionintro.html)

会话扩展提供了一种机制，用于方便地记录 SQLite 数据库中某些或所有表的更改，并将这些更改打包成“变更集”或“补丁集”文件，以便稍后将相同的更改集应用到具有相同模式和兼容起始数据的另一个数据库中。一个“变更集”也可以被反转并用于“撤消”一个会话。

本文介绍了会话扩展。接口的详细信息在单独的 Session Extension C 语言接口 文档中。

## 1.1\. 典型使用案例

假设 SQLite 用作特定设计应用的 application file format。两个用户，Alice 和 Bob，每个人都从一个约一千兆字节大小的基线设计开始。他们全天并行工作，各自对设计进行自定义和微调。在一天结束时，他们希望将他们的变更合并为单一的统一设计。

会话扩展通过记录 Alice 和 Bob 数据库的所有更改，并将这些更改写入变更集或补丁集文件中，从而便于此过程。一天结束时，Alice 可以将她的变更集发送给 Bob，并且 Bob 可以“应用”到他的数据库中。结果（假设没有冲突）是 Bob 的数据库现在包含了他自己的变更和 Alice 的变更。同样地，Bob 可以将他的工作变更集发送给 Alice，她可以将他的变更应用到她的数据库中。

换句话说，会话扩展为 SQLite 数据库文件提供了一个类似于 unix [patch](https://en.wikipedia.org/wiki/Patch_(Unix)) 实用程序或版本控制系统（如 [Fossil](https://www.fossil-scm.org/)、[Git](https://git-scm.com) 或 [Mercurial](http://www.mercurial-scm.org/)）的“合并”功能的设施。

## 1.2\. 获取会话扩展

自 version 3.13.0（2016-05-18）起，会话扩展已包含在 SQLite amalgamation 源分发中。默认情况下，会话扩展是禁用的。要启用它，请使用以下编译器开关构建：

```sql
-DSQLITE_ENABLE_SESSION -DSQLITE_ENABLE_PREUPDATE_HOOK

```

或者，如果使用 autoconf 构建系统，请在配置脚本中传递 --enable-session 选项。

## 1.3\. 限制

+   在 SQLite 版本 3.17.0 之前，会话扩展仅适用于 rowid tables，而不适用于 WITHOUT ROWID 表。从 3.17.0 开始，rowid 和 WITHOUT ROWID 表都得到支持。然而，对于 WITHOUT ROWID 表的更改，需要额外的步骤来记录主键。

+   不支持 virtual tables。虚拟表的更改不会被捕获。

+   会话扩展仅适用于具有声明的主键的表。表的主键可以是 INTEGER PRIMARY KEY（rowid 别名）或外部主键。

+   SQLite 允许在主键列中存储 NULL 值。但是，会话扩展会忽略所有这些行。会话模块不记录影响具有一个或多个主键列中的 NULL 值的行的任何更改。

# 2\. 概念

## 2.1\. 更改集和补丁集

会话模块围绕创建和操作更改集展开。更改集是一段数据块，编码了对数据库的一系列更改。更改集中的每个更改都是以下之一：

+   一个**INSERT**。INSERT 更改包含要添加到数据库表中的单行。INSERT 更改的负载包含新行每个字段的值。

+   一个**DELETE**。DELETE 更改表示要从数据库表中删除的行，由其主键值标识。DELETE 更改的负载包含删除行的所有字段的值。

+   一个**UPDATE**。UPDATE 更改表示修改数据库表中单行的一个或多个非主键字段，由其主键字段标识。UPDATE 更改的负载包括：

    +   标识修改行的主键值，

    +   每个修改字段的新值，以及

    +   行的每个修改字段的原始值。

    UPDATE 更改不包含有关未由更改修改的非主键字段的任何信息。UPDATE 更改无法指定对主键字段的修改。

单个更改集可能包含适用于多个数据库表的更改。对于每个更改集至少包含一条变更的表，它还编码以下数据：

+   数据库表的名称，

+   表中的列数，以及

+   这些列中哪些是主键列。

更改集只能应用于包含与更改集中存储的上述三个条件匹配的表的数据库。

补丁集类似于更改集。它比更改集稍微紧凑，但提供了更有限的冲突检测和解决选项（详情请参见下一节）。补丁集与更改集的区别在于：

+   对于**DELETE**更改，负载仅包括主键字段。其他字段的原始值不作为补丁集的一部分存储。

+   对于**UPDATE**更改，负载仅包括主键字段和修改字段的新值。修改字段的原始值不作为补丁集的一部分存储。

## 2.2\. 冲突

当将更改集或补丁集应用于数据库时，会尝试为每个 INSERT 更改插入一行新行，为每个 DELETE 更改删除一行，并为每个 UPDATE 更改修改一行。如果目标数据库的内容与记录更改集时的原始数据库状态完全相同，这很简单。但是，如果目标数据库的内容不完全处于这种状态，当应用更改集或补丁集时可能会发生冲突。

处理**INSERT**变更时，可能会发生以下冲突：

+   目标数据库可能已包含与 INSERT 变更指定的相同主键值的行。

+   当新行插入时，可能会违反一些其他数据库约束，例如唯一约束或检查约束。

处理**DELETE**变更时，可能会检测到以下冲突：

+   目标数据库可能不包含指定主键值的行以进行删除。

+   目标数据库可能包含具有指定主键值的行，但其他字段可能包含与变更集内存储的值不匹配的值。使用补丁集时不会检测到此类冲突。

处理**UPDATE**变更时，可能会检测到以下冲突：

+   目标数据库可能不包含指定主键值的行以进行修改。

+   目标数据库可能包含具有指定主键值的行，但由变更集内存储的原始值与将被修改的字段的当前值不匹配。使用补丁集时不会检测到此类冲突。

+   当更新行时，可能会违反一些其他数据库约束，例如唯一约束或检查约束。

根据冲突类型，会话应用程序具有多种可配置选项用于处理冲突，从忽略冲突变更、中止整个变更集应用程序或尽管存在冲突但仍应用变更。详情请参考 sqlite3changeset_apply() API 的文档。

## 2.3\. 变更集构建

配置完成会话对象后，它开始监视其配置表格的更改。但每次修改数据库中的行时，它并不记录整个变更。相反，它仅记录每个插入行的主键字段，以及任何更新或删除行的主键和所有原始行值。如果单个会话多次修改一行，则不会记录新信息。

调用 sqlite3session_changeset()或 sqlite3session_patchset()时，从数据库文件中读取创建变更集或补丁集所需的其他信息。具体来说，

+   对于因 INSERT 操作而记录为主键的每个主键，会话模块会检查表中是否仍存在具有匹配主键的行。如果存在，则将添加 INSERT 变更到变更集中。

+   对于由于 UPDATE 或 DELETE 操作记录的每个主键，会话模块还检查表中是否存在具有匹配主键的行。如果可以找到一行，但一个或多个非主键字段与原始记录的值不匹配，则将 UPDATE 添加到变更集中。或者，如果根本没有具有指定主键的行，则向变更集添加 DELETE。如果行确实存在但未修改任何非主键字段，则不向变更集添加更改。

由上述可知，如果在单个会话内进行更改并撤消更改（例如，如果插入行然后再次删除），则会话模块不报告任何更改。或者如果在同一会话内多次更新行，则所有更新都合并为任何变更集或补丁集 blob 中的单个更新。

# 3\. 使用会话扩展

本节提供示例，演示如何使用会话扩展。

## 3.1\. 捕获变更集

下面的示例代码演示了在执行 SQL 命令时捕获变更集所涉及的步骤。总结如下：

1.  通过调用 sqlite3session_create() API 函数创建会话对象（类型为 sqlite3_session*）。

    单个会话对象通过单个 sqlite3* 数据库句柄监视对单个数据库（即“main”、“temp”或已附加的数据库）的更改。

1.  会话对象已配置为监视更改的表集。

    默认情况下，会话对象不监视任何数据库表的更改。在监视之前，必须对其进行配置。有三种方法可以配置要监视更改的表集：

    +   通过为每个表使用一次 sqlite3session_attach() 调用显式指定表，或者

    +   通过使用 NULL 参数调用 sqlite3session_attach() 指定监视数据库中所有表的更改。

    +   通过配置回调，在每次写入表时第一次调用时，指示会话模块是否应监视表上的更改。

    下面的示例代码使用上述方法中的第二种方法 - 它监视所有数据库表的更改。

1.  通过执行 SQL 语句对数据库进行更改。会话对象记录这些更改。

1.  使用调用 sqlite3session_changeset() 函数从会话对象中提取变更集 blob（或者，如果使用补丁集，则调用 sqlite3session_patchset() 函数）。

1.  使用调用 sqlite3session_delete() API 函数删除会话对象。

    从 session 对象中提取变更集或补丁集后，不必删除该会话对象。它可以保持连接到数据库句柄，并继续监视配置的表中的变更，就像之前一样。但是，如果在会话对象上第二次调用 sqlite3session_changeset()或 sqlite3session_patchset()，则变更集或补丁集将包含自会话创建以来连接上发生的*所有*变更。换句话说，会话对象不会因调用 sqlite3session_changeset()或 sqlite3session_patchset()而重置或归零。

```sql
*/*
** Argument zSql points to a buffer containing an SQL script to execute
** against the database handle passed as the first argument. As well as
** executing the SQL script, this function collects a changeset recording
** all changes made to the "main" database file. Assuming no error occurs,
** output variables (*ppChangeset) and (*pnChangeset) are set to point
** to a buffer containing the changeset and the size of the changeset in
** bytes before returning SQLITE_OK. In this case it is the responsibility
** of the caller to eventually free the changeset blob by passing it to
** the sqlite3_free function.
**
** Or, if an error does occur, return an SQLite error code. The final
** value of (*pChangeset) and (*pnChangeset) are undefined in this case.
*/*
int sql_exec_changeset(
  sqlite3 *db,                  */* Database handle */*
  const char *zSql,             */* SQL script to execute */*
  int *pnChangeset,             */* OUT: Size of changeset blob in bytes */*
  void **ppChangeset            */* OUT: Pointer to changeset blob */*
){
  sqlite3_session *pSession = 0;
  int rc;

  */* Create a new session object */*
  rc = sqlite3session_create(db, "main", &pSession);

  */* Configure the session object to record changes to all tables */*
  if( rc==SQLITE_OK ) rc = sqlite3session_attach(pSession, NULL);

  */* Execute the SQL script */*
  if( rc==SQLITE_OK ) rc = sqlite3_exec(db, zSql, 0, 0, 0);

  */* Collect the changeset */*
  if( rc==SQLITE_OK ){
    rc = sqlite3session_changeset(pSession, pnChangeset, ppChangeset);
  }

  */* Delete the session object */*
  sqlite3session_delete(pSession);

  return rc;
}

```

## 3.2\. 将变更集应用到数据库

将变更集应用到数据库比捕获变更集更简单。通常，如下面的示例代码所示，只需调用一次 sqlite3changeset_apply()即可。

在应用变更集时复杂的情况通常出现在冲突解决中。有关详细信息，请参阅上面链接的 API 文档。

```sql
*/*
** Conflict handler callback used by apply_changeset(). See below.
*/*
static int xConflict(void *pCtx, int eConflict, sqlite3_changeset_iter *pIter){
  int ret = (int)pCtx;
  return ret;
}

*/*
** Apply the changeset contained in blob pChangeset, size nChangeset bytes,
** to the main database of the database handle passed as the first argument.
** Return SQLITE_OK if successful, or an SQLite error code if an error
** occurs.
**
** If parameter bIgnoreConflicts is true, then any conflicting changes
** within the changeset are simply ignored. Or, if bIgnoreConflicts is
** false, then this call fails with an SQLITE_ABORT error if a changeset
** conflict is encountered.
*/*
int apply_changeset(
  sqlite3 *db,                  */* Database handle */*
  int bIgnoreConflicts,         */* True to ignore conflicting changes */*
  int nChangeset,               */* Size of changeset in bytes */*
  void *pChangeset              */* Pointer to changeset blob */*
){
  return sqlite3changeset_apply(
      db,
      nChangeset, pChangeset,
      0, xConflict,
      (void*)bIgnoreConflicts
  );
}

```

## 3.3\. 检查变更集的内容

下面的示例代码演示了用于迭代并提取变更集中所有变更数据的技术。总结如下：

1.  调用 sqlite3changeset_start() API 来创建和初始化一个迭代器，以便迭代变更集的内容。最初，迭代器不指向任何元素。

1.  第一次调用 sqlite3changeset_next()会将迭代器移动到变更集中第一个变更处（如果变更集完全为空，则移动到 EOF）。sqlite3changeset_next()在将迭代器移动到有效条目时返回 SQLITE_ROW，在将迭代器移动到 EOF 时返回 SQLITE_DONE，在发生错误时返回 SQLite 错误代码。

1.  如果迭代器指向有效条目，则可以使用 sqlite3changeset_op() API 来确定迭代器指向的变更类型（INSERT、UPDATE 或 DELETE）。此外，相同的 API 还可用于获取应用变更的表的名称、预期列数和主键列数。

1.  如果迭代器指向有效的 INSERT 或 UPDATE 条目，则可以使用 sqlite3changeset_new() API 来获取变更负载中的 new.*值。

1.  如果迭代器指向有效的 DELETE 或 UPDATE 条目，则可以使用 sqlite3changeset_old() API 来获取变更负载中的 old.*值。

1.  使用调用 sqlite3changeset_finalize() API 来删除迭代器。如果在迭代过程中发生错误，则返回 SQLite 错误代码（即使 sqlite3changeset_next()已经返回相同的错误代码）。如果没有发生错误，则返回 SQLITE_OK。

```sql
*/*
** Print the contents of the changeset to stdout.
*/*
static int print_changeset(void *pChangeset, int nChangeset){
  int rc;
  sqlite3_changeset_iter *pIter = 0;

  */* Create an iterator to iterate through the changeset */*
  rc = sqlite3changeset_start(&pIter, nChangeset, pChangeset);
  if( rc!=SQLITE_OK ) return rc;

  */* This loop runs once for each change in the changeset */*
  while( SQLITE_ROW==sqlite3changeset_next(pIter) ){
    const char *zTab;           */* Table change applies to */*
    int nCol;                   */* Number of columns in table zTab */*
    int op;                     */* SQLITE_INSERT, UPDATE or DELETE */*
    sqlite3_value *pVal;

    */* Print the type of operation and the table it is on */*
    rc = sqlite3changeset_op(pIter, &zTab, &nCol, &op, 0);
    if( rc!=SQLITE_OK ) goto exit_print_changeset;
    printf("%s on table %s\n",
      op==SQLITE_INSERT?"INSERT" : op==SQLITE_UPDATE?"UPDATE" : "DELETE",
      zTab
    );

    */* If this is an UPDATE or DELETE, print the old.* values */*
    if( op==SQLITE_UPDATE || op==SQLITE_DELETE ){
      printf("Old values:");
      for(i=0; i<nCol; i++){
        rc = sqlite3changeset_old(pIter, i, &pVal);
        if( rc!=SQLITE_OK ) goto exit_print_changeset;
        printf(" %s", pVal ? sqlite3_value_text(pVal) : "-");
      }
      printf("\n");
    }

    */* If this is an UPDATE or INSERT, print the new.* values */*
    if( op==SQLITE_UPDATE || op==SQLITE_INSERT ){
      printf("New values:");
      for(i=0; i<nCol; i++){
        rc = sqlite3changeset_new(pIter, i, &pVal);
        if( rc!=SQLITE_OK ) goto exit_print_changeset;
        printf(" %s", pVal ? sqlite3_value_text(pVal) : "-");
      }
      printf("\n");
    }
  }

  */* Clean up the changeset and return an error code (or SQLITE_OK) */*
 exit_print_changeset:
  rc2 = sqlite3changeset_finalize(pIter);
  if( rc==SQLITE_OK ) rc = rc2;
  return rc;
}

```

# 4\. 扩展功能

大多数应用程序仅使用前一部分描述的会话模块功能。但是，以下额外功能可供使用和操作变更集和补丁集二进制数据：

+   可以使用 sqlite3changeset_concat()或 sqlite3_changegroup 接口合并两个或多个变更集/补丁集。

+   可以使用 sqlite3changeset_invert() API 函数“反转”一个变更集。反转后的变更集会撤销原始变更。如果变更集 C^+是变更集 C 的逆变更集，则将变更集 C 应用于数据库，然后再应用 C^+，数据库应该保持不变。
