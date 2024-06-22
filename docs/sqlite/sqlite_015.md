# 1\. 结果码与错误码

> 原文：[`sqlite.com/rescode.html`](https://sqlite.com/rescode.html)

## 概述

SQLite 的 C 语言接口中的许多例程返回数字结果码，指示成功或失败，并在失败时提供了失败原因的一些想法。本文旨在解释每个数字结果码的含义。

“错误码”是指示出现问题的“结果码”的一个子集。只有少数非错误结果码：SQLITE_OK，SQLITE_ROW 和 SQLITE_DONE。术语“错误码”表示除这三个之外的任何结果码。

# 2\. 主结果码与扩展结果码

结果码是签名的 32 位整数。结果码的最低有效 8 位定义了一个广泛的类别，称为“主结果码”。更高位提供了关于错误更详细的信息，称为“扩展结果码”。

注意，主结果码始终是扩展结果码的一部分。给定完整的 32 位扩展结果码，应用程序始终可以通过提取扩展结果码的最低有效 8 位找到相应的主结果码。

所有扩展结果码也都是错误码。因此，“扩展结果码”和“扩展错误码”的术语是可互换的。

出于历史兼容性考虑，默认情况下，C 语言接口返回主结果码。可以使用 sqlite3_extended_errcode()接口检索最近错误的扩展结果码。可以使用 sqlite3_extended_result_codes()接口将数据库连接置于返回扩展结果码而非主结果码的模式中。

# 3\. 定义

所有结果码都是整数。所有结果码的符号名称都是在 sqlite3.h 头文件中使用“#define”宏创建的。sqlite3.h 头文件中有单独的部分用于 result code definitions 和 extended result code definitions。

主结果码符号名称的形式为“SQLITE_XXXXXX”，其中 XXXXXX 是一系列大写字母字符。扩展结果码名称的形式为“SQLITE_XXXXXX_YYYYYYY”，其中 XXXXXX 部分是相应的主结果码，YYYYYYY 是进一步分类结果码的扩展。

现有结果码的名称和数值是固定且不变的。但是，SQLite 的未来版本中可能会出现新的结果码，特别是新的扩展结果码。

# 4\. 主结果码列表

31 个结果码在 sqlite3.h 中定义，并按字母顺序列出如下：

+   SQLITE_ABORT (4)

+   SQLITE_AUTH (23)

+   SQLITE_BUSY (5)

+   SQLITE_CANTOPEN (14)

+   SQLITE_CONSTRAINT (19)

+   SQLITE_CORRUPT (11)

+   SQLITE_DONE (101)

+   SQLITE_EMPTY (16)

+   SQLITE_ERROR (1)

+   SQLITE_FORMAT (24)

+   SQLITE_FULL (13)

+   SQLITE_INTERNAL (2)

+   SQLITE_INTERRUPT (9)

+   SQLITE_IOERR (10)

+   SQLITE_LOCKED (6)

+   SQLITE_MISMATCH (20)

+   SQLITE_MISUSE (21)

+   SQLITE_NOLFS (22)

+   SQLITE_NOMEM (7)

+   SQLITE_NOTADB (26)

+   SQLITE_NOTFOUND (12)

+   SQLITE_NOTICE (27)

+   SQLITE_OK (0)

+   SQLITE_PERM (3)

+   SQLITE_PROTOCOL (15)

+   SQLITE_RANGE (25)

+   SQLITE_READONLY (8)

+   SQLITE_ROW (100)

+   SQLITE_SCHEMA (17)

+   SQLITE_TOOBIG (18)

+   SQLITE_WARNING (28)

# -   扩展结果代码列表

-   74 个扩展结果代码在 sqlite3.h 中定义并按字母顺序列出如下：

+   SQLITE_ABORT_ROLLBACK (516)

+   SQLITE_AUTH_USER (279)

+   SQLITE_BUSY_RECOVERY (261)

+   SQLITE_BUSY_SNAPSHOT (517)

+   SQLITE_BUSY_TIMEOUT (773)

+   SQLITE_CANTOPEN_CONVPATH (1038)

+   SQLITE_CANTOPEN_DIRTYWAL (1294)

+   SQLITE_CANTOPEN_FULLPATH (782)

+   SQLITE_CANTOPEN_ISDIR (526)

+   SQLITE_CANTOPEN_NOTEMPDIR (270)

+   SQLITE_CANTOPEN_SYMLINK (1550)

+   SQLITE_CONSTRAINT_CHECK (275)

+   SQLITE_CONSTRAINT_COMMITHOOK (531)

+   SQLITE_CONSTRAINT_DATATYPE (3091)

+   SQLITE_CONSTRAINT_FOREIGNKEY (787)

+   SQLITE_CONSTRAINT_FUNCTION (1043)

+   SQLITE_CONSTRAINT_NOTNULL (1299)

+   SQLITE_CONSTRAINT_PINNED (2835)

+   SQLITE_CONSTRAINT_PRIMARYKEY (1555)

+   SQLITE_CONSTRAINT_ROWID (2579)

+   SQLITE_CONSTRAINT_TRIGGER (1811)

+   SQLITE_CONSTRAINT_UNIQUE (2067)

+   SQLITE_CONSTRAINT_VTAB (2323)

+   SQLITE_CORRUPT_INDEX (779)

+   SQLITE_CORRUPT_SEQUENCE (523)

+   SQLITE_CORRUPT_VTAB (267)是指**SQLITE_CORRUPT_VTAB (267)**。

+   SQLITE_ERROR_MISSING_COLLSEQ (257)是指**SQLITE_ERROR_MISSING_COLLSEQ (257)**。

+   SQLITE_ERROR_RETRY (513)是指**SQLITE_ERROR_RETRY (513)**。

+   SQLITE_ERROR_SNAPSHOT (769)是指**SQLITE_ERROR_SNAPSHOT (769)**。

+   SQLITE_IOERR_ACCESS (3338)是指**SQLITE_IOERR_ACCESS (3338)**。

+   SQLITE_IOERR_AUTH (7178)是指**SQLITE_IOERR_AUTH (7178)**。

+   SQLITE_IOERR_BEGIN_ATOMIC (7434)是指**SQLITE_IOERR_BEGIN_ATOMIC (7434)**。

+   SQLITE_IOERR_BLOCKED (2826)是指**SQLITE_IOERR_BLOCKED (2826)**。

+   SQLITE_IOERR_CHECKRESERVEDLOCK (3594)是指**SQLITE_IOERR_CHECKRESERVEDLOCK (3594)**。

+   SQLITE_IOERR_CLOSE (4106)是指**SQLITE_IOERR_CLOSE (4106)**。

+   SQLITE_IOERR_COMMIT_ATOMIC (7690)是指**SQLITE_IOERR_COMMIT_ATOMIC (7690)**。

+   SQLITE_IOERR_CONVPATH (6666)是指**SQLITE_IOERR_CONVPATH (6666)**。

+   SQLITE_IOERR_CORRUPTFS (8458)是指**SQLITE_IOERR_CORRUPTFS (8458)**。

+   SQLITE_IOERR_DATA (8202)是指**SQLITE_IOERR_DATA (8202)**。

+   SQLITE_IOERR_DELETE (2570)是指**SQLITE_IOERR_DELETE (2570)**。

+   SQLITE_IOERR_DELETE_NOENT (5898)是指**SQLITE_IOERR_DELETE_NOENT (5898)**。

+   SQLITE_IOERR_DIR_CLOSE (4362)是指**SQLITE_IOERR_DIR_CLOSE (4362)**。

+   SQLITE_IOERR_DIR_FSYNC (1290)是指**SQLITE_IOERR_DIR_FSYNC (1290)**。

+   SQLITE_IOERR_FSTAT (1802)是指**SQLITE_IOERR_FSTAT (1802)**。

+   SQLITE_IOERR_FSYNC (1034)是指**SQLITE_IOERR_FSYNC (1034)**。

+   SQLITE_IOERR_GETTEMPPATH (6410)是指**SQLITE_IOERR_GETTEMPPATH (6410)**。

+   SQLITE_IOERR_LOCK (3850)是指**SQLITE_IOERR_LOCK (3850)**。

+   SQLITE_IOERR_MMAP (6154)是指**SQLITE_IOERR_MMAP (6154)**。

+   SQLITE_IOERR_NOMEM (3082)是指**SQLITE_IOERR_NOMEM (3082)**。

+   SQLITE_IOERR_RDLOCK (2314)是指**SQLITE_IOERR_RDLOCK (2314)**。

+   SQLITE_IOERR_READ (266)是指**SQLITE_IOERR_READ (266)**。

+   SQLITE_IOERR_ROLLBACK_ATOMIC (7946)是指**SQLITE_IOERR_ROLLBACK_ATOMIC (7946)**。

+   SQLITE_IOERR_SEEK (5642)是指**SQLITE_IOERR_SEEK (5642)**。

+   SQLITE_IOERR_SHMLOCK (5130)是指**SQLITE_IOERR_SHMLOCK (5130)**。

+   SQLITE_IOERR_SHMMAP (5386)是指**SQLITE_IOERR_SHMMAP (5386)**。

+   SQLITE_IOERR_SHMOPEN (4618)是指**SQLITE_IOERR_SHMOPEN (4618)**。

+   SQLITE_IOERR_SHMSIZE (4874)是指**SQLITE_IOERR_SHMSIZE (4874)**。

+   SQLITE_IOERR_SHORT_READ (522)是指**SQLITE_IOERR_SHORT_READ (522)**。

+   SQLITE_IOERR_TRUNCATE (1546)是指**SQLITE_IOERR_TRUNCATE (1546)**。

+   SQLITE_IOERR_UNLOCK (2058)是指**SQLITE_IOERR_UNLOCK (2058)**。

+   SQLITE_IOERR_VNODE (6922)是指**SQLITE_IOERR_VNODE (6922)**。

+   SQLITE_IOERR_WRITE (778)是指**SQLITE_IOERR_WRITE (778)**。

+   SQLITE_LOCKED_SHAREDCACHE (262)是指**SQLITE_LOCKED_SHAREDCACHE (262)**。

+   SQLITE_LOCKED_VTAB (518)是指**SQLITE_LOCKED_VTAB (518)**。

+   SQLITE_NOTICE_RECOVER_ROLLBACK (539)是指**SQLITE_NOTICE_RECOVER_ROLLBACK (539)**。

+   SQLITE_NOTICE_RECOVER_WAL (283)是指**SQLITE_NOTICE_RECOVER_WAL (283)**。

+   SQLITE_OK_LOAD_PERMANENTLY (256)是指**SQLITE_OK_LOAD_PERMANENTLY (256)**。

+   SQLITE_READONLY_CANTINIT (1288)是指**SQLITE_READONLY_CANTINIT (1288)**。

+   SQLITE_READONLY_CANTLOCK (520)是指**SQLITE_READONLY_CANTLOCK (520)**。

+   SQLITE_READONLY_DBMOVED (1032)是指**SQLITE_READONLY_DBMOVED (1032)**。

+   SQLITE_READONLY_DIRECTORY (1544)是指**SQLITE_READONLY_DIRECTORY (1544)**。

+   SQLITE_READONLY_RECOVERY (264)是指**SQLITE_READONLY_RECOVERY (264)**。

+   SQLITE_READONLY_ROLLBACK (776)是指**SQLITE_READONLY_ROLLBACK (776)**。

+   SQLITE_WARNING_AUTOINDEX (284)是指**SQLITE_WARNING_AUTOINDEX (284)**。

# 6\. Result Code Meanings

所有 105 个结果代码值的含义如下，按数字顺序显示。

### (0) SQLITE_OK

SQLITE_OK 结果代码表示操作成功且没有错误。大多数其他结果代码表示错误。

### (1) SQLITE_ERROR

SQLITE_ERROR 结果代码是一种通用错误代码，在没有其他更具体的错误代码可用时使用。

### (2) SQLITE_INTERNAL

SQLITE_INTERNAL 结果代码表示内部故障。在 SQLite 的工作版本中，应用程序不应看到此结果代码。如果应用程序确实遇到此结果代码，则表明数据库引擎中存在错误。

SQLite 当前不生成此结果代码。然而，应用程序定义的 SQL 函数或虚拟表，或 VFSes，或其他扩展可能导致返回此结果代码。

### (3) SQLITE_PERM

SQLITE_PERM 结果代码指示无法为新创建的数据库提供请求的访问模式。

### (4) SQLITE_ABORT

SQLITE_ABORT 结果代码指示操作在完成之前被中止，通常是由应用程序请求。另请参阅：SQLITE_INTERRUPT。

如果 sqlite3_exec()的回调函数返回非零，则 sqlite3_exec()将返回 SQLITE_ABORT。

如果在与待处理的读取或写入操作在同一数据库连接上发生 ROLLBACK 操作，则可能会导致待处理的读取或写入操作失败，并出现 SQLITE_ABORT 或 SQLITE_ABORT_ROLLBACK 错误。

除了作为结果代码外，SQLITE_ABORT 值还用作从 sqlite3_vtab_on_conflict()接口返回的冲突解决模式。

### (5) SQLITE_BUSY

SQLITE_BUSY 结果代码指示由于其他数据库连接，通常是独立进程中的数据库连接的并发活动，无法写入（或在某些情况下读取）数据库文件。

例如，如果进程 A 正在进行大型写入事务的中间阶段，并且同时进程 B 试图启动新的写入事务，进程 B 将会收到 SQLITE_BUSY 结果，因为 SQLite 一次只支持一个写入者。进程 B 需要等待进程 A 完成其事务，然后才能启动新事务。sqlite3_busy_timeout()和 sqlite3_busy_handler()接口以及 busy_timeout pragma 可用于帮助进程 B 处理 SQLITE_BUSY 错误。

SQLITE_BUSY 错误可能发生在事务的任何时间点：事务最初开始时，在任何写入或更新操作期间，或者在事务提交时。为了避免在事务中间遇到 SQLITE_BUSY 错误，应用程序可以使用 BEGIN IMMEDIATE 而不仅仅是 BEGIN 来开始事务。BEGIN IMMEDIATE 命令可能本身返回 SQLITE_BUSY，但如果成功，则 SQLite 保证在接下来的 COMMIT 之后对同一数据库的后续操作不会返回 SQLITE_BUSY。

参见：SQLITE_BUSY_RECOVERY 和 SQLITE_BUSY_SNAPSHOT。

SQLITE_BUSY 结果代码与 SQLITE_LOCKED 不同之处在于，SQLITE_BUSY 表示与另一个可能在不同进程中的独立 数据库连接 冲突，而 SQLITE_LOCKED 表示与同一 数据库连接 （或者有 共享缓存 的数据库连接）内的冲突。

### (6) SQLITE_LOCKED

SQLITE_LOCKED 结果代码表示由于同一 数据库连接 内部冲突或与使用 共享缓存 的不同数据库连接之间的冲突，写操作无法继续。

例如，在同一 数据库连接 上的另一个线程正在读取表时，无法运行 DROP TABLE 语句，因为删除表将在并发读取者下删除表。

SQLITE_LOCKED 结果代码与 SQLITE_BUSY 不同之处在于，SQLITE_LOCKED 表示在同一 数据库连接 （或具有 共享缓存 的连接）上的冲突，而 SQLITE_BUSY 表示与不同数据库连接上的冲突，可能在不同进程中。

### (7) SQLITE_NOMEM

SQLITE_NOMEM 结果代码表示 SQLite 无法分配完成操作所需的所有内存。换句话说，在需要继续操作的内存分配失败时，sqlite3_malloc() 或 sqlite3_realloc() 内部调用失败。

### (8) SQLITE_READONLY

SQLITE_READONLY 结果代码在尝试修改某些数据但当前数据库连接没有写权限时返回。

### (9) SQLITE_INTERRUPT

SQLITE_INTERRUPT 结果代码表示操作被 sqlite3_interrupt() 接口中断。参见：SQLITE_ABORT

### (10) SQLITE_IOERR

SQLITE_IOERR 结果代码表明由于操作系统报告 I/O 错误，操作无法完成。

满磁盘驱动器通常会导致 SQLITE_FULL 错误，而不是 SQLITE_IOERR 错误。

I/O 错误有许多不同的扩展结果代码，用于标识失败的特定 I/O 操作。

### (11) SQLITE_CORRUPT

SQLITE_CORRUPT 结果代码表示数据库文件已损坏。请参阅如何损坏数据库文件以进一步讨论损坏可能发生的情况。

### (12) SQLITE_NOTFOUND

SQLITE_NOTFOUND 结果代码以三种方式暴露：

1.  SQLITE_NOTFOUND 可以由 sqlite3_file_control()接口返回，指示底层 VFS 未识别作为第三个参数传递的 file control opcode。

1.  SQLITE_NOTFOUND 也可以由 sqlite3_vfs 对象的 xSetSystemCall()方法返回。

1.  SQLITE_NOTFOUND 可以由 sqlite3_vtab_rhs_value()返回，指示约束的右操作数对 xBestIndex 方法不可用。

SQLITE_NOTFOUND 结果代码也在 SQLite 实现中内部使用，但这些内部用途对应用程序不可见。

### (13) SQLITE_FULL

SQLITE_FULL 结果代码表示由于磁盘已满，写操作无法完成。请注意，此错误可能在尝试向主数据库文件写入信息时发生，也可能在写入临时磁盘文件时发生。

有时应用程序会遇到此错误，尽管主磁盘空间充裕，因为在将信息写入临时磁盘文件时，系统将临时文件存储在独立分区中，其空间远少于主磁盘。

### (14) SQLITE_CANTOPEN

SQLITE_CANTOPEN 结果代码表示 SQLite 无法打开文件。问题文件可能是主数据库文件或几个临时磁盘文件之一。

### (15) SQLITE_PROTOCOL

SQLITE_PROTOCOL 结果代码指示 SQLite 使用的文件锁定协议存在问题。当前仅在 WAL 模式下尝试启动新事务时返回 SQLITE_PROTOCOL 错误。存在一种竞争条件，当两个独立的数据库连接同时尝试在 WAL 模式下启动事务时可能会发生。竞争的失败者会稍作延迟后重试。如果同一个连接在多次竞争中失去锁定多达几十次，并持续多秒钟，它最终会放弃并返回 SQLITE_PROTOCOL。实际上，SQLITE_PROTOCOL 错误极少出现，仅在多个进程强烈竞争写入同一数据库时才会出现。

### (16) SQLITE_EMPTY

SQLITE_EMPTY 结果代码目前未使用。

### (17) SQLITE_SCHEMA

SQLITE_SCHEMA 结果代码表示数据库架构已更改。这个结果代码可以在使用 sqlite3_prepare() 或 sqlite3_prepare16() 生成的 预处理语句 在准备和运行语句之间，数据库架构被其他进程更改的情况下返回。

如果从 sqlite3_prepare_v2() 生成了一个 预处理语句，那么如果模式发生更改，该语句将自动重新准备，最多 SQLITE_MAX_SCHEMA_RETRY 次（默认为 50 次）。只有在这些重试次数之后失败仍然持续时，sqlite3_step() 接口才会向应用程序返回 SQLITE_SCHEMA。

### (18) SQLITE_TOOBIG

SQLITE_TOOBIG 错误代码表示一个字符串或 BLOB 太大。在 SQLite 中，字符串或 BLOB 的默认最大长度为 1,000,000,000 字节。当 SQLite 遇到超过编译时或运行时限制的字符串或 BLOB 时，将导致 SQLITE_TOOBIG 错误。

SQLITE_TOOBIG 错误代码还可能发生在将超大的 SQL 语句传递到 sqlite3_prepare_v2() 接口之一时。SQL 语句的最大长度默认为较小的值，即 1,000,000,000 字节。可以通过编译时使用 SQLITE_MAX_SQL_LENGTH 或运行时使用 sqlite3_limit(db,SQLITE_LIMIT_SQL_LENGTH,...) 设置最大 SQL 语句长度。

### (19) SQLITE_CONSTRAINT

SQLITE_CONSTRAINT 错误代码表示在处理 SQL 语句时发生了 SQL 约束违反。可以通过查阅伴随的错误消息（通过 sqlite3_errmsg() 或 sqlite3_errmsg16() 返回）或查看 扩展错误代码 来获取有关失败约束的额外信息。

SQLITE_CONSTRAINT 代码还可以作为 虚拟表 实现的 xBestIndex() 方法的返回值。当 xBestIndex() 返回 SQLITE_CONSTRAINT 时，这表示提交给 xBestIndex() 的特定输入组合不能生成可用的查询计划，因此不应进一步考虑。

### (20) SQLITE_MISMATCH

SQLITE_MISMATCH 错误代码指示数据类型不匹配。

通常情况下，SQLite 对值的类型与容器的声明类型之间的不匹配很宽容。例如，SQLite 允许应用程序将大的 BLOB 存储在声明为 BOOLEAN 类型的列中。但在少数情况下，SQLite 对类型要求严格。当类型不匹配时，会返回 SQLITE_MISMATCH 错误。

表的 rowid 必须是整数。尝试将 rowid 设置为除整数之外的任何值（或自动转换为下一个可用整数 rowid 的 NULL）将导致 SQLITE_MISMATCH 错误。

### (21) SQLITE_MISUSE

如果应用程序以未定义或不支持的方式使用任何 SQLite 接口，可能会返回 SQLITE_MISUSE 返回码。例如，在已完成的准备语句之后使用该准备语句可能会导致 SQLITE_MISUSE 错误。

SQLite 试图检测并报告使用此结果代码的误用。然而，并不能保证误用检测一定成功。误用检测是概率性的。应用程序不应该依赖于 SQLITE_MISUSE 的返回值。

如果 SQLite 从任何接口返回 SQLITE_MISUSE，则表示应用程序编码错误，需要修复。不要发布有时从标准 SQLite 接口返回 SQLITE_MISUSE 的应用程序，因为这种应用程序可能存在严重的错误。

### (22) SQLITE_NOLFS

在不支持大文件的系统上，如果数据库增长到大于文件系统可以处理的大小，可能会返回 SQLITE_NOLFS 错误。"NOLFS"代表"NO Large File Support"。

### (23) SQLITE_AUTH

当授权回调指示正在准备的 SQL 语句未经授权时，将返回 SQLITE_AUTH 错误。

### (24) SQLITE_FORMAT

SQLITE_FORMAT 错误代码当前未被 SQLite 使用。

### (25) SQLITE_RANGE

SQLITE_RANGE 错误表明参数编号参数超出了 sqlite3_bind 例程之一或者在 sqlite3_column 例程之一中的列编号范围。

### (26) SQLITE_NOTADB

尝试打开文件时，SQLITE_NOTADB 错误表示正在打开的文件似乎不是 SQLite 数据库文件。

### (27) SQLITE_NOTICE

SQLITE_NOTICE（或其扩展错误代码之一）不会被任何 C/C++接口返回。然而，有时会在 sqlite3_log()回调的第一个参数中使用 SQLITE_NOTICE 来指示正在进行异常操作。

### (28) SQLITE_WARNING

SQLITE_WARNING 结果码不会被任何 C/C++ 接口返回。然而，SQLITE_WARNING（或者其 扩展错误码之一）有时作为 sqlite3_log() 回调的第一个参数，表示正在进行一项不寻常且可能不明智的操作。

### (100) SQLITE_ROW

sqlite3_step() 返回的 SQLITE_ROW 结果码表示还有另一行输出可用。

### (101) SQLITE_DONE

SQLITE_DONE 结果码表示操作已完成。SQLITE_DONE 结果码最常见作为 sqlite3_step() 的返回值，表示 SQL 语句已执行完毕。但 SQLITE_DONE 也可以由其他多步接口返回，如 sqlite3_backup_step()。

### (256) SQLITE_OK_LOAD_PERMANENTLY

sqlite3_load_extension() 接口将一个 extension 加载到单个数据库连接中。默认行为是在数据库连接关闭时自动卸载该扩展。然而，如果扩展入口点返回的是 SQLITE_OK_LOAD_PERMANENTLY 而不是 SQLITE_OK，则在数据库连接关闭后，扩展将继续加载到进程地址空间中。换句话说，当数据库连接关闭时，不会为扩展的 sqlite3_vfs 对象调用 xDlClose 方法。

SQLITE_OK_LOAD_PERMANENTLY 返回码对于注册新的 VFSes 的可加载扩展（例如）非常有用。

### (257) SQLITE_ERROR_MISSING_COLLSEQ

SQLITE_ERROR_MISSING_COLLSEQ 结果码意味着无法准备一个 SQL 语句，因为该 SQL 语句中指定的排序序列无法找到。

有时在遇到此错误码时，sqlite3_prepare_v2() 例程将错误转换为 SQLITE_ERROR_RETRY，并尝试使用不需要使用未知排序序列的不同查询计划再次准备 SQL 语句。

### (261) SQLITE_BUSY_RECOVERY

SQLITE_BUSY_RECOVERY 错误码是用于 SQLITE_BUSY 的 扩展错误码，表明由于另一个进程正在恢复 WAL mode 数据库文件而导致操作无法继续。SQLITE_BUSY_RECOVERY 错误码仅在 WAL mode 数据库中发生。

### (262) SQLITE_LOCKED_SHAREDCACHE

**SQLITE_LOCKED_SHAREDCACHE**结果代码指示，一个 SQLite 数据记录的访问被另一个正在使用相同记录的数据库连接在共享缓存模式下阻塞。当两个或更多数据库连接共享同一缓存，并且其中一个连接正在修改该缓存中的记录时，其他连接在修改进行期间被阻塞，以防止读取者看到损坏或部分完成的更改。

### (264) SQLITE_READONLY_RECOVERY

**SQLITE_READONLY_RECOVERY**错误代码是扩展错误代码中的一种，用于 SQLITE_READONLY。**SQLITE_READONLY_RECOVERY**错误代码指示无法打开 WAL 模式数据库，因为需要对数据库文件进行恢复，而恢复需要写访问权限，但只有读访问权限可用。

### (266) SQLITE_IOERR_READ

**SQLITE_IOERR_READ**错误代码是扩展错误代码中的一种，用于指示在尝试从磁盘上的文件读取时，VFS 层发生 I/O 错误。此错误可能由硬件故障或因文件系统在文件打开时卸载而引起。

### (267) SQLITE_CORRUPT_VTAB

**SQLITE_CORRUPT_VTAB**错误代码是扩展错误代码中的一种，用于 SQLITE_CORRUPT，被虚拟表使用。虚拟表可能返回**SQLITE_CORRUPT_VTAB**来指示虚拟表中的内容损坏。

### (270) SQLITE_CANTOPEN_NOTEMPDIR

**SQLITE_CANTOPEN_NOTEMPDIR**错误代码不再使用。

### (275) SQLITE_CONSTRAINT_CHECK

**SQLITE_CONSTRAINT_CHECK**错误代码是扩展错误代码中的一种，用于指示 SQLITE_CONSTRAINT 因 CHECK 约束失败而出错。

### (279) SQLITE_AUTH_USER

**SQLITE_AUTH_USER**错误代码是扩展错误代码中的一种，用于 SQLITE_AUTH。它指示试图在缺乏足够授权的用户的数据库上执行操作。

### (283) SQLITE_NOTICE_RECOVER_WAL

当 WAL 模式数据库文件被恢复时，**SQLITE_NOTICE_RECOVER_WAL**结果代码被传递给 sqlite3_log()的回调函数。

### (284) SQLITE_WARNING_AUTOINDEX

当 sqlite3_log()的回调函数在使用自动索引时传递**SQLITE_WARNING_AUTOINDEX**结果代码。这可以作为应用程序设计者的一个警告，提示数据库可能需要额外的索引。

### (513) SQLITE_ERROR_RETRY

**SQLITE_ERROR_RETRY**用于内部使用，用以促使 sqlite3_prepare_v2()（或其创建准备语句的类似函数）再次尝试准备在上一次尝试中由于错误而失败的语句。

### (516) SQLITE_ABORT_ROLLBACK

SQLITE_ABORT_ROLLBACK 错误代码是扩展错误代码用于 SQLITE_ABORT，表示 SQL 语句由于第一次开始时活动的事务被回滚而中止。当回滚发生时，待定的写操作总是失败并显示此错误。如果在回滚的事务中更改了模式，则挂起的读操作只有在这种情况下才会失败。

### (517) SQLITE_BUSY_SNAPSHOT

SQLITE_BUSY_SNAPSHOT 错误代码是扩展错误代码用于 WAL 模式数据库，当一个数据库连接尝试将读事务升级为写事务，但发现另一个数据库连接已经写入数据库，从而使之前的读取无效时发生。

以下场景说明了 SQLITE_BUSY_SNAPSHOT 错误可能如何产生：

1.  进程 A 在数据库上启动读事务，并执行一个或多个 SELECT 语句。进程 A 保持事务开启。

1.  进程 B 更新数据库，改变了进程 A 之前读取的值。

1.  进程 A 现在试图向数据库写入。但是进程 A 对数据库内容的视图现在已经过时，因为进程 B 在进程 A 读取之后修改了数据库文件。因此，进程 A 会收到 SQLITE_BUSY_SNAPSHOT 错误。

### (518) SQLITE_LOCKED_VTAB

SQLITE_LOCKED_VTAB 结果代码不被 SQLite 核心使用，但可供扩展使用。虚拟表实现可以返回此结果代码，以指示它们由于其他线程或进程持有的锁而无法完成当前操作。

R-Tree 扩展在尝试更新 R-Tree 时，如果另一个预处理语句正在活跃地读取 R-Tree，则会返回此结果代码。更新无法进行，因为对 R-Tree 的任何更改可能涉及节点的重排和重新平衡，这将破坏读取游标，导致某些行重复出现，而其他行被省略。

### (520) SQLITE_READONLY_CANTLOCK

SQLITE_READONLY_CANTLOCK 错误代码是扩展错误代码用于 SQLITE_READONLY。SQLITE_READONLY_CANTLOCK 错误代码表示 SQLite 无法在 WAL 模式数据库上获取读锁，因为与该数据库关联的共享内存文件是只读的。

### (522) SQLITE_IOERR_SHORT_READ

SQLITE_IOERR_SHORT_READ 错误代码是扩展错误代码用于 SQLITE_IOERR，表示在 VFS 层中进行读取尝试时无法获取所请求的字节数。这可能是由于截断文件引起的。

### (523) SQLITE_CORRUPT_SEQUENCE

SQLITE_CORRUPT_SEQUENCE 结果代码意味着 sqlite_sequence 表的模式已损坏。sqlite_sequence 表用于帮助实现自增功能。sqlite_sequence 表应具有以下格式：

> ```sql
>   CREATE TABLE sqlite_sequence(name,seq);
>   
> ```

如果 SQLite 发现 sqlite_sequence 表有任何其他格式，它将返回 SQLITE_CORRUPT_SEQUENCE 错误。

### (526) SQLITE_CANTOPEN_ISDIR

SQLITE_CANTOPEN_ISDIR 错误代码是 SQLITE_CANTOPEN 的扩展错误代码，表示文件打开操作失败，因为文件实际上是一个目录。

### (531) SQLITE_CONSTRAINT_COMMITHOOK

SQLITE_CONSTRAINT_COMMITHOOK 错误代码是 SQLITE_CONSTRAINT 的扩展错误代码，表示一个提交挂钩回调返回非零，从而导致 SQL 语句被回滚。

### (539) SQLITE_NOTICE_RECOVER_ROLLBACK

SQLITE_NOTICE_RECOVER_ROLLBACK 结果代码在 sqlite3_log()的回调中传递，当热日志被回滚时。

### (769) SQLITE_ERROR_SNAPSHOT

在尝试使用 sqlite3_snapshot_open()接口在数据库的历史版本上启动读事务时，可能会返回 SQLITE_ERROR_SNAPSHOT 结果代码。如果历史快照不再可用，则读事务将因 SQLITE_ERROR_SNAPSHOT 而失败。只有在使用-DSQLITE_ENABLE_SNAPSHOT 编译 SQLite 时才可能出现此错误代码。

### (773) SQLITE_BUSY_TIMEOUT

SQLITE_BUSY_TIMEOUT 错误代码表示，在 VFS 层中阻塞的 Posix 建议文件锁请求由于超时而失败。阻塞的 Posix 建议锁仅作为专有的 SQLite 扩展可用，即使是这样，只有在编译 SQLite 时启用了 SQLITE_EANBLE_SETLK_TIMEOUT 编译时选项才支持。

### (776) SQLITE_READONLY_ROLLBACK

SQLITE_READONLY_ROLLBACK 错误代码是 SQLITE_READONLY 的扩展错误代码。SQLITE_READONLY_ROLLBACK 错误代码表示无法打开数据库，因为它具有需要回滚但无法回滚的热日志。

### (778) SQLITE_IOERR_WRITE

SQLITE_IOERR_WRITE 错误代码是 SQLITE_IOERR 的扩展错误代码，指示在尝试写入磁盘上的文件时，在 VFS 层中发生 I/O 错误。这种错误可能是由硬件故障或因文件系统在文件打开时卸载而导致的。如果文件系统已满，则不应发生此错误，因为有一个单独的错误代码（SQLITE_FULL）用于此目的。

### (779) SQLITE_CORRUPT_INDEX

SQLITE_CORRUPT_INDEX 结果代码表示 SQLite 检测到索引中的条目丢失或曾丢失。这是 SQLITE_CORRUPT 错误代码的特殊情况，建议通过运行 REINDEX 命令解决问题，假设数据库文件的其他地方没有其他问题。

### (782) SQLITE_CANTOPEN_FULLPATH

SQLITE_CANTOPEN_FULLPATH 错误代码是扩展错误代码，表示 SQLITE_CANTOPEN 的一个扩展，表明文件打开操作失败，因为操作系统无法将文件名转换为完整路径名。

### (787) SQLITE_CONSTRAINT_FOREIGNKEY

SQLITE_CONSTRAINT_FOREIGNKEY 错误代码是扩展错误代码，表示 SQLITE_CONSTRAINT 的一个扩展，指示外键约束失败。

### (1032) SQLITE_READONLY_DBMOVED

SQLITE_READONLY_DBMOVED 错误代码是扩展错误代码，表示 SQLITE_READONLY。SQLITE_READONLY_DBMOVED 错误代码表示数据库无法修改，因为数据库文件在打开后已被移动，如果进程崩溃，则任何尝试修改数据库的操作可能导致数据库损坏，因为回滚日志未能正确命名。

### (1034) SQLITE_IOERR_FSYNC

SQLITE_IOERR_FSYNC 错误代码是扩展错误代码，表示 SQLITE_IOERR 在 VFS 层中发生 I/O 错误，尝试将先前写入的内容从操作系统和/或磁盘控制缓冲区刷新到持久存储中。换句话说，此代码指示 unix 中的 fsync()系统调用或 windows 中的 FlushFileBuffers()系统调用存在问题。

### (1038) SQLITE_CANTOPEN_CONVPATH

SQLITE_CANTOPEN_CONVPATH 错误代码是扩展错误代码，表示 SQLITE_CANTOPEN 的一个扩展，仅在 Cygwin VFS 中使用，指示 cygwin_conv_path()系统调用在尝试打开文件时失败。另请参见：SQLITE_IOERR_CONVPATH

### (1043) SQLITE_CONSTRAINT_FUNCTION

SQLITE_CONSTRAINT_FUNCTION 错误代码目前未被 SQLite 核心使用。但是，此错误代码可供扩展函数使用。

### (1288) SQLITE_READONLY_CANTINIT

SQLITE_READONLY_CANTINIT 结果代码源自 VFS 中 xShmMap 方法，用于指示用于 WAL 模式的共享内存区存在，但其内容对当前进程不可靠且不可用，因为当前进程在共享内存区上没有写入权限。（WAL 模式的共享内存区通常是一个带有“-wal”后缀的文件，它被映射到进程空间中。如果当前进程对该文件没有写入权限，则无法将数据写入共享内存。）

SQLite 中的高级逻辑通常会拦截错误代码并创建一个临时的内存共享区域，以便当前进程至少可以读取数据库的内容。这个结果代码不应该到达应用程序接口层。

### (1290) SQLITE_IOERR_DIR_FSYNC

SQLITE_IOERR_DIR_FSYNC 错误代码是 扩展错误代码，表示在尝试在目录上调用 fsync() 时，在 VFS 层中发生 I/O 错误。Unix VFS 尝试在创建或删除某些文件后对目录执行 fsync()，以确保在断电或系统崩溃后这些文件仍将出现在文件系统中。此错误代码指示尝试执行该 fsync() 时出现问题。

### (1294) SQLITE_CANTOPEN_DIRTYWAL

SQLITE_CANTOPEN_DIRTYWAL 结果代码目前未使用。

### (1299) SQLITE_CONSTRAINT_NOTNULL

SQLITE_CONSTRAINT_NOTNULL 错误代码是 扩展错误代码，表示非空约束失败。

### (1544) SQLITE_READONLY_DIRECTORY

SQLITE_READONLY_DIRECTORY 结果代码指示数据库是只读的，因为进程没有在与数据库相同的目录中创建日志文件的权限，而创建日志文件是写入的先决条件。

### (1546) SQLITE_IOERR_TRUNCATE

SQLITE_IOERR_TRUNCATE 错误代码是 扩展错误代码，表示在尝试将文件截断为较小大小时，在 VFS 层中发生 I/O 错误。

### (1550) SQLITE_CANTOPEN_SYMLINK

SQLITE_CANTOPEN_SYMLINK 结果代码由 sqlite3_open() 接口及其衍生接口在使用 SQLITE_OPEN_NOFOLLOW 标志且数据库文件是符号链接时返回。

### (1555) SQLITE_CONSTRAINT_PRIMARYKEY

SQLITE_CONSTRAINT_PRIMARYKEY 错误代码是 扩展错误代码，表示主键约束失败。

### (1802) SQLITE_IOERR_FSTAT

SQLITE_IOERR_FSTAT 错误代码是 扩展错误代码，表示在尝试在文件上调用 fstat()（或等效操作）以确定诸如文件大小或访问权限等信息时，在 VFS 层中发生 I/O 错误。

### (1811) SQLITE_CONSTRAINT_TRIGGER

SQLITE_CONSTRAINT_TRIGGER 错误代码是 扩展错误代码，表示在触发器中触发了 RAISE 函数，导致 SQL 语句中止。

### (2058) SQLITE_IOERR_UNLOCK

SQLITE_IOERR_UNLOCK 错误代码是 扩展错误代码 用于 SQLITE_IOERR，指示 sqlite3_io_methods 对象上的 xUnlock 方法内部发生的 I/O 错误。

### (2067) SQLITE_CONSTRAINT_UNIQUE

SQLITE_CONSTRAINT_UNIQUE 错误代码是 扩展错误代码，用于 SQLITE_CONSTRAINT，表明 UNIQUE 约束 失败。

### (2314) SQLITE_IOERR_RDLOCK

SQLITE_IOERR_RDLOCK 错误代码是 扩展错误代码，用于 SQLITE_IOERR，指示在尝试获取读取锁时 sqlite3_io_methods 对象上的 xLock 方法内部发生的 I/O 错误。

### (2323) SQLITE_CONSTRAINT_VTAB

SQLITE_CONSTRAINT_VTAB 错误代码当前未被 SQLite 核心使用。但是，此错误代码可供应用程序定义的 虚拟表 使用。

### (2570) SQLITE_IOERR_DELETE

SQLITE_IOERR_DELETE 错误代码是 扩展错误代码，用于 SQLITE_IOERR，指示在 sqlite3_vfs 对象上的 xDelete 方法内部发生的 I/O 错误。

### (2579) SQLITE_CONSTRAINT_ROWID

SQLITE_CONSTRAINT_ROWID 错误代码是 扩展错误代码，用于 SQLITE_CONSTRAINT，表明 rowid 不唯一。

### (2826) SQLITE_IOERR_BLOCKED

SQLITE_IOERR_BLOCKED 错误代码不再使用。

### (2835) SQLITE_CONSTRAINT_PINNED

SQLITE_CONSTRAINT_PINNED 错误代码是 扩展错误代码，用于 SQLITE_CONSTRAINT，表明 UPDATE 触发器 在更新过程中尝试删除正在更新的行。

### (3082) SQLITE_IOERR_NOMEM

SQLITE_IOERR_NOMEM 错误代码有时由 VFS 层返回，表示由于无法分配足够的内存而无法完成操作。这个错误代码通常在返回给应用程序之前由 SQLite 的更高层次转换为 SQLITE_NOMEM。

### (3091) SQLITE_CONSTRAINT_DATATYPE

SQLITE_CONSTRAINT_DATATYPE 错误代码是 扩展错误代码，用于 SQLITE_CONSTRAINT，表明插入或更新尝试将与表定义为 STRICT 的列声明类型不一致的值存储。

### (3338) SQLITE_IOERR_ACCESS

SQLITE_IOERR_ACCESS 错误代码是 扩展错误代码，用于 SQLITE_IOERR，指示在 sqlite3_vfs 对象上的 xAccess 方法内部发生的 I/O 错误。

### (3594) SQLITE_IOERR_CHECKRESERVEDLOCK

SQLITE_IOERR_CHECKRESERVEDLOCK 错误码是扩展错误码中的一个，用于指示在 sqlite3_io_methods 对象的 xCheckReservedLock 方法中发生的 I/O 错误。

### (3850) SQLITE_IOERR_LOCK

SQLITE_IOERR_LOCK 错误码是扩展错误码中的一个，用于指示在咨询文件锁逻辑中发生的 I/O 错误。通常，SQLITE_IOERR_LOCK 错误表示在获取 PENDING 锁时出现问题。但它也可以指示 Mac 上某些专用 VFSes 上的其他锁定错误。

### (4106) SQLITE_IOERR_CLOSE

SQLITE_IOERR_CLOSE 错误码是扩展错误码中的一个，用于指示在 sqlite3_io_methods 对象的 xClose 方法中发生的 I/O 错误。

### (4362) SQLITE_IOERR_DIR_CLOSE

SQLITE_IOERR_DIR_CLOSE 错误码不再使用。

### (4618) SQLITE_IOERR_SHMOPEN

SQLITE_IOERR_SHMOPEN 错误码是扩展错误码中的一个，用于指示在尝试打开新的共享内存段时，在 sqlite3_io_methods 对象的 xShmMap 方法中发生的 I/O 错误。

### (4874) SQLITE_IOERR_SHMSIZE

SQLITE_IOERR_SHMSIZE 错误码是扩展错误码中的一个，用于指示在尝试作为 WAL 模式事务处理的一部分放大一个"shm"文件时，在 sqlite3_io_methods 对象的 xShmMap 方法中发生的 I/O 错误。此错误可能表示底层文件系统卷空间不足。

### (5130) SQLITE_IOERR_SHMLOCK

SQLITE_IOERR_SHMLOCK 错误码不再使用。

### (5386) SQLITE_IOERR_SHMMAP

SQLITE_IOERR_SHMMAP 错误码是扩展错误码中的一个，用于指示在尝试将共享内存段映射到进程地址空间时，在 sqlite3_io_methods 对象的 xShmMap 方法中发生的 I/O 错误。

### (5642) SQLITE_IOERR_SEEK

SQLITE_IOERR_SEEK 错误码是扩展错误码中的一个，用于指示在尝试将文件描述符定位到读取或写入操作将发生的文件的起始点时，在 sqlite3_io_methods 对象的 xRead 或 xWrite 方法中发生的 I/O 错误。

### (5898) SQLITE_IOERR_DELETE_NOENT

SQLITE_IOERR_DELETE_NOENT 错误码是扩展错误码中的一个，用于指示在尝试删除不存在的文件时，sqlite3_vfs 对象的 xDelete 方法失败。

### (6154) SQLITE_IOERR_MMAP

SQLITE_IOERR_MMAP 错误代码是扩展错误代码，用于 SQLITE_IOERR，指示在尝试将数据库文件的部分映射或取消映射到进程地址空间时，sqlite3_io_methods 对象上的 xFetch 或 xUnfetch 方法中发生了 I/O 错误。

### (6410) SQLITE_IOERR_GETTEMPPATH

SQLITE_IOERR_GETTEMPPATH 错误代码是扩展错误代码，用于 SQLITE_IOERR，指示 VFS 无法确定放置临时文件的合适目录。

### (6666) SQLITE_IOERR_CONVPATH

SQLITE_IOERR_CONVPATH 错误代码是扩展错误代码，用于 SQLITE_IOERR，仅由 Cygwin VFS 使用，指示 cygwin_conv_path()系统调用失败。另请参阅：SQLITE_CANTOPEN_CONVPATH

### (6922) SQLITE_IOERR_VNODE

SQLITE_IOERR_VNODE 错误代码是由扩展保留的代码。SQLite 核心不使用此代码。

### (7178) SQLITE_IOERR_AUTH

SQLITE_IOERR_AUTH 错误代码是由扩展保留的代码。SQLite 核心不使用此代码。

### (7434) SQLITE_IOERR_BEGIN_ATOMIC

SQLITE_IOERR_BEGIN_ATOMIC 错误代码指示底层操作系统在 SQLITE_FCNTL_BEGIN_ATOMIC_WRITE 文件控制上报告了错误。只有在启用 SQLITE_ENABLE_ATOMIC_WRITE 并且数据库托管在支持原子写入的文件系统上时才会出现此问题。

### (7690) SQLITE_IOERR_COMMIT_ATOMIC

SQLITE_IOERR_COMMIT_ATOMIC 错误代码指示底层操作系统在 SQLITE_FCNTL_COMMIT_ATOMIC_WRITE 文件控制上报告了错误。只有在启用 SQLITE_ENABLE_ATOMIC_WRITE 并且数据库托管在支持原子写入的文件系统上时才会出现此问题。

### (7946) SQLITE_IOERR_ROLLBACK_ATOMIC

SQLITE_IOERR_ROLLBACK_ATOMIC 错误代码指示底层操作系统在 SQLITE_FCNTL_ROLLBACK_ATOMIC_WRITE 文件控制上报告了错误。只有在启用 SQLITE_ENABLE_ATOMIC_WRITE 并且数据库托管在支持原子写入的文件系统上时才会出现此问题。

### (8202) SQLITE_IOERR_DATA

SQLITE_IOERR_DATA 错误代码是扩展错误代码，用于 SQLITE_IOERR，仅由 checksum VFS shim 使用，用于指示数据库文件页面的校验和不正确。

### (8458) SQLITE_IOERR_CORRUPTFS

SQLITE_IOERR_CORRUPTFS 错误码是 VFS 扩展错误码，用于 SQLITE_IOERR，仅由 VFS 使用，用于指示由于请求未落入文件边界而不是普通设备故障导致的寻址或读取失败。这通常表明文件系统损坏。
