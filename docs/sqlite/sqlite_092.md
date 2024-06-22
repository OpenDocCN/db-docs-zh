# 1\. 概述

> 原文：[`sqlite.com/compile.html`](https://sqlite.com/compile.html)

对于大多数情况，SQLite 可以使用默认的编译选项正常构建。然而，如果需要，下文记录的编译时选项可以用来省略 SQLite 的功能（导致编译后库的大小更小），或者修改一些参数的默认值。

我们已尽一切努力确保各种编译选项的组合能和谐地工作并生成可用的库。然而，强烈建议在使用非标准编译选项构建的 SQLite 库之前，执行 SQLite 测试套件以检查错误。

# 2\. 推荐的编译时选项

推荐以下编译时选项给能够使用的应用程序，以减少 SQLite 使用的 CPU 循环数和内存字节数。并非每个应用程序都能使用所有这些编译时选项。例如，SQLITE_THREADSAFE=0 选项只能被从不同时访问多个线程的应用程序使用。而 SQLITE_OMIT_PROGRESS_CALLBACK 选项只能被不使用 sqlite3_progress_handler() 接口的应用程序使用，等等。

不可能测试每种 SQLite 的编译时选项组合。但下面的一组编译时选项是始终经过全面测试的配置之一。

1.  **SQLITE_DQS=0**。此设置禁用了双引号字符串字面量的不良特性。

1.  **SQLITE_THREADSAFE=0**. 设置 -DSQLITE_THREADSAFE=0 导致 SQLite 中的所有互斥和线程安全逻辑被省略。这个单一的编译时选项使 SQLite 运行速度提高约 2%，同时还减小了库的大小约 2%。但缺点是，使用这个编译时选项意味着 SQLite 永远不能被多个线程同时使用，即使每个线程都有自己的数据库连接。

1.  **SQLITE_DEFAULT_MEMSTATUS=0**. 这个设置导致禁用跟踪内存使用的 sqlite3_status() 接口。这有助于使 sqlite3_malloc() 例程运行更快，而由于 SQLite 在内部使用 sqlite3_malloc()，整个库的速度也会更快。

1.  **SQLITE_DEFAULT_WAL_SYNCHRONOUS=1**. 为了在断电后最大限度地保证数据库安全，建议设置 PRAGMA synchronous=FULL。然而，在 WAL 模式 下，使用 PRAGMA synchronous=NORMAL 可以保证完整的数据库一致性。在 WAL 模式 下使用 PRAGMA synchronous=NORMAL，数据库的最近更改可能会在断电后被回滚，但不会损坏数据库。此外，在 WAL 模式下，使用 synchronous=NORMAL 比默认的 synchronous=FULL 提高了事务提交速度。因此，建议在切换到 WAL 模式时将 synchronous 设置从 FULL 改为 NORMAL。这个编译时选项将实现这一目标。

1.  **SQLITE_LIKE_DOESNT_MATCH_BLOBS**。历史上，SQLite 允许 BLOB 作为 LIKE 和 GLOB 操作符的操作数。但是，如果操作数之一是 BLOB，将 BLOB 作为 LIKE 或 GLOB 的操作数会使 LIKE 优化 变得复杂且减慢速度。当设置此选项时，意味着如果任一操作数是 BLOB，LIKE 和 GLOB 操作符始终返回 FALSE。这简化了 LIKE 优化 的实现，并允许使用 LIKE 优化 的查询运行更快。

1.  **SQLITE_MAX_EXPR_DEPTH=0**。将最大表达式解析树深度设置为零会禁用对表达式解析树深度的所有检查，从而简化代码，提高执行速度，并减少解析树的内存使用。

1.  **SQLITE_OMIT_DECLTYPE**。通过省略从查询结果集的列返回声明类型（很少需要的能力），可以使 prepared statements 消耗的内存更少。

1.  **SQLITE_OMIT_DEPRECATED**。省略弃用的接口和特性不会帮助 SQLite 更快地运行。但是，它会减少库的占用空间，并且这是正确的做法。

1.  **SQLITE_OMIT_PROGRESS_CALLBACK**。进度处理程序回调计数器必须在 bytecode engine 的内部循环中进行检查。通过省略此接口，可以从 bytecode engine 的内部循环中删除一个条件语句，从而帮助 SQL 语句运行稍快。

1.  **SQLITE_OMIT_SHARED_CACHE**。省略使用 shared cache 的可能性允许在代码的性能关键部分消除许多条件分支。这可以显著提升性能。

1.  **SQLITE_USE_ALLOCA**。在支持 alloca() 的系统上，利用 alloca() 动态分配临时堆栈空间，用于单个函数内部的使用。如果不使用此选项，则临时空间从堆中分配。

1.  **SQLITE_OMIT_AUTOINIT**。SQLite 库需要在使用某些接口之前通过调用 sqlite3_initialize() 进行初始化。通常情况下，第一次需要时会自动进行初始化。然而，使用 SQLITE_OMIT_AUTOINIT 选项时，自动初始化被省略。这使得许多 API 调用可以稍微快一些（因为它们不必检查初始化是否已经发生，如果尚未调用则运行初始化），但也意味着应用程序必须手动调用 sqlite3_initialize()。如果 SQLite 编译时使用了 -DSQLITE_OMIT_AUTOINIT，而在未先调用 sqlite3_initialize() 的情况下调用了像 sqlite3_malloc()、sqlite3_vfs_find() 或 sqlite3_open() 这样的例程，则可能导致段错误。

1.  **SQLITE_STRICT_SUBTYPE=1**. 这个选项会在应用程序定义的函数调用 sqlite3_result_subtype() 接口时，如果该函数没有设置 SQLITE_RESULT_SUBTYPE 属性，则会引发错误。但在注册了 SQLITE_RESULT_SUBTYPE 属性的函数中，sqlite3_result_subtype() 接口是可靠的。这个编译时选项旨在早期引起开发者对这个问题的关注。

当使用了上述所有推荐的编译时选项时，SQLite 库将减小约 3%，并且使用的 CPU 周期减少约 5%。因此，这些选项并不会带来巨大的差异。但在某些设计情况下，每一点帮助都很重要。

库级配置选项，例如上面列出的选项，可以选择地在客户端头文件中定义。定义 SQLITE_CUSTOM_INCLUDE=myconfig.h（不带引号）将导致 sqlite3.c 在编译过程的早期包含 myconfig.h，使客户端能够自定义标志，而无需显式地将所有标志传递给编译器。

# 3\. 平台配置

**_HAVE_SQLITE_CONFIG_H**

> 如果定义了 _HAVE_SQLITE_CONFIG_H 宏，那么 SQLite 源代码将尝试 #include 一个名为 "sqlite_cfg.h" 的文件。通常情况下，"sqlite_cfg.h" 文件包含其他配置选项，特别是由 autoconf 脚本生成的 "HAVE_*INTERFACE*" 类型选项。请注意，此头文件仅用于平台级配置，而不是库级配置。要在自定义头文件中设置 SQLite 级配置标志，请定义 SQLITE_CUSTOM_INCLUDE=myconfig.h，如前一节所述。

**HAVE_FDATASYNC**

> 如果 `HAVE_FDATASYNC` 编译时选项为真，则 Unix 系统上默认的 VFS 尝试在适当的情况下使用 `fdatasync()` 而不是 `fsync()`。如果此标志缺失或为假，则始终使用 `fsync()`。

**HAVE_GMTIME_R**

> 如果 `HAVE_GMTIME_R` 选项为真，并且如果 SQLITE_OMIT_DATETIME_FUNCS 为真，则 `CURRENT_TIME`、`CURRENT_DATE` 和 `CURRENT_TIMESTAMP` 关键字将使用线程安全的 `gmtime_r()` 接口而不是 `gmtime()`。在通常情况下，如果未定义或为假，则使用内置的 日期和时间函数 来实现 `CURRENT_TIME`、`CURRENT_DATE` 和 `CURRENT_TIMESTAMP` 关键字，不会调用 `gmtime_r()` 或 `gmtime()`。

**HAVE_ISNAN**

> 如果 `HAVE_ISNAN` 选项为真，则 SQLite 调用系统库中的 `isnan()` 函数来判断双精度浮点数是否为 NaN。如果 `HAVE_ISNAN` 未定义或为假，则 SQLite 使用自己的实现来替代 `isnan()`。

**HAVE_LOCALTIME_R**

> 如果 `HAVE_LOCALTIME_R` 选项为真，则 SQLite 使用线程安全的 `localtime_r()` 库例程而不是 `localtime()` 来帮助实现内置的 本地时间修饰符 到 日期和时间函数 的实现。

**HAVE_LOCALTIME_S**

> 如果 `HAVE_LOCALTIME_S` 选项为真，则 SQLite 使用线程安全的 `localtime_s()` 库例程而不是 `localtime()` 来帮助实现内置的 本地时间修饰符 到 日期和时间函数 的实现。

**HAVE_MALLOC_USABLE_SIZE**

> 如果 `HAVE_MALLOC_USABLE_SIZE` 选项为真，则 SQLite 尝试使用 malloc_usable_size() 接口来找到从标准库 malloc() 或 realloc() 函数获取的内存分配的大小。此选项仅适用于使用标准库 malloc() 的情况。在 Apple 系统上，将使用 "zone malloc"，因此此选项不适用。当然，如果应用程序使用 SQLITE_CONFIG_MALLOC 提供自己的 malloc 实现，则此选项无效。
> 
> 如果 `HAVE_MALLOC_USABLE_SIZE` 选项被省略或为假，则 SQLite 使用系统 malloc() 和 realloc() 的包装器，每次分配都会增加 8 个字节，并在初始 8 个字节中写入分配的大小，然后 SQLite 还实现了自己的 malloc_usable_size() 版本，该版本查询这个 8 字节的前缀以获取分配的大小。这种方法可以工作，但不是最优的。建议应用尽可能使用 `HAVE_MALLOC_USABLE_SIZE`。

**HAVE_STRCHRNUL**

> 如果 `HAVE_STRCHRNUL` 选项为真，则 SQLite 使用 `strchrnul()` 库函数。如果该选项不存在或为假，则 SQLite 使用自己的 strchrnul() 实现。

**HAVE_UTIME**

> 如果 `HAVE_UTIME` 选项为真，则内置但非标准的 "unix-dotfile" VFS 将使用 utime() 系统调用，而不是 utimes()，来设置锁文件的最后访问时间。

**SQLITE_BYTEORDER=*(0|1234|4321)***

> SQLite 需要知道目标 CPU 的本地字节顺序是大端还是小端。如果目标字节顺序在运行时不能确定，则 SQLITE_BYTEORDER 预处理器设置为 0；对于大端机器设置为 4321，对于小端机器设置为 1234。代码中有 #ifdef 用于为所有常见平台和编译器自动设置 SQLITE_BYTEORDER。然而，在为晦涩目标编译 SQLite 时，适当设置 SQLITE_BYTEORDER 可能是有利的。如果目标字节顺序无法在编译时确定，则 SQLite 会退回到执行时检查，这始终有效，尽管会有小的性能损失。

# 4\. 选项设置默认参数值

**SQLITE_DEFAULT_AUTOMATIC_INDEX=*<0 或 1>***

> 此宏确定了对于新打开的数据库连接，PRAGMA automatic_index 的初始设置。在所有 SQLite 版本中，通过 3.7.17，如果忽略了此编译时选项，新的数据库连接通常会启用自动索引。然而，这可能会在未来的 SQLite 发布版本中改变。
> 
> 另见：SQLITE_OMIT_AUTOMATIC_INDEX

**SQLITE_DEFAULT_AUTOVACUUM=*<0 或 1 或 2>***

> 此宏确定了 SQLite 是否默认创建数据库时将 auto_vacuum 标志设置为 OFF (0)、FULL (1) 或 INCREMENTAL (2)。默认值为 0，意味着创建数据库时自动清理被关闭。在任何情况下，编译时的默认值可能会被 PRAGMA auto_vacuum 命令覆盖。

**SQLITE_DEFAULT_CACHE_SIZE=*<N>***

> 这个宏设置了每个附加数据库的默认页面缓存的最大大小。正数表示限制为 N 页。如果 N 为负数，则表示将缓存大小限制为-N*1024 字节。建议的最大缓存大小可以通过 PRAGMA cache_size 命令覆盖。默认值为-2000，这意味着每个缓存最大为 2048000 字节。

**SQLITE_DEFAULT_FILE_FORMAT=*<1 或 4>***

> SQLite 在创建新数据库文件时使用的默认模式格式编号由此宏设置。这些模式格式都非常相似。格式 1 和格式 4 之间的区别在于，格式 4 理解降序索引，并且对布尔值有更紧凑的编码。
> 
> 自 SQLite 3.3.0（2006-01-10）以来的所有版本都能读取和写入 1 到 4 之间的任何模式格式。但是旧版本的 SQLite 可能无法读取大于 1 的格式。为了确保旧版本的 SQLite 能够读取和写入由新版本的 SQLite 创建的数据库文件，默认模式格式在 SQLite 3.7.9（2011-11-01）之前的版本中设为 1。从版本 3.7.10（2012-01-16）开始，默认的模式格式为 4。
> 
> 可以使用 PRAGMA legacy_file_format 命令在运行时设置新数据库的模式格式编号。

**SQLITE_DEFAULT_FILE_PERMISSIONS=*N***

> 在 Unix 下新创建的数据库文件的默认数字文件权限。如果未指定，则默认为 0644，这意味着文件对全局可读，但只能由创建者可写。

**SQLITE_DEFAULT_FOREIGN_KEYS=*<0 或 1>***

> 此宏确定新数据库连接是否默认启用或禁用外键约束的强制执行。每个数据库连接始终可以在运行时使用 foreign_keys pragma 打开和关闭外键约束的强制执行。默认情况下，外键约束的强制执行通常是关闭的，但如果设置了这个编译时参数为 1，则默认情况下外键约束的强制执行将打开。

**SQLITE_DEFAULT_MMAP_SIZE=*N***

> 这个宏设置了每个打开的数据库文件的内存映射 I/O 使用的默认限制。如果*N*为零，则默认情况下禁用内存映射 I/O。这个编译时限制和 SQLITE_MAX_MMAP_SIZE 可以在启动时通过 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE)调用或在运行时使用 mmap_size pragma 进行修改。

