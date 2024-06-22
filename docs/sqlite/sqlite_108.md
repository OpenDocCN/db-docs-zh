# SQLite 版本 3 的 C/C++接口

> 原文：[`sqlite.com/capi3.html`](https://sqlite.com/capi3.html)

**注意：** 本文档于 2004 年编写，作为帮助程序员从 SQLite 版本 2 迁移到 SQLite 版本 3 的指南。本文档中的信息基本上仍然正确，但多年来已经有许多变化和增强。我们建议使用以下文档代替：

+   SQLite C/C++接口简介

+   SQLite C/C++参考指南

### 1.0 概述

SQLite 版本 3.0 是 SQLite 的新版本，源自 SQLite 2.8.13 代码库，但具有不兼容的文件格式和 API。SQLite 版本 3.0 是为了满足以下功能需求而创建的：

+   支持 UTF-16。

+   用户可定义的文本排序序列。

+   能够在索引列中存储 BLOBs。

由于每个功能需要对数据库文件格式进行不兼容的更改，因此必须转移到版本 3.0。其他不兼容的更改，如 API 的清理工作，也同时引入，因为最好一次性解决所有不兼容的更改。

版本 3.0 的 API 与 2.X 版本的 API 类似，但有一些重要的更改。最显著的是，所有 API 函数和数据结构开头的"`sqlite_`"前缀都改为"`sqlite3_`"。这样可以避免混淆两个 API，并允许同时链接 SQLite 2.X 和 SQLite 3.0。

对于 UTF-16 字符串，没有统一的 C 数据类型。因此，SQLite 使用 void*作为引用 UTF-16 字符串的通用类型。客户端软件可以将 void*转换为适合其系统的任何数据类型。

### 2.0 C/C++接口

SQLite 3.0 的 API 包含 83 个单独的函数，还有几个数据结构和#define。 (一个完整的 API 参考作为单独的文档提供。) 幸运的是，界面并不像其大小所暗示的那样复杂。 简单的程序仍然只需要 3 个函数：sqlite3_open()，sqlite3_exec()，和 sqlite3_close()。 更多对数据库引擎执行的控制是通过 sqlite3_prepare_v2()编译 SQLite 语句为字节码，以及 sqlite3_step()执行该字节码来提供的。 一组名字以 sqlite3_column_ 开头的例程用于提取查询结果集的信息。 许多接口函数是成对出现的，有 UTF-8 和 UTF-16 版本。 还有一组例程用于实现用户定义的 SQL 函数和用户定义的文本排序序列。

#### 2.1 打开和关闭数据库

> ```sql
>    typedef struct sqlite3 sqlite3;
>    int sqlite3_open(const char*, sqlite3**);
>    int sqlite3_open16(const void*, sqlite3**);
>    int sqlite3_close(sqlite3*);
>    const char *sqlite3_errmsg(sqlite3*);
>    const void *sqlite3_errmsg16(sqlite3*);
>    int sqlite3_errcode(sqlite3*);
> 
> ```

sqlite3_open()例程返回一个整数错误代码，而不是版本 2 接口所做的指向 sqlite3 结构的指针。 sqlite3_open()和 sqlite3_open16()的区别在于，sqlite3_open16()接受 UTF-16（主机本地字节顺序）作为数据库文件名的表示。 如果需要创建一个新的数据库文件，那么 sqlite3_open16()将内部文本表示设置为 UTF-16，而 sqlite3_open()将文本表示设置为 UTF-8。

打开和/或创建数据库文件被推迟直到文件真正需要。 这允许使用 PRAGMA 语句设置选项和参数，如本机文本表示和默认页面大小。

sqlite3_errcode() 例程返回最近一次重要 API 调用的结果代码。sqlite3_errmsg() 返回最近错误的英语文本错误消息。错误消息用 UTF-8 表示，并且是短暂的 - 在下一次调用任何 SQLite API 函数时可能会消失。sqlite3_errmsg16() 的工作方式类似于 sqlite3_errmsg()，只是它返回主机本地字节顺序中表示的 UTF-16 的错误消息。

SQLite 版本 3 的错误代码与版本 2 中保持不变。它们如下：

> ```sql
> #define SQLITE_OK           0   /* Successful result */
> #define SQLITE_ERROR        1   /* SQL error or missing database */
> #define SQLITE_INTERNAL     2   /* An internal logic error in SQLite */
> #define SQLITE_PERM         3   /* Access permission denied */
> #define SQLITE_ABORT        4   /* Callback routine requested an abort */
> #define SQLITE_BUSY         5   /* The database file is locked */
> #define SQLITE_LOCKED       6   /* A table in the database is locked */
> #define SQLITE_NOMEM        7   /* A malloc() failed */
> #define SQLITE_READONLY     8   /* Attempt to write a readonly database */
> #define SQLITE_INTERRUPT    9   /* Operation terminated by sqlite_interrupt() */
> #define SQLITE_IOERR       10   /* Some kind of disk I/O error occurred */
> #define SQLITE_CORRUPT     11   /* The database disk image is malformed */
> #define SQLITE_NOTFOUND    12   /* (Internal Only) Table or record not found */
> #define SQLITE_FULL        13   /* Insertion failed because database is full */
> #define SQLITE_CANTOPEN    14   /* Unable to open the database file */
> #define SQLITE_PROTOCOL    15   /* Database lock protocol error */
> #define SQLITE_EMPTY       16   /* (Internal Only) Database table is empty */
> #define SQLITE_SCHEMA      17   /* The database schema changed */
> #define SQLITE_TOOBIG      18   /* Too much data for one row of a table */
> #define SQLITE_CONSTRAINT  19   /* Abort due to contraint violation */
> #define SQLITE_MISMATCH    20   /* Data type mismatch */
> #define SQLITE_MISUSE      21   /* Library used incorrectly */
> #define SQLITE_NOLFS       22   /* Uses OS features not supported on host */
> #define SQLITE_AUTH        23   /* Authorization denied */
> #define SQLITE_ROW         100  /* sqlite_step() has another row ready */
> #define SQLITE_DONE        101  /* sqlite_step() has finished executing */
> 
> ```

#### 2.2 执行 SQL 语句

> ```sql
>    typedef int (*sqlite_callback)(void*,int,char**, char**);
>    int sqlite3_exec(sqlite3*, const char *sql, sqlite_callback, void*, char**);
> 
> ```

sqlite3_exec() 函数的工作方式与 SQLite 版本 2 中的类似。在第二个参数中指定的零个或多个 SQL 语句被编译并执行。查询结果返回到回调函数。

在 SQLite 版本 3 中，sqlite3_exec 例程只是对预编译语句接口的封装。

> ```sql
>    typedef struct sqlite3_stmt sqlite3_stmt;
>    int sqlite3_prepare(sqlite3*, const char*, int, sqlite3_stmt**, const char**);
>    int sqlite3_prepare16(sqlite3*, const void*, int, sqlite3_stmt**, const void**);
>    int sqlite3_finalize(sqlite3_stmt*);
>    int sqlite3_reset(sqlite3_stmt*);
> 
> ```

sqlite3_prepare 接口将单个 SQL 语句编译成字节码以供稍后执行。这个接口现在是访问数据库的首选方式。

SQL 语句是 sqlite3_prepare() 的 UTF-8 字符串。sqlite3_prepare16() 的工作方式相同，只是它期望 SQL 输入为 UTF-16 字符串。输入字符串中的第一个 SQL 语句被编译。如果有的话，第五个参数填入指向输入字符串中下一个（未编译）SQLite 语句的指针。sqlite3_finalize() 例程释放预编译的 SQL 语句。所有预编译语句在关闭数据库之前必须被最终化。sqlite3_reset() 例程重置预编译的 SQL 语句，使其可以再次执行。

SQL 语句可以包含形如 "?" 或 "?nnn" 或 ":aaa" 的标记，其中 "nnn" 是整数，"aaa" 是标识符。这些标记表示稍后将由 sqlite3_bind 接口填充的未指定的文字值（或"通配符"）。每个通配符都有一个相关联的数字，即语句中的顺序或 "?nnn" 形式中的 "nnn"。允许同一通配符在同一 SQL 语句中出现多次，此时所有该通配符的实例将填充相同的值。未绑定的通配符值为 NULL。

> ```sql
>    int sqlite3_bind_blob(sqlite3_stmt*, int, const void*, int n, void(*)(void*));
>    int sqlite3_bind_double(sqlite3_stmt*, int, double);
>    int sqlite3_bind_int(sqlite3_stmt*, int, int);
>    int sqlite3_bind_int64(sqlite3_stmt*, int, long long int);
>    int sqlite3_bind_null(sqlite3_stmt*, int);
>    int sqlite3_bind_text(sqlite3_stmt*, int, const char*, int n, void(*)(void*));
>    int sqlite3_bind_text16(sqlite3_stmt*, int, const void*, int n, void(*)(void*));
>    int sqlite3_bind_value(sqlite3_stmt*, int, const sqlite3_value*);
> 
> ```

在 sqlite3_bind 系列例程中，用于将数值分配给准备好的 SQL 语句中的通配符。未绑定的通配符会被解释为 NULL。绑定不会被 sqlite3_reset() 重置。但是，在 sqlite3_reset() 后，通配符可以重新绑定到新的数值。

SQL 语句准备好（可选地绑定）后，使用以下方式执行：

> ```sql
>    int sqlite3_step(sqlite3_stmt*);
> 
> ```

如果 sqlite3_step() 返回 SQLITE_ROW，则表示返回了结果集中的单行数据；如果返回 SQLITE_DONE，则表示执行完成，可能是正常完成或因错误而完成。如果无法打开数据库文件，则可能返回 SQLITE_BUSY。如果返回值是 SQLITE_ROW，则可以使用以下例程提取该行结果集的信息：

> ```sql
>    const void *sqlite3_column_blob(sqlite3_stmt*, int iCol);
>    int sqlite3_column_bytes(sqlite3_stmt*, int iCol);
>    int sqlite3_column_bytes16(sqlite3_stmt*, int iCol);
>    int sqlite3_column_count(sqlite3_stmt*);
>    const char *sqlite3_column_decltype(sqlite3_stmt *, int iCol);
>    const void *sqlite3_column_decltype16(sqlite3_stmt *, int iCol);
>    double sqlite3_column_double(sqlite3_stmt*, int iCol);
>    int sqlite3_column_int(sqlite3_stmt*, int iCol);
>    long long int sqlite3_column_int64(sqlite3_stmt*, int iCol);
>    const char *sqlite3_column_name(sqlite3_stmt*, int iCol);
>    const void *sqlite3_column_name16(sqlite3_stmt*, int iCol);
>    const unsigned char *sqlite3_column_text(sqlite3_stmt*, int iCol);
>    const void *sqlite3_column_text16(sqlite3_stmt*, int iCol);
>    int sqlite3_column_type(sqlite3_stmt*, int iCol);
> 
> ```

sqlite3_column_count() 函数返回结果集中的列数。sqlite3_column_count() 可以在 sqlite3_prepare_v2() 之后的任何时候调用。sqlite3_data_count() 与 sqlite3_column_count() 类似，但只能在 sqlite3_step() 之后调用。如果前一次对 sqlite3_step() 的调用返回 SQLITE_DONE 或错误代码，则 sqlite3_data_count() 返回 0，而 sqlite3_column_count() 将继续返回结果集中的列数。

返回的数据使用其他 sqlite3_column_***() 函数进行检查，所有这些函数的第二个参数都是列号。列从左到右以零为索引。注意，这与参数的索引不同，参数从 1 开始索引。

sqlite3_column_type() 函数返回第 N 列的值的数据类型。返回值是以下之一：

> ```sql
>    #define SQLITE_INTEGER  1
>    #define SQLITE_FLOAT    2
>    #define SQLITE_TEXT     3
>    #define SQLITE_BLOB     4
>    #define SQLITE_NULL     5
> 
> ```

sqlite3_column_decltype() 返回在 CREATE TABLE 语句中声明的列的类型。对于表达式，返回类型为空字符串。sqlite3_column_name() 返回第 N 列的名称。sqlite3_column_bytes() 返回具有 BLOB 类型的列中的字节数，或具有 UTF-8 编码的 TEXT 字符串中的字节数。sqlite3_column_bytes16() 对于 BLOB，返回相同的字节数，但对于 UTF-16 编码的 TEXT 字符串返回字节数。sqlite3_column_blob() 返回 BLOB 数据。sqlite3_column_text() 返回 UTF-8 编码的 TEXT 数据。sqlite3_column_text16() 返回 UTF-16 编码的 TEXT 数据。sqlite3_column_int() 返回主机机器的本机整数格式的 INTEGER 数据。sqlite3_column_int64() 返回 64 位 INTEGER 数据。最后，sqlite3_column_double() 返回浮点数数据。

不需要以 sqlite3_column_type() 指定的格式检索数据。如果请求了不同的格式，数据会自动转换。

数据格式转换可能会使先前调用 sqlite3_column_blob()、sqlite3_column_text() 和/或 sqlite3_column_text16() 返回的指针无效。以下情况可能会使指针无效：

+   初始内容是 BLOB，且调用 sqlite3_column_text() 或 sqlite3_column_text16()。字符串可能需要添加零终结符。

+   初始内容是 UTF-8 文本，并调用 sqlite3_column_bytes16() 或 sqlite3_column_text16()。内容必须转换为 UTF-16。

+   初始内容是 UTF-16 文本，并调用 sqlite3_column_bytes() 或 sqlite3_column_text()。内容必须转换为 UTF-8。

注意，UTF-16be 和 UTF-16le 之间的转换始终在原地完成，并不会使先前的指针无效，但当然，先前指针指向的缓冲区内容将被修改。在可能的情况下，其他类型的转换也在原地完成，但有时不可能，在这些情况下，先前的指针会无效。

最安全和最容易记住的策略是：假设任何来自

+   sqlite3_column_blob()，

+   sqlite3_column_text()，或

+   sqlite3_column_text16()

由后续调用无效。

+   sqlite3_column_bytes()，

+   sqlite3_column_bytes16()，

+   sqlite3_column_text()，或

+   sqlite3_column_text16()。

这意味着在调用 sqlite3_column_blob()、sqlite3_column_text() 或 sqlite3_column_text16() 之前，应始终调用 sqlite3_column_bytes() 或 sqlite3_column_bytes16()。

#### 2.3 用户自定义函数

可以使用以下例程创建用户定义的函数：

> ```sql
>    typedef struct sqlite3_value sqlite3_value;
>    int sqlite3_create_function(
>      sqlite3 *,
>      const char *zFunctionName,
>      int nArg,
>      int eTextRep,
>      void*,
>      void (*xFunc)(sqlite3_context*,int,sqlite3_value**),
>      void (*xStep)(sqlite3_context*,int,sqlite3_value**),
>      void (*xFinal)(sqlite3_context*)
>    );
>    int sqlite3_create_function16(
>      sqlite3*,
>      const void *zFunctionName,
>      int nArg,
>      int eTextRep,
>      void*,
>      void (*xFunc)(sqlite3_context*,int,sqlite3_value**),
>      void (*xStep)(sqlite3_context*,int,sqlite3_value**),
>      void (*xFinal)(sqlite3_context*)
>    );
>    #define SQLITE_UTF8     1
>    #define SQLITE_UTF16    2
>    #define SQLITE_UTF16BE  3
>    #define SQLITE_UTF16LE  4
>    #define SQLITE_ANY      5
> 
> ```

参数 nArg 指定函数的参数数量。值为 0 表示允许任意数量的参数。参数 eTextRep 指定此函数的参数所期望的文本值表示形式。此参数的值应为上述定义的参数之一。SQLite 版本 3 允许使用不同的文本表示实现同一函数的多个实现。数据库引擎选择最小化所需文本转换次数的函数。

普通函数只指定 xFunc，将 xStep 和 xFinal 设置为 NULL。聚合函数指定 xStep 和 xFinal，并将 xFunc 设置为 NULL。没有单独的 sqlite3_create_aggregate() API。

函数名以 UTF-8 指定。单独的 sqlite3_create_function16() API 的工作方式与 sqlite_create_function() 相同，只是函数名以 UTF-16 主机字节顺序指定。

请注意，函数的参数现在是指向 sqlite3_value 结构的指针，而不是指向字符串的指针，如 SQLite 版本 2.X 中所示。以下例程用于从这些“值”中提取有用信息：

> ```sql
>    const void *sqlite3_value_blob(sqlite3_value*);
>    int sqlite3_value_bytes(sqlite3_value*);
>    int sqlite3_value_bytes16(sqlite3_value*);
>    double sqlite3_value_double(sqlite3_value*);
>    int sqlite3_value_int(sqlite3_value*);
>    long long int sqlite3_value_int64(sqlite3_value*);
>    const unsigned char *sqlite3_value_text(sqlite3_value*);
>    const void *sqlite3_value_text16(sqlite3_value*);
>    int sqlite3_value_type(sqlite3_value*);
> 
> ```

函数实现使用以下 API 获取上下文并报告结果：

> ```sql
>    void *sqlite3_aggregate_context(sqlite3_context*, int nbyte);
>    void *sqlite3_user_data(sqlite3_context*);
>    void sqlite3_result_blob(sqlite3_context*, const void*, int n, void(*)(void*));
>    void sqlite3_result_double(sqlite3_context*, double);
>    void sqlite3_result_error(sqlite3_context*, const char*, int);
>    void sqlite3_result_error16(sqlite3_context*, const void*, int);
>    void sqlite3_result_int(sqlite3_context*, int);
>    void sqlite3_result_int64(sqlite3_context*, long long int);
>    void sqlite3_result_null(sqlite3_context*);
>    void sqlite3_result_text(sqlite3_context*, const char*, int n, void(*)(void*));
>    void sqlite3_result_text16(sqlite3_context*, const void*, int n, void(*)(void*));
>    void sqlite3_result_value(sqlite3_context*, sqlite3_value*);
>    void *sqlite3_get_auxdata(sqlite3_context*, int);
>    void sqlite3_set_auxdata(sqlite3_context*, int, void*, void (*)(void*));
> 
> ```

#### 2.4 用户定义排序序列

以下例程用于实现用户定义的排序序列：

> ```sql
>    sqlite3_create_collation(sqlite3*, const char *zName, int eTextRep, void*,
>       int(*xCompare)(void*,int,const void*,int,const void*));
>    sqlite3_create_collation16(sqlite3*, const void *zName, int eTextRep, void*,
>       int(*xCompare)(void*,int,const void*,int,const void*));
>    sqlite3_collation_needed(sqlite3*, void*, 
>       void(*)(void*,sqlite3*,int eTextRep,const char*));
>    sqlite3_collation_needed16(sqlite3*, void*,
>       void(*)(void*,sqlite3*,int eTextRep,const void*));
> 
> ```

函数 `sqlite3_create_collation()` 指定一个排序序列的名称和一个比较函数来实现该排序序列。比较函数仅用于比较文本值。参数 eTextRep 是 SQLITE_UTF8、SQLITE_UTF16LE、SQLITE_UTF16BE 或 SQLITE_ANY 中的一个，用于指定比较函数所使用的文本表示形式。对于 UTF-8、UTF-16LE 和 UTF-16BE 文本表示形式，可以为同一个排序序列存在不同的比较函数。函数 `sqlite3_create_collation16()` 类似于 `sqlite3_create_collation()`，不同之处在于排序名称使用 UTF-16 主机字节顺序而不是 UTF-8。

`sqlite3_collation_needed()` 例程注册一个回调函数，如果遇到未知的排序序列，数据库引擎将调用该回调。回调函数可以查找适当的比较函数，并根据需要调用 `sqlite3_create_collation()`。回调的第四个参数是排序序列的名称，以 UTF-8 编码。对于 `sqlite3_collation_need16()`，回调以 UTF-16 主机字节顺序发送排序序列名称。
