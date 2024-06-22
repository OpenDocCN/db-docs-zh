# 发布历史

> 原文：[`sqlite.com/changes.html`](https://sqlite.com/changes.html)

这一页提供了 SQLite 变更的高级摘要。更详细的信息，请查看 [`www.sqlite.org/src/timeline`](https://www.sqlite.org/src/timeline) 和 [`www.sqlite.org/src/timeline?t=release`](https://www.sqlite.org/src/timeline?t=release) 上的 Fossil 提交日志。参见 chronology 简明列出的发布。

### 2024-05-23 (3.46.0)

1.  多种方式增强 PRAGMA optimize，使其更加 简单易用：

    1.  PRAGMA optimize 现在自动实施临时的 分析限制，以防止大型数据库运行时过度消耗。

    1.  添加了新的 0x10000 位掩码选项，用于检查所有表的更新。

    1.  自动重新分析没有 sqlite_stat1 条目的表。

1.  对 日期和时间函数 的增强：

    1.  SQL 函数 strftime() 现在支持 %G、%g、%U 和 %V。

    1.  新的 'ceiling' 和 'floor' 修饰符控制了在将日期按整数月份和/或年份移动时解决 模糊日期 所使用的算法。

    1.  如果 SQLite 知道时间已经在 UTC 或 localtime 中，'utc' 和 'localtime' 修饰符 现在不起作用。

1.  允许在 数值文字 中的数字之间使用下划线 ("_") 字符。

1.  添加了 json_pretty() SQL 函数。

1.  查询计划器的改进：

    1.  “VALUES-as-coroutine” 优化使得 INSERT 语句中 VALUES 子句有数千行的情况下，解析和运行时间大约减少了一半，内存使用也减少了约一半。

    1.  即使索引记录不比表记录小，也允许在类似 "SELECT count(DISTINCT col) FROM ..." 的查询中使用索引。

    1.  改进了对 SQL 函数的值为常量的识别，因为其所有参数都是常量。

    1.  增强了 WHERE 子句推入优化，以便能够推送包含不相关子查询的 WHERE 子句项。

1.  如果解析器堆栈溢出，为 SQL 解析器堆栈分配额外的内存，而不是报告“解析器堆栈溢出”错误。

1.  JSON 变更：

    1.  允许在 JSON5 字符串文字中使用 ASCII 控制字符。

    1.  修复了-> 和 ->> 运算符，以便当右操作数是看起来像整数的字符串时，仍将其视为字符串，因为这是 PostgreSQL 的行为。

1.  允许将大型十六进制文字用作表列的默认值。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2024-05-23 13:25:27 96c92aba00c8375bc32fafcdf12429c58bd8aabfcadab6683e35bbb9cdebf19e

1.  sqlite3.c 的 SHA3-256：094429ea827fcd32275e767134bc6c7b9ea394a2c5a9e653dd0a0690b2c11358

### 2024-04-15（3.45.3）

1.  修复了一个长期存在的错误（追溯到版本 3.24.0），如果触发器在响应 UPSERT 时触发，可能会（很少）导致 UPDATE 触发器的“old.*”值不正确。[论坛帖子 284955a3cd454a15](https://sqlite.org/forum/forumpost/284955a3cd454a15)。

1.  修复了在 sum()中的一个错误，可能导致它返回 NULL，而应该返回 Infinity。[论坛帖子 23b8688ef4](https://sqlite.org/forum/forumpost/23b8688ef4)。

1.  其他琐事修正和编译器警告修复，这些问题是自上一个补丁发布以来出现的。有关详情，请参见[timeline](https://sqlite.org/src/timeline?from=version-3.45.2&to=version-3.45.3&to2=branch-3.45)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2024-04-15 13:34:05 8653b758870e6ef0c98d46b3ace27849054af85da891eb121e9aaa537f1e8355

1.  sqlite3.c 的 SHA3-256：21dbe688a71b449d28e2a8ec6a43e7520e54df456e02b6d4f6a1d1c7a998c826

### 2024-03-12（3.45.2）

1.  修复了在 UPSERT 中的错误，该错误由版本 3.35.0（2021-03-12）的增强 3a 引入，可能导致索引与其表不同步。[论坛主题 919c6579c8](https://sqlite.org/forum/forumpost/919c6579c8)。

1.  减少了在版本 3.35.0（2021-03-12）中作为 8e 项目添加的 NOT NULL 强度减少优化的范围。该优化在某些情况下尝试，但导致查询结果不正确。[论坛主题 440f2a2f17](https://sqlite.org/forum/forumpost/440f2a2f17)。

1.  其他琐碎的更正和编译器警告修复，这些都是自上一个补丁发布以来出现的。请参阅[时间轴](https://sqlite.org/src/timeline?from=version-3.45.1&to=version-3.45.2&to2=branch-3.45)获取详细信息。

    **Hashes:**

1.  SQLITE_SOURCE_ID："2024-03-12 11:06:23 d8cd6d49b46a395b13955387d05e9e1a2a47e54fb99f3c9b59835bbefad6af77"

1.  sqlite3.c 的 SHA3-256 哈希值为 bd76ad2dc9cde151e469e86627a7e8753aa8ef1a6f657c5a80ba48324b53226b

### 2024-01-30 (3.45.1)

1.  修复了 JSON BLOB 输入 bug，并承诺在后续版本中支持此异常，以保证向后兼容性。

1.  修复了 PRAGMA integrity_check 命令，使其在包含 FTS3 和 FTS5 表的只读数据库上正常工作。这解决了在版本 3.44.0 引入的问题，但是直到 3.45.0 发布后才被发现。

1.  修复了处理损坏的 JSONB 输入时的问题：

    1.  防止将损坏的 JSONB 转换为文本时出现指数级运行时间。

    1.  修复了在将损坏的 JSONB 转换为文本时可能读取 JSONB blob 结尾之后一个字节的错误。

    1.  使用 jfuzz 进行增强测试，以防止未来可能出现的 JSONB 问题，如上述问题。

1.  修复了一个长期存在的 bug，在访问使用内存映射数据库时，可能会在内存映射段的末尾读取几个字节的 bug。

1.  修复了一个长期存在的 bug，由于针对一类故意设计来测试查询规划器但在其他情况下毫无意义的 SQL 语句生成了错误的字节码，可能会导致字节码引擎发生空指针解引用。

    **哈希：**

1.  SQLITE_SOURCE_ID：2024-01-30 16:01:20 e876e51a0ed5c5b3126f52e532044363a014bc594cfefa87ffb5b82257cc467a

1.  sqlite3.c 的 SHA3-256：0474604df9e1b69a5544295dd046aad954749279780d557da80f44b958100295

### 2024-01-15 (3.45.0)

1.  为应用定义的 SQL 函数添加了 SQLITE_RESULT_SUBTYPE 属性。所有调用 sqlite3_result_subtype()的应用定义的 SQL 函数必须注册这个新属性。如果未这样做，可能导致对 sqlite3_result_subtype() 的调用行为像是一个无操作。使用-DSQLITE_STRICT_SUBTYPE=1 进行编译，如果一个不是 SQLITE_RESULT_SUBTYPE 的函数调用了 sqlite3_result_subtype()，则会引发 SQL 错误。对于每个使用子类型的应用程序，推荐使用-DSQLITE_STRICT_SUBTYPE=1 作为编译时选项。

1.  对 JSON SQL 函数进行了增强：

    1.  所有 JSON 函数都被重写以使用一种称为 JSONB 的新内部解析树格式。这种新的解析树格式是可序列化的，因此可以存储在数据库中，以避免每次使用 JSON 值时都进行不必要的重新解析。

    1.  新版本的生成 JSON 的函数生成二进制 JSONB 而不是 JSON 文本。

    1.  json_valid() 函数新增了一个可选的第二参数，用于指定第一个参数“格式良好”的含义。

1.  将 FTS5 tokendata 选项 添加到 FTS5 虚拟表中。

1.  现在默认启用了 SQLITE_DIRECT_OVERFLOW_READ 优化。在编译时使用 -DSQLITE_DIRECT_OVERFLOW_READ=0 可以禁用它。

1.  查询规划器改进：

    1.  当更好的等式约束可用时，不允许传递约束优化欺骗查询规划器使用范围约束。([Forum post 2568d1f6e6](https://sqlite.org/forum/forumpost/2568d1f6e6).)

    1.  查询规划器现在在忽略 ANALYZE 标识为低质量的索引方面做得更好。([Forum post 6f0958b03b](https://sqlite.org/forum/forumpost/6f0958b03b).)

1.  将 SQLITE_MAX_PAGE_COUNT 的默认值从 1073741824 增加到 4294967294。

1.  对 CLI 进行了增强：

    1.  改进了在 Windows 上显示 UTF-8 内容的方法。

    1.  自动检测 ".dump" 脚本的播放并根据需要更改设置，如 ".dbconfig defensive off" 和 ".dbconfig dqs_dll on"。

    **哈希：**

1.  SQLITE_SOURCE_ID: 2024-01-15 17:01:13 1066602b2b1976fe58b5150777cced894af17c803e068f5918390d6915b46e1d

1.  sqlite3.c 的 SHA3-256: f56d8e5e8c61d87b957f1cc60b3042c134d7bc0ca3aba002e6999e8f0af310a3

### 2023-11-24 (3.44.2)

1.  修复了在 3.44.1 中修复的 CLI 中的错误。

1.  修复了在 3.44.1 版本发布标签后几分钟内在内部模糊测试中发现的 FTS5 中的问题。

1.  修复了在上一个版本发布后的第二天由模糊测试器发现的不完整的 assert() 语句。

1.  修复了在使用 GCC 16 进行调试构建时出现的几个无害的编译器警告。

    **哈希：**

1.  SQLITE_SOURCE_ID: 2023-11-24 11:41:44 ebead0e7230cd33bcec9f95d2183069565b9e709bf745c9b5db65cc0cbf92c0f

1.  sqlite3.c 的 SHA3-256 值：bd70b012e2d1b3efa132d905224cd0ab476a69b892f8c6b21135756ec7ffbb13

### 2023-11-22 (3.44.1)

1.  更改 CLI 以在 Windows 上的控制台 I/O 中使用 UTF-16。这使得旧的 Windows7 机器能够正确显示 Unicode 文本。

1.  其他一些不常见的错误修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2023-11-22 14:18:12 d295f48e8f367b066b881780c98bdf980a1d550397d5ba0b0e49842c95b3e8b4

1.  sqlite3.c 的 SHA3-256 值：e359dc502a73f3a8ad8e976a51231134d25cb93ad557a724dd92fe0c5897113a

### 2023-11-01 (3.44.0)

1.  聚合函数现在可以在其最后一个参数后包含 ORDER BY 子句。函数的参数按指定顺序处理。这对于像 string_agg()和 json_group_array()这样的函数非常重要。

1.  增加对 concat()和 concat_ws()标量 SQL 函数的支持，兼容 PostgreSQL、SQLServer 和 MySQL。

1.  增加对 string_agg()聚合 SQL 函数的支持，兼容 PostgreSQL 和 SQLServer。

1.  strftime() SQL 函数上增加了新的转换字符：%e %F %I %k %l %p %P %R %T %u

1.  增加了新的 C 语言 API：sqlite3_get_clientdata() 和 sqlite3_set_clientdata()。

1.  现在在运行 CREATE TABLE 语句本身时，会引发与 CREATE TABLE 相关的许多错误，而不是在实际使用表格时延迟处理。

1.  PRAGMA integrity_check 命令现在使用新的 xIntegrity 方法验证各种内置虚拟表中内容的一致性。这适用于 FTS3、FTS4、FTS5、RTREE 和 GEOPOLY 扩展。

1.  现在，SQLITE_DBCONFIG_DEFENSIVE 设置阻止了 PRAGMA writable_schema 的开启。以前，writable_schema 可以被开启，但实际上不允许模式可写。现在，它根本无法被开启。

1.  将内置的 FTS3，FTS4，FTS5，RTREE 和 GEOPOLY 虚拟表标记为 SQLITE_VTAB_INNOCUOUS，以便它们可以在高安全性部署的触发器中使用。

1.  PRAGMA case_sensitive_like 语句已弃用，因为在模式包含 LIKE 操作符时使用它可能会导致通过 PRAGMA integrity_check 报告数据库损坏。

1.  SQLITE_USE_SEH（结构化异常处理）现在在使用 Microsoft C 编译器构建 SQLite 时默认启用。可以使用 -DSQLITE_OMIT_SEH 来禁用它。

1.  查询规划器优化：

    1.  在部分索引扫描中，如果 WHERE 子句意味着表列的常量值，则用常量替换该表列的出现。这增加了部分索引成为覆盖索引的可能性。

    1.  禁用视图扫描优化（添加于版本 3.42.0 - 项目 1c），因为它导致多个性能回退。取而代之，将 DISTINCT 子查询的预估行数减少 8 倍。

1.  SQLite 现在在运行时检测底层硬件是否支持比 "double" 精度更高的 "long double"，并根据其发现使用适当的浮点数例程。

1.  Windows 版的 CLI 现在默认在支持的平台上使用 UTF-8 进行输入和输出。--no-utf8 选项可用于禁用 UTF8 支持。

    **哈希：**

1.  SQLITE_SOURCE_ID: 2023-11-01 11:23:50 17129ba1ff7f0daf37100ee82d507aef7827cf38de1866e2633096ae6ad8130

1.  sqlite3.c 的 SHA3-256 哈希值为：d9e6530096136067644b1cb2057b3b0fa51070df99ec61971f73c9ba6aa9a36e

### 2023-10-10 (3.43.2)

1.  修复了几个晦涩的 UAF 错误和一个晦涩的内存泄漏。

1.  从标准库中省略使用 CLI 中的 sprintf() 函数，因为在某些平台上现在会生成警告。

1.  避免将 double 类型转换为 unsigned long long 整数，因为某些平台不能正确进行此类转换。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2023-10-10 12:14:04 4310099cce5a487035fa535dd3002c59ac7f1d1bec68d7cf317fd3e769484790

1.  sqlite3.c 的 SHA3-256 哈希值为：e17a3dc69330bd109256fb5a6e2b3ce8fbec48892a800389eb7c0f8856703161

### 2023-09-11 (3.43.1)

1.  修复了 sum()、avg() 和 total() 聚合函数处理无穷大值的回归问题。

1.  修复了 json_array_length() 函数的一个 bug，当参数直接来自 json_remove() 时会出现该 bug。

1.  修复了在版本 3.42.0 中引入的省略未使用子查询列优化，以便在子查询是一个其中一个 arm 是 DISTINCT 而另一个不是的复合情况下正确工作。

1.  其他次要修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2023-09-11 12:01:27 2d3a40c05c49e1a49264912b1a05bc2143ac0e7c3df588276ce80a4cbc9bd1b0

1.  sqlite3.c 的 SHA3-256 哈希值为：391af0a4755e31ae8b29776a4a060b678823ffe4c4db558567567c688a578589

### 2023-08-24 (3.43.0)

1.  增加了对 Contentless-Delete FTS5 索引 的支持。这是一种 FTS5 全文搜索索引的变体，它在索引内容时省略存储内容，同时允许记录被删除。

1.  增强了 日期和时间函数：

    1.  添加了新的时间偏移修饰符，形式为 `±YYYY-MM-DD HH:MM:SS.SSS`。

    1.  添加了 timediff() SQL 函数。

1.  添加了 octet_length(X) SQL 函数。

1.  添加了 sqlite3_stmt_explain() API。

1.  查询计划增强：

    1.  泛化了 LEFT JOIN 强度降低优化，使其也适用于 RIGHT 和 FULL JOINs。将其重命名为 OUTER JOIN 强度降低。

    1.  在 OUTER JOIN 强度降低优化的定理证明器中增强，以减少虚假阴性。

1.  十进制扩展 的增强：

    1.  新函数 decimal_pow2(N) 返回整数 N（取值范围为 -20000 到 +20000）的第 N 次幂的 2。

    1.  新函数 decimal_exp(X) 的功能类似于 decimal(X)，但它以指数表示返回结果，末尾带有 "e+NN"。

    1.  如果 X 是浮点数值，则 decimal(X) 函数现在完全展开该值为其精确的十进制等价值。

1.  对于大型 JSON 字符串的某些处理，JSON 处理 的性能提升导致某些处理类型的性能提升了 2 倍。

1.  新的 makefile 目标 "verify-source" 用于检查源树中是否有意外更改。（仅适用于 标准源代码，不适用于 预编译的合并压缩包。）

1.  添加了 SQLITE_USE_SEH 编译时选项，该选项在 Windows 上使用内存映射的 shm 文件（作为 WAL 模式 的一部分）时启用结构化异常处理。在使用 Makefile.msc 在 Windows 上构建时，默认启用此选项。

1.  Unix 的 VFS 现在默认假定 nanosleep() 系统调用可用，除非使用 -DHAVE_NANOSLEEP=0 编译。

    **哈希：**

1.  SQLITE_SOURCE_ID：2023-08-24 12:36:59 0f80b798b3f4b81a7bb4233c58294edd0f1156f36b6ecf5ab8e83631d468778c

1.  sqlite3.c 的 SHA3-256：a6fc5379891d77b69a7d324cd24a437307af66cfdc3fef5dfceec3c82c8d4078

### 2023-05-16 (3.42.0)

1.  添加 FTS5 安全删除命令。此选项在删除内容时会从 FTS5 倒排索引中删除所有法医痕迹。

1.  改进 JSON SQL 函数，以支持 JSON5 扩展。

1.  SQLITE_CONFIG_LOG 和 SQLITE_CONFIG_PCACHE_HDRSZ 调用现在允许在 sqlite3_initialize() 之后 *进行*。

1.  新的 sqlite3_db_config() 选项：SQLITE_DBCONFIG_STMT_SCANSTATUS 和 SQLITE_DBCONFIG_REVERSE_SCANORDER。

1.  查询规划器改进：

    1.  默认启用“视图计数”优化。

    1.  避免在子查询中计算未使用的列。

    1.  改进 WHERE 子句推送优化。

1.  CLI 的增强功能：

    1.  添加 --unsafe-testing 命令行选项。如果没有此选项，则某些点命令（例如 ".testctrl"）现在被禁用，因为这些命令仅用于测试，如果被误用可能会导致故障。

    1.  即使在 --safe 模式下，也允许命令 ".log on" 和 ".log off"。

    1.  "--" 作为命令行参数意味着所有以 "-" 开头的后续参数都被解释为普通的非选项参数。

    1.  魔术参数 ":inf" 和 ":nan" 绑定到浮点文字 Infinity 和 NaN。

    1.  --utf8 命令行选项在 Windows 控制台的交互会话中省略所有与 MBCS 的转换，并在这些会话期间为 UTF-8 I/O 设置控制台代码页。在所有其他平台上，--utf8 选项是一个无操作。

1.  允许 应用程序定义的 SQL 函数 具有与连接关键字相同的名称：CROSS、FULL、INNER、LEFT、NATURAL、OUTER 或 RIGHT。

1.  对 PRAGMA integrity_check 进行了增强：

    1.  当将 NaN 值存储在 NOT NULL 列中时，检测并引发错误。

    1.  改进的错误消息输出在发现 b 树内部错误时标识了 b 树的根页。

1.  允许配置 会话扩展 以捕获缺乏显式 ROWID 的表的更改。

1.  向 日期和时间函数 添加了 亚秒修饰符。

1.  传递给 sqlite3_sleep() 的负值将被解释为 0。

1.  JSON 数组和对象的最大递归深度从 2000 降低到 1000。

1.  扩展了内置的 printf() 函数，使 逗号选项 现在除了整数转换外，还可用于浮点数转换。

1.  杂项 bug 修复和性能优化

    **哈希：**

1.  SQLITE_SOURCE_ID：2023-05-16 12:36:15 831d0fb2836b71c9bc51067c49fee4b8f18047814f2ff22d817d25195cf350b0

1.  sqlite3.c 的 SHA3-256：6aa3fadf000000625353bbaa1e83af114c40c240a0aa5a2c1c2aabcfc28d4f92

### 2023-03-22（3.41.2）

1.  在以下情况下对内存缓冲区进行读取时的多个修复（注意：*读取*而不是*写入*）：

    1.  在使用非标准 SQLITE_ENABLE_STAT4 编译选项处理损坏的数据库文件时。

    1.  在命令行界面（CLI）中，当 sqlite3_error_offset() 函数返回超出范围的值时（另请参阅下面关于 sqlite3_error_offset() 的修复）。

    1.  在 恢复扩展 中。

    1.  在处理损坏的数据库文件时，在 FTS3 中。

1.  修复 sqlite3_error_offset() 函数，使其在报告与 生成列 相关的错误时不返回超出范围的值。

1.  修复查询优化器中的多个问题，这些问题导致对奇特的模糊器生成的查询产生错误结果。

1.  将页面缓存对象中的引用计数器大小增加到 64 位，以确保计数器永远不会溢出。

1.  修复由补丁版本 3.41.1 中的 bug 修复引起的性能回归。

1.  修复几个不正确的 assert() 语句。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2023-03-22 11:56:21 0d1fc92f94cb6b76bffe3ec34d69cffde2924203304e8ffc4155597af0c191da

1.  sqlite3.c 的 SHA3-256：c83f68b7aac1e7d3ed0fcdb9857742f024449e1300bfb3375131a6180b36cf7c

### 2023-03-10 (3.41.1)

1.  提供编译时选项 -DHAVE_LOG2=0 和 -DHAVE_LOG10=0，以便在省略标准库函数 log2() 和 log10() 的系统上编译 SQLite。

1.  确保在 "`CREATE TABLE t1 AS SELECT CAST(7 AS INT) AS x;`" 中，列 t1.x 的数据类型继续保持为 INT，而不是 NUM，以确保与历史兼容性。

1.  加强 PRAGMA integrity_check 来检测索引记录末尾出现额外字节的情况。

1.  修复用户社区报告的各种隐晦错误。详细信息请参见 [变更时间线](https://sqlite.org/src/timeline?from=version-3.41.0&to=version-3.41.1)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2023-03-10 12:13:52 20399f3eda5ec249d147ba9e48da6e87f969d7966a9a896764ca437ff7e737ff

1.  sqlite3.c 的 SHA3-256：d0d9db8425570f4a57def04fb8f4ac84f5c3e4e71d3d4d10472e3639c5fdf09f

### 2023-02-21 (3.41.0)

1.  查询规划器改进：

    1.  在包含 GROUP BY 子句的聚合查询中使用 索引表达式。

    1.  查询规划器更加清楚地意识到索引是否为 覆盖索引，并相应地调整预测的运行时间。

    1.  查询规划器更加积极地使用 协程，而不是将子查询和视图材料化。

    1.  对内置表值函数 json_tree() 和 json_each() 的查询现在通常将 "ORDER BY rowid" 视为无操作。

    1.  增强了查询规划器使用 索引表达式 的能力，即使表达式已通过 常量传播优化 进行修改。（参见 [论坛帖子 0a539c7](https://sqlite.org/forum/forumpost/0a539c76db3b9e29)。）

1.  添加了内置的 unhex() SQL 函数。

1.  将 base64 和 base85 应用定义函数作为扩展添加，并将该扩展包含在 CLI 中。

1.  添加了 sqlite3_stmt_scanstatus_v2() 接口。（仅当 SQLite 使用 SQLITE_ENABLE_STMT_SCANSTATUS 编译时可用。）

1.  使用 sqlite3_deserialize() 创建的内存数据库现在将其文件名报告为空字符串，而不是 'x'。

1.  对 CLI 的更改：

    1.  添加了新的 base64() 和 base85() SQL 函数。

    1.  使用新的 sqlite3_stmt_scanstatus_v2() 接口增强了 EXPLAIN QUERY PLAN 输出，当使用 SQLITE_ENABLE_STMT_SCANSTATUS 编译时。

    1.  "`.scanstats est`" 命令提供了查询规划器在配置文件中的估计。

    1.  连续提示表明输入当前是否位于字符串文字、标识符文字、注释、触发器定义等内部。

    1.  增强了 --safe 命令行选项以禁止危险的 SQL 函数。

    1.  默认情况下，CLI 构建已禁用 双引号字符串问题。遗留用例可以使用运行时命令 "`.dbconfig dqs_dml on`" 和 "`.dbconfig dqs_ddl on`" 重新启用该问题。

1.  增强 PRAGMA integrity_check 命令，以检测表中的文本字符串是否等效于但不是字节完全相同于索引中的相同字符串。

1.  增强 carray table-valued function，使其能够绑定一个 BLOB 对象数组。

1.  添加了 sqlite3_is_interrupted() 接口。

1.  长时间运行的 sqlite3_prepare() 及类似调用现在会触发 progress handler callback 并响应 sqlite3_interrupt()。

1.  函数 sqlite3_vtab_in_first() 和 sqlite3_vtab_in_next() 被增强，以可靠地检测它们是否在未使用 sqlite3_vtab_in() 进行多值 IN 处理的参数上调用。在这种情况下，它们返回 SQLITE_ERROR 而不是 SQLITE_MISUSE。

1.  解析器现在忽略 IN 运算符右侧子查询周围多余的括号，因此在这方面 SQLite 现在与 PostgreSQL 的处理方式相同。以前，SQLite 将子查询视为带有隐含 "LIMIT 1" 的表达式。

1.  添加了 SQLITE_FCNTL_RESET_CACHE 选项到 sqlite3_file_control() API。

1.  Makefile 改进：

    1.  新的 makefile 目标 "sqlite3r.c" 构建一个包含 recovery extension 的 汇编。

    1.  新的 makefile 目标 "devtest" 和 "releasetest" 分别用于在提交检查前运行快速开发测试和进行完整发布测试。

1.  杂项性能增强。

    **Hashes:**

1.  SQLITE_SOURCE_ID: 2023-02-21 18:09:37 05941c2a04037fc3ed2ffae11f5d2260706f89431f463518740f72ada350866d

1.  SQLite3.c 的 SHA3-256: 02bd9e678460946810801565667fdb8f0c29c78e51240512d2e5bb3dbdee7464

### 2022-12-28 (3.40.1)

1.  修复--safe 命令行选项的 CLI，使其正确禁止使用可能导致有害副作用的 SQL 函数如 writefile()。

1.  修复 memsys5 备选内存分配器中的潜在无限循环。此 bug 是在版本 3.39.0 中由性能优化引入的。

1.  各种其他不太明显的修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-12-28 14:03:47 df5c253c0b3dd24916e4ec7cf77d3db5294cc9fd45ae7b9c5e82ad8197f38a24

1.  sqlite3.c 的 SHA3-256: 4d6800e9032ff349376fe612e422b49ba5eb4e378fac0b3e405235d09dd366ab

### 2022-11-16 (3.40.0)

1.  增加支持将 [SQLite 编译为 WASM](https://sqlite.org/wasm) 并在 Web 浏览器中运行的功能。注：WASM 构建及其接口被视为“beta”，如有必要，可能会进行轻微更改。我们预计将在下一个发布版本中完成接口的最终化。

1.  添加恢复扩展，该扩展可能能够从损坏的数据库文件中恢复部分内容。

1.  查询优化器增强：

    1.  在表格上识别覆盖索引，表格具有超过 63 列，其中第 63 列之后的列被查询使用和/或由索引引用。

    1.  提取表达式索引中包含的表达式的值，在实际情况下，而不是重新计算表达式。

    1.  非 NULL 和 IS NULL 操作符（及其等效操作符）避免从磁盘加载大字符串和 BLOB 值的内容。

    1.  避免在其上执行全扫描一次的视图材料化。使用并在计算完成后丢弃视图的行。

    1.  允许在聚合查询中作为左连接的右操作数中展开子查询。

1.  添加了名为 sqlite3_filename 的新 typedef，并用于表示数据库文件的名称。各种接口已修改以使用新的 typedef 代替"char*"。尽管重新构建某些旧应用程序可能会导致（无害的）编译器警告，但此接口更改应完全向后兼容。

1.  添加了 sqlite3_value_encoding()接口。

1.  安全增强：SQLITE_DBCONFIG_DEFENSIVE 被增强以禁止更改 schema_version。在防御模式下，schema_version 变为只读。

1.  对 PRAGMA integrity_check 语句进行了增强：

    1.  非严格表中具有文本亲和性的列不应包含数值。

    1.  具有数值亲和性的非严格表中的列不应包含可转换为数字的文本值。

    1.  验证 WITHOUT ROWID 表的行是否按正确顺序排列。

1.  增强 VACUUM INTO 语句，以符合 PRAGMA synchronous 设置。

1.  增强 sqlite3_strglob()和 sqlite3_strlike() API，使其能够接受其字符串参数的 NULL 指针并生成合理的结果。

1.  提供了新的 SQLITE_MAX_ALLOCATION_SIZE 编译时选项，用于限制内存分配的大小。

1.  将 SQLite 内置的伪随机数生成器（PRNG）的算法从 RC4 更改为 Chacha20。

1.  允许两个或多个索引具有相同的名称，只要它们位于不同的模式中。

1.  杂项性能优化导致典型工作负载使用的 CPU 周期减少约 1%。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-11-16 12:10:08 89c459e766ea7e9165d0beeb124708b955a4950d0f4792f457465d71b158d318

1.  sqlite3.c 的 SHA3-256：ab8da6bc754642989e67d581f26683dc705b068cea671970f0a7d32cfacbad19

### 2022-09-29 (3.39.4)

1.  修复在 Windows 上的构建，以便与 -DSQLITE_OMIT_AUTOINIT 一起工作。

1.  修复 B 树平衡器中的长期问题，如果应用程序使用应用程序定义的页面缓存，在罕见情况下可能导致数据库损坏。

1.  增强 SQLITE_DBCONFIG_DEFENSIVE，以禁止如果触发器体中的一个或多个语句写入影子表则不允许 CREATE TRIGGER 语句。

1.  修复 FTS3 中内存分配大小计算可能导致整数溢出的问题。

1.  修复在[ICU 扩展](https://sqlite.org/src/dir/ext/icu)中的 sqlite3_set_auxdata()接口的误用。

    **哈希值：**

1.  SQLITE_SOURCE_ID：2022-09-29 15:55:41 a29f9949895322123f7c38fbe94c649a9d6e6c9cd0c3b41c96d694552f26b309

1.  sqlite3.c 的 SHA3-256：f65082298127e2ddae6539beb94f5204b591df64ba2c7da83c7d0faffd6959d8

### 2022-09-05 (3.39.3)

1.  如果语句使用可能会中止的 SQL 函数并影响两个或更多数据库行的 DML 语句，则在语句日志中使用。参见[论坛帖子 9b9e4716c0d7bbd1](https://sqlite.org/forum/forumpost/9b9e4716c0d7bbd1)。

1.  使用互斥锁保护 PRAGMA temp_store_directory 和 PRAGMA data_store_directory 语句，尽管它们已被弃用并且文档化为非线程安全。参见[论坛帖子 719a11e1314d1c70](https://sqlite.org/forum/forumpost/719a11e1314d1c70)。

1.  其他错误和警告修复。查看[时间轴](https://sqlite.org/src/timeline?p=version-3.39.3&bt=version-3.39.2)获取详细信息。

    **哈希值：**

1.  SQLITE_SOURCE_ID：2022-09-05 11:02:23 4635f4a69c8c2a8df242b384a992aea71224e39a2ccab42d8c0b0602f1e826e8

1.  sqlite3.c 的 SHA3-256：2fc273cf8032b601c9e06207efa0ae80eb73d5a1d283eb91096c815fa9640257

### 2022-07-21 (3.39.2)

1.  修复查询规划器中重新排列 FROM 子句顺序导致的性能回归问题。

1.  应用 CVE-2022-35737、Chromium 漏洞 1343348 和 1345947 的修复，[论坛帖子 3607259d3c](https://sqlite.org/forum/forumpost/3607259d3c)，以及内部测试发现的其他小问题。

    **哈希值：**

1.  SQLITE_SOURCE_ID：2022-07-21 15:24:47 698edb77537b67c41adc68f9b892db56bcf9a55e00371a61420f3ddd668e6603

1.  sqlite3.c 的 SHA3-256：bffbaafa94706f0ed234f183af3eb46e6485e7e2c75983173ded76e0da805f11

### 2022-07-13 (3.39.1)

1.  修复查询结果错误的问题，该查询使用包含一个右联接的复合 SELECT 的视图，且该视图不是包含视图的查询的第一个 FROM 子句项。[论坛帖子 174afeae5734d42d](https://sqlite.org/forum/forumpost/174afeae5734d42d)。

1.  修复一些无害的编译器警告。

1.  修复 ALTER TABLE RENAME 的长期问题，只有在设置 sqlite3_limit(SQLITE_LIMIT_SQL_LENGTH)为非常小的值时才会出现。

1.  修复 FTS3 中的长期问题，该问题只会在编译时启用 SQLITE_ENABLE_FTS3_PARENTHESIS 选项时出现。

1.  修复构建，使得在同时提供 SQLITE_DEBUG 和 SQLITE_OMIT_WINDOWFUNC 编译时选项时正常工作。

1.  修复 REGEXP 扩展的初始前缀优化，使其即使前缀包含需要 3 字节 UTF8 编码的字符时也能正确工作。

1.  增强 sqlite_stmt 虚拟表，使其缓冲其所有输出。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-07-13 19:41:41 7c16541a0efb3985578181171c9f2bb3fdc4bad6a2ec85c6e31ab96f3eff201f

1.  sqlite3.c 的 SHA3-256 值为：6d13fcf1c31133da541d1eb8a83552d746f39b81a0657bd4077fed0221749511

### 2022-06-25 (3.39.0)

1.  长期以来（很久未实现的）增加了对 RIGHT 和 FULL OUTER JOIN 的支持。

1.  添加了新的二进制比较运算符 IS NOT DISTINCT FROM 和 IS DISTINCT FROM，它们分别等效于 IS 和 IS NOT，以与 PostgreSQL 和 SQL 标准兼容。

1.  新增了从 sqlite3_vtab_distinct() 接口返回的新的返回码（值为 "3"），该返回码表示一个查询同时包含 DISTINCT 和 ORDER BY 子句。

1.  添加了 sqlite3_db_name() 接口。

1.  Unix 操作系统接口在打开文件之前会将数据库文件名中的所有符号链接解析为数据库的规范名称。如果在使用 SQLITE_OPEN_NOFOLLOW 标志调用 sqlite3_open_v2() 或类似函数时，路径中的任何元素都是符号链接，则打开操作将失败。

1.  延迟视图的材料化直到实际需要，从而避免如果材料化最终未被使用时的不必要工作。

1.  SELECT 语句 的 HAVING 子句 现在可以在任何聚合查询上使用，即使查询本身没有 GROUP BY 子句。

1.  多项 微优化 总体上减少了约 2.3% 的 CPU 周期。

    **Hashes:**

1.  SQLITE_SOURCE_ID: 2022-06-25 14:57:57 14e166f40dbfa6e055543f8301525f2ca2e96a02a57269818b9e69e162e98918

1.  sqlite3.c 的 SHA3-256 值为：d9c439cacad5e4992d0d25989cfd27a4c4f59a3183c97873bc03f0ad1aa78b7a

### 2022-05-06 (3.38.5)

1.  修复了 3.38.4 版本中 CLI 的一个失误，详情请见 这里。

    **Hashes:**

1.  SQLITE_SOURCE_ID: 2022-05-06 15:25:27 78d9c993d404cdfaa7fdd2973fa1052e3da9f66215cff9c5540ebe55c407d9fe

1.  SHA3-256 for sqlite3.c: b05ef42ed234009b4b3dfb36c5f5ccf6d728da80f25ee560291269cf6cfe635f

### 2022-05-04 (3.38.4)

1.  修复了 3.38.0 版本中布隆过滤器下拉优化的字节码问题，其中字节码中的错误导致字节码引擎在下拉优化遇到 NULL 键时进入无限循环。参见 [论坛帖子 2482b32700384a0f](https://sqlite.org/forum/forumpost/2482b32700384a0f)。

1.  其他小修补。详见 [时间线](https://sqlite.org/src/timeline?p=branch-3.38&bt=version-3.38.3)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-05-04 15:45:55 d402f49871152670a62f4f28cacb15d814f2c1644e9347ad7d258e562978e45e

1.  SHA3-256 for sqlite3.c: e6a50effb021858c200e885664611ed3c5e949413ff2dca452ac7ee336b9de1d

### 2022-04-27 (3.38.3)

1.  修复了查询规划器在优化自动索引和布隆过滤器构建时过于激进的情况，使用不适当的 ON 子句项限制自动索引或布隆过滤器的大小，导致输出中缺少行。参见 [论坛帖子 0d3200f4f3bcd3a3](https://sqlite.org/forum/forumpost/0d3200f4f3bcd3a3)。

1.  其他小修补。详见 [时间线](https://sqlite.org/src/timeline?p=version-3.38.3&bt=version-3.38.2)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-04-27 12:03:15 9547e2c38a1c6f751a77d4d796894dec4dc5d8f5d79b1cd39e1ffc50df7b3be4

1.  SHA3-256 for sqlite3.c: d4d66feffad66ea82073fbb97ae9c84e3615887ebc5168226ccee28d82424517

### 2022-03-26 (3.38.2)

1.  修复了新的布隆过滤器优化中发现的一个用户问题，可能导致在使用带有 WHERE 子句约束的 LEFT JOIN 时出现错误答案，该约束指定 LEFT JOIN 的右表中的某一列为 NULL。参见 [论坛帖子 031e262a89b6a9d2](https://sqlite.org/forum/forumpost/031e262a89b6a9d2)。

1.  其他小修补。详见 [时间线](https://sqlite.org/src/timeline?p=version-3.38.2&bt=version-3.38.1)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-03-26 13:51:10 d33c709cc0af66bc5b6dc6216eba9f1f0b40960b9ae83694c986fbf4c1d6f08f

1.  sqlite3.c 的 SHA3-256: 0fbac6b6999f894184899431fb77b9792324c61246b2a010d736694ccaa6d613

### 2022-03-12 (3.38.1)

1.  修复了新的 Bloom 过滤器优化可能导致一些晦涩的查询得到不正确答案的问题。

1.  修复了日期和时间函数的 localtime 修改器，使其保留了小数秒。

1.  修复了 sqlite_offset SQL 函数使其在一些特殊情况下（如参数为虚拟列或视图的列）能够正确工作。

1.  修复了对虚拟表的行值 IN 操作符约束的问题，使得即使虚拟表实现依赖字节码来过滤不满足约束条件的行时，也能够正确工作。

1.  对 assert()语句、测试用例和文档进行了其他较小的修复。详细信息请参见[source code timeline](https://sqlite.org/src/timeline?p=version-3.38.1&bt=version-3.38.0)。

    **哈希值:**

1.  SQLITE_SOURCE_ID: 2022-03-12 13:37:29 38c210fdd258658321c85ec9c01a072fda3ada94540e3239d29b34dc547a8cbc

1.  sqlite3.c 的 SHA3-256: 262ba071e960a8a0a6ce39307ae30244a2b0dc9fe1c4c09d0e1070d4353cd92c

### 2022-02-22 (3.38.0)

1.  添加了-> 和 ->> 操作符以方便处理 JSON。这些新操作符与 MySQL 和 PostgreSQL 兼容。

1.  JSON 函数现在是内置的。不再需要使用-DSQLITE_ENABLE_JSON1 编译选项来启用 JSON 支持。JSON 默认启用。通过新的-DSQLITE_OMIT_JSON 编译选项来禁用 JSON 接口。

1.  日期和时间函数的增强:

    1.  添加了 unixepoch()函数。

    1.  添加了 auto 修改器和 julianday 修改器。

1.  将 printf() SQL 函数 重命名为 format()，以提升兼容性。原始的 printf() 名称保留为向后兼容的别名。

1.  添加了 sqlite3_error_offset() 接口，有时可以帮助将 SQL 错误定位到输入 SQL 文本中的特定字符，从而应用程序可以提供更好的错误消息。

1.  对 虚拟表 的接口进行了增强：

    1.  添加了 sqlite3_vtab_distinct() 接口。

    1.  添加了 sqlite3_vtab_rhs_value() 接口。

    1.  添加了新的操作符类型 SQLITE_INDEX_CONSTRAINT_LIMIT 和 SQLITE_INDEX_CONSTRAINT_OFFSET。

    1.  添加了 sqlite3_vtab_in() 接口（及相关接口），以便虚拟表能够一次性处理 IN 操作符 的约束条件，而不是逐个处理 IN 操作符右侧的每个值。

1.  CLI 增强：

    1.  增强了列式输出模式，以正确处理嵌入文本中的制表符和换行符。

    1.  增加了诸如 "--wrap N"、"--wordwrap on" 和 "--quote" 等选项到 列式输出模式。

    1.  添加了 .mode qbox 的别名。

    1.  .import 命令 自动消除列名的歧义。

    1.  使用新的 sqlite3_error_offset() 接口提供更好的错误消息。

1.  查询规划器增强：

    1.  使用布隆过滤器加快大型分析查询的速度。

    1.  使用平衡合并树来评估带有 ORDER BY 子句的 UNION 或 UNION ALL 复合 SELECT 语句。

1.  ALTER TABLE 语句现在在 sqlite_schema 表 中静默忽略无法解析的条目，当 PRAGMA writable_schema=ON 时。

    **哈希：**

1.  SQLITE_SOURCE_ID: 2022-02-22 18:58:40 40fa792d359f84c3b9e9d6623743e1a59826274e221df1bde8f47086968a1bab

1.  sqlite3.c 的 SHA3-256 哈希值：a69af0a88d59271a2dd3c846a3e93cbd29e7c499864f6c0462a3b4160bee1762

### 2022-01-06 (3.37.2)

1.  修复了在版本 3.35.0（2021-03-12）引入的[bug](https://sqlite.org/forum/forumpost/b03d86f9516cb3a2)，当在 PRAGMA temp_store=MEMORY 模式下回滚 SAVEPOINT，同时进行其他更改并提交外部事务时，可能导致数据库损坏。[Check-in 73c2b50211d3ae26](https://sqlite.org/src/info/73c2b50211d3ae26)。

1.  修复了 ON DELETE CASCADE 和 ON UPDATE CASCADE 中的一个长期问题，即在本地 DDL 更改后未重置用于实现级联更改的 bytecode 的缓存。[Check-in 5232c9777fe4fb13](https://sqlite.org/src/info/5232c9777fe4fb13)。

1.  其他应不会影响生产构建的次要修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2022-01-06 13:25:41 872ba256cbf61d9290b571c0e6d82a20c224ca3ad82971edc46b29818d5d17a0

1.  sqlite3.c 的 SHA3-256 哈希值：1bb01c382295cba85ec4685cedc52a7477cdae71cc37f1ad0f48719a17af1e1e

### 2021-12-30 (3.37.1)

1.  修复了由版本 3.35.0 引入的 UPSERT 增强功能引起的 bug，可能导致对一些复杂但有效的 SQL 生成错误的字节码，可能导致空指针解引用。

1.  修复了在读取损坏的数据库文件时，FTS5 中可能发生的 OOB 读取问题。

1.  提高了 CLI 中 --safe 选项的健壮性。

1.  对 assert() 语句和测试用例进行了其他次要修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-12-30 15:30:28 378629bf2ea546f73eee84063c5358439a12f7300e433f18c9e1bddd948dea62

1.  sqlite3.c 的 SHA3-256 哈希值：915afb3f29c2d217ea0c283326a9df7d505e6c73b40236f0b33ded91f812d174

### 2021-11-27 (3.37.0)

1.  STRICT tables 提供了一种规范的数据类型管理样式，适合喜欢这种类型的开发人员。

1.  当添加包含 CHECK 约束或包含 NOT NULL 约束的生成列的列时，ALTER TABLE ADD COLUMN 现在会检查数据库中现有行的新约束，并且仅在不违反约束的情况下继续。

1.  添加了 PRAGMA table_list 语句。

1.  CLI 增强功能：

    1.  增加了.connection 命令，允许 CLI 同时保持多个数据库连接。

    1.  增加了--安全命令行选项，禁用可能导致超出命令行单个数据库文件的副作用的点命令和 SQL 语句。

    1.  在读取跨多行的 SQL 语句时性能有所提升。

1.  添加了 sqlite3_autovacuum_pages()接口。

1.  sqlite3_deserialize()对 TEMP 数据库不起作用，从未起作用。现在在文档中已经说明了这一限制。

1.  查询规划器现在在子查询和视图上省略 ORDER BY 子句，如果删除这些子句不会改变查询的语义。

1.  generate_series 表值函数扩展已修改，以便第一个参数（"START"）现在是必需的。这样做是为了演示如何编写带有必需参数的表值函数。使用-DZERO_ARGUMENT_GENERATE_SERIES 编译选项可恢复传统行为。

1.  添加了新的 sqlite3_changes64()和 sqlite3_total_changes64()接口。

1.  添加了 SQLITE_OPEN_EXRESCODE 标志选项到 sqlite3_open_v2()。

1.  使用更少的内存来保存数据库架构。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-11-27 14:13:22 bd41822c7424d393a30e92ff6cb254d25c26769889c1499a18a0b9339f5d6c8a

1.  SHA3-256 for sqlite3.c: a202a950ab401cda052e81259e96d6e64ad91faaaaf5690d769f64c2ab962f27

### 2021-06-18 (3.36.0)

1.  改进解释查询计划输出，使其更易于理解。

1.  在标记开头的字节顺序标记被跳过，就像它们是空白一样。

1.  任何试图访问 VIEW 或子查询的 rowid 都会引发错误。以前，VIEW 的 rowid 是不确定的，通常为 NULL。对于需要的应用程序，可通过编译时选项-DSQLITE_ALLOW_ROWID_IN_VIEW 来恢复旧有的行为。

1.  sqlite3_deserialize()和 sqlite3_serialize()接口现在默认启用。不再需要-DSQLITE_ENABLE_DESERIALIZE 编译时选项。取而代之的是一个新的-DSQLITE_OMIT_DESERIALIZE 编译时选项来省略这些接口。

1.  "memdb" VFS 现在允许相同的内存数据库在同一进程中的多个数据库连接之间共享，只要数据库名称以"/"开头。

1.  回退到存在到 IN 优化（在 SQLite 3.35.0 更改日志中的第 8b 项），因为发现它更经常减慢查询而不是加快查询。

1.  改进常量传播优化，使其适用于非连接查询。

1.  [正则表达式扩展](https://sqlite.org/src/file/ext/misc/regexp.c)现在包含在 CLI 构建中。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-06-18 18:36:39 5c9a6c06871cb9fe42814af9c039eb6da5427a6ec28f187af7ebfb62eafa66e5

1.  SHA3-256 for sqlite3.c: 2a8e87aaa414ac2d45ace8eb74e710935423607a8de0fafcb36bbde5b952d157

### 2021-04-19 (3.35.5)

1.  修复新的 ALTER TABLE DROP COLUMN 功能中的缺陷，可能会损坏数据库文件。

1.  修复一个可能导致查询结果不正确的晦涩的查询优化器问题。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-04-19 18:32:05 1b256d97b553a9611efca188a3d995a2fff712759044ba480f9a0c9e98fae886

1.  sqlite3.c 的 SHA3-256：e42291343e8f03940e57fffcf1631e7921013b94419c2f943e816d3edf4e1bbe

### 2021-04-02 (3.35.4)

1.  修复查询规划器优化中标识的问题。Ticket [de7db14784a08053](https://sqlite.org/src/info/de7db14784a08053)。

1.  修复新的 RETURNING 语法中的缺陷。Ticket [132994c8b1063bfb](https://sqlite.org/src/info/132994c8b1063bfb)。

1.  修复新的 RETURNING 特性，如果 RETURNING 子句中的任何项引用了未知表，则引发错误，而不是默默忽略该错误。

1.  修复与聚合函数处理相关联的断言，该断言由于推送优化而被错误触发。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-04-02 15:20:15 5d4c65779dab868b285519b19e4cf9d451d50c6048f06f653aa701ec212df45e

1.  sqlite3.c 的 SHA3-256：528b8a26bf5ffd4c7b4647b5b799f86e8fb1a075f715b87a414e94fba3d09dbe

### 2021-03-26 (3.35.3)

1.  增强字节码引擎的 OP_OpenDup 操作码，以便即使被复制的游标本身来自 OP_OpenDup，它也能正常工作。修复[ticket bb8a9fd4a9b7fce5](https://www.sqlite.org/src/info/bb8a9fd4a9b7fce5)。这个问题之所以显现出来，是因为最近的 MATERIALIZED 提示增强。

1.  当材料化相关的共通表达式时，必须为每个用例单独执行，因为这对于正确性是必需的。这修复了由 MATERIALIZED 提示增强引入的问题。

1.  修复 Unix VFS 的文件名正规化器中的问题。

1.  修正了 CLI 中的"box"输出模式，使其能够处理返回零列一行或多行的语句（例如 PRAGMA incremental_vacuum）。[论坛帖子 afbbcb5b72](https://sqlite.org/forum/forumpost/afbbcb5b72)。

1.  改进了由于错误的公共表达式生成的错误消息。[论坛帖子 aa5a0431c99e](https://sqlite.org/forum/forumpost/aa5a0431c99e631)。

1.  修正了一些不正确的 assert()语句。

1.  修复了 SELECT 语句语法图，以便正确显示 FROM 子句语法。[论坛帖子 9ed02582fe](https://sqlite.org/forum/forumpost/9ed02582fe)。

1.  修复了 EBCDIC 字符分类器，使其能够理解换行符作为空白字符。[论坛帖子 58540ce22dcd](https://sqlite.org/forum/forumpost/58540ce22dcd5fdcd)。

1.  在实现（不支持的）[整数虚拟表](https://sqlite.org/src/file/ext/misc/wholenumber.c)扩展的 xBestIndex 方法中进行了改进，以更好地说服查询规划器避免尝试材料化具有无限行数的表。[论坛帖子 b52a020ce4](https://sqlite.org/forum/forumpost/b52a020ce4)。

    **哈希：**

1.  SQLITE_SOURCE_ID：2021-03-26 12:12:52 4c5e6c200adc8afe0814936c67a971efc516d1bd739cb620235592f18f40be2a

1.  SHA3-256 for sqlite3.c: 91ca6c0a30ebfdba4420bb35f4fd9149d13e45fc853d86ad7527db363e282683

### 2021-03-17（3.35.2）

1.  修复了在版本 3.35.0 中引入的[appendvfs.c](https://www.sqlite.org/src/file/ext/misc/appendvfs.c)扩展中的问题。

1.  确保没有参数的日期/时间函数（生成依赖于当前时间的响应）被视为非确定性函数。票号[2c6c8689fb5f3d2f](https://sqlite.org/src/info/2c6c8689fb5f3d2f)

1.  修复了 sqldiff 实用程序程序中的问题，该问题与虚拟表定义中的异常空白字符有关。

1.  限制了新的 UNION ALL 优化，该优化描述在 3.35.0 发布说明的第 8c 项中，以免尝试生成过多的新子查询。详见 [forum thread 140a67d3d2](https://sqlite.org/forum/forumpost/140a67d3d2)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: 2021-03-17 19:07:21 ea80f3002f4120f5dcee76e8779dfdc88e1e096c5cdd06904c20fd26d50c3827

1.  sqlite3.c 的 SHA3-256 哈希值为 e8edc7b1512a2e050d548d0840bec6eef83cc297af1426c34c0ee8720f378a11

### 2021-03-15（3.35.1）

1.  修复了一个 bug，该 bug 在新的 DROP COLUMN 功能在被用于带有索引并且在索引定义中被引用的列时出现。详见 [a bug](https://www.sqlite.org/src/info/1c24a659e6d7f3a1)。

1.  改进了 CLI 中 .dump 命令的内置文档。

    **Hashes:**

1.  SQLITE_SOURCE_ID: 2021-03-15 16:53:57 aea12399bf1fdc76af43499d4624c3afa17c3e6c2459b71c195804bb98def66a

1.  sqlite3.c 的 SHA3-256 哈希值为 fc79e27fd030226c07691b7d7c23aa81c8d46bc3bef5af39060e1507c82b0523

### 2021-03-12（3.35.0）

1.  增加了内置的 SQL 数学函数（lang_mathfunc.html）。（需要在编译时使用 -DSQLITE_ENABLE_MATH_FUNCTIONS 选项。）

1.  增加了对 ALTER TABLE DROP COLUMN 的支持。

1.  广义化了 UPSERT：

    1.  允许按顺序评估多个 ON CONFLICT 子句，

    1.  最终的 ON CONFLICT 子句可以省略冲突目标，但仍然使用 DO UPDATE。

1.  为 DELETE, INSERT 和 UPDATE 语句增加对 RETURNING 子句的支持。

1.  当运行 VACUUM 时，使用更少的内存来处理包含非常大的 TEXT 或 BLOB 值的数据库。不再需要一次性将整个 TEXT 或 BLOB 存储在内存中。

1.  在指定公共表达式时，增加了对 MATERIALIZED 和 NOT MATERIALIZED 提示的支持。以前的默认行为是 NOT MATERIALIZED，但现在对于多次使用的 CTE，已更改为 MATERIALIZED。

1.  SQLITE_DBCONFIG_ENABLE_TRIGGER 和 SQLITE_DBCONFIG_ENABLE_VIEW 设置已修改，以仅控制主数据库模式或附加数据库模式中的触发器和视图，而不包括 TEMP 模式。TEMP 模式的触发器和视图始终允许。

1.  查询规划器/优化器改进：

    1.  优化了 min/max 优化，使其与上一个版本的 IN 运算符和 OP_SeekScan 优化更好地配合。

    1.  尝试将 WHERE 子句中的 EXISTS 运算符处理为 IN 运算符，如果这是有效的转换，并且可能改善性能。

    1.  允许即使父查询是连接，也可以展开 UNION ALL 子查询。

    1.  在 WHERE 子句中的 IS NOT NULL 表达式上，即使禁用了 STAT4，也可以使用索引（如果适用）。

    1.  形如 "x IS NULL" 或 "x IS NOT NULL" 的表达式，如果 "x" 是具有 "NOT NULL" 约束且未参与外连接的列，则可能被转换为简单的 FALSE 或 TRUE。

    1.  如果 UPDATE 不修改与外键关联的任何列，则在 UPDATE 语句上避免检查外键约束。

    1.  允许将 WHERE 条件推入包含窗口函数的子查询中，只要 WHERE 条件完全由常量和子查询中所有窗口函数的 PARTITION BY 子句中的表达式副本组成。

1.  CLI 增强：

    1.  加强“.stats”命令，接受新参数“stmt”和“vmstep”，分别用于准备语句统计和仅显示虚拟机步数。

    1.  添加了“.filectrl data_version”命令。

    1.  加强“.once”和“.output”命令，如果目标参数以“|”开头（表示输出被重定向到管道），则不需要引号。

1.  Bug 修复：

    1.  修复处理语法不正确的带有相关 WHERE 子句和“HAVING 0”子句的 SELECT 语句时潜在的 NULL 指针解引用问题。（也在 3.34.1 补丁发布中修复。）

    1.  修复了版本 3.33.0 中 IN 操作符优化的错误，可能导致错误答案。

    1.  修复了当模式以“%”结尾且有“ESCAPE '_'”子句时 LIKE 操作符可能出现的错误答案。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-03-12 15:10:09 acd63062eb06748bfe9e4886639e4f2b54ea6a496a83f10716abbaba4115500b

1.  SHA3-256 for sqlite3.c: 73a740d881735bef9de7f7bce8c9e6b9e57fe3e77fa7d76a6e8fc5c262fbaedf

### 2021-01-20（3.34.1）

1.  修复处理同时具有相关 WHERE 子句和“HAVING 0”子句的子查询和父查询为聚合时可能出现的使用后释放错误。

1.  修复文档中的拼写错误。

1.  修复了扩展中的一些小问题。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2021-01-20 14:10:07 10e20c0b43500cfb9bbc0eaa061c57514f715d87238f4d835880cd846b9ebd1f

1.  SHA3-256 for sqlite3.c: 799a7be90651fc7296113b641a70b028c142d767b25af1d0a78f93dcf1a2bf20

### 2020-12-01（3.34.0）

1.  添加了 sqlite3_txn_state()接口，用于报告数据库连接的当前事务状态。

1.  加强递归公共表表达式，支持两个或更多递归项，类似 SQL Server 的做法，这有助于更容易编写和更快执行对图形的查询。

1.  在 CHECK 约束 失败时改进了错误消息。

1.  CLI 增强功能：

    1.  .read dot-command 现在除了文件名外还接受管道。

    1.  添加了 --data-only 和 --nosys 选项到 .dump dot-command。

    1.  添加了 --nosys 选项到 .schema dot-command。

    1.  表名引用在 .import dot-command 中正常工作。

    1.  内置于 CLI 中的表值函数扩展 generate_series(START,END,STEP) 现在已经建立起来。

    1.  .databases dot-command 现在显示每个数据库文件的状态，由 sqlite3_db_readonly() 和 sqlite3_txn_state() 决定。

    1.  添加了 --tabs 命令行选项，设置 .mode tabs。

    1.  如果无法打开作为其参数的文件，则 --init 选项报告错误。 --init 选项现在还支持 --bail 选项。

1.  查询规划器改进：

    1.  改进了运行 DISTINCT 运算符的成本估算。

    1.  在使用多列索引进行 UPDATE 或 DELETE 操作时，索引的前几列对索引查找有用，推迟主表搜索，直到评估完所有 WHERE 子句约束，以防这些约束可以由索引的未使用的后续项覆盖，从而避免不必要的主表搜索。

    1.  新的 OP_SeekScan 操作码用于改进多列索引查找的性能，后续列受 IN 运算符约束时。

1.  即使一个或多个附加数据库文件是只读的，BEGIN IMMEDIATE 和 BEGIN EXCLUSIVE 命令现在也能正常工作。

1.  增强了 FTS5 来支持 trigram indexes。

1.  在有数百个连接同时访问同一数据库文件的情况下，改进了 WAL 模式的锁定原语的性能。

1.  增强了 carray() 表值函数，包括使用辅助的 sqlite3_carray_bind()接口的单参数形式。

1.  substr() SQL 函数现在还可以称为 "substring()"，以与 SQL Server 兼容。

1.  语法图现在以[Pikchr](https://pikchr.org/)脚本的形式实现，并以 SVG 渲染，以提高可读性和维护性。

    **哈希：**

1.  SQLITE_SOURCE_ID：2020-12-01 16:14:00 a26b6597e3ae272231b96f9982c3bcc17ddec2f2b6eb4df06a224b91089fed5b

1.  SHA3-256 for sqlite3.c：fbd895b0655a337b2cd657675f314188a4e9fe614444cc63dfeb3f066f674514

### 2020-08-14（3.33.0）

1.  支持按照 PostgreSQL 语法进行的 UPDATE FROM 操作。

1.  将数据库文件的最大大小增加到 281 TB。

1.  扩展了 PRAGMA integrity_check 语句，以便可以选择性地仅验证单个表及其索引，而不是整个数据库文件。

1.  添加了进行任意精度十进制算术的 decimal 扩展。

1.  增强了与 IEEE 754 binary64 数字一起工作的 ieee754 扩展。

1.  CLI 增强：

    1.  添加了四种新的输出模式："box"、"json"、"markdown"和"table"。

    1.  "column" 输出模式会自动展开列以包含最长的输出行，并在未设置的情况下自动打开 ".header"。

    1.  "quote" 输出模式遵循 ".separator"。

    1.  十进制扩展和 ieee754 扩展都内置到 CLI 中

1.  查询规划器改进：

    1.  添加了在使用 INDEXED BY 查询计划时找到全索引扫描的能力，此前这些查询会因为"无法找到查询解决方案"而失败。

    1.  更好地检测缺失、不完整和/或不良的 sqlite_stat1 数据，并生成良好的查询计划，尽管有误导信息。

    1.  改进了像"SELECT min(x) FROM t WHERE y IN (?,?,?)"这样的查询性能，假设在 t(x,y)上有索引。

1.  在 WAL 模式，如果写入程序崩溃并将 shm 文件留在不一致状态下，后续事务现在能够恢复 shm 文件，即使存在活动读取事务。在此改进之前，shm 文件恢复会导致 SQLITE_PROTOCOL 错误。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2020-08-14 13:23:32 fca8dc8b578f215a969cd899336378966156154710873e68b3d9ac5881b0ff3f

1.  SHA3-256 for sqlite3.c: d00b7fffa6d33af2303430eaf394321da2960604d25a4471c7af566344f2abf9

### 2020-06-18 (3.32.3)

1.  各种次要错误修复，包括对票据[8f157e8010b22af0](https://www.sqlite.org/src/info/8f157e8010b22af0)、[9fb26d37cefaba40](https://www.sqlite.org/src/info/9fb26d37cefaba40)、[e367f31901ea8700](https://www.sqlite.org/src/info/e367f31901ea8700)、[b706351ce2ecf59a](https://www.sqlite.org/src/info/b706351ce2ecf59a)、[7c6d876f84e6e7e2](https://www.sqlite.org/src/info/7c6d876f84e6e7e2)和[c8d3b9f0a750a529](https://www.sqlite.org/src/info/c8d3b9f0a750a529)的修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2020-06-18 14:00:33 7ebdfa80be8e8e73324b8d66b3460222eb74c7e9dfd655b48d6ca7e1933cc8fd

1.  SHA3-256 for sqlite3.c: d00b7fffa6d33af2303430eaf394321da2960604d25a4471c7af566344f2abf9

### 2020-06-04 (3.32.2)

1.  修复字节码引擎中的长期存在的错误，该错误可能导致 COMMIT 命令错误地报告成功。票据[810dc8038872e212](https://www.sqlite.org/src/info/810dc8038872e212)

    **哈希值：**

1.  SQLITE_SOURCE_ID：2020-06-04 12:58:43 ec02243ea6ce33b090870ae55ab8aa2534b54d216d45c4aa2fdbb00e86861e8c

1.  sqlite3.c 的 SHA3-256 值：f17a2a57f7eebc72d405f3b640b4a49bcd02364a9c36e04feeb145eccafa3f8d

### 2020-05-25（3.32.1）

1.  修复了两个长期存在的漏洞，这些漏洞允许恶意的 SQL 语句使运行 SQLite 的进程崩溃。这些漏洞在 3.32.0 版本发布后约 24 小时被第三方公布，但并不局限于 3.32.0 版本。

1.  其他轻微的编译器警告修复和其他事项。

    **哈希值：**

1.  SQLITE_SOURCE_ID：2020-05-25 16:19:56 0c1fcf4711a2e66c813aed38cf41cd3e2123ee8eb6db98118086764c4ba83350

1.  sqlite3.c 的 SHA3-256 值：f695ae21abf045e4ee77980a67ab2c6e03275009e593ee860a2eabf840482372

### 2020-05-22（3.32.0）

1.  增加了对近似分析的支持，使用 PRAGMA analysis_limit 命令。

1.  添加了字节码虚拟表。

1.  将校验和 VFS shim 添加到包含在源代码树中的可运行加载扩展集中。

1.  添加了 iif() SQL 函数。

1.  INSERT 和 UPDATE 语句现在在计算 CHECK 约束之前始终应用列亲和性。理论上，这个 bug 修复可能会导致具有非正统 CHECK 约束的传统数据库出现问题，因为它们要求插入的输入类型与声明的列类型不同。详细信息请参见票证[86ba67afafded936](https://sqlite.org/src/info/86ba67afafded936)。

1.  添加了 sqlite3_create_filename()、sqlite3_free_filename() 和 sqlite3_database_file_object() 接口，以更好地支持 VFS shim 实现。

1.  将参数的默认上限从 999 增加到 32766。

1.  作为可选的 可加载扩展，添加了 UINT 排序序列的代码。

1.  CLI 的增强功能：

    1.  为 .import 命令添加选项：--csv、--ascii、--skip

    1.  .dump 命令现在接受多个 LIKE 模式参数，并输出所有匹配表的并集。

    1.  在调试构建中添加了 .oom 命令。

    1.  将 --bom 选项添加到 .excel、.output 和 .once 命令。

    1.  增强 .filectrl 命令以支持 --schema 选项。

    1.  UINT 排序序列扩展自动加载。

1.  ESCAPE 子句的 LIKE 操作符现在覆盖通配符字符，使其行为与 PostgreSQL 相匹配。

1.  SQLITE_SOURCE_ID: 2020-05-22 17:46:16 5998789c9c744bce92e4cff7636bba800a75574243d6977e1fc8281e360f8d5a

1.  SHA3-256 for sqlite3.c: 33ed868b21b62ce1d0352ed88bdbd9880a42f29046497a222df6459fc32a356f

### 2020-01-27 (3.31.1)

1.  恢复 SQLite 用于内部使用的数据结构的数据布局。使用 SQLite 的应用程序不应引用内部 SQLite 数据结构，但有些应用程序确实这样做了，在 3.30.0 中对其中一个数据结构的更改破坏了一款流行且广泛部署的应用程序。在 SQLite 中撤销这一更改，至少暂时给那些行为异常的应用程序的开发者时间来修复其代码。

1.  修复了 sqlite3ext.h 头文件中的拼写错误，阻止了 sqlite3_stmt_isexplain() 和 sqlite3_value_frombind() 接口被 运行时可加载扩展 调用。

    **哈希：**

1.  SQLITE_SOURCE_ID: 2020-01-27 19:55:54 3bfa9cc97da10598521b342961df8f5f68c7388fa117345eeb516eaa837bb4d6

1.  SHA3-256 for sqlite3.c: de465c64f09529429a38cbdf637acce4dfda6897f93e3db3594009e0fed56d27

### 2020-01-22 (3.31.0)

1.  增加了对 生成列 的支持。

1.  新增了 sqlite3_hard_heap_limit64() 接口以及相应的 PRAGMA hard_heap_limit 命令。

1.  增强了 function_list pragma，显示每个函数的参数数量、函数类型（标量、聚合、窗口）以及函数属性标志 SQLITE_DETERMINISTIC，SQLITE_DIRECTONLY，SQLITE_INNOCUOUS，和/或 SQLITE_SUBTYPE。

1.  向 DBSTAT virtual table 添加了 aggregated mode 特性。

1.  在 sqlite3_open_v2() 函数中添加了 SQLITE_OPEN_NOFOLLOW 选项，用于阻止 SQLite 打开符号链接。

1.  增加了 "#-N" 数组表示法用于 JSON function path arguments。

1.  新增了 SQLITE_DBCONFIG_TRUSTED_SCHEMA 连接设置，也可以通过新的 trusted_schema pragma 和编译时选项 -DSQLITE_TRUSTED_SCHEMA 进行控制。

1.  添加了 API sqlite3_filename_database()，sqlite3_filename_journal()，和 sqlite3_filename_wal()，这些对于特定扩展非常有用。

1.  新增了 sqlite3_uri_key() 接口。

1.  升级了 sqlite3_uri_parameter() 函数，使其除了数据库文件名外，还可以处理回滚日志或 WAL 文件名。

1.  提供了向 应用程序定义的 SQL 函数 标记新属性 SQLITE_INNOCUOUS 或 SQLITE_DIRECTONLY 的能力。

1.  向 sqlite3_vtab_config() 添加新的动词，以便虚拟表的 xConnect 方法可以声明虚拟表为 SQLITE_VTAB_INNOCUOUS 或 SQLITE_VTAB_DIRECTONLY。

1.  对 sqlite3_interrupt() 的更快响应。

1.  添加了 [uuid.c](https://sqlite.org/src/file/ext/misc/uuid.c) 扩展模块，实现了处理 RFC-4122 UUID 的函数。

1.  预留内存分配器 已增强，以支持两个具有不同大小分配的内存池。这使得更多的内存分配可以通过预留内存池来覆盖，同时将每个连接的堆内存使用量减少到 48KB，而不是 120KB。

1.  遗留文件格式 pragma 已停用。现在它是一个空操作。取而代之的是，提供了 SQLITE_DBCONFIG_LEGACY_FILE_FORMAT 选项用于 sqlite3_db_config()。由于以下原因，遗留文件格式 pragma 已停用：(1) 它很少有用，(2) 在具有生成列和降序索引表的模式中与 VACUUM 不兼容。票号 [6484e6ce678fffab](https://www.sqlite.org/src/info/6484e6ce678fffab)

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2020-01-22 18:38:59 f6affdd41608946fcfcea914ece149038a8b25a62bbe719ed2561c649b86d824

1.  sqlite3.c 的 SHA3-256: a5fca0b9f8cbf80ac89b97193378c719d4af4b7d647729d8df9c0c0fca7b1388

### 2019-10-10 (3.30.1)

1.  修复了查询展开器中的一个 bug，该 bug 可能导致使用新的 FILTER 子句 的嵌套查询出现段错误。Ticket [1079ad19993d13fa](https://www.sqlite.org/src/info/1079ad19993d13fa)

1.  在 3.30.0 版本发布以来，挑选了对其他不常见问题的修复。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2019-10-10 20:19:45 18db032d058f1436ce3dea84081f4ee5a0f2259ad97301d43c426bc7f3df1b0b

1.  sqlite3.c 的 SHA3-256 哈希值为: f96fafe4c110ed7d77fc70a7d690e5edd1e64fefb84b3b5969a722d885de1f2d

### 2019-10-04 (3.30.0)

1.  增加了对聚合函数的 FILTER 子句 的支持。

1.  在 ORDER BY 子句中添加了对 NULLS FIRST 和 NULLS LAST 语法的支持。

1.  index_info 和 index_xinfo pragma 被增强，以提供关于 WITHOUT ROWID 表的磁盘表示的信息。

1.  添加了 sqlite3_drop_modules() 接口，允许应用程序禁用它们不需要的自动加载的虚拟表。

1.  对 CLI 中的 .recover dot-command 进行了改进，以便从损坏的数据库文件中恢复更多内容。

1.  增强了 RBU 扩展，以支持表达式索引。

1.  更改模式解析器，如果 sqlite_master 表 的 type、name 和 tbl_name 列之一已损坏且数据库连接未处于 writable_schema 模式，则会报错。

1.  PRAGMA function_list、PRAGMA module_list 和 PRAGMA pragma_list 命令现在默认在所有构建中启用。使用 -DSQLITE_OMIT_INTROSPECTION_PRAGMAS 可以禁用它们。

1.  增加了 SQLITE_DBCONFIG_ENABLE_VIEW 选项，用于 sqlite3_db_config()。

1.  在 TCL Interface 中增加了 config method 以便能够禁用 SQLITE_DBCONFIG_ENABLE_VIEW 以及从 TCL 控制其他 sqlite3_db_config() 选项。

1.  为 应用程序定义的 SQL 函数 添加了 SQLITE_DIRECTONLY 标志，以防止这些函数在触发器和视图中使用。

1.  旧版 SQLITE_ENABLE_STAT3 编译时选项现在不起作用。

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2019-10-04 15:03:17 c20a35336432025445f9f7e289d0cc3e4003fb17f45a4ce74c6269c407c6e09f

1.  sqlite3.c 的 SHA3-256 哈希值为：f04393dd47205a4ee2b98ff737dc51a3fdbcc14c055b88d58f5b27d0672158f5

### 2019-07-10 (3.29.0)

1.  在 sqlite3_db_config() 中添加了 SQLITE_DBCONFIG_DQS_DML 和 SQLITE_DBCONFIG_DQS_DDL 动作，用于激活和停用 双引号字符串文本 的不良特性。由于遗留兼容性，两者默认为“开”，但建议开发者将其设置为“关”，可能使用 -DSQLITE_DQS=0 编译时选项。

1.  -DSQLITE_DQS=0 现在是一个 推荐的编译时选项。

1.  改进了 查询规划器：

    1.  当其中一个操作数为常量时，改进了 AND 和 OR 运算符的优化。

    1.  改进了 LIKE 优化，用于左侧列具有数值亲和性的情况。

1.  添加了"[sqlite_dbdata](https://sqlite.org/src/file/ext/misc/dbdata.c)"虚拟表，用于从 SQLite 数据库中提取原始的低级内容，甚至是损坏的数据库。

1.  改进了舍入行为，使得使用 round()函数对二进制数字进行舍入的结果更接近于习惯于十进制思维的人们的预期。

1.  CLI 的增强：

    1.  添加了".recover"命令，该命令尝试从损坏的数据库文件中尽可能恢复内容。

    1.  添加了对测试非常有用的".filectrl"命令。

    1.  将长期存在的".testctrl"命令添加到".help"菜单。

    1.  添加了".dbconfig"命令。

    **哈希：**

1.  SQLITE_SOURCE_ID：2019-07-10 17:32:03 fc82b73eaac8b36950e527f12c4b5dc1e147e6f4ad2217ae43ad82882a88bfa6

1.  sqlite3.c 中的 SHA3-256 哈希：d9a5daf7697a827f4b2638276ce639fa04e8e8bb5fd3a6b683cfad10f1c81b12

### 2019-04-16（3.28.0）

1.  窗口函数的增强：

    1.  添加对 EXCLUDE 子句的支持。

    1.  添加对窗口链的支持。

    1.  添加对 GROUPS 帧的支持。

    1.  添加了对"<expr> PRECEDING"和"<expr> FOLLOWING"范围边界在 RANGE 帧中的支持。

1.  添加了新的 sqlite3_stmt_isexplain(S)接口，用于确定预处理语句是否为 EXPLAIN。

1.  增强了 VACUUM INTO，使其适用于只读数据库。

1.  新的查询优化：

    1.  在存在 ESCAPE 关键字并且 PRAGMA case_sensitive_like 打开时，启用了 LIKE 优化。

    1.  在由部分索引驱动的查询中，避免对部分索引 WHERE 子句中命名的约束进行不必要的测试，因为我们知道该约束必须始终为真。

1.  改进了 TCL 接口：

    1.  在 function 方法中添加了-returntype 选项。

    1.  添加了新的 bind_fallback 方法。

1.  改进了 CLI：

    1.  增加了对绑定参数和.parameter 命令的支持。

    1.  修复了 readfile()函数，使其在读取空文件时返回空 BLOB 而不是抛出内存不足错误。

    1.  修复了 writefile()函数，以便在创建新文件路径的新目录时，给予它们掩码权限而不是与文件相同的权限。

    1.  修改了--update 选项在.archive 命令中，使其跳过已存在且未更改的文件。添加了新的--insert 选项，其工作方式类似于以前的--update 选项。

1.  添加了[fossildelta.c](https://sqlite.org/src/file/ext/misc/fossildelta.c)扩展，可以创建、应用和分解[Fossil DVCS 文件增量格式](https://fossil-scm.org/fossil/doc/trunk/www/delta_format.wiki)，该格式被 RBU 扩展使用。

1.  添加了 SQLITE_DBCONFIG_WRITABLE_SCHEMA 动词，用于 sqlite3_db_config()接口，其与 PRAGMA writable_schema 执行相同的工作，但不使用 SQL 解析器。

1.  添加了 sqlite3_value_frombind() API，用于确定 SQL 函数的参数是否来自绑定参数。

1.  对 fts3_tokenizer()进行了安全性和兼容性增强：

    1.  fts3_tokenizer()函数总是返回 NULL，除非启用了传统的应用定义的 FTS3 分词器接口，使用 sqlite3_db_config(SQLITE_DBCONFIG_ENABLE_FTS3_TOKENIZER)设置，或者 fts3_tokenizer()的第一个参数是绑定参数。

    1.  两个参数版本的 fts3_tokenizer()即使没有设置 sqlite3_db_config(SQLITE_DBCONFIG_ENABLE_FTS3_TOKENIZER)，也接受指向分词器方法对象的指针，如果第二个参数是一个绑定参数

1.  提高了对损坏数据库文件的鲁棒性。

1.  各种性能增强

1.  建立了官方 SQLite 源代码树的 Git 镜像。SQLite 的官方源使用[Fossil DVCS](https://fossil-scm.org/)进行维护，位于[`sqlite.org/src`](https://sqlite.org/src)。Git 镜像可在[`github.com/sqlite/sqlite`](https://github.com/sqlite/sqlite)查看。

    **哈希值:**

1.  SQLITE_SOURCE_ID: 2019-04-16 19:49:53 884b4b7e502b4e991677b53971277adfaf0a04a284f8e483e2553d0f83156b50

1.  SHA3-256 for sqlite3.c: 411efca996b65448d9798eb203d6ebe9627b7161a646f5d00911e2902a57b2e9

### 2019-02-25 (3.27.2)

1.  修复了在版本 3.27.0 中由尝试优化引入的 IN 运算符中的一个错误。Ticket [df46dfb631f75694](https://www.sqlite.org/src/info/df46dfb631f75694)

1.  修复了一个导致当误用窗口函数时导致崩溃的 bug。Ticket [4feb3159c6bc3f7e33959](https://www.sqlite.org/src/info/4feb3159c6bc3f7e33959)。

1.  修复了各种文档错别字。

    **哈希值：**

1.  SQLITE_SOURCE_ID: bd49a8271d650fa89e446b42e513b595a717b9212c91dd384aab871fc1d0f6d7

1.  sqlite3.c 的 SHA3-256 值为：1dbae33bff261f979d0042338f72c9e734b11a80720fb32498bae9150cc576e7

### 2019-02-08 (3.27.1)

1.  修复了查询优化器中的一个 bug：OR 优化 和试图直接从 表达式索引 读取值而不重新计算表达式之间的不良交互。Ticket [4e8e4857d32d401f](https://www.sqlite.org/src/info/4e8e4857d32d401f)

    **哈希值：**

1.  SQLITE_SOURCE_ID: 2019-02-08 13:17:39 0eca3dd3d38b31c92b49ca2d311128b74584714d9e7de895b1a6286ef959a1dd

1.  sqlite3.c 的 SHA3-256 值为：11c14992660d5ac713ea8bea48dc5e6123f26bc8d3075fe5585d1a217d090233

### 2019-02-07 (3.27.0)

1.  添加了 VACUUM INTO 命令。

1.  如果使用了 双引号字符串文字，在 错误日志 上发出 SQLITE_WARNING 消息。

1.  sqlite3_normalized_sql() 接口适用于使用 sqlite3_prepare_v2() 或 sqlite3_prepare_v3() 创建的任何准备好的语句。不再需要使用 SQLITE_PREPARE_NORMALIZE 来使用 sqlite3_normalized_sql()。

1.  向 FTS3 和 FTS5 添加了 remove_diacritics=2 选项。

1.  向 sqlite3_prepare_v3() 添加了 SQLITE_PREPARE_NO_VTAB 选项。使用该选项可防止 影子表 的循环引用导致资源泄漏。

1.  增强了 sqlite3_deserialize() 接口：

    1.  为 sqlite3_deserialize 创建的内存数据库设置大小上限的 SQLITE_FCNTL_SIZE_LIMIT 文件控制。默认的大小上限是 1GiB，或者可以通过 sqlite3_config(SQLITE_CONFIG_MEMDB_MAXSIZE) 和/或 SQLITE_MEMDB_DEFAULT_MAXSIZE 指定其他值。

    1.  尊重了 SQLITE_DESERIALIZE_READONLY 标志，此标志先前在文档中有描述，但以前是一个无操作。

    1.  增强 TCL 接口 的 "deserialize" 命令，新增了 "--maxsize N" 和 "--readonly BOOLEAN" 选项。

1.  增强了 CLI，主要是为了支持 SQLite 库本身的测试和调试：

    1.  在 ".open --hexdb" 上添加了对 "--hexdb" 的支持。用于生成 "hexdb" 文本的 "[dbtotxt](https://sqlite.org/src/doc/trunk/tool/dbtotxt.md)" 实用程序程序已添加到源码树中。

    1.  在 ".open --deserialize" 上添加了 "--maxsize N" 选项的支持。

    1.  添加了 "--memtrace" 命令行选项，用于显示所有内存分配和释放。

    1.  在使用 SQLITE_DEBUG 编译时，添加 ".eqp trace" 选项，以便一次性启用带缩进的字节码程序列表和 PRAGMA vdbe_trace。

    1.  添加了 ".progress" 命令以访问 sqlite3_progress_handler() 接口。

    1.  为 ".backup" 命令添加了 "--async" 选项。

    1.  在 ".trace" 命令中添加了选项 "--expanded"、"--normalized"、"--plain"、"--profile"、"--row"、"--stmt" 和 "--close"。

1.  增强对针对恶意损坏数据库运行的恶意 SQL 的健壮性。

    **Bug 修复：**

1.  不要使用部分索引来在 IN 运算符上执行表扫描。票号[1d958d90596593a774](https://www.sqlite.org/src/info/1d958d90596593a774)。

1.  修复了查询展开器，使其在包含使用窗口函数的子查询的查询中正常工作。票号[709fcd17810f65f717](https://www.sqlite.org/src/info/f09fcd17810f65f717)。

1.  确保 ALTER TABLE 修改视图和触发器中的 WITH 子句中嵌入的表和列名。

1.  修复了一个解析器错误，该错误阻止了对表值函数周围括号的使用。

1.  修复了关于 OR 优化在表达式索引上的问题。票号[d96eba87698a428c1d](https://www.sqlite.org/src/info/d96eba87698a428c1d)。

1.  修复了左连接强化优化中的问题，该优化由于 IS NOT NULL 操作符的存在而不当应用。票号[5948e09b8c415bc45d](https://www.sqlite.org/src/info/5948e09b8c415bc45d)。

1.  修复了 REPLACE 命令，以便它不再能将 NULL 值偷偷插入非 NULL 列，即使非 NULL 列有默认的 NULL 值。票号[e6f1f2e34dceeb1ed6](https://www.sqlite.org/src/info/e6f1f2e34dceeb1ed6)。

1.  修复了在相关子查询中使用窗口函数的问题。票号[d0866b26f83e9c55e3](https://www.sqlite.org/src/info/d0866b26f83e9c55e3)。

1.  修复了 ALTER TABLE RENAME COLUMN 命令，以便它适用于具有冗余唯一约束的表。票号[bc8d94f0fbd633fd9a](https://www.sqlite.org/src/info/bc8d94f0fbd633fd9a)。

1.  修复了一个导致在使用表达式索引的表中插入 zeroblob 值时被截断的错误。票号[bb4bdb9f7f654b0bb9](https://www.sqlite.org/src/info/bb4bdb9f7f654b0bb9)。

    **哈希：**

1.  SQLITE_SOURCE_ID："2019-02-07 17:02:52 97744701c3bd414e6c9d7182639d8c2ce7cf124c4fce625071ae65658ac61713 "

1.  sqlite3.c 的 SHA3-256：ca011a10ee8515b33e5643444b98ee3d74dc45d3ac766c3700320def52bc6aba

### 2018-12-01（3.26.0）

1.  优化：在对具有表达式索引的表进行 UPDATE 时，如果表达式索引不引用更新的表的任何列，则不更新表达式索引。

1.  允许虚拟表实现的 xBestIndex()方法返回 SQLITE_CONSTRAINT，表示建议的查询计划不可用，不应再进一步考虑。

1.  添加了 SQLITE_DBCONFIG_DEFENSIVE 选项，禁用使用普通 SQL 创建损坏数据库文件的能力。

1.  当启用 SQLITE_DBCONFIG_DEFENSIVE 选项时，添加对只读影子表的支持。

1.  添加了 PRAGMA legacy_alter_table 命令，启用后使 ALTER TABLE 命令在兼容性上 behave like SQLite 的旧版本（3.25.0 之前的版本）。

1.  添加了 PRAGMA table_xinfo，其功能与 PRAGMA table_info 相同，但也显示虚拟表中的隐藏列。

1.  添加了[explain 虚拟表](https://sqlite.org/src/file/ext/misc/explain.c)作为运行时可加载扩展。

1.  在查询规划器中添加了限制计数器，以防止某些病态 SQL 输入导致过多的 sqlite3_prepare()时间。

1.  添加了对 sqlite3_normalized_sql()接口的支持，编译时需启用 SQLITE_ENABLE_NORMALIZE。

1.  增强触发器，使其可以使用存在于定义触发器的模式之外模式中的 表值函数。

1.  对 CLI 进行了改进：

    1.  改进了 ".help" 命令。

    1.  如果存在 SQLITE_HISTORY 环境变量，则指定命令行编辑历史文件的名称。

    1.  与打开新数据库相关的 --deserialize 选项导致数据库文件被读入内存，并使用 sqlite3_deserialize() API 访问。这简化了在不修改磁盘上文件的情况下对数据库运行测试的过程。

1.  对 geopoly 扩展进行了改进：

    1.  总是使用二进制格式存储多边形，这样更快且占用更少空间。

    1.  增加了 geopoly_regular() 函数。

    1.  增加了 geopoly_ccw() 函数。

1.  对 session 扩展进行了改进：

    1.  增加了 SQLITE_CHANGESETAPPLY_INVERT 标志

    1.  增加了 sqlite3changeset_start_v2() 接口和 SQLITE_CHANGESETSTART_INVERT 标志。

    1.  增加了 [changesetfuzz.c](https://sqlite.org/src/file/ext/session/changesetfuzz.c) 测试用例生成工具。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2018-12-01 12:34:55 bf8c1b2b7a5960c282e543b9c293686dccff272512d08865f4600fb58238b4f9"

1.  sqlite3.c 的 SHA3-256：72c08830da9b5d1cb397c612c0e870d7f5eb41a323b41aa3d8aa5ae9ccedb2c4

### 2018-11-05 (3.25.3)

1.  不允许在公共表达式递归部分使用 窗口函数。票证 [e8275b415a2f03bee](https://sqlite.org/src/info/e8275b415a2f03bee)

1.  修复了虚拟表上 typeof() 和 length() 的行为。票证 [69d642332d25aa3b7315a6d385](https://sqlite.org/src/info/69d642332d25aa3b7315a6d385)

1.  加强了防御措施，以防故意损坏的数据库文件。

1.  修复了查询规划器中的问题，当使用具有冗余列的主键时使用行值表达式会导致问题。Ticket [1a84668dcfdebaf12415d](https://sqlite.org/src/info/1a84668dcfdebaf12415d)

1.  修复了查询规划器在 SQLITE_ENABLE_STAT4 编译时选项下在 LEFT JOIN 的 ON 子句中对 IS NOT NULL 运算符的正确处理问题。[65eb38f6e46de8c75e188a17ec](https://sqlite.org/src/info/65eb38f6e46de8c75e188a17ec)

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2018-11-05 20:37:38 89e099fbe5e13c33e683bef07361231ca525b88f7907be7092058007b75036f2"

1.  sqlite3.c 的 SHA3-256 值：45586e4df74de3a43f3a1f8c7a78c3c3f02edce01af7d10cafe68bb94476a5c5

### 2018-09-25 (3.25.2)

1.  添加了 PRAGMA legacy_alter_table=ON 命令，使得 "ALTER TABLE RENAME" 命令的行为与 SQLite 3.24.0 及更早版本相同：重命名表格在触发器和视图中的引用不会被更新。这个新的 pragma 提供了对旧程序的兼容性解决方案，这些程序期望 ALTER TABLE 的旧版奇怪行为。

1.  修复了新的 窗口函数 实现中的问题，当在视图中使用涉及窗口函数的复杂表达式时发生故障。

1.  修复了与各种其他编译器警告和与不常见配置相关的小问题。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2018-09-25 19:08:10 fb90e7189ae6d62e77ba3a308ca5d683f90bbe633cf681865365b8e92792d1c7"

1.  sqlite3.c 的 SHA3-256 值：34c23ff91631ae10354f8c9d62fd7d65732b3d7f3acfd0bbae31ff4a62fe28af

### 2018-09-18 (3.25.1)

1.  在 3.25.0 版本的 ALTER TABLE 中添加了额外的健全性检查，有时会在修改的表格具有更新虚拟表的触发器时引发误报。这个误报导致 ALTER TABLE 回滚，因此使模式保持不变。Ticket [b41031ea2b537237](https://sqlite.org/src/info/b41031ea2b537237).

1.  对于涉及窗口函数的某些查询，修复 3.25.0 版本中与 ORDER BY LIMIT 优化相关的字节码无限循环的问题。需要额外的修正。Ticket [510cde277783b5fb](https://sqlite.org/src/info/510cde277783b5fb)

    **哈希：**

1.  SQLITE_SOURCE_ID：“2018-09-18 20:20:44 2ac9003de44da7dafa3fbb1915ac5725a9275c86bf2f3b7aa19321bf1460b386”

1.  sqlite3.c 中的 SHA3-256：1b2302e7a54cc99c84ff699a299f61f069a28e1ed090b89e4430ca80ae2aab06

### 2018-09-15（3.25.0）

1.  添加对窗口函数的支持。

1.  增强 ALTER TABLE 命令：

    1.  添加支持通过 ALTER TABLE *table* RENAME COLUMN *oldname* TO *newname*在表内重命名列的功能。

    1.  修复表重命名功能，以便在触发器（lang_createtrigger.html）和视图（lang_createview.html）中更新对重命名表的引用。

1.  查询优化器改进：

    1.  避免在聚合查询中加载不在聚合函数内且不在 GROUP BY 子句中的无用列。

    1.  IN-early-out 优化：当在多列索引上进行查找，并且使用 IN 运算符在非最左列上时，如果第一个 IN 值没有匹配的行，则在继续下一个 IN 值之前，检查确保存在匹配右侧列的行。

    1.  使用传递属性尝试在 WHERE 子句中传播常量值。例如，将“a=99 AND b=a”转换为“a=99 AND b=99”。

1.  在 Unix 的 VFS 中的每个 inode 都使用单独的互斥锁，而不是所有 inode 共享一个互斥锁，以便在多线程环境中稍微提高并发性能。

1.  增强 PRAGMA integrity_check 命令，以改进对页面空闲列表中问题的检测。

1.  在命令行 shell 的“.dump”命令中将无穷大输出为 1e999。

1.  添加了 SQLITE_FCNTL_DATA_VERSION 文件控制。

1.  添加了 Geopoly 模块。

    **Bug 修复：**

1.  在极其晦涩的情况下，ORDER BY LIMIT 优化可能导致准备语句的字节码进入无限循环，因为查询优化器的一系列轻微缺陷交汇在一起。修复了票号 [9936b2fa443fec03ff25](https://www.sqlite.org/src/info/9936b2fa443fec03ff25)。

1.  当 UPSERT 中约束检查的顺序被重新排列时，确保插入内容的亲和转换发生在任何约束检查之前。修复了票号 [79cad5e4b2e219dd197242e9e](https://www.sqlite.org/src/info/79cad5e4b2e219dd197242e9e)。

1.  在 ".eqp full" 逻辑关闭了 ".stats on" 命令之后，避免使用准备语句执行 CLI。修复了票号 [7be932dfa60a8a6b3b26bcf76](https://www.sqlite.org/src/info/7be932dfa60a8a6b3b26bcf76)。

1.  LIKE 优化生成的字节码错误，因此如果左操作数具有数值亲和性并且右操作数模式为 '/%' 或模式以 ESCAPE 字符开头，则得到错误答案。修复了票号 [c94369cae9b561b1f996d0054b](https://www.sqlite.org/src/info/c94369cae9b561b1f996d0054b)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2018-09-15 04:01:47 b63af6c3bd33152742648d5d2e8dc5d5fcbcdd27df409272b6aea00a6f761760"

1.  对 sqlite3.c 的 SHA3-256：989e3ff37f2b5eea8e42205f808ccf0ba86c6ea6aa928ad2c011f33a108ac45d。

### 2018-06-04 (3.24.0)

1.  增加了对 PostgreSQL 风格的 UPSERT 支持。

1.  增加了在 R-tree 表中的辅助列支持。

1.  添加了用于发现 SQLite 使用的 SQL 关键字的 C 语言 API：sqlite3_keyword_count()、sqlite3_keyword_name() 和 sqlite3_keyword_check()。

1.  添加了基于 sqlite3_str 对象的动态字符串的 C 语言 API。

1.  加强了 ALTER TABLE，使其能够识别 "true" 和 "false" 作为 DEFAULT 的有效参数。

1.  将排序器参考优化作为编译时选项添加。仅在使用 SQLITE_ENABLE_SORTER_REFERENCES 编译时可用。

1.  改进了 EXPLAIN QUERY PLAN 原始输出的格式，从而提供更好的查询计划信息以及计划中各组件之间的关系。

1.  添加了 SQLITE_DBCONFIG_RESET_DATABASE 选项到 sqlite3_db_config() API。

    **CLI 增强：**

1.  自动拦截原始的 EXPLAIN QUERY PLAN 输出并将其重新格式化为 ASCII 艺术图。

1.  以 "#" 开头且不在 SQL 语句中间的行被解释为注释。

1.  向 ".backup" 命令添加了 --append 选项。

1.  添加了 ".dbconfig" 命令。

    **性能：**

1.  当数据库文件的内容实际上没有变化时，UPDATE 避免不必要的低级磁盘写入。例如，如果列 x 中的值已经是 25，则 "UPDATE t1 SET x=25 WHERE y=?" 不会生成额外的磁盘 I/O。类似地，当对跨多个页面的记录进行 UPDATE 时，只会将实际更改的页面子集写入磁盘。这是一种低级性能优化，不影响 TRIGGER 或其他更高级别的 SQL 结构的行为。

1.  使用 ORDER BY 和 LIMIT 的查询现在尝试避免计算那些不可能在 LIMIT 下返回的行。这可以极大地提升 ORDER BY LIMIT 查询的性能，特别是当 LIMIT 相对于无限制输出行数较小时。

1.  允许 OR 优化即使 OR 表达式也已转换为 IN 表达式也可以继续进行。OR 优化的使用现在在 EXPLAIN QUERY PLAN 输出中也更加清晰地显示。

1.  查询规划器更加积极地使用自动索引来处理视图和子查询，这些情况下无法创建持久索引。

1.  在适当的情况下，利用一遍完成的 UPDATE 和 DELETE 查询计划来使用 R-Tree 扩展。

1.  在 LEMON 生成的解析器中进行性能改进。

    **错误修复：**

1.  对于左连接的右表，直接计算表达式的值，而不是从表达式索引中加载预计算值，因为表达式索引可能不包含正确的值。Ticket [7fa8049685b50b5aeb0c2](https://sqlite.org/src/info/7fa8049685b50b5aeb0c2)

1.  不要试图使用左连接的 WHERE 子句中的术语来启用右表的索引查找。Ticket [4ba5abf65c5b0f9a96a7a](https://sqlite.org/src/info/4ba5abf65c5b0f9a96a7a)

1.  修复了在 CSV 虚拟表中打开错误后可能出现的内存泄漏问题。

1.  解决了长期存在的问题，即由于 sqlite_sequence 表中的损坏架构而导致 AUTOINCREMENT 崩溃的问题。Ticket [d8dc2b3a58cd5dc2918a1](https://www.sqlite.org/src/info/d8dc2b3a58cd5dc29)

1.  修复了 json_each()函数，在输入是简单值而不是数组或对象时返回其“fullkey”列的有效结果的问题。

    **哈希：**

1.  SQLITE_SOURCE_ID："2018-06-04 19:24:41 c7ee0833225bfd8c5ec2f9bf62b97c4e04d03bd9566366d5221ac8fb199a87ca"

1.  sqlite3.c 的 SHA3-256：0d384704e1c66026228336d1e91771d295bf688c9c44c7a44f25a4c16c26ab3c

### 2018-04-10（3.23.1）

1.  修复了新的 LEFT JOIN 强化优化中的两个问题。票号[1e39b966ae9ee739](https://sqlite.org/src/info/1e39b966ae9ee739)和[fac496b61722daf2](https://sqlite.org/src/info/fac496b61722daf2)。

1.  修复了 FTS5 xBestIndex 方法的不正确行为。票号[2b8aed9f7c9e61e8](https://sqlite.org/src/info/2b8aed9f7c9e61e8)。

1.  修复了对未初始化虚拟机寄存器的无害引用。票号[093420fc0eb7cba7](https://sqlite.org/src/info/093420fc0eb7cba7)。

1.  修复了 CLI，使其在使用-DSQLITE_UNTESTABLE 时能够构建。

1.  修复了[eval.c](https://sqlite.org/src/file/ext/misc/eval.c)扩展，使其能够与 PRAGMA empty_result_callbacks=ON 一起工作。

1.  修复了 generate_series 虚拟表，使其在任何约束为空时正确返回零行。

1.  解析器性能有所提升。

    **哈希值：**

1.  SQLITE_SOURCE_ID："2018-04-10 17:39:29 4bb2294022060e61de7da5c227a69ccd846ba330e31626ebcd59a94efd148b3b"。

1.  对 sqlite3.c 进行 SHA3-256：65750d1e506f416a0b0b9dd22d171379679c733e3460549754dc68c92705b5dc。

### 2018-04-02（3.23.0）

1.  在使用 SQLITE_ENABLE_DESERIALIZE 编译时选项时，添加了 sqlite3_serialize()和 sqlite3_deserialize()接口。

1.  现在识别 TRUE 和 FALSE 作为常量。（为了兼容性，如果存在名为"true"或"false"的列，则标识符将指向列而不是布尔常量。）

1.  支持操作符 IS TRUE、IS FALSE、IS NOT TRUE 和 IS NOT FALSE。

1.  添加了 SQLITE_DBSTATUS_CACHE_SPILL 选项，用于报告发生的缓存溢出次数。

1.  在 内置 printf 实现中，“alternate-form-2” 标志（"!"）现在会导致字符串替换测量宽度和精度时使用字符而不是字节。

1.  如果 虚拟表 实现中的 xColumn 方法使用 sqlite3_result_error() 返回错误消息，则优先使用该错误消息，而不是内部生成的消息。

1.  为 CLI 添加了 -A 命令行选项，以便更轻松地管理 SQLite 存档文件。

1.  在 Zipfile 虚拟表 中添加对 INSERT OR REPLACE、INSERT OR IGNORE 和 UPDATE OR REPLACE 的支持。

1.  增强 sqlite3changeset_apply() 接口，使其能够抵御故意损坏的 changeset 对象的攻击。

1.  添加了 [sqlite3_normalize()](https://sqlite.org/src/file/ext/misc/normalize.c) 扩展函数。

1.  查询优化器增强：

    1.  改进 omit-left-join 优化 使其在右表是 UNIQUE 但不一定 NOT NULL 的情况下也能工作。

    1.  改进 push-down 优化 使其适用于许多 LEFT JOIN。

    1.  添加了 LEFT JOIN 强化优化，如果 WHERE 子句中存在术语会阻止 LEFT JOIN 的额外全空行出现在输出集中，则将 LEFT JOIN 转换为普通 JOIN。

    1.  当 AUTOINCREMENT 表使用小于最大值的 rowid 更新时，避免向 sqlite_sequence 表进行不必要的写入。

1.  Bug 修复：

    1.  修复解析器以接受有效的 行值 语法。Ticket [7310e2fb3d046a5](https://www.sqlite.org/src/info/7310e2fb3d046a5)

    1.  修复查询规划器，使其考虑 WHERE 子句中子表达式中表值函数参数的依赖关系。Ticket [80177f0c226ff54](https://www.sqlite.org/src/info/80177f0c226ff54)

    1.  修复复杂 OR 连接的 WHERE 和 STAT4 中的不正确结果。Ticket [ec32177c99ccac2](https://www.sqlite.org/src/info/ec32177c99ccac2)

    1.  由于自动数据类型转换，表达式索引中的潜在损坏修复。Ticket [343634942dd54ab](https://www.sqlite.org/src/info/343634942dd54ab)

    1.  FTS4 中的断言错误。Ticket [d6ec09eccf68cfc](https://www.sqlite.org/src/info/d6ec09eccf68cfc)

    1.  在行值中小于运算符的不正确结果。Ticket [f484b65f3d62305](https://www.sqlite.org/src/info/f484b65f3d62305)

    1.  总是将非零浮点值解释为 TRUE，即使整数部分为零。Ticket [36fae083b450e3a](https://www.sqlite.org/src/info/36fae083b450e3a)

    1.  修复了在 fsdir(PATH) 表值函数 中的问题，这导致如果 fsdir() 表作为连接中的内部表使用，会发生段错误。该问题在邮件列表中报告，并通过提交 [7ce4e71c1b7251be](https://www.sqlite.org/src/info/7ce4e71c1b7251be) 进行了修复。

    1.  当 sqlite_master 表损坏以至于 sqlite_sequence 表根页面实际上是 btree 索引页面时，发出错误而不是断言错误或空指针解引用。Check-in [525deb7a67fbd647](https://www.sqlite.org/src/info/525deb7a67fbd647)

    1.  修复 ANALYZE 命令，使其对以 "sqlite" 开头的表计算统计信息。Check-in [0249d9aecf69948d](https://sqlite.org/src/info/0249d9aecf69948d)

1.  由 [OSSFuzz](https://github.com/google/oss-fuzz) 检测到的其他问题修复：

    1.  修复损坏数据库文件的 VACUUM 可能的无限循环问题。检查 [27754b74ddf64](https://www.sqlite.org/src/info/27754b74ddf64)

    1.  在触发器和视图的 `WITH` 子句中禁止 parameters。检查 [b918d4b4e546d](https://www.sqlite.org/src/info/b918d4b4e546d)

    1.  修复 row value 处理中可能的内存泄漏。检查 [2df6bbf1b8ca8](https://www.sqlite.org/src/info/2df6bbf1b8ca8)

    1.  改进 replace() SQL 函数 的性能，针对大量替换的兆字节大小字符串，以避免在测试期间 OSSFuzz 超时。检查 [fab2c2b07b5d3](https://www.sqlite.org/src/info/fab2c2b07b5d3)

    1.  当 `sqlite_master` 表包含 `CREATE TABLE AS` 语句时，请提供适当的错误消息。以前这会导致断言错误或空指针解引用。由 OSSFuzz 在 GDAL 项目上发现的问题。检查 [d75e67654aa96](https://www.sqlite.org/src/info/d75e67654aa96)

    1.  移除不正确的 `assert()` 语句。检查 [823779d31eb09cda](https://www.sqlite.org/src/info/823779d31eb09cda)。

    1.  修复在 INTEGER PRIMARY KEY 上使用 LIKE 优化 的问题。检查 [b850dd159918af56](https://www.sqlite.org/src/info/b850dd159918af56)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2018-04-02 11:04:16 736b53f57f70b23172c30880186dce7ad9baa3b74e3838cae5847cffb98f5cd2"

1.  `sqlite3.c` 的 SHA3-256：4bed3dc2dc905ff55e2c21fd2725551fc0ca50912a9c96c6af712a4289cb24fa

### 2018-01-22 (3.22.0)

1.  现在，sqlite3_trace_v2() 的输出显示触发器内运行的每个单独 SQL 语句。

1.  添加了从 WAL mode 数据库中读取数据的能力，即使应用程序对数据库及其所在目录缺乏写权限，只要该目录中存在 -shm 和 -wal 文件。

1.  添加了标量 SQL 函数 rtreecheck()到 R-Tree 扩展。

1.  添加了 sqlite3_vtab_nochange()和 sqlite3_value_nochange()接口，帮助虚拟表实现优化 UPDATE 操作。

1.  添加了 sqlite3_vtab_collation()接口。

1.  在 FTS5 中添加了对"^"初始标记语法的支持。

1.  新扩展：

    1.  虚拟表 Zipfile 可以读取和写入[ZIP 归档文件](https://en.wikipedia.org/wiki/Zip_(file_format))。

    1.  在[fileio.c](https://sqlite.org/src/file/ext/misc/fileio.c)扩展中添加了 fsdir(PATH) 表值函数，用于列出目录中的文件。

    1.  [sqlite_btreeinfo](https://sqlite.org/src/file/ext/misc/btreeinfo.c)命名的虚拟表，用于自省和估算数据库中 B 树的大小。

    1.  [追加 VFS](https://sqlite.org/src/file/ext/misc/appendvfs.c)是一个 VFS 垫片，允许 SQLite 数据库追加到其他文件。这允许（例如）将数据库追加到可执行文件中，然后打开和读取数据库。

1.  查询规划器增强：

    1.  使用索引快速计算聚合函数 min()或 max()的优化扩展到表达式索引上。

    1.  是否将 FROM 子查询实现为协程或使用查询展开的决定现在考虑外部查询结果集是否“复杂”（如果包含函数或表达式子查询）。复杂的结果集倾向于使用协程。

    1.  规划器避免使用具有未知排序函数的索引的查询计划。

    1.  查询规划器即使不是查询的最右边连接，也会省略未使用的 LEFT JOIN。

1.  其他性能优化：

    1.  对文本到浮点数转换子例程 sqlite3AtoF() 进行了更小更快的实现。

    1.  Lemon 解析器生成器 创建了更快的解析器。

    1.  使用 strcspn() C 库例程来加速 LIKE 和 GLOB 操作符。

1.  对 命令行 shell 进行了改进：

    1.  ".schema" 命令显示了虚拟表的结构。

    1.  添加了对读写 SQLite Archive 文件的支持，使用 .archive command。

    1.  添加了实验性的 .expert command。

    1.  添加了 ".eqp trigger" 变种的 ".eqp" 命令。

    1.  增强了 ".lint fkey-indexes" 命令，使其在 WITHOUT ROWID 表格中工作。

    1.  如果 shell 的文件名参数是 ZIP 归档而不是 SQLite 数据库，则 shell 将自动使用 Zipfile 虚拟表 打开该 ZIP 归档。

    1.  添加了 edit() SQL 函数。

    1.  添加了 .excel command 以简化将数据库内容导出到电子表格。

    1.  使用 [Append VFS](https://sqlite.org/src/file/ext/misc/appendvfs.c) 打开数据库，当命令行或 .open 命令使用 --append 标志时。

1.  增强了 SQLITE_ENABLE_UPDATE_DELETE_LIMIT 编译时选项，使其适用于 WITHOUT ROWID 表格。

1.  提供了 sqlite_offset(X) SQL 函数，当编译时使用 -DSQLITE_ENABLE_OFFSET_SQL_FUNC 时，返回到数据库文件中保存值 X 的记录的字节偏移量。

1.  Bug 修复：

    1.  UPDATE 中在 WHERE 子句中使用 OR 运算符导致无限循环。此问题在 3.17.0 中引入，并在一年后的邮件列表中报告。Ticket [47b2581aa9bfecec](https://www.sqlite.org/src/info/47b2581aa9bfecec)。

    1.  当使用跳跃前向唯一优化时，查询结果不正确。票号 [ef9318757b152e3a](https://sqlite.org/src/info/ef9318757b152e3a)。

    1.  在带有 ORDER BY DESC 的连接中出现查询结果不正确。票号 [123c9ba32130a6c9](https://sqlite.org/src/info/123c9ba32130a6c9)。

    1.  在 CREATE TABLE AS 和简单 SELECT 之间存在不一致的结果集列名。

    1.  在表达式索引上执行 REPLACE 时出现断言错误。票号 [dc3f932f5a147771](https://sqlite.org/src/info/dc3f932f5a147771)

    1.  在对常量索引执行 IN 操作时出现断言错误。票号 [aa98619ad08ddcab](https://sqlite.org/src/info/aa98619ad08ddcab)

    **哈希值：**

1.  SQLITE_SOURCE_ID："2018-01-22 18:45:57 0c55d179733b46d8d0ba4d88e01a25e10677046ee3da1d5b1581e86726f2171d"

1.  sqlite3.c 的 SHA3-256：206df47ebc49cd1710ac0dd716ce5de5854826536993f4feab7a49d136b85069

### 2017-10-24（3.21.0）

1.  在可用时利用[F2FS 文件系统](https://en.wikipedia.org/wiki/F2FS)的原子写入功能，以大大减少事务开销。目前这需要在编译时启用 SQLITE_ENABLE_BATCH_ATOMIC_WRITE 选项。

1.  允许在事务内部使用 ATTACH 和 DETACH 命令。

1.  如果主键正好包含一列，则允许对 WITHOUT ROWID 虚拟表进行写操作。

1.  现在在 WAL 重置后头部写入后发生的"fsync()"使用检查点的同步设置。这意味着在 mac 上如果设置了 PRAGMA checkpoint_fullfsync，它将使用"fullfsync"。

1.  sqlite3_sourceid()函数试图检测源代码是否与版本控制中检查的内容不同，如果有修改，则显示版本哈希的最后四个字符为“alt1”或“alt2”。这旨在检测意外或粗心的编辑。伪造者可以破坏此功能。

1.  改进了对右侧聚合查询中列名的去引用，适用于 CREATE TABLE AS 语句。

1.  unix VFS 减少了“stat()”系统调用。

1.  增强了 LIKE 优化，使其可以与 ESCAPE 子句一起使用。

1.  增强了 PRAGMA integrity_check 和 PRAGMA quick_check，以检测以前遗漏的隐晦行损坏。还更新了这两个 PRAGMA，以便在 fileformat2.html#record_format 中遇到损坏时返回错误文本，而不是 SQLITE_CORRUPT。

1.  查询规划器现在更倾向于使用协程实现 FROM 子句子查询，而不是使用查询平坦化优化。支持使用协程进行子查询可能不再被禁用。

1.  将关于!=、IS、IS NOT、NOT NULL 以及 IS NULL 约束传递到虚拟表的 xBestIndex 方法中。

1.  增强了 CSV 虚拟表，使其在最后一个换行符缺失时接受输入的最后一行。

1.  移除很少使用的“scratch”内存分配器。用 SQLITE_CONFIG_SMALL_MALLOC 配置设置替换，以向 SQLite 提示在可能的情况下应避免大内存分配。

1.  在现有的联合虚拟表扩展中增加了[swarm 虚拟表](https://sqlite.org/src/file/ext/misc/unionvtab.c)。

1.  添加了[sqlite_dbpage 虚拟表](https://sqlite.org/src/file/src/dbpage.c)，用于直接访问数据库文件的页面。源代码内置于综合包中，并通过-DSQLITE_ENABLE_DBPAGE_VTAB 编译时选项激活。

1.  添加了一个新类型的 fts5vocab 虚拟表 - "instance" - 提供 FTS5 全文索引的最低级别直接访问。

1.  由于在一些较旧的笔记本电脑上导致 Firefox 中出现问题，移除了 Windows VFS 中的 rand_s()调用。

1.  [src/shell.c](https://sqlite.org/src/finfo?name=src/shell.c)源代码不再受版本控制，用于命令行 shell。该文件现在作为构建过程的一部分生成。

1.  杂项 microoptimizations 将 CPU 使用率降低约 2.1%。

1.  Bug 修复:

    1.  修复了一个由 OSSFuzz 发现的错误的 assert()语句。Ticket [cb91bf4290c211d](https://sqlite.org/src/info/cb91bf4290c211d)

    1.  修复了在 sqlite3_result_pointer()中的一个隐晦的内存泄漏。Ticket [7486aa54b968e9b](https://sqlite.org/src/info/7486aa54b968e9b)

    1.  通过推迟模式重置直到查询规划器完成运行，避免了可能的延迟释放错误。Ticket [be436a7f4587ce5](https://sqlite.org/src/info/be436a7f4587ce5)

    1.  仅在 COLLATE 正确的情况下，使用表达式索引来优化 ORDER BY 或 GROUP BY。Ticket [e20dd54ab0e4383](https://sqlite.org/src/info/e20dd54ab0e4383)

    1.  修复了一个断言错误，当索引表达式中的表达式真的是常量时会出现。Ticket [aa98619ad08ddca](https://sqlite.org/src/info/aa98619ad08ddca)

    1.  修复了一个断言错误，可能会在 PRAGMA reverse_unordered_selects 之后发生。Ticket [cb91bf4290c211d](https://sqlite.org/src/info/cb91bf4290c211d)

    1.  修复可能发生的 segfault，该问题可能出现在使用 IN 或 EXISTS 子查询中的表值函数中。 Ticket [b899b6042f97f5](https://sqlite.org/src/info/b899b6042f97f5)

    1.  修复在编译特定恶劣的公共表达式时可能出现的整数溢出问题。 这是由 OSSFuzz 发现的另一个问题。 Check-in [6ee8cb6ae5](https://sqlite.org/src/info/6ee8cb6ae5)。

    1.  修复在查询损坏的数据库文件时可能发生的潜在越界读取问题，这是由 Google Project Zero 的 Natalie Silvanovich 检测到的问题。 Check-in [04925dee41a21f](https://sqlite.org/src/info/04925dee41a21f)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-10-24 18:55:49 1a584e499906b5c87ec7d43d4abce641fdf017c42125b083109bc77c4de48827"

1.  SHA3-256 for sqlite3.c: 84c181c0283d0320f488357fc8aab51898370c157601459ebee49d779036fe03

### 2017-08-24 (3.20.1)

1.  修复了新的 sqlite3_result_pointer() 接口可能出现的内存泄漏问题。 Ticket [7486aa54b968e9b5](https://sqlite.org/src/info/7486aa54b968e9b5)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-08-24 16:21:36 8d3a7ea6c5690d6b7c3767558f4f01b511c55463e3f9e64506801fe9b74dce34"

1.  SHA3-256 for sqlite3.c: 93b1a6d69b48dc39697d1d3a1e4c30b55da0bdd2cad0c054462f91081832954a

### 2017-08-01 (3.20.0)

1.  更新了由 sqlite3_errmsg() 返回的某些错误代码的错误消息文本。

1.  添加新的 指针传递接口。

1.  部分扩展插件进行了不兼容的更改，以利用新的 指针传递接口 所提供的改进安全性。

    1.  扩展 FTS5 → 需要使用 sqlite3_bind_pointer() 来查找 fts5_api 指针。

    1.  carray(PTR,N) → 需要使用 sqlite3_bind_pointer() 来设置 PTR 参数。

    1.  [remember(V,PTR)](https://www.sqlite.org/src/file/ext/misc/remember.c) → 需要使用 sqlite3_bind_pointer() 来设置 PTR 参数。

1.  添加了 SQLITE_STMT 虚拟表扩展。

1.  添加了 COMPLETION 扩展 - 旨在为交互式用户界面提供选项补全建议。这是一个正在进行中的工作。预计在未来版本中会有进一步的增强。

1.  添加了 UNION 虚拟表扩展。

1.  内置的日期和时间函数已经增强，以便在 CHECK 约束，在表达式索引以及部分索引的 WHERE 子句中使用，前提是它们不使用'now'、'localtime'或'utc'关键字。更多信息。

1.  添加了带有额外的"prepFlags"参数的 sqlite3_prepare_v3()和 sqlite3_prepare16_v3()接口。

1.  为 sqlite3_prepare_v3()提供了 SQLITE_PREPARE_PERSISTENT 标志，并使用它来限制 FTS3、FTS5 和 R-Tree 扩展对 lookaside 内存的滥用。

1.  添加了 PRAGMA secure_delete=FAST 命令。当 secure_delete 设置为 FAST 时，旧内容将被覆盖为零，只要这不会增加 I/O 的量。删除的内容可能仍然存在于空闲页列表，但将从所有 b-tree 页中清除。

1.  对命令行 shell 进行了增强：

    1.  使用 COMPLETION 扩展为 readline 和 linenoise 添加了选项补全支持。

    1.  添加了".cd"命令。

    1.  增强了".schema"命令，以显示所有附加数据库的模式。

    1.  增强了".tables"，以便在所有附加数据库的模式名称为"main"以外时显示模式名称。

    1.  ".import"命令会忽略初始的 UTF-8 BOM。

    1.  给".dump"命令添加了"--newlines"选项，以便输出 U+000a 和 U+000d 字符时，直接输出而不是使用 replace()函数转义。

1.  查询规划器增强：

    1.  在为 OR 扫描的每个 ORed 项生成单独循环时，将任何常量 WHERE 表达式移出循环，就像顶级循环一样。

    1.  查询规划器检查绑定参数的值，以帮助确定是否可以使用部分索引。

    1.  在选择具有相同估计成本的两个计划时，偏向于不使用排序器的计划。

    1.  最后评估涉及相关子查询的 WHERE 子句约束，希望根本不需要评估它们。

    1.  如果子查询在 LEFT JOIN 的右侧读取 virtual table 的数据，则不要对其进行 flattening optimization，因为这样会阻止查询规划器在子查询结果上创建 automatic indexes，从而可能降低查询速度。

1.  为 sqlite3_stmt_status()接口添加 SQLITE_STMTSTATUS_REPREPARE，SQLITE_STMTSTATUS_RUN 和 SQLITE_STMTSTATUS_MEMUSED 选项。

1.  为 PRAGMA integrity_check，PRAGMA quick_check 和 PRAGMA foreign_key_check 提供 PRAGMA functions。

1.  为 TCL interface eval method 添加了-withoutnulls 选项。

1.  增强 sqlite3_analyzer.exe 实用程序程序，使其显示 btree 页面上元数据的字节数。

1.  SQLITE_DBCONFIG_ENABLE_QPSG 运行时选项和 SQLITE_ENABLE_QPSG 编译时选项启用了查询规划器稳定性保证。另见票号[892fc34f173e99d8](https://www.sqlite.org/src/info/892fc34f173e99d8)。

1.  杂项优化导致 CPU 使用周期减少了 2%。

    **Bug Fixes:**

1.  修复了对使用优化平铺的查询的 sqlite3_column_name()行为，以使结果与不使用该优化的其他查询以及 PostgreSQL、MySQL 和 SQLServer 的结果一致。票号[de3403bf5ae](https://sqlite.org/src/info/de3403bf5ae)。

1.  修复了查询规划器的错误，使其在 WHERE 子句使用 IS 运算符时知道不要在 LEFT JOIN 的右表上使用自动索引。修复票号[ce68383bf6aba](https://sqlite.org/src/info/ce68383bf6aba)。

1.  确保查询规划器知道扁平化 LEFT JOIN 的任何列都可以为 NULL，即使该列标记为“NOT NULL”。修复票号[892fc34f173e99d8](https://sqlite.org/src/info/892fc34f173e99d8)。

1.  修复在使用附加数据库的数据库连接上运行 PRAGMA integrity_check 时的罕见假阳性。票号[a4e06e75a9ab61a12](https://sqlite.org/src/info/a4e06e75a9ab61a12)。

1.  修复了一个由 OSSFuzz 发现的错误，如果使用某些有问题的 CREATE TABLE 声明，会导致断言错误。票号[bc115541132dad136](https://sqlite.org/src/info/bc115541132dad136)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-08-01 13:24:15 9501e22dfeebdcefa783575e47c60b514d7c2e0cad73b2a496c0bc4b680900a8"

1.  sqlite3.c 的 SHA3-256：79b7f3b977360456350219cba0ba0e5eb55910565eab68ea83edda2f968ebe95

### 2017-06-17 (3.18.2)

1.  修复在 WHERE 子句中使用 IN 操作符可能导致重复输出行的错误。Ticket [61fe9745](https://sqlite.org/src/info/61fe9745)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2017-06-17 09:59:36 036ebf729e4b21035d7f4f8e35a6f705e6bf99887889e2dc14ebf2242e7930dd"

1.  SHA3-256 for sqlite3.c: b0bd014f2776b9f9508a3fc6432f70e2436bf54475369f88f0aeef75b0eec93e

### 2017-06-16 (3.18.1)

1.  修复与自动清理相关的一个可能导致数据库损坏的错误。该错误在版本 3.16.0 (2017-01-02) 中引入。Ticket [fda22108](https://sqlite.org/src/info/fda22108)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2017-06-16 13:41:15 77bb46233db03a3338bacf7e56f439be3dfd1926ea0c44d252eeafa7a7b31c06"

1.  SHA3-256 for sqlite3.c: 334eaf776db9d09a4e69d6012c266bc837107edc2c981739ef82081cb11c5723

### 2017-06-08 (3.19.3)

1.  修复与自动清理相关的一个可能导致数据库损坏的错误。该错误在版本 3.16.0 (2017-01-02) 中引入。Ticket [fda22108](https://sqlite.org/src/info/fda22108)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2017-06-08 14:26:16 0ee482a1e0eae22e08edc8978c9733a96603d4509645f348ebf55b579e89636b"

1.  SHA3-256 for sqlite3.c: 368f1d31272b1739f804bcfa5485e5de62678015c4adbe575003ded85c164bb8

### 2017-05-25 (3.19.2)

1.  修复 LEFT JOIN 扁平化优化中的更多错误。Ticket [7fde638e94287d2c](https://www.sqlite.org/src/info/7fde638e94287d2c)。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2017-05-25 16:50:27 edb4e819b0c058c7d74d27ebd14cc5ceb2bad6a6144a486a970182b7afe3f8b9"

1.  SHA3-256 for sqlite3.c: 1be0c457869c1f7eba58c3b5097b9ec307a15be338308bee8e5be8570bcf5d1e

### 2017-05-24 (3.19.1)

1.  修复 LEFT JOIN 扁平化优化中的一个错误。Ticket [cad1ab4cb7b0fc](https://www.sqlite.org/src/info/cad1ab4cb7b0fc)。

1.  删除导致旧版 MSVC 出现问题的多余分号。

    **哈希值：**

1.  SQLITE_SOURCE_ID："2017-05-24 13:08:33 f6d7b988f40217821a382bc298180e9e6794f3ed79a83c6ef5cae048989b3f86"

1.  SHA3-256 for sqlite3.c：996b2aff37b6e0c6663d0312cd921bbdf6826c989cbbb07dadde5e9672889bca

### 2017-05-22（3.19.0）

1.  SQLITE_READ 授权回调 对每个在查询中引用的表调用一次，其列名为空字符串，对于从中未提取列的表。

1.  当在表达式上使用索引时，尽量使用索引中已有的表达式值，而不是加载原始列并重新计算表达式。

1.  增强扁平化优化，使其能够扁平化 LEFT JOIN 右侧的视图。

1.  在.dump 输出的字符串中，使用 replace()代替 char()来转义换行和回车字符。

1.  避免在不涉及受外键约束的列的 UPDATE 语句中进行不必要的外键处理。

1.  在使用索引的 DISTINCT 查询中，如果有合适的索引可用，则尝试跳过到下一个不同的条目，而不是逐行遍历。

1.  避免在对无关表进行更改时无谓地使 sqlite3_blob 句柄失效。

1.  将 HAVING 子句中仅使用 GROUP BY 子句中提到的列的任何条件移到 WHERE 子句中，以加快处理速度。

1.  如果相同的 VIEW 在同一查询中出现多次，则重复使用该 VIEW 的同一材料化。

1.  增强 PRAGMA integrity_check 以识别具有两个或更多相同 rowid 的表。

1.  增强 FTS5 查询语法，使得可以对任意表达式应用列过滤器。

1.  增强了 json_extract()函数以缓存和重用 JSON 输入文本的解析。

1.  添加了[anycollseq.c](https://sqlite.org/src/file/ext/misc/anycollseq.c) 可加载扩展，允许通用 SQLite 数据库连接读取包含未知和/或应用特定排序序列的模式。

    **错误修复：**

1.  修复了在 REPLACE 中可能导致数据库损坏，包含两个或更多具有相同 rowid 的行的问题。修复了票号[f68dc596c4e6018d](https://www.sqlite.org/src/info/f68dc596c4e6018d)。

1.  修复了 PRAGMA integrity_check 中的问题，导致后续的 VACUUM 行为不佳。

1.  修复了 PRAGMA foreign_key_check 命令，以便在 WITHOUT ROWID 表上正确工作。

1.  修复了 B-tree 逻辑中的错误，可能导致 IN 操作符查询的不正确重复答案。票号[61fe9745](https://sqlite.org/src/info/61fe9745)

1.  不允许 JSON 数值常量中的前导零。修复了票号[b93be8729a895a528e2](https://www.sqlite.org/src/info/b93be8729a895a528e2)。

1.  不允许 JSON 字符串中存在控制字符。修复了票号[6c9b5514077fed34551](https://www.sqlite.org/src/info/6c9b5514077fed34551)。

1.  为了避免递归下降解析器中的过度堆栈使用，限制了 JSON 对象和数组的递归深度。修复了票号[981329adeef51011052](https://www.sqlite.org/src/info/981329adeef51011052)。

    **哈希：**

1.  SQLITE_SOURCE_ID: "2017-05-22 13:58:13 28a94eb282822cad1d1420f2dad6bf65e4b8b9062eda4a0b9ee8270b2c608e40"

1.  sqlite3.c 的 SHA3-256：c30326aa1a9cc342061b755725eac9270109acf878bc59200dd4b1cea6bc2908

### 2017-03-30 (3.18.0)

1.  添加了 PRAGMA optimize 命令

1.  由 sqlite_source_id() SQL 函数返回的 SQLite 版本标识符，以及 sqlite3_sourceid() C API 中的标识符，并在 SQLITE_SOURCE_ID 宏中找到的标识符，现在是 64 位 SHA3-256 哈希，而不是 40 位 SHA1 哈希。

1.  添加了 json_patch() SQL 函数到 JSON1 扩展。

1.  增强 LIKE optimization，使其适用于左侧任意表达式，只要右侧的 LIKE 模式不以数字或减号开头。

1.  添加了 sqlite3_set_last_insert_rowid()接口，并在 FTS3、FTS4 和 FTS5 扩展中使用新接口，以确保 sqlite3_last_insert_rowid()接口始终返回合理的值。

1.  增强 PRAGMA integrity_check 和 PRAGMA quick_check，以便验证 CHECK constraints。

1.  增强连接查询计划，早期检测空表并在不必要的情况下停止执行。

1.  增强 sqlite3_mprintf()系列接口和 printf SQL function，在整数中使用","格式修饰符，千位数标记处添加逗号分隔符（例如："%,d"）。

1.  添加了-DSQLITE_MAX_MEMORY=*N*编译时选项。

1.  添加了.sha3sum dot-command 和.selftest dot-command 到命令行 shell。

1.  开始强制执行 SQLITE_LIMIT_VDBE_OP。例如，在接受来自不受信任用户的 SQL 查询的系统中，可以使用此选项防止过大的预编译语句。

1.  各种性能改进。

    **Bug 修复：**

1.  确保处理带有排序序列的索引表达式时的正确性。修复了 Ticket [eb703ba7b50c1a5](https://www.sqlite.org/src/info/eb703ba7b50c1a5)。

1.  修复了日期和时间函数中'start of ...'修饰符的错误。Ticket [6097cb92745327a1](https://www.sqlite.org/src/info/6097cb92745327a1)

1.  修复了复杂递归触发器中潜在的段错误，这是由于版本 3.15.0 中作为性能优化的一部分引入的 OP_Once 操作码中的错误导致的。Ticket [06796225f59c057c](https://www.sqlite.org/src/info/06796225f59c057c)

1.  在 RBU 扩展中，添加额外的同步操作，以避免断电后可能导致的数据损坏。

1.  嵌套 SQL 语句的 sqlite3_trace_v2()输出应始终以“--”注释标记开头。

    **哈希：**

1.  SQLITE_SOURCE_ID: "2017-03-28 18:48:43 424a0d380332858ee55bdebc4af3789f74e70a2b3ba1cf29d84b9b4bcf3e2e37"

1.  对于 sqlite3.c 的 SHA3-256：cbf322df1f76be57fb3be84f3da1fc71d1d3dfdb7e7c2757fb0ff630b3bc2e5d

### 2017-02-13 (3.17.0)

1.  R-Tree 扩展性能提升约 25%。

    1.  在可用时使用编译器内置函数（例如 __builtin_bswap32()或 _byteswap_ulong()）进行字节交换。

    1.  使用 sqlite3_blob 键/值访问对象，而不是 SQL 来从 R-Tree 节点中提取内容。

    1.  其他杂项增强，如循环展开。

1.  添加 SQLITE_DEFAULT_LOOKASIDE 编译时选项。

1.  将默认的 lookaside 大小从 512,125 增加到 1200,100，这样可以在每个连接中增加 56KB 的额外内存，同时提供更好的性能。对内存敏感的应用程序可以在编译时、启动时或运行时恢复旧的默认值。

1.  当可用时，使用编译器内建的 __builtin_sub_overflow()、__builtin_add_overflow()和 __builtin_mul_overflow()。 （可以通过 SQLITE_DISABLE_INTRINSIC 编译时选项省略所有编译器内建功能。）

1.  添加了 SQLITE_ENABLE_NULL_TRIM 编译时选项，这可以使某些应用程序的数据库文件显著减小，但可能与 SQLite 的旧版本不兼容。

1.  将 SQLITE_DEFAULT_PCACHE_INITSZ 从 100 改为 20，以提升性能。

1.  添加了 SQLITE_UINT64_TYPE 编译时选项，作为 SQLITE_INT64_TYPE 的类比。

1.  将一些 UPDATE 操作改为单遍执行，而不是两遍执行。

1.  增强会话扩展以支持 WITHOUT ROWID 表。

1.  修复了从具有数十万行的多行 VALUES 子句创建视图时的性能问题和潜在的堆栈溢出。

1.  添加了[sha1.c](https://www.sqlite.org/src/file/ext/misc/sha1.c)扩展。

1.  在命令行 Shell 中增强".mode"命令，使其对于"line"、"list"、"column"和"tcl"模式恢复默认的列和行分隔符。

1.  增强 SQLITE_DIRECT_OVERFLOW_READ 选项，使其在 WAL 模式下工作，只要被读取的页面不在 WAL 文件中。

1.  增强 Lemon 解析器生成器，使其能够将解析器对象存储为堆栈变量，而不是从堆中分配空间，并在汇编中利用该增强功能。

1.  其他性能改进。使用大约减少了 6.5%的 CPU 周期。

    **错误修复：**

1.  如果 LEFT JOIN 的 ON 子句引用了 ON 子句右侧的表，则抛出错误。这与 PostgreSQL 的行为相同。以前，SQLite 会悄悄地将 LEFT JOIN 转换为 INNER JOIN。修复了票号 [25e335f802dd](https://www.sqlite.org/src/info/25e335f802dd)。

1.  对于自动索引的列，使用正确的关联性。票号 [7ffd1ca1d2ad4ec](https://www.sqlite.org/src/info/7ffd1ca1d2ad4ec)。

1.  确保 sqlite3_blob_reopen() 接口能够正确处理短行。修复了票号 [e6e962d6b0f06f46e](https://www.sqlite.org/src/info/e6e962d6b0f06f46e)。

    **哈希值：**

1.  SQLITE_SOURCE_ID：“2017-02-13 16:02:40 ada05cfa86ad7f5645450ac7a2a21c9aa6e57d2c”

1.  sqlite3.c 的 SHA1 值为：cc7d708bb073c44102a59ed63ce6142da1f174d1

### 2017-01-06（3.16.2）

1.  修复了 REPLACE 语句在没有次要索引的 WITHOUT ROWID 表上的问题，以使其与触发器和外键正确工作。这是由于在版本 3.16.0 中添加的性能优化引起的新 bug。票号 [30027b613b4](https://www.sqlite.org/src/info/30027b613b4)

1.  修复了 sqlite3_value_text() 接口，使其能够正确地将 zeroblob() 生成的内容转换为全 0x00 字符的字符串。这是一个长期存在的问题，由 [OSS-Fuzz](https://github.com/google/oss-fuzz) 在 3.16.1 发布后发现。

1.  修复了字节码生成器以处理 FROM 子句中的子查询，该子查询本身是一个 UNION ALL，其中 UNION ALL 的一侧是包含 ORDER BY 的视图。这是一个长期存在的问题，在 3.16.1 发布后被发现。见票号 [190c2507](https://www.sqlite.org/src/info/190c2507)。

1.  调整了 sqlite3_column_count() API，以便在 PRAGMA 语句中更频繁地返回与之前版本相同的值，以减少对可能以意外方式使用该接口的应用程序的干扰。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-01-06 16:32:41 a65a62893ca8319e89e48b8a38cf8a59c69a8209"

1.  sqlite3.c 的 SHA1 值: 2bebdc3f24911c0d12b6d6c0123c3f84d6946b08

### 2017-01-03 (3.16.1)

1.  修复了关于 触发器 中使用 行值 的 bug（参见票号 [8c9458e7](https://www.sqlite.org/src/info/8c9458e7)，此问题出现在版本 3.15.0，但是在发布 3.16.0 后不久才报告）。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-01-03 18:27:03 979f04392853b8053817a3eea2fc679947b437fd"

1.  sqlite3.c 的 SHA1 值: 354f6223490b30fd5320b4066b1535e4ce33988d

### 2017-01-02 (3.16.0)

1.  使用的 CPU 循环数减少了 9%。（查看 CPU 性能测量报告，了解如何计算此性能增加的详细信息。）

1.  添加了对 PRAGMA 函数 的实验性支持。

1.  在 sqlite3_db_config() 中添加了 SQLITE_DBCONFIG_NO_CKPT_ON_CLOSE 选项。

1.  增强了 日期和时间函数，以便 'unixepoch' 修饰符适用于支持的日期的完整范围。

1.  将 lookaside 内存分配器的默认配置从每个 128 字节的 500 个槽更改为每个 512 字节的 125 个槽。

1.  增强了 "WHERE x NOT NULL" 的 部分索引，以便在 "x" 列出现在 LIKE 或 GLOB 运算符中时可以使用。

1.  增强了 sqlite3_interrupt()，使其中断正在进行的 checkpoint 操作。

1.  加强了 LIKE 和 GLOB 匹配算法，使得在模式包含多个通配符时更快。

1.  添加了 SQLITE_FCNTL_WIN32_GET_HANDLE 文件控制操作码。

1.  向 命令行 shell 添加了 ".mode quote"。

1.  向 命令行 shell 添加了 ".lint fkey-indexes"。

1.  向 命令行 shell 添加了 ".imposter dot-command"。

1.  添加了 [remember(V,PTR)](https://www.sqlite.org/src/file/ext/misc/remember.c) SQL 函数作为 可加载扩展。

1.  将 SQLITE_OMIT_BUILTIN_TEST 编译时选项重命名为 SQLITE_UNTESTABLE，以更好地反映使用它的含义。

    **Bug Fixes:**

1.  修复查询规划器中一个长期存在的 bug，在左连接中导致错误结果的问题，其中左侧表是一个子查询，连接约束来自左侧子查询的裸列名。票证 [2df0107b](https://www.sqlite.org/src/info/2df0107b)。

1.  正确处理查询规划器中的整数文字 -0x8000000000000000。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2017-01-02 11:57:58 04ac0b75b1716541b2b97704f4809cb7ef19cccf"

1.  sqlite3.c 的 SHA1 值为：e2920fb885569d14197c9b7958e6f1db573ee669

### 2016-11-28 (3.15.2)

1.  对版本 3.15.0 中引入的 行值 逻辑进行了多个 bug 修复。

1.  修复在 ATTACH/DETACH 后跟随恶意构造的语法错误可能导致的空指针解引用。票证 [2f1b168ab4d4844](https://www.sqlite.org/src/info/2f1b168ab4d4844)。

1.  修复内置函数 instr() 在内存不足条件下可能导致的崩溃。

1.  在 JSON 扩展 中，修复 JSON 验证器，使其能够正确拒绝字符串中的无效反斜杠转义。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2016-11-28 19:13:37 bbd85d235f7037c6a033a9690534391ffeacecc8"

1.  sqlite3.c 的 SHA1 值为：06d77b42a3e70609f8d4bbb97caf53652f1082cb

### 2016-11-04 (3.15.1)

1.  添加了 SQLITE_FCNTL_WIN32_GET_HANDLE 文件控制操作码。

1.  修正 VACUUM 命令，使其将多余内容写入磁盘，而非全部存储在内存中，从而避免较大数据库文件可能导致的内存不足错误。这修复了版本 3.15.0 引入的问题。

1.  修正了一个自 3.8.0（2013-08-26）以来存在的情况，在 LEFT JOIN 的 ON 子句中使用 OR 连接的术语可能导致结果不正确。Ticket [34a579141b2c5ac](https://www.sqlite.org/src/info/34a579141b2c5ac)。

1.  修正了一个情况，在 LEFT JOIN 的 ON 子句中使用 row values 可能导致结果不正确的问题。Ticket [fef4bb4bd9185ec8f](https://www.sqlite.org/src/info/fef4bb4bd9185ec8f)。

    **Hashes：**

1.  SQLITE_SOURCE_ID：“2016-11-04 12:08:49 1136863c76576110e710dd5d69ab6bf347c65e36”

1.  sqlite3.c 的 SHA1：e7c26a7be3e431dd06898f8d262c4ef240c07366

### 2016-10-14 (3.15.0)

1.  增加了对 row values 的支持。

1.  允许在 partial index 的 WHERE 子句中使用 deterministic SQL functions。

1.  在 unix VFS 上的“modeof=*filename*” URI 参数上添加了支持。

1.  增加了对 SQLITE_DBCONFIG_MAINDBNAME 的支持。

1.  增加了对 ATTACH-ed 数据库的 VACUUM 功能。

1.  命令行 shell 的增强：

    1.  添加“.testcase”和“.check” dot-commands。

    1.  在“.open” dot-command 中添加了“--new”选项，导致在打开之前清除数据库中的任何先前内容。

1.  增强了处理“ORDER BY term”的 fts5vocab 虚拟表的效率。

1.  杂项微优化可将常见工作负载的 CPU 使用率降低超过 7%。此版本中的大多数优化都集中在前端（sqlite3_prepare_v2()）。

    **Bug 修复：**

1.  现在，乘法运算符在所有边缘情况下都正确检测到 64 位整数溢出，并提升为浮点数。修复了 ticket [1ec41379c9c1e400](https://www.sqlite.org/src/info/1ec41379c9c1e400)。

1.  在使用 IN 操作符 时，正确处理具有冗余唯一索引的列。修复了 ticket [0eab1ac759](https://www.sqlite.org/src/info/0eab1ac759)。

1.  在表达式索引上，跳过范围查询中的 NULL 条目。修复了 ticket [4baa46491212947](https://www.sqlite.org/src/tktview/4baa46491212947)。

1.  确保在执行 "INSERT ... SELECT" 语句时，sqlite_sequence 表中的 AUTOINCREMENT 计数器通过 "Xfer 优化" 得到初始化。修复了 ticket [7b3328086a5c116c](https://www.sqlite.org/src/info/7b3328086a5c116c)。

1.  确保 ORDER BY LIMIT 优化（来自 check-in [559733b09e](https://www.sqlite.org/src/info/559733b09e9630fa)）在 INTEGER PRIMARY KEYs 上与 IN 操作符一起正常工作。修复了 ticket [96c1454c](https://www.sqlite.org/src/info/96c1454cbfd9509)。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2016-10-14 10:20:30 707875582fcba352b4906a595ad89198d84711d8"

1.  sqlite3.c 的 SHA1 哈希值为：fba106f8f6493c66eeed08a2dfff0907de54ae76

### 2016-09-12 (3.14.2)

1.  在 winsqlite3.dll 中改进了对 STDCALL 调用约定的支持。

1.  修复 sqlite3_trace_v2() 接口，确保在回调函数或掩码参数为零时禁用，与文档一致。

1.  修复了 EXPLAIN 列表中使用 -DSQLITE_ENABLE_EXPLAIN_COMMENTS 编译选项时生成的注释错误和改进。

1.  修复命令行 shell 中的 ".read" 命令，以便正确理解其非交互式输入。在 command-line shell 中进行修复。

1.  在 IN 操作符的 RHS 上计算选择的正确亲和性。修复了 ticket [199df4168c](https://sqlite.org/src/info/199df4168c)。

1.  ORDER BY LIMIT 优化只有在查询计划实际使用查询内最内层的 IN 运算符循环时才有效。修复了票务 [0c4df46116e90f92](https://sqlite.org/src/info/0c4df46116e90f92)。

1.  修复了导致一些 DELETE 操作无效的内部代码生成器问题。票务 [ef360601](https://sqlite.org/src/info/ef360601)

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2016-09-12 18:50:49 29dbef4b8585f753861a36d6dd102ca634197bd6"

1.  sqlite3.c 的 SHA1 值为：bcc4a1989db45e7f223191f2d0f66c1c28946383

### 2016-08-11 (3.14.1)

1.  对页面缓存“截断”操作的性能增强，可在具有大 页面缓存 的系统上减少 COMMIT 时间数十毫秒。

1.  修复了 sqldiff 的 --rbu 选项。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2016-08-11 18:53:32 a12d8059770df4bca59e321c266410344242bf7b"

1.  sqlite3.c 的 SHA1 值为：d545b24892278272ce4e40e0567d69c8babf12ea

### 2016-08-08 (3.14)

![](img/8337fe7fa3ad24e3f6ee1b8b32709296.png)

用一块自制的派庆祝 SQLite 的“π发布”。

1.  增加了对 WITHOUT ROWID 虚拟表 的支持。

1.  改进了查询规划器，以便即使一个或多个分离子使用 LIKE、GLOB、REGEXP、MATCH 运算符，也可以在 虚拟表 上使用 OR 优化。

1.  增加了用于读取基于 [RFC 4180](https://www.ietf.org/rfc/rfc4180.txt) 格式的逗号分隔值文件的 CSV 虚拟表。

1.  增加了 carray() 表值函数 扩展。

1.  使用扩展入口点的新返回码 SQLITE_OK_LOAD_PERMANENTLY 启用了持续可加载扩展。

1.  增加了 SQLITE_DBSTATUS_CACHE_USED_SHARED 选项到 sqlite3_db_status()。

1.  添加了[vfsstat.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/vfsstat.c)可加载扩展，这是一个 VFS shim，用于测量 I/O 操作，还有一个同名的 eponymous virtual table，提供对这些测量数据的访问。

1.  改进了同时具有 ORDER BY 和 LIMIT 的查询运行算法，在只有内层循环自然生成正确顺序行的情况下。

1.  增强了 Lemon parser generator，使其生成更快的解析器。

1.  PRAGMA compile_options 命令现在试图显示生成库的编译器版本号。

1.  增强了 PRAGMA table_info，使其提供有关 eponymous virtual tables 的信息。

1.  添加了"win32-none" VFS，类似于"default "win32" VFS，但会忽略所有文件锁。

1.  查询规划器在合适的情况下使用 partial index 的全扫描，而不是主表的全扫描。

1.  允许 table-valued functions 出现在 IN 操作符的右侧。

1.  创建了 dbhash.exe 命令行实用程序。

1.  增加了两个新的 C 语言接口：sqlite3_expanded_sql() 和 sqlite3_trace_v2()。这些新接口替代了现已弃用的 sqlite3_trace() 和 sqlite3_profile() 的功能。

1.  将 json_quote() SQL 函数添加到 json1 扩展中。

1.  禁用在重新解析模式时的授权回调。

1.  添加了 SQLITE_ENABLE_UNKNOWN_SQL_FUNCTION 编译时选项，并在构建 command-line shell 时默认启用该选项。

    **Bug 修复：**

1.  修复了 ALTER TABLE 命令，以便在向 legacy file format 数据库添加列时不会破坏降序索引。票证[f68bf68513a1c15f](https://www.sqlite.org/src/info/f68bf68513a1c15f)

1.  修复了当传递的 WHERE 子句引用不存在的排序序列时可能发生的 NULL 指针解引用/崩溃。票证[e8d439c77685eca6](https://www.sqlite.org/src/info/e8d439c77685eca6)。

1.  改进了索引扫描的成本估算，其中包括可以使用索引中的列部分或完全评估的 WHERE 子句，而无需进行表查找。这修复了在版本 3.12.0 引入的 ORDER BY LIMIT 优化后某些晦涩查询的性能回归问题。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2016-08-08 13:40:27 d5e98057028abcf7217d0d2b2e29bbbcdf09d6de"

1.  sqlite3.c 的 SHA1 值为：234a3275d03a287434ace3ccdf1afb208e6b0e92

### 2016-05-18 (3.13.0)

1.  尽可能地推迟与 TEMP 文件相关的 I/O 操作，希望最终能完全避免 I/O。

1.  将 session 扩展合并到主干中。

1.  为命令行 shell 添加了".auth ON|OFF"命令。

1.  为命令行 shell 的".schema"和".fullschema"命令添加了"--indent"选项，以进行漂亮的打印输出。

1.  为命令行 shell 添加了".eqp full"选项，它在每个评估的语句上执行 EXPLAIN 和 EXPLAIN QUERY PLAN。

1.  在 Windows 平台上改进了对 Unicode 文件名的处理命令行 shell。

1.  改进了对应用程序不完整或不正确修改的 sqlite_stat1 表所导致的愚蠢查询规划决策的抵抗力。

1.  添加了 sqlite3_db_config(db,SQLITE_DBCONFIG_ENABLE_LOAD_EXTENSION)接口，允许在保持 sqlite3_load_extension() C-API 启用的同时禁用 load_extension() SQL 函数以提升安全性。

1.  更改 Unix 上的临时目录搜索算法，以允许具有写和执行权限但没有读权限的目录作为临时目录。将此标准应用于“.”回退目录。

    **错误修复：**

1.  修复了多行单遍 DELETE 优化中的问题，该问题导致在 WHERE 子句中存在自引用子查询时计算出错误答案。修复票证[dc6ebeda9396087](https://www.sqlite.org/src/info/dc6ebeda9396087)

1.  修复了当表是带有 INTEGER PRIMARY KEY 和 WHERE 子句包含 OR 且表具有一个或多个能触发 OR 优化但没有引用任何表列的索引时，DELETE 可能导致段错误的问题。票证[16c9801ceba49](https://www.sqlite.org/src/info/16c9801ceba49)。

1.  在检查 WHERE 子句推送优化时，请验证复合内部 SELECT 的所有项均为非聚合项，而不仅仅是最后一项。修复票证[f7f8c97e97597](https://www.sqlite.org/src/info/f7f8c97e97597)。

1.  修复了 Windows 中的锁竞争条件，在两个或更多进程尝试同时恢复相同热日志时可能发生的情况。

    **哈希：**

1.  SQLITE_SOURCE_ID："2016-05-18 10:57:30 fc49f556e48970561d7ab6a2f24fdd7d9eb81ff2"

1.  SHA1 for sqlite3.c: 9b9171b1e6ce7a980e6b714e9c0d9112657ad552

    **修复的错误被回退到补丁版本 3.12.2（2016-04-18）：**

1.  修复了版本 3.12.0 和 3.12.1 中的向后兼容性问题：声明为`"INTEGER" PRIMARY KEY`（在数据类型关键字周围带有引号）的列未被识别为 INTEGER PRIMARY KEY，导致不兼容的数据库文件。票证 [7d7525cb01b68](https://www.sqlite.org/src/info/7d7525cb01b68)

1.  修复一个 bug（自版本 3.9.0 以来存在），可能导致 DELETE 操作在打开 PRAGMA reverse_unordered_selects 时丢失行。票证 [a306e56ff68b8fa5](https://www.sqlite.org/src/info/a306e56ff68b8fa5)

1.  修复代码生成器中的一个 bug，如果两个或更多的虚拟表被连接，并且在连接的外部循环中使用的虚拟表具有 IN 运算符约束，则可能导致结果不正确。

1.  在确定用于排序大量数据的缓存大小时，正确解释负的“PRAGMA cache_size”值。

    **错误修复已回溯到补丁版本 3.12.1（2016-04-08）：**

1.  修复版本 3.12.0 引入的边界条件错误，可能在大量使用 SAVEPOINT 时导致崩溃。票证 [7f7f8026eda38](https://www.sqlite.org/src/info/7f7f8026eda38).

1.  修复视图，使其在可能的情况下从定义它们的表继承列数据类型。

1.  修复查询规划器，使得 IS 和 IS NULL 运算符能够驱动左外连接上的索引。

### 2016-04-18 (3.12.2)

1.  修复了版本 3.12.0 和 3.12.1 中的向后兼容性问题：声明为`"INTEGER" PRIMARY KEY`（在数据类型关键字周围带有引号）的列未被识别为 INTEGER PRIMARY KEY，导致不兼容的数据库文件。票证 [7d7525cb01b68](https://www.sqlite.org/src/info/7d7525cb01b68)

1.  修复一个 bug（自 版本 3.9.0 起存在），可能导致 DELETE 操作在打开 PRAGMA reverse_unordered_selects 的情况下丢失行。Ticket [a306e56ff68b8fa5](https://www.sqlite.org/src/info/a306e56ff68b8fa5)

1.  修复代码生成器中的一个 bug，如果两个或更多 虚拟表 被连接，并且连接的外部循环中使用的虚拟表具有 IN 操作符 约束，则可能导致结果不正确。

1.  在确定用于排序大量数据的缓存大小时，正确解释负 "PRAGMA cache_size" 值。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2016-04-18 17:30:31 92dc59fd5ad66f646666042eb04195e3a61a9e8e"

1.  SHA1 for sqlite3.c: de5a5898ebd3a3477d4652db143746d008b24c83

### 2016-04-08 (3.12.1)

1.  修复由版本 3.12.0 引入的一个边界条件错误，可能导致在大量使用 SAVEPOINT 时崩溃。Ticket [7f7f8026eda38](https://www.sqlite.org/src/info/7f7f8026eda38)。

1.  修复 视图，使其在可能的情况下从定义它们的表继承列数据类型。

1.  修复查询规划器，使得 IS 和 IS NULL 操作符能够驱动 LEFT OUTER JOIN 上的索引。

    **哈希值：**

1.  SQLITE_SOURCE_ID: "2016-04-08 15:09:49 fe7d3b75fe1bde41511b323925af8ae1b910bc4d"

1.  SHA1 for sqlite3.c: ebb18593350779850e3e1a930eb84a70fca8c1d1

### 2016-04-01 (3.9.3)

1.  将一个 [简单的查询规划器优化](https://www.sqlite.org/src/info/c648539b52ca28c0) 回退，允许 IS 操作符驱动 LEFT OUTER JOIN 上的索引。与 版本 3.9.2 基线没有其他变化。

### 2016-03-29 (3.12.0)

**可能会造成破坏性变更：**

1.  SQLITE_DEFAULT_PAGE_SIZE 从 1024 增加到 4096。SQLITE_DEFAULT_CACHE_SIZE 从 2000 改变为 -2000，因此默认使用相同数量的缓存内存。请查看关于 版本 3.12.0 页面大小更改 的应用注释以获取更多信息。

    **性能增强：**

1.  对 Lemon 解析生成器 进行了增强，使其创建更小、更快的 SQL 解析器。

1.  仅当两个或更多附加数据库都被修改，PRAGMA synchronous 未设置为 OFF，并且 journal_mode 未设置为 OFF、MEMORY 或 WAL 时，才会创建 主日志 文件。

1.  仅当其大小超过阈值时才会创建 语句日志 文件。否则，日志将保存在内存中，并且不会发生 I/O。可以在编译时使用 SQLITE_STMTJRNL_SPILL 或在启动时使用 sqlite3_config(SQLITE_CONFIG_STMTJRNL_SPILL) 配置阈值。

1.  查询规划器能够优化 IN 操作符在 虚拟表 上的应用，即使 xBestIndex 方法未将虚拟表列的 sqlite3_index_constraint_usage.omit 标志设置为 IN 操作符左侧。

1.  查询规划器现在在 3 路或更高连接中更好地优化 虚拟表 访问，其中虚拟表的约束条件分布在连接的两个或更多其他表之间。

1.  更有效地处理 应用定义的 SQL 函数，特别是在应用程序定义了数百或数千个自定义函数的情况下。

1.  查询规划器在估算 ORDER BY 的成本时会考虑 LIMIT 子句。

1.  配置脚本（在 unix 上）现在自动检测 pread() 和 pwrite()，并设置编译时选项以使用这些操作系统接口（如果可用）。

1.  减少了保存模式所需的内存量。

1.  其他杂项微优化，以提升性能并减少内存使用。

    **新功能：**

1.  添加了 SQLITE_DBCONFIG_ENABLE_FTS3_TOKENIZER 选项到 sqlite3_db_config()，允许在运行时启用或禁用 fts3_tokenizer() SQL 函数的两参数版本。

1.  已添加 [sqlite3rbu_bp_progress()](https://www.sqlite.org/src/artifact/d7cc99350?ln=403-443) 接口到 RBU 扩展中。

1.  PRAGMA defer_foreign_keys=ON 语句现在还会禁用外键的 RESTRICT actions。

1.  添加了 sqlite3_system_errno() 接口。

1.  添加了 SQLITE_DEFAULT_SYNCHRONOUS 和 SQLITE_DEFAULT_WAL_SYNCHRONOUS 编译时选项。SQLITE_DEFAULT_SYNCHRONOUS 编译时选项取代了不再支持的 SQLITE_EXTRA_DURABLE 选项。

1.  增强了 命令行 shell 中的 ".stats" 命令，显示从 /proc 获得的更多 I/O 性能信息（如果可用）。

    **错误修复：**

1.  确保来自单个语句中多个触发器的 sqlite3_set_auxdata() 值不会相互干扰。Ticket [dc9b1c91](https://www.sqlite.org/src/info/dc9b1c91)。

1.  修复了代码生成器，以支持形如 "x IN (SELECT...)" 的表达式，其中右侧的 SELECT 语句是相关子查询。Ticket [5e3c886796e5512e](https://www.sqlite.org/src/info/5e3c886796e5512e)。

1.  修复了与 sqlite3_db_readonly() 接口相关的一个无害的 TSAN 警告。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2016-03-29 10:14:15 e9bb4cf40f4971974a74468ef922bdee481c988b"

1.  sqlite3.c 的 SHA1：cba2be96d27cb51978cd4a200397a4ad178986eb

### 2016-03-03 (3.11.1)

1.  改进了 Makefile 和 VisualStudio 使用的构建脚本。

1.  修复了 FTS5 中 'optimize' 命令可能导致索引损坏的问题。

1.  修复了如果使用 FTS5 查询损坏的数据库文件可能发生的缓冲区过读的问题。

1.  将 spellfix1 扩展的最大 "scope" 值从 6 增加到 30。

1.  SQLITE_SOURCE_ID: "2016-03-03 16:17:53 f047920ce16971e573bc6ec9a48b118c9de2b3a7"

1.  sqlite3.c 的 SHA1：3da832fd2af36eaedb05d61a8f4c2bb9f3d54265

### 2016-02-15 (3.11.0)

**一般改进：**

1.  加强了 WAL mode，以便与大于 cache_size 的事务高效工作。

1.  添加了 FTS5 detail option。

1.  在 PRAGMA synchronous 中添加了 "EXTRA" 选项，在 DELETE 模式下取消回滚日志时，会对包含目录进行同步，以提高耐久性。SQLITE_EXTRA_DURABLE 编译时选项默认启用 PRAGMA synchronous=EXTRA。

1.  加强了 query planner ，使其能够在 OR optimization 的一部分作为 covering index 使用。

1.  避免在 UPDATE 语句中对未更改列重新计算 NOT NULL 和 CHECK constraints。

1.  进行了许多微优化，使得该库比上一个版本更快。

    **增强了 command-line shell：**

1.  默认情况下，shell 现在处于 "auto-explain" 模式。EXPLAIN 命令的输出会自动格式化。

1.  添加了 ".vfslist" dot-command。

1.  编译时选项 SQLITE_ENABLE_EXPLAIN_COMMENTS 现在在标准构建中默认开启。

    **增强了 TCL Interface：**

1.  如果使用 "-uri 1" 选项打开数据库连接，则 "backup" 和 "restore" 命令将遵循 URI filenames。

1.  添加了 "-sourceid" 选项到 "sqlite3" 命令。

    **Makefile 改进：**

1.  在配置脚本中改进了 pthreads 的检测。

1.  添加了从 amalgamation tarball 进行 MSVC Windows 构建的能力。

    **Bug 修复：**

1.  修复了在协程之间错误共享 VDBE 临时寄存器的问题，这可能导致在特定情况下查询结果不正确。Ticket [d06a25c84454a](https://www.sqlite.org/src/info/d06a25c84454a)。

1.  修复了在特定情况下可能导致 json1 扩展下的 sqlite3_result_subtype() 接口问题。修复 ticket [f45ac567eaa9f9](https://www.sqlite.org/src/info/f45ac567eaa9f9)。

1.  在 JSON 字符串中转义控制字符。修复 ticket [ad2559db380abf8](https://www.sqlite.org/src/info/ad2559db380abf8)。

1.  只要未定义 SQLITE_OMIT_DEPRECATED，则在内置的 unix VFSes 中重新启用 xCurrentTime 和 xGetLastError 方法。

    **向后兼容性：**

1.  由于持续的安全性问题，除非 SQLite 使用 SQLITE_ENABLE_FTS3_TOKENIZER 编译，否则禁用了不常用和少知名的 fts3_tokenizer() 函数的两参数版本。

    **哈希值：**

1.  SQLITE_SOURCE_ID："2016-02-15 17:29:24 3d862f207e3adc00f78066799ac5a8c282430a5f"。

1.  sqlite3.c 的 SHA1 值为 df01436c5fcfe72d1a95bc172158219796e1a90b。

### 2016-01-20 (3.10.2)

**关键错误修复:**

1.  版本 3.10.0 在 LIKE 操作符中引入了大小写折叠错误，此补丁版本修复了该问题。Ticket [80369eddd5c94](https://www.sqlite.org/src/info/80369eddd5c94)。

    **其他杂项错误修复:**

1.  修复了一个使用-DSQLITE_HAS_CODEC 编译时可能出现的使用后释放问题。

1.  修复了使用-DSQLITE_OMIT_WAL 编译时构建失败的问题。

1.  修复了汇编中的配置脚本，以便在树莓派上再次使用--readline 选项。

    **哈希:**

1.  SQLITE_SOURCE_ID: "2016-01-20 15:27:19 17efb4209f97fb4971656086b138599a91a75ff9"

1.  sqlite3.c 的 SHA1 值为: f7088b19d97cd7a1c805ee95c696abd54f01de4f

### 2016-01-14 (3.10.1)

**新功能:**

1.  添加了 SQLITE_FCNTL_JOURNAL_POINTER 文件控制。

    **错误修复:**

1.  修复了查询规划器中 16 个月前的一个 bug，该 bug 可能在标量子查询尝试使用块排序优化时生成不正确的结果。Ticket [cb3aa0641d9a4](https://www.sqlite.org/src/info/cb3aa0641d9a4)。

    **哈希:**

1.  SQLITE_SOURCE_ID: "2016-01-13 21:41:56 254419c36766225ca542ae873ed38255e3fb8588"

1.  sqlite3.c 的 SHA1 值为: 1398ba8e4043550a533cdd0834bfdad1c9eab0f4

### 2016-01-06 (3.10.0)

**一般改进:**

1.  增加了对 virtual tables 上 LIKE、GLOB 和 REGEXP 操作符的支持。

1.  在 sqlite3_index_info 中为 sqlite3_module.xBestIndex 方法添加了 colUsed 字段。

1.  增强了 PRAGMA cache_spill 语句，接受一个 32 位整数参数，该参数是禁止溢出缓存的阈值。

1.  在 unix 系统上，如果打开数据库文件的是一个符号链接，则相应的日志文件基于实际文件名，而不是符号链接名。

1.  添加了 "--transaction" 选项到 sqldiff。

1.  添加了 sqlite3_db_cacheflush() 接口。

1.  添加了 sqlite3_strlike() 接口。

1.  在使用 内存映射 I/O 时，将数据库文件映射为只读，以防应用程序中的杂散指针和/或数组溢出意外修改数据库文件。

1.  添加了 *实验性* 的 sqlite3_snapshot_get()，sqlite3_snapshot_open() 和 sqlite3_snapshot_free() 接口。这些内容可能在后续版本中更改或移除。

1.  加强了 日期和时间函数 中的 'utc' 修饰符，使其在日期/时间已知为 UTC 时成为无操作。 （这不会破坏兼容性，因为长期以来在这种情况下的行为一直被文档化为“未定义”。）

1.  在 json 扩展中添加了 json_group_array() 和 json_group_object() SQL 函数。

1.  添加了 SQLITE_LIKE_DOESNT_MATCH_BLOBS 编译时选项。

1.  许多小的性能优化。

    **可移植性增强：**

1.  解决了 HP/UX 上 HP C 编译器优化器中的符号扩展 bug。 [(详情)](https://www.sqlite.org/src/fdiff?sbs=1&v1=869c95b0fc73026d&v2=232c242a0ccb3d67)

    **命令行 shell 的增强功能：**

1.  添加了 ".changes ON|OFF" 和 ".vfsinfo" 点命令。

1.  在 Windows 的 [cmd.exe](https://en.wikipedia.org/wiki/Cmd.exe) 中运行时进行 MBCS 和 UTF8 之间的转换。

    **Makefile 的增强功能：**

1.  向各种由 autoconf 生成的 configure 脚本添加了 --enable-editline 和 --enable-static-shell 选项。

1.  在 makefile 中省略了所有对 "awk" 的使用，以便于 MSVC 用户更轻松地构建。

    **重要修复：**

1.  修复了不一致的整数与浮点数比较操作，如果索引在表列上创建，该列包含相似幅度的大整数和浮点数值，则可能导致索引损坏。票号[38a97a87a6](https://www.sqlite.org/src/tktview?name=38a97a87a6)。

1.  修复查询规划器中的无限循环问题，可能在格式不正确的常规表达式中发生。

1.  在 sqldiff 工具中修复了各种错误。

    **哈希：**

1.  SQLITE_SOURCE_ID："2016-01-06 11:01:07 fd0a50f0797d154fefff724624f00548b5320566"

1.  sqlite3.c 的 SHA1：b92ca988ebb6df02ac0c8f866dbf3256740408ac

### 2015-11-02（3.9.2）

1.  修复模式解析器，使其解释某些（模糊且格式不正确的）CREATE TABLE 语句与遗留相同。修复了票号[ac661962a2aeab3c331](https://www.sqlite.org/src/info/ac661962a2aeab3c331)

1.  修复了查询规划器的问题，该问题可能由于在相关标量子查询的 FROM 子句中使用自动索引而导致答案不正确。修复了票号[8a2adec1](https://www.sqlite.org/src/info/8a2adec1)。

1.  SQLITE_SOURCE_ID："2015-11-02 18:31:45 bda77dda9697c463c3d0704014d51627fceee328"

1.  sqlite3.c 的 SHA1：1c4013876f50bbaa3e6f0f98e0147c76287684c1

### 2015-10-16（3.9.1）

1.  修复 json1 扩展，使其不再将 ASCII 换页符识别为空白字符，以符合 RFC-7159。修复了票号[57eec374ae1d0a1d](https://www.sqlite.org/src/info/57eec374ae1d0a1d)

1.  添加了一些#ifdef 和构建脚本更改，以解决在 3.9.0 发布后出现的编译问题。

1.  SQLITE_SOURCE_ID："2015-10-16 17:31:12 767c1727fec4ce11b83f25b3f1bfcfe68a2c8b02"

1.  sqlite3.c 的 SHA1：5e6d1873a32d82c2cf8581f143649940cac8ae49

### 2015-10-14（3.9.0）

**政策更改：**

1.  SQLite 的版本编号约定已修订为使用[语义版本控制](http://semver.org/)的新标准。

    **新功能和增强：**

1.  将 json1 扩展 模块添加到源树和 amalgamation 中。通过编译选项 SQLITE_ENABLE_JSON1 启用支持。

1.  将 Full Text Search version 5 (FTS5) 添加到 amalgamation，并使用 SQLITE_ENABLE_FTS5 启用。FTS5 在至少一个发布周期内将被视为“实验性”（可能有不兼容的更改）。

1.  CREATE VIEW 语句现在接受视图名称后面的可选列名列表。

1.  增加了对表达式索引（indexes on expressions）的支持。

1.  在 SELECT 语句的 FROM 子句中，增加了对表值函数（table-valued functions）的支持。

1.  增加了对 eponymous 虚拟表 的支持。

1.  现在，当创建一个视图（VIEW）时，可以引用未定义的表和函数。当视图在查询中使用时，会报告缺少的表和函数。

1.  添加了 sqlite3_value_subtype() 和 sqlite3_result_subtype() 接口（由 json1 扩展 使用）。

1.  查询规划器现在可以使用包含 WHERE 子句中 AND 连接项的 partial indexes。

1.  更新了 sqlite3_analyzer.exe 实用工具，报告每个 B 树的深度，并显示索引和 WITHOUT ROWID 表的平均扇出。

1.  增强了 dbstat 虚拟表，使其可以作为 table-valued function 使用，其中参数是要分析的模式。

    **其他变更：**

1.  已弃用并未记录 8 年的接口 sqlite3_memory_alarm() 现在变成了一个空操作（no-op）。

    **重要修复：**

1.  修复了 [SQLite Encryption Extension](https://www.sqlite.org/see/doc/trunk/www/readme.wiki) 中的一个严重错误，可能会导致数据库在 VACUUM 命令改变加密随机数大小时变得不可读且无法恢复。

1.  在 sqlite3_initialize() 的实现中增加了内存屏障，以帮助确保其线程安全。

1.  修复了 OR 优化，以便始终忽略不使用索引的子计划。

1.  不要对来自 LEFT JOIN 的 ON 或 USING 子句中源自的条件应用 WHERE-clause push-down optimization。修复了票号 [c2a19d81652f40568c](https://www.sqlite.org/src/info/c2a19d81652f40568c) 的问题。

1.  SQLITE_SOURCE_ID: "2015-10-14 12:29:53 a721fc0d89495518fe5612e2e3bbc60befd2e90d"

1.  sqlite3.c 的 SHA1 值为：c03e47e152ddb9c342b84ffb39448bf4a2bd4288

### 2015-07-29 (3.8.11.1)

1.  恢复了 PRAGMA cache_size 的一个未记录的副作用：如果数据库之前未被访问，则强制解析数据库模式。

1.  修复了在 WITHOUT ROWID 表中存在已久的问题，该问题是在 3.8.11 发布几小时后报告的，涉及 sqlite3_changes()。

1.  SQLITE_SOURCE_ID: "2015-07-29 20:00:57 cf538e2783e468bbc25e7cb2a9ee64d3e0e80b2f"

1.  sqlite3.c 的 SHA1 值为：3be71d99121fe5b17f057011025bcf84e7cc6c84

### 2015-07-27 (3.8.11)

1.  添加了实验性的 RBU 扩展。请注意，此扩展是实验性的，可能会以不兼容的方式更改。

1.  添加了实验性的 FTS5 扩展。请注意，此扩展是实验性的，可能会以不兼容的方式更改。

1.  添加了 sqlite3_value_dup() 和 sqlite3_value_free() 接口。

1.  增强了 spellfix1 扩展以支持 ON CONFLICT 子句。

1.  IS 操作符 现在能够驱动索引。

1.  增强查询规划器，允许在由协同程序实现的 FROM 子句子查询上进行 自动索引。

1.  不允许在 通用表达式 中使用 "rowid"。

1.  添加了 PRAGMA cell_size_check 命令，以更好和更早地检测数据库文件损坏。

1.  在 FTS3 的 matchinfo() 函数中添加了 matchinfo 'b' flag。

1.  改进数据库文件的模糊测试，修复发现的问题。

1.  添加模糊检查测试程序，并在 "make test" 中自动运行此程序，使用 SQL 和数据库测试案例。

1.  添加了 SQLITE_MUTEX_STATIC_VFS1 静态互斥体，并在 Windows VFS 中使用它。

1.  当未完成执行的语句被 sqlite3_reset() 或 sqlite3_finalize() 调用时，将调用 sqlite3_profile() 回调。

1.  加强页面缓存，使其能够预先分配一块内存用于初始设置页面缓存行。将默认预分配设置为 100 页。在常见工作负载下性能提升约 5%。

1.  各种微小的优化导致相对于上一个版本，同样数量的 CPU 循环执行更多工作量增加了 22.3%。SQLite 现在比 版本 3.8.0 快两倍，比 版本 3.3.9 快三倍。（在 Ubuntu 14.04 x64 上使用 gcc 4.8.2 和 -Os，使用 [cachegrind](http://valgrind.org/docs/manual/cg-manual.html) 在 [speedtest1.c](https://www.sqlite.org/src/artifact/83f6b3318f7ee) 工作负载上测量。您的性能可能有所不同。）

1.  添加了 sqlite3_result_zeroblob64() 和 sqlite3_bind_zeroblob64() 接口。

    **重要的错误修复：**

1.  修复 CREATE TABLE AS，以确保 TEXT 类型的列不会最终持有 INT 值。票证 [f2ad7de056ab1dc9200](https://www.sqlite.org/src/info/f2ad7de056ab1dc9200)

1.  修复 CREATE TABLE AS，确保在右侧 SELECT 语句因错误而中止时不在 sqlite_master 表 中留下 NULL 条目。票证 [873cae2b6e25b](https://www.sqlite.org/src/info/873cae2b6e25b)

1.  修复 skip-scan 优化，以便在 OR 优化 在 WITHOUT ROWID 表上使用时正确工作。票证 [8fd39115d8f46](https://www.sqlite.org/src/info/8fd39115d8f46)

1.  修复 sqlite3_memory_used() 和 sqlite3_memory_highwater() 接口，确保它们实际上提供一个 64 位的答案。

    **Hashes:**

1.  SQLITE_SOURCE_ID: "2015-07-27 13:49:41 b8e92227a469de677a66da62e4361f099c0b79d0"

1.  sqlite3.c 的 SHA1 值: 719f6891abcd9c459b5460b191d731cd12a3643e

### 2015-05-20 (3.8.10.2)

1.  修复由 版本 3.8.7 引入的索引损坏问题。如果表有两个嵌套触发器，将键值转换为 INTEGER，然后再转换回 TEXT，索引中带有 TEXT 键的索引可能会因为 INSERT 到相应的表中而损坏。票证 [34cd55d68e0](https://www.sqlite.org/src/info/34cd55d68e0e6e7c9a0711aab81a2ee3c354b4c0)

1.  SQLITE_SOURCE_ID: "2015-05-20 18:17:19 2ef4f3a5b1d1d0c4338f8243d40a2452cc1f7fe4"

1.  sqlite3.c 的 SHA1 值: 638abb77965332c956dbbd2c8e4248e84da4eb63

### 2015-05-09 (3.8.10.1)

1.  使 sqlite3_compileoption_used() 响应 SQLITE_ENABLE_DBSTAT_VTAB 编译时选项。

1.  修复在某些 MSVC 版本的 命令行 shell 中的一个无害警告。

1.  修复 dbstat 虚拟表 的一些次要问题。

1.  SQLITE_SOURCE_ID: "2015-05-09 12:14:55 05b4b1f2a937c06c90db70c09890038f6c98ec40"

1.  sqlite3.c 的 SHA1 值为：85e4e1c08c7df28ef61bb9759a0d466e0eefbaa2

### 2015-05-07 (3.8.10)

1.  添加了用于计算两个 SQLite 数据库文件差异的 sqldiff.exe 实用程序程序。

1.  将 matchinfo y flag 添加到 FTS3 的 matchinfo()函数中。

1.  ORDER BY，VACUUM，CREATE INDEX，PRAGMA integrity_check 和 PRAGMA quick_check 的性能改进。

1.  修复了在 SQL 模糊测试期间发现的许多隐晦问题。

1.  在接口文档中标识重要对象的所有方法。(示例)

1.  将 American Fuzzy Lop fuzzer 作为 SQLite 的测试策略的标准组成部分。

1.  将“.binary”和“.limits”命令添加到命令行 shell。

1.  在使用 SQLITE_ENABLE_DBSTAT_VTAB 选项编译时，将 dbstat 虚拟表作为标准构建的一部分。

1.  SQLITE_SOURCE_ID: "2015-05-07 11:53:08 cf975957b9ae671f34bb65f049acf351e650d437"

1.  sqlite3.c 的 SHA1 值为：0b34f0de356a3f21b9dfc761f3b7821b6353c570

### 2015-04-08 (3.8.9)

1.  将[VxWorks-7](https://wiki.example.org/vxworks-7)添加为官方支持和测试平台。

1.  添加了 sqlite3_status64()接口。

1.  修复内存大小跟踪问题，即使 SQLite 使用超过 2GiB 的内存也能正常工作。

1.  添加了 PRAGMA index_xinfo 命令。

1.  修复 sqlite3_blob_read()和 sqlite3_blob_write()接口中潜在的 32 位整数溢出问题。

1.  确保在使用 SQLITE_OMIT_AUTORESET 编译时，即使在 SQLITE_BUSY 和 SQLITE_LOCKED 的扩展错误码下，准备好的语句也会自动重置。

1.  修正 sqlite3_analyzer.exe 实用程序中关于 WITHOUT ROWID 表的错误计数。

1.  将 ".dbinfo" 命令添加到 命令行 shell。

1.  提高 fts3/4 查询的性能，这些查询使用 OR 运算符和至少一个辅助 fts 函数。

1.  修复 fts3 snippet() 函数中的一个 bug，导致它在以列中的第一个标记开头的片段中省略前导分隔符字符。

1.  SQLITE_SOURCE_ID："2015-04-08 12:16:33 8a8ffc862e96f57aa698f93de10dee28e69f6e09"

1.  sqlite3.c 的 SHA1 值：49f1c3ae347e1327b5aaa6c7f76126bdf09c6f42

### 2015-02-25 (3.8.8.3)

1.  修复一个 bug（ticket [2326c258d02ead33](https://www.sqlite.org/src/info/2326c258d02ead33)），该 bug 可导致在 LEFT JOIN 的 ON 子句中出现部分索引的限定约束时产生不正确的结果。

1.  在 unix 版本的 命令行 shell 中添加了与 "[linenoise](https://github.com/antirez/linenoise)" 命令行编辑库链接的功能。

1.  SQLITE_SOURCE_ID："2015-02-25 13:29:11 9d6c1880fb75660bbabd693175579529785f8a6b"

1.  sqlite3.c 的 SHA1 值：74ee38c8c6fd175ec85a47276dfcefe8a262827a

### 2015-01-30 (3.8.8.2)

1.  改进 sqlite3_wal_checkpoint_v2(TRUNCATE) 接口，以便即使没有需要执行的检查点工作，也可以截断 WAL 文件。

1.  SQLITE_SOURCE_ID："2015-01-30 14:30:45 7757fc721220e136620a89c9d28247f28bbbc098"

1.  sqlite3.c 的 SHA1 值：85ce79948116aa9a087ec345c9d2ce2c1d3cd8af

### 2015-01-20 (3.8.8.1)

1.  修复排序逻辑中的一个 bug，自版本 3.8.4 以来存在，该 bug 可导致在查询结果集中包含 ORDER BY 子句、LIMIT 子句，并且大约有 60 个或更多列时，输出显示顺序错误。Ticket [f97c4637102a3ae72b79](https://www.sqlite.org/src/tktview?name=f97c4637102a3ae72b79)。

1.  SQLITE_SOURCE_ID: "2015-01-20 16:51:25 f73337e3e289915a76ca96e7a05a1a8d4e890d55"

1.  SHA1 for sqlite3.c: 33987fb50dcc09f1429a653d6b47672f5a96f19e

### 2015-01-16 (3.8.8)

**新功能：**

1.  添加了可以用来确定数据库文件是否被另一个进程修改的 PRAGMA data_version 命令。

1.  在 sqlite3_wal_checkpoint_v2()接口中添加了 SQLITE_CHECKPOINT_TRUNCATE 选项，并相应增强了 PRAGMA wal_checkpoint。

1.  添加了 sqlite3_stmt_scanstatus()接口，仅在使用 SQLITE_ENABLE_STMT_SCANSTATUS 编译时可用。

1.  sqlite3_table_column_metadata()增强以在 WITHOUT ROWID 表上正常工作，并在列名参数为 NULL 时检查表的存在。 现在，该接口默认包含在构建中，无需 SQLITE_ENABLE_COLUMN_METADATA 编译时选项。

1.  添加了 SQLITE_ENABLE_API_ARMOR 编译时选项。

1.  添加了 SQLITE_REVERSE_UNORDERED_SELECTS 编译时选项。

1.  添加了 SQLITE_SORTER_PMASZ 编译时选项和 SQLITE_CONFIG_PMASZ 启动时选项。

1.  在 sqlite3_config()中添加了 SQLITE_CONFIG_PCACHE_HDRSZ 选项，使应用程序更容易确定用于 SQLITE_CONFIG_PAGECACHE 的适当内存量。

1.  VALUES 子句中的行数不再受 SQLITE_LIMIT_COMPOUND_SELECT 的限制。

1.  添加了[eval.c](https://www.sqlite.org/src/artifact/f971962e92ebb8b0) 可加载扩展，实现了一个递归评估 SQL 的 eval() SQL 函数。

    **性能增强：**

1.  减少在平衡 B 树过程中涉及的`memcpy()`操作数量，总体性能提升 3.2%。

1.  改进了跳跃扫描优化(skip-scan optimization)的成本估算。

1.  如果适用，自动索引优化现在能够生成部分索引。

    **错误修复：**

1.  确保在断电后通过调用`fsync()`截断日志文件后设置"PRAGMA journal_mode=TRUNCATE"以保证耐久性。

1.  查询规划器现在意识到在 LEFT JOIN 的右表中的任何列都可能为 NULL，即使该列有 NOT NULL 约束。在这些情况下避免尝试优化 NULL 测试。修复 Ticket [6f2222d550f5b0ee7ed](https://www.sqlite.org/src/info/6f2222d550f5b0ee7ed)。

1.  确保 ORDER BY 在使用降序索引实现 DISTINCT 运算符时也能按升序放置行。修复 Ticket [c5ea805691bfc4204b1cb9e](https://www.sqlite.org/src/info/c5ea805691bfc4204b1cb9e)。

1.  修复在共享缓存模式下运行时在压力下可能出现的数据竞态条件，其中一些线程正在打开和关闭连接。

1.  通过[american fuzzy lop](http://lcamtuf.coredump.cx/afl/)发现并修复了一些隐晦的崩溃错误。Ticket [a59ae93ee990a55](https://www.sqlite.org/src/info/a59ae93ee990a55)。

1.  解决了 GCC 优化器在编译时（gcc 4.2.1 on MacOS 10.7）导致 R-Tree 扩展在使用-O3 时计算错误结果的问题。

    **其他更改：**

1.  禁用 strchrnul() C 库例程的使用，除非使用了特定的编译时选项-DHAVE_STRCHRNULL。

1.  改进了 likelihood()、likely()和 unlikely() SQL 提示函数的效果和准确性。

1.  SQLITE_SOURCE_ID: "2015-01-16 12:08:06 7d68a42face3ab14ed88407d4331872f5b243fdf"

1.  sqlite3.c 的 SHA1：91aea4cc722371d58aae3d22e94d2a4165276905

### 2014-12-09 (3.8.7.4)

1.  Bug 修复：添加了在前一个发布版本中遗漏的互斥体。

1.  SQLITE_SOURCE_ID: "2014-12-09 01:34:36 f66f7a17b78ba617acde90fc810107f34f1a1f2e"

1.  sqlite3.c 的 SHA1：0a56693a3c24aa3217098afab1b6fecccdedfd23

### 2014-12-05 (3.8.7.3)

1.  Bug 修复：确保缓存的 KeyInfo 对象（应用程序不可见的内部抽象）在共享缓存模式下操作时，频繁关闭并重新打开某些数据库连接，同时保持其他数据库连接在同一共享缓存中持续打开时不会过期。票证 [e4a18565a36884b00edf](https://www.sqlite.org/src/info/e4a18565a36884b00edf)。

1.  Bug 修复：意识到在 LEFT JOIN 的右侧表中，即使列有 NOT NULL 约束，任何列都可以为 NULL。不要应用假设该列永不为 NULL 的优化。票证 [6f2222d550f5b0ee7ed](https://www.sqlite.org/src/info/6f2222d550f5b0ee7ed)。

1.  SQLITE_SOURCE_ID: "2014-12-05 22:29:24 647e77e853e81a5effeb4c33477910400a67ba86"

1.  sqlite3.c 的 SHA1：3ad2f5ba3a4a3e3e51a1dac9fda9224b359f0261

### 2014-11-18 (3.8.7.2)

1.  加强 ROLLBACK 命令，以便在模式未更改的情况下允许挂起的查询继续执行。以前，ROLLBACK 会导致所有挂起的查询失败，并显示 SQLITE_ABORT 或 SQLITE_ABORT_ROLLBACK 错误。如果 ROLLBACK 修改了模式，仍会返回该错误。

1.  Bug 修复: 确保 OP_Column 返回的 NULL 结果完全为 NULL，不要设置 MEM_Ephem 位。Ticket [094d39a4c95ee4](https://www.sqlite.org/src/info/094d39a4c95ee4)。

1.  Bug 修复: sqlite3_mprintf() 中的 %c 格式能够处理大于 70 的精度。

1.  Bug 修复: 不要自动从一个 SELECT 中删除形成 IN 运算符右侧的 DISTINCT 关键字，因为如果 SELECT 同时包含 LIMIT，则这是必需的。Ticket [db87229497](https://www.sqlite.org/src/info/db87229497)。

1.  SQLITE_SOURCE_ID: "2014-11-18 20:57:56 2ab564bf9655b7c7b97ab85cafc8a48329b27f93"

1.  sqlite3.c 的 SHA1 值: b2a68d5783f48dba6a8cb50d8bf69b238c5ec53a

### 2014-10-29 (3.8.7.1)

1.  在 PRAGMA journal_mode=TRUNCATE 模式下，在截断日志文件后立即调用 fsync()，以确保事务在断电时是持久的。

1.  修复了在使用 ALTER TABLE ADD COLUMN 添加到表末尾的字段的 NULL 值时可能发生的断言错误。

1.  除非设置了编译时选项 HAVE_STRCHRNULL，否则不要尝试使用标准 C 库中的 strchrnul() 函数。

1.  修复了在带有 VIEW 的 rowid 在 WHERE 子句中执行 UPDATE 或 DELETE 时遇到的一些问题。

1.  SQLITE_SOURCE_ID: "2014-10-29 13:59:56 3b7b72c4685aa5cf5e675c2c47ebec10d9704221"

1.  sqlite3.c 的 SHA1 值: 2d25bd1a73dc40f538f3a81c28e6efa5999bdf0c

### 2014-10-17 (3.8.7)

**性能增强:**

1.  许多微优化导致相同 CPU 周期的工作量增加了 20.3%。自 版本 3.8.0 以来的累积性能提升为 61%。（在 Ubuntu 13.10 x64 上使用 gcc 4.8.1 和 -Os，在 [speedtest1.c](https://www.sqlite.org/src/artifact/83f6b3318f7ee) 的工作负载上使用 [cachegrind](http://valgrind.org/docs/manual/cg-manual.html) 进行测量。您的性能可能会有所不同。）

1.  排序器可以使用辅助帮助线程来增加实时响应。此功能默认关闭，可以通过 PRAGMA threads 命令或 SQLITE_DEFAULT_WORKER_THREADS 编译时选项启用。

1.  加强了跳过扫描优化，使其能够跳过索引中间出现的索引条目，而不仅仅是索引的左侧。

1.  改进了 CAST 运算符的优化。

1.  在查询规划器如何使用 sqlite_stat4 信息估算计划成本方面进行了各种改进。

    **新功能:**

1.  添加了具有 64 位长度参数的新接口：sqlite3_malloc64()，sqlite3_realloc64()，sqlite3_bind_blob64()，sqlite3_result_blob64()，sqlite3_bind_text64()和 sqlite3_result_text64()。

1.  添加了新接口 sqlite3_msize()，用于返回从 sqlite3_malloc64()及其变体获取的内存分配的大小。

1.  在 sqlite3_limit()和 PRAGMA threads 命令中增加了 SQLITE_LIMIT_WORKER_THREADS 选项，用于配置可用工作线程的数量。

1.  spellfix1 扩展允许应用程序可选地为每个 INSERT 指定 rowid。

1.  添加了[用户认证](https://www.sqlite.org/src/doc/trunk/ext/userauth/user-auth.txt)扩展。

    **错误修复:**

1.  修复了部分索引实现中的一个 bug，如果部分索引用于子查询或视图，可能会导致答案不正确。票证[98d973b8f5](https://www.sqlite.org/src/info/98d973b8f5)。

1.  修复了查询规划器的一个 bug，可能导致使用 DESC 索引实现 ORDER BY 子句时，表被错误地扫描（从而颠倒输出顺序）。Ticket [ba7cbfaedc7e6](https://www.sqlite.org/src/info/ba7cbfaedc7e6)。

1.  修复了 sqlite3_trace() 中的一个 bug，有时导致无法打印 SQL 语句，如果该语句需要重新准备。Ticket [11d5aa455e0d98f3c1e6a08](https://www.sqlite.org/src/info/11d5aa455e0d98f3c1e6a08)

1.  修复了一个错误的 assert() 语句。 Ticket [369d57fb8e5ccdff06f1](https://www.sqlite.org/src/info/369d57fb8e5ccdff06f1)

    **测试、调试和分析更改：**

1.  在 命令行 shell 编译时，使用 ".selecttrace" 和 ".wheretrace" 命令显示 ASCII 艺术风格的抽象语法树图，同时需要编译开启 SQLITE_DEBUG、SQLITE_ENABLE_SELECTTRACE 和 SQLITE_ENABLE_WHERETRACE。同时提供 sqlite3TreeViewExpr() 和 sqlite3TreeViewSelect() 入口点，可以在调试器中断点时调用以显示解析树。

1.  放弃了对 SQLITE_ENABLE_TREE_EXPLAIN 的支持。SELECTTRACE 机制提供了更有用的诊断信息。

1.  为配置辅助内存使用增加了新选项到 命令行 shell: --pagecache, --lookaside 和 --scratch。

1.  SQLITE_SOURCE_ID: "2014-10-17 11:24:17 e4ab094f8afce0817f4074e823fabe59fc29ebb4"

1.  sqlite3.c 的 SHA1 校验值: 56dcf5e931a9e1fa12fc2d600cd91d3bf9b639cd

### 2014-08-15 (3.8.6)

1.  在 SQL 解析器中添加了对 十六进制整数字面量 的支持。 (例如: 0x123abc)

1.  增强了 PRAGMA integrity_check 命令，以检测 UNIQUE 和 NOT NULL 约束的违规情况。

1.  将 SQLITE_MAX_ATTACHED 的最大值从 62 增加到 125。

1.  在在 WAL 模式下超时之前发出 SQLITE_PROTOCOL 错误的时间从 1 秒增加到 10 秒。

1.  添加了 likely(X) SQL 函数。

1.  FTS4 现在默认包含 unicode61 分词器。

1.  当运行 ANALYZE 时，确保所有预编译语句都会触发自动重新准备。

1.  在源代码树中添加了一个新的可加载扩展源代码文件：[fileio.c](https://www.sqlite.org/src/finfo?name=ext/misc/fileio.c)

1.  在命令行 shell 中添加了扩展函数 readfile(X) 和 writefile(X,Y)（使用从前述条目中的 fileio.c 复制/粘贴的代码）。

1.  在命令行 shell 中添加了.fullschema 点命令。

    **性能增强：**

1.  在 IN 操作符右侧的子查询上停用 DISTINCT 关键字。

1.  添加了作为一系列比较来评估 IN 操作符的能力，作为使用表查找的替代方案。在右侧的 IN 操作符较小且/或频繁更改时，使用比较序列实现在可能更快的情况下。

1.  查询规划器现在使用 sqlite_stat4 信息（由 ANALYZE 创建）来帮助确定是否适合使用跳过扫描优化。

1.  确保查询规划器在尝试使用自制的临时索引替代模式定义的索引时不会出现。

1.  对 VDBE 代码进行了其他微小的调整，以提高质量。

    **错误修复：**

1.  修复了在版本 3.8.2 中添加 WITHOUT ROWID 支持时引入的错误，允许给非唯一 NOT NULL 列添加唯一索引。Ticket [9a6daf340df99ba93c](https://www.sqlite.org/src/info/9a6daf340df99ba93c)

1.  修复了在上一个版本中引入的 R-Tree 扩展 中的错误，该错误可能导致在 IN 操作符 的左侧使用 R-Tree 的 rowid 时出现不正确的结果。Ticket [d2889096e7bdeac6](https://www.sqlite.org/src/info/d2889096e7bdeac6)。

1.  修复了 sqlite3_stmt_busy() 接口，以便对已经步进但从未重置的 ROLLBACK 语句给出正确的答案。

1.  修复了一个错误，如果具有聚合函数的 DEFAULT 列尝试使用其 DEFAULT，将导致空指针被解引用。Ticket [3a88d85f36704eebe1](https://www.sqlite.org/src/info/3a88d85f36704eebe1)

1.  来自 命令行 shell 的 CSV 输出现在始终使用 CRNL 作为行分隔符，并避免在数据中插入 CR 前缀的 NL。

1.  修复了 IN 操作符 中与 列亲和性 有关的问题。Ticket [9a8b09f8e6](https://www.sqlite.org/src/info/9a8b09f8e6)。

1.  修复了 ANALYZE 命令，以便在 sqlite_stat4 表中的 WITHOUT ROWID 表中添加正确的样本。Ticket [b2fa5424e6fcb15](https://www.sqlite.org/src/info/b2fa5424e6fcb15)。

1.  SQLITE_SOURCE_ID: "2014-08-15 11:46:33 9491ba7d738528f168657adb43a198238abde19e"

1.  sqlite3.c 的 SHA1 值：72c64f05cd9babb9c0f9b3c82536d83be7804b1c

### 2014-06-04 (3.8.5)

1.  增加了对索引的 部分排序 的支持。

1.  增强了查询规划器，以便始终优先选择使用 WHERE 子句术语的超集的索引而不是其他索引。

1.  对 FTS4 的 automerge 命令 进行改进，以更好地控制完全文本索引的索引大小，特别是针对大量更新的情况。

1.  添加了 sqlite3_rtree_query_callback() 接口到 R-Tree 扩展。

1.  添加了新的 URI 查询参数 "nolock" 和 "immutable"。

1.  在只读数据库连接中不再记住 CHECK 约束，从而减少内存使用。

1.  为 WITHOUT ROWID 表启用 OR 优化。

1.  将形如 "x IN (?)"（右侧列表中只有一个值的情况）的表达式优化为 "x==?"，类似地优化 "x NOT IN (?)"。

1.  向 命令行 shell 添加了 ".system" 和 ".once" 命令。

1.  添加了 SQLITE_IOCAP_IMMUTABLE 位到 VFS 的 xDeviceCharacteristics 方法返回的位集合中。

1.  添加了 SQLITE_TESTCTRL_BYTEORDER 测试控制功能。

    **错误修复：**

1.  在没有 FROM 子句的查询中忽略了 OFFSET 子句。Ticket [07d6a0453d](https://www.sqlite.org/src/info/07d6a0453d)。

1.  在涉及形如 "x IN (?)" 的表达式的查询中出现断言错误。Ticket [e39d032577](https://www.sqlite.org/src/info/e39d032577)。

1.  报告了不正确的列数据类型。Ticket [a8a0d2996a](https://www.sqlite.org/src/info/a8a0d2996a)。

1.  在针对具有超过 16 个索引（每个索引对应不同列）并且通过 OR 连接的约束使用所有索引时，查询返回了重复的行。Ticket [10fb063b11](https://www.sqlite.org/src/info/10fb063b11)。

1.  在 UPDATE OR REPLACE 操作中，部分索引导致断言错误。Ticket [2ea3e9fe63](https://www.sqlite.org/src/info/2ea3e9fe63)。

1.  当调用未记录的 SQL 函数 sqlite_rename_parent() 时，参数为空时发生崩溃。Ticket [264b970c43](https://www.sqlite.org/src/info/264b970c4379fd)。

1.  如果查询具有相同的 GROUP BY，那么会忽略 ORDER BY。Ticket [b75a9ca6b0](https://www.sqlite.org/src/info/b75a9ca6b0499)

1.  group_concat(x,'') SQL 函数在所有输入都是空字符串时返回 NULL，而不是空字符串。Ticket [55746f9e65](https://www.sqlite.org/src/info/55746f9e65f85)

1.  修复了在 VDBE 代码生成器中的 bug，在执行 INSERT INTO ... SELECT 语句时，被插入的列数大于目标表中的列数时会导致崩溃。Ticket [e9654505cfd](https://www.sqlite.org/src/info/e9654505cfda9)

1.  修复了命令行 shell 中 CSV 导入的问题，当 CSV 文件第一行最左侧字段大小为零字节且未引用时，不会导入任何数据。

1.  修复了 FTS4 中的问题，左起第一列包含 notindexed 列 名称作为前缀时未被索引而不是其名称完全匹配的列。

1.  修复了 sqlite3_db_readonly() 接口，以便在数据库由于文件格式写入版本号过大而只读时返回 true。

1.  SQLITE_SOURCE_ID: "2014-06-04 14:06:34 b1ed4f2a34ba66c29b130f8d13e9092758019212"

1.  sqlite3.c 的 SHA1：7bc194957238c61b1a47f301270286be5bc5208c

### 2014-04-03 (3.8.4.3)

1.  为可能导致在混合 DISTINCT、子查询中的 GROUP BY 和 ORDER BY 的查询上产生不正确结果的问题添加了一个 [一字符修复](https://www.sqlite.org/src/fdiff?sbs=1&v1=7d539cedb1c&v2=ebad891b7494d&smhdr)。Ticket 98825a79ce14](https://www.sqlite.org/src/info/98825a79ce1456863)。

1.  SQLITE_SOURCE_ID: "2014-04-03 16:53:12 a611fa96c4a848614efe899130359c9f6fb889c3"

1.  sqlite3.c 的 SHA1：310a1faeb9332a3cd8d1f53b4a2e055abf537bdc

### 2014-03-26 (3.8.4.2)

1.  修复搜索损坏的数据库文件时可能导致的潜在缓冲区溢出问题。

1.  SQLITE_SOURCE_ID: "2014-03-26 18:51:19 02ea166372bdb2ef9d8dfbb05e78a97609673a8e"

1.  SHA1 for sqlite3.c: 4685ca86c2ea0649ed9f59a500013e90b3fe6d03

### 2014-03-11 (3.8.4.1)

1.  解决了破坏一些配置下使用 Microsoft Visual Studio 构建的 C 预处理宏冲突问题。

1.  在计算 skip-scan 优化 的成本时，考虑到需要多次查找的事实。

1.  SQLITE_SOURCE_ID: "2014-03-11 15:27:36 018d317b1257ce68a92908b05c9c7cf1494050d0"

1.  SHA1 for sqlite3.c: d5cd1535053a50aa8633725e3595740b33709ac5

### 2014-03-10 (3.8.4)

1.  为了提高性能进行代码优化和重构。

1.  在命令行 shell 中增加了 ".clone" 和 ".save" 命令。

1.  更新命令行 shell 上的横幅，以便在用户使用临时内存数据库时向新手用户发出警告。

1.  修复命令行 shell 中对 editline 的支持。

1.  增加对使用 sqlite3_test_control() 的 SQLITE_TESTCTRL_VDBE_COVERAGE 动词进行 VDBE 程序覆盖测试支持的功能。

1.  更新 _FILE_OFFSET_BITS 宏，以便在 QNX 上再次正常构建。

1.  将 SrcList.nSrc 的数据类型从 u8 更改为 int，以解决 AIX 上 C 编译器的问题。

1.  使得扩展在 Cygwin 上加载正常工作。

1.  Bug fix: 修复 char() SQL 函数在没有参数调用时返回空字符串而不是"内存不足"错误的问题。

1.  Bug fix: DISTINCT 现在能够识别 zeroblob 和全 0x00 字节的 blob 是相同的东西。[票号 [fccbde530a]](https://www.sqlite.org/src/info/fccbde530a)

1.  修复错误：修正在 WHERE 子句中包含 IS NOT NULL 条件和同时包含 OR 条件，以及使用 SQLITE_ENABLE_STAT4 编译的查询时计算正确答案的问题。[票号 [4c86b126f2]](https://www.sqlite.org/src/info/4c86b126f2)

1.  Bug 修复：确保在普通表与 WITHOUT ROWID 表之间的连接中正确解析"rowid"列。[票据 [c34d0557f7]](https://www.sqlite.org/src/info/c34d0557f7)

1.  Bug 修复：确保在并发协程中不会使用相同的临时寄存器来实现包含 ORDER BY 子句的复合 SELECT 语句，因为这样的使用可能导致错误的答案。[票据 [8c63ff0eca]](https://www.sqlite.org/src/info/8c63ff0eca)

1.  Bug 修复：确保"ORDER BY random()"子句不会被优化掉。[票据 [65bdeb9739]](https://www.sqlite.org/src/info/65bdeb9739)

1.  Bug 修复：修复了在 TRIGGER 内包含的子查询语句中可能发生的名称解析错误。[票据 [4ef7e3cfca]](https://www.sqlite.org/src/info/4ef7e3cfca)

1.  Bug 修复：修复形如"DEFAULT(-(-9223372036854775808))"的列默认值表达式，使其能够正确工作，将列初始化为大约等于+9223372036854775808.0 的浮点值。

1.  SQLITE_SOURCE_ID："2014-03-10 12:20:37 530a1ee7dc2435f80960ce4710a3c2d2bfaaccc5"

1.  sqlite3.c 的 SHA1 值：b0c22e5f15f5ba2afd017ecd990ea507918afe1c

### 2014-02-11 (3.8.3.1)

1.  修复了一个 bug（票据 [4c86b126f2](https://www.sqlite.org/src/info/4c86b126f2)），当使用 SQLITE_ENABLE_STAT3 或 SQLITE_ENABLE_STAT4 编译时选项，在 WHERE 子句中的 OR 子句和 IS NOT NULL 操作符一起使用时，某些查询可能会导致行丢失。

1.  修复了一个对 VS2013 造成问题的无害编译器警告。

1.  SQLITE_SOURCE_ID："2014-02-11 14:52:19 ea3317a4803d71d88183b29f1d3086f46d68a00e"

1.  sqlite3.c 的 SHA1 值：990004ef2d0eec6a339e4caa562423897fe02bf0

### 2014-02-03 (3.8.3)

1.  增加了对通用表达式和 WITH 子句的支持。

1.  添加了 printf() SQL 函数。

1.  在 sqlite3_create_function()及其相关接口的第四个参数中添加了 SQLITE_DETERMINISTIC 作为可选位，使应用程序能够创建新函数，这些函数可以在具有常量参数时从内部循环中分离出来。

1.  添加 SQLITE_READONLY_DBMOVED 错误代码，在事务开始时返回，表示基础数据库文件已被重命名或移动。

1.  允许在 ATTACH 的文件名参数中使用任意表达式，包括函数调用和子查询。

1.  允许在任何适用于 SELECT 语句的地方使用 VALUES 子句。

1.  当 N==0 时，重新播种 sqlite3_randomness(N,P)使用的 PRNG。在 Unix 上进行 fork()后自动重新播种。

1.  增强 spellfix1 虚拟表，使其能够通过 rowid 进行高效搜索。

1.  性能增强。

1.  在运行 EXPLAIN 时改进了 VDBE 字节码显示的注释。

1.  添加"%token_class"指令到 Lemon 解析器生成器，并用它来简化语法。

1.  更改 Lemon 源代码，避免调用 OpenBSD 认为危险的 C 库函数（例如：sprintf）。

1.  修复错误：在命令行 shell 的 CSV 导入功能中，当转义的双引号出现在 CRLN 行的末尾时，不要结束字段。

1.  SQLITE_SOURCE_ID："2014-02-03 13:52:03 e816dd924619db5f766de6df74ea2194f3e3b538"

1.  sqlite3.c 的 SHA1 值：98a07da78f71b0275e8d9c510486877adc31dbee

### 2013-12-06 (3.8.2)

1.  更改了 CAST 表达式的定义行为，当将浮点值大于+9223372036854775807 转换为整数时，结果是最大可能的整数+9223372036854775807，而不是最小可能的整数-9223372036854775808。在此更改后，CAST(9223372036854775809.0 as INT)的结果为+9223372036854775807，而不是-9223372036854775808。 **← 可能存在不兼容的更改！**

1.  增加了对 WITHOUT ROWID 表的支持。

1.  将跳跃扫描优化器（skip-scan optimization）添加到查询规划器中。

1.  扩展了虚拟表接口，特别是 sqlite3_index_info 对象，允许虚拟表报告其查询将返回的行数估计。

1.  更新了 R-Tree 扩展，以利用增强的虚拟表接口。

1.  添加了 SQLITE_ENABLE_EXPLAIN_COMMENTS 编译时选项。

1.  改进了插入到 EXPLAIN 输出中的注释，当启用 SQLITE_ENABLE_EXPLAIN_COMMENTS 编译时选项时。

1.  VDBE 性能增强，尤其是 OP_Column 操作码。

1.  在预处理语句中，将内部循环中的常量子表达式因子提取到初始化代码中。

1.  改进了命令行 shell 的".explain"输出格式，使循环缩进，更好地显示程序的结构。

1.  加强了命令行 shell 的".timer"功能，除了系统和用户时间外，还显示了墙钟时间。

1.  SQLITE_SOURCE_ID："2013-12-06 14:53:30 27392118af4c38c5203a04b8013e1afdb1cebd0d"

1.  sqlite3.c 的 SHA1 哈希值：6422c7d69866f5ea3db0968f67ee596e7114544e

### 2013-10-17 (3.8.1)

1.  增加了用作查询规划器提示的 unlikely() 和 likelihood() SQL 函数。

1.  查询规划器的增强：

    1.  考虑 WHERE 子句中不能与索引一起使用的条件仍可能减少输出行数的事实。

    1.  估算表和索引行的大小，并在全表扫描和 "count(*)" 操作中使用最小适用的 B-Tree。

1.  添加了 soft_heap_limit pragma。

1.  增加对 SQLITE_ENABLE_STAT4 的支持。

1.  在 sqlite_stat1.stat 字段末尾增加了对 "sz=NNN" 参数的支持，用于指定表和索引行的平均字节长度。

1.  如果更新操作未涉及到任何外键相关的列，则避免在更新操作中运行外键约束检查。

1.  增加了 SQLITE_MINIMUM_FILE_DESCRIPTOR 编译时选项。

1.  在 Windows 上添加了 win32-longpath VFS，允许文件名长度达到 32K 字符。

1.  对 Date And Time Functions 进行增强，确保同一 sqlite3_step() 调用中多次函数调用返回的当前时间（例如 julianday('now')）始终相同。

1.  添加了 "totype.c" 扩展，实现了 tointeger() 和 toreal() SQL 函数。

1.  FTS4 查询更能够利用 docid<$limit 约束来限制所需的 I/O 数量。

1.  在 fts4aux 虚拟表中添加了隐藏的 fts4aux languageid column。

1.  VACUUM 命令使数据库压缩约 1%。

1.  更新了 sqlite3_analyzer 实用程序程序，以提供更好的描述并在 "Non-sequential pages" 计算更精确的估计。

1.  重构 PRAGMA 语句的实现，以提升解析性能。

1.  Unix 系统上用于保存临时文件的目录现在可以通过 SQLITE_TMPDIR 环境变量进行设置，该变量优先于 TMPDIR 环境变量。然而，全局变量 sqlite3_temp_directory 的优先级仍高于这两个环境变量。

1.  添加了 PRAGMA stats 语句。

1.  **Bug fix:** 即使表上有 部分索引，也返回 "SELECT count(*) FROM table" 的正确答案。Ticket [a5c8ed66ca](https://www.sqlite.org/src/info/a5c8ed66ca)。

1.  SQLITE_SOURCE_ID: "2013-10-17 12:57:35 c78be6d786c19073b3a6730dfe3fb1be54f5657a"

1.  sqlite3.c 的 SHA1 值：0a54d76566728c2ba96292a49b138e4f69a7c391

### 2013-09-03 (3.8.0.2)

1.  修复了优化中的一个 bug，该优化尝试省略未使用的 LEFT JOIN。

1.  SQLITE_SOURCE_ID: "2013-09-03 17:11:13 7dd4968f235d6e1ca9547cda9cf3bd570e1609ef"

1.  sqlite3.c 的 SHA1 值：6cf0c7b46975a87a0dc3fba69c229a7de61b0c21

### 2013-08-29 (3.8.0.1)

1.  修复一个错误，该错误导致 CSV 输入行以 CRNL 结尾时的带引号空字符串被命令行 shell 误读。

1.  修复了查询计划器中的一个 bug，涉及带有 BETWEEN 或 LIKE/GLOB 约束的 LEFT JOIN，然后再内连接到右侧，该 bug 包含 OR 约束。

1.  修复查询计划器的一个 bug，该 bug 在查询带有多于四列的 UNIQUE 或 PRIMARY KEY 约束的表时可能导致段错误。

1.  SQLITE_SOURCE_ID: "2013-08-29 17:35:01 352362bc01660edfbda08179d60f09e2038a2f49"

1.  sqlite3.c 的 SHA1 值：99906bf63e6cef63d6f3d7f8526ac4a70e76559e

### 2013-08-26 (3.8.0)

1.  添加对部分索引的支持。

1.  切换至用于更快速和更好的查询计划的 下一代查询计划器。

1.  EXPLAIN QUERY PLAN 输出不再显示每个连接中生成的行数的估算值。

1.  添加了 FTS4 notindexed option，允许在 FTS4 表中使用非索引列。

1.  添加了 sqlite3_stmt_status()的 SQLITE_STMTSTATUS_VM_STEP 选项。

1.  添加了 cache_spill pragma。

1.  添加了 query_only pragma。

1.  添加了 defer_foreign_keys pragma 和 sqlite3_db_status(db, SQLITE_DBSTATUS_DEFERRED_FKS,...) C 语言接口。

1.  在源代码树的 ext/misc 子目录中作为可加载扩展添加了"percentile()"函数。

1.  添加了 SQLITE_ALLOW_URI_AUTHORITY 编译时选项。

1.  添加了 sqlite3_cancel_auto_extension(X)接口。

1.  缺少 FROM 子句的运行中 SELECT 语句（或任何从未读取或写入任何数据库文件的其他语句）不会阻止读取事务关闭。

1.  添加了 SQLITE_DEFAULT_AUTOMATIC_INDEX 编译时选项。将此选项设置为 0 可默认禁用自动索引。

1.  在 SQLITE_CONFIG_LOG 使用自动索引时，对 SQLITE_WARNING_AUTOINDEX 发出警告的查询计划器。

1.  添加了 SQLITE_FTS3_MAX_EXPR_DEPTH 编译时选项。

1.  向 next_char()扩展 SQL 函数添加了可选的第五个参数，定义排序序列。

1.  当读取事务在较旧的快照上时，以 WAL 模式返回 SQLITE_BUSY_SNAPSHOT 扩展错误代码，因为读取事务无法升级为写入事务。

1.  对 sqlite3_analyzer 实用程序进行了增强，以提供表的每个单独索引的大小信息，除了聚合大小之外。

1.  允许通过 SQL 语句从应用程序定义的 SQL 函数的实现中自由地打开和关闭读取事务，如果该函数由不访问任何数据库表的 SELECT 语句调用。

1.  在所有（unix）系统上禁用 posix_fallocate()，除非使用了 HAVE_POSIX_FALLOCATE 编译时选项。

1.  更新命令行 shell 中的".import"命令，以支持多行字段和正确的 RFC-4180 引用，并在输入文本不严格符合 RFC-4180 时发出警告和/或错误消息。

1.  Bug 修复：在 FTS4 的 unicode61 分词器中，将所有私有代码点视为标识符符号。

1.  Bug 修复：在 ORDER BY 子句中的裸标识符紧密绑定到输出列名，但在表达式中的标识符紧密绑定到输入列名。在 GROUP BY 子句中的标识符始终优先选择输出列名。

1.  Bug 修复：通过迁移到 NGQP 解决了传统查询优化器中的多个问题。

1.  SQLITE_SOURCE_ID："2013-08-26 04:50:08 f64cd21e2e23ed7cff48f7dafa5e76adde9321c2"

1.  sqlite3.c 的 SHA1：b7347f4b4c2a840e6ba12040093d606bd16ea21e

### 2013-05-20（3.7.17）

1.  为 memory-mapped I/O 添加支持。

1.  添加了 sqlite3_strglob()便捷接口。

1.  将整数分配给数据库头中偏移量为 68 的位置，作为 SQLite 用作应用文件格式时的应用程序 ID。添加了 PRAGMA application_id 命令以查询和设置应用程序 ID。

1.  在错误日志中报告回滚恢复为 SQLITE_NOTICE_RECOVER_ROLLBACK。将 WAL 恢复的错误日志代码从 SQLITE_OK 更改为 SQLITE_NOTICE_RECOVER_WAL。

1.  在错误日志中将未链接的数据库文件和数据库文件名别名的风险使用报告为 SQLITE_WARNING 消息。

1.  增加 SQLITE_TRACE_SIZE_LIMIT 编译时选项。

1.  将 SQLITE_MAX_SCHEMA_RETRY 的默认值增加到 50，并确保在可能强制语句重试的每个地方都遵守。

1.  添加一个名为"mptester"的新测试工具，用于验证当多个进程同时使用相同的数据库文件时的正确操作。

1.  增强扩展加载机制，以使其更加灵活（同时仍然保持向后兼容性）：

    1.  如果默认入口点"sqlite3_extension_init"在可加载扩展中不存在，还可以尝试一个入口点"sqlite3_X_init"，其中"X"基于共享库文件名。这允许每个扩展有一个不同的入口点，从而使它们可以在没有代码更改的情况下进行静态链接。

    1.  传递给 sqlite3_load_extension()的共享库文件名可以省略文件名后缀，将自动添加适当的体系结构相关后缀（".so"，".dylib"或".dll"）。

1.  将许多新的可加载扩展添加到源代码树中，包括 amatch，closure，fuzzer，ieee754，nextchar，regexp，spellfix 和 wholenumber。有关每个扩展执行的详细信息，请参阅每个扩展源文件的标头注释。

1.  增强 FTS3 以避免在匹配运算符的右侧有大量术语时使用过多的堆栈空间。这种变化的副作用是 MATCH 运算符一次只能容纳 12 个 NEAR 运算符。

1.  加强 fts4aux 虚拟表，使其可以成为一个临时表。

1.  将 fts3tokenize 虚拟表添加到全文搜索逻辑中。

1.  查询规划增强：尽可能使用约束的传递属性将约束移到联接的外部循环中，从而减少内部循环中需要执行的工作量。

1.  在 Unix 上停止使用 posix_fallocate()，因为它不适用于所有文件系统。

1.  在 Windows VFS 中改进了跟踪和调试功能。

1.  Bug 修复：在一个数据库连接关闭时，另一个连接处于写事务中时，在共享缓存模式中修复潜在的**数据库损坏 bug**。票号[e636a050b7](https://www.sqlite.org/src/info/e636a050b7)。

1.  Bug 修复：只有在没有其他匹配项时，才考虑结果集中的 AS 名称来解析 WHERE 子句中的标识符。在 ORDER BY 子句中，AS 名称优先于任何列名。票号[2500cdb9be05](https://www.sqlite.org/src/info/2500cdb9be05)

1.  Bug 修复：除非保证所有外部循环最多只返回一行结果，否则不允许虚拟表取消 ORDER BY 子句。票号[ba82a4a41eac1](https://www.sqlite.org/src/info/ba82a4a41eac1)。

1.  Bug 修复：如果使用 IN 约束条件，则不要抑制虚拟表查询的 ORDER BY 子句。票号[f69b96e3076e](https://www.sqlite.org/src/info/f69b96e3076e)。

1.  Bug 修复：当使用“.quit”命令终止时，命令行 shell 会返回退出码 0。

1.  Bug 修复：确保 PRAGMA 语句出现在 sqlite3_trace()输出中。

1.  Bug 修复：当使用带有 COLLATE 操作符的 ORDER BY 子句的复合查询时，确保排序按指定的排序规则进行，并且与复合查询相关的比较使用本地排序。票号[6709574d2a8d8](https://www.sqlite.org/src/info/6709574d2a8d8)。

1.  Bug 修复：确保在进行 UPDATE 更改行 ID 时，authorizer 回调得到一个有效指向字符串"ROWID"的指针。票号[0eb70d77cb05bb2272](https://www.sqlite.org/src/info/0eb70d77cb05bb2272)。

1.  Bug 修复：不要将 WHERE 子句项移动到 LEFT JOIN 的 ON 子句中包含的 OR 表达式内部。票号[f2369304e4](https://www.sqlite.org/src/info/f2369304e4)。

1.  Bug 修复：确保在尝试执行需要缺失的排序序列的操作时始终报告错误。票号[0fc59f908b](https://www.sqlite.org/src/info/0fc59f908b)。

1.  SQLITE_SOURCE_ID: "2013-05-20 00:56:22 118a3b35693b134d56ebd780123b7fd6f1497668"

1.  SHA1 for sqlite3.c: 246987605d0503c700a08b9ee99a6b5d67454aab

### 2013-04-12 (3.7.16.2)

1.  修复自版本 3.7.13 以来存在的一个 bug，当两个或更多进程尝试同时访问同一数据库文件，并且第三个进程在向同一文件提交时崩溃时，可能导致数据库损坏。详见票号[7ff3120e4f](https://www.sqlite.org/src/info/7ff3120e4f)。

1.  SQLITE_SOURCE_ID: "2013-04-12 11:52:43 cbea02d93865ce0e06789db95fd9168ebac970c7"

1.  SHA1 for sqlite3.c: d466b54789dff4fb0238b9232e74896deaefab94

### 2013-03-29 (3.7.16.1)

1.  修复在版本 3.7.15 中引入的 ORDER BY 优化器中的一个 bug，有时会优化掉实际上需要排序步骤的情况。票号[a179fe7465](https://www.sqlite.org/src/info/a179fe7465)。

1.  修复在 CAST 表达式中长期存在的一个 bug，即使它们的最高有效字节不为零，也会将 UTF16 字符识别为数字。票号[689137afb6da41](https://www.sqlite.org/src/info/689137afb6da41)。

1.  修复了应用于子字段时 FTS3 的 NEAR 运算符中的一个 bug。Ticket [38b1ae018f](https://www.sqlite.org/src/info/38b1ae018f)。

1.  修复了存储引擎中一个长期存在的 bug，该 bug（非常罕见地）会导致虚报 SQLITE_CORRUPT 错误，但在其他情况下是无害的。Ticket [6bfb98dfc0c](https://www.sqlite.org/src/info/6bfb98dfc0c)。

1.  SQLITE_OMIT_MERGE_SORT 选项已被移除。合并排序器现在是 SQLite 的必需组件。

1.  修复源代码注释中的大量拼写错误。

1.  SQLITE_SOURCE_ID: "2013-03-29 13:44:34 527231bc67285f01fb18d4451b28f61da3c4e39d"

1.  sqlite3.c 的 SHA1 值为：7a91ceceac9bcf47ceb8219126276e5518f7ff5a

### 2013-03-18 (3.7.16)

1.  添加了 PRAGMA foreign_key_check 命令。

1.  为所有 SQLITE_CONSTRAINT 错误添加了新的扩展错误码。

1.  为当数据库由于需要回滚恢复但是只读时无法打开时，添加了 SQLITE_READONLY_ROLLBACK 扩展错误码。

1.  添加了 SQL 函数 unicode(A) 和 char(X1,...,XN)。

1.  改进了 PRAGMA incremental_vacuum 的性能，特别是在空闲页数量大于可适应于自由列表的单个主干页的情况下。

1.  改进了包含聚合函数 min() 或 max() 的查询的优化。

1.  改进虚拟表，使其在 WHERE 子句包含 IN 操作符时可以潜在地使用索引。

1.  允许即使在 WHERE 子句中的前置索引受到 IN 操作符的约束，也可以用于排序的索引。

1.  改进了 PRAGMA table_info 命令，使 "pk" 列成为递增整数，以显示主键中列的顺序。

1.  改进查询优化器，以利用传递连接约束。

1.  改进了查询优化器的性能。

1.  允许 PRAGMA integrity_check 的错误消息长度超过 20000 字节。

1.  改进了深度嵌套查询的名称解析。

1.  添加了 test_regexp.c 模块，用于演示如何实现 REGEXP 操作符。

1.  RTREE 扩展中改进了错误消息。

1.  增强命令行 shell，使得对 ".exit" 命令的非零参数立即导致 shell 关闭数据库连接而不是正常关闭。

1.  改进了命令行 shell 中对无效布尔参数 dot-commands 的错误消息。

1.  改进了“外键不匹配”的错误消息，显示涉及的两个表的名称。

1.  移除 unix VFS 中所有对 umask() 的使用。

1.  添加了 PRAGMA vdbe_addoptrace 和 PRAGMA vdbe_debug 命令。

1.  改用 strncmp() 或等效函数替代 memcmp() 比较非零结尾的字符串。

1.  更新 cygwin 接口，省略已弃用的 API 调用。

1.  增强 spellfix1 扩展，使得可以通过将类似 'edit_cost_table=TABLE' 的字符串插入到 "command" 字段中在运行时更改编辑距离成本表。

1.  Bug 修复：修复长期存在的问题，该问题可能导致在将整数字段与两个或多个文本字段进行比较的三路或更大连接中出现错误的查询结果。Ticket [fc7bd6358f](https://www.sqlite.org/src/info/fc7bd6358f)

1.  Bug 修复：如果视图上的 16 位引用计数器因查询过于复杂而溢出，则发出错误消息。

1.  Bug 修复：在深度嵌套的 UNION ALL 查询中，避免在 LIMIT 和 OFFSET 子句中泄漏内存。

1.  Bug 修复：确保在运行 pragmas table_info、index_list、index_info 和 foreign_key_list 之前，模式是最新的。

1.  SQLITE_SOURCE_ID: "2013-03-18 11:39:23 66d5f2b76750f3520eb7a495f6247206758f5b90"

1.  sqlite3.c 的 SHA1 值为：7308ab891ca1b2ebc596025cfe4dc36f1ee89cf6

### 2013-01-09 (3.7.15.2)

1.  修复了一个 bug，在版本 3.7.15 中引入，导致在需要时优化掉三路联接中的 ORDER BY 子句。Ticket [598f5f7596b055](https://www.sqlite.org/src/info/598f5f7596b055)。

1.  SQLITE_SOURCE_ID 为："2013-01-09 11:53:05 c0e09560d26f0a6456be9dd3447f5311eb4f238f"。

1.  sqlite3.c 的 SHA1 值为：5741f47d1bc38aa0a8c38f09e60a5fe0031f272d。

### 2012-12-19 (3.7.15.1)

1.  修复了一个 bug，在版本 3.7.15 中引入，导致在 SELECT 语句的结果列的 AS 名称用作 WHERE 子句中的逻辑项时导致段错误。Ticket [a7b7803e8d1e869](https://www.sqlite.org/src/info/a7b7803e8d1e869)。

1.  SQLITE_SOURCE_ID 为："2012-12-19 20:39:10 6b85b767d0ff7975146156a99ad673f2c1a23318"。

1.  sqlite3.c 的 SHA1 值为：bbbaa68061e925bd4d7d18d7e1270935c5f7e39a。

### 2012-12-12 (3.7.15)

1.  增加了 sqlite3_errstr()接口。

1.  当由于 SQLITE_SCHEMA 错误而自动重新准备语句时，避免多次调用 sqlite3_trace()回调函数。

1.  增加了对 Windows Phone 8 平台的支持。

1.  加强了 IN 操作符的处理，以利用数值亲和性的索引。

1.  在可能的情况下，通过使用覆盖索引进行全表扫描，理论上索引较小，因此可以通过较少的 I/O 进行扫描。

1.  加强了查询优化器，使得 ORDER BY 子句更加积极地进行优化，特别是在联接中，ORDER BY 子句的各项来自不同的表。

1.  增加了作为协程实现 FROM 子句子查询的能力，而不是将子查询显现为临时表。

1.  增强了命令行 shell：

    1.  增加了".print"命令。

    1.  ".width"命令中的负数导致右对齐。

    1.  在使用 SQLITE_DEBUG 编译时添加了".wheretrace"命令。

1.  增加了 busy_timeout pragma。

1.  增加了 instr() SQL 函数。

1.  添加了 SQLITE_FCNTL_BUSYHANDLER 文件控制，用于允许 VFS 实现获取繁忙处理程序回调的访问。

1.  内置 VFSes 中的 xDelete 方法现在在要删除的文件不存在时返回 SQLITE_IOERR_DELETE_NOENT。

1.  增强了对 QNX 的支持。

1.  解决了 MSVC 编译器在 ARM 目标时的优化器错误。

1.  Bug 修复：在共享缓存模式中避免各种并发问题。

1.  Bug 修复：避免在同时使用备份 API，共享缓存和 SQLite 加密扩展时发生死锁或崩溃。

1.  Bug 修复：使用 TCL 接口创建的 SQL 函数遵循“nullvalue”设置。

1.  Bug 修复：修复了在大于 16GB 的数据库上为 CREATE INDEX 时发生的 32 位溢出问题。

1.  Bug 修复：在共享缓存模式中，在 CHECK 约束或视图内使用 COLLATE 操作符时避免段错误。

1.  SQLITE_SOURCE_ID："2012-12-12 13:36:53 cd0b37c52658bfdf992b1e3dc467bae1835a94ae"

1.  sqlite3.c 的 SHA1 值：2b413611f5e3e3b6ef5f618f2a9209cdf25cbcff"

### 2012-10-04 (3.7.14.1)

1.  修复了一个错误（票证[[d02e1406a58ea02d]]](https://www.sqlite.org/src/tktview/d02e1406a58ea02d))，导致在 LEFT JOIN 中包含 ON 子句中的 OR 时发生段错误。

1.  解决 VisualStudio-2012 编译器优化器中的一个错误，导致在 ARM 上编译 SQLite 时生成无效代码。

1.  修复了 TCL 接口，以便 TCL 实现的 SQL 函数尊重“nullvalue”设置。

1.  SQLITE_SOURCE_ID："2012-10-04 19:37:12 091570e46d04e84b67228e0bdbcd6e1fb60c6bdb"

1.  sqlite3.c 的 SHA1 值：62aaecaacab3a4bf4a8fe4aec1cfdc1571fe9a44

### 2012-09-03 (3.7.14)

1.  放弃了对 OS/2 的内置支持。如果需要升级 OS/2 应用程序以使用此版本或更高版本的 SQLite，则可以使用 sqlite3_vfs_register() 接口添加一个应用程序定义的 VFS。本次发布中删除的代码可以作为应用程序定义的 VFS 的基线。

1.  从命令行 shell 的“.dump”命令的输出重建数据库时，确保浮点值的精确保留。

1.  添加了 sqlite3_close_v2() 接口。

1.  更新了命令行 shell，以便可以使用 SQLITE_OMIT_FLOATING_POINT 和 SQLITE_OMIT_AUTOINIT 进行构建。

1.  改进了 Windows 的 makefile 和构建过程。

1.  增强了 PRAGMA integrity_check 和 PRAGMA quick_check，使其可以选择性地仅检查单个附加数据库，而不是所有附加数据库。

1.  改进了 WAL mode 处理，确保始终至少有一个有效的读标记可用，以便只读进程始终可以读取数据库。

1.  在 ORDER BY 和 CREATE INDEX 中使用的排序器中的性能增强。

1.  添加了 SQLITE_DISABLE_FTS4_DEFERRED 编译时选项。

1.  更好地处理包含在子查询中的聚合查询。

1.  增强查询规划器，使其在使用 or optimization 的查询上尝试使用 covering index。

1.  SQLITE_SOURCE_ID："2012-09-03 15:42:36 c0d89d4a9752922f9e367362366efde4f1b06f2a"

1.  sqlite3.c 的 SHA1 值：5fdf596b29bb426001f28b488ff356ae14d5a5a6

### 2012-06-11 (3.7.13)

1.  允许使用 URI 文件名 指定的 内存数据库 使用 共享缓存，因此可以从多个数据库连接访问同一内存数据库。

1.  识别并使用 URI 文件名 中的 mode=memory 查询参数。

1.  在任何一个连接关闭时避免重置 共享缓存 连接的模式。而是等待最后一个连接关闭后再重置模式。

1.  在 RTREE 扩展中，将 64 位浮点数舍入为 32 位以进行存储时，始终选择使边界框变大的舍入方向。

1.  调整 Unix 驱动程序，避免不必要地调用 fchown()。

1.  将接口 sqlite3_quota_ferror() 和 sqlite3_quota_file_available() 添加到 test_quota.c 模块中。

1.  sqlite3_create_module() 和 sqlite3_create_module_v2() 接口在任何尝试重载或替换 虚拟表 模块时返回 SQLITE_MISUSE。在这种情况下始终调用析构函数，符合历史和当前文档的要求。

1.  SQLITE_SOURCE_ID: "2012-06-11 02:05:22 f5b5a13f7394dc143aa136f1d4faba6839eaa6dc"

1.  sqlite3.c 的 SHA1 值为 ff0a771d6252545740ba9685e312b0e3bb6a641b

### 2012-05-22 (3.7.12.1)

1.  修复在 3.7.12 版本中可能导致某些复杂嵌套聚合查询段错误的 bug [(ticket c2ad16f997)](https://www.sqlite.org/src/info/c2ad16f997ee9c)。

1.  修复各种其他次要测试脚本问题。

1.  SQLITE_SOURCE_ID: "2012-05-22 02:45:53 6d326d44fd1d626aae0e8456e5fa2049f1ce0789"

1.  sqlite3.c 的 SHA1 值为 d494e8d81607f0515d4f386156fb0fd86d5ba7df

### 2012-05-14 (3.7.12)

1.  为 sqlite3_db_status() 添加 SQLITE_DBSTATUS_CACHE_WRITE 选项。

1.  优化 typeof() 和 length() SQL 函数，以避免不必要地从磁盘读取数据库内容。

1.  添加 FTS4 "merge" 命令，FTS4 "automerge" 命令 和 FTS4 "integrity-check" 命令。

1.  报告特定 CHECK 约束失败的名称。

1.  在命令行 shell 中，如果 ".output" 命令的参数的第一个字符是 "|"，则使用 popen() 而不是 fopen()。

1.  利用 Windows 的 OVERLAPPED 特性在 VFS 中，避免一些系统调用，从而获得性能改进。

1.  当其中一侧始终为假时，更加积极地优化 AND 运算符。

1.  在 WHERE 子句中包含许多可全部索引的 OR 连接术语的查询中，提高性能。

1.  添加 SQLITE_RTREE_INT_ONLY 编译时选项，以强制 R*Tree 扩展模块 使用整数而不是浮点值进行存储和计算。

1.  加强 PRAGMA integrity_check 命令，在处理多千兆字节数据库时大大减少内存使用。

1.  添加到 test_quota.c 附加模块的新接口。

1.  添加 ".trace" dot-command 到命令行 shell。

1.  允许递归调用虚拟表构造函数。

1.  在复合查询的 ORDER BY 子句上改进优化。

1.  在聚合查询中包含的聚合子查询的优化改进。

1.  Bug 修复：修复 RELEASE 命令，以防止取消待处理的查询。这修复了 3.7.11 中引入的问题。

1.  Bug 修复：只有在结果集的子集受到 UNIQUE 约束并且该子集中的列均不为空时，才不将 DISTINCT 丢弃。Ticket [385a5b56b9](https://www.sqlite.org/src/info/385a5b56b9)。

1.  Bug 修复：不要优化掉一个具有与唯一索引相同项的 ORDER BY 子句，除非这些项也是 NOT NULL。Ticket [2a5629202f](https://www.sqlite.org/src/info/2a5629202f)。

1.  SQLITE_SOURCE_ID: "2012-05-14 01:41:23 8654aa9540fe9fd210899d83d17f3f407096c004"

1.  sqlite3.c 的 SHA1 值为 57e2104a0f7b3f528e7f6b7a8e553e2357ccd2e1

### 2012-03-20 (3.7.11)

1.  增强了 INSERT 语法，允许通过 VALUES 子句插入多行。

1.  增强了 CREATE VIRTUAL TABLE 命令，以支持 IF NOT EXISTS 子句。

1.  添加了 sqlite3_stricmp()接口，作为 sqlite3_strnicmp()的对应物。

1.  添加了 sqlite3_db_readonly()接口。

1.  增加了 SQLITE_FCNTL_PRAGMA 文件控制，允许 VFS 实现添加新的 PRAGMA 语句或覆盖内置的 PRAGMA。

1.  形式为："SELECT max(x), y FROM table"的查询将返回包含最大 x 值的同一行上的 y 值。

1.  增加了对 FTS4 languageid option 的支持。

1.  文档中记录了对 FTS4 content option 的支持。实际上，这个功能从 version 3.7.9 开始就已经存在于代码中，但现在才被正式支持。

1.  待处理语句不再阻塞 ROLLBACK，而是在 ROLLBACK 后的下一次访问时返回 SQLITE_ABORT。

1.  改进了命令行 shell 中 CSV 输入的处理。

1.  修复了一个[bug](https://www.sqlite.org/src/info/b7c8682cc1)，该 bug 在 version 3.7.10 中引入，可能会导致 LEFT JOIN 在 WHERE 子句由 OR 连接的可索引项时被错误地转换为 INNER JOIN。

1.  SQLITE_SOURCE_ID: "2012-03-20 11:35:50 00bb9c9ce4f465e6ac321ced2a9d0062dc364669"

1.  sqlite3.c 的 SHA1 值为：d460d7eda3a9dccd291aed2a9fda868b9b120a10

### 2012-01-16 (3.7.10)

1.  默认的 schema format number 从 1 改为 4。这意味着，除非运行 PRAGMA legacy_file_format=ON 语句，否则新创建的数据库文件将无法被 SQLite 3.3.0 (2006-01-10) 之前的版本读取。它还意味着默认启用了 descending indices。

1.  `sqlite3_pcache_methods` 结构和 SQLITE_CONFIG_PCACHE 和 SQLITE_CONFIG_GETPCACHE 配置参数已被弃用。它们被一个新的 sqlite3_pcache_methods2 结构体和 SQLITE_CONFIG_PCACHE2 和 SQLITE_CONFIG_GETPCACHE2 配置参数所取代。

1.  在 VFS 接口中添加了 powersafe overwrite 属性。提供了 SQLITE_IOCAP_POWERSAFE_OVERWRITE I/O 能力、SQLITE_POWERSAFE_OVERWRITE 编译时选项，以及 URI filenames 的 "psow=BOOLEAN" 查询参数。

1.  增加了 sqlite3_db_release_memory() 接口和 shrink_memory pragma。

1.  增加了 sqlite3_db_filename() 接口。

1.  增加了 sqlite3_stmt_busy() 接口。

1.  增加了 sqlite3_uri_boolean() 和 sqlite3_uri_int64() 接口。

1.  如果 PRAGMA cache_size 的参数为负数 N，则意味着无论页面大小如何，要使用大约 -1024*N 字节的内存作为页面缓存。

1.  增强了默认内存分配器，利用 Windows 上的 _msize()，Mac 上的 malloc_size()和 Linux 上的 malloc_usable_size()。

1.  增强了查询规划器，以支持在 rowid 上具有范围约束的索引查询。

1.  增强了查询规划器的展平逻辑，允许 UNION ALL 复合体被提升到替换简单包装的 SELECT，即使这些复合体是连接操作。

1.  增强了查询规划器，使得 xfer 优化可以在目标表最初为空的情况下与 INTEGER PRIMARY KEY ON CONFLICT 一起使用。

1.  增强了 windows VFS，使得所有系统调用都可以使用 xSetSystemCall 接口进行覆盖。

1.  更新了"unix-dotfile" VFS，使用 mkdir()和 rmdir()代替 open()和 unlink()来锁定目录。

1.  增强了 test_quota.c 扩展以支持带配额的类似 stdio 的接口。

1.  将 unix VFS 更改为能够容忍返回请求字节不足的 read()系统调用。

1.  更改了 unix 和 windows VFS，将扇区大小报告为 4096，而不是旧的默认 512。

1.  在 TCL 接口中，为用于创建新数据库连接对象的"sqlite3" TCL 命令添加了-uri 选项。

1.  添加了 SQLITE_TESTCTRL_EXPLAIN_STMT 测试控制选项，与 SQLITE_ENABLE_TREE_EXPLAIN 编译时选项一起使用，以便使命令行 shell 能够显示它处理的 SQL 语句的 ASCII 艺术解析树，用于调试和分析。

1.  **Bug fix:** 在重新启动 WAL 时添加了额外的 xSync，以防止在断电后极不可能但理论上可能发生的数据库损坏。票号[ff5be73dee](https://www.sqlite.org/src/info/ff5be73dee)

1.  **Bug fix:** 更改 VDBE，使所有寄存器初始化为 Invalid 而不是 NULL。票号[7bbfb7d442](https://www.sqlite.org/src/info/7bbfb7d442)

1.  **Bug 修复：** 修复了由于 32 位整数溢出可能导致的问题。 票 [ac00f496b7e2](https://www.sqlite.org/src/info/ac0ff496b7e2)

1.  SQLITE_SOURCE_ID: "2012-01-16 13:28:40 ebd01a8deffb5024a5d7494eef800d2366d97204"

1.  sqlite3.c 的 SHA1 值：6497cbbaad47220bd41e2e4216c54706e7ae95d4

### 2011-11-01 (3.7.9)

1.  如果在 FTS4 中搜索的标记（MATCH 运算符的右侧）以 "^" 开头，则该标记必须是文档字段中的第一个。 **** 可能的不兼容变更 ****

1.  添加了选项 SQLITE_DBSTATUS_CACHE_HIT 和 SQLITE_DBSTATUS_CACHE_MISS 到 sqlite3_db_status() 接口。

1.  移除了对 SQLITE_ENABLE_STAT2 的支持，并用更强大的 SQLITE_ENABLE_STAT3 选项替换。

1.  对 sqlite3_analyzer 实用程序程序进行了增强，包括 --pageinfo 和 --stats 选项以及对多路复用数据库的支持。

1.  增强了 sqlite3_data_count() 接口，使其可以用来确定在准备好的语句上是否已经看到了 SQLITE_DONE。

1.  添加了 SQLITE_FCNTL_OVERWRITE 文件控制，SQLite 核心通过它向 VFS 表示当前事务将覆盖整个数据库文件。

1.  将默认的 lookaside 内存分配器 分配大小从 100 增加到 128 字节。

1.  加强了查询规划器，使其能够在 WHERE 子句中的 OR 表达式中因子化术语，以便寻找更好的索引。

1.  添加了 SQLITE_DIRECT_OVERFLOW_READ 编译时选项，导致直接从数据库文件中读取溢出页面，绕过 页面缓存。

1.  删除了 sqlite3_mprintf()字符串渲染例程中格式说明符的精度和宽度值的限制。

1.  修复了一个 bug，阻止 UTF16 编码数据库中某些虚拟表上的 ALTER TABLE ... RENAME 操作。

1.  修复了 ASCII 到浮点转换中的一个 bug，导致在转换具有极大指数的数字时性能慢且结果不正确。

1.  修复了一个 bug，在聚合查询中使用多个聚合函数且其参数包含复杂表达式的情况下，由于字符串文字内部的大小写不同而导致结果不正确。

1.  修复了一个 bug，如果其名称大写，导致 page_count 和 quick_check pragmas 工作不正确。

1.  修复了一个 bug，如果启用 count_changes pragma，导致 VACUUM 失败。

1.  修复了虚拟表实现中的一个 bug，如果在事务内部删除 FTS4 表并且之后发生 SAVEPOINT，则会导致崩溃。

1.  SQLITE_SOURCE_ID："2011-11-01 00:52:41 c7c6050ef060877ebe77b41d959e9df13f8c9b5e"

1.  sqlite3.c 的 SHA1 值为：becd16877f4f9b281b91c97e106089497d71bb47

### 2011-09-19 (3.7.8)

1.  对于非常大的表，创建索引的性能提升达到数量级的改进。

1.  改进了 Windows VFS 以更好地抵御来自防病毒软件的干扰。

1.  在存在 DISTINCT 关键字时，改进了查询计划优化。

1.  允许在 Unix VFS 中覆盖更多系统调用，以提供更好的 Chromium 沙盒支持。

1.  将前瞻缓存行的默认大小从 100 增加到 128 字节。

1.  对 test_quota.c 模块进行了增强，以便可以跟踪预先存在的文件。

1.  Bug 修复：虚拟表现在正确处理 IS NOT NULL 约束。

1.  Bug 修复：在 WHERE 子句中与索引一起使用的嵌套相关子查询的处理正确。

1.  SQLITE_SOURCE_ID: "2011-09-19 14:49:19 3e0da808d2f5b4d12046e05980ca04578f581177"

1.  sqlite3.c 的 SHA1: bfcd74a655636b592c5dba6d0d5729c0f8e3b4de

### 2011-06-28 (3.7.7.1)

1.  修复了一个导致使用 sqlite3_prepare()编译的 PRAGMA case_sensitive_like 语句在 SQLITE_SCHEMA 错误下失败的[错误](https://www.sqlite.org/src/info/25ee812710)。

1.  SQLITE_SOURCE_ID: "2011-06-28 17:39:05 af0d91adf497f5f36ec3813f04235a6e195a605f"

1.  sqlite3.c 的 SHA1: d47594b8a02f6cf58e91fb673e96cb1b397aace0

### 2011-06-23 (3.7.7)

1.  增加了对 URI 文件名的支持。

1.  添加了 sqlite3_vtab_config()接口，以支持带有虚拟表的 ON CONFLICT 子句。

1.  在虚拟表中添加了 xSavepoint、xRelease 和 xRollbackTo 方法，以支持 SAVEPOINT。

1.  更新内置的 FTS3/FTS4 和 RTREE 虚拟表，以支持 ON CONFLICT 子句和 REPLACE。

1.  避免不必要地重新解析数据库模式。

1.  增加了对 FTS4 前缀选项和 FTS4 排序选项的支持。

1.  允许在存在读/写连接时以只读模式打开 WAL 模式数据库。

1.  增加了对短文件名的支持。

1.  SQLITE_SOURCE_ID: "2011-06-23 19:49:22 4374b7e83ea0a3fbc3691f9c0c936272862f32f2"

1.  sqlite3.c 的 SHA1: 5bbe79e206ae5ffeeca760dbd0d66862228db551

### 2011-05-19 (3.7.6.3)

1.  修复了 WAL 模式的一个问题，如果 cache_size 设置得非常小（小于 10），并且 SQLite 受到内存压力，事务可能会静默回滚。

### 2011-04-17 (3.7.6.2)

1.  修正了 open(2)系统调用的函数原型，以符合 POSIX 标准。如果不修复此问题，在 NetBSD 上 pthread 将无法正常工作。

1.  SQLITE_SOURCE_ID："2011-04-17 17:25:17 154ddbc17120be2915eb03edc52af1225eb7cb5e"

1.  sqlite3.c 的 SHA1 值为：806577fd524dd5f3bfd8d4d27392ed2752bc9701

### 2011-04-13 (3.7.6.1)

1.  修复了 3.7.6 版本中的一个错误，只有在使用带有 HAVE_POSIX_FALLOCATE 编译时选项的 SQLite 构建，并且关闭了 SQLITE_ENABLE_LOCKING_MODE 时，才会出现此错误，当使用 SQLITE_FCNTL_SIZE_HINT 文件控制时。

1.  SQLITE_SOURCE_ID："2011-04-13 14:40:25 a35e83eac7b185f4d363d7fa51677f2fdfa27695"

1.  sqlite3.c 的 SHA1 值为：b81bfa27d3e09caf3251475863b1ce6dd9f6ab66

### 2011-04-12 (3.7.6)

1.  添加了 sqlite3_wal_checkpoint_v2()接口，并增强了 wal_checkpoint pragma 以支持阻塞检查点。

1.  改进了查询规划器，使其更准确地估计计划成本，并因此在选择正确的计划时做得更好，特别是在使用 SQLITE_ENABLE_STAT2 时。

1.  修复了一个错误，该错误导致在未通过调用 sqlite3_finalize()终止之前的具有失败外键约束的一个语句之后，运行具有外键约束的另一个语句时，延迟外键约束未被执行。

1.  现在对可能导致溢出的整数算术操作使用浮点数进行执行。

1.  将 VFS 对象上的版本号增加到 3，并添加了新的方法 xSetSysCall、xGetSysCall 和 xNextSysCall，用于进行全覆盖测试。

1.  将 SQLITE_MAX_ATTACHED 的最大值从 30 增加到 62（尽管默认值仍为 10）。

1.  对 FTS4 的增强：

    1.  添加了 fts4aux 表

    1.  增加了对压缩的 FTS4 内容的支持

1.  增强了 ANALYZE 命令，支持将索引名称作为其参数，以便仅分析该索引。

1.  在 unix 和类 unix 平台上添加了"unix-excl"内置 VFS。

1.  SQLITE_SOURCE_ID："2011-04-12 01:58:40 f9d43fa363d54beab6f45db005abac0a7c0c47a7"

1.  sqlite3.c 的 SHA1 值：f38df08547efae0ff4343da607b723f588bbd66b

### 2011-02-01 (3.7.5)

1.  添加了 sqlite3_vsnprintf()接口。

1.  添加了 SQLITE_DBSTATUS_LOOKASIDE_HIT，SQLITE_DBSTATUS_LOOKASIDE_MISS_SIZE 和 SQLITE_DBSTATUS_LOOKASIDE_MISS_FULL 选项，用于 sqlite3_db_status()接口。

1.  增加了 SQLITE_OMIT_AUTORESET 编译时选项。

1.  添加了 SQLITE_DEFAULT_FOREIGN_KEYS 编译时选项。

1.  更新了 sqlite3_stmt_readonly()，使其对所有已准备好的语句的结果具有明确定义，并使其与 VACUUM 一起使用。

1.  为命令行 shell 添加了"-heap"选项。

1.  修复了一个关于频繁进出 WAL 模式和 VACUUM 可能导致数据库损坏的 bug（理论上）[bug](https://www.sqlite.org/src/info/5d863f876e)。

1.  增强了 sqlite3_trace()机制，以便显示由虚拟表生成的嵌套 SQL 语句，但以注释形式显示且不展开参数。这在使用 FTS3/4 和/或 RTREE 虚拟表时极大地改善了跟踪输出。

1.  将所有内置 VFS 的 xFileControl()方法更改为在识别操作码不明确时返回 SQLITE_NOTFOUND 而不是 SQLITE_ERROR。

1.  SQLite 核心在数据库的 PRAGMA synchronous 设置为 OFF 时，会通过 SQLITE_FCNTL_SYNC_OMITTED 文件控制 调用 VFS，而不是调用 xSync。

### 2010-12-07 (3.7.4)

1.  添加了 sqlite3_blob_reopen() 接口，允许将现有的 sqlite3_blob 对象重新绑定到新行。

1.  使用新的 sqlite3_blob_reopen() 接口来改善 FTS 的性能。

1.  如果 PRAGMA locking_mode 设置为 EXCLUSIVE，则允许不支持共享内存的 VFS 访问 WAL 数据库。

1.  增强了 EXPLAIN QUERY PLAN 的功能。

1.  添加了 sqlite3_stmt_readonly() 接口。

1.  添加了 PRAGMA checkpoint_fullfsync。

1.  添加了 SQLITE_FCNTL_FILE_POINTER 选项到 sqlite3_file_control()。

1.  添加了对 FTS4 的支持，并增强了 FTS 的 matchinfo() 函数。

1.  添加了 test_superlock.c 模块，该模块提供了获取独占锁到回滚或 WAL 数据库的示例代码。

1.  添加了 test_multiplex.c 模块，该模块提供了一个示例 VFS，可以将 DB 分割为多个固定大小的文件进行多路复用（分片）。

1.  修复了与 or 优化 相关的 [非常隐晦的 bug](https://www.sqlite.org/src/info/80ba201079)。

### 2010-10-08 (3.7.3)

1.  添加了带有析构回调的 sqlite3_create_function_v2() 接口。

1.  添加了支持使用应用程序提供的回调函数定义查询区域边界的 自定义 r-tree 查询。

1.  默认页面缓存更加努力地避免使用超出 SQLITE_CONFIG_PAGECACHE 分配的内存。或者如果页面缓存从堆中分配，它也会尽量避免超过 sqlite3_soft_heap_limit64()，即使没有设置 SQLITE_ENABLE_MEMORY_MANAGEMENT。

1.  添加了 sqlite3_soft_heap_limit64() 接口，作为 sqlite3_soft_heap_limit() 的替代品。

1.  现在即使表没有索引，ANALYZE 命令也会收集表的统计信息。

1.  对查询规划器进行了调整，帮助其更好地找到每个查询的最有效查询计划。

1.  增强了内部文本到数字转换程序，使其能够与 UTF8 或 UTF16 一起工作，从而避免一些 UTF16 到 UTF8 的文本转换。

1.  修复了在 win32 系统中大型 WAL 事务导致过多内存使用的问题。

1.  加强了 VDBE 与 B-Tree 层之间的接口，使 VDBE 提供提示给 B-Tree 层，告知何时可以安全地在临时表中使用哈希而不是 B-Tree。

1.  各种文档增强。

### 2010-08-24 (3.7.2)

1.  修复了一个[古老且非常隐蔽的 bug](https://www.sqlite.org/src/info/5e10420e8d)，在使用 incremental_vacuum 时可能导致数据库 free-page list 损坏的问题。

### 2010-08-23 (3.7.1)

1.  添加了新命令 SQLITE_DBSTATUS_SCHEMA_USED 和 SQLITE_DBSTATUS_STMT_USED 到 sqlite3_db_status() 接口，用于报告连接中用于保存模式和准备语句的内存量。

1.  将数据库页面的最大大小从 32KiB 增加到 64KiB。

1.  即使右侧字符串不包含通配符，也要使用 LIKE 优化。

1.  为 unix 和 windows 都添加了 SQLITE_FCNTL_CHUNK_SIZE 动词，以便通过 sqlite3_file_control()接口使数据库文件以大块增长，以减少磁盘碎片化。

1.  修复了查询规划器中的一个错误，该错误导致在某些复杂连接中相对于 3.6.23.1 版本出现性能回退。

1.  修复了 OS/2 后端中的一个拼写错误。

1.  重构了分页器模块。

1.  编译时选项 SQLITE_MAX_PAGE_SIZE 现在被静默忽略。最大页大小已硬编码为 65536 字节。

### 2010-08-04 (3.7.0.1)

1.  修复了一个潜在的数据库损坏错误，如果版本 3.7.0 和版本 3.6.23.1 交替写入同一数据库文件时可能会发生。[Ticket [51ae9cad317a1]](https://www.sqlite.org/src/info/51ae9cad317a1)

1.  修复了与版本 3.7.0 查询规划器增强相关的性能回退。

### 2010-07-21 (3.7.0)

1.  增加了对预写式日志的支持。

1.  查询规划器增强 - 当这样做可以减少预估查询时间时，将自动创建瞬时索引。

1.  查询规划器增强 - 如果查询还包含强制正确输出顺序的 GROUP BY 子句，则 ORDER BY 将成为无操作。

1.  为 sqlite3_db_status()接口添加了 SQLITE_DBSTATUS_CACHE_USED 动词。

1.  现在逻辑数据库大小存储在数据库头部，以便可以在不损坏数据库的情况下向数据库文件末尾附加字节，并且 SQLite 可以在缺乏 ftruncate() 支持的系统上正常工作。

### 2010-03-26 (3.6.23.1)

1.  修复了 FTS3 的 offsets() 函数中的一个拼写错误。

1.  修复了一个遗漏的 "sync"，如果省略可能会导致数据库损坏，特别是当一个回滚操作正在完成时发生电源故障或操作系统崩溃时。

### 2010-03-09 (3.6.23)

1.  添加了 secure_delete pragma。

1.  添加了 sqlite3_compileoption_used() 和 sqlite3_compileoption_get() 接口，以及 compile_options pragma，以及 sqlite_compileoption_used() 和 sqlite_compileoption_get() SQL 函数。

1.  添加了 sqlite3_log() 接口，以及 SQLITE_CONFIG_LOG 动词到 sqlite3_config()。".log" 命令添加到 命令行界面。

1.  对 FTS3 进行了改进。

1.  在支持 SQLITE_OMIT_FLOATING_POINT 方面进行了改进和 bug 修复。

1.  integrity_check pragma 已增强以检测乱序的行标识。

1.  从 命令行界面 中移除了 ".genfkey" 操作符。

1.  对共托管的 Lemon LALR(1) parser generator 进行更新（这些更新没有影响到 SQLite）。

1.  各种小 bug 修复和性能提升。

### 2010-01-06 (3.6.22)

1.  修复了在查询的 WHERE 子句中使用 CAST 或 OR 运算符时可能（很少）导致查询结果不正确的错误。

1.  继续改进和完善 FTS3。

1.  其他杂项 bug 修复。

### 2009-12-07 (3.6.21)

1.  由 sqlite3_trace() 生成的 SQL 输出现在修改为包括 bound parameters 的值。

1.  针对 SQLite 的一个特定用户的使用场景进行了性能优化。通过 Valgrind 测量，CPU 操作次数减少了 12%。实际的性能提升可能因工作负载的不同而有所不同。具体变化包括：

    1.  ifnull() 和 coalesce() SQL 函数现在使用内联 VDBE 代码实现，而不是调用外部函数，因此不需要评估未使用的参数。

    1.  如果 substr() SQL 函数只计算前缀，则不必测量整个输入字符串的长度。

    1.  禁止了不必要的 OP_IsNull、OP_Affinity 和 OP_MustBeInt VDBE 操作码。

    1.  为了性能进行了各种代码重构。

1.  FTS3 扩展经过了重大的重构和清理。现在提供了新的 FTS3 文档。

1.  修复了 SQLITE_SECURE_DELETE 编译时选项，确保即使应用了 truncate 优化，内容也会被删除。

1.  对 命令行界面 中的“点命令”处理进行了改进。

1.  其他一些小的 bug 修复和文档增强。

### 2009-11-04 (3.6.20)

1.  优化器增强：当 LIKE 操作符的右操作数绑定发生变化或任何范围约束在 SQLITE_ENABLE_STAT2 下发生变化时，prepared statements 会自动重新编译。

1.  各种小 bug 修复和文档增强。

### 2009-10-30 (3.6.16.1)

1.  对版本 3.6.16 的一个小补丁，修复了 [OP_If bug](https://www.sqlite.org/src/info/6b00e0a34c)。

### 2009-10-14 (3.6.19)

1.  增加了对 外键约束 的支持。外键约束默认禁用。使用 foreign_keys pragma 来启用它们。

1.  将 IS 和 IS NOT 操作符泛化为可以接受任意表达式作为其右操作数。

1.  TCL 接口已增强，当链接到 TCL 8.6 或更高版本时，使用[TCL 解释器的非递归引擎（NRE）](http://www.tcl-lang.org/cgi-bin/tct/tip/322.html)接口。

1.  修复了 3.6.18 中引入的一个错误，可能会导致尝试在只读数据库上写入时出现段错误。

### 2009-09-11 (3.6.18)

1.  SQLite 源代码的版本控制已从 CVS 过渡到[Fossil](http://www.fossil-scm.org/)。

1.  查询规划器增强。

1.  编译时选项 SQLITE_ENABLE_STAT2 会导致 ANALYZE 命令收集每个索引的一个小直方图，以帮助 SQLite 在竞争的范围查询索引中做出更好的选择。

1.  可以使用 PRAGMA recursive_triggers 语句启用递归触发器。

1.  当递归触发器启用时，删除触发器在由于 REPLACE 冲突解决而删除行时触发。此功能仅在启用递归触发器时才可用。

1.  添加了 SQLITE_OPEN_SHAREDCACHE 和 SQLITE_OPEN_PRIVATECACHE 标志，用于 sqlite3_open_v2()，用于覆盖单个数据库连接的全局共享缓存模式设置。

1.  添加了改进的版本识别功能：C 预处理器宏 SQLITE_SOURCE_ID，C/C++接口 sqlite3_sourceid()，以及 SQL 函数 sqlite_source_id()。

1.  触发器修复了一个模糊的错误([[efc02f9779]](https://www.sqlite.org/src/info/efc02f9779))。

### 2009-08-10 (3.6.17)

1.  公开了 sqlite3_strnicmp()接口，供扩展和应用程序使用。

1.  移除了对虚拟表和共享缓存模式的限制。现在可以同时使用虚拟表和共享缓存。

1.  在支持提供 100% 分支测试覆盖率 的过程中，简化了许多代码并修复了模糊 bug。

### 2009-06-27 (3.6.16)

1.  修复了偶尔会导致带有自修改触发器的索引表的 INSERT 或 UPDATE 操作失败的 bug (票号 #3929)。

1.  其他次要 bug 修复和性能优化。

### 2009-06-15 (3.6.15)

1.  重构了 SQL 表达式的内部表示，以在嵌入式平台上减少内存使用。

1.  减少了使用的堆栈空间量。

1.  修复了 HP/UX 和 Sparc 上的 64 位对齐 bug。

1.  sqlite3_create_function() 接口系列现在在传递无效参数组合时返回 SQLITE_MISUSE，而不是 SQLITE_ERROR。

1.  当使用 CREATE TABLE ... AS SELECT ... 创建新表时，列的数据类型为简化的 SQLite 数据类型 (TEXT、INT、REAL、NUMERIC 或 BLOB)，而不是源表的原始数据类型的副本。

1.  解决了检查热回滚日志时的竞争条件。

1.  sqlite3_shutdown() 接口在 Windows 下释放所有互斥锁。

1.  增强了对损坏数据库文件的健壮性。

1.  继续改进测试套件，并修复测试套件改进中发现的模糊 bug 和不一致性。

### 2009-05-25 (3.6.14.2)

1.  修复了在 version 3.6.14 中引入的代码生成器 bug。在模糊情况下，此 bug 可能导致查询结果不正确。票号 #3879。

### 2009-05-19 (3.6.14.1)

1.  修复了 group_concat() 中的一个 bug，票号 #3841

1.  修复了分页缓存中的性能 bug，票号 #3844

1.  修复了 sqlite3_backup 实现中的一个 bug，可能导致备份数据库损坏。票号 #3858。

### 2009-05-07 (3.6.14)

1.  添加了可选的 异步 VFS 模块。

1.  增强了查询优化器，以便 virtual tables 能够在 WHERE 子句中使用 OR 和 IN 运算符。

1.  在 btree 和 pager 层的速度改进。

1.  添加了 SQLITE_HAVE_ISNAN 编译时选项，这将导致使用标准数学库中的 isnan()函数，而不是 SQLite 自己的自制 NaN 检查器。

1.  无数次要 bug 修复、文档改进、新的和改进的测试用例、代码简化和清理。

### 2009-04-13 (3.6.13)

1.  修复了在版本 3.6.12 中的一个 bug，当在空数据库的 sqlite_master 表上运行 count(*)时导致段错误。Ticket #3774。

1.  修复了在版本 3.6.12 中的一个 bug，该 bug 导致在使用 DEFAULT 值插入表时，DEFAULT 值表达式中存在函数时发生段错误。Ticket #3791。

1.  修复了在 Sparc 上的数据结构对齐问题。Ticket #3777。

1.  其他次要的 bug 修复。

### 2009-03-31 (3.6.12)

1.  修复了一个 bug，在内存数据库中回滚 incremental_vacuum 时导致数据库损坏。Ticket #3761。

1.  添加了 sqlite3_unlock_notify()接口。

1.  添加了 reverse_unordered_selects pragma。

1.  Windows 上的默认页面大小会自动调整以匹配底层文件系统的能力。

1.  在 CLI 中添加了新的".genfkey"命令，用于生成实现外键约束的触发器。

1.  "count(*)"查询的性能改进。

1.  减少了堆内存的使用量，特别是 TRIGGERs。

### 2009-02-18 (3.6.11)

1.  添加了热备份接口。

1.  在 CLI 中添加了新的命令".backup"和".restore"。

1.  在 TCL 接口中添加了新方法 backup 和 restore。

1.  对 syntaxdiagrams.html 进行了改进。

1.  各种次要错误修复。

### 2009-01-15 (3.6.10)

1.  修复了一个可能导致数据库损坏的缓存一致性问题。票号 #3584。

### 2009-01-14 (3.6.9)

1.  修复了两个 bug，它们结合在一起可能导致查询结果不正确。这两个 bug 单独来看是无害的；只有在它们一起出现时才会引起问题。票号 #3581。

### 2009-01-12 (3.6.8)

1.  添加了对嵌套事务的支持 lang_savepoint.html。

1.  加强了查询优化器，使其能够有效地处理 WHERE 子句中的 OR-connected constraints，利用多个索引。

1.  添加了对 FTS3 查询模式中使用括号的支持，使用 SQLITE_ENABLE_FTS3_PARENTHESIS 编译时选项。

### 2008-12-16 (3.6.7)

1.  在 os_unix.c 中重新组织 Unix 接口。

1.  在 Mac OS X 上添加了"Proxy Locking"支持。

1.  更改了 sqlite3_auto_extension()接口的原型，保证向后兼容，但可能会在新版本的应用程序中引起警告。

1.  更改了 sqlite3_vfs 对象的 xDlSym 方法签名，保证向后兼容，但可能会引起编译器警告。

1.  添加了不必要的强制转换和变量初始化，以抑制烦人的编译器警告。

1.  修复了各种次要错误。

### 2008-11-26 (3.6.6.2)

1.  修复了 B 树删除算法中的一个错误，可能导致数据库损坏。该错误首次在 version 3.6.6 的[5899]提交中出现，时间为 2008-11-13。

1.  修复了在磁盘 I/O 错误后可能导致的内存泄漏。

### 2008-11-22 (3.6.6.1)

1.  修复了页缓存中的一个错误，可能导致回滚后数据库损坏。此错误首次出现在 version 3.6.4 中。

1.  另外两个非常次要的 bug 修复。

### 2008-11-19 (3.6.6)

1.  修复了一个宏定义问题，导致 malloc.html#memsys5 无法编译。

1.  修复了虚拟表提交机制中的问题，导致 FTS3 崩溃。票号 #3497。

1.  添加了 应用程序定义的页面缓存。

1.  增加了对 VxWorks 的内置支持。

### 2008-11-12 (3.6.5)

1.  将 journal_mode pragma 添加了 MEMORY 选项。

1.  添加了 sqlite3_db_mutex() 接口。

1.  添加了编译时选项 SQLITE_OMIT_TRUNCATE_OPTIMIZATION。

1.  修复了 truncate optimization，使得 sqlite3_changes() 和 sqlite3_total_changes() 接口以及 count_changes pragma 返回正确的值。

1.  添加了 sqlite3_extended_errcode() 接口。

1.  现在即使存在待处理的查询，COMMIT 命令也会成功执行。如果存在待处理的增量 BLOB I/O 请求，则返回 SQLITE_BUSY。

1.  当尝试在仍有一个或多个查询挂起时 ROLLBACK 时，错误代码更改为 SQLITE_BUSY（而不是 SQLITE_ERROR）。

1.  删除了对实验性内存分配器 memsys4 和 memsys6 的所有支持。

1.  添加了编译时选项 SQLITE_ZERO_MALLOC。

### 2008-10-15 (3.6.4)

1.  为 DELETE 和 UPDATE 语句添加对 LIMIT 和 ORDER BY 子句的选项支持。仅在 SQLite 编译时启用了 SQLITE_ENABLE_UPDATE_DELETE_LIMIT 时有效。

1.  添加了用于性能监控的 sqlite3_stmt_status() 接口。

1.  添加了 INDEXED BY 子句。

1.  在 Mac OS X 上，默认启用了 LOCKING_STYLE 扩展。

1.  添加了 TRUNCATE 选项到 PRAGMA journal_mode。

1.  在 B-Tree 层的树平衡逻辑中进行了性能增强。

1.  添加了用于自动生成触发器以强制执行外键约束的**genfkey**程序的[源代码](https://www.sqlite.org/src/finfo?name=tool/genfkey.c)和[文档](https://www.sqlite.org/src/finfo?name=tool/genfkey.README)。

1.  添加了 SQLITE_OMIT_TRUNCATE_OPTIMIZATION 编译时选项。

1.  SQL 语言文档已转换为使用 syntax diagrams，而非 BNF。

1.  其他次要错误修复

### 2008-09-22 (3.6.3)

1.  修复了由前一个版本引入的 SELECT DISTINCT 逻辑中的错误。

1.  其他次要错误修复

### 2008-08-30 (3.6.2)

1.  将页面管理器子系统拆分为单独的页面管理器和页缓存子系统。

1.  将标识符解析过程分离为单独的文件。

1.  Bug fixes

### 2008-08-06 (3.6.1)

1.  添加了 lookaside 内存分配器，在某些工作负载上可以提高超过 15%的速度。（具体效果可能有所不同。）

1.  添加了用于控制默认 lookaside 配置的 sqlite3_config()的 SQLITE_CONFIG_LOOKASIDE 动词。

1.  添加了 sqlite3_status()接口的 SQLITE_STATUS_PAGECACHE_SIZE 和 SQLITE_STATUS_SCRATCH_SIZE 动词。

1.  修改了 SQLITE_CONFIG_PAGECACHE 和 SQLITE_CONFIG_SCRATCH，以消除在缓冲区大小计算中的"+4"魔数。

1.  添加了用于在每个数据库连接上单独控制和监视 lookaside 分配器的 sqlite3_db_config()和 sqlite3_db_status()接口。

1.  大量其他性能增强

1.  杂项次要错误修复

### 2008-07-16 (3.6.0 beta)

1.  修改了虚拟文件系统接口，以支持更广泛的嵌入式系统。详细信息请参见 35to36.html。***潜在的不兼容更改***

1.  所有用于控制编译时选项的 C 预处理宏现在都以前缀"SQLITE_"开头。这可能需要更改使用自定义 makefile 和自定义编译时选项编译 SQLite 的应用程序，因此我们标记这一点为***潜在的不兼容更改***

1.  编译时选项`SQLITE_MUTEX_APPDEF`不再受支持。现在可以使用 sqlite3_config()接口并使用 SQLITE_CONFIG_MUTEX 动词在运行时添加替代的互斥实现。***潜在的不兼容更改***

1.  将包含右侧表达式中 NULL 的 IN 和 NOT IN 操作符的处理与 SQL 标准及其他 SQL 数据库引擎进行了兼容。这是一个错误修复，但由于可能会破坏依赖于旧有错误行为的遗留应用程序，因此我们标记它为***潜在的不兼容更改***

1.  对复合子查询生成的结果列名进行了简化，只显示原始表的列名而省略表名。这使得 SQLite 的操作更像其他 SQL 数据库引擎。

1.  添加了 sqlite3_config()接口，用于对整个 SQLite 库进行运行时配置。

1.  添加了 sqlite3_status()接口，用于查询关于整个 SQLite 库及其子系统的运行时状态信息。

1.  添加了 sqlite3_initialize()和 sqlite3_shutdown()接口。

1.  添加了 SQLITE_OPEN_NOMUTEX 选项到 sqlite3_open_v2()。

1.  添加了 PRAGMA page_count 命令。

1.  添加了 sqlite3_next_stmt()接口。

1.  添加了一个新的 R*Tree 虚拟表

### 2008-05-14 (3.5.9)

1.  增加了对日志模式 PRAGMA 和持久日志的*实验性*支持。

1.  持久日志模式是独占锁定模式下的默认行为。

1.  修复了左连接的性能退化问题（参见票号 #3015），该问题错误地引入在版本 3.5.8 中。

1.  性能增强：重新设计了用于解释和渲染可变长度整数的内部例程。

1.  修复了 sqlite3_mprintf()中的缓冲区溢出问题，当传递没有零终止符的字符串给“%.*s”时发生。

1.  在处理过程中，始终将 IEEE 浮点 NaN 值转换为 NULL。（票号 #3060）

1.  确保当连接在保留锁上阻塞时，能够在释放锁后继续操作。（票号 #3093）

1.  现在，“配置”脚本应该会自动为 Unix 系统配置大文件支持。当遇到大文件且未启用大文件支持时，改进了错误消息。

1.  避免在磁盘满或 I/O 错误后缓存页面泄漏。

1.  还有许多次要的错误修复和性能增强……

### 2008-04-16 (3.5.8)

1.  通过 sqlite3_randomness()接口公开 SQLite 的内部伪随机数生成器（PRNG）。

1.  新接口 sqlite3_context_db_handle()返回调用应用定义的 SQL 函数的数据库连接句柄。

1.  新接口 sqlite3_limit()允许在每个连接和运行时设置大小和长度限制。

1.  改进了崩溃鲁棒性：将数据库页面大小写入回滚日志头部。

1.  允许 VACUUM 命令更改数据库文件的页面大小。

1.  允许 VFS 的 xAccess（）方法返回-1 以表示内存分配错误。

1.  性能改进：OP_IdxDelete 操作码使用非压缩记录，避免为每个索引记录删除调用一个 OP_MakeRecord 操作码。

1.  性能改进：常量子表达式从循环中分离出来。

1.  性能改进：OP_Column 的结果被重复使用，而不是发出多个 OP_Column 操作码。

1.  修复了 RTRIM 排序序列中的一个错误。

1.  修复了导致 Firefox 崩溃的 SQLITE_SECURE_DELETE 选项中的错误。确保在每次发布之前始终测试 SQLITE_SECURE_DELETE。

1.  其他杂项性能增强。

1.  其他杂项小错误修复。

### 2008-03-17（3.5.7）

1.  修复了一个错误（票号＃2927），该错误出现在 3.5.5 版本新的虚拟机代码中，用于复合选择的寄存器分配。

1.  ALTER TABLE 使用双引号而不是单引号引用文件名。

1.  使用 WHERE 子句在 UPDATE 或 DELETE 语句中减少物化视图的大小。（优化）

1.  如果外部查询是聚合并且内部查询包含 ORDER BY，则不应用平坦化优化。（票号＃2943）

1.  额外的 OS/2 更新。

1.  添加了一个实验性的二次幂、首次适配内存分配器。

1.  从代码中删除所有 sprintf（）的实例。

1.  接受“Z”作为日期字符串末尾的协调世界时时区。

1.  修复了在 LIKE 优化器中的一个错误，当第一个通配符之前的最后一个字符是大写字母“Z”时发生。

1.  添加了“bitvec”对象，用于跟踪已记录哪些页面。特别适用于大型数据库文件，提高速度并减少内存消耗。

1.  修复了在 Mac OS X 上使 SQLITE_ENABLE_LOCKING_STYLE 宏再次起作用的错误。

1.  将语句日志存储在临时文件目录中，而不是与数据库文件一起存放。

1.  对配置脚本进行了许多改进和清理。

### 2008-02-06（3.5.6）

1.  修复了一个 bug (票号 #2913)，阻止虚拟表在 LEFT JOIN 中工作。该问题是在 3.5.5 发布前不久引入的。

1.  更新了 OS/2 移植层。

1.  添加了新的 sqlite3_result_error_code() API，并在 ATTACH 的实现中使用它，以便在 ATTACH 失败时返回正确的错误代码。

### 2008-01-31 (3.5.5)

1.  将底层虚拟机转换为基于寄存器而不是基于堆栈的机器。唯一可见的用户更改在 EXPLAIN 的输出中。

1.  添加了内建的 RTRIM 排序序列。

### 2007-12-14 (3.5.4)

1.  修复了在 UPDATE 或 DELETE 中发生的严重 bug，当 OR REPLACE 子句或触发器导致删除同一表中的行时。 (参见票号 #2832。) 这个 bug 最有可能导致段错误，尽管数据库损坏也是可能的。

1.  将 ORDER BY 的处理带入符合 SQL 标准的情况，即结果别名和表列名冲突的情况。正确的行为是优先选择结果别名。旧版本的 SQLite 错误地选择了表列。 (参见票号 #2822。)

1.  VACUUM 命令保留了 legacy_file_format pragma 的设置。 (票号 #2804。)

1.  产品化并正式支持 group_concat() SQL 函数。

1.  优化了一些 IN 操作符表达式的优化。

1.  添加了通过设置 auto_vacuum pragma 并对数据库进行 VACUUM 来更改数据库的 自动清理 状态的功能。

1.  FTS3 中的前缀搜索更加高效。

1.  放宽 CLI 中的 SQL 语句长度限制，以便可以播放具有非常大的 BLOB 和字符串的数据库的 ".dump" 输出以重新创建数据库。

1.  其他一些小的错误修复和优化。

### 2007-11-27 (3.5.3)

1.  将网站和文档文件从源代码树中移出，放入[单独的 CM 系统](https://www.sqlite.org/docsrc/)。

1.  修复了 INSERT INTO ... SELECT ...语句中存在已久的多行选择问题。

1.  修复了在 BEFORE 触发器中使用 RAISE(IGNORE)存在的长期 bug。

1.  修正了~运算符的操作符优先级。

1.  在 Win32 平台上，当尝试删除不存在的文件时，不返回错误。

1.  允许排序序列名称用引号引起来。

1.  修改了 TCL 接口，使用 sqlite3_prepare_v2()。

1.  修复了在 malloc()失败后可能出现的多个错误。

1.  sqlite3_step()在传入 NULL 参数时，不再崩溃，而是返回 SQLITE_MISUSE。

1.  FTS3 现在完全使用 SQLite 内存分配器。可以将 FTS3 集成附加到 SQLite 集成中，生成包含两者的超级集成。

1.  DISTINCT 关键字现在有时会在适当的索引可用且优化器认为其使用有利时使用索引。

### 2007-11-05 (3.5.2)

1.  放弃了 SQLITE_OMIT_MEMORY_ALLOCATION 编译时选项的支持。

1.  在 Windows 下始终使用 FILE_FLAG_RANDOM_ACCESS 打开文件。

1.  内置的 SUBSTR()函数的第三个参数现在是可选的。

1.  修复了一个 bug：在模式更改后重新解析模式时，不要调用授权器。

1.  在 mem3.c 中添加了实验性的无 malloc 内存分配器。

1.  虚拟机将 64 位整数和浮点常数存储在二进制而非文本中，以提升性能。

1.  修复了 test_async.c 中的竞争条件。

1.  向 CLI 添加了".timer"命令。

### 2007-10-04 (3.5.1)

1.  ***Nota Bene:** 我们在此版本中不使用"alpha"或"beta"术语，因为代码稳定，如果使用这些术语，没有人会升级。然而，我们仍保留在未来版本中对新 VFS 接口进行不兼容更改的权利。*

1.  修复了处理 SQLITE_FULL 错误时可能导致数据库损坏的错误。票号#2686。

1.  `test_async.c`驱动现在在同一数据库上被多个进程同时使用时可以正确执行完全文件锁定。

1.  CLI 忽略行尾的空白字符（包括注释）。

1.  确保查询优化器检查复合 SELECT 语句的所有项的依赖关系。票号#2640。

1.  添加演示代码，展示如何为不带文件系统的原始大容量存储构建 VFS。

1.  在 VFS 层的`xGetTempname()`方法中增加了输出缓冲区大小参数。

1.  当启动新事务时，页面器中的粘性 SQLITE_FULL 或 SQLITE_IOERR 错误会被重置。

### 2007-09-04（3.5.0）alpha

1.  重新设计操作系统接口层。详见 34to35.html。***可能存在不兼容的改动***。

1.  sqlite3_release_memory()、sqlite3_soft_heap_limit()和 sqlite3_enable_shared_cache()接口现在可以跨所有进程中的所有线程工作，而不仅限于它们被调用的单个线程。***可能存在不兼容的改动***。

1.  增加了 sqlite3_open_v2()接口。

1.  重新实现了内存分配子系统，并使其在编译时可替换。

1.  创建了一个新的互斥子系统，并使其在编译时可复制。

1.  现在可以通过不同线程同时使用同一个数据库连接。

### 2007-08-13（3.4.2）

1.  修复了在自动清理模式下执行 ROLLBACK 命令并设置非常小的 sqlite3_soft_heap_limit 时可能出现的数据库损坏错误。票号#2565。

1.  添加能够通过小型 sqlite3_soft_heap_limit 运行完整回归测试的功能。

1.  修复了使用较小软堆限制时的其他小问题。

1.  对[GCC bug 32575](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=32575)的解决方法。

1.  改进了对误用的聚合函数的错误检测。

1.  改进合并生成脚本，以便所有符号都以 SQLITE_PRIVATE 或 SQLITE_API 为前缀。

### 2007-07-20 (3.4.1)

1.  修复 VACUUM 中的一个 bug，如果两个进程同时连接到数据库，并且一个进程先执行了 VACUUM，然后另一个进程修改了数据库，可能导致数据库损坏。

1.  当计算要在表达式上使用的排序序列时，现在将"+column"表达式视为与"column"相同。

1.  在 TCL 语言接口中，"@variable"代替"$variable"将始终绑定为 blob。

1.  添加了 PRAGMA freelist_count 以确定当前 freelist 的大小。

1.  PRAGMA auto_vacuum=incremental 设置现在是持久的。

1.  在 Unix 下将 FD_CLOEXEC 添加到所有打开的文件。

1.  修复了在应用于降序索引时的 min()/max()优化中的一个 bug。

1.  确保在 64 位机器上 TCL 语言接口能正确处理 64 位整数。

1.  允许在 SQL 语句中使用值为-9223372036854775808 的整数字面量。

1.  在虚拟表中添加了“隐藏”列的功能。

1.  在合并编译中的所有内部函数上使用宏 SQLITE_PRIVATE（默认为"static"）。

1.  将可插拔的标记器和[ICU](https://icu.unicode.org)的分词支持添加到 FTS2 中。

1.  其他小 bug 修复和文档增强。

### 2007-06-18 (3.4.0)

1.  修复一个 bug，如果在显式事务的中间发生 SQLITE_BUSY 错误，并且稍后提交了该事务，可能导致数据库损坏。Ticket #2409。

1.  修复一个 bug，如果自动 VACUUM 模式打开，并且在事务中的缓存溢出之后的 CREATE TABLE 或 CREATE INDEX 语句遭遇 malloc()失败，可能导致数据库损坏。参见 ticket #2418。

1.  明确添加了 SQLite 可处理的大小和数量上限，详见 上限页面。这一变更可能会对极端使用 SQLite 的应用程序造成兼容性问题，这也是当前版本为 3.4.0 而不是 3.3.18 的原因。

1.  增加了对 增量 BLOB I/O 的支持。

1.  添加了 sqlite3_bind_zeroblob() API 和 zeroblob() SQL 函数。

1.  增加了对 增量 VACUUM 的支持。

1.  添加了 SQLITE_MIXED_ENDIAN_64BIT_FLOAT 编译时选项，以支持具有特殊字节顺序的 ARM7 处理器。

1.  从核心库中移除了所有 sprintf() 和 strcpy() 的实例。

1.  增加了对 [Unicode 国际组件 (ICU)](https://icu.unicode.org) 在全文搜索扩展中的支持。

1.  在 Windows 操作系统驱动程序中，如果尝试获取排他锁失败，则重新获取共享锁。票号 #2354。

1.  修复 REPLACE() 函数，使其在第二个参数为空字符串时返回 NULL。票号 #2324。

1.  在 sqlite3_column_blob() 和相关 API 中，记录类型转换的危险，并修复不必要的类型转换。票号 #2321。

1.  对 TRIM() 函数进行国际化。票号 #2323。

1.  在移动可能重叠的内存区域之间时，使用 memmove() 而不是 memcpy()。票号 #2334。

1.  修复涉及子查询的优化器 bug，在同时具有 ORDER BY 和 LIMIT 子句的复合 SELECT 中。票号 #2339。

1.  确保 sqlite3_snprintf() 接口在缓冲区大小小于 1 时不会对缓冲区进行零终止。票号 #2341。

1.  修复内置的 printf 逻辑，使其在打印浮点 NaN 时输出 "NaN" 而不是 "Inf"。票号 #2345。

1.  在将 BLOB 转换为 TEXT 时，使用主数据库的文本编码。票号 #2349。

1.  在将整数转换为 NUMERIC 时，尽量保留完整精度。票号 #2364。

1.  修复了对 UTF16 代码点 0xE000 处理的 bug。

1.  在查询优化器中匹配 WHERE 约束与索引时，考虑显式的 collate 子句。Ticket #2391

1.  修复查询优化器，以正确处理左连接的 ON 子句中的常量表达式。Ticket #2403

1.  修复查询优化器，以正确处理 rowid 与 NULL 的比较。Ticket #2404

1.  修复可能由恶意 SQL 语句引起的许多潜在段错误。

### 2007-04-25 (3.3.17)

1.  当数据库头的"write_version"值大于库理解的值时，将数据库设置为只读，而不是不可读。

1.  其他次要 bug 修复

### 2007-04-18 (3.3.16)

1.  修复一个导致在 UNIQUE 列中出现 NULL 时导致 VACUUM 失败的错误。

1.  恢复在 Version 3.3.14 中添加的性能改进，但在 Version 3.3.15 中退化的部分。

1.  修复在子查询中处理 ORDER BY 表达式在复合 SELECT 语句中的问题。

1.  修复在多线程环境中销毁 WinCE 上锁时可能导致的潜在段错误。

1.  文档更新。

### 2007-04-09 (3.3.15)

1.  修复在 3.3.14 中引入的一个 bug，导致 CREATE TEMP TABLE 回滚时使数据库连接卡住。

1.  修复一个导致在降序查询被数据库更改中断时返回额外 NULL 行的 bug。

1.  触发器上的 FOR EACH STATEMENT 子句现在会导致语法错误。以前会被静默忽略。

1.  修复一个微不足道且相对无害的问题，可能会在 I/O 错误后导致资源泄漏。

1.  对测试套件进行了许多改进。测试覆盖率现在超过了 98%

### 2007-04-02 (3.3.14)

1.  修复一个 bug（ticket #2273），当 IN 运算符用于两列索引的一个术语时，右边的 IN 运算符包含 NULL 时可能导致段错误。

1.  添加了一个用于确定底层介质扇区大小的新 OS 接口方法：sqlite3OsSectorSize()。

1.  一种针对形如 INSERT INTO *table1* SELECT * FROM *table2* 的语句的新算法更快且减少了碎片化。VACUUM 使用这种形式的语句，因此运行更快且碎片化更少。

1.  通过减少磁盘 I/O 实现的性能增强：

    1.  在删除行时不要读取溢出链的最后一页 - 只需将该页添加到空闲列表。

    1.  不要将正在删除的页面存储在回滚日志中。

    1.  不要读取从空闲列表提取的页面的（无意义的）内容。

    1.  除非另一个进程更改了底层数据库文件，否则不要刷新页缓存（从而避免缓存重新填充）。

    1.  在排他访问模式下提交事务或提交 TEMP 数据库时，截断而不是删除回滚日志。

1.  增加了使用 "PRAGMA locking_mode=EXCLUSIVE" 支持独占访问模式的功能。

1.  在嵌入式平台上具有堆栈空间限制时，使用堆空间而不是栈空间来存储大缓冲区在页中 - 这非常有用。

1.  添加了一个名为 "sqlite3.c" 的 makefile 目标，用于构建包含核心 SQLite 库 C 代码的单个文件的合并文件。

1.  使用 GCC 选项“-fstrict-aliasing”编译时，确保库正常工作。

1.  删除了残留的 SQLITE_PROTOCOL 错误。

1.  提高了测试覆盖率，修复了其他一些小错误，修补了内存泄漏，重构了代码并/或建议在某些地方进行更轻松的阅读。

### 2007-02-13 (3.3.13)

1.  在 sqlite3_analyzer 输出中添加“碎片化”测量。

1.  添加 COLLATE 操作符，用于显式设置表达式使用的排序顺序。此功能在额外测试后被视为实验性功能。

1.  允许在联接中最多使用 64 个表 - 旧的限制是 32。

1.  添加了两个新的实验性函数：randomblob() 和 hex()。它们的预期用途是帮助生成 [UUIDs](http://en.wikipedia.org/wiki/UUID)。

1.  修复了 PRAGMA count_changes 导致在具有触发器的表上更新时产生不正确结果的问题。

1.  修复了在具有唯一索引约束的连接中，ORDER BY 子句优化器中的一个 bug。

1.  修复了 TCL 接口的"copy"方法中的一个 bug。

1.  在 fts1 和 fts2 模块中修复了 bug。

### 2007-01-27 (3.3.12)

1.  修复了在版本 3.3.9 中添加的 IS NULL 优化中的另一个 bug。

1.  修复了在深度嵌套视图中发生的断言错误。

1.  限制了 PRAGMA integrity_check 生成的输出量。

1.  对语法进行了轻微的更改，以支持更广泛的编译器。

### 2007-01-22 (3.3.11)

1.  修复了新的 sqlite3_prepare_v2() API 实现中的另一个 bug。我们最终会搞定它的……

1.  修复了在版本 3.3.9 中添加的 IS NULL 优化中的另一个 bug - 此 bug 导致包含在 LEFT JOIN 的 WHERE 子句中，对左连接的右表进行 IS NULL 约束时产生不正确的结果。

1.  在 WinCE 中，将 AreFileApisANSI()设置为无操作宏，因为 WinCE 不支持此函数。

### 2007-01-09 (3.3.10)

1.  修复了新的 sqlite3_prepare_v2() API 实现中可能导致段错误的 bug。

1.  修复了 strftime()函数中的 1 秒钟舍入误差。

1.  增强了 Windows 操作系统层，提供详细的错误代码。

1.  解决了一个 Win2k 的问题，使得 SQLite 可以使用单字符数据库文件名。

1.  user_version 和 schema_version pragmas 现在在结果集中正确设置其列名。

1.  更新了文档

### 2007-01-04 (3.3.9)

1.  修复了 pager.c 中的 bug，如果两个进程同时尝试恢复热日志，可能会导致数据库损坏。

1.  添加了 sqlite3_prepare_v2() API。

1.  修复了命令行 shell 中".dump"命令，以再次显示索引、触发器和视图。

1.  更改 table_info pragma，如果没有默认值则返回 NULL。

1.  支持在 win95 文件名中使用非 ASCII 字符。

1.  查询优化器增强：

    1.  优化器更好地使用索引满足基于整数主键排序的 ORDER BY 子句。

    1.  使用索引满足 WHERE 子句中的 IS NULL 操作符。

    1.  修复了一个导致优化器错过 OR 优化机会的 bug。

    1.  优化器在 FROM 子句中即使存在 LEFT JOIN，也有更多重排表的自由度。

1.  增加了对 WinCE 的扩展加载支持。

1.  允许在表定义的 DEFAULT 子句中使用约束名。

1.  添加了“.bail”命令到命令行 shell。

1.  使命令行 shell 输出的 CSV（逗号分隔值）更符合通用实践。

1.  添加了实验性的 FTS2 模块。

1.  使用 sqlite3_mprintf() 替代 strdup()，以避免对 libc 的依赖。

1.  VACUUM 使用官方 TEMP 文件夹中的临时文件，而不是与原始数据库相同的目录。

1.  Windows 上临时文件名前缀从“sqlite”更改为“etilqs”。

### 2006-10-09（3.3.8）

1.  支持使用 FTS1 模块进行全文搜索。

1.  增加了 Mac OS X 锁定补丁（beta - 默认禁用）。

1.  引入扩展错误代码并添加各种类型的 I/O 错误代码。

1.  增加了对 CREATE/DROP TRIGGER/VIEW 的 IF EXISTS 支持。

1.  修复了回归测试套件，使其与 Tcl8.5 兼容。

1.  增强了 sqlite3_set_authorizer() 以提供对 SQL 函数调用的通知。

1.  添加了实验性 API：sqlite3_auto_extension()。

1.  各种次要 bug 修复。

### 2006-08-12（3.3.7）

1.  增加了对虚拟表的支持（beta 版）。

1.  增加了对动态加载扩展的支持（beta 版）。

1.  sqlite3_interrupt() 例程可以为不同线程调用。

1.  添加了 MATCH 操作符。

1.  默认文件格式现在是 1。

### 2006-06-06（3.3.6）

1.  在 Windows 上与病毒扫描程序更兼容。

1.  更快的 :memory: 数据库。

1.  修复了 UTF-8 到 UTF-16 转换中的一个难以察觉的段错误。

1.  添加了 OS/2 的驱动程序。

1.  对于聚合查询，正确返回列的元信息。

1.  改进了 EXPLAIN QUERY PLAN 的输出。

1.  现在子查询中可以使用 LIMIT 0。

1.  查询优化器中的错误修复和性能增强。

1.  正确处理 ATTACH 和 DETACH 中的空文件名。

1.  改进了解析器中的语法错误消息。

1.  修复了 IN 运算符的类型强制规则。

### 2006-04-05 (3.3.5)

1.  CHECK 约束正确使用冲突解决算法。

1.  SUM() 函数在整数溢出时抛出错误。

1.  在复合查询中选择列名时应从最左边的 SELECT 开始而不是最右边。

1.  sqlite3_create_collation() 函数支持 SQLITE_UTF16_ALIGNED 标志。

1.  SQLITE_SECURE_DELETE 编译时选项使删除操作用零覆盖旧数据。

1.  检测 abs() 中的整数溢出。

1.  random() 函数提供了 64 位的随机性而不仅仅是 32 位。

1.  解析器检测并报告自动机栈溢出。

1.  更改 round() 函数返回 REAL 而不是 TEXT。

1.  允许 LEFT OUTER JOIN 的左表中包含聚合子查询的 WHERE 子句项。

1.  在文本到数值转换中跳过前导空格。

1.  各种小错误和文档拼写修正以及性能增强。

### 2006-02-11 (3.3.4)

1.  修复了 Unix 互斥锁实现中的错误，这可能导致多线程系统中的死锁。

1.  修复了 64 位机器上的对齐问题。

1.  添加了 fullfsync pragma。

1.  修复了一个优化器 bug，可能导致一些不寻常的 LEFT OUTER JOIN 出现错误结果。

1.  SUM 函数检测整数溢出并转换为使用浮点数累积近似结果。

1.  主机参数名称可以以 '@' 开头，以兼容 SQL Server。

1.  其他杂项 bug 修复。

### 2006-01-31 (3.3.3)

1.  移除了在创建索引时 ON CONFLICT 子句的支持 - 它从未正确工作过，因此不应该造成任何向后兼容性问题。

1.  现在通知授权回调 ALTER TABLE ADD COLUMN 命令

1.  对 TEMP 数据库模式进行任何更改后，所有预备语句都无效，并且必须使用新的 sqlite3_prepare() 调用重新创建。

1.  为版本 3.3 的首个稳定发布做准备的其他次要 bug 修复

### 2006-01-24（3.3.2 beta）

1.  bug 修复和速度改进。改进的测试覆盖率。

1.  OS 层面接口的变更：现在互斥锁必须是递归的。

1.  停止使用线程特定数据处理内存不足异常

### 2006-01-16（3.3.1 alpha）

1.  无数 bug 修复

1.  速度改进

1.  现在数据库连接可以被多个线程使用，而不仅限于它们创建时所在的线程。

### 2006-01-11（3.3.0 alpha）

1.  CHECK 约束

1.  在 CREATE/DROP TABLE/INDEX 上的 IF EXISTS 和 IF NOT EXISTS 子句。

1.  DESC 索引

1.  更有效地编码布尔值，从而使数据库文件更小

1.  更积极地使用 SQLITE_OMIT_FLOATING_POINT

1.  分离 INTEGER 和 REAL 的亲和性

1.  为 OS 接口增加了一个虚拟函数层

1.  在 TCL 接口中添加了 "exists" 方法

1.  改进了对内存不足错误的响应

1.  数据库缓存可以选择在同一线程的连接之间共享

1.  可选的 READ UNCOMMITTED 隔离级别（而非默认的 SERIALIZABLE 隔离级别）以及当数据库连接共享一个公共缓存时的表级锁定。

### 2005-12-19（3.2.8）

1.  修复一个隐晦的 bug，可能在以下不寻常的情况下导致数据库损坏：一个大的 INSERT 或 UPDATE 语句作为更大事务的一部分失败，因为唯一性约束，但包含的事务提交了。

### 2005-12-19（2.8.17）

1.  修复一个隐晦的 bug，可能在以下不寻常的情况下导致数据库损坏：一个大的 INSERT 或 UPDATE 语句作为更大事务的一部分失败，因为唯一性约束，但包含的事务提交了。

### 2005-09-24（3.2.7）

1.  GROUP BY 现在再次将 NULL 视为相等，正如应该的那样

1.  现在在 Solaris、OpenBSD 和其他不支持 fdatasync() 函数的 Unix 变体上编译

1.  现在再次在 MSVC++6 上编译

1.  修复未初始化变量导致各种复杂查询故障

1.  正确计算仅在左表上约束的 LEFT OUTER JOIN

### 2005-09-17 (3.2.6)

1.  修复在数据库大于 1GiB 的情况下，如果 VACUUM（或自动 VACUUM）失败并回滚可能导致数据库损坏的 bug

1.  LIKE 优化现在对 COLLATE NOCASE 的列起作用

1.  ORDER BY 和 GROUP BY 现在使用有界内存

1.  增加对 COUNT(DISTINCT expr) 的支持

1.  更改 SUM() 处理 NULL 值的方式，以符合 SQL 标准

1.  在可能的情况下使用 fdatasync() 替代 fsync()，以加快提交速度

1.  在联接中使用 CROSS 关键字会关闭表重新排序优化

1.  增加了实验性且未记录的 EXPLAIN QUERY PLAN 功能

1.  在 Windows 中使用 Unicode API

### 2005-08-27 (3.2.5)

1.  修复 DELETE 和 UPDATE 语句影响超过 40960 行的 bug。

1.  更改 makefile，不再需要 GNUmake 扩展

1.  修复 configure 脚本上 --enable-threadsafe 选项

1.  修复代码生成器的 bug，当 IN 运算符的左侧是常量且右侧是 SELECT 语句时发生

1.  PRAGMA synchronous=off 语句现在不仅禁用主日志文件的同步，还禁用正常的回滚日志

### 2005-08-24 (3.2.4)

1.  修复在前一个版本中引入的一个 bug，可能在生成复杂 WHERE 子句的代码时导致段错误。

1.  允许浮点文字以小数点开始或结束。

### 2005-08-21 (3.2.3)

1.  增加对 CAST 操作符的支持

1.  Tcl 接口允许将 BLOB 值传递给用户定义的函数

1.  在 Tcl 接口中添加了 "transaction" 方法

1.  允许列的 DEFAULT 值调用具有常数操作数的函数

1.  增加了 ANALYZE 命令，用于收集索引的统计信息，并在选择优化器中使用这些统计信息。

1.  删除了 WHERE 子句中项数限制（以前为 100）。

1.  现在，IN 操作符的右侧可以是表达式列表，而不仅仅是常量列表。

1.  重构了优化器，以便更好地利用索引。

1.  调整联接中表的顺序，自动优化以更好地利用索引。

1.  即使左侧不是索引的最左项，现在 IN 操作符也是优化的候选项。可以在同一索引上使用多个 IN 操作符。

1.  使用 BETWEEN 和 OR 的 WHERE 子句表达式现在可以进行优化。

1.  增加了 "case_sensitive_like" pragma 和 SQLITE_CASE_SENSITIVE_LIKE 编译时选项，将其默认值设置为 "on"。

1.  当启用 case_sensitive_like pragma 时，使用索引来辅助 GLOB 表达式和 LIKE 表达式。

1.  增加了与 MySQL 兼容的重音引号引用支持。

1.  提升了测试覆盖率。

1.  数十个小 bug 修复。

### 2005-06-12 (3.2.2)

1.  增加了 sqlite3_db_handle() API。

1.  增加了 sqlite3_get_autocommit() API。

1.  向解析器添加了 REGEXP 操作符。标准构建中没有支持此操作符的功能，但用户可以使用 sqlite3_create_function() 自行添加。

1.  提升速度并减小库的占用空间。

1.  解决了 64 位架构上的字节对齐问题。

1.  多处小 bug 修复和文档更新。

### 2005-03-29 (3.2.1)

1.  修复了新 ADD COLUMN 注释中的内存分配错误。

1.  文档更新。

### 2005-03-21 (3.2.0)

1.  增加对 ALTER TABLE ADD COLUMN 的支持。

1.  增加对 ISO-8601 日期/时间字符串中 "T" 分隔符的支持。

1.  改进对 Cygwin 的支持。

1.  大量 bug 修复和文档更新。

### 2005-03-17 (3.1.6)

1.  修复了向具有约 125 列的表插入记录时可能导致数据库损坏的 bug。

1.  sqlite3_step() 现在更有可能调用繁忙处理程序，并且更不太可能返回 SQLITE_BUSY。

1.  修复了以前在 malloc() 失败后可能发生的内存泄漏。

### 2005-03-11 (3.1.5)

1.  在 Mac OS X 上用于控制同步到磁盘的 ioctl 是 F_FULLFSYNC，而不是 F_FULLSYNC。上一个版本出错了。

### 2005-03-11 (3.1.4)

1.  修复了 autovacuum 中的一个 bug，如果由于约束违反而导致 CREATE UNIQUE INDEX 失败，可能会导致数据库损坏。此问题仅在打开了版本 3.1 中引入的新 autovacuum 功能时才会出现。

1.  如果 synchronous pragma 设置为非 "full"，则禁用 F_FULLSYNC ioctl（目前仅在 Mac OS X 上支持）。

1.  向未来的 3.2 版本数据库文件格式增加了额外的前向兼容性。

1.  修复了 WHERE 子句中 (rowid<'2') 形式的一个 bug

1.  新增了 SQLITE_OMIT_... 编译时选项

1.  更新了手册页面

1.  删除了 shell 中使用 strcasecmp() 的用法

1.  Windows DLL 导出符号 Tclsqlite_Init 和 Sqlite_Init

### 2005-02-19 (3.1.3)

1.  修复了在删除包含 AUTOINCREMENT 表的表后对数据库进行 VACUUM 的问题。

1.  向未来的 3.2 版本数据库文件格式增加了前向兼容性。

1.  文档更新

### 2005-02-15 (3.1.2)

1.  修复一个 bug，如果同一个数据库有两个打开的连接，一个连接执行了 VACUUM，另一个对数据库进行了一些更改，可能导致数据库损坏。

1.  允许在 LIMIT 子句中使用 "?" 参数。

1.  修复 VACUUM，使其与 AUTOINCREMENT 一起工作。

1.  修复了 AUTOVACUUM 中的竞争条件，可能导致数据库损坏。

1.  为 sqlite3.h 包含文件添加了数值版本号。

1.  其他次要 bug 修复和性能优化。

### 2005-02-15 (2.8.16)

1.  修复一个 bug，如果同一个数据库有两个打开的连接，一个连接执行了 VACUUM，另一个对数据库进行了一些更改，可能导致数据库损坏。

1.  正确处理在 CREATE INDEX 语句中的带引号名称。

1.  修正了 sqlite.h 和 sqlite3.h 之间的命名冲突。

1.  复制表达式时避免过多的堆使用。

1.  其他次要错误修复。

### 2005-02-01 (3.1.1 BETA)

1.  在 TCL 接口中自动缓存准备好的语句。

1.  ATTACH 和 DETACH 以及其他一些操作会导致现有的预处理语句过期。

1.  大量次要错误修复。

### 2005-01-21 (3.1.0 ALPHA)

1.  添加了自动清理支持。

1.  添加了 CURRENT_TIME、CURRENT_DATE 和 CURRENT_TIMESTAMP。

1.  添加了 EXISTS 子句的支持。

1.  添加了对相关子查询的支持。

1.  在 LIKE 操作符上添加了 ESCAPE 子句。

1.  添加了对 ALTER TABLE ... RENAME TABLE ... 的支持。

1.  在 INTEGER PRIMARY KEY 上支持 AUTOINCREMENT 关键字。

1.  许多 SQLITE_OMIT_ 宏插入以在编译时省略功能并减少库占用空间。

1.  添加了 REINDEX 命令。

1.  引擎如果能从索引获取所需的所有信息，则不再查询主表。

1.  许多讨厌的错误已修复。

### 2004-10-12 (3.0.8)

1.  添加了对 DEFERRED、IMMEDIATE 和 EXCLUSIVE 事务的支持。

1.  允许在已有一个或多个预编译 SQL 语句的情况下创建新的用户定义函数。

1.  为 MinGW/MSYS 解决了可移植性问题。

1.  修复了 64 位 Sparc 机器上的字节对齐问题。

1.  修复了 shell 的 ".import" 命令，使其忽略行尾的 \r 字符。

1.  shell 中的 "csv" 模式选项将字符串放在双引号内。

1.  修正了文档中的拼写错误。

1.  将代码中的数组常量转换为类型为 "const"。

1.  大量代码优化，特别是旨在减小代码占用空间的优化。

### 2004-09-18 (3.0.7)

1.  BTree 模块使用 malloc() 分配大缓冲区，而不是从堆栈中分配，以在堆栈空间有限的机器上运行更顺畅。

1.  解决了命名冲突，使得版本 2.8 和 3.0 可以在同一个 ANSI-C 源文件中链接和使用。

1.  新接口：sqlite3_bind_parameter_index()

1.  添加了对形如 "?nnn" 的通配符参数的支持。

1.  修复了在 64 位系统上发现的问题。

1.  从版本 3.0 中删除了 encode.c 文件（包含未使用的例程）。

1.  sqlite3_trace() 回调在每个语句执行之前发生，而不是在编译语句时发生。

1.  Makefile 更新和杂项错误修复。

### 2004-09-02（3.0.6 测试版）

1.  更好地检测和处理损坏的数据库文件。

1.  sqlite3_step() 接口在由于锁定无法提交更改时返回 SQLITE_BUSY。

1.  将 LIKE 和 GLOB 的实现合并为单一的模式匹配子程序。

1.  杂项代码大小优化和错误修复。

### 2004-08-29（3.0.5 测试版）

1.  支持 ":AAA" 样式的绑定参数名。

1.  添加新的 sqlite3_bind_parameter_name() 接口。

1.  支持在 TCL 绑定的 SQL 语句中嵌入 TCL 变量名。

1.  TCL 绑定在传输数据时不一定需要进行字符串转换。

1.  TEMP 表的数据库直到需要时才创建。

1.  添加了使用 "sqlite_temp_directory" 全局变量指定替代临时文件目录的功能。

1.  编译时选项（SQLITE_BUSY_RESERVED_LOCK）在争夺 RESERVED 锁时调用繁忙处理程序。

1.  各种错误修复和优化。

### 2004-08-09（3.0.4 测试版）

1.  CREATE TABLE 和 DROP TABLE 现在可以正确作为预处理语句执行。

1.  修复 VACUUM 和 UNIQUE 索引中的错误。

1.  在命令行 shell 中添加 ".import" 命令。

1.  修复可能导致索引损坏的 bug，当尝试删除表的行被挂起查询阻止时。

1.  库大小优化。

1.  其他小错误修复。

### 2004-07-22（2.8.15）

1.  这只是一个维护版本。修复了各种小错误并增强了可移植性。

### 2004-07-22（3.0.3 测试版）

1.  SQLite 3.0 的第二个测试版发布。

1.  添加支持 "PRAGMA page_size" 来调整数据库的页面大小。

1.  各种错误修复和文档更新。

### 2004-06-30（3.0.2 测试版）

1.  SQLite 3.0 的第一个测试版发布。

### 2004-06-22（3.0.1 alpha）

1.  ***** Alpha Release - 仅供研究和测试使用 *****

1.  大量 bug 修复。

### 2004-06-18 (3.0.0 alpha)

1.  ***** Alpha Release - 仅供研究和测试使用 *****

1.  支持包括 UTF-8、UTF-16 和用户定义排序序列在内的国际化。

1.  新的文件格式，对于典型用途大小缩小 25% 到 35%。

1.  改进并发性能。

1.  ATTACHed 数据库的原子提交。

1.  清理 APIs 中的无用信息。

1.  支持 BLOB。

1.  支持 64 位 rowid。

1.  更多信息。

### 2004-06-09 (2.8.14)

1.  修复 min() 和 max() 优化器，使其在 FROM 子句为子查询时正常工作。

1.  忽略 shell 中 "." 命令末尾的额外空白。

1.  将 sqlite_encode_binary() 和 sqlite_decode_binary() 与库捆绑。

1.  TEMP_STORE 和 DEFAULT_TEMP_STORE pragma 现在起作用。

1.  修改代码以在 OpenWatcom 中编译干净。

1.  修复 INSTEAD OF 触发器和 IN 操作符中的 NULL 导致的 VDBE 栈溢出问题。

1.  添加全局变量 sqlite_temp_directory，若设置，则定义临时文件存储目录。

1.  sqlite_interrupt() 与 VACUUM 协同工作。

1.  其他次要 bug 修复。

### 2004-03-08 (2.8.13)

1.  重构部分代码，以减小代码占用空间。现在代码也稍微更快。

1.  sqlite_exec() 现在作为 sqlite_compile() 和 sqlite_step() 的包装器实现。

1.  内置 min() 和 max() 函数现在遵守 NUMERIC 和 TEXT 数据类型之间的差异。以前，min() 和 max() 总是假定其参数为 NUMERIC 类型。

1.  内置日期/时间函数新增 HH:MM:SS 修改器。

1.  添加实验性的 sqlite_last_statement_changes() API。修复 last_insert_rowid() 函数，以便在触发器中正确工作。

1.  为数据库加密 API 添加函数原型。

1.  修复多个小问题。

### 2004-02-08 (2.8.12)

1.  修复了一个 bug，如果在 COMMIT 操作中断的中间发生了电源故障或外部程序停止，可能会损坏回滚日志。损坏的日志在回滚时可能导致数据库损坏。

1.  减小了各种模块的大小并增加了速度，特别是虚拟机。

1.  允许"<expr> IN <table>"作为"<expr> IN (SELECT * FROM <table>)"的简写。

1.  优化了 sqlite_mprintf()例程。

1.  确保 MIN()和 MAX()优化在子查询中正常工作。

### 2004-01-14 (2.8.11)

1.  修复了 IN 操作符在子查询中处理 NULL 的 bug。该 bug 是上一个版本引入的。

### 2004-01-14 (2.8.10)

1.  修复了 Unix 平台上潜在的数据库损坏问题，原因是每次 close()文件时，所有 POSIX 咨询锁都会被清除。解决方案是在锁定时禁止所有 close()调用。

1.  在某些 COUNT(*)的边缘情况下进行了性能增强。

1.  确保内存后端在 malloc()失败时能够合理响应。

1.  允许从用户定义的 SQL 函数内部调用 sqlite_exec()。

1.  使用"long double"提高了浮点数转换的精度。

1.  在实验性日期/时间函数中修复了 bug。

### 2004-01-06 (2.8.9)

1.  修复了 32 位整数溢出问题，如果将大的负数（小于-2147483648）插入索引的数值列中，可能导致数据库索引损坏。

1.  修复了多线程 Linux 实现中的锁定问题。

1.  即使区域设置请求","，也始终使用"."作为小数点。

1.  向实验性日期/时间函数添加了 UTC 到本地时间的转换。

1.  日期/时间函数的 bug 修复。

### 2003-12-18 (2.8.8)

1.  修复了在 2.8.0 中引入的可能导致数据库损坏的关键 bug。

1.  修复了不使用索引的 3 路连接的问题。

1.  VACUUM 命令现在支持非回调 API。

1.  改进了"PRAGMA integrity_check"命令的准确性。

### 2003-12-04 (2.8.7)

1.  添加了实验性的 sqlite_bind()和 sqlite_reset() API。

1.  如果数据库名称为空字符串，则打开一个新的数据库文件，该文件在关闭数据库时自动删除。

1.  在由 Lemon 生成的解析器中进行了性能增强。

1.  修订了实验性的日期/时间函数。

1.  不允许在永久表上使用临时索引。

1.  文档更新和错字修正。

1.  添加了实验性的 sqlite_progress_handler() 回调 API。

1.  移除对 Oracle8 外连接语法的支持。

1.  允许 GLOB 和 LIKE 操作符作为函数工作。

1.  其他次要的文档和 makefile 更改以及错误修复。

### 2003-08-22 (2.8.6)

1.  将 CVS 仓库迁移到 www.sqlite.org。

1.  更新了 NULL 处理文档。

1.  添加了实验性的日期/时间函数。

1.  错误修复：正确评估视图中的视图而不会段错误。

1.  错误修复：防止如果删除与表同名的触发器可能会导致数据库损坏。

1.  错误修复：在设置 EMPTY_RESULT_CALLBACKS pragma 后，允许在空数据库上执行 VACUUM 而不会段错误。

1.  错误修复：如果整数值不能适应 32 位整数，则将其存储为双精度浮点数。

1.  错误修复：确保在写入数据库文件之前将日志文件目录条目提交到磁盘。

### 2003-07-22 (2.8.5)

1.  使 LIMIT 在复合 SELECT 语句中工作。

1.  LIMIT 0 现在显示没有行。使用 LIMIT -1 来查看所有行。

1.  正确处理 INTEGER PRIMARY KEY 和浮点数之间的比较。

1.  修复了新的 ATTACH 和 DETACH 命令中的几个重要错误。

1.  更新了 NULL 处理文档。

1.  允许在 sqlite_compile() 和 sqlite_step() 中使用 NULL 参数。

1.  许多次要的错误修复。

### 2003-06-29 (2.8.4)

1.  增强了 "PRAGMA integrity_check" 命令以验证索引。

1.  为新的 ATTACH 和 DETACH 命令添加了授权钩子。

1.  许多文档更新。

1.  许多次要的错误修复。

### 2003-06-04 (2.8.3)

1.  修复一个问题，如果在包含 INTEGER PRIMARY KEY 和一个或多个索引的表上执行 INSERT OR REPLACE 或 UPDATE OR REPLACE，可能会损坏表上的索引。

1.  修复了 Windows 锁定代码中的一个错误，使得 Win95 和 WinNT 系统同时访问时锁定正常工作。

1.  使 INSERT 和 UPDATE 语句能够引用"rowid"（或"_rowid_"或"oid"）列的功能已添加。

1.  其他重要的 bug 修复。

### 2003-05-17（2.8.2）

1.  修复了一个问题，如果从主数据库中删除具有 TEMP 索引的表，则可能损坏数据库文件。

### 2003-05-17（2.8.1）

1.  重新激活了 VACUUM 命令，该命令可以回收数据库文件中未使用的磁盘空间。

1.  增加了 ATTACH 和 DETACH 命令，允许同时与多个数据库文件交互。

1.  增加了对 TEMP 触发器和索引的支持。

1.  增加了对内存数据库的支持。

1.  移除了实验性的 sqlite_open_aux_file()。其功能被新的 ATTACH 命令所取代。

1.  更改了 ON CONFLICT 子句的优先顺序，使得 BEGIN 语句上的 ON CONFLICT 子句比约束上的 ON CONFLICT 子句具有更高的优先级。

1.  许多许多 bug 修复和兼容性增强。

### 2003-02-16（2.8.0）

1.  修改了日志文件格式，使其在操作系统崩溃或停电后更能抵抗损坏。

1.  添加了一个新的不使用回调返回数据的 C/C++ API。

### 2003-01-25（2.7.6）

1.  性能改进。现在库的速度更快了。

1.  添加了**sqlite_set_authorizer()** API。尚未撰写正式文档 - 请参阅源代码注释以了解如何使用此函数。

1.  修复了 GLOB 运算符中的一个错误，该错误导致其无法与大写字母一起工作。

1.  各种次要 bug 修复。

### 2002-12-28（2.7.5）

1.  修复了 pager.c 中未初始化的变量，可能导致数据库损坏的概率约为 40 亿分之一。

### 2002-12-17（2.7.4）

1.  数据库文件现在可以增长到 2⁴¹ 字节。旧的限制为 2³¹ 字节。

1.  优化器现在将按相反的顺序扫描表，以满足 ORDER BY ... DESC 子句的要求时。

1.  现在即使在 sqlite_open()中传入相对路径，也会记住数据库文件的完整路径。这允许库在 chdir()后继续正确操作。

1.  VDBE 中的速度改进。

1.  大量小错误修复。

### 2002-10-31（2.7.3）

1.  各种编译器兼容性修复。

1.  修复"expr IN ()"操作符中的一个错误。

1.  接受括号中的列名。

1.  修复 VDBE 中字符串内存管理的问题。

1.  修复"table_info" pragma 中的一个错误。

1.  在 Windows DLL 中导出 sqlite_function_type() API 函数。

1.  修复在 Windows 下的锁定行为。

1.  修复 LEFT OUTER JOIN 中的一个错误。

### 2002-09-25（2.7.2）

1.  防止巨大事务中的日志文件溢出。

1.  修复 sqlite_open()失败时发生的内存泄漏。

1.  即使结果集用于 INSERT，也要遵循 SELECT 的 ORDER BY 和 LIMIT 子句。

1.  不要在用于保存 TEMP 表的文件上放写锁。

1.  添加关于 SELECT DISTINCT 和 SQLite 处理 NULL 的文档。

1.  修复单个 sqlite_exec()调用执行数千条 SQL 语句时性能下降的问题。

### 2002-08-31（2.7.1）

1.  修复版本 2.7.0 中引入的 ORDER BY 逻辑中的一个错误。

1.  现在分词器接受 C 风格的注释。

1.  当源是 SELECT 语句时，INSERT 运行速度稍快。

### 2002-08-25（2.7.0）

1.  在排序时区分数字和文本值。文本值按 memcmp()排序。数字值按数值顺序排序。

1.  通过模拟缺失于 Win95/98/ME 的读者/写者锁，在 Windows 下允许多个同时读取者。

1.  如果另一个事务已经激活，则尝试启动事务时现在会返回错误。

### 2002-08-13（2.6.3）

1.  添加读取小端和大端数据库的功能。因此，可以在 Linux 或 Windows 下读取和写入在 SunOS 或 Mac OS X 下创建的数据库，反之亦然。

1.  转移到新网站：https://www.sqlite.org/

1.  允许事务跨 Linux 线程。

1.  在 GROUP BY 查询的 ORDER BY 子句处理中修复了一个错误。

### 2002-07-31 (2.6.2)

1.  COPY 命令读取的文本文件现在可以具有 LF、CRLF 或 CR 作为行终止符。

1.  在数据库初始化期间遇到 SQLITE_BUSY 时进行正确处理。

1.  修复在临时表上的 UPDATE 触发器。

1.  更新了文档。

### 2002-07-19 (2.6.1)

1.  在库中包含一个静态字符串，该字符串响应 RCS 的"ident"命令，并包含库的版本号。

1.  修复了在打开"count_changes"开关时删除表的所有行时发生的断言失败。

1.  在自动从 2.5.6 升级到 2.6.0 数据库格式期间出现问题时进行更好的错误报告。

### 2002-07-18 (2.6.0)

1.  更改索引格式以纠正从版本 2.1.0 开始的设计缺陷。*** 这是一种不兼容的文件格式更改 *** 当库的 2.6.0 或更高版本试图打开由 2.5.6 或更早版本创建的数据库文件时，它将自动且不可逆地转换文件格式。**在使用库的版本 2.6.0 打开旧数据库文件之前，请先备份旧的数据库文件。**

### 2002-07-07 (2.5.6)

1.  修复回滚中的更多问题。增强测试套件，以大量测试回滚逻辑，以防止未来任何问题。

### 2002-07-06 (2.5.5)

1.  修复了在回滚期间可能导致数据库损坏的 bug。这个 bug 是在版本 2.4.0 中由检查[410]的 freelist 优化引入的。

1.  修复了 VIEW 的聚合函数中的一个 bug。

1.  其他较小的更改和增强。

### 2002-07-01 (2.5.4)

1.  再次使"AS"关键字可选。

1.  现在列的数据类型出现在回调的第四个参数中。

1.  添加了**sqlite_open_aux_file()** API，尽管它仍然大部分未记录和未经测试。

1.  添加了额外的测试用例，并修复了这些测试用例发现的一些错误。

### 2002-06-25 (2.5.3)

1.  Bug 修复：由于版本 2.4.0 中引入的优化（check-in [410]），可能导致数据库损坏。现在该问题应该已经修复。不建议使用版本 2.4.0 至 2.5.2。

### 2002-06-25 (2.5.2)

1.  新增了**SQLITE_TEMP_MASTER**表，用于记录临时表的模式，方式类似于**SQLITE_MASTER**对于持久表的记录。

1.  对 UNION ALL 进行了优化。

1.  修复了 LEFT OUTER JOIN 处理中的一个 bug。

1.  现在 LIMIT 子句可以在子查询中工作。

1.  ORDER BY 在子查询中可以工作。

1.  新增了 TypeOf()函数，用于确定表达式是数值还是文本。

1.  自增现在对从 SELECT 进行的 INSERT 有效。

### 2002-06-19 (2.5.1)

1.  查询优化器现在尝试使用索引实现 ORDER BY 子句。如果适当的索引不可用，则仍然使用排序。

### 2002-06-17 (2.5.0)

1.  增加了对行触发器的支持。

1.  增加了对 NULL 的 SQL-92 兼容处理。

1.  增加了对完整 SQL-92 连接语法和 LEFT OUTER JOIN 的支持。

1.  双引号括起的字符串解释为列名而不是文本字面量。

1.  解析（但不实现）外键。

1.  解析器、分页器和 WHERE 子句代码生成器中的性能改进。

1.  使得在子查询中 LIMIT 子句可以工作。（虽然 ORDER BY 仍然不工作。）

1.  在 sqlite_*_printf()中添加了"%Q"扩展。

1.  太多要提及的 Bug 修复（请查看变更日志）。

### 2002-05-10 (2.4.12)

1.  添加了逻辑以检测库 API 例程被无序调用的情况。

### 2002-05-08 (2.4.11)

1.  Bug 修复：某些（相当复杂的）视图的结果集中列名生成不正确。这可能在某些情况下导致段错误。

### 2002-05-03 (2.4.10)

1.  Bug 修复：当复合 SELECT 用作子查询时生成正确的列标题。

1.  将 sqlite_encode_binary()和 sqlite_decode_binary()函数添加到源树中，但尚未链接到库中。

1.  更新了文档。

1.  从 Windows DLLs 导出 sqlite_changes()函数。

1.  修复错误：在缺少 FROM 子句的查询中不要尝试子查询扁平化优化。这样做会导致段错误。

### 2002-04-22 (2.4.9)

1.  修复导致 Windows 98 下预编译二进制文件 SQLITE.EXE 报告"内存不足"的错误。

### 2002-04-20 (2.4.8)

1.  确保在 shell 中的**.dump**命令输出中，在对应的 TABLE 之后创建 VIEW。

1.  速度提升：不对 TEMP 表执行同步更新。

1.  对 shell 进行了许多改进和增强。

1.  使 GLOB 和 LIKE 操作符成为可以被程序员覆盖的函数。这允许例如将 LIKE 操作符更改为区分大小写。

### 2002-04-12 (2.4.7)

1.  允许在 SELECT 语句的列列表中使用 TABLE.*。

1.  允许没有 FROM 子句的 SELECT 语句。

1.  添加了**last_insert_rowid()** SQL 函数。

1.  不计算发生 IGNORE 冲突解决的行数。

1.  确保 INSERT 的 VALUES 子句中函数表达式正确。

1.  添加了**sqlite_changes()** API 函数，以返回最近操作中更改的行数。

### 2002-04-02 (2.4.6)

1.  修复错误：正确处理 JOIN 的 WHERE 子句中不包含比较运算符的项。

### 2002-04-02 (2.4.5)

1.  修复错误：正确处理出现在 JOIN 的 WHERE 子句中的函数。

1.  当设置 PRAGMA vdbe_trace=ON 时，当 P3 操作数值为指向结构体的指针而不是字符串指针时，正确打印其操作数值。

1.  当将显式 NULL 插入 INTEGER PRIMARY KEY 时，自动将 NULL 值转换为唯一键。

### 2002-03-30 (2.4.4)

1.  允许"VIEW"作为列名。

1.  添加了对 CASE 表达式的支持（Dan Kennedy 的补丁）。

1.  在交付中添加了 RPMS（Doug Henry 的补丁）。

1.  修正文档中的拼写错误。

1.  切换到一个带有自己 CVSTrac bug 跟踪系统的新 CVS 存储库的配置管理。

### 2002-03-23 (2.4.3)

1.  修复了在 SELECT 中使用复合 SELECT 作为子查询时发生的 bug。

1.  **sqlite_get_table()** 函数现在在给定返回不同列数的两个或更多个 SELECT 的情况下返回错误。

### 2002-03-20 (2.4.2)

1.  修复 Bug：修复了在视图的 SELECT 语句中，当 ROWID 是一个列时导致断言失败的问题。

1.  修复 Bug：修复了在 VDBE 中未初始化的变量可能导致断言失败的问题。

1.  修改 os.h 头文件，使其更可靠地检测编译是为 Windows 还是 Unix。

### 2002-03-13 (2.4.1)

1.  在 FROM 子句中使用未命名子查询会导致段错误。

1.  解析器现在要求在执行语句之前看到分号或输入结束。这样做可以避免在 UPDATE 或 DELETE 语句中错误拼写 WHERE 关键字时造成意外灾难。

### 2002-03-11 (2.4.0)

1.  将 sanity_check PRAGMA 的名称更改为 **integrity_check**，并使其在所有编译中可用。

1.  处理带有索引列的 min() 或 max()，且没有 WHERE 或 GROUP BY 子句的情况，作为一种特殊情况，避免完全表扫描。

1.  自动生成的 ROWID 现在是顺序的。

1.  不允许命令行 shell 的点命令出现在真实 SQL 命令中间。

1.  修改了 Lemon parser generator 的内容，使得解析器表格缩小了 4 倍。

1.  添加了对用 C 实现的用户定义函数的支持。

1.  添加了对新函数 **coalesce()**、**lower()**、**upper()** 和 **random()** 的支持。

1.  添加了对 VIEW 的支持。

1.  添加了子查询展开优化器。

1.  修改 B-Tree 和 Pager 模块，使不包含实际数据（空闲页）的磁盘页面不被日志记录，并且在内存中更改时不会写回磁盘。这不会影响数据库完整性，因为这些页面不包含实际数据，但确实使大型 INSERT 操作快了大约 2.5 倍，大型 DELETE 操作快了大约 5 倍。

1.  使 CACHE_SIZE pragma 持久化。

1.  添加了 SYNCHRONOUS pragma。

1.  修复了在数据库包含临时表时事务内更新失败的错误。

### 2002-02-19 (2.3.3)

1.  允许使用方括号引用标识符，以与 MS-Access 兼容。

1.  在 SELECT 的 FROM 子句中添加了对子查询的支持。

1.  在 Windows 下更高效地实现了 sqliteFileExists()。（由 Joel Luscy 完成）

1.  INSERT 的 VALUES 子句现在可以包含表达式，包括标量 SELECT 子句。

1.  添加了对 CREATE TABLE AS SELECT 的支持。

1.  Bug 修复：在单个事务内创建和删除表时无法正常工作。

### 2002-02-14 (2.3.2)

1.  Bug 修复：在 pager.c 中存在一个错误的 assert()。实际代码都是正确的（据所知），所以如果使用-DNDEBUG=1 编译，一切应该正常工作。当未禁用 assert 时，可能会出现故障。

### 2002-02-13 (2.3.1)

1.  Bug 修复：如果设置了"PRAGMA full_column_names=ON;"并且进行了使用 rowid 的查询（如："SELECT rowid, * FROM ..."），则断言失败。

### 2002-02-03 (2.3.0)

1.  修复了 INSERT 命令中的一个严重错误，如果数据源是 SELECT，并且 INSERT 子句指定了除默认顺序外的某些顺序的列，则数据会进入错误的列。

1.  添加了解决冲突约束的能力，除了中止和回滚。详见关于"ON CONFLICT"子句的文档。

1.  当关闭时，临时文件现在会被操作系统自动删除。程序崩溃时不会再有悬挂的临时文件。（如果操作系统崩溃，Unix 下重启后 fsck 会删除文件。我不知道在 Windows 下会发生什么。）

1.  现在支持 NOT NULL 约束。

1.  在 COPY 命令中，将 NULL 放入数据为'\N'的列。

1.  在 COPY 命令中，现在可以使用反斜杠来转义换行符。

1.  添加了 SANITY_CHECK pragma。

### 2002-01-28 (2.2.5)

1.  重要 Bug 修复：如果 IN 运算符的左操作数或右操作数源自 INTEGER PRIMARY KEY，则无法正常工作。

1.  在 sqlite 命令行访问程序的输出中，不要转义反斜杠'\'字符。

### 2002-01-22 (2.2.4)

1.  在 SELECT 语句的列列表中，AS 右侧的标签现在可以作为 WHERE、ORDER BY、GROUP BY 和/或 HAVING 子句中表达式的一部分使用。

1.  修复了 sqlite 命令的-separator 命令行选项中的一个 bug。

1.  修复了比较大写字符串与大于'Z'但小于'a'的字符时排序顺序的问题。

1.  如果 ORDER BY 或 GROUP BY 表达式是常量，则报告错误。

### 2002-01-16 (2.2.3)

1.  修复了 VC++ 7.0 中的警告消息。（来自 nicolas352001 的补丁）

1.  使库支持线程安全。（代码已存在且似乎可用，但尚未经过严格测试。）

1.  添加了新的**sqlite_last_insert_rowid()** API 函数。

### 2002-01-14 (2.2.2)

1.  Bug 修复：当临时表带有索引并且与由另一个进程创建的永久表同名时，断言失败。

1.  Bug 修复：包含 INTEGER PRIMARY KEY 和索引的表的更新可能会失败。

### 2002-01-09 (2.2.1)

1.  Bug 修复：在 WHERE 子句为"ROWID=x"且不存在此 rowid 的情况下，尝试删除表的单行时导致错误。

1.  Bug 修复：将 NULL 作为 sqlite_open()的第三个参数传入有时会导致核心转储。

1.  Bug 修复：在单个事务内连续执行 DROP TABLE 和使用相同名称创建 TABLE 会导致核心转储。

1.  来自 A. Rottmann 的 Makefile 更新

### 2001-12-22 (2.2.0)

1.  列类型为 INTEGER PRIMARY KEY 的列实际上在表的底层 B-Tree 表示中被用作主键。

1.  在实现上述整数主键更改时，发现并修复了几个晦涩的无关 Bug。

1.  添加了能够在 SELECT 语句结果部分的较大列列表中放置 "*" 的能力。例如：<nobr>“**SELECT rowid, * FROM table1;**”</nobr>。

1.  更新了注释和文档。

### 2001-12-15 (2.1.7)

1.  修复了**CREATE TEMPORARY TABLE**中的一个 bug，导致表最初在主数据库文件中分配，而不是在单独的临时文件中。这个 bug 可能会导致库出现断言失败，并且可能会在主数据库文件中导致“页面泄漏”。

1.  修复了 b-tree 子系统中的一个 bug，有时会在数据库扫描期间导致表的第一行重复。

### 2001-12-14 (2.1.6)

1.  再次修复锁定机制，以防止 **sqlite_exec()** 不必要地返回 SQLITE_PROTOCOL。这次 bug 是锁定代码中的竞争条件。这个变更影响 POSIX 和 Windows 用户。

### 2001-12-06 (2.1.5)

1.  修复了另一个问题的 bug（与 2.1.4 中修复的问题无关），有时会导致 **sqlite_exec()** 不必要地返回 SQLITE_PROTOCOL。这次 bug 在 POSIX 锁定代码中，不会影响 Windows 用户。

### 2001-12-05 (2.1.4)

1.  有时 **sqlite_exec()** 在应该返回 SQLITE_BUSY 时会返回 SQLITE_PROTOCOL。

1.  修复前一个 bug 的修复揭示了一个死锁，也已修复。

1.  在 sqlite shell 的第二个参数中添加了放置单个 .command 的能力。

1.  更新了常见问题解答（FAQ）。

### 2001-11-24 (2.1.3)

1.  修复比较操作符（例如：“**<**”，“**==**”等）的行为，使其与索引中条目的顺序一致。

1.  正确处理 SQL 表达式中比机器整数表示更大的整数。

### 2001-11-23 (2.1.2)

1.  改动以支持 64 位架构。

1.  修复了锁定协议中的一个 bug。

1.  修复了一个 bug，这个 bug 可能在 DROP TABLE 后导致数据库变得不可读，因为会损坏 SQLITE_MASTER 表。

1.  更改代码，以便此版本的库可以读取由上述 bug 导致的 2.1.1 版本数据库即使 SQLITE_MASTER 表（轻微）损坏。

### 2001-11-13 (2.1.1)

1.  Bug 修复：有时当列的实际值为 NULL 时，会将任意字符串传递给回调函数。

### 2001-11-12 (2.1.0)

1.  更改数据记录的格式，以便可以存储高达 16MB 大小的记录。

1.  更改索引的格式，以实现更好的查询优化。

1.  在 SELECT 语句中实现"LIMIT ... OFFSET ..."子句。

### 2001-11-03 (2.0.8)

1.  在 API 函数中选择性地使用**const**。这应该是完全向后兼容的。

1.  更新文档

1.  通过限制排序器和列表的数量，简化 VDBE 的设计为 1。实际上，无论如何都只会使用一个排序器和一个列表。

### 2001-10-22 (2.0.7)

1.  任何 UTF-8 字符或 ISO8859 字符都可以作为标识符的一部分使用。

1.  由 Christian Werner 提供的补丁，以提高 ODBC 兼容性并修复 round()函数中的一个 bug。

1.  修复某些内存泄漏问题，如果 malloc()失败。只要 malloc()正常工作，我们一直没有内存泄漏问题。

1.  更新一些测试脚本，以便在 Windows 上运行，除了 Unix 之外。

### 2001-10-19 (2.0.6)

1.  添加 EMPTY_RESULT_CALLBACKS pragma

1.  支持在列和表名中使用 UTF-8 和 ISO8859 字符。

1.  Bug 修复：在开启 FULL_COLUMN_NAMES pragma 时计算正确的表名。

### 2001-10-15 (2.0.5)

1.  添加 COUNT_CHANGES pragma。

1.  更改 FULL_COLUMN_NAMES pragma 以帮助 ODBC 驱动程序。

1.  Bug 修复："SELECT count(*)"对空表返回 NULL。现在返回 0。

### 2001-10-13 (2.0.4)

1.  Bug 修复：一个不太明显且相对无害的 bug，在 gcc 优化开启时导致某个测试失败。此版本修复了该问题。

### 2001-10-13 (2.0.3)

1.  Bug 修复：**sqlite_busy_timeout()**函数在失败之前延迟了 1000 倍时间过长。

1.  修复了 bug：如果持有数据库文件的磁盘变满或由于其他原因停止接受写入，则断言将失败。新测试已添加以检测将来类似的问题。

1.  添加了新操作符：**&**（按位与）**|**（按位或）、**~**（按位取反）、**<<**（左移）、**>>**（右移）。

1.  添加了新函数：**round()** 和 **abs()**。

### 2001-10-09 (2.0.2)

1.  修复了锁定协议中的两个 bug。（一个 bug 掩盖了另一个。）

1.  删除了一些未使用的 "#include <unistd.h>"，这些未使用的文件导致 VC++ 出现问题。</unistd.h>

1.  修复了 **sqlite.h**，使其可以从 C++ 中使用。

1.  添加了 FULL_COLUMN_NAMES pragma。当设置为 "ON" 时，列名报告为 TABLE.COLUMN 而不仅仅是 COLUMN。

1.  添加了 TABLE_INFO() 和 INDEX_INFO() pragma 以帮助支持 ODBC 接口。

1.  添加了对 TEMPORARY 表和索引的支持。

### 2001-10-02 (2.0.1)

1.  从 btree.c 中删除了一些 C++ 风格的注释，以便它可以使用除 gcc 以外的编译器进行编译。

1.  如果数据中有嵌入式换行符，则 shell 的 ".dump" 输出无法正常工作。这是一个从版本 1.0 传承下来的旧 bug。为了修复它，".dump" 输出不再使用 COPY 命令，而是生成 INSERT 语句。

1.  扩展表达式语法以支持 "expr NOT NULL"（在 "NOT" 和 "NULL" 之间有一个空格）以及 "expr NOTNULL"（没有空格）。

### 2001-09-28 (2.0.0)

1.  自动构建 Linux 和 Windows 的二进制文件，并将它们放在网站上。

### 2001-09-28 (2.0-alpha-4)

1.  整合了 A. Rottmann 的 makefile 补丁，以使用 LIBTOOL。

### 2001-09-27 (2.0-alpha-3)

1.  SQLite 现在在 CREATE UNIQUE INDEX 中遵守 UNIQUE 关键字。主键必须是唯一的。

1.  文件格式已更改回 alpha-1 版本。

1.  修复了回滚和锁定行为

### 2001-09-20 (2.0-alpha-2)

1.  版本 2.0 的初始发布。将库重命名为 "SQLus" 的想法被放弃，而是保留了 "SQLite" 名称并增加了主版本号。

1.  分页器和 btree 子系统重新添加。它们现在是唯一可用的后端。

1.  移除了 Dbbe 抽象和 GDBM 和内存驱动程序。

1.  所有代码的版权已声明放弃。该库现在属于公共领域。

### 2001-07-23 (1.0.32)

1.  分页器和 btree 子系统已移除。这些将用于后续的 SQL 服务器库，名为"SQLus"。

1.  添加了使用带引号的字符串作为表名和列名表达式的能力。

### 2001-04-15 (1.0.31)

1.  添加了分页子系统，但尚未使用。

1.  更可靠地处理内存不足错误。

1.  测试套件新增了新的测试。

### 2001-04-06 (1.0.30)

1.  移除了在上一版本中引入的**sqlite_encoding** TCL 变量。

1.  向 sqlite TCL 命令添加了**-encoding**和**-tcl-uses-utf**选项。

1.  添加了测试以确保 tclsqlite 是使用与 Tcl 头文件和库匹配的编译的。

### 2001-04-05 (1.0.29)

1.  如果在配置时提供了--enable-utf8 选项，则库现在默认假定数据存储为 UTF-8。默认行为始终假定为 iso8859-x。这仅影响 LIKE 和 GLOB 运算符以及 LENGTH 和 SUBSTR 函数。

1.  如果库未配置为 UTF-8，并且 Tcl 库是使用 UTF-8 的较新版本之一，则在 TCL 接口内部会进行从 UTF-8 到 iso8859 的转换，然后再转回来。

### 2001-04-04 (1.0.28)

1.  添加了对事务的有限支持。目前，事务将在 GDBM 后端上对表进行锁定。目前还不支持回滚或原子提交。

1.  添加了特殊列名 ROWID、OID 和 _ROWID_，它们分别指代每个表的每行的唯一随机整数键。

1.  向回归套件添加了额外的测试，以覆盖新的 ROWID 功能和下文提到的 TCL 接口漏洞。

1.  对 Lemon 解析生成器进行了更改，以便在使用 MSVC 编译时能更好地工作。

1.  在由 Oleg Oleinick 指出的 TCL 接口中修复了漏洞。

### 2001-03-20 (1.0.27)

1.  在执行 DELETE 和 UPDATE 时，库原先会将待删除或更新的记录号写入临时文件中，现在改为将记录号保存在内存中。

1.  没有 WHILE 子句的 DELETE 命令只会从磁盘上移除数据库文件，而不是逐条删除记录。

### 2001-03-20 (1.0.26)

1.  修复了 Windows 上的一个严重 bug。Windows 用户应该升级。对 Unix 没有影响。

### 2001-03-15 (1.0.25)

1.  修改测试脚本，以标识依赖系统负载和处理器速度的测试，并警告用户一个（罕见）测试的失败并不一定意味着库出现故障。代码未更改。

### 2001-03-14 (1.0.24)

1.  修复一个 bug，该 bug 导致在 "malloc(0)" 返回 NULL 的系统上 UPDATE 命令执行失败。这个问题在 Windows、Linux 或 HPUX 上不会出现，但会导致在 QNX 上库无法正常工作。

### 2001-02-20 (1.0.23)

1.  修复了由 Mark Muranwski 发现的一个与 "memory:" 数据库的临时文件放置算法相关的（并且是次要的） bug。

### 2001-02-19 (1.0.22)

1.  先前的修复并不完全正确。这次修复似乎更有效。

### 2001-02-19 (1.0.21)

1.  当 WHERE 子句包含一些可以使用索引满足的条件和其他无法满足的条件时，UPDATE 语句无法正常工作。已修复。

### 2001-02-11 (1.0.20)

1.  将开发变更合并到主干中。未来工作将朝向使用 B 树文件结构，将使用单独的 CVS 源代码树。这个 CVS 树将继续支持 SQLite 的 GDBM 版本。

### 2001-02-06 (1.0.19)

1.  修复一个在 QNX 上引起问题的奇怪（但是有效）的 C 声明。逻辑没有变化。

### 2001-01-04 (1.0.18)

1.  在出现错误时打印有问题的 SQL 语句。

1.  不再要求在 CREATE TABLE 语句的约束之间使用逗号。

1.  向 shell 添加了 "-echo" 选项。

1.  对注释进行了修改。

### 2000-12-10 (1.0.17)

1.  重新编写 **sqlite_complete()** 以提高其速度。

1.  对其他代码进行了一些微小的调整，以使其运行速度更快。

1.  添加了**sqlite_complete()**和内存泄漏的新测试。

### 2000-11-28（1.0.16）

1.  文档更新。主要是修正了拼写错误和拼写错误。

### 2000-10-23（1.0.15）

1.  文档更新

1.  从 vdbe.c 的内部循环中删除了一些健全性检查代码，以帮助库运行速度更快。只有在使用-DNDEBUG 编译时才删除该代码。

### 2000-10-19（1.0.14）

1.  添加了一个"memory:"后端驱动程序，将其数据库存储在内存哈希表中。

### 2000-10-19（1.0.13）

1.  将 GDBM 驱动程序分离到一个单独的文件中，以便预期添加新的驱动程序。

1.  允许数据库名称以驱动程序类型为前缀。目前，唯一的驱动程序类型是"gdbm:"。

### 2000-10-17（1.0.12）

1.  修复了新**sqlite_..._printf()**例程中'%q'格式指令的一个导致核心转储的偏移错误。

1.  增加了**sqlite_interrupt()**接口。

1.  在 shell 中，当用户按下 Control-C 时会调用**sqlite_interrupt()**。

1.  修正了一些**sqlite_exec()**返回错误代码错误的情况。

### 2000-10-11（1.0.10）

1.  添加了如何为 Windows95/98 编译的说明。

1.  删除了一些未使用的变量。等等。

### 2000-10-09（1.0.9）

1.  添加了**sqlite_..._printf()**接口例程。

1.  修改了**sqlite** shell 程序，以使用新的接口例程。

1.  修改了**sqlite** shell 程序，如果显式请求，打印内置 SQLITE_MASTER 表的架构。

### 2000-09-30（1.0.8）

1.  开始撰写有关 TCL 接口的文档。

### 2000-09-29（未发布）

1.  添加了**sqlite_get_table()** API

1.  根据上述更改更新了文档。

1.  修改了**sqlite** shell 程序，以使用新的 sqlite_get_table() API，以便以多列的方式打印表格列表，类似于"ls"打印文件名。

1.  修改了**sqlite** shell 程序，在“.schema”命令的输出中每个 CREATE 语句的末尾打印了一个分号。

### 2000-09-21（未发布）

1.  更改了 tclsqlite 的 "eval" 方法，如果未指定回调脚本，则返回结果列表。

1.  更改 tclsqlite.c 以使用 Tcl_Obj 接口

1.  将 tclsqlite.c 添加到 libsqlite.a 库中

### 2000-09-14 (1.0.5)

1.  将浮点数值的打印格式从 "%g" 改为 "%.15g"。

1.  更改比较函数，使指数表示的数字（例如：1.234e+05）按数值顺序排序。

### 2000-08-28 (1.0.4)

1.  添加了函数 **length()** 和 **substr()**。

1.  修复了 **sqlite** shell 程序的一个 bug，当输出模式为 "column" 且数据的第一行包含 NULL 时，导致核心转储。

### 2000-08-22 (1.0.3)

1.  在 sqlite shell 中，将 "Database opened READ ONLY" 消息打印到 stderr 而不是 stdout。

1.  在 sqlite shell 中，现在在初始启动时打印版本号。

1.  将 **sqlite_version[]** 字符串常量添加到库中

1.  更新 Makefile

1.  Bug 修复：对于包含 WHERE 子句且具有子查询的索引表上的查询，生成的 VDBE 代码不正确。

### 2000-08-18 (1.0.1)

1.  修复了 configure 脚本中的一个 bug。

1.  对网站进行了小幅修订。

### 2000-08-17 (1.0)

1.  更改 **sqlite** 程序，使其能够读取其没有写权限的数据库。（它以前会拒绝所有访问，如果不能写入。）

### 2000-08-09

1.  将回车符视为空白。

### 2000-08-08

1.  在 "sqlite" 命令 shell 的 ".table" 命令中添加了模式匹配。

### 2000-08-04

1.  更新了文档

1.  在 Tcl 接口中添加了 "busy" 和 "timeout" 方法。

### 2000-08-03

1.  文件格式版本号多次存储在 sqlite_master.tcl 中。这虽然无害，但是不必要。现已修复。

### 2000-08-02

1.  索引的文件格式略有更改，以解决在具有大量相同键条目的大型索引中，使用 GDBM 时可能出现的低效率问题。 **不兼容更改**

### 2000-08-01

1.  解析器的堆栈在非常长的 UPDATE 语句上溢出。此问题现已修复。

### 2000-07-31

1.  完成 VDBE 教程。

1.  添加了编译到 WinNT 的文档。

1.  修复 WinNT 的配置程序问题。

1.  修复了 HPUX 的配置问题。

### 2000-07-29

1.  结果的列名标签更清晰。

### 2000-07-28

1.  添加了**sqlite_busy_handler()**和**sqlite_busy_timeout()**接口。

### 2000-06-23

1.  开始编写 VDBE 教程。

### 2000-06-21

1.  清理注释和变量名。对文档进行更改。代码没有功能性变化。

### 2000-06-19

1.  UPDATE 语句中的列名区分大小写。此错误已修复。

### 2000-06-18

1.  添加了字符串连接运算符(||)。

### 2000-06-12

1.  将 fcnt()函数添加到 SQL 解释器中。fcnt()函数返回已发生的数据库“获取”操作次数。此函数设计用于测试脚本，以验证查询是否高效和适当优化。据我所知，fcnt()没有其他有用的目的。

1.  添加了大量利用新 fcnt()函数的测试。新测试未发现任何新问题。

### 2000-06-08

1.  添加了大量新的测试用例。

1.  修复添加测试用例时发现的几个错误。

1.  开始添加大量新文档。

### 2000-06-06

1.  添加了复合选择运算符：**UNION**，**UNION ALL**，**INTERSECT**和**EXCEPT**。

1.  添加了在表达式中使用**(SELECT ...)**的支持。

1.  添加对**IN**和**BETWEEN**操作符的支持。

1.  添加对**GROUP BY**和**HAVING**的支持。

1.  NULL 值现在作为 NULL 指针而不是空字符串报告给回调函数。

### 2000-06-03

1.  添加了对表列默认值的支持。

1.  改进了测试覆盖率。修复了几个由改进测试发现的隐秘错误。

### 2000-06-02

1.  在对任何文件进行 UPDATE、INSERT 或 DELETE 更改之前，现在会锁定所有数据库文件。我认为这样做可以安全地同时从多个进程访问同一个数据库。

1.  代码看起来稳定，因此我们现在称其为"beta"版本。

### 2000-06-01

1.  改进了文件锁定支持，使得两个或更多进程（或线程）能够同时访问同一个数据库。不过在这方面还需要更多工作。

### 2000-05-31

1.  向 SELECT 语句添加了对聚合函数的支持（例如**COUNT(*)**，**MIN(...)**

1.  添加了对**SELECT DISTINCT ...**的支持

### 2000-05-30

1.  添加了**LIKE**运算符。

1.  添加了一个**GLOB**运算符：类似于**LIKE**，但它使用 Unix shell 的通配符而不是 SQL 的'%'和'_'通配符。

1.  添加了**COPY**命令，模仿自[PostgreSQL](http://www.postgresql.org/)，使得 SQLite 现在能够读取 PostgreSQL 的**pg_dump**数据库转储实用程序的输出。

1.  添加了一个**VACUUM**命令，调用底层数据库文件的**gdbm_reorganize()**函数。

1.  以及许多、许多的错误修复...

### 2000-05-29

1.  初始公开发布的 Alpha 版本