**SQLITE_DEFAULT_JOURNAL_SIZE_LIMIT=*<bytes>***

> 此选项设置了在持久日志模式和独占锁定模式下的回滚日志文件以及在 WAL 模式下写前日志文件的大小限制。当省略此编译时选项时，回滚日志或写前日志的大小没有上限。可以使用 journal_size_limit pragma 在运行时更改日志文件大小限制。

**SQLITE_DEFAULT_LOCKING_MODE=*<1 or 0>***

> 如果设置为 1，则默认的 locking_mode 为 EXCLUSIVE。如果省略或设置为 0，则默认的 locking_mode 为 NORMAL。

**SQLITE_DEFAULT_LOOKASIDE=*SZ,N***

> 设置 lookaside 内存分配器的默认内存池大小为每个 SZ 字节的 N 个条目。可以使用 sqlite3_config(SQLITE_CONFIG_LOOKASIDE)在启动时和/或在每个数据库连接打开时使用 sqlite3_db_config(db, SQLITE_DBCONFIG_LOOKASIDE)进行修改。

**SQLITE_DEFAULT_MEMSTATUS=*<1 or 0>***

> 此宏用于确定是否默认启用并禁用通过 SQLITE_CONFIG_MEMSTATUS 参数传递给 sqlite3_config()的功能。默认值为 1（启用了与 SQLITE_CONFIG_MEMSTATUS 相关的功能）。
> 
> 当内存使用跟踪被禁用时，sqlite3_memory_used()和 sqlite3_memory_highwater()接口，sqlite3_status64(SQLITE_STATUS_MEMORY_USED)接口以及 SQLITE_MAX_MEMORY 编译时选项都不可用。

**SQLITE_DEFAULT_PCACHE_INITSZ=*N***

> 当 SQLITE_CONFIG_PAGECACHE 配置选项未使用且页面缓存的内存来自 sqlite3_malloc()时，此宏确定页面缓存模块最初分配的页面数。此宏设置的页面数在单个分配中分配，从而减少了内存分配器的负载。

**SQLITE_DEFAULT_PAGE_SIZE=*<bytes>***

> 此宏用于设置创建数据库时使用的默认页面大小。分配的值必须是 2 的幂。默认值为 4096。编译时的默认值可以通过 PRAGMA page_size 命令在运行时进行覆盖。

**SQLITE_DEFAULT_SYNCHRONOUS=*<0-3>***

> 此宏确定了 PRAGMA synchronous 设置的默认值。如果在编译时未被覆盖，则默认设置为 2（FULL）。

**SQLITE_DEFAULT_WAL_SYNCHRONOUS=*<0-3>***

> 此宏确定在 WAL 模式 下打开的数据库文件的 PRAGMA synchronous 设置的默认值。如果在编译时未被覆盖，则此值与 SQLITE_DEFAULT_SYNCHRONOUS 相同。
> 
> 如果 SQLITE_DEFAULT_WAL_SYNCHRONOUS 与 SQLITE_DEFAULT_SYNCHRONOUS 不同，并且应用程序未使用 PRAGMA synchronous 语句修改数据库文件的同步设置，那么当数据库连接首次切换到 WAL 模式时，同步设置将会被改变为 SQLITE_DEFAULT_WAL_SYNCHRONOUS 定义的值。如果 SQLITE_DEFAULT_WAL_SYNCHRONOUS 的值在编译时未被覆盖，则它将始终与 SQLITE_DEFAULT_SYNCHRONOUS 相同，因此不会发生自动同步设置更改。

**SQLITE_DEFAULT_WAL_AUTOCHECKPOINT=*<pages>***

> 此宏设置了 WAL 自动检查点功能的默认页面计数。如果未指定，默认页面计数为 1000。

**SQLITE_DEFAULT_WORKER_THREADS=*N***

> 此宏设置了 SQLITE_LIMIT_WORKER_THREADS 参数的默认值。 SQLITE_LIMIT_WORKER_THREADS 参数设置了单个预处理语句将启动的辅助线程的最大数量，以帮助其处理查询。如果未指定，默认最大值为 0。此处设置的值不能超过 SQLITE_MAX_WORKER_THREADS。

**SQLITE_DQS=*N***

> 此宏确定了 SQLITE_DBCONFIG_DQS_DDL 和 SQLITE_DBCONFIG_DQS_DML 的默认值，这些值进而决定了 SQLite 如何处理每个双引号字符串字面量。 "DQS"名称代表"双引号字符串"。 *N*参数应为整数 0、1、2 或 3。
> 
> > | SQLITE_DQS | 允许双引号字符串 | 备注 |
> > | --- | --- | --- |
> > | 在 DDL 中 | 在 DML 中 |
> > | 3 | 是 | 是 | 默认 |
> > | 2 | 是 | 否 |   |
> > | 1 | 否 | 是 |   |
> > | 0 | 否 | 否 | 推荐 |
> > 
> 推荐设置为 0，表示在所有情况下都不允许双引号字符串。但是，默认设置为 3，以最大兼容性支持旧应用程序。

**SQLITE_EXTRA_DURABLE**

> SQLITE_EXTRA_DURABLE 编译时选项曾导致默认的 PRAGMA synchronous 设置为 EXTRA，而不是 FULL。此选项已不再支持。请改用 SQLITE_DEFAULT_SYNCHRONOUS=3。

**SQLITE_FTS3_MAX_EXPR_DEPTH=*N***

> 此宏设置搜索树的最大深度，该树对应于 FTS3 或 FTS4 全文索引中 MATCH 运算符右侧的内容。全文搜索使用递归算法，因此树的深度受限以防止使用过多的堆栈空间。默认限制为 12。该限制对于 MATCH 运算符右侧多达 4095 个搜索项是足够的，并且保持堆栈空间使用低于 2000 字节。
> 
> 对于普通的 FTS3/FTS4 查询，搜索树的深度大约是 MATCH 运算符右侧项数目的以 2 为底的对数。然而，对于 短语查询 和 邻近查询，搜索树的深度与右侧项的数目成线性关系。因此，12 的默认深度限制对于 MATCH 运算符右侧多达 4095 个普通项是足够的，但对于 11 或 12 个短语或邻近项来说则仅足够。即便如此，默认值对大多数应用程序来说已经完全足够。

**SQLITE_JSON_MAX_DEPTH=*N***

> 此宏设置 JSON 对象和数组的最大嵌套深度。默认值为 1000。
> 
> JSON SQL 函数 使用 [递归下降解析器](https://en.wikipedia.org/wiki/Recursive_descent_parser)。这意味着深度嵌套的 JSON 可能需要大量堆栈空间来解析。在堆栈空间有限的系统上，SQLite 可以编译为具有大大减少的最大 JSON 嵌套深度，以避免可能的堆栈溢出，即使是来自恶意输入。通常情况下，10 或 20 的值即足够处理最复杂的真实世界 JSON 数据。

**SQLITE_LIKE_DOESNT_MATCH_BLOBS**

> 此编译时选项会导致 LIKE 操作符在其中一个操作数为 BLOB 时始终返回 False。LIKE 的默认行为是在比较之前将 BLOB 操作数强制转换为 TEXT。
> 
> 此编译时选项使得 SQLite 在处理使用 LIKE 操作符的查询时更加高效，但会牺牲向后兼容性。然而，这种向后兼容性的破坏可能仅仅是一个技术细节。在 LIKE 处理逻辑中长期存在一个 bug（参见 [`www.sqlite.org/src/info/05f43be8fdda9f`](https://www.sqlite.org/src/info/05f43be8fdda9f)），导致对 BLOB 操作数的处理不正确，并且在近 10 年的使用中没有人注意到这个 bug。因此，对于大多数用户来说，启用此编译时选项可能是安全的，并且能够节省一点 CPU 时间在 LIKE 查询上。
> 
> 此编译时选项仅影响 SQL LIKE 操作符，并且不会影响 sqlite3_strlike() 的 C 语言接口。

**SQLITE_MAX_MEMORY=*N***

> 此选项限制了 SQLite 将从 malloc() 请求的总内存量为 *N* 字节。如果 SQLite 尝试分配新内存，使得 SQLite 持有的所有分配的总和超过 *N* 字节，将导致内存不足错误。这是一个硬上限。另请参阅 sqlite3_soft_heap_limit() 接口。
> 
> 此选项是 *总* 分配内存量的限制。参见 SQLITE_MAX_ALLOCATION_SIZE 选项，用于限制任何单个内存分配允许的内存量。
> 
> 只有在通过 sqlite3_memory_used()和 sqlite3_status64(SQLITE_STATUS_MEMORY_USED)接口可用的内存使用统计信息时，此限制才能发挥作用。如果没有该内存使用信息，SQLite 就无法知道何时即将超过限制，从而无法阻止过度的内存分配。默认情况下，内存使用跟踪已打开，但可以在编译时使用 SQLITE_DEFAULT_MEMSTATUS 选项或在启动时使用 sqlite3_config(SQLITE_CONFIG_MEMSTATUS)来禁用。

**SQLITE_MAX_MMAP_SIZE=*N***

> 此宏设置了对内存映射 I/O 使用的地址空间的硬性上限。将此值设置为 0 将完全禁用内存映射 I/O，并导致与内存映射 I/O 相关的逻辑在构建过程中被省略。只要其他设置的值小于此处定义的最大值，此选项就不会改变默认的内存映射 I/O 地址空间大小（由 SQLITE_DEFAULT_MMAP_SIZE 或 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE)设置）或运行时的内存映射 I/O 地址空间大小（由 sqlite3_file_control(SQLITE_FCNTL_MMAP_SIZE)或 PRAGMA mmap_size)。

