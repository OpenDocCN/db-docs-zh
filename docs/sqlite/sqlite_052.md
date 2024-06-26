# SQLite 中的限制

> 原文：[`sqlite.com/limits.html`](https://sqlite.com/limits.html)

"限制"在本文的上下文中指的是不能超过的大小或数量。我们关注的是诸如 BLOB 中的最大字节数或表中的最大列数等问题。

SQLite 最初设计时避免任意限制的策略。当然，每个在有限内存和磁盘空间上运行的程序都有某种类型的限制。但在 SQLite 中，这些限制并没有定义得很好。政策是，如果它适合内存并且您可以用 32 位整数计数它，那么它应该可以工作。

不幸的是，无限制政策已被证明会造成问题。因为上限没有很好地定义，所以它们没有被测试，当将 SQLite 推向极限时通常会发现错误。因此，自大约发布 3.5.8 版本的 SQLite（2008-04-16）以来，它具有明确定义的限制，并且这些限制作为测试套件的一部分进行测试。

本文定义了 SQLite 的限制是什么以及如何为特定应用程序定制这些限制。限制的默认设置通常非常大且几乎适用于每个应用程序。一些应用程序可能希望在某些地方增加限制，但我们预计这种需求会很少。更常见的是，应用程序可能希望重新编译 SQLite，以避免在高级 SQL 语句生成器中存在 bug 或帮助阻止注入恶意 SQL 语句的攻击者时过多地利用资源。

运行时，某些限制可以通过使用 sqlite3_limit()接口在每个连接的基础上进行更改，使用该接口为该接口定义的一个限制类别。运行时限制适用于具有多个数据库的应用程序，其中一些仅用于内部使用，而其他可能受到潜在敌对外部代理控制或受到影响。例如，Web 浏览器应用程序可能使用内部数据库跟踪历史页面查看，但有一个或多个由从 Internet 下载的 JavaScript 应用程序创建和控制的单独数据库。sqlite3_limit()接口允许由受信任代码管理的内部数据库不受限制，同时在由不受信任的外部代码创建或控制的数据库上设置严格限制，以帮助防止拒绝服务攻击。

1.  **字符串或 BLOB 的最大长度**

    在 SQLite 中，字符串或 BLOB 中的最大字节数由预处理器宏 SQLITE_MAX_LENGTH 定义。该宏的默认值为 10 亿（1 千万或 1,000,000,000）。您可以使用类似以下的命令行选项在编译时提高或降低此值：

    > -DSQLITE_MAX_LENGTH=123456789

    当前的实现仅支持字符串或 BLOB 长度最多为 2³¹-1 或 2147483647。并且一些内置函数，如 hex()，在此点之前可能会失败。在安全敏感的应用程序中，最好不要尝试增加最大字符串和 blob 长度。实际上，如果可能的话，最好将最大字符串和 BLOB 长度降低到几百万左右的范围。

    在 SQLite 的 INSERT 和 SELECT 处理的某个部分，数据库中每行的完整内容被编码为单个 BLOB。因此，SQLITE_MAX_LENGTH 参数还确定了行中的最大字节数。

    通过使用 sqlite3_limit(db,SQLITE_LIMIT_LENGTH,size)接口可以在运行时降低字符串或 BLOB 的最大长度。

1.  **列的最大数量**

    SQLITE_MAX_COLUMN 编译时参数用于设置上限：

    +   表中的列数

    +   索引中的列数

    +   视图中的列数

    +   UPDATE 语句的 SET 子句中的项数

    +   SELECT 语句结果集中的列数

    +   GROUP BY 或 ORDER BY 子句中的项数

    +   INSERT 语句中值的数量

    SQLITE_MAX_COLUMN 的默认设置为 2000。您可以在编译时将其更改为 32767 等大的值。另一方面，许多经验丰富的数据库设计师会认为，规范化良好的数据库永远不会需要超过 100 个列。

    在大多数应用程序中，列数较少，只有几十个。SQLite 代码生成器中使用算法的地方的复杂度为 O(N²)，其中 N 是列的数量。因此，如果您将 SQLITE_MAX_COLUMN 重新定义为一个非常巨大的数字，并且生成使用大量列的 SQL，您可能会发现 sqlite3_prepare_v2()运行缓慢。

    可以使用 sqlite3_limit(db,SQLITE_LIMIT_COLUMN,size)接口在运行时降低最大列数。

1.  **SQL 语句的最大长度**

    SQL 语句中文本的最大字节数限制为 SQLITE_MAX_SQL_LENGTH，默认为 1,000,000,000。

    如果 SQL 语句的长度限制为 100 万字节，那么显然您将无法通过将它们作为字面值嵌入到 INSERT 语句中来插入几百万字节的字符串。但无论如何，您都不应该这样做。使用宿主参数进行数据。准备短的 SQL 语句如下：

    > INSERT INTO tab1 VALUES(?,?,?);

    然后使用 sqlite3_bind_XXXX()函数将大字符串值绑定到 SQL 语句。使用绑定可以避免在字符串中转义引号字符，减少 SQL 注入攻击的风险。它还运行更快，因为大字符串不需要解析或复制太多次。

    可以使用 sqlite3_limit(db, SQLITE_LIMIT_SQL_LENGTH, size) 接口在运行时降低 SQL 语句的最大长度。

1.  **连接中的最大表数**

    SQLite 不支持包含超过 64 张表的连接。此限制源于 SQLite 代码生成器在查询优化器中使用的位图，每个连接表对应一个位。

    SQLite 使用高效的 查询规划算法，因此即使是大型连接也可以快速地进行 准备。因此，没有机制来提高或降低连接中表的数量限制。

1.  **表达式树的最大深度**

    SQLite 将表达式解析为树进行处理。在代码生成期间，SQLite 递归地遍历此树。因此，表达式树的深度受限以避免使用过多的堆栈空间。

    参数 `SQLITE_MAX_EXPR_DEPTH` 确定表达式树的最大深度。如果该值为 0，则不会施加限制。当前实现的默认值为 1000。

    运行时可以使用 sqlite3_limit(db, SQLITE_LIMIT_EXPR_DEPTH, size) 接口降低表达式树的最大深度，前提是 `SQLITE_MAX_EXPR_DEPTH` 初始值为正数。换句话说，如果表达式深度在编译时有限制，则可以在运行时降低最大表达式深度。如果 `SQLITE_MAX_EXPR_DEPTH` 在编译时设置为 0（表示表达式深度无限制），则 sqlite3_limit(db, SQLITE_LIMIT_EXPR_DEPTH, size) 不起作用。

1.  **函数中的最大参数数目**

    参数 `SQLITE_MAX_FUNCTION_ARG` 确定可以传递给 SQL 函数的最大参数数目。此限制的默认值为 100。SQLite 应该能够处理具有数千个参数的函数。但是，我们怀疑任何试图调用超过少数参数的函数的人实际上是在寻找使用 SQLite 的系统中的安全漏洞，而不是做有用的工作，因此出于这个原因我们将此参数设置相对较低。

    函数的参数数量有时存储在有符号字符中。因此，`SQLITE_MAX_FUNCTION_ARG` 的硬上限为 127。

    可以使用 sqlite3_limit(db, SQLITE_LIMIT_FUNCTION_ARG, size) 接口在运行时降低函数中的最大参数数目。

1.  **复合 SELECT 语句中的最大项数**

    复合 SELECT 语句是由联合、联合全部、差集或交集运算符连接的两个或多个 SELECT 语句。我们称复合 SELECT 中的每个单独 SELECT 语句为一个“项”。

    SQLite 中的代码生成器使用递归算法处理复合 SELECT 语句。为了限制堆栈的大小，因此我们限制复合 SELECT 中的项数。最大项数是 SQLITE_MAX_COMPOUND_SELECT，默认为 500。我们认为这是一个很大的分配，因为在实际应用中，我们几乎从不见复合选择中的项数超过个位数。

    可以使用 sqlite3_limit(db,SQLITE_LIMIT_COMPOUND_SELECT,size) 接口在运行时降低复合 SELECT 项的最大数目。

1.  **LIKE 或 GLOB 模式的最大长度**

    SQLite 默认的 LIKE 和 GLOB 实现中使用的模式匹配算法在某些极端情况下可能表现为 O(N²) 的性能（其中 N 是模式中的字符数）。为了避免来自能够指定自己 LIKE 或 GLOB 模式的不良分子的拒绝服务攻击，LIKE 或 GLOB 模式的长度限制为 SQLITE_MAX_LIKE_PATTERN_LENGTH 字节。此限制的默认值为 50000。现代工作站甚至可以快速评估 50000 字节的极端 LIKE 或 GLOB 模式。仅当模式长度达到数百万字节时，拒绝服务问题才会出现。尽管如此，由于大多数有用的 LIKE 或 GLOB 模式长度最多为几十字节，偏执的应用程序开发人员可能希望将此参数减少到几百字节的范围，如果他们知道外部用户能够生成任意模式。

    可以使用 sqlite3_limit(db,SQLITE_LIMIT_LIKE_PATTERN_LENGTH,size) 接口在运行时降低 LIKE 或 GLOB 模式的最大长度。

1.  **单个 SQL 语句中的最大主机参数数目**

    主机 parameter 是 SQL 语句中的占位符，使用 sqlite3_bind_XXXX() 接口之一填充。许多 SQL 程序员习惯使用问号（"?"）作为主机参数。SQLite 还支持以 ":", "$" 或 "@" 开头的命名主机参数，以及形如 "?123" 的编号主机参数。

    SQLite 语句中的每个主机参数都分配一个编号。这些编号通常从 1 开始，并且每次新增参数时递增。但是，当使用 "?123" 形式时，主机参数编号是跟随问号后面的数字。

    SQLite 为容纳所有主机参数分配空间，从 1 到使用的最大主机参数编号。因此，包含主机参数如?1000000000 的 SQL 语句将需要大量存储空间。这可能轻易超出主机机器的资源。为了防止过多的内存分配，主机参数编号的最大值是 SQLITE_MAX_VARIABLE_NUMBER，默认为 999（适用于 SQLite 版本 3.32.0 之前，即 2020-05-22 发布）或 32766（适用于 SQLite 版本 3.32.0 之后）。

    最大主机参数数量可以在运行时使用 sqlite3_limit(db,SQLITE_LIMIT_VARIABLE_NUMBER,size)接口降低。

1.  **触发器递归的最大深度**

    SQLite 限制触发器的递归深度，以防止涉及递归触发器的语句使用无限量的内存。

    在 SQLite 3.6.18 版本（2009-09-11）之前，触发器不是递归的，因此这个限制是无意义的。从 3.6.18 版本开始，支持递归触发器，但必须使用 PRAGMA recursive_triggers 语句显式启用。从 3.7.0 版本（2009-09-11）开始，默认启用递归触发器，但可以使用 PRAGMA recursive_triggers 手动禁用。只有在启用递归触发器时，SQLITE_MAX_TRIGGER_DEPTH 才有意义。

    默认的最大触发器递归深度为 1000。

1.  **附加数据库的最大数量**

    ATTACH 语句是 SQLite 的一个扩展，允许将两个或更多数据库关联到同一个数据库连接，并且操作它们就像它们是一个单一的数据库。同时附加的数据库数量限制为默认的 SQLITE_MAX_ATTACHED，其默认值为 10。最大附加数据库数量不能超过 125。

    最大可以附加的数据库数量可以在运行时使用 sqlite3_limit(db,SQLITE_LIMIT_ATTACHED,size)接口降低。

1.  **数据库文件中的页面最大数量**

    SQLite 能够限制数据库文件的大小，以防止数据库文件过大并消耗过多的磁盘空间。SQLITE_MAX_PAGE_COUNT 参数是单个数据库文件中允许的最大页面数。尝试插入新数据会使数据库文件增长超过此限制将返回 SQLITE_FULL。

    SQLITE_MAX_PAGE_COUNT 的最大可能设置为 4294967294（2³²-2）。从版本 3.45.0（2024-01-15）开始，4294967294 也是 SQLITE_MAX_PAGE_COUNT 的默认值。使用默认页面大小 4096 字节时，这给出了约 17.5 太字节的最大数据库大小。如果页面大小增加到最大的 65536 字节，则数据库文件可以增长到约 281 太字节。

    可以使用 max_page_count PRAGMA 在运行时提高或降低此限制。

1.  **表中的最大行数**

    表中的理论最大行数为 2⁶⁴（18446744073709551616 或约 1.8e+19）。此限制是不可达的，因为 281 太字节的最大数据库大小将首先达到。281 太字节的数据库最多可以容纳约 2e+13 行，仅当没有索引且每行包含非常少的数据时才能达到这一限制。

1.  **最大数据库大小**

    每个数据库由一个或多个“页”组成。在单个数据库内，每个页面大小相同，但不同的数据库可以具有介于 512 和 65536 之间的二次幂页面大小。数据库文件的最大大小为 4294967294 页。在页面大小为 65536 字节时，这相当于大约 1.4e+14 字节（281 太字节，或 256 tebibytes，或 281474 千兆字节或 256,000 gibibytes）的最大数据库大小。

    此特定上限未经过测试，因为开发人员无法访问能够达到此限制的硬件。然而，测试验证了 SQLite 在数据库达到底层文件系统的最大文件大小（通常远小于最大理论数据库大小）时的正确和合理行为，以及在由于磁盘空间耗尽而无法增长时的行为。

1.  **架构中的最大表数**

    数据库文件中每个表和索引至少需要一个页面。前一句中的“索引”指的是通过 CREATE INDEX 语句显式创建的索引或由 UNIQUE 和 PRIMARY KEY 约束隐式创建的索引。由于数据库文件中页面的最大数量为 2147483646（略多于 20 亿），这也是模式中表和索引数量的上限。

    每当打开数据库时，整个架构将被扫描和解析，并且架构的解析树将保存在内存中。这意味着数据库连接的启动时间和初始内存使用量与架构的大小成正比。
