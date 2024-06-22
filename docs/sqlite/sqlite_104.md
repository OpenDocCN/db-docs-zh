# SQLite Version 2 的 C 语言接口

> 原文：[`sqlite.com/c_interface.html`](https://sqlite.com/c_interface.html)

| **编者按：** 本文档描述的是已于 2004 年被弃用并由 SQLite3 取代的 SQLite 版本 2。这份文档作为 SQLite 历史记录的一部分被保留下来。现代程序员应参考本网站其他地方的更为更新的 SQLite 文档。 |
| --- |

SQLite 库设计得非常易于从 C 或 C++程序中使用。本文档概述了 C/C++编程接口。

### 1.0 核心 API

SQLite 库的接口由三个核心函数、一个不透明数据结构和一些用作返回值的常量组成。核心接口如下：

> ```sql
> typedef struct sqlite sqlite;
> #define SQLITE_OK           0   /* Successful result */
> 
> sqlite *sqlite_open(const char *dbname, int mode, char **errmsg);
> 
> void sqlite_close(sqlite *db);
> 
> int sqlite_exec(
>   sqlite *db,
>   char *sql,
>   int (*xCallback)(void*,int,char**,char**),
>   void *pArg,
>   char **errmsg
> );
> 
> ```

以上内容是您在 C 或 C++程序中使用 SQLite 所需了解的全部。还有其他可用的接口函数（如下所述），但我们将从上面描述的核心函数开始。

#### 1.1 打开数据库

使用**sqlite_open**函数打开一个现有的 SQLite 数据库或创建一个新的 SQLite 数据库。第一个参数是数据库名称。第二个参数意图是表明数据库是用于读写还是仅用于读取。但在当前实现中，**sqlite_open**的第二个参数被忽略。第三个参数是一个指向字符串指针的指针。如果第三个参数不为 NULL 并且在尝试打开数据库时发生错误，则会将错误消息写入从 malloc()获取的内存中，并且*errmsg 将指向这条错误消息。调用函数在完成后负责释放内存。

SQLite 数据库的名称是将包含数据库的文件的名称。如果文件不存在，SQLite 将尝试创建并初始化它。如果文件是只读的（由于权限位或因为它位于只读介质如 CD-ROM 上），那么 SQLite 仅打开数据库以供读取。整个 SQL 数据库存储在磁盘上的单个文件中。但在执行 SQL 命令期间可能会创建额外的临时文件，用于存储数据库回滚日志或查询的临时和中间结果。

**sqlite_open** 函数的返回值是一个指向不透明**sqlite**结构的指针。该指针将成为处理相同数据库的所有后续 SQLite 函数调用的第一个参数。如果由于任何原因打开失败，则返回 NULL。

#### 1.2 关闭数据库

要关闭 SQLite 数据库，请调用**sqlite_close**函数，并将先前从**sqlite_open**调用中获得的 sqlite 结构指针传递给它。如果数据库关闭时有事务处于活动状态，则事务将被回滚。

#### 1.3 执行 SQL 语句

**sqlite_exec** 函数用于处理 SQL 语句和查询。该函数需要以下 5 个参数：

1.  一个指向先前从**sqlite_open**调用中获取的 sqlite 结构的指针。

1.  包含一个或多个 SQL 语句和/或查询文本的以零结尾的字符串。

1.  一个指向回调函数的指针，该函数在查询结果的每一行中调用一次。此参数可以为 NULL，如果为 NULL，则永远不会调用回调函数。

1.  一个指针，将作为回调函数的第一个参数传递。

1.  指向错误字符串的指针。错误消息被写入从 malloc() 获得的空间，并且错误字符串指向了 malloc 分配的空间。调用函数在使用完后负责释放此空间。此参数可能为空，在这种情况下，错误消息不会被报告回调用函数。

回调函数用于接收查询结果。回调函数的原型如下所示：

> ```sql
> int Callback(void *pArg, int argc, char **argv, char **columnNames){
>   return 0;
> }
> 
> ```

回调的第一个参数只是 **sqlite_exec** 的第四个参数的一个副本。此参数可用于通过客户端代码将任意信息传递给回调函数。第二个参数是查询结果中的列数。第三个参数是一个指向字符串指针数组的指针，其中每个字符串是该记录结果的单个列。请注意，回调函数将数据库中的 NULL 值报告为 NULL 指针，这与空字符串非常不同。如果第 i 个参数是空字符串，我们会得到：

> ```sql
> argv[i][0] == 0
> 
> ```

但是如果第 i 个参数为空，我们会得到：

> ```sql
> argv[i] == 0
> 
> ```

列的名称包含在第 *argc* 个参数的第四个参数的前几个条目中。如果 SHOW_DATATYPES pragma 打开（默认关闭），则第四个参数的第二 *argc* 个条目是相应列的数据类型。

如果 EMPTY_RESULT_CALLBACKS pragma 设置为 ON 并且查询的结果是空集，则回调将使用第三个参数 (argv) 设置为 0。换句话说，

> ```sql
> argv == 0
> 
> ```

第二个参数 (argc) 和第四个参数 (columnNames) 仍然有效，并且可以用于确定结果列的数量和名称（如果有结果）。默认行为是如果结果集为空，则根本不调用回调函数。

回调函数通常应返回 0。如果回调函数返回非零值，则立即中止查询，**sqlite_exec** 将返回 SQLITE_ABORT。

#### 1.4 错误代码

**sqlite_exec** 函数通常返回 SQLITE_OK。但如果出现问题，它可以返回不同的值以指示错误类型。以下是返回代码的完整列表：

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
> #define SQLITE_CONSTRAINT  19   /* Abort due to constraint violation */
> #define SQLITE_MISMATCH    20   /* Data type mismatch */
> #define SQLITE_MISUSE      21   /* Library used incorrectly */
> #define SQLITE_NOLFS       22   /* Uses OS features not supported on host */
> #define SQLITE_AUTH        23   /* Authorization denied */
> #define SQLITE_ROW         100  /* sqlite_step() has another row ready */
> #define SQLITE_DONE        101  /* sqlite_step() has finished executing */
> 
> ```

这些不同返回值的含义如下：

> SQLITE_OK
> 
> 如果一切正常且没有错误，则返回此值。
> 
> SQLITE_INTERNAL
> 
> 此值表示 SQLite 库内部一致性检查失败。这只会在 SQLite 库中存在错误时发生。如果您从 **sqlite_exec** 调用中获得 SQLITE_INTERNAL 回复，请在 SQLite 邮件列表上报告此问题。
> 
> SQLITE_ERROR
> 
> 此返回值表明传递给 **sqlite_exec** 的 SQL 中存在错误。
> 
> SQLITE_PERM
> 
> 此返回值表示数据库文件的访问权限使得无法打开该文件。
> 
> SQLITE_ABORT
> 
> 如果回调函数返回非零，则返回此值。
> 
> SQLITE_BUSY
> 
> 此返回代码表示另一个程序或线程已锁定数据库。SQLite 允许两个或更多线程同时读取数据库，但同一时间只能有一个线程可以写入数据库。在 SQLite 中，锁定是对整个数据库的。
> 
> SQLITE_LOCKED
> 
> 此返回代码与 SQLITE_BUSY 类似，表明数据库已锁定。但锁定的来源是对 **sqlite_exec** 的递归调用。此返回仅在您尝试从先前调用 **sqlite_exec** 的查询的回调函数中调用 sqlite_exec 时才会发生。允许递归调用 **sqlite_exec**，只要它们不尝试写入相同的表。
> 
> SQLITE_NOMEM
> 
> 如果调用 **malloc** 失败，则返回此值。
> 
> SQLITE_READONLY
> 
> 此返回代码指示尝试向仅供读取的数据库文件写入时返回。
> 
> SQLITE_INTERRUPT
> 
> 如果调用**sqlite_interrupt**中断正在进行的数据库操作，将返回此值。
> 
> SQLITE_IOERR
> 
> 如果操作系统通知 SQLite 无法执行某些磁盘 I/O 操作，将返回此值。这可能意味着磁盘上没有剩余空间了。
> 
> SQLITE_CORRUPT
> 
> 如果 SQLite 检测到它正在处理的数据库已损坏，则返回此值。损坏可能是由于恶意进程向数据库文件写入，也可能是由于 SQLite 先前未检测到的逻辑错误引起的。如果发生这种情况，SQLite 被迫以损坏的状态离开数据库文件时，也将返回此值。后者只应该由硬件或操作系统故障引起。
> 
> SQLITE_FULL
> 
> 如果插入失败，因为磁盘上没有剩余空间，或者数据库太大而无法容纳更多信息，则返回此值。后一种情况应仅发生在大于 2GB 大小的数据库中。
> 
> SQLITE_CANTOPEN
> 
> 如果由于某些原因无法打开数据库文件，将返回此值。
> 
> SQLITE_PROTOCOL
> 
> 如果其他进程正在干扰文件锁，并且违反了 SQLite 在其回滚日志文件上使用的文件锁定协议，将返回此值。
> 
> SQLITE_SCHEMA
> 
> 当数据库首次打开时，SQLite 将数据库架构读入内存，并使用该架构解析新的 SQL 语句。如果另一个进程更改了架构，则当前正在处理的命令将中止，因为生成的虚拟机代码假定旧的架构。这是这种情况的返回代码。通常重新尝试命令可以解决问题。
> 
> SQLITE_TOOBIG
> 
> SQLite 不会在单个表的单行中存储超过约 1 兆字节的数据。如果尝试在单行中存储超过 1 兆字节的数据，这将是您收到的返回代码。
> 
> SQLITE_CONSTRAINT
> 
> 如果 SQL 语句违反了数据库约束，则返回此常量。
> 
> SQLITE_MISMATCH
> 
> 当试图将非整数数据插入标记为 INTEGER PRIMARY KEY 的列时，会发生此错误。对于大多数列，SQLite 会忽略数据类型并允许存储任何类型的数据。但是 INTEGER PRIMARY KEY 列只允许存储整数数据。
> 
> SQLITE_MISUSE
> 
> 如果有一个或多个 SQLite API 例程使用不正确，可能会发生此错误。不正确的用法包括在使用**sqlite_close**关闭数据库后调用**sqlite_exec**，或者同时从两个独立的线程中的相同数据库指针调用**sqlite_exec**。
> 
> SQLITE_NOLFS
> 
> 这个错误表示您尝试在不支持大文件的旧版 Unix 机器上创建或访问大于 2GB 的文件数据库文件。
> 
> SQLITE_AUTH
> 
> 这个错误表明授权回调已经禁止了您尝试执行的 SQL。
> 
> SQLITE_ROW
> 
> 这是**sqlite_step**例程的返回代码之一，它是非回调 API 的一部分。它表示另一行结果数据可用。
> 
> SQLITE_DONE
> 
> 这是**sqlite_step**例程的返回代码之一，它是非回调 API 的一部分。它表示 SQL 语句已完全执行，并且**sqlite_finalize**例程已准备就绪可以调用。

### 2.0 Accessing Data Without Using A Callback Function

上面描述的**sqlite_exec**例程曾经是从 SQLite 数据库中检索数据的唯一方式。但是许多程序员发现使用回调函数获取结果很不方便。因此，从 SQLite 版本 2.7.7 开始，提供了第二个不使用回调函数的访问接口。

新接口使用三个独立的函数来替换单一的**sqlite_exec**函数。

> ```sql
> typedef struct sqlite_vm sqlite_vm;
> 
> int sqlite_compile(
>   sqlite *db,              /* The open database */
>   const char *zSql,        /* SQL statement to be compiled */
>   const char **pzTail,     /* OUT: uncompiled tail of zSql */
>   sqlite_vm **ppVm,        /* OUT: the virtual machine to execute zSql */
>   char **pzErrmsg          /* OUT: Error message. */
> );
> 
> int sqlite_step(
>   sqlite_vm *pVm,          /* The virtual machine to execute */
>   int *pN,                 /* OUT: Number of columns in result */
>   const char ***pazValue,  /* OUT: Column data */
>   const char ***pazColName /* OUT: Column names and datatypes */
> );
> 
> int sqlite_finalize(
>   sqlite_vm *pVm,          /* The virtual machine to be finalized */
>   char **pzErrMsg          /* OUT: Error message */
> );
> 
> ```

策略是使用**sqlite_compile**编译一条单独的 SQL 语句，然后针对每行输出多次调用**sqlite_step**，最后调用**sqlite_finalize**来清理 SQL 执行完成后的状态。

#### 2.1 编译 SQL 语句成虚拟机

**sqlite_compile** "编译"一条单独的 SQL 语句（由第二个参数指定），并生成一个能执行该语句的虚拟机。与大多数接口例程一样，第一个参数必须是来自对**sqlite_open**的先前调用的 sqlite 结构的指针。

虚拟机的指针存储在作为第四个参数传入的指针中。用于存储虚拟机的空间是动态分配的。为避免内存泄漏，调用函数在使用完虚拟机后必须调用**sqlite_finalize**。如果在编译过程中遇到错误，第四个参数可能会设置为 NULL。

如果在编译过程中遇到任何错误，错误消息将被写入从**malloc**获取的内存中，第五个参数将指向该内存。如果第五个参数为 NULL，则不会生成错误消息。如果第五个参数不为 NULL，则调用函数应通过调用**sqlite_freemem**来释放包含错误消息的内存。

如果第二个参数实际上包含两个或更多条 SQL 语句，那么只有第一条语句被编译。（这与**sqlite_exec**的行为不同，后者会执行其输入字符串中的所有 SQL 语句。）**sqlite_compile**的第三个参数指向输入中第一条 SQL 语句结束后的第一个字符。如果第二个参数只包含一条 SQL 语句，那么第三个参数将指向第二个参数结尾处的'\000'终结符。

如果成功，**sqlite_compile** 返回 SQLITE_OK。否则，返回错误代码。

#### 2.2 SQL 语句逐步执行

使用 **sqlite_compile** 生成虚拟机后，通过一个或多个调用 **sqlite_step** 来执行它。每次调用 **sqlite_step**（除了最后一次）返回一个结果的单行。结果中的列数存储在第二个参数指向的整数中。第三个参数指定的指针被设置为指向一个指向列值的指针数组。第四个参数指针被设置为指向一个指向列名和数据类型的指针数组。**sqlite_step** 的第二至第四个参数传递的信息与使用 **sqlite_exec** 接口时 **callback** 例程的第二至第四个参数相同。不同的是，使用 **sqlite_step** 时，无论是否打开了 SHOW_DATATYPES pragma，列数据类型信息始终包含在第四个参数中。

每次调用 **sqlite_step** 返回一个整数代码，指示该步骤期间发生的情况。此代码可能是 SQLITE_BUSY、SQLITE_ROW、SQLITE_DONE、SQLITE_ERROR 或 SQLITE_MISUSE。

如果虚拟机无法打开数据库文件，因为它被另一个线程或进程锁定，**sqlite_step** 将返回 SQLITE_BUSY。调用函数应该做一些其他活动，或者短暂地休眠，以便让锁有机会清除，然后再次调用 **sqlite_step**。这可以重复多次。

每当有另一行结果数据可用时，**sqlite_step** 将返回 SQLITE_ROW。行数据存储在一个指向字符串的指针数组中，第二个参数指向这个数组。

当所有处理完成时，**sqlite_step**将返回 SQLITE_DONE 或 SQLITE_ERROR。SQLITE_DONE 表示语句成功完成，SQLITE_ERROR 表示发生运行时错误。（错误的详细信息来自**sqlite_finalize**。）试图在**sqlite_step**返回 SQLITE_DONE 或 SQLITE_ERROR 后再次调用它是对库的误用。

当**sqlite_step**返回 SQLITE_DONE 或 SQLITE_ERROR 时，*pN 和*pazColName 的值将设置为结果集中的列数和列的名称，就像对于 SQLITE_ROW 返回一样。这允许调用代码在结果集为空的情况下找到结果列数和列名及数据类型。当返回代码为 SQLITE_DONE 或 SQLITE_ERROR 时，*pazValue 参数始终设置为 NULL。如果正在执行的 SQL 是不返回结果的语句（如 INSERT 或 UPDATE），那么*pN 将设置为零，*pazColName 将设置为 NULL。

如果滥用库试图不当调用**sqlite_step**，它将尝试返回 SQLITE_MISUSE。如果您在两个或多个线程中同时调用同一个虚拟机上的 sqlite_step()，或者在它返回 SQLITE_DONE 或 SQLITE_ERROR 后再次调用 sqlite_step()，或者向 sqlite_step()传递无效的虚拟机指针，则可能发生这种情况。您不应依赖 SQLITE_MISUSE 返回代码来指示错误。界面的误用可能不会被检测到，并导致程序崩溃。SQLITE_MISUSE 仅用作调试辅助工具 - 以帮助您在事故发生之前检测到不正确的使用。误用检测逻辑不能保证在每种情况下都能正常工作。

#### 2.3 删除虚拟机

每个**sqlite_compile**创建的虚拟机最终都应该交给**sqlite_finalize**。sqlite_finalize()过程释放虚拟机使用的内存和其他资源。如果不调用 sqlite_finalize()，程序中会出现资源泄漏。

**sqlite_finalize**例程还返回指示虚拟机执行的 SQL 操作成功或失败的结果代码。sqlite_finalize()返回的值将与通过**sqlite_exec**执行相同 SQL 时返回的值相同。返回的错误消息也将相同。

在**sqlite_step**返回 SQLITE_DONE 之前对虚拟机调用**sqlite_finalize**也是可以接受的。这样做会中断正在进行的操作。部分完成的更改将被回滚，并且数据库将恢复到其原始状态（除非使用 SQL 中的 ON CONFLICT 子句选择了替代恢复算法）。其效果等同于**sqlite_exec**的回调函数返回非零值的情况。

在一个从未传递给**sqlite_step**的虚拟机上调用**sqlite_finalize**也是可以接受的。

### 3.0 扩展 API

只有 1.0 节中描述的三个核心例程是使用 SQLite 所必需的。但是还有许多其他提供有用接口的函数。这些扩展例程如下：

> ```sql
> int sqlite_last_insert_rowid(sqlite*);
> 
> int sqlite_changes(sqlite*);
> 
> int sqlite_get_table(
>   sqlite*,
>   char *sql,
>   char ***result,
>   int *nrow,
>   int *ncolumn,
>   char **errmsg
> );
> 
> void sqlite_free_table(char**);
> 
> void sqlite_interrupt(sqlite*);
> 
> int sqlite_complete(const char *sql);
> 
> void sqlite_busy_handler(sqlite*, int (*)(void*,const char*,int), void*);
> 
> void sqlite_busy_timeout(sqlite*, int ms);
> 
> const char sqlite_version[];
> 
> const char sqlite_encoding[];
> 
> int sqlite_exec_printf(
>   sqlite*,
>   char *sql,
>   int (*)(void*,int,char**,char**),
>   void*,
>   char **errmsg,
>   ...
> );
> 
> int sqlite_exec_vprintf(
>   sqlite*,
>   char *sql,
>   int (*)(void*,int,char**,char**),
>   void*,
>   char **errmsg,
>   va_list
> );
> 
> int sqlite_get_table_printf(
>   sqlite*,
>   char *sql,
>   char ***result,
>   int *nrow,
>   int *ncolumn,
>   char **errmsg,
>   ...
> );
> 
> int sqlite_get_table_vprintf(
>   sqlite*,
>   char *sql,
>   char ***result,
>   int *nrow,
>   int *ncolumn,
>   char **errmsg,
>   va_list
> );
> 
> char *sqlite_mprintf(const char *zFormat, ...);
> 
> char *sqlite_vmprintf(const char *zFormat, va_list);
> 
> void sqlite_freemem(char*);
> 
> void sqlite_progress_handler(sqlite*, int, int (*)(void*), void*);
> 
> ```

所有上述定义都包含在源代码树中的"sqlite.h"头文件中。

#### 3.1 最近插入的 ROWID

每个 SQLite 表的每一行都有一个唯一的整数键。如果表有一个标记为 INTEGER PRIMARY KEY 的列，那么该列将作为键。如果没有 INTEGER PRIMARY KEY 列，则键是一个唯一的整数。可以在 SELECT 语句中访问行的键，或者在 WHERE 或 ORDER BY 子句中使用以下任何一个名称："ROWID"、"OID"或"_ROWID_"。

当向没有 INTEGER PRIMARY KEY 列的表插入数据时，或者如果表有 INTEGER PRIMARY KEY 列但在 INSERT 语句的 VALUES 子句中没有指定该列的值，则会自动生成该键值。可以使用**sqlite_last_insert_rowid** API 函数找到最近一次 INSERT 语句的键值。

#### 3.2 修改的行数

**sqlite_changes** API 函数返回自数据库上次静止以来插入、删除或修改的行数。"静止"数据库是指没有未完成的**sqlite_exec**调用，也没有由**sqlite_compile**创建但未由**sqlite_finalize**最终化的 VM。通常情况下，**sqlite_changes**返回最近一次**sqlite_exec**调用或自最近一次**sqlite_compile**以来插入、删除或修改的行数。但如果有嵌套调用**sqlite_exec**（即一个**sqlite_exec**的回调例程调用另一个**sqlite_exec**），或者在仍存在另一个 VM 的情况下调用**sqlite_compile**创建新 VM，则**sqlite_changes**返回的数字的含义更为复杂。报告的数字包括后来由 ROLLBACK 或 ABORT 撤消的任何更改。但因为 DROP TABLE 而删除的行数*不*计入其中。

SQLite 通过删除表然后重新创建来实现不带 WHERE 子句的命令"**DELETE FROM table**"。这比逐个删除表中元素要快得多。但这也意味着，无论原表中有多少元素，从**sqlite_changes**返回的值都将是零。如果需要准确计算删除的元素数量，请改用"**DELETE FROM table WHERE 1**"。

#### 3.3 从 malloc()获取的内存中进行查询

**sqlite_get_table**函数是**sqlite_exec**的一个包装器，它收集连续回调中的所有信息，并将其写入从 malloc()获得的内存中。这是一个便利函数，允许应用程序通过单个函数调用获取数据库查询的整个结果。

**sqlite_get_table**的主要结果是一个指向字符串指针的数组。这个数组中的每个元素对应于结果中每行每列的一个元素。NULL 结果由 NULL 指针表示。除了常规数据外，数组的开头还有一个额外的行，其中包含结果每列的名称。

例如，考虑以下查询：

> SELECT employee_name, login, host FROM users WHERE login LIKE 'd%';

此查询将返回以字母"d"开头的每位员工的姓名、登录名和主机名。如果将此查询提交给**sqlite_get_table**，结果可能如下所示：

> nrow = 2
> 
> ncolumn = 3
> 
> result[0] = "employee_name"
> 
> result[1] = "login"
> 
> result[2] = "host"
> 
> result[3] = "dummy"
> 
> result[4] = "No such user"
> 
> result[5] = 0
> 
> result[6] = "D. Richard Hipp"
> 
> result[7] = "drh"
> 
> result[8] = "zadok"

注意，“dummy”记录的“host”值为 NULL，因此 result[]数组在该位置包含一个 NULL 指针。

如果查询的结果集为空，默认情况下**sqlite_get_table**会将 nrow 设置为 0，并将其结果参数设置为 NULL。但是如果打开了 EMPTY_RESULT_CALLBACKS pragma，则结果参数初始化为仅列名。例如，考虑以下结果集为空的查询：

> SELECT employee_name, login, host FROM users WHERE employee_name IS NULL;

默认行为提供如下结果：

> nrow = 0
> 
> ncolumn = 0
> 
> result = 0

但是如果打开了 EMPTY_RESULT_CALLBACKS pragma，则返回如下结果：

> nrow = 0
> 
> ncolumn = 3
> 
> result[0] = "employee_name"
> 
> result[1] = "login"
> 
> result[2] = "host"

用于存储**sqlite_get_table**返回信息的内存是从 malloc()获取的。但调用函数不应直接尝试释放此信息。而是在不再需要表时将完整表传递给**sqlite_free_table**。可以安全地使用空指针调用**sqlite_free_table**，例如，如果结果集为空，则会返回空指针。

**sqlite_get_table**例程返回与**sqlite_exec**相同的整数结果代码。

#### 3.4 中断 SQLite 操作

**sqlite_interrupt**函数可以从不同的线程或信号处理程序中调用，以使当前数据库操作在第一次机会时退出。当发生这种情况时，启动数据库操作的**sqlite_exec**例程（或等效例程）将返回 SQLITE_INTERRUPT。

#### 3.5 测试完整的 SQL 语句

SQLite 的下一个接口例程是一个方便函数，用于测试一个字符串是否构成完整的 SQL 语句。如果**sqlite_complete**函数在其输入为字符串时返回 true，则该参数形成完整的 SQL 语句。不能保证该语句的语法是否正确，但至少我们知道该语句是完整的。如果**sqlite_complete**返回 false，则需要更多文本来完成 SQL 语句。

对于**sqlite_complete**函数的目的，如果 SQL 语句以分号结束，则该 SQL 语句是完整的。

**sqlite**命令行实用程序使用**sqlite_complete**函数来确定何时需要调用**sqlite_exec**。在接收到每行输入后，**sqlite**在其缓冲区上调用**sqlite_complete**。如果**sqlite_complete**返回 true，则调用**sqlite_exec**并重置输入缓冲区。如果**sqlite_complete**返回 false，则将提示更改为继续提示，并读取另一行文本并添加到输入缓冲区中。

#### 3.6 库版本字符串

SQLite 库导出名为 **sqlite_version** 的字符串常量，其中包含库的版本号。头文件包含一个名为 SQLITE_VERSION 的宏，具有相同的信息。如果需要，程序可以将 SQLITE_VERSION 宏与 **sqlite_version** 字符串常量进行比较，以验证头文件的版本号与库的版本号是否匹配。

#### 3.7 库字符编码

默认情况下，SQLite 假定所有数据使用固定大小的 8 位字符（iso8859）。但是，如果在配置脚本中给出了 --enable-utf8 选项，则库将假定使用 UTF-8 可变大小字符。这会影响 LIKE 和 GLOB 运算符以及 LENGTH() 和 SUBSTR() 函数。静态字符串 **sqlite_encoding** 将设置为 "UTF-8" 或 "iso8859"，以指示库的编译方式。此外，**sqlite.h** 头文件将根据需要定义 **SQLITE_UTF8** 或 **SQLITE_ISO8859** 中的一个宏。

请注意，SQLite 使用的字符编码机制不能在运行时更改。这只是一个编译时选项。**sqlite_encoding** 字符串仅告诉您库是如何编译的。

#### 3.8 更改库对锁定文件的响应

**sqlite_busy_handler** 过程可用于向打开的 SQLite 数据库注册一个忙碌回调。每当 SQLite 尝试访问一个已锁定的数据库时，将调用忙碌回调。回调通常会执行一些其他有用的工作，或者可能休眠，以便让锁有机会释放。如果回调返回非零值，则 SQLite 再次尝试访问数据库，循环重复。如果回调返回零，则 SQLite 中止当前操作并返回 SQLITE_BUSY。

**sqlite_busy_handler** 的参数包括从 **sqlite_open** 返回的不透明结构体指针，一个指向繁忙回调函数的指针，以及一个通用指针，将作为繁忙回调的第一个参数传递。当 SQLite 调用繁忙回调时，它会传递三个参数给它：作为 **sqlite_busy_handler** 第三个参数传入的通用指针，SQLite 正在尝试访问的数据库表或索引的名称，以及库试图访问数据库表或索引的次数。

对于常见情况，我们希望繁忙回调休眠，SQLite 库提供了一个便利函数 **sqlite_busy_timeout**。**sqlite_busy_timeout** 的第一个参数是指向打开的 SQLite 数据库的指针，第二个参数是毫秒数。在执行完 **sqlite_busy_timeout** 后，SQLite 库将等待锁至少指定的毫秒数，然后返回 SQLITE_BUSY。将超时设置为零毫秒将恢复默认行为。

#### 3.9 使用 `_printf()` 封装函数

这四个实用函数

+   **sqlite_exec_printf()**

+   **sqlite_exec_vprintf()**

+   **sqlite_get_table_printf()**

+   **sqlite_get_table_vprintf()**

实现了与 **sqlite_exec** 和 **sqlite_get_table** 相同的查询功能。但是，这四个 **_printf** 程序不接受完整的 SQL 语句作为它们的第二个参数，而是接受一个类似于 printf 的格式字符串。要执行的 SQL 语句是从此格式字符串生成的，并根据附加到函数调用末尾的任何其他参数生成。

使用 SQLite printf 函数而不是 **sprintf** 有两个优点。首先，使用 SQLite printf 函数，不会像使用 **sprintf** 那样存在溢出静态缓冲区的危险。SQLite printf 函数会自动分配（和稍后释放）足够容纳生成的 SQL 语句所需的内存。

SQLite printf 例程相对于 **sprintf** 的第二个优点是具有两个新的格式选项，专门设计用于支持 SQL 中的字符串字面值。在格式字符串内部，%q 格式选项的工作方式与 %s 非常相似，它从参数列表中读取以 null 结尾的字符串，并将其插入结果中。但是 %q 通过制作被替换字符串中每个单引号（'）字符的两个副本来翻译插入的字符串。这样做的效果是转义字符串字面值中的单引号的字符串字面值的字符串末尾含义。%Q 格式选项类似工作；它像 %q 一样翻译单引号，并额外用单引号括起生成的字符串。如果 %Q 格式选项的参数是 NULL 指针，则生成的字符串是没有单引号的 NULL。

考虑一个例子。假设您试图将一个字符串值插入到数据库表中，该字符串值是从用户输入获取的。假设要插入的字符串存储在名为 zString 的变量中。执行插入操作的代码可能如下所示：

> ```sql
> sqlite_exec_printf(db,
>   "INSERT INTO table1 VALUES('%s')",
>   0, 0, 0, zString);
> 
> ```

如果 zString 变量保存的文本类似于 "Hello"，那么这个语句将正常工作。但是假设用户输入了类似于 "Hi y'all!" 的字符串。生成的 SQL 语句如下所示：

> ```sql
> INSERT INTO table1 VALUES('Hi y'all')
> 
> ```

这不是有效的 SQL，因为单词 "y'all" 中有撇号。但是如果使用 %q 格式选项而不是 %s，如下所示：

> ```sql
> sqlite_exec_printf(db,
>   "INSERT INTO table1 VALUES('%q')",
>   0, 0, 0, zString);
> 
> ```

那么生成的 SQL 将如下所示：

> ```sql
> INSERT INTO table1 VALUES('Hi y''all')
> 
> ```

这里撇号已经转义，SQL 语句形式良好。当从可能包含单引号字符（'）的数据中动态生成 SQL 时，建议始终使用 SQLite printf 例程和 %q 格式选项，而不是 **sprintf**。

如果使用 %Q 格式选项而不是 %q，如下所示：

> ```sql
> sqlite_exec_printf(db,
>   "INSERT INTO table1 VALUES(%Q)",
>   0, 0, 0, zString);
> 
> ```

那么生成的 SQL 将如下所示：

> ```sql
> INSERT INTO table1 VALUES('Hi y''all')
> 
> ```

如果 zString 变量的值为 NULL，生成的 SQL 将如下所示：

> ```sql
> INSERT INTO table1 VALUES(NULL)
> 
> ```

所有上述 _printf() 例程都围绕以下两个函数构建：

> ```sql
> char *sqlite_mprintf(const char *zFormat, ...);
> char *sqlite_vmprintf(const char *zFormat, va_list);
> 
> ```

**sqlite_mprintf()** 例程的工作方式类似于标准库 **sprintf()**，不同之处在于它将结果写入从 malloc() 获得的内存中，并返回指向 malloc 缓冲区的指针。 **sqlite_mprintf()** 也理解上述的 %q 和 %Q 扩展。 **sqlite_vmprintf()** 是相同例程的可变参数版本。 这些例程返回的字符串指针应通过将其传递给 **sqlite_freemem()** 来释放。

#### 3.10 在大查询期间执行后台作业

**sqlite_progress_handler()** 例程可用于向 SQLite 数据库注册回调例程，在对 **sqlite_exec()**、**sqlite_step()** 和各种包装函数进行长时间调用期间定期调用。

回调函数在每个 N 虚拟机操作时调用，其中 N 作为第二个参数提供给 **sqlite_progress_handler()**。 **sqlite_progress_handler()** 的第三个和第四个参数分别是要调用的例程的指针以及作为其第一个参数传递的 void 指针。

执行每个虚拟机操作所需的时间可以根据许多因素而变化。 对于 1 GHz PC，每秒的典型值在半秒和三百万之间，但可能会更高或更低，具体取决于查询。 因此，很难根据虚拟机操作安排后台操作。 相反，建议相对频繁地安排回调（例如每 1000 条指令），并使用外部定时器例程确定是否需要运行后台作业。

### 4.0 添加新的 SQL 函数

从版本 2.4.0 开始，SQLite 允许使用 C 代码扩展 SQL 语言。 使用以下接口：

> ```sql
> typedef struct sqlite_func sqlite_func;
> 
> int sqlite_create_function(
>   sqlite *db,
>   const char *zName,
>   int nArg,
>   void (*xFunc)(sqlite_func*,int,const char**),
>   void *pUserData
> );
> int sqlite_create_aggregate(
>   sqlite *db,
>   const char *zName,
>   int nArg,
>   void (*xStep)(sqlite_func*,int,const char**),
>   void (*xFinalize)(sqlite_func*),
>   void *pUserData
> );
> 
> char *sqlite_set_result_string(sqlite_func*,const char*,int);
> void sqlite_set_result_int(sqlite_func*,int);
> void sqlite_set_result_double(sqlite_func*,double);
> void sqlite_set_result_error(sqlite_func*,const char*,int);
> 
> void *sqlite_user_data(sqlite_func*);
> void *sqlite_aggregate_context(sqlite_func*, int nBytes);
> int sqlite_aggregate_count(sqlite_func*);
> 
> ```

**sqlite_create_function()**接口用于创建常规函数，**sqlite_create_aggregate()**用于创建新的聚合函数。在两种情况下，**db**参数是一个打开的 SQLite 数据库，这些函数将在其上注册，**zName**是新函数的名称，**nArg**是参数数量，**pUserData**是一个指针，将无变化地传递给函数的 C 实现。这两个例程在成功时返回 0，如果有任何错误则返回非零值。

函数名的长度不能超过 255 个字符。任何尝试创建长度超过 255 个字符的函数都会导致错误。

对于常规函数，每次函数调用都会调用**xFunc**回调函数。实现**xFunc**的时候应调用其中一个**sqlite_set_result_...**接口来返回其结果。**sqlite_user_data()**例程可用于检索在注册函数时传递的**pUserData**指针。

对于聚合函数，**xStep**回调在结果的每一行上调用一次，然后在结束时调用**xFinalize**来计算最终答案。**xStep**例程可以使用**sqlite_aggregate_context()**接口来分配内存，该内存将是特定 SQL 函数实例的唯一。此内存在调用**xFinalize**后将自动删除。**sqlite_aggregate_count()**例程可用于查找传递给聚合函数的数据行数。**xFinalize**回调应调用其中一个**sqlite_set_result_...**接口来设置聚合的最终结果。

SQLite 现在使用这个接口实现其所有内置函数。有关如何创建新 SQL 函数的更多信息和示例，请查看文件**func.c**中的 SQLite 源代码。

### 5.0 多线程与 SQLite

如果 SQLite 是用 THREADSAFE 预处理宏设置为 1 编译的，则可以安全地在同一进程的两个或更多线程中同时使用 SQLite。但是每个线程应该有自己从 **sqlite_open** 返回的 **sqlite*** 指针。不允许两个或更多线程同时访问同一个 **sqlite*** 指针。

在网站上提供的预编译 SQLite 库中，Unix 版本被编译为关闭 THREADSAFE，而 Windows 版本被编译为开启 THREADSAFE。如果你需要不同的设置，你需要重新编译。

在 Unix 系统下，**sqlite*** 指针不应该在 **fork()** 系统调用中传递到子进程中。子进程应该在 **fork()** 之后打开自己的数据库副本。

### 6.0 使用示例

关于 SQLite C/C++ 接口的使用示例，请参考源码中的 **sqlite** 程序文件 [src/shell.c](https://sqlite.org/src/file/src/shell.c.in)。关于 SQLite 的其他信息请参考 cli.html。另请参考 SQLite 的 Tcl 接口源文件 [src/tclsqlite.c](https://sqlite.org/src/file/src/tclsqlite.c)。