**SQLITE_MAX_SCHEMA_RETRY=*N***

> 每当数据库模式发生变化时，预编译语句会自动重新编译以适应新模式。这里存在竞争条件，如果一个线程不断更改模式，另一个线程可能会在预编译和重新编译预编译语句上旋转，而无法完成任何真正的工作。这个参数通过强制旋转线程在尝试重新编译预编译语句的固定次数后放弃，防止了无限循环。默认设置为 50，对大多数应用程序来说完全足够。

**SQLITE_MAX_WORKER_THREADS=*N***

> 设置一个上限，用于限制单个预编译语句在 CPU 密集型计算（主要是排序）中使用的最大辅助线程数，该设置由 sqlite3_limit(db,SQLITE_LIMIT_WORKER_THREADS,N)确定。还请参阅 SQLITE_DEFAULT_WORKER_THREADS 选项。

**SQLITE_MEMDB_DEFAULT_MAXSIZE=*N***

> 设置使用 sqlite3_deserialize()创建的内存数据库的默认大小限制（以字节为单位）。这只是默认值。可以在启动时使用 sqlite3_config(SQLITE_CONFIG_MEMDB_MAXSIZE,N)或在运行时为单个数据库使用 SQLITE_FCNTL_SIZE_LIMIT file-control 来更改限制。如果未指定默认值，则使用 1073741824。

**SQLITE_MINIMUM_FILE_DESCRIPTOR=*N***

> Unix VFS 永远不会使用小于*N*的文件描述符。*N*的默认值为 3。
> 
> 避免使用低编号文件描述符可以防止意外数据库损坏。例如，如果使用文件描述符 2 打开数据库文件，然后 assert() 失败并调用 write(2,...)，那么很可能会因为用断言错误消息覆盖数据库文件的一部分而导致数据库损坏。只使用较高值的文件描述符可以避免这种潜在问题。禁用防止使用低编号文件描述符的保护可以通过将编译时选项设置为 0 来实现。

**SQLITE_POWERSAFE_OVERWRITE=*<0 或 1>* **

> 此选项更改了关于 powersafe overwrite 在 unix 和 windows VFSes 下的默认假设。将 SQLITE_POWERSAFE_OVERWRITE 设置为 1 会使 SQLite 假设应用级别的写入不会更改写入字节范围之外的字节，即使写入发生在断电之前。如果将 SQLITE_POWERSAFE_OVERWRITE 设置为 0，则 SQLite 假设同一个扇区中其他字节可能会因断电而被更改或损坏。

**SQLITE_PRINTF_PRECISION_LIMIT=*N***

> 此选项限制了对 printf() SQL function 和其他 C 语言字符串格式化函数（如 sqlite3_mprintf() 和 sqlite3_str_appendf()）的最大宽度和精度。这可以防止恶意或故障脚本通过调用如 "`printf('%*s',2147483647,'hi')`" 这样的格式化字符串而导致内存使用过多。通常情况下，*N* 的值设为约 100000 就足够了。
> 
> printf() SQL 函数受 SQLITE_LIMIT_LENGTH 限制，该限制由 sqlite3_limit()决定。因此，任何 printf()结果的宽度或精度超过 SQLITE_LIMIT_LENGTH 都会导致 SQLITE_TOOBIG 错误。但是，printf()函数的底层格式化是由一个子程序完成的，该子程序无法访问 SQLITE_LIMIT_LENGTH。因此，底层格式化会在一个可能比 SQLITE_LIMIT_LENGTH 大得多的内存分配中完成，并且只有在所有格式化完成后才会执行 SQLITE_LIMIT_LENGTH 检查。因此可能存在超过 SQLITE_LIMIT_LENGTH 的临时缓冲区。SQLITE_PRINTF_PRECISION_LIMIT 选项是在 SQLITE_LIMIT_LENGTH 检查之前对底层格式化子程序内使用的临时缓冲区进行额外检查的一种方式。
> 
> 注意不要将 SQLITE_PRINTF_PRECISION_LIMIT 设置得太低。SQLite 使用其内置的 printf()功能来格式化存储在 sqlite_schema 表中的 CREATE 语句文本。因此，SQLITE_PRINTF_PRECISION_LIMIT 应至少与您可能遇到的最大表、索引、视图或触发器定义大小相同。
> 
> 如果宽度或精度超过 SQLITE_PRINTF_PRECISION_LIMIT，不会引发错误。相反，大宽度或精度会被静默截断。
> 
> SQLITE_PRINTF_PRECISION_LIMIT 的默认值为 2147483647（0x7fffffff）。

**SQLITE_QUERY_PLANNER_LIMIT=*N***

> 在查询规划过程中，SQLite 会枚举所有可用的索引和 WHERE 子句约束的组合。对于某些病态查询，这些索引和约束组合的数量可能非常大，导致查询规划器性能较慢。SQLITE_QUERY_PLANNER_LIMIT 值（与相关的 SQLITE_QUERY_PLANNER_LIMIT_INCR 设置一起）限制了查询规划器将考虑的索引和约束组合的数量，以防止查询规划器使用过多的 CPU 时间。SQLITE_QUERY_PLANNER_LIMIT 的默认值设置得足够高，以确保在实际查询中永远不会达到。查询规划器的搜索限制仅适用于故意设计以使用过多规划时间的查询。

**SQLITE_QUERY_PLANNER_LIMIT_INCR=*N***

> SQLITE_QUERY_PLANNER_LIMIT 选项设定了查询规划器在考虑索引和约束组合时的初始基线值。在处理联接的每个表之前，基线查询规划器限制会通过 SQLITE_QUERY_PLANNER_LIMIT_INCR 递增，确保每个表至少能够向优化器提议一些索引和约束组合，即使联接中的先前表已经耗尽了基线限制。此编译时选项和 SQLITE_QUERY_PLANNER_LIMIT 选项的默认值设置得足够高，以确保它们在实际查询中不会被达到。

**SQLITE_REVERSE_UNORDERED_SELECTS**

> 此选项会默认启用 PRAGMA reverse_unordered_selects 设置。当启用时，缺少 ORDER BY 子句的 SELECT 语句将按照相反的顺序执行。
> 
> 此选项可用于检测应用程序（错误地）假设 SELECT 语句中没有 ORDER BY 子句的行的顺序总是相同。

**SQLITE_SORTER_PMASZ=*N***

> 如果通过 PRAGMA threads 设置启用了多线程处理，则在要排序的内容超过 cache_size 和由 SQLITE_CONFIG_PMASZ 启动时选项确定的 PMA 大小的最小值时，排序操作将尝试启动辅助线程。此编译时选项设置了 SQLITE_CONFIG_PMASZ 起始时选项的默认值。默认值为 250。

**SQLITE_STMTJRNL_SPILL=*N***

> SQLITE_STMTJRNL_SPILL 编译时选项确定了 SQLITE_CONFIG_STMTJRNL_SPILL 起始时设置的默认值。该设置决定了内存中语句日志移动到磁盘的大小阈值。

**SQLITE_WIN32_MALLOC**

> 此选项启用了使用 Windows Heap API 函数进行内存分配而不是标准库 malloc()和 free()例程。

**YYSTACKDEPTH=*<max_depth>***

> 此宏设置了 SQLite 中 SQL 解析器使用的 LALR(1)栈的最大深度。默认值为 100。典型应用程序将使用的栈深度不到 20。包含需要超过 100 个 LALR(1)栈条目的 SQL 语句的开发人员应认真考虑重构他们的 SQL，因为它很可能远远超出任何人类理解的能力。

# 5\. 设置大小限制的选项

编译时选项将设置 SQLite 中各种结构的大小上限。这些编译时选项通常设置一个硬性上限，可以在运行时通过单独的数据库连接使用 sqlite3_limit()接口进行更改。

编译时设置上限的选项被单独记录。以下是可用设置的列表：

+   SQLITE_MAX_ATTACHED

+   SQLITE_MAX_COLUMN

+   SQLITE_MAX_COMPOUND_SELECT

+   SQLITE_MAX_EXPR_DEPTH

+   SQLITE_MAX_FUNCTION_ARG

+   SQLITE_MAX_LENGTH

+   SQLITE_MAX_LIKE_PATTERN_LENGTH

+   SQLITE_MAX_PAGE_COUNT

+   SQLITE_MAX_SQL_LENGTH

+   SQLITE_MAX_VARIABLE_NUMBER

还有一些大小限制，无法使用 sqlite3_limit()进行修改。例如：

+   SQLITE_FTS3_MAX_EXPR_DEPTH

+   SQLITE_JSON_MAX_DEPTH

+   SQLITE_MAX_ALLOCATION_SIZE

+   SQLITE_MAX_MEMORY

+   SQLITE_MAX_MMAP_SIZE

+   SQLITE_PRINTF_PRECISION_LIMIT

+   SQLITE_TRACE_SIZE_LIMIT

+   YYSTACKDEPTH

# 6\. 控制操作特性的选项

**SQLITE_4_BYTE_ALIGNED_MALLOC**

> 在大多数系统上，malloc()系统调用返回的缓冲区对齐到 8 字节边界。但是在一些系统上（例如：Windows），malloc()返回 4 字节对齐的指针。这个编译时选项必须在从 malloc()返回 4 字节对齐指针的系统上使用。

**SQLITE_CASE_SENSITIVE_LIKE**

> 如果存在这个选项，则内置的 LIKE 运算符将区分大小写。可以在运行时使用 case_sensitive_like pragma 实现相同效果。

**SQLITE_DIRECT_OVERFLOW_READ**

> 当存在此选项时，在读取事务期间，直接从磁盘读取数据库文件中溢出页面中的内容，绕过页面缓存。在执行大量读取大型 BLOB 或字符串的应用程序中，此选项可提高读取性能。
> 
> 从版本 3.45.0（2024-01-15）开始，默认情况下启用了此选项。要禁用它，请使用 -DSQLITE_DIRECT_OVERFLOW_READ=0。

**SQLITE_HAVE_ISNAN**

> 如果存在这个选项，则 SQLite 将使用系统数学库中的 isnan()函数。这是 HAVE_ISNAN 配置选项的别名。

**SQLITE_MAX_ALLOCATION_SIZE=*N***

> 这个编译时选项设置了可以使用 sqlite3_malloc64()、sqlite3_realloc64()等请求的内存分配大小的上限。默认值为 2,147,483,391（0x7ffffeff），应将其视为上限。大多数应用程序可以通过几百万字节的最大分配大小完成。
> 
> 这是任何单个内存分配的最大大小限制。这*不是*总内存分配量的限制。有关总内存分配量的限制，请参阅 SQLITE_MAX_MEMORY。
> 
> 减少单个内存分配的最大大小可提供额外的防御，防止拒绝服务攻击试图通过执行许多大内存分配来耗尽系统内存。这也是对应用程序缺陷的额外防御层，其中内存分配的大小计算使用有符号 32 位整数，可能会溢出 → 通过较小的最大分配大小，这种错误的内存分配大小计算很可能会因内存不足错误而更早地被发现，而不是在整数实际溢出之前。

**SQLITE_OS_OTHER=*<0 或 1>***

> 该选项导致 SQLite 省略其针对 Unix、Windows 和 OS/2 的内置操作系统接口。生成的库将没有默认的操作系统接口。应用程序必须在使用 SQLite 之前使用 sqlite3_vfs_register()注册适当的接口。应用程序还必须为 sqlite3_os_init()和 sqlite3_os_end()接口提供实现。通常的做法是，提供的 sqlite3_os_init()调用 sqlite3_vfs_register()。SQLite 初始化时会自动调用 sqlite3_os_init()。
> 
> 当在构建具有自定义操作系统的嵌入式平台时，通常会使用此选项。

**SQLITE_SECURE_DELETE**

> 此编译时选项更改了 secure_delete pragma 的默认设置。如果未使用此选项，则 secure_delete 默认为关闭。如果存在此选项，则 secure_delete 默认为打开。
> 
> secure_delete 设置导致删除的内容被用零覆盖。由于需要额外的 I/O 操作，这会带来一些性能损耗。但另一方面，secure_delete 可以防止敏感信息的片段在数据库文件的未使用部分残留。更多信息请参阅关于 secure_delete pragma 的文档。

**SQLITE_THREADSAFE=*<0 or 1 or 2>***

> 此选项控制 SQLite 是否包含代码以确保在多线程环境中安全运行。默认值是 SQLITE_THREADSAFE=1，适合在多线程环境中使用。当编译时设为 SQLITE_THREADSAFE=0 时，所有互斥代码将被省略，因此在多线程程序中使用 SQLite 是不安全的。当编译时设为 SQLITE_THREADSAFE=2 时，SQLite 可以在多线程程序中使用，只要没有两个线程同时尝试使用相同的 数据库连接（或从该数据库连接派生的任何 prepared statements）。
> 
> 换句话说，SQLITE_THREADSAFE=1 将默认的 线程模式 设置为 Serialized。SQLITE_THREADSAFE=2 将默认的 线程模式 设置为 Multi-threaded。而 SQLITE_THREADSAFE=0 则将 线程模式 设置为 Single-threaded。
> 
> 可以使用 sqlite3_threadsafe() 接口在运行时确定 SQLITE_THREADSAFE 的值。
> 
> 当 SQLite 编译为 SQLITE_THREADSAFE=1 或 SQLITE_THREADSAFE=2 时，可以使用 sqlite3_config() 接口以及以下动词来在运行时更改 线程模式。
> 
> +   SQLITE_CONFIG_SINGLETHREAD
> +   
> +   SQLITE_CONFIG_MULTITHREAD
> +   
> +   SQLITE_CONFIG_SERIALIZED
> +   
> sqlite3_open_v2()的 SQLITE_OPEN_NOMUTEX 和 SQLITE_OPEN_FULLMUTEX 标志也可用于调整运行时个别数据库连接的线程模式。
> 
> 注意，当 SQLite 编译时使用 SQLITE_THREADSAFE=0，会从构建中省略使 SQLite 线程安全的代码。当出现这种情况时，不可能在启动时或运行时更改线程模式。
> 
> 查看线程模式文档，了解在多线程环境中使用 SQLite 的其他信息。

**SQLITE_TEMP_STORE=*<0 到 3>***

> 此选项控制临时文件是存储在磁盘上还是内存中。此编译时选项的各种设置含义如下：
> 
> | SQLITE_TEMP_STORE | 含义 |
> | --- | --- |
> | 0 | 始终使用临时文件 |
> | 1 | 默认使用文件，但允许使用 PRAGMA temp_store 命令进行覆盖 |
> | 2 | 默认使用内存，但允许使用 PRAGMA temp_store 命令进行覆盖 |
> | 3 | 始终使用内存 |
> 
> 默认设置为 1。可以在 tempfiles.html 中找到更多信息。

**SQLITE_TRACE_SIZE_LIMIT=*N***

> 如果将此宏定义为正整数 *N*，则在 sqlite3_trace()输出中扩展为参数的字符串和 BLOB 的长度将限制为 *N* 字节。

**SQLITE_TRUSTED_SCHEMA=*<0 或 1>***

> 此宏确定了 SQLITE_DBCONFIG_TRUSTED_SCHEMA 和 PRAGMA trusted_schema 设置的默认值。如果未指定替代方案，则受信模式设置默认为 ON（值为 1），以保持与旧版兼容性。然而，为了最佳安全性，实现 应用程序定义的 SQL 函数 和/或 虚拟表 的系统应考虑将默认设置为 OFF。

**SQLITE_USE_URI**

> 此选项导致默认启用 URI 文件名 处理逻辑。

# 7\. 启用通常关闭的功能选项

**SQLITE_ALLOW_URI_AUTHORITY**

> URI 文件名 如果其权限部分不为空或者为 "localhost"，通常会抛出错误。但是，如果 SQLite 在编译时使用了 SQLITE_ALLOW_URI_AUTHORITY 选项，则 URI 会被转换为统一命名约定（UNC）文件名，并以这种方式传递给底层操作系统。
> 
> 一些未来版本的 SQLite 可能会更改，以默认启用此功能。

**SQLITE_ALLOW_COVERING_INDEX_SCAN=*<0 或 1>***

> 此 C 预处理宏确定了 SQLITE_CONFIG_COVERING_INDEX_SCAN 配置设置的默认值。它默认为 1（开启），这意味着在可能的情况下使用覆盖索引来进行全表扫描，以减少 I/O 并提高性能。然而，对于全扫描使用覆盖索引将导致结果与传统方式不同，这可能导致一些（编码不正确的）传统应用程序出现问题。因此，在希望最小化在传统应用程序中暴露错误风险的系统上，可以在编译时禁用覆盖索引扫描选项。

**SQLITE_ENABLE_8_3_NAMES=*<1 或 2>***

> 如果定义了此 C 预处理宏，则包含额外的代码，允许 SQLite 在仅支持 8+3 文件名的文件系统上运行。如果此宏的值为 1，则默认行为是继续使用长文件名，并仅在使用带有"`8_3_names=1`"查询参数的 URI 文件名打开数据库连接时才使用 8+3 文件名。如果此宏的值为 2，则使用 8+3 文件名成为默认行为，但可以通过使用`8_3_names=0`查询参数来禁用。

**SQLITE_ENABLE_API_ARMOR**

> 当定义了此 C 预处理宏时，将激活额外的代码，用于检测 SQLite API 的误用，例如将 NULL 指针传递给必需的参数或在对象被销毁后继续使用。启用此选项后，如果检测到非法的 API 使用，接口通常会返回 SQLITE_MISUSE。
> 
> SQLITE_ENABLE_API_ARMOR 选项并不能保证检测到所有的非法 API 使用。即使启用了 SQLITE_ENABLE_API_ARMOR，向 C 语言 API 传递不正确的值仍可能导致进程崩溃，如段错误或空指针解引用等其他原因。SQLITE_ENABLE_API_ARMOR 编译时选项旨在帮助应用程序测试和调试。应用程序不应依赖 SQLITE_ENABLE_API_ARMOR 来确保安全性。SQLITE_ENABLE_API_ARMOR 适合作为应用程序错误的第二道防线，但不应是唯一的防线。如果任何 SQLite 接口返回 SQLITE_MISUSE，则表示应用程序使用 SQLite 与规范相违背，并且应用程序存在错误。SQLITE_MISUSE 的返回提供了应用程序处理该错误的机会，而不是简单地使进程崩溃或调用未定义行为，但仅此而已。应用程序在常规处理中既不应使用也不依赖 SQLITE_MISUSE。

**SQLITE_ENABLE_ATOMIC_WRITE**

> 如果定义了此 C 预处理宏，并且如果数据库文件的 sqlite3_io_methods 对象的 xDeviceCharacteristics 方法（通过 SQLITE_IOCAP_ATOMIC 位之一）报告文件系统支持原子写入，并且如果事务仅涉及对数据库文件的单个页面的更改，则事务将通过仅一个单页数据库的单个写请求提交，并且不会创建或写入回滚日志。在支持原子写入的文件系统上，这种优化可以显著提高小更新的速度。然而，很少有文件系统支持此功能，并且检查此功能的代码路径会减慢在缺乏原子写入能力的系统上的写入性能，因此此功能默认情况下被禁用。

**SQLITE_ENABLE_BATCH_ATOMIC_WRITE**

> 此编译时选项使 SQLite 能够利用底层文件系统中的批量原子写入能力。从 SQLite 版本 3.21.0（2017-10-24）开始，此功能仅在[F2FS](https://en.wikipedia.org/wiki/F2FS)上受支持。然而，接口是通用实现的，使用 sqlite3_file_control()与 SQLITE_FCNTL_BEGIN_ATOMIC_WRITE 和 SQLITE_FCNTL_COMMIT_ATOMIC_WRITE，因此未来可以将此功能添加到其他文件系统中。当启用此选项时，SQLite 会自动检测底层文件系统是否支持批量原子写入，并且在这样做时避免写入用于事务控制的回滚日志。这可以使事务速度提高两倍以上，同时减少 SSD 存储设备的磨损。
> 
> SQLite 的未来版本可能会默认启用批量原子写入功能，届时此编译时选项将变得多余。

**SQLITE_ENABLE_BYTECODE_VTAB**

> 此选项启用 bytecode 和 tables_used 虚拟表。

**SQLITE_ENABLE_COLUMN_METADATA**

> 当定义了这个 C 预处理宏时，SQLite 会包含一些额外的 API，这些 API 方便地提供了关于表和查询的元数据访问。此选项启用的 API 包括：
> 
> +   sqlite3_column_database_name()
> +   
> +   sqlite3_column_database_name16()
> +   
> +   sqlite3_column_table_name()
> +   
> +   sqlite3_column_table_name16()
> +   
> +   sqlite3_column_origin_name()
> +   
> +   sqlite3_column_origin_name16()

**SQLITE_ENABLE_DBPAGE_VTAB**

> 此选项启用 SQLITE_DBPAGE 虚拟表。

**SQLITE_ENABLE_DBSTAT_VTAB**

> 此选项启用 dbstat 虚拟表。

**SQLITE_ENABLE_DESERIALIZE**

> 这个选项曾经用于启用 sqlite3_serialize()和 sqlite3_deserialize()接口。然而，从 SQLite 3.36.0（2021-06-18）开始，默认启用这些接口，并添加了一个新的编译时选项 SQLITE_OMIT_DESERIALIZE 来省略它们。

**SQLITE_ENABLE_EXPLAIN_COMMENTS**

> 此选项向 SQLite 添加额外的逻辑，将注释文本插入 EXPLAIN 输出中。这些额外的注释会占用额外的内存，因此会使预编译语句变得更大，稍微变慢，所以默认情况下大多数应用会关闭它们。但是一些应用程序，比如 SQLite 的命令行 shell，更注重 EXPLAIN 输出的清晰度，因此可以通过此编译时选项启用。如果启用了 SQLITE_DEBUG，则会自动启用 SQLITE_ENABLE_EXPLAIN_COMMENTS 编译时选项。

**SQLITE_ENABLE_FTS3**

> 当此选项在合并中定义时，全文搜索引擎的第 3 版和第 4 版将自动添加到构建中。

**SQLITE_ENABLE_FTS3_PARENTHESIS**

> 此选项修改了 FTS3 中的查询模式解析器，使其支持 AND 和 NOT 操作符（除了通常的 OR 和 NEAR），并允许查询表达式包含嵌套括号。

**SQLITE_ENABLE_FTS3_TOKENIZER**

> 此选项启用了 fts3_tokenizer()接口的双参数版本。fts3_tokenizer()的第二个参数应该是一个指向实现应用程序定义的分词器的函数（以 BLOB 编码）。如果恶意行为者能够使用任意的第二个参数运行 fts3_tokenizer()的双参数版本，则可能导致崩溃或接管进程。
> 
> 出于安全考虑，除非使用此编译时选项，否则从版本 3.11.0（2016-02-15）开始，将禁用 fts3_tokenizer()的双参数功能。版本 3.12.0（2016-03-29）添加了 sqlite3_db_config(db,SQLITE_DBCONFIG_ENABLE_FTS3_TOKENIZER,1,0)接口，用于在运行时激活 fts3_tokenizer()的双参数版本，适用于特定的数据库连接。

**SQLITE_ENABLE_FTS4**

> 当此选项在合并中定义时，全文搜索引擎的第 3 版和第 4 版将自动添加到构建中。

**SQLITE_ENABLE_FTS5**

> 当此选项在合并中定义时，全文搜索引擎的第 5 版（fts5）将自动添加到构建中。

**SQLITE_ENABLE_GEOPOLY**

> 当此选项在合并中定义时，Geopoly 扩展将包含在构建中。

**SQLITE_ENABLE_HIDDEN_COLUMNS**

> 当在合并中定义此选项时，虚拟表启用了隐藏列功能。

**SQLITE_ENABLE_ICU**

> 此选项导致向 SQLite 中添加[国际组件](https://icu.unicode.org)或 "ICU" 扩展。

**SQLITE_ENABLE_IOTRACE**

> 当 SQLite 核心和命令行界面（CLI）同时使用此选项编译时，CLI 提供一个名为 ".iotrace" 的额外命令，用于提供 I/O 活动的低级日志。此选项是实验性的，可能在将来的版本中停止支持。

**SQLITE_ENABLE_MATH_FUNCTIONS**

> 此宏启用了内置 SQL 数学函数。此选项会自动添加到 Unix 平台上的 Makefile 中的 configure 脚本中，除非使用 --disable-math 选项。在使用 "Makefile.msc" 为 nmake 构建的 Windows 构建中也包括此选项。

**SQLITE_ENABLE_JSON1**

> 这个编译时选项是一个无操作。在 SQLite 版本 3.38.0（2022-02-22）之前，需要编译这个选项才能在构建中包含 JSON SQL 函数。然而，从 SQLite 版本 3.38.0 开始，这些函数默认包含在内。使用 -DSQLITE_OMIT_JSON 选项来排除它们。

**SQLITE_ENABLE_LOCKING_STYLE**

> 此选项在 Mac OS X 的 OS 接口层启用了额外的逻辑。额外的逻辑试图确定底层文件系统的类型，并选择适当的锁定策略，以确保该文件系统类型正常工作。有五种锁定策略可用：
> 
> +   POSIX 锁定样式。这是默认的锁定样式，也是其他（非 Mac OS X）Unix 使用的样式。使用 fcntl() 系统调用来获取和释放锁。
> +   
> +   AFP 锁定样式。此锁定样式用于使用 AFP（苹果文件共享协议）协议的网络文件系统。通过调用库函数 _AFPFSSetLock()获取锁定。
> +   
> +   Flock 锁定样式。此样式用于不支持 POSIX 锁定样式的文件系统。使用 flock()系统调用获取和释放锁定。
> +   
> +   点文件锁定样式。当文件系统既不支持 flock 也不支持 POSIX 锁定样式时，将使用此锁定样式。数据库锁通过在与数据库文件相关的已知位置（一个“点文件”）创建条目来获取，并通过删除相同文件来释放。
> +   
> +   无锁定样式。如果以上任何一种都无法支持，则使用此锁定样式。不使用任何数据库锁定机制。当使用此系统时，单个数据库不安全地可以由多个客户端访问。
> +   
> 此外，还提供了五种额外的 VFS 实现，以及默认的实现。在调用 sqlite3_open_v2()时，可以指定其中一种额外的 VFS 实现，应用程序可以绕过文件系统检测逻辑，显式选择上述五种锁定样式之一。这五种额外的 VFS 实现分别称为"unix-posix"、"unix-afp"、"unix-flock"、"unix-dotfile"和"unix-none"。

**SQLITE_ENABLE_MEMORY_MANAGEMENT**

> 此选项增加了 SQLite 的额外逻辑，允许其在请求时释放未使用的内存。必须启用此编译时选项才能使 sqlite3_release_memory()接口工作。如果未使用此编译时选项，则 sqlite3_release_memory()接口将是一个空操作。

**SQLITE_ENABLE_MEMSYS3**

> 此选项包含了在 SQLite 中实现替代内存分配器的代码。当使用 sqlite3_config()中的 SQLITE_CONFIG_HEAP 选项提供一个大内存块作为所有内存分配的来源时，才会启用这个替代内存分配器。MEMSYS3 内存分配器使用了一种混合分配算法，模仿了 dlmalloc()。SQLITE_ENABLE_MEMSYS3 和 SQLITE_ENABLE_MEMSYS5 只能同时启用一个。

**SQLITE_ENABLE_MEMSYS5**

> 此选项包含了在 SQLite 中实现替代内存分配器的代码。当使用 sqlite3_config()中的 SQLITE_CONFIG_HEAP 选项提供一个大内存块作为所有内存分配的来源时，才会启用这个替代内存分配器。MEMSYS5 模块将所有分配都舍入到最接近的二的幂，并使用首次适配、伙伴分配算法，以在某些操作限制条件下提供强大的防碎片化和崩溃保证。

**SQLITE_ENABLE_NORMALIZE**

> 此选项包含了 sqlite3_normalized_sql() API。

**SQLITE_ENABLE_NULL_TRIM**

> 此选项启用了一种优化，可以在磁盘上节省空间，省略行末尾的 NULL 列。
> 
> 使用此选项生成的数据库无法被 SQLite 版本 3.1.6（2005-03-17）及更早的版本读取。此外，使用此选项生成的数据库可能会触发 sqlite3_blob_reopen()接口中的[e6e962d6b0f06f46](https://www.sqlite.org/src/info/e6e962d6b0f06f46e)漏洞。因此，默认情况下禁用此优化。不过，未来的 SQLite 版本可能会默认启用此优化。

**SQLITE_ENABLE_OFFSET_SQL_FUNC**

> 此选项启用了 sqlite_offset(X) SQL 函数的支持。
> 
> sqlite_offset(X) SQL 函数需要 B-tree 存储引擎上的新接口，在运行 SQL 语句的虚拟机中需要一个新的操作码，并且在代码生成器的关键路径中需要一个新的条件。为了避免在不需要 sqlite_offset(X)的应用程序中出现的开销，该函数默认情况下被禁用。

**SQLITE_ENABLE_PREUPDATE_HOOK**

> 此选项启用了几个新的 API，它们在对行 ID 表进行任何更改之前提供回调。这些回调可用于记录变更发生之前行的状态。
> 
> Preupdate 钩子的操作类似于更新钩子，但是回调是在变更之前调用的，而不是之后，并且只有在使用了这个编译时选项时才会提供 preupdate 钩子接口。
> 
> 最初添加 preupdate 钩子接口是为了支持 session 扩展。

**SQLITE_ENABLE_QPSG**

> 此选项导致查询规划器稳定性保证(QPSG)默认启用。通常情况下，QPSG 处于关闭状态，必须在运行时使用 SQLITE_DBCONFIG_ENABLE_QPSG 选项通过 sqlite3_db_config()接口激活。

**SQLITE_ENABLE_RBU**

> 启用实现 RBU 扩展的代码。

**SQLITE_ENABLE_RTREE**

> 此选项导致 SQLite 包含对 R*Tree 索引扩展的支持。

**SQLITE_ENABLE_SESSION**

> 此选项启用了 session 扩展。

**SQLITE_ENABLE_SNAPSHOT**

> 此选项启用了支持 sqlite3_snapshot 对象及其相关接口的代码：
> 
> +   sqlite3_snapshot_get() (构造函数)
> +   
> +   sqlite3_snapshot_free()（析构函数）
> +   
> +   sqlite3_snapshot_open()
> +   
> +   sqlite3_snapshot_cmp()
> +   
> +   sqlite3_snapshot_recover()

**SQLITE_ENABLE_SORTER_REFERENCES**

> 此选项激活了一种优化，减少了排序器所需的内存，但会在排序后进行额外的 B 树查找。
> 
> 默认的排序过程是将最终输出的所有信息聚集到一个“记录”中，并将完整的记录传递给排序器。但在某些情况下，例如如果一些输出列包含大的 BLOB 值，每个记录的大小可能会很大，这意味着排序器必须使用更多的内存和/或将更多内容写入临时存储。
> 
> 当启用 SQLITE_ENABLE_SORTER_REFERENCES 时，传递给排序器的记录通常只包含 ROWID 值。这样的记录要小得多。这意味着排序器处理的“有效载荷”大大减少，因此可以运行得更快。排序完成后，ROWID 用于在原始表中查找输出列的值。这需要再次搜索表，可能导致减速，也可能提升性能，这取决于值的大小。
> 
> 即使在编译时启用了 SQLITE_ENABLE_SORTER_REFERENCES 选项，排序器引用仍然默认禁用。要使用排序器引用，应用程序必须在启动时使用 sqlite3_config(SQLITE_CONFIG_SORTERREF_SIZE) 接口设置排序器引用大小阈值。
> 
> 因为 SQLite 开发人员不知道 SQLITE_ENABLE_SORTER_REFERENCES 选项是否有助于或损害性能，因此当前（2018-05-04）默认情况下禁用。根据对其对性能影响的了解，它可能在未来的某个版本中默认启用。

**SQLITE_ENABLE_STMT_SCANSTATUS**

> 此选项启用 sqlite3_stmt_scanstatus()和 sqlite3_stmt_scanstatus_v2()接口。这些接口通常会被从构建中省略，因为它们会对即使不使用该功能的语句也会施加性能惩罚。

**SQLITE_ENABLE_STMTVTAB**

> 此编译时选项启用 SQLITE_STMT 虚拟表逻辑。

**SQLITE_RTREE_INT_ONLY**

> 此编译时选项已被弃用且未经测试。

**SQLITE_ENABLE_SQLLOG**

> 此选项启用额外的代码（尤其是 SQLITE_CONFIG_SQLLOG 选项），可用于创建应用程序执行的所有 SQLite 处理的日志。这些日志对于离线分析应用程序行为特别有用，尤其是用于性能分析。为了使 SQLITE_ENABLE_SQLLOG 选项有用，需要一些额外的代码。SQLite 源代码树中的["test_sqllog.c"](https://www.sqlite.org/src/doc/trunk/src/test_sqllog.c)源代码文件是所需额外代码的工作示例。在 Unix 和 Windows 系统上，开发人员可以将"test_sqllog.c"源代码文件的文本附加到"sqlite3.c"合并文件的末尾，使用-DSQLITE_ENABLE_SQLLOG 选项重新编译应用程序，然后使用环境变量控制日志记录。有关更多详细信息，请参阅"test_sqllog.c"源文件的标题注释。

**SQLITE_ENABLE_STAT2**

> 此选项曾经导致 ANALYZE 命令在**sqlite_stat2**表中收集索引直方图数据。但此功能在 SQLite 版本 3.7.9（2011-11-01）以后被 SQLITE_ENABLE_STAT3 所取代。现在，SQLITE_ENABLE_STAT2 编译时选项已经是无效操作。

**SQLITE_ENABLE_STAT3**

> 此选项曾经导致 ANALYZE 命令在**sqlite_stat3**表中收集索引直方图数据。但此功能在 SQLite 版本 3.8.1（2013-10-17）以后被 SQLITE_ENABLE_STAT4 所取代。尽管 SQLITE_ENABLE_STAT3 编译时选项在版本 3.29.0（2019-07-10）之前仍然得到支持，但现在已经是无效操作。

**SQLITE_ENABLE_STAT4**

> 此选项为 ANALYZE 命令和查询规划器增加了额外的逻辑，有助于 SQLite 在特定情况下选择更好的查询计划。ANALYZE 命令被改进以从每个索引的所有列收集直方图数据，并将这些数据存储在 sqlite_stat4 表中。查询规划器随后将使用直方图数据来帮助它做出更好的索引选择。这个编译时选项的缺点是违反了查询规划器稳定性保证，使得在大规模生产应用中确保一致的性能变得更加困难。
> 
> SQLITE_ENABLE_STAT4 是 SQLITE_ENABLE_STAT3 的增强版本。STAT3 仅为每个索引的最左列记录直方图数据，而 STAT4 增强版记录每个索引的所有列的直方图数据。SQLITE_ENABLE_STAT3 编译时选项现在已经是无效操作。

**SQLITE_ENABLE_TREE_EXPLAIN**

> 此编译时选项已不再使用。

**SQLITE_ENABLE_UPDATE_DELETE_LIMIT**

> 此选项允许在 UPDATE 和 DELETE 语句中使用可选的 ORDER BY 和 LIMIT 子句。
> 
> 如果定义了此选项，则在使用 Lemon 解析器生成器生成 parse.c 文件时也必须定义。因此，此选项仅在从源代码构建库时可用，而不适用于从合并版本或网站上为非类 Unix 平台提供的预打包的 C 文件集合构建时使用。

**SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION**

> 当启用了编译时选项 SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION 时，在运行 EXPLAIN 或 EXPLAIN QUERY PLAN 时，SQLite 将抑制“未知函数”错误。在抛出错误之前，SQLite 会插入一个名为“unknown()”的替代空操作函数。仅在 EXPLAIN 和 EXPLAIN QUERY PLAN 上才会用“unknown()”替换未识别的函数，而在普通语句中不会发生。
> 
> 当在命令行 shell 中使用时，SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION 功能允许将包含应用程序定义函数的 SQL 文本粘贴到 shell 中进行分析和调试，而无需创建并加载实现应用程序定义函数的扩展。

**SQLITE_ENABLE_UNLOCK_NOTIFY**

> 此选项启用了 sqlite3_unlock_notify()接口及其相关功能。请参阅标题为 Using the SQLite Unlock Notification Feature 的文档获取更多信息。

**SQLITE_INTROSPECTION_PRAGMAS**

> 此选项已过时。曾经启用了一些额外的 PRAGMA 语句，如 PRAGMA function_list，PRAGMA module_list 和 PRAGMA pragma_list，但这些 PRAGMA 现在都默认启用。参见 SQLITE_OMIT_INTROSPECTION_PRAGMAS。

**SQLITE_SOUNDEX**

> 此选项启用了 soundex() SQL 函数。

**SQLITE_STRICT_SUBTYPE=1**

> 此选项导致应用定义的 SQL 函数在调用 sqlite3_result_subtype()接口时引发 SQL 错误，但未使用 SQLITE_RESULT_SUBTYPE 属性注册。这个推荐选项有助于在开发周期的早期识别应用程序定义的 SQL 函数实现中的问题。

**SQLITE_USE_ALLOCA**

> 如果启用此选项，则在适当的情况下将使用 alloca()内存分配器。这将导致二进制文件稍小且速度稍快。当然，SQLITE_USE_ALLOCA 编译时仅在支持 alloca()的系统上有效。

**SQLITE_USE_FCNTL_TRACE**

> 此选项导致 SQLite 发出额外的 SQLITE_FCNTL_TRACE 文件控制，以向 VFS 提供补充信息。"vfslog.c" 扩展程序利用此功能提供增强的 VFS 活动日志。

**SQLITE_USE_SEH**

> 此选项曾经是一个开关，用于启用现在由 SQLITE_OMIT_SEH 控制的功能。客户端代码不应使用此定义，因为它仅在库内部使用。

**SQLITE_HAVE_ZLIB**

> 此选项导致某些扩展链接到[zlib 压缩库](https://zlib.net)。
> 
> 此选项对 SQLite 核心没有影响，仅用于扩展。此选项对于 SQL Archive 在 命令行 shell 中支持的压缩和解压缩函数是必需的。
> 
> 使用此选项进行编译时，通常需要添加一个链接器选项以将 zlib 库包含到构建中。通常这个选项是 "-lz"，但在不同系统上可能会有所不同。
> 
> 在 Windows 系统上使用 MSVC 编译时，可以将 zlib 源代码放在源树的 compat/zlib 子目录中，然后在 nmake 命令中添加 USE_ZLIB=1 选项，使得 Makefile.msc 自动构建和使用适当的 zlib 库实现。

**YYTRACKMAXSTACKDEPTH**

> 此选项使得 LALR(1) 解析器的堆栈深度可以通过 sqlite3_status(SQLITE_STATUS_PARSER_STACK,...) 接口进行跟踪和报告。SQLite 的 LALR(1) 解析器具有固定的堆栈深度（在编译时使用 YYSTACKDEPTH 选项确定）。此选项可用于帮助确定应用程序是否接近超过最大的 LALR(1) 堆栈深度。

# 8\. 禁用通常开启的功能的选项

**SQLITE_DISABLE_LFS**

> 如果定义了这个 C 预处理宏，将禁用大文件支持。

**SQLITE_DISABLE_DIRSYNC**

> 如果定义了这个 C 预处理宏，目录同步将被禁用。SQLite 通常在删除文件时尝试同步父目录，以确保目录条目立即更新到磁盘上。

**SQLITE_DISABLE_FTS3_UNICODE**

> 如果定义了这个 C 预处理宏，则在构建中会省略 FTS3 中的 unicode61 分词器，并且对应用程序不可用。

**SQLITE_DISABLE_FTS4_DEFERRED**

> 如果此 C 预处理宏禁用了 FTS4 中的“延迟标记”优化。“延迟标记”优化避免了加载集合中大部分文档中存在的术语的大型 posting 列表，而是在文档源中简单地扫描这些标记。FTS4 在有或无此优化的情况下应该得到完全相同的答案。

**SQLITE_DISABLE_INTRINSIC**

> 此选项禁用了在 GCC 和 Clang 中的特定于编译器的内建函数的使用，例如 __builtin_bswap32() 和 __builtin_add_overflow()，或者在 MSVC 中的 _byteswap_ulong() 和 _ReadWriteBarrier()。

**SQLITE_DISABLE_PAGECACHE_OVERFLOW_STATS**

> 此选项禁用了对 sqlite3_status()的收集，其中包括 SQLITE_STATUS_PAGECACHE_OVERFLOW 和 SQLITE_STATUS_PAGECACHE_SIZE 统计信息。设置此选项已显示在高并发多线程应用中提高性能。

# 9\. 省略功能的选项

下面的选项可以通过省略未使用的功能来减少编译库的大小。这可能只在嵌入式系统中特别空间紧张时才有用，因为即使包含所有功能，SQLite 库也相对较小。不要忘记告诉你的编译器优化二进制大小！（如果使用 GCC，则使用 -Os 选项）。告诉编译器优化大小通常比使用任何这些编译时选项的影响要大得多。你还应该验证调试选项是否已禁用。

本节中的宏不需要值。以下编译开关都有相同的效果：

-DSQLITE_OMIT_ALTERTABLE

-DSQLITE_OMIT_ALTERTABLE=1

-DSQLITE_OMIT_ALTERTABLE=0

如果定义了任何这些选项，那么在使用 Lemon parser generator 工具生成 parse.c 文件以及编译生成 keywordhash.h 文件的 'mkkeywordhash' 工具时，必须同时定义相同的 SQLITE_OMIT_* 选项。因此，这些选项只能在库从规范源构建时使用，而不是从 合并 构建。一些 SQLITE_OMIT_* 选项可能在与 合并 使用时工作，或者看起来工作。但这并不保证。总的来说，为了利用 SQLITE_OMIT_* 选项，始终从规范源文件编译。

> ***重要提示：** SQLITE_OMIT_* 选项可能不适用于 合并。SQLITE_OMIT_* 编译时选项通常仅在从规范源文件构建 SQLite 时才能正常工作。*

可以生成特定版本的 SQLite 合并，它们可以使用预定的一组 SQLITE_OMIT_* 选项。要执行此操作，请在规范源代码分发中复制 Makefile.linux-gcc 的 makefile 模板。将您的副本命名为简单的 "Makefile"。然后编辑 "Makefile" 设置适当的编译时选项。然后键入：

```sql
make clean; make sqlite3.c

```

然后，生成的 "sqlite3.c" 合并代码文件（及其相关的头文件 "sqlite3.h"）可以移动到非 Unix 平台，使用本地编译器进行最终编译。

SQLITE_OMIT_* 选项不受支持。这意味着在当前版本中，从构建中省略代码的 SQLITE_OMIT_* 选项可能在下一个版本中成为无操作选项。或者反过来：在当前版本中无操作的 SQLITE_OMIT_* 选项可能会导致下一个版本中排除代码。此外，并非所有 SQLITE_OMIT_* 选项都经过测试。一些 SQLITE_OMIT_* 选项可能会导致 SQLite 发生故障和/或提供不正确的答案。

> ***重要提示：** SQLITE_OMIT_* 编译选项大多数情况下不受支持。*

下列为可用的 OMIT 选项：

**SQLITE_OMIT_ALTERTABLE**

> 当定义此选项时，ALTER TABLE 命令不包含在库中。执行 ALTER TABLE 语句会导致解析错误。

**SQLITE_OMIT_ANALYZE**

> 当定义此选项时，ANALYZE 命令将从构建中省略。

**SQLITE_OMIT_ATTACH**

> 当定义此选项时，ATTACH 和 DETACH 命令将从构建中省略。

**SQLITE_OMIT_AUTHORIZATION**

> 定义此选项会从库中省略授权回调功能。库中不存在 sqlite3_set_authorizer() API 函数。

**SQLITE_OMIT_AUTOINCREMENT**

> 此选项省略了 AUTOINCREMENT 功能。当定义此宏时，声明为 "INTEGER PRIMARY KEY AUTOINCREMENT" 的列在插入 NULL 时表现与声明为 "INTEGER PRIMARY KEY" 的列相同。sqlite_sequence 系统表既不会被创建，也不会被尊重（如果已存在）。

**SQLITE_OMIT_AUTOINIT**

> 为了与不具备 sqlite3_initialize() 接口的旧版本 SQLite 向后兼容，sqlite3_initialize() 接口会在进入某些关键接口（如 sqlite3_open()、sqlite3_vfs_register() 和 sqlite3_mprintf()）时自动调用。以这种方式自动调用 sqlite3_initialize() 的开销可以通过使用 SQLITE_OMIT_AUTOINIT C 预处理宏来省略。当使用 SQLITE_OMIT_AUTOINIT 构建时，SQLite 不会自动初始化自身，应用程序需要在开始使用 SQLite 库之前直接调用 sqlite3_initialize()。

**SQLITE_OMIT_AUTOMATIC_INDEX**

> 此选项用于省略 automatic indexing 功能。另请参见：SQLITE_DEFAULT_AUTOMATIC_INDEX。

**SQLITE_OMIT_AUTORESET**

> 默认情况下，sqlite3_step() 接口将在必要时自动调用 sqlite3_reset() 来重置 prepared statement。此编译时选项更改了此行为，使得在 sqlite3_step() 返回除了 SQLITE_ROW, SQLITE_BUSY 或 SQLITE_LOCKED 之外的值后再次调用它时，它将返回 SQLITE_MISUSE，除非期间有调用 sqlite3_reset()。
> 
> 在 SQLite version 3.6.23.1 (2010-03-26)及更早版本中，sqlite3_step()如果在返回除 SQLITE_ROW 之外的任何其他内容后再次调用且没有调用 sqlite3_reset()之间，它将始终返回 SQLITE_MISUSE。这在一些编写不良的智能手机应用程序上造成了问题，这些应用程序没有正确处理 SQLITE_LOCKED 和 SQLITE_BUSY 的错误返回。为了不必修复许多有缺陷的智能手机应用程序，SQLite 在 3.6.23.2 中更改了其行为，以自动重置准备好的语句。但是，这种变化在其他不正确实现的应用程序中引起了问题，这些应用程序实际上是在寻找 SQLITE_MISUSE 返回来终止其查询循环。 （每当应用程序从 SQLite 获得 SQLITE_MISUSE 错误代码时，这意味着应用程序正在误用 SQLite 接口，因此实现是不正确的。）在 SQLite version 3.7.5 (2011-02-01)中添加了 SQLITE_OMIT_AUTORESET 接口，以尝试使所有（损坏的）应用程序重新工作，而无需实际修复这些应用程序。

**SQLITE_OMIT_AUTOVACUUM**

> 如果定义了此选项，库将无法创建或写入支持 auto_vacuum 的数据库。执行 PRAGMA auto_vacuum 语句不会报错（因为未知的 PRAGMA 会被静默忽略），但不会返回值或修改数据库文件中的自动清理标志。如果使用此选项编译的库打开支持自动清理的数据库，则会自动以只读模式打开。

**SQLITE_OMIT_BETWEEN_OPTIMIZATION**

> 此选项禁用了在使用 BETWEEN 运算符的 WHERE 子句中使用索引的功能。

**SQLITE_OMIT_BLOB_LITERAL**

> 当定义了这个选项时，不可能使用 X'ABCD' 语法在 SQL 语句中指定一个 blob。

**SQLITE_OMIT_BTREECOUNT**

> 这个选项现在不再用于任何内容。它是一个空操作。

**SQLITE_OMIT_BUILTIN_TEST**

> 这个编译时选项已重命名为 SQLITE_UNTESTABLE。

**SQLITE_OMIT_CASE_SENSITIVE_LIKE_PRAGMA**

> 这个编译时选项禁用了 PRAGMA case_sensitive_like 命令。

**SQLITE_OMIT_CAST**

> 这个选项导致 SQLite 省略对 CAST 运算符的支持。

**SQLITE_OMIT_CHECK**

> 这个选项导致 SQLite 省略对 CHECK 约束的支持。解析器仍然会接受 SQL 语句中的 CHECK 约束，但不会强制执行。

**SQLITE_OMIT_COMPILEOPTION_DIAGS**

> 这个选项用于省略 SQLite 中可用的编译时选项诊断，包括 sqlite3_compileoption_used() 和 sqlite3_compileoption_get() C/C++ 函数，以及 sqlite_compileoption_used() 和 sqlite_compileoption_get() SQL 函数，以及 compile_options pragma。

**SQLITE_OMIT_COMPLETE**

> 这个选项会导致 sqlite3_complete() 和 sqlite3_complete16() 接口被省略。

**SQLITE_OMIT_COMPOUND_SELECT**

> 这个选项用于省略复合 SELECT 功能。使用 UNION、UNION ALL、INTERSECT 或 EXCEPT 复合 SELECT 操作符的 SELECT 语句将导致解析错误。
> 
> 在 VALUES 子句中有多个值的 INSERT 语句在内部被实现为一个复合 SELECT。因此，这个选项还禁用了使用 INSERT INTO ... VALUES ... 语句插入多行的能力。

**SQLITE_OMIT_CTE**

> 这个选项会导致省略对 common table expressions 的支持。

**SQLITE_OMIT_DATETIME_FUNCS**

> 如果定义了这个选项，SQLite 内置的日期和时间操作函数会被省略。具体而言，SQL 函数 julianday()、date()、time()、datetime() 和 strftime() 将不可用。默认列值 CURRENT_TIME、CURRENT_DATE 和 CURRENT_TIMESTAMP 仍然可用。

**SQLITE_OMIT_DECLTYPE**

> 这个选项会导致 SQLite 省略对 sqlite3_column_decltype() 和 sqlite3_column_decltype16() 接口的支持。

**SQLITE_OMIT_DEPRECATED**

> 这个选项会导致 SQLite 省略对标记为过时的接口的支持。包括 sqlite3_aggregate_count()、sqlite3_expired()、sqlite3_transfer_bindings()、sqlite3_global_recover()、sqlite3_thread_cleanup() 和 sqlite3_memory_alarm() 接口以及 PRAGMA 语句 PRAGMA count_changes、PRAGMA data_store_directory、PRAGMA default_cache_size、PRAGMA empty_result_callbacks、PRAGMA full_column_names、PRAGMA short_column_names 和 PRAGMA temp_store_directory。

**SQLITE_OMIT_DESERIALIZE**

> 这个选项会导致构建过程中省略 sqlite3_serialize() 和 sqlite3_deserialize() 接口。

**SQLITE_OMIT_DISKIO**

> 此选项省略了所有写入磁盘的支持，并强制数据库仅存在于内存中。此选项未被维护，可能不适用于 SQLite 的新版本。

**SQLITE_OMIT_EXPLAIN**

> 定义此选项会导致库中的 EXPLAIN 命令被省略。尝试执行 EXPLAIN 语句会导致解析错误。

**SQLITE_OMIT_FLAG_PRAGMAS**

> 此选项省略了一些查询和设置布尔属性的 PRAGMA 命令的支持子集。

**SQLITE_OMIT_FLOATING_POINT**

> 此选项用于在 SQLite 库中省略对浮点数的支持。当指定时，将浮点数作为文字（即"1.01"）会导致解析错误。
> 
> 未来，该选项可能还会禁用其他浮点功能，例如 sqlite3_result_double()，sqlite3_bind_double()，sqlite3_value_double()和 sqlite3_column_double() API 函数。

**SQLITE_OMIT_FOREIGN_KEY**

> 如果定义了此选项，则不会识别外键约束语法。

**SQLITE_OMIT_GENERATED_COLUMNS**

> 如果定义了此选项，则不会识别生成列语法。

**SQLITE_OMIT_GET_TABLE**

> 该选项导致 sqlite3_get_table()和 sqlite3_free_table()的支持被省略。

**SQLITE_OMIT_HEX_INTEGER**

> 该选项省略了对十六进制整数文字的支持。

**SQLITE_OMIT_INCRBLOB**

> 该选项导致对增量 BLOB I/O 的支持被省略。

**SQLITE_OMIT_INTEGRITY_CHECK**

> 此选项省略了对 integrity_check pragma 的支持。

**SQLITE_OMIT_INTROSPECTION_PRAGMAS**

> 此选项省略了对 PRAGMA function_list、PRAGMA module_list 和 PRAGMA pragma_list 的支持。

**SQLITE_OMIT_JSON**

> 此选项从构建中省略了 JSON SQL 函数。

**SQLITE_OMIT_LIKE_OPTIMIZATION**

> 此选项禁用了 SQLite 使用索引来帮助解析 WHERE 子句中的 LIKE 和 GLOB 操作符的能力。

**SQLITE_OMIT_LOAD_EXTENSION**

> 此选项从 SQLite 中省略了整个扩展加载机制，包括 sqlite3_enable_load_extension()和 sqlite3_load_extension()接口。

**SQLITE_OMIT_LOCALTIME**

> 此选项省略了日期和时间函数中的"localtime"修饰符。在尝试在不支持本地时间概念的平台上编译日期和时间函数时，此选项有时会很有用。

**SQLITE_OMIT_LOOKASIDE**

> 此选项省略了"lookaside"内存分配器的使用。

**SQLITE_OMIT_MEMORYDB**

> 当定义此选项时，库不会对特殊的数据库名称":memory:"（通常用于创建内存数据库）给予特殊处理。如果":memory:"传递给 sqlite3_open()、sqlite3_open16()或 sqlite3_open_v2()，将打开或创建一个文件以此名称。

**SQLITE_OMIT_OR_OPTIMIZATION**

> 此选项禁用了 SQLite 使用索引来处理 WHERE 子句中由 OR 连接的项的能力。

**SQLITE_OMIT_PAGER_PRAGMAS**

> 定义此选项会从构建中省略与分页子系统相关的编译指令。

**SQLITE_OMIT_PRAGMA**

> 此选项用于省略库中的 PRAGMA 命令。请注意，定义这些宏以省略特定 pragma 也是很有用的，因为它们可能还会移除其他子系统中的支持代码。该宏仅移除 PRAGMA 命令。

**SQLITE_OMIT_PROGRESS_CALLBACK**

> 此选项可能定义为省略在长时间运行的 SQL 语句期间发出“进度”回调的能力。库中不包含 sqlite3_progress_handler() API 函数。

**SQLITE_OMIT_QUICKBALANCE**

> 此选项省略了一种替代、更快的 B-Tree 平衡算法。使用此选项会使 SQLite 稍微变小，但运行速度略有降低。

**SQLITE_OMIT_REINDEX**

> 当定义此选项时，库中不包含 REINDEX 命令。执行 REINDEX 语句将导致解析错误。

**SQLITE_OMIT_SCHEMA_PRAGMAS**

> 定义此选项将省略用于查询数据库模式的 pragma。

**SQLITE_OMIT_SCHEMA_VERSION_PRAGMAS**

> 定义此选项将省略用于查询和修改数据库模式版本和用户版本的 pragma。具体而言，schema_version 和 user_version PRAGMA 都会被省略。

**SQLITE_OMIT_SHARED_CACHE**

> 此选项构建的 SQLite 不支持共享缓存模式。sqlite3_enable_shared_cache() 函数以及与共享缓存管理相关的大量 B-Tree 子系统逻辑均被省略。
> 
> 此编译时选项建议大多数应用程序使用，因为它可以提升性能并减少库的占用空间。

**SQLITE_OMIT_SEH**

> 如果定义了此选项，在 Windows 版本中将禁用结构化异常处理（SEH）。SEH 是一种 Windows 特有的技术，用于捕获访问内存映射文件时引发的异常。SEH 用于拦截在访问与 WAL 模式处理相关的内存映射 shm 文件时可能发生的错误。如果操作系统在 SQLite 尝试访问 shm 文件时引发错误，此选项会使这些错误被 SQLite 捕获并处理，而不会中止整个进程。
> 
> 这只在使用 MSVC 编译 Windows 时才会生效。

**SQLITE_OMIT_SUBQUERY**

> 如果定义了此宏，则省略对子查询和 IN()运算符的支持。

**SQLITE_OMIT_TCL_VARIABLE**

> 如果定义了此宏，则省略了用于自动将 SQL 变量绑定到 TCL 变量的特殊"$<variable-name>"语法。</variable-name>

**SQLITE_OMIT_TEMPDB**

> 此选项省略了对 TEMP 或 TEMPORARY 表的支持。

**SQLITE_OMIT_TRACE**

> 此选项省略了对 sqlite3_profile()和 sqlite3_trace()接口及其相关逻辑的支持。

**SQLITE_OMIT_TRIGGER**

> 定义此选项会省略对 TRIGGER 对象的支持。在这种情况下，既不支持 CREATE TRIGGER 命令，也不支持 DROP TRIGGER 命令，尝试执行任何这些命令都会导致语法错误。此选项还禁用了对外键约束的执行，因为实现触发器的代码（此选项所省略的代码）也用于实现外键动作。

**SQLITE_OMIT_TRUNCATE_OPTIMIZATION**

> 默认的 SQLite 构建中，如果 DELETE 语句没有 WHERE 子句并且作用于没有触发器的表，则会发生优化，导致通过删除并重新创建表来执行 DELETE 操作。删除并重新创建表通常比逐行删除表内容要快得多。这就是“截断优化”。

**SQLITE_OMIT_UTF16**

> 此宏用于省略对 UTF16 文本编码的支持。当定义此宏时，所有返回或接受 UTF16 编码文本的 API 函数都不可用。可以通过这些函数的名称以'16'结尾来识别这些函数，例如 sqlite3_prepare16()，sqlite3_column_text16()和 sqlite3_bind_text16()。

**SQLITE_OMIT_VACUUM**

> 当定义此选项时，库中不包括 VACUUM 命令。执行 VACUUM 语句会导致解析错误。

**SQLITE_OMIT_VIEW**

> 定义此选项将省略对 VIEW 对象的支持。在这种情况下，CREATE VIEW 和 DROP VIEW 命令都不可用，并且尝试执行任何这些命令都会导致解析错误。
> 
> 警告：如果定义了此宏，则无法打开包含 VIEW 对象的模式的数据库。

**SQLITE_OMIT_VIRTUALTABLE**

> 此选项省略了 SQLite 中的虚拟表机制的支持。

**SQLITE_OMIT_WAL**

> 此选项省略了“写前日志”（即“WAL”）功能。

**SQLITE_OMIT_WINDOWFUNC**

> 此选项从构建中省略了窗口函数的支持。

**SQLITE_OMIT_WSD**

> 此选项构建的 SQLite 库版本不包含可写静态数据（WSD）。WSD 指全局变量和/或静态变量。一些平台不支持 WSD，在这些平台上，此选项对 SQLite 的正常工作是必要的。
> 
> 与其他省略选项不同，这个选项实际上会增加 SQLite 的大小，并使其运行速度稍慢。只有当 SQLite 被构建为不支持 WSD 的嵌入式目标时，才使用此选项。

**SQLITE_OMIT_XFER_OPT**

> 此选项省略了支持优化的功能，有助于使 "INSERT INTO ... SELECT ..." 形式的语句运行更快。

**SQLITE_UNTESTABLE**

> 标准的 SQLite 构建包括与 sqlite3_test_control() 相关的少量逻辑，用于验证 SQLite 核心中其他难以验证的部分。此编译时选项省略了这些额外的测试逻辑。在 SQLite 版本 3.16.0（2017-01-02）之前，此编译时选项被称为 "SQLITE_OMIT_BUILTIN_TEST"。更改名称是为了更好地描述使用该选项的影响。
> 
> 设置此编译时选项会导致无法对 SQLite 进行全面测试。分支测试覆盖率从 100% 下降到约 95%。
> 
> SQLite 开发者遵循 "飞行测试你所飞行的，测试你所飞行的" 的 NASA 原则。如果在交付时启用此选项但在测试时禁用，则违反了此原则。但如果在测试期间启用此选项，则不会到达所有分支。因此，不鼓励使用此编译时选项。

**SQLITE_ZERO_MALLOC**

> 此选项将在构建过程中省略默认内存分配器（默认分配器）和调试内存分配器（调试分配器），并替换为一个总是失败的存根内存分配器。SQLite 将无法使用这个存根内存分配器运行，因为它无法分配内存。但可以在启动时使用 sqlite3_config(SQLITE_CONFIG_MALLOC,...) 或 sqlite3_config(SQLITE_CONFIG_HEAP,...) 替换这个存根。因此，这个编译时选项的净效果是允许 SQLite 编译并链接到一个不支持 malloc()、free() 和/或 realloc() 的系统库。

# 10\. 分析和调试选项

**SQLITE_DEBUG**

> SQLite 源代码包含成千上万的 assert() 语句，用于验证内部假设和子程序的前置条件和后置条件。这些 assert() 语句通常是关闭的（它们不生成代码），因为打开它们会使 SQLite 运行速度大约慢三倍。但是对于测试和分析，打开 assert() 语句是很有用的。SQLITE_DEBUG 编译时选项就是用来实现这一点。
> 
> SQLITE_DEBUG 还启用了一些其他调试功能，例如特殊的 PRAGMA 语句，用于开启用于故障排除和分析 VDBE 和代码生成器的跟踪和列表功能。

**SQLITE_MEMDEBUG**

> SQLITE_MEMDEBUG 选项导致 SQLite 内部使用一个带有检测功能的 调试内存分配器 作为默认内存分配器。这个带有检测功能的内存分配器检查动态分配内存的误用。误用的示例包括在释放后继续使用内存、在内存分配结束后写入数据、释放未从内存分配器获取的内存或未初始化新分配的内存。

# 11\. Windows 特定选项

**SQLITE_WIN32_HEAP_CREATE**

> 当启用时，此选项强制 Win32 原生内存分配器创建一个私有堆来保存所有内存分配。

**SQLITE_WIN32_MALLOC_VALIDATE**

> 当启用时，此选项强制 Win32 原生内存分配器在调用 assert() 时进行策略性调用 HeapValidate() 函数。

# 12\. 编译器链接和调用约定控制

下列宏指定了某些类型的 SQLite 构建的接口细节。Makefiles 通常会自动设置这些宏。应用程序开发者通常不需要关注这些宏。以下文档关于这些宏的描述仅为完整性考虑。

**SQLITE_API**

> 此宏标识了 SQLite 的外部可见接口。有时此宏被设置为 "extern"。但其定义取决于编译器。

**SQLITE_APICALL**

> 此宏标识了 SQLite 中公共接口例程所使用的调用约定，这些接口接受固定数量的参数。通常此宏被定义为空，不过在 Windows 平台上的构建中，有时会被设置为 "__cdecl" 或 "__stdcall"。默认情况下是 "__cdecl"，但当 SQLite 被编译为 Windows 系统库时会使用 "__stdcall"。
> 
> 单个函数声明中应当包含以下其中之一：SQLITE_APICALL，SQLITE_CDECL，或者 SQLITE_SYSAPI。

**SQLITE_CALLBACK**

> 此宏指定了 SQLite 中回调指针所使用的调用约定。通常此宏被定义为空，不过在 Windows 平台上的构建中，有时会被设置为 "__cdecl" 或 "__stdcall"。默认情况下是 "__cdecl"，但当 SQLite 被编译为 Windows 系统库时会使用 "__stdcall"。

**SQLITE_CDECL**

> 此宏指定 SQLite 中 varargs 接口例程使用的调用约定。通常情况下，此宏被定义为空，但在 Windows 构建中有时会设置为 "__cdecl"。此宏用于 varargs 例程，因此不能设置为 "__stdcall"，因为 "__stdcall" 调用约定不支持 varargs 函数。
> 
> 单个函数声明中应包含以下选项之一：SQLITE_APICALL、SQLITE_CDECL 或 SQLITE_SYSAPI。

**SQLITE_EXTERN**

> 此宏指定 SQLite 中公共接口变量的链接。通常应默认为 "extern"。

**SQLITE_STDCALL**

> 此宏已不再使用，现在已经被弃用。

**SQLITE_SYSAPI**

> 此宏标识 SQLite 构建的目标平台的操作系统接口所使用的调用约定。通常情况下，此宏被定义为空，但在 Windows 构建中有时会设置为 "__stdcall"。
> 
> 单个函数声明中应包含以下选项之一：SQLITE_APICALL、SQLITE_CDECL 或 SQLITE_SYSAPI。

**SQLITE_TCLAPI**

> 此宏指定 [TCL](http://www.tcl.tk) 库接口例程所使用的调用约定。SQLite 核心不使用此宏，仅供 TCL 接口 和 TCL 测试套件 使用。通常情况下，此宏被定义为空，但在 Windows 构建中有时会设置为 "__cdecl"。此宏用于 TCL 库接口例程，这些例程始终作为 __cdecl 编译，即使在偏好使用 __stdcall 的平台上也是如此，因此除非平台具有支持 __stdcall 的自定义 TCL 库构建，否则不应将此宏设置为 __stdcall。
> 
> 此宏不能与 SQLITE_APICALL，SQLITE_CALLBACK，SQLITE_CDECL 或 SQLITE_SYSAPI 的任何组合一起使用。
