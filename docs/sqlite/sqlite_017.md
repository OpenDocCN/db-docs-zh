# PRAGMA 语句

> 原文：[`sqlite.com/pragma.html`](https://sqlite.com/pragma.html)

PRAGMA 语句是 SQLite 特有的 SQL 扩展，用于修改 SQLite 库的操作或查询 SQLite 库的内部（非表格）数据。PRAGMA 语句使用与其他 SQLite 命令相同的接口发出（例如 SELECT、INSERT），但在以下重要方面有所不同：

+   PRAGMA 命令专用于 SQLite，与其他任何 SQL 数据库引擎都不兼容。

+   未来的 SQLite 版本可能会删除特定的 PRAGMA 语句并添加其他的 PRAGMA 语句。不保证向后兼容性。

+   如果发出未知的 PRAGMA，不会生成错误消息。未知的 PRAGMA 会被简单地忽略。这意味着如果 PRAGMA 语句中有拼写错误，库不会通知用户这个事实。

+   一些 PRAGMA 在 SQL 编译阶段生效，而不是执行阶段。这意味着如果使用 C 语言的 sqlite3_prepare()、sqlite3_step()、sqlite3_finalize() API（或者类似的封装接口），PRAGMA 可能会在 sqlite3_prepare() 调用期间运行，而不是在通常的 SQL 语句 sqlite3_step() 调用期间运行。或者 PRAGMA 可能会像正常的 SQL 语句一样在 sqlite3_step() 期间运行。PRAGMA 是否在 sqlite3_prepare() 还是 sqlite3_step() 期间运行取决于具体的 PRAGMA 和 SQLite 的具体版本。

+   EXPLAIN 和 EXPLAIN QUERY PLAN 前缀仅影响在 sqlite3_step() 执行期间 SQL 语句的行为。这意味着在 sqlite3_prepare() 阶段生效的 PRAGMA 语句无论是否以 "EXPLAIN" 开头，行为都是一样的。

SQLite 的 C 语言 API 提供了 SQLITE_FCNTL_PRAGMA 文件控制，这使得 VFS 实现有机会添加新的 PRAGMA 语句或覆盖内置 PRAGMA 语句的含义。

* * *

## PRAGMA 命令语法

**pragma-stmt:**

<svg class="pikchr" viewBox="0 0 824.352 99.576"><text x="75" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">PRAGMA</text> <text x="218" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">模式名称</text> <text x="320" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">.</text> <text x="435" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">指示符名称</text> <text x="555" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="656" y="82" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">指示符值</text> <text x="758" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="555" y="44" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">=</text> <text x="656" y="44" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">指示符值</text></svg>

**指示符值：**

<svg class="pikchr" viewBox="0 0 264.499 110.16"><text x="132" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">带符号数</text> <text x="92" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">名称</text> <text x="125" y="92" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">带符号文字</text></svg>

**带符号数：**

<svg class="pikchr" viewBox="0 0 292.013 99.576"><text x="66" y="44" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">+</text> <text x="191" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">数值文字</text> <text x="66" y="82" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">-</text></svg>

指示符可以接受零个或一个参数。参数可以用括号括起来，也可以用等号与指示符名称分隔开来。这两种语法产生相同的结果。在许多指示符中，参数是布尔值。布尔值可以是以下之一：

**1 yes true on

0 no false off**

关键字参数可选择性地出现在引号中。（例如：`'yes' [FALSE]`。）一些指示符将字符串文字作为其参数。当指示符采用关键字参数时，通常也会采用相应的数值等价物。例如，“0”和“no”意思相同，同样，“1”和“yes”也一样。在查询设置的值时，许多指示符返回数字而不是关键字。

PRAGMA 可能在 PRAGMA 名称之前具有可选的模式名称。模式名称是 附加 的数据库的名称或 "main" 或 "temp" 用于主数据库和 TEMP 数据库。如果省略了可选的模式名称，则假定为 "main"。在某些 PRAGMA 中，模式名称无意义且仅仅被忽略。在下面的文档中，显示对模式名称有意义的 PRAGMA 使用 "*schema.*" 前缀。

* * *

## PRAGMA 函数

PRAGMA 返回结果且没有副作用的 PRAGMA 可以像 SELECT 语句一样从普通的 表值函数 中访问。对于每个参与的 PRAGMA，相应的表值函数具有相同的名称，前缀为 7 个字符的 "pragma_"。PRAGMA 参数和模式（如果有）作为参数传递给表值函数，模式作为可选的最后一个参数。

例如，可以使用 index_info pragma 读取索引中列的信息如下：

> ```sql
> PRAGMA index_info('idx52');
> 
> ```

或者，可以使用相同的内容读取：

> ```sql
> SELECT * FROM pragma_index_info('idx52');
> 
> ```

表值函数格式的优点在于查询可以返回 PRAGMA 列的子集，可以包括 WHERE 子句，可以使用聚合函数，并且表值函数可以作为联接中的多个数据源之一。例如，要获取模式中所有索引列的列表，可以查询：

> ```sql
> SELECT DISTINCT m.name || '.' || ii.name AS 'indexed-columns'
>   FROM sqlite_schema AS m,
>        pragma_index_list(m.name) AS il,
>        pragma_index_info(il.name) AS ii
>  WHERE m.type='table'
>  ORDER BY 1;
> 
> ```

附加说明：

+   表值函数仅适用于内置 PRAGMA，不适用于使用 SQLITE_FCNTL_PRAGMA 文件控制定义的 PRAGMA。

+   表值函数仅适用于返回结果且没有副作用的 PRAGMA。

+   此功能可用于通过首先创建一个使用 [information schema](https://en.wikipedia.org/wiki/Information_schema) 实现的单独模式来实现。

    > ```sql
    > ATTACH ':memory:' AS 'information_schema';
    > 
    > ```

    然后在该模式中创建 视图，使用表值 PRAGMA 函数实现官方信息模式表。

+   PRAGMA 功能的表值函数特性在 SQLite 版本 3.16.0 (2017-01-02) 中添加。SQLite 的早期版本无法使用此功能。

* * *

## PRAGMA 列表

+   分析限制

+   应用程序 ID

+   自动清理

+   自动索引

+   忙碌超时

+   缓存大小

+   缓存溢出

+   ~~区分大小写的 like¹~~

+   单元大小检查

+   全同步检查点

+   排序列表

+   编译选项

+   ~~更改计数¹~~

+   ~~数据存储目录¹~~

+   数据版本

+   数据库列表

+   ~~默认缓存大小¹~~

+   延迟外键检查

+   ~~空结果回调¹~~

+   编码方式

+   外键检查

+   外键列表

+   外键

+   空闲列表数

+   ~~全列名¹~~

+   完全同步

+   函数列表

+   硬堆限制

+   忽略检查约束

+   增量清理

+   索引信息

+   索引列表

+   索引扩展信息

+   完整性检查

+   日志模式

+   日志大小限制

+   传统表修改

+   传统文件格式

+   锁定模式

+   最大页数

+   内存映射大小

+   模块列表

+   优化

+   页数

+   页大小

+   解析器追踪²

+   PRAGMA 列表

+   仅查询

+   快速检查

+   读未提交

+   递归触发器

+   反向无序选择

+   模式版本³

+   安全删除

+   ~~短列名¹~~

+   内存收缩

+   软堆限制

+   统计³

+   同步方式

+   表信息

+   表列表

+   表扩展信息

+   临时存储

+   ~~临时存储目录¹~~

+   线程

+   可信模式

+   用户版本

+   VDBE 操作追踪²

+   VDBE 调试²

+   VDBE 列表²

+   VDBE 追踪²

+   WAL 自动检查点

+   wal_checkpoint

+   writable_schema³

注意：

1.  被 ~~划掉~~ 的 pragma 已经被弃用。不要使用它们。它们仅存在于历史兼容性。

1.  这些 pragma 仅在使用非标准编译时选项的构建中可用。

1.  这些 pragma 用于测试 SQLite，不建议在应用程序中使用。

<h _id="pragma_analysis_limit" style="display:none">PRAGMA analysis_limit</h>

* * *

**PRAGMA analysis_limit;

PRAGMA analysis_limit =** *N***;**

查询或更改 approximate ANALYZE 设置的限制。这是 ANALYZE 命令在每个索引中大约检查的行数。如果省略参数 *N*，则分析限制保持不变。如果限制为零，则分析限制被禁用，ANALYZE 命令将检查每个索引的所有行。如果 N 大于零，则将分析限制设置为 N，并且后续的 ANALYZE 命令将在检查大约 N 行后停止分析每个索引。如果 N 是负数或者不是整数值，则 pragma 的行为就像省略了 N 参数一样。在所有情况下，返回的值是用于后续 ANALYZE 命令的新分析限制。

这个 pragma 可以帮助 ANALYZE 命令在大型数据库上运行得更快。当仅检查每个索引的一部分时，分析的结果可能不太好，但通常是足够好的。将 N 设置为 100 或 1000 可以让 ANALYZE 命令在庞大的数据库文件上快速完成。

这个 pragma 是在 SQLite 版本 3.32.0（2020-05-22）中添加的。当前的实现仅使用 N 值的低 31 位 - 高位比特会被忽略。未来的 SQLite 版本可能会开始使用高位比特。

从 SQLite 版本 3.46.0（2024-05-23）开始，推荐使用 ANALYZE 时配合使用 PRAGMA optimize 命令。PRAGMA optimize 将自动设置一个合理的临时分析限制，确保 PRAGMA optimize 命令即使在大型数据库上也能快速完成。使用 PRAGMA optimize 而不是直接运行 ANALYZE 的应用程序不需要设置分析限制。<h _id="pragma_application_id" style="display:none">PRAGMA application_id</h>

* * *

**PRAGMA** *schema.***application_id;

PRAGMA** *schema.***application_id =** *integer* **;**

应用程序标识 PRAGMA 用于查询或设置位于数据库头偏移 68 的 32 位有符号大端“应用程序 ID”整数。将 SQLite 用作其应用程序文件格式的应用程序应设置应用程序 ID 整数为唯一整数，以便像[file(1)](http://www.darwinsys.com/file/)这样的实用程序可以确定具体的文件类型，而不仅仅报告“SQLite3 Database”。可以通过查阅 SQLite 源代码库中的[magic.txt](https://www.sqlite.org/src/artifact?ci=trunk&filename=magic.txt)文件来查看分配的应用程序 ID 列表。

另请参阅 user_version pragma。 <h _id="pragma_auto_vacuum" style="display:none">PRAGMA auto_vacuum</h>

* * *

**PRAGMA** *schema.***auto_vacuum;

PRAGMA ** *schema.***auto_vacuum =** *0 | NONE | 1 | FULL | 2 | INCREMENTAL***;**

查询或设置数据库中的自动清理状态。

自动清理的默认设置为 0 或“none”，除非使用 SQLITE_DEFAULT_AUTOVACUUM 编译时选项。设置为“none”表示自动清理已禁用。当禁用自动清理并从数据库中删除数据时，数据库文件的大小保持不变。未使用的数据库文件页面将添加到“freelist”中，并在后续插入时重用。因此，没有数据库文件空间会丢失。但是，数据库文件不会收缩。在这种模式下，可以使用 VACUUM 命令重建整个数据库文件，从而回收未使用的磁盘空间。

当自动清理模式为 1 或“full”时，空闲列表页面将移动到数据库文件的末尾，并且在每次事务提交时将数据库文件截断以删除空闲列表页面。但请注意，自动清理仅会从文件中截断空闲列表页面。自动清理不会对数据库进行碎片整理或重新打包单个数据库页面，就像 VACUUM 命令所做的那样。实际上，由于在文件中移动页面，自动清理可能会加剧碎片化问题。

自动清理仅在数据库存储了额外信息以便追踪每个数据库页面到其引用者时才可能实现。因此，在创建任何表之前必须打开自动清理。在创建表后无法启用或禁用自动清理。

当自动清理的值为 2 或“incremental”时，存储在数据库文件中的额外信息用于执行自动清理，但不会像 auto_vacuum=full 那样在每次提交时自动执行清理。在增量模式下，必须调用单独的 incremental_vacuum PRAGMA 来触发自动清理。

数据库连接可以随时在完全和增量自动清理模式之间更改。但是，从"none"更改为"full"或"incremental"只能在数据库是新的情况下（尚未创建任何表）或通过运行 VACUUM 命令时才能发生。要更改自动清理模式，请首先使用 auto_vacuum PRAGMA 设置新的所需模式，然后调用 VACUUM 命令重新组织整个数据库文件。即使在空数据库上，从"full"或"incremental"更改回"none"始终需要运行 VACUUM。

当未提供参数调用 auto_vacuum PRAGMA 时，它返回当前的 auto_vacuum 模式。

<h _id="pragma_automatic_index" style="display:none">PRAGMA automatic_index</h>

* * *

**PRAGMA automatic_index;**

**PRAGMA automatic_index =** *boolean***;**

查询、设置或清除自动索引功能。

自动索引从版本 3.7.17（2013-05-20）开始默认启用，但在 SQLite 的未来版本中可能会更改。 <h _id="pragma_busy_timeout" style="display:none">PRAGMA busy_timeout</h>

* * *

**PRAGMA busy_timeout;**

PRAGMA busy_timeout =** *milliseconds***;**

查询或更改繁忙超时的设置。此 PRAGMA 是 sqlite3_busy_timeout() C 语言接口的替代，用于不提供直接访问 sqlite3_busy_timeout()的语言绑定。

每个数据库连接只能拥有一个繁忙处理程序。此 PRAGMA 为进程设置繁忙处理程序，可能会覆盖之前设置的任何繁忙处理程序。 <h _id="pragma_cache_size" style="display:none">PRAGMA cache_size</h>

* * *

**PRAGMA** *schema.***cache_size;

PRAGMA **schema.**cache_size =** *pages***;

PRAGMA **schema.**cache_size = -**kibibytes**;**

查询或更改 SQLite 每个打开数据库文件时将在内存中持有的建议最大数据库磁盘页面数。是否遵循此建议取决于应用定义的页面缓存。内置于 SQLite 中的默认页面缓存遵循请求，但是替代的应用定义页面缓存实现可能选择以不同方式解释建议的缓存大小，或者完全忽略它。默认建议的缓存大小为-2000，这意味着缓存大小限制为 2048000 字节的内存。可以使用 SQLITE_DEFAULT_CACHE_SIZE 编译时选项更改默认建议的缓存大小。TEMP 数据库的默认建议缓存大小为 0 页。

如果参数 N 是正数，则建议的缓存大小设置为 N。如果参数 N 是负数，则根据当前页面大小调整缓存页的数量，以使用大约 abs(N*1024) 字节的内存。SQLite 记住页面缓存的页数，而不是使用的内存量。因此，如果使用负数设置缓存大小，然后随后更改页面大小（使用 PRAGMA page_size 命令），则缓存内存的最大量会按比例增加或减少。

*向后兼容性说明:* 在版本 3.7.10（2012-01-16）之前，使用负 N 的 `cache_size` 的行为有所不同。在早期版本中，缓存中的页面数量设置为 N 的绝对值。

使用 `cache_size` pragma 更改缓存大小时，此更改仅在当前会话中有效。当关闭并重新打开数据库时，缓存大小将恢复为默认值。

默认页面缓存实现不会一次性分配全部缓存内存的完整量。缓存内存会按需以较小的块分配。`page_cache` 设置是缓存可以使用的内存量的（建议的）上限，而不是它将一直使用的内存量。这是默认页面缓存实现的行为，但如果应用程序定义了页面缓存，它可以自由地采用不同的行为。<h _id="pragma_cache_spill" style="display:none">PRAGMA cache_spill</h>

* * *

**PRAGMA cache_spill;**

PRAGMA cache_spill=***boolean***;

PRAGMA** *schema.***cache_spill=*N*;**

`cache_spill` 命令允许或禁用分页器在事务进行中将脏缓存页溢出到数据库文件的能力。默认情况下启用 `cache_spill`，大多数应用程序应该保持这种设置，因为缓存溢出通常是有利的。然而，缓存溢出会导致数据库文件获取排他锁的副作用。因此，一些具有大型长时间运行事务的应用程序可能希望禁用缓存溢出，以防止应用程序在事务提交之前获取对数据库的排他锁。

"PRAGMA cache_spill=*N*" 形式的此 pragma 设置了溢出发生所需的最小缓存大小阈值。在进行溢出之前，缓存中的页面数量必须超过 `cache_spill` 阈值和由 PRAGMA cache_size 语句设置的最大缓存大小。

"PRAGMA cache_spill=*boolean*" 形式的这个 pragma 适用于连接到数据库连接的所有数据库。但这个语句的 "PRAGMA cache_spill=*N*" 形式只适用于 "main" 模式或者语句中指定的其他模式。

* * *

**PRAGMA case_sensitive_like =** *boolean***;**

LIKE 操作符的默认行为是对 ASCII 字符不区分大小写。因此，默认情况下 **'a' LIKE 'A'** 是 true。case_sensitive_like pragma 安装一个新的应用程序定义的 LIKE 函数，该函数是否区分大小写取决于 case_sensitive_like pragma 的值。禁用 case_sensitive_like 时，表达默认的 LIKE 行为。启用 case_sensitive_like 时，大小写变得重要。所以，例如，**'a' LIKE 'A'** 是 false，但 **'a' LIKE 'a'** 仍然是 true。

这个 pragma 使用 sqlite3_create_function() 来重载 LIKE 和 GLOB 函数，这可能会覆盖应用程序注册的之前版本的 LIKE 和 GLOB。这个 pragma 只改变 SQL LIKE 操作符的行为。它不会改变 sqlite3_strlike() C 语言接口的行为，后者总是不区分大小写的。

**警告：**如果数据库中的模式任何地方使用了 LIKE 操作符，比如在 CHECK 约束 中、在 expression index 中或者在 partial index 的 WHERE 子句中，那么使用这个 PRAGMA 改变 LIKE 操作符的定义可能导致数据库看起来是损坏的。PRAGMA integrity_check 会报告错误。实际上数据库并不是真的损坏，因为将 LIKE 的行为改回到模式定义和数据库填充时的样子会清除问题。如果 LIKE 只出现在索引中，那么可以通过运行 REINDEX 来清除问题。尽管如此，使用 case_sensitive_like pragma 是不建议的。

**这个 pragma 已被弃用**，只是为了向后兼容而存在。新应用程序应避免使用这个 pragma。旧应用程序应尽早停止使用这个 pragma。当 SQLite 使用 SQLITE_OMIT_DEPRECATED 编译时，可以省略这个 pragma。

<h _id="pragma_cell_size_check" style="display:none">PRAGMA cell_size_check

* * *

**PRAGMA cell_size_check**

PRAGMA cell_size_check =** *boolean***;**

cell_size_check pragma 在最初从磁盘读取数据库 B 树页面时启用或禁用额外的健全性检查。启用单元大小检查时，数据库损坏会更早地被检测到，且不太可能“蔓延”。但是，进行额外检查会带来一定的性能损耗，因此默认情况下关闭单元大小检查。 <h _id="pragma_checkpoint_fullfsync" style="display:none">PRAGMA checkpoint_fullfsync</h>

* * *

**PRAGMA checkpoint_fullfsync

PRAGMA checkpoint_fullfsync =** *boolean***;**

查询或更改检查点操作的全 fsync 标志。如果设置了此标志，则在支持 F_FULLFSYNC 的系统上，在检查点操作期间使用 F_FULLFSYNC 同步方法。检查点 _fullfsync 标志的默认值为关闭。只有 Mac OS-X 支持 F_FULLFSYNC。

如果设置了 fullfsync 标志，则所有同步操作都使用 F_FULLFSYNC 同步方法，并且 checkpoint_fullfsync 设置是无关紧要的。

<h _id="pragma_collation_list" style="display:none">PRAGMA collation_list</h>

* * *

**PRAGMA collation_list;**

返回当前数据库连接定义的排序序列列表。

<h _id="pragma_compile_options" style="display:none">PRAGMA compile_options</h>

* * *

**PRAGMA compile_options;**

此 pragma 在构建 SQLite 时返回编译时选项的名称，每行一个选项。返回的选项名称省略了"SQLITE_"前缀。还可以参考 C/C++接口 sqlite3_compileoption_get()和 SQL 函数 sqlite_compileoption_get()。

<h _id="pragma_count_changes" style="display:none">PRAGMA count_changes</h>

* * *

**PRAGMA count_changes;

PRAGMA count_changes =** boolean**;**

查询或更改计数更改标志。通常情况下，当未设置计数更改标志时，INSERT，UPDATE 和 DELETE 语句不返回数据。当设置了计数更改时，每个命令返回一个包含一个整数值的单行数据 - 命令插入，修改或删除的行数。返回的更改计数不包括触发器执行的任何插入，修改或删除，任何由外键操作自动进行的更改，或由 upsert 引起的更新。

另一种获取行更改计数的方法是使用 sqlite3_changes()或 sqlite3_total_changes()接口。不过有一个细微的区别。当针对使用 INSTEAD OF trigger 的视图运行 INSERT、UPDATE 或 DELETE 时，count_changes pragma 报告触发触发器的视图中的行数，而 sqlite3_changes()和 sqlite3_total_changes()不会。

**此 pragma 已弃用**，仅用于向后兼容。新应用程序应避免使用此 pragma。较旧的应用程序应尽早停止使用此 pragma。当使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，此 pragma 可被省略。

<h _id="pragma_data_store_directory" style="display:none">PRAGMA 数据存储目录</h>

* * *

**PRAGMA 数据存储目录;

`PR   PRAGMA 数据存储目录 = '***directory-name***';**

查询或更改 sqlite3_data_directory 全局变量的值，Windows 操作系统接口后端用于确定使用相对路径名指定的数据库文件存储位置。

更改 data_store_directory 设置不是线程安全的。如果应用程序中的另一个线程同时运行任何 SQLite 接口，请勿更改 data_store_directory 设置。这样做会导致未定义的行为。更改 data_store_directory 设置会写入 sqlite3_data_directory 全局变量，而该全局变量没有受到互斥锁的保护。

该功能是为 WinRT 提供的，WinRT 没有用于读取或更改当前工作目录的 OS 机制。在任何其他情况下使用此 pragma 是不鼓励的，并且可能在将来的版本中被禁止。

**此 pragma 已弃用**，仅用于向后兼容。新应用程序应避免使用此 pragma。较旧的应用程序应尽早停止使用此 pragma。当使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，此 pragma 可被省略。

<h _id="pragma_data_version" style="display:none">PRAGMA 数据版本</h>

* * *

**PRAGMA** *schema.***data_version;**

"PRAGMA 数据版本"命令提供了数据库文件已修改的指示。将数据库内容保存在内存中的交互式程序或在屏幕上显示数据库内容的程序可以使用 PRAGMA 数据版本命令来确定它们是否需要刷新并重新加载其内存或更新屏幕显示。

两次调用相同连接的 "PRAGMA data_version" 返回的整数值如果在期间其他连接提交了数据库更改，则不同。对于在同一数据库连接上进行的提交，"PRAGMA data_version" 值保持不变。"PRAGMA data_version" 的行为对所有数据库连接都相同，包括独立进程中的数据库连接和 共享缓存 数据库连接。

"PRAGMA data_version" 的值是每个数据库连接的本地属性，因此在分开的数据库连接上同时调用两次 "PRAGMA data_version" 返回的值通常是不同的，即使底层数据库相同。只有在不同时间点上由同一数据库连接返回的 "PRAGMA data_version" 值之间进行比较才有意义。<h _id="pragma_database_list" style="display:none">PRAGMA database_list</h>

* * *

**PRAGMA database_list;**

此 pragma 的工作方式类似于查询，返回当前数据库连接附加的每个数据库的一行。第二列是主数据库文件为"main"，用于存储 TEMP 对象的数据库文件为"temp"，或者是附加数据库的名称。第三列是数据库文件本身的名称，如果数据库没有关联文件则为空字符串。

<h _id="pragma_default_cache_size" style="display:none">PRAGMA default_cache_size</h>

* * *

**PRAGMA** *schema.***default_cache_size;

PRAGMA** *schema.***default_cache_size =** *Number-of-pages***;**

此 pragma 查询或设置建议的每个打开的数据库文件分配的磁盘缓存的最大页面数。此 pragma 与 cache_size 的区别在于此处设置的值跨数据库连接保持不变。默认缓存大小的值存储在数据库文件头部偏移量为 48 的 4 字节大端整数中。

**此 pragma 已被弃用**，仅用于向后兼容。新应用程序应避免使用此 pragma。旧应用程序应尽早停止使用此 pragma。在使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，可以从构建中省略此 pragma。

<h _id="pragma_defer_foreign_keys" style="display:none">PRAGMA defer_foreign_keys</h>

* * *

**PRAGMA defer_foreign_keys

PRAGMA defer_foreign_keys =** *boolean***;**

当 defer_foreign_keys PRAGMA 开启时，所有外键约束的强制执行被延迟直到最外层事务提交。defer_foreign_keys pragma 默认为 OFF，因此仅在创建为“DEFERRABLE INITIALLY DEFERRED”的外键约束时才会延迟。每次提交或回滚时，defer_foreign_keys pragma 都会自动关闭。因此，每个事务必须单独启用 defer_foreign_keys pragma。当然，如果启用了外键约束，这个 pragma 才有意义。

sqlite3_db_status(db,SQLITE_DBSTATUS_DEFERRED_FKS,...) C 语言接口可用于在事务期间确定是否存在延迟和未解析的外键约束。

<h _id="pragma_empty_result_callbacks" style="display:none">PRAGMA 空结果回调</h>

* * *

**PRAGMA 空结果回调;

PRAGMA 空结果回调 =** *boolean***;**

查询或更改空结果回调标志。

空结果回调标志仅影响 sqlite3_exec() API。通常情况下，当清除空结果回调标志时，提供给 sqlite3_exec()的回调函数不会在返回零行数据的命令中调用。在这种情况下设置空结果回调时，回调函数将精确调用一次，第三个参数设置为 0（NULL）。这是为了使使用 sqlite3_exec() API 的程序在查询不返回数据时也能检索到列名。

**此 pragma 已被弃用**，仅用于向后兼容。新应用程序应避免使用此 pragma。旧应用程序应尽快停止使用此 pragma。在使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，可能会从构建中省略此 pragma。

<h _id="pragma_encoding" style="display:none">PRAGMA 编码</h>

* * *

**PRAGMA 编码;

PRAGMA 编码 = 'UTF-8';

PRAGMA 编码 = 'UTF-16';

PRAGMA 编码 = 'UTF-16le';

PRAGMA 编码 = 'UTF-16be';**

在第一种形式中，如果主数据库已经创建，则此 pragma 返回主数据库使用的文本编码之一：'UTF-8'、'UTF-16le'（小端 UTF-16 编码）或'UTF-16be'（大端 UTF-16 编码）。如果主数据库尚未创建，则返回的值是会话创建主数据库时将使用的文本编码。

第二至第五种形式的此 pragma 设置了如果此会话创建主数据库，则主数据库将使用的编码。字符串'UTF-16'被解释为“使用本地机器字节顺序的 UTF-16 编码”。创建数据库后无法更改文本编码，任何尝试这样做都将被静默忽略。

如果没有首先使用此 PRAGMA 设置编码，则主数据库将使用由用于打开连接的 API 确定的默认编码。

一旦为数据库设置了编码，就无法更改。

由 ATTACH 命令创建的数据库始终使用与主数据库相同的编码。试图使用与“主”数据库不同的文本编码 ATTACH 数据库将失败。

<h _id="pragma_foreign_key_check" style="display:none">PRAGMA foreign_key_check</h>

* * *

**PRAGMA** *schema.*`foreign_key_check`;

PRAGMA** *schema.***foreign_key_check(***table-name***);**

foreign_key_check` PRAGMA`检查数据库或名为“*table-name*”的表，查找违反的外键约束。每个外键违规都会返回一行输出。每个结果行有四列。第一列是包含 REFERENCES 子句的表的名称。第二列是包含无效 REFERENCES 子句的行的 rowid，如果子表是 WITHOUT ROWID 表，则为 NULL。第三列是被引用的表的名称。第四列是失败的特定外键约束的索引。foreign_key_check` PRAGMA 输出的第四列与 foreign_key_list PRAGMA 输出的第一列相同。指定“*table-name*”时，只会检查由 CREATE TABLE 语句中 REFERENCES 子句创建的外键约束。

<h _id="pragma_foreign_key_list" style="display:none">PRAGMA foreign_key_list</h>

* * *

**PRAGMA foreign_key_list(***table-name***);**

此 PRAGMA 为每个在 CREATE TABLE 语句中的 REFERENCES 子句创建的外键约束返回一行。 <h _id="pragma_foreign_keys" style="display:none">PRAGMA foreign_keys</h>

* * *

**PRAGMA foreign_keys;

PRAGMA foreign_keys =** *boolean***;**

查询、设置或清除外键约束的执行。

此 PRAGMA 在事务内无效；只有在没有待处理的 BEGIN 或 SAVEPOINT 时才能启用或禁用外键约束执行。

更改 foreign_keys 设置会影响使用数据库连接准备的所有语句的执行，包括在更改设置之前准备的语句。在更改 foreign_keys 设置后，使用传统 sqlite3_prepare()接口准备的现有语句可能会因 SQLITE_SCHEMA 错误而失败。

自 SQLite version 3.6.19 起，默认情况下外键强制执行设置为关闭。但是，这可能会在将来的 SQLite 发布版本中更改。可以在编译时使用 SQLITE_DEFAULT_FOREIGN_KEYS 预处理宏指定外键强制执行的默认设置。为了尽量减少未来的问题，应用程序应根据应用程序的要求设置外键强制执行标志，而不依赖于默认设置。 <h _id="pragma_freelist_count" style="display:none">PRAGMA freelist_count</h>

* * *

**PRAGMA** *schema.***freelist_count;**

返回数据库文件中未使用页面的数量。

<h _id="pragma_full_column_names" style="display:none">PRAGMA full_column_names</h>

* * *

**PRAGMA full_column_names;

PRAGMA full_column_names =** *boolean***;**

查询或更改 full_column_names 标志。此标志与 short_column_names 标志一起决定 SQLite 分配 SELECT 语句结果列名称的方式。结果列名称按以下顺序应用规则命名：

1.  如果结果上有 AS 子句，则列的名称为 AS 子句的右侧。

1.  如果结果是一般表达式而不仅仅是源表列的名称，则结果的名称是表达式文本的副本。

1.  如果 short_column_names pragma 打开，则结果的名称为源表列名，不包含源表名前缀：COLUMN。

1.  如果同时关闭 short_column_names 和 full_column_names pragma，则适用情况（2）。

1.  结果列的名称是源表和源列名的组合：TABLE.COLUMN

**此 pragma 已被弃用**，仅用于向后兼容。新应用程序应避免使用此 pragma。旧应用程序应尽快停止使用此 pragma。在使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，可能会省略此 pragma。

<h _id="pragma_fullfsync" style="display:none">PRAGMA fullfsync</h>

* * *

**PRAGMA fullfsync

PRAGMA fullfsync =** *boolean***;**

查询或更改 fullfsync 标志。此标志确定系统是否支持 F_FULLFSYNC 同步方法的使用。fullfsync 标志的默认值为关闭。只有 Mac OS X 支持 F_FULLFSYNC。

另请参阅 checkpoint_fullfsync。

<h _id="pragma_function_list" style="display:none">PRAGMA function_list</h>

* * *

**PRAGMA function_list;**

此 PRAGMA 返回数据库连接已知的 SQL 函数列表。结果的每一行描述一个单独 SQL 函数的调用签名。某些 SQL 函数在结果集中可能会有多行，如果它们可以（例如）以不同数量的参数被调用，或者可以接受各种编码的文本。 <h _id="pragma_hard_heap_limit" style="display:none">PRAGMA hard_heap_limit</h>

* * *

**PRAGMA hard_heap_limit

PRAGMA hard_heap_limit=***N*

此 PRAGMA 命令会调用 sqlite3_hard_heap_limit64() 接口，并传递参数 N，如果指定了 N 并且 N 是一个小于当前硬堆限制的正整数。hard_heap_limit PRAGMA 始终返回与 sqlite3_hard_heap_limit64(-1) C 语言函数返回的相同整数。也就是说，它始终返回经过此 PRAGMA 强加的硬堆限制值之后的值。

此 PRAGMA 命令只能降低堆限制，不能提高堆限制。必须使用 C 语言接口 sqlite3_hard_heap_limit64() 来提高堆限制。

另请参阅 soft_heap_limit PRAGMA。 <h _id="pragma_ignore_check_constraints" style="display:none">PRAGMA ignore_check_constraints</h>

* * *

**PRAGMA ignore_check_constraints =** *boolean***;**

这个 PRAGMA 命令用于启用或禁用 CHECK 约束的执行。默认设置为 off，意味着 CHECK 约束默认情况下是启用的。

<h _id="pragma_incremental_vacuum" style="display:none">PRAGMA incremental_vacuum</h>

* * *

**PRAGMA** *schema.***incremental_vacuum***(N)***;

**PRAGMA** *schema.***incremental_vacuum;**

incremental_vacuum PRAGMA 命令会移除最多 *N* 页从 freelist 中。数据库文件会相应地被截断相同数量的页。如果数据库不是在 auto_vacuum=incremental 模式下，或者 freelist 上没有页，incremental_vacuum PRAGMA 将不起作用。如果 freelist 上的页少于 *N* 页，或者 *N* 小于 1，或者 "(*N*)" 参数被省略，则整个 freelist 将被清除。

<h _id="pragma_index_info" style="display:none">PRAGMA index_info</h>

* * *

**PRAGMA** *schema.***index_info(***index-name***);**

此 PRAGMA 对每个指定索引中的每个关键列返回一行。关键列是实际在 CREATE INDEX 索引语句或者创建索引的 UNIQUE 约束 或 PRIMARY KEY 约束 中命名的列。索引条目通常还包含指回正在索引的表行的辅助列。辅助索引列不会被 index_info PRAGMA 显示，但它们会被 index_xinfo PRAGMA 列出。

index_info PRAGMA 的输出列如下：

1.  索引列在索引中的排名。 （0 表示最左边。）

1.  索引列在被索引表中的排名。值为-1 表示 rowid，值为-2 表示使用表达式。

1.  正在索引的列的名称。如果列是 rowid 或表达式，则此列为 NULL。

如果没有名为*index-name*的索引，但存在名为 WITHOUT ROWID 的表，则（从 SQLite 版本 3.30.0 起，于 2019-10-04）此编译器返回 WITHOUT ROWID 表的主键列，因为它们在底层 B 树的记录中使用，即删除了重复列。<h _id="pragma_index_list" style="display:none">PRAGMA index_list</h>

* * *

**PRAGMA** *schema.* ***index_list(***table-name***);**

此编译器返回与给定表关联的每个索引的一行。

来自 index_list 编译器的输出列如下：

1.  为内部跟踪目的分配给每个索引的序列号。

1.  索引的名称。

1.  如果索引是唯一的，则为"1"，否则为"0"。

1.  如果索引是通过 CREATE INDEX 语句创建的，则为"c"，如果索引是通过 UNIQUE 约束创建的，则为"u"，如果索引是通过 PRIMARY KEY 约束创建的，则为"pk"。

1.  如果索引是部分索引，则为"1"，否则为"0"。

<h _id="pragma_index_xinfo" style="display:none">PRAGMA index_xinfo</h>

* * *

**PRAGMA** *schema.* ***index_xinfo(***index-name***);**

此编译器返回索引中每一列的信息。与此 index_info 编译器不同，此编译器返回索引中每一列的信息，而不仅仅是关键列。（关键列是实际上在 CREATE INDEX 索引语句或 UNIQUE 约束或 PRIMARY KEY 约束中命名的列，用于定位对应于每个索引条目的表条目的辅助列。）

来自 index_xinfo 编译器的输出列如下：

1.  列在索引中的排名（0 表示最左边。关键列在辅助列之前。）

1.  索引列在被索引表中的排名，如果索引列是被索引表的 rowid，则为-1，如果索引在表达式上，则为-2。

1.  正在索引的列的名称，如果索引列是被索引表的 rowid 或表达式，则为 NULL。

1.  如果索引列按索引以逆序（DESC）排序，则为 1，否则为 0。

1.  用于比较索引列中的值的排序序列的名称。

1.  如果索引列是键列，则返回 1，如果索引列是辅助列，则返回 0。

如果不存在名为*索引名称*的索引但存在同名的 WITHOUT ROWID 表，则（截至 SQLite version 3.30.0 于 2019-10-04）此 pragma 返回 WITHOUT ROWID 表的列，这些列按照在底层 B 树记录中使用的方式排列，即首先是去重的主键列，然后是数据列。<h _id="pragma_integrity_check" style="display:none">PRAGMA integrity_check</h>

* * *

**PRAGMA** *schema.***integrity_check;

PRAGMA** *schema.***integrity_check(***N***)

PRAGMA** *schema.***integrity_check(***TABLENAME***)**

此 pragma 对数据库进行低级别的格式化和一致性检查。integrity_check pragma 查找以下内容：

+   不按顺序的表或索引条目

+   格式错误的记录

+   缺失的页码

+   缺失或多余的索引条目

+   UNIQUE、CHECK 和 NOT NULL 约束的错误

+   freelist 的一致性

+   数据库中被多次使用或根本未使用的部分

如果 integrity_check pragma 发现问题，则以描述问题的字符串形式返回结果（作为多行，每行一个列）。pragma integrity_check 在分析退出之前最多返回*N*个错误，其中*N*默认为 100。如果 pragma integrity_check 未发现错误，则返回单行字符串'ok'。

通常情况下，会检查整个数据库文件。但是，如果参数是*TABLENAME*，则仅对名为*TABLENAME*及其相关索引执行检查。这称为“部分完整性检查”。由于仅检查数据库的子集，因此无法检测到未使用文件的部分或同一文件部分被两个或多个表重复使用的错误。仅当*TABLENAME*是 sqlite_schema 或其别名之一时，才会在部分完整性检查中验证 freelist。支持部分完整性检查的功能是从版本 3.33.0（2020-08-14）开始添加的。

PRAGMA integrity_check 无法找到 FOREIGN KEY 错误。请使用 PRAGMA foreign_key_check 命令来查找外键约束的错误。

另请参阅 PRAGMA quick_check 命令，该命令执行大部分 PRAGMA integrity_check 的检查，但运行速度更快。

<h _id="pragma_journal_mode" style="display:none">PRAGMA journal_mode</h>

* * *

**PRAGMA** *schema.***journal_mode;

PRAGMA** *schema.***journal_mode = *DELETE | TRUNCATE | PERSIST | MEMORY | WAL | OFF***

此 pragma 查询或设置与当前数据库连接关联的数据库的日志模式。

查询当前数据库的日志模式时，第一种形式的此 pragma 查询*数据库*。当省略*数据库*时，默认查询“主”数据库。

第二种形式改变了"*数据库*"或者如果省略"*数据库*"，则为所有附加数据库的日志模式。新的日志模式将被返回。如果无法更改日志模式，则将返回原始的日志模式。

DELETE 日志模式是正常行为。在 DELETE 模式下，回滚日志在每次事务结束时被删除。事实上，删除操作是导致事务提交的动作。（请参阅名为 SQLite 中的原子提交的文档以获取更多详细信息。）

TRUNCATE 日志模式通过将回滚日志截断为零长度而不是删除它来提交事务。在许多系统上，截断文件比删除文件要快得多，因为不需要更改包含目录。

PERSIST 日志模式会在每次事务结束时阻止回滚日志被删除。相反，日志的头部会被覆盖为零。这将防止其他数据库连接回滚日志。在删除或截断文件比用零覆盖文件的第一个块要昂贵得多的平台上，PERSIST 日志模式是一种优化方式。参见：PRAGMA journal_size_limit 和 SQLITE_DEFAULT_JOURNAL_SIZE_LIMIT。

MEMORY 日志模式将回滚日志存储在易失性 RAM 中。这节省了磁盘 I/O，但以牺牲数据库的安全性和完整性为代价。如果在设置了 MEMORY 日志模式时，使用 SQLite 的应用程序在事务中间崩溃，则数据库文件很可能会损坏。

WAL 日志模式使用预写式日志而不是回滚日志来实现事务。WAL 日志模式是持久的；设置后它在多个数据库连接和在关闭和重新打开数据库后仍然有效。在 WAL 日志模式下的数据库只能被 SQLite 版本 3.7.0（2010-07-21）或更高版本访问。

OFF 日志模式完全禁用了回滚日志。不会创建任何回滚日志，因此也就没有回滚日志需要删除。OFF 日志模式还禁用了 SQLite 的原子提交和回滚能力。ROLLBACK 命令不再起作用；其行为变得未定义。当日志模式为 OFF 时，应用程序必须避免使用 ROLLBACK 命令。如果应用程序在设置 OFF 日志模式的情况下在事务中间崩溃，则数据库文件很可能会损坏。没有日志记录，语句无法在约束错误后部分取消完成的操作。这也可能导致数据库处于损坏状态。例如，如果重复条目导致 CREATE UNIQUE INDEX 语句在中途失败，它将留下部分创建的、因此是损坏的索引。由于 OFF 日志模式允许使用普通 SQL 来损坏数据库文件，因此在启用 SQLITE_DBCONFIG_DEFENSIVE 时会禁用它。

请注意，内存数据库的`journal_mode`只能是 MEMORY 或者 OFF，不能更改为其他值。尝试将内存数据库的`journal_mode`设置为 MEMORY 或 OFF 以外的任何设置都会被忽略。同时请注意，在活动事务期间无法更改`journal_mode`。

<h _id="pragma_journal_size_limit" style="display:none">PRAGMA journal_size_limit</h>

* * *

**PRAGMA** *schema.***journal_size_limit

**PRAGMA** *schema.***journal_size_limit =** *N* **;**

如果数据库连接处于独占锁定模式或持久日志模式（PRAGMA journal_mode=persist）下，提交事务后，回滚日志文件可能会保留在文件系统中。这会提升后续事务的性能，因为覆盖现有文件比向文件追加更快，但同时也会消耗文件系统空间。在大型事务（例如 VACUUM）之后，回滚日志文件可能会占用非常大的空间。

类似地，在 WAL 模式下，写入前日志文件在检查点后不会被截断。相反，SQLite 会重用现有文件来存储后续的 WAL 条目，因为覆盖比追加更快速。

`journal_size_limit` pragma 可用于限制事务或检查点后在文件系统中保留的回滚日志和 WAL 文件的大小。每次提交事务或 WAL 文件重置时，SQLite 会将文件系统中保留的回滚日志文件或 WAL 文件的大小与此 pragma 设置的大小限制进行比较，如果日志或 WAL 文件超过限制大小，则会截断到限制大小。

上述第二种 pragma 形式用于为指定的数据库设置以字节为单位的新限制。负数表示无限制。要始终将回滚日志和 WAL 文件截断到其最小大小，请将 journal_size_limit 设置为零。上述第一和第二种 pragma 形式均返回一个包含单个整数列的单行结果行 - 字节中的日志大小限制的值。默认的日志大小限制为-1（无限制）。SQLITE_DEFAULT_JOURNAL_SIZE_LIMIT 预处理宏可用于在编译时更改默认的日志大小限制。

此 pragma 仅在在 pragma 名称之前指定的单个数据库上操作（如果未指定数据库，则在“main”数据库上操作）。无法使用单个 PRAGMA 语句更改所有附加数据库的日志大小限制。必须为每个附加数据库分别设置大小限制。<h _id="pragma_legacy_alter_table" style="display:none">PRAGMA legacy_alter_table</h>

* * *

**PRAGMA legacy_alter_table;

PRAGMA legacy_alter_table = *boolean***

此 pragma 设置或查询 legacy_alter_table 标志的值。当此标志打开时，ALTER TABLE RENAME 命令（用于更改表名称）的工作方式与 SQLite 3.24.0（2018-06-04）及更早版本相同。更具体地说，当此标志打开时，ALTER TABLE RENAME 命令仅重写其 CREATE TABLE 语句及其相关的 CREATE INDEX 和 CREATE TRIGGER 语句中表名称的初始出现。对表的其他引用保持不变，包括：

+   在触发器和视图的主体内引用表。

+   在原始 CREATE TABLE 语句中的 CHECK 约束内引用表。

+   在 partial indexes 的 WHERE 子句中引用表。

此 pragma 的默认设置为 OFF，这意味着在模式中对表的所有引用都会转换为新名称。

此 pragma 提供了一个解决方案，用于包含期望旧版 SQLite 中发现的 ALTER TABLE RENAME 的不完整行为的旧程序。新应用程序应保持此标志关闭。

为了兼容旧版 virtual table 实现，在执行 sqlite3_module.xRename 方法时临时开启此标志。此标志的值在 sqlite3_module.xRename 方法执行完毕后将会恢复。

还可以使用 SQLITE_DBCONFIG_LEGACY_ALTER_TABLE 选项通过 sqlite3_db_config()接口切换 legacy alter table 行为的打开和关闭。

传统的 alter table 行为是一个连接特定的设置。打开或关闭此功能会影响 数据库连接 中所有附加的数据库文件。设置不会持久保存。在一个连接中更改此设置不会影响任何其他连接。 <h _id="pragma_legacy_file_format" style="display:none">PRAGMA legacy_file_format</h>

* * *

**PRAGMA legacy_file_format;**

此 `pragma` 不再起作用。它已成为无操作。由 `PRAGMA legacy_file_format` 提供的功能现在可以使用 SQLITE_DBCONFIG_LEGACY_FILE_FORMAT 选项通过 sqlite3_db_config() C 语言接口获得。

<h _id="pragma_locking_mode" style="display:none">PRAGMA locking_mode</h>

* * *

**PRAGMA** *schema.***locking_mode;

PRAGMA** *schema.***locking_mode = *NORMAL | EXCLUSIVE***

此 pragma 设置或查询数据库连接的锁定模式。锁定模式可以是 NORMAL 或 EXCLUSIVE。

在 NORMAL 锁定模式下（默认情况下，除非在编译时使用 SQLITE_DEFAULT_LOCKING_MODE 进行了覆盖），数据库连接在每个读或写事务结束时解锁数据库文件。当锁定模式设置为 EXCLUSIVE 时，数据库连接永远不会释放文件锁。第一次以 EXCLUSIVE 模式读取数据库时，获取并持有共享锁。第一次写入数据库时，获取并持有独占锁。

连接在 EXCLUSIVE 模式下获取的数据库锁可以通过关闭数据库连接或者通过将锁定模式设置回 NORMAL，并访问数据库文件（读或写）来释放。仅仅将锁定模式设置为 NORMAL 是不够的 - 锁定直到下次访问数据库文件才会释放。

将锁定模式设置为 EXCLUSIVE 有三个理由。

1.  应用程序希望阻止其他进程访问数据库文件。

1.  减少文件系统操作的系统调用次数，可能导致轻微的性能提升。

1.  WAL 数据库可以在没有共享内存的情况下以 EXCLUSIVE 模式访问。(附加信息)

当 locking_mode pragma 指定特定数据库时，例如：

> PRAGMA **main.**locking_mode=EXCLUSIVE;

那么锁定模式仅适用于命名数据库。如果没有数据库名称限定符在 "locking_mode" 关键字之前，则锁定模式适用于所有数据库，包括后续使用 ATTACH 命令添加的任何新数据库。

"temp"数据库（用于存储 TEMP 表和索引）和内存数据库总是使用独占锁定模式。temp 和内存数据库的锁定模式不能更改。所有其他数据库默认使用正常锁定模式，并受此 PRAGMA 影响。

如果首次进入 WAL 日志模式时锁定模式为 EXCLUSIVE，则在退出 WAL 日志模式之前无法将锁定模式更改为 NORMAL。如果首次进入 WAL 日志模式时锁定模式为 NORMAL，则可以随时在 NORMAL 和 EXCLUSIVE 之间更改锁定模式，而无需退出 WAL 日志模式。

<h _id="pragma_max_page_count" style="display:none">PRAGMA max_page_count</h>

* * *

**PRAGMA** *schema.***max_page_count;

**PRAGMA** *schema.***max_page_count =** *N***;**

查询或设置数据库文件中的最大页面数。两种形式的 PRAGMA 返回最大页面数。第二种形式尝试修改最大页面数。最大页面数不能低于当前数据库大小。

<h _id="pragma_mmap_size" style="display:none">PRAGMA mmap_size</h>

* * *

**PRAGMA** *schema.***mmap_size;

**PRAGMA** *schema.***mmap_size=***N*

查询或更改单个数据库上设置的用于内存映射 I/O 的最大字节数。第一种形式（无参数）查询当前限制。第二种形式（带有数值参数）为指定的数据库或所有数据库（如果省略了可选数据库名称）设置限制。在第二种形式中，如果省略了数据库名称，则设置的限制将成为后续由 ATTACH 语句添加到数据库连接的所有数据库的默认限制。

参数 N 是将使用内存映射 I/O 访问的数据库文件的最大字节数。如果 N 为零，则禁用内存映射 I/O。如果 N 为负数，则限制将恢复为最近的 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE)设定的默认值，或者如果未设置启动时限制，则为编译时默认值 SQLITE_DEFAULT_MMAP_SIZE。

PRAGMA mmap_size 语句不会增加内存映射 I/O 使用的地址空间超过 SQLITE_MAX_MMAP_SIZE 编译时选项设定的硬限制，也不会超过由 sqlite3_config(SQLITE_CONFIG_MMAP_SIZE)函数的第二个参数在启动时设定的硬限制。

在使用内存映射 I/O 区域时，不能在活动使用中更改内存映射 I/O 区域的大小，以避免在运行 SQL 语句时取消映射内存。因此，如果先前的 mmap_size 非零且同一数据库连接上有其他 SQL 语句并发运行，则 mmap_size pragma 可能是一个空操作。

<h _id="pragma_module_list" style="display:none">PRAGMA 模块列表</h>

* * *

**PRAGMA 模块列表;**

此命令返回注册到数据库连接的虚拟表模块列表。 <h _id="pragma_optimize" style="display:none">PRAGMA 优化</h>

* * *

**PRAGMA optimize;

PRAGMA optimize(***MASK***);

PRAGMA** *schema***.optimize;

PRAGMA** *schema***.optimize(***MASK***);**

尝试优化数据库。前两种形式中的所有模式都会被优化，而在后两种形式中仅会优化指定的模式。

在大多数应用程序中，按以下方式使用 PRAGMA optimize 将帮助 SQLite 实现最佳的查询性能：

1.  使用短期数据库连接的应用程序应在每次关闭数据库连接之前运行"PRAGMA optimize;"一次。

1.  使用长期数据库连接的应用程序在首次打开连接时应运行"PRAGMA optimize=0x10002;"，然后定期运行"PRAGMA optimize;"，比如每天或每小时运行一次。

1.  所有应用程序在模式更改后，特别是在一个或多个 CREATE INDEX 语句后，应运行"PRAGMA optimize;"。

通常情况下，此命令几乎不会执行操作，并且非常快速。只有在需要对一个或多个表执行 ANALYZE 时，才会设置一个临时的分析限制，该限制仅在此命令执行期间有效，以防止 ANALYZE 运行时间过长。

建议实践是，使用短期数据库连接的应用程序应在关闭数据库连接时运行"PRAGMA optimize"一次。使用长期数据库连接的应用程序应在首次打开数据库连接时运行"PRAGMA optimize=0x10002"，然后定期运行"PRAGMA optimize" - 比如每天运行一次。所有应用程序应在模式更改后运行"PRAGMA optimize"，特别是 CREATE INDEX 之后。

预计此命令执行的优化细节会随时间改变和改进。应用程序应预期该命令将在未来版本中执行新的优化。

可选的 MASK 参数是执行优化的位掩码：

| 0x00001 | 调试模式。实际上不执行任何优化，而是返回每个将要执行的优化的一行文本。默认关闭。 |
| --- | --- |
| 0x00002 | 对可能受益的表运行 ANALYZE。默认开启。 |
| 0x00010 | 在运行 ANALYZE 时，设置临时的 PRAGMA analysis_limit 以防止过多运行时间。默认启用。 |
| 0x10000 | 检查所有表的大小，而不仅仅是最近未被使用的表，看看是否有任何表显著增长和缩小，因此可能受益于重新分析。默认情况下关闭。 |

默认 MASK 为 0xfffe。

要查看在不实际执行优化的情况下应执行的所有优化，请运行"PRAGMA optimize(-1)"。

**确定何时运行分析**

在当前实现中，仅当以下所有条件均为真时，才分析表：

1.  MASK 位 0x02 被设置。

1.  该表是普通表，而不是视图或虚拟表。

1.  表名不以"sqlite_"开头。

1.  以下一个或多个为真：

    1.  MASK 的 0x10000 位被设置

    1.  该表上的一个或多个索引缺少 sqlite_stat1 表中的条目。

    1.  查询规划器在当前数据库连接的生命周期内，某个时刻对该表的一个或多个索引使用了 sqlite_stat1 统计信息。

1.  以下一个或多个为真：

    1.  该表上的一个或多个索引缺少 sqlite_stat1 表中的条目。

    1.  与上次在表上运行 ANALYZE 时，表中的行数增加或减少了 10 倍。

未来版本中，表分析的规则可能会发生变化。未来可能会添加新的 MASK 值。对于向后兼容性，未来版本的此 Pragma 可能会接受字符串字面参数，而不是位掩码参数。 <h _id="pragma_page_count" style="display:none">PRAGMA page_count</h>

* * *

**PRAGMA** *schema.* ***page_count;***

返回数据库文件中的总页面数。

<h _id="pragma_page_size" style="display:none">PRAGMA page_size</h>

* * *

**PRAGMA** *schema.* ***page_size;***

PRAGMA** *schema.* ***page_size =** *bytes***;**

查询或设置数据库的页面大小。页面大小必须是 512 到 65536 之间的 2 的幂次方数。

当创建新数据库时，SQLite 根据平台和文件系统为数据库分配页面大小。多年来，默认页面大小几乎总是 1024 字节，但从 SQLite 版本 3.12.0（2016-03-29）开始，默认页面大小增加到 4096 字节。默认页面大小适用于大多数应用程序。

指定新的页面大小不会立即更改页面大小。相反，新页面大小将被记住，并在首次创建数据库时使用，如果在发出 page_size pragma 时数据库尚不存在，或者在不处于 WAL 模式的情况下运行同一数据库连接上的下一个 VACUUM 命令时使用。

SQLITE_DEFAULT_PAGE_SIZE 编译时选项可以用来更改分配给新数据库的默认页面大小。 <h _id="pragma_parser_trace" style="display:none">PRAGMA parser_trace</h>

* * *

**PRAGMA parser_trace =** *boolean***;**

如果 SQLite 是使用 SQLITE_DEBUG 编译选项编译的，那么 parser_trace pragma 可以用来启用对 SQLite 内部使用的 SQL 解析器的跟踪。此功能用于调试 SQLite 本身。

此 pragma 用于在调试 SQLite 本身时使用。仅在使用 SQLITE_DEBUG 编译选项时可用。

<h _id="pragma_pragma_list" style="display:none">PRAGMA pragma_list</h>

* * *

**PRAGMA pragma_list;**

此 pragma 返回数据库连接已知的 PRAGMA 命令列表。 <h _id="pragma_query_only" style="display:none">PRAGMA query_only</h>

* * *

**PRAGMA query_only;

PRAGMA query_only =** *boolean***;**

query_only pragma 在启用时防止对数据库文件进行数据更改。当启用此 pragma 时，任何尝试创建、删除、删除、插入或更新都将导致 SQLITE_READONLY 错误。然而，数据库并非真正只读。您仍然可以运行 checkpoint 或 COMMIT，并且 sqlite3_db_readonly() 的返回值不受影响。

<h _id="pragma_quick_check" style="display:none">PRAGMA quick_check</h>

* * *

**PRAGMA** *schema.***quick_check;

PRAGMA** *schema.***quick_check(***N***)**

PRAGMA *schema.***quick_check(***TABLENAME***)**

pragma 与 integrity_check 类似，但它不验证 UNIQUE 约束，也不验证索引内容是否匹配表内容。通过跳过 UNIQUE 和索引一致性检查，quick_check 能够运行得更快。PRAGMA quick_check 的运行时间为 O(N)，而 PRAGMA integrity_check 需要 O(NlogN) 的时间，其中 N 是数据库中的总行数。否则两个 pragma 是相同的。

<h _id="pragma_read_uncommitted" style="display:none">PRAGMA read_uncommitted</h>

* * *

**PRAGMA read_uncommitted;

PRAGMA read_uncommitted =** *boolean***;**

查询、设置或清除 READ UNCOMMITTED 隔离级别。SQLite 的默认隔离级别为 SERIALIZABLE。任何进程或线程都可以选择 READ UNCOMMITTED 隔离级别，但在共享常规页面和模式缓存的连接之间仍将使用 SERIALIZABLE。使用 sqlite3_enable_shared_cache() API 可以启用缓存共享。默认情况下禁用缓存共享。

有关更多信息，请参阅 SQLite Shared-Cache Mode。

<h _id="pragma_recursive_triggers" style="display:none">PRAGMA recursive_triggers</h>

* * *

**PRAGMA recursive_triggers;

PRAGMA recursive_triggers =** *boolean***;**

查询、设置或清除递归触发器的功能。

更改 recursive_triggers 设置会影响使用数据库连接准备的所有语句的执行，包括更改设置之前准备的语句。使用旧版 sqlite3_prepare()接口准备的现有语句在更改 recursive_triggers 设置后可能会因为 SQLITE_SCHEMA 错误而失败。

在 SQLite 版本 3.6.18 (2009-09-11) 之前，不支持递归触发器。SQLite 的行为总是假定这个编译指示已关闭。支持递归触发器是从版本 3.6.18 开始添加的，但最初默认关闭，以保证兼容性。未来的 SQLite 版本可能会默认打开递归触发器。

触发器的递归深度受到 SQLITE_MAX_TRIGGER_DEPTH 编译时选项设定的严格上限以及运行时由 sqlite3_limit(db,SQLITE_LIMIT_TRIGGER_DEPTH,...)设定的限制。

<h _id="pragma_reverse_unordered_selects" style="display:none">PRAGMA reverse_unordered_selects</h>

* * *

**PRAGMA reverse_unordered_selects;

PRAGMA reverse_unordered_selects =** *boolean***;**

启用此 PRAGMA 后，许多没有 ORDER BY 子句的 SELECT 语句会以它们通常所产生结果的相反顺序输出。这可以帮助调试那些对结果顺序做出无效假设的应用程序。reverse_unordered_selects PRAGMA 适用于大多数 SELECT 语句，但查询规划器有时可能会选择不易反转的算法，此时输出将以相同顺序出现，不受 reverse_unordered_selects 设置影响。

如果 SELECT 语句省略了 ORDER BY 子句，SQLite 不保证结果的顺序。尽管如此，结果的顺序在多次运行中不会改变，因此许多应用程序错误地依赖于任意的输出顺序。然而，有时新版本的 SQLite 会包含优化器增强，导致没有 ORDER BY 子句的查询的输出顺序发生变化。当发生这种情况时，依赖于特定输出顺序的应用程序可能会出现故障。通过在此 PRAGMA 禁用和启用的情况下多次运行应用程序，可以早期识别和修复应用程序对输出顺序做出错误假设可能导致的问题，减少因链接到不同版本的 SQLite 而可能引起的问题。

<h _id="pragma_schema_version" style="display:none">PRAGMA schema_version</h>

* * *

**PRAGMA** *schema.***schema_version;

PRAGMA** *schema.***schema_version =** *integer* ;

schema_version PRAGMA 将获取或设置在数据库头部的偏移量 40 处的模式版本整数值。

每当模式更改时，SQLite 会自动增加模式版本。每个 SQL 语句运行时都会检查模式版本，以确保与 SQL 语句 准备 时的模式没有变化。通过使用 "PRAGMA schema_version=N" 来改变 schema_version 的值来破坏此机制，可能导致 SQL 语句使用过时的模式运行，这会导致错误的结果和/或 数据库损坏。读取 schema_version 总是安全的，但改变 schema_version 的值可能会引发问题。因此，当为数据库连接启用 防御模式 时，试图改变 schema_version 值会默默失败。

**警告：** 滥用此 pragma 可能导致 数据库损坏。

对于此 pragma，VACUUM 命令被视为模式更改，因为 VACUUM 通常会改变 sqlite_schema 表 中条目的 "rootpage" 值。

另请参阅 application_id pragma 和 user_version pragma。 <h _id="pragma_secure_delete" style="display:none">PRAGMA secure_delete</h>

* * *

**PRAGMA** *schema.***secure_delete;

PRAGMA** *schema.***secure_delete =** *boolean*|**FAST**

查询或更改安全删除设置。当 `secure_delete` 开启时，SQLite 会用零覆盖已删除的内容。安全删除的默认设置由 SQLITE_SECURE_DELETE 编译时选项决定，默认情况下是关闭的。关闭安全删除设置可以通过减少 CPU 循环和磁盘 I/O 的次数来提升性能。希望在删除或更新内容后避免留下法证痕迹的应用程序应在执行删除或更新操作之前启用安全删除 pragma，或者在执行删除或更新操作后运行 VACUUM。

"fast" 设置安全删除（大约在 2017-08-01 添加）是介于 "on" 和 "off" 之间的中间设置。当安全删除设置为 "fast" 时，SQLite 只有在不增加 I/O 的情况下才会用零覆盖已删除的内容。换句话说，"fast" 设置会使用更多的 CPU 循环，但不会增加更多的 I/O。这将从 b-tree 页面 中清除所有旧内容，但在 freelist 页面 上留下法证痕迹。

当存在 附加数据库 且未指定任何数据库时，所有数据库的安全删除设置将被更改。新附加数据库的安全删除设置是在评估 ATTACH 命令时主数据库的设置。

当多个数据库连接共享同一缓存时，更改一个数据库连接上的安全删除标志会影响所有连接。

**限制：** secure_delete pragma 仅导致从普通表中删除的内容被擦除。如果虚拟表在影子表中存储内容，则从虚拟表中删除内容并不一定会从影子表中完全移除法庭取证痕迹。特别地，与 SQLite 捆绑的 FTS3 和 FTS5 虚拟表即使启用 secure_delete pragma，仍可能在其影子表中留下法庭取证痕迹。

<h _id="pragma_short_column_names" style="display:none">PRAGMA short_column_names</h>

* * *

**PRAGMA short_column_names;**

PRAGMA short_column_names =** *boolean***;**

查询或更改短列名标志。此标志影响 SQLite 在 SELECT 语句返回的数据列命名方式。详细信息请参阅 full_column_names pragma。

**此 pragma 已被弃用**，仅供向后兼容性使用。新应用程序应避免使用此 pragma。旧应用程序应尽早停止使用此 pragma。当使用 SQLITE_OMIT_DEPRECATED 编译 SQLite 时，此 pragma 可能会被省略。

<h _id="pragma_shrink_memory" style="display:none">PRAGMA shrink_memory</h>

* * *

**PRAGMA shrink_memory**

此 pragma 会导致调用其上的数据库连接尽可能释放内存，通过调用 sqlite3_db_release_memory()实现。

<h _id="pragma_soft_heap_limit" style="display:none">PRAGMA soft_heap_limit</h>

* * *

**PRAGMA soft_heap_limit**

PRAGMA soft_heap_limit=***N***

此 pragma 通过指定的非负整数 N 调用 sqlite3_soft_heap_limit64()接口。如果指定了 N，则 soft_heap_limit pragma 总是返回与 sqlite3_soft_heap_limit64(-1) C 语言函数相同的整数。

另请参阅 hard_heap_limit pragma。<h _id="pragma_stats" style="display:none">PRAGMA stats</h>

* * *

**PRAGMA stats;**

此 pragma 返回关于表和索引的辅助信息。返回的信息在测试期间用于帮助验证查询规划器的正确操作。此 pragma 的格式和含义可能会随后的版本发生变化。由于其易变性，故意不记录此 pragma 的行为和输出格式。

此 pragma 的预期使用仅限于测试和验证 SQLite。此 pragma 可能会在无通知的情况下更改，不建议应用程序使用。

<h _id="pragma_synchronous" style="display:none">PRAGMA synchronous</h>

* * *

**PRAGMA** *schema.***synchronous;**

PRAGMA** *schema.***synchronous =** *0 | OFF | 1 | NORMAL | 2 | FULL | 3 | EXTRA***;**

查询或更改"synchronous"标志的设置。第一个（查询）形式将返回同步设置为整数。第二个形式更改同步设置。各种同步设置的含义如下：

**EXTRA** (3)

EXTRA 同步与 FULL 类似，但额外增加了在 DELETE 模式下提交事务后，同步包含回滚日志的目录。如果提交后紧随电源故障，EXTRA 可以提供额外的持久性。

**FULL** (2)

当同步为 FULL（2）时，SQLite 数据库引擎将使用 VFS 的 xSync 方法确保所有内容在继续之前都已安全写入磁盘表面。这确保操作系统崩溃或电源故障不会损坏数据库。FULL 同步非常安全，但速度较慢。当不处于 WAL 模式时，FULL 是最常用的同步设置。

**NORMAL** (1)

当同步为 NORMAL（1）时，SQLite 数据库引擎仍会在最关键的时刻进行同步，但频率比 FULL 模式低。在旧文件系统上，在 journal_mode=DELETE 模式下，恰好在错误的时间断电可能会导致数据库损坏，虽然概率非常小（但不为零）。在现代文件系统上，使用 synchronous=NORMAL 模式可能会安全，WAL 模式使用 synchronous=NORMAL 模式始终一致，但 WAL 模式会失去持久性。使用 synchronous=NORMAL 模式提交的事务可能在断电或系统崩溃后回滚。无论同步设置或日志模式如何，事务在应用程序崩溃时都是持久的。对于大多数运行在 WAL 模式下的应用程序来说，synchronous=NORMAL 设置是一个不错的选择。

**OFF** (0)

当同步为 OFF（0）时，SQLite 在将数据交给操作系统后立即继续运行而无需同步。如果运行 SQLite 的应用程序崩溃，数据将是安全的，但如果操作系统崩溃或计算机在数据写入磁盘表面之前失电，数据库可能会损坏。另一方面，同步为 OFF 时，提交操作可以快上数个数量级。

在 WAL 模式下，当同步设置为 NORMAL（1）时，在每个 checkpoint 之前同步 WAL 文件，每个完成的 checkpoint 后同步数据库文件，并在检查点后开始重用 WAL 文件时同步 WAL 文件头部，但在大多数事务中不进行同步操作。在 WAL 模式下设置为 FULL 时，在每个事务提交后额外同步 WAL 文件。在每个事务后的额外 WAL 同步有助于确保事务在断电后仍然可靠。使用 synchronous=FULL 或者 synchronous=NORMAL 均可以确保事务一致性，但如果耐久性不是问题，则通常情况下在 WAL 模式下仅需要 synchronous=NORMAL。

TEMP 模式始终具有 synchronous=OFF，因为 TEMP 的内容是短暂的，不会预期在断电后保留。尝试更改 TEMP 的同步设置会被静默忽略。

另请参阅 fullfsync 和 checkpoint_fullfsync pragma。

<h _id="pragma_table_info" style="display:none">PRAGMA table_info</h>

* * *

**PRAGMA** *schema.***table_info(***table-name***);**

此 **pragma** 为指定表中的每个普通列返回一行。结果集中的列包括："name"（列名）；"type"（数据类型，如果有的话，否则为空）；"notnull"（列是否可为 NULL）；"dflt_value"（列的默认值）；和 "pk"（对于非主键列为零，对于主键列为基于 1 的索引）。

"cid"列不应被视为比"当前结果集中的排名"更多的意义。

在 table_info pragma 中指定的表也可以是视图。

此 **pragma** 不会显示关于生成列或隐藏列的信息。使用 PRAGMA table_xinfo 可以获取包含生成列和隐藏列在内的更完整的列列表。 <h _id="pragma_table_list" style="display:none">PRAGMA table_list</h>

* * *

**PRAGMA table_list;

PRAGMA** *schema.***table_list;

PRAGMA table_list(***table-name***);**

此 **pragma** 返回关于模式中表和视图的信息，每行输出一张表。table_list pragma 首次出现在 SQLite 版本 3.37.0（2021-11-27）中。在其初始版本中，table_list pragma 返回的列包括下面列出的列。未来版本的 SQLite 可能会添加额外的输出列。

1.  **schema**：表或视图所在的模式（例如"main"或"temp"）。

1.  **name**：表或视图的名称。

1.  **type**：对象的类型 - 可能是"table"、"view"、"shadow"（用于 shadow tables）或"virtual"（用于 virtual tables）。

1.  **ncol**：表中的列数，包括生成列和隐藏列。

1.  **wr**: 如果表是 WITHOUT ROWID 表则为 1，否则为 0。

1.  **strict**: 如果表是 STRICT 表则为 1，否则为 0。

1.  *未来版本可能会添加额外的列。*

默认行为是显示所有模式中的所有表。如果在指示之前出现*模式.*名称，则仅显示该模式中的表。如果提供了*表名*参数，则仅返回关于该表的信息。 <h _id="pragma_table_xinfo" style="display:none">PRAGMA table_xinfo</h>

* * *

**PRAGMA** *模式.***table_xinfo(***表名***);**

此指示返回指定表中每一列的一行，包括生成的列和隐藏的列。输出与 PRAGMA table_info 的输出相同，另外还有一个名为"hidden"的列，其值表示普通列（0）、动态或存储生成的列（2 或 3）或虚拟表中的隐藏列（1）。对于这些字段非零的行，是 PRAGMA table_info 中省略的行。 <h _id="pragma_temp_store" style="display:none">PRAGMA temp_store</h>

* * *

**PRAGMA temp_store;

PRAGMA temp_store =** *0 | 默认 | 1 | 文件 | 2 | 内存***;**

查询或更改"temp_store"参数的设置。当 temp_store 为 DEFAULT（0）时，使用编译时 C 预处理宏 SQLITE_TEMP_STORE 确定临时表和索引的存储位置。当 temp_store 为 MEMORY（2）时，临时表和索引保留在纯内存数据库中。当 temp_store 为 FILE（1）时，临时表和索引存储在文件中。当指定 FILE 时，可以使用 temp_store_directory 指示来指定包含临时文件的目录。更改 temp_store 设置时，所有现有的临时表、索引、触发器和视图将立即被删除。

可能由库的编译时 C 预处理符号 SQLITE_TEMP_STORE 覆盖此指示设置。以下表格总结了 SQLITE_TEMP_STORE 预处理宏和 temp_store 指示的交互：

> | SQLITE_TEMP_STORE | PRAGMA temp_store | 用于 TEMP 表和索引的存储 |
> | --- | --- | --- |
> | 0 | *任意* | 文件 |
> | 1 | 0 | 文件 |
> | 1 | 1 | 文件 |
> | 1 | 2 | 内存 |
> | 2 | 0 | 内存 |
> | 2 | 1 | 文件 |
> | 2 | 2 | 内存 |
> | 3 | *任意* | 内存 |

<h _id="pragma_temp_store_directory" style="display:none">PRAGMA temp_store_directory</h>

* * *

**PRAGMA temp_store_directory;

PRAGMA temp_store_directory = '***目录名***';**

查询或更改[sqlite3_temp_directory](https://sqlite.org/c3ref/temp_directory.html)全局变量的值，许多操作系统接口后端使用此变量确定存储[临时表](https://sqlite.org/inmemorydb.html#temp_db)和索引的位置。

更改 temp_store_directory 设置时，立即删除发出该命令的数据库连接中的所有现有临时表、索引、触发器和视图。在实践中，应在打开进程的第一个数据库连接后立即设置 temp_store_directory。如果在同一进程中的其他数据库连接打开时更改 temp_store_directory，则行为是未定义的，并且可能是不希望的。

更改 temp_store_directory 设置不是线程安全的。如果应用程序内的另一个线程同时运行任何 SQLite 接口，则永远不要更改 temp_store_directory 设置。这样做会导致未定义的行为。更改 temp_store_directory 设置会写入[sqlite3_temp_directory](https://sqlite.org/c3ref/temp_directory.html)全局变量，该全局变量未受到互斥锁保护。

值*directory-name*应该用单引号括起来。要将目录恢复为默认设置，请将*directory-name*设置为空字符串，例如，*PRAGMA temp_store_directory = ''*。如果找不到*directory-name*或无法写入，则会引发错误。

临时文件的默认目录取决于操作系统。某些操作系统接口可能选择忽略此变量，并将临时文件放置在与此处指定的目录不同的其他目录中。从这个意义上说，此命令只是建议性的。

**此命令已弃用**，仅为向后兼容性而存在。新应用程序应避免使用此命令。旧应用程序应尽快停止使用此命令。在使用[SQLITE_OMIT_DEPRECATED](https://sqlite.org/compile.html#omit_deprecated)编译时选项编译 SQLite 时，可能会从构建中省略此命令。

<h _id="pragma_threads" style="display:none">PRAGMA threads</h>

* * *

**PRAGMA threads;**

PRAGMA threads = **N**;

查询或更改当前数据库连接的[sqlite3_limit](https://sqlite.org/c3ref/limit.html)(db,[SQLITE_LIMIT_WORKER_THREADS](https://sqlite.org/c3ref/c_limit_attached.html#sqlitelimitworkerthreads),...)限制的值。此限制设置了辅助线程数量的上限，这些线程允许在查询中启动以帮助处理预处理语句。默认限制为 0，除非使用[SQLITE_DEFAULT_WORKER_THREADS](https://sqlite.org/compile.html#default_worker_threads)编译时选项进行更改。当限制为零时，意味着不会启动任何辅助线程。

此命令是对[sqlite3_limit](https://sqlite.org/c3ref/limit.html)(db,[SQLITE_LIMIT_WORKER_THREADS](https://sqlite.org/c3ref/c_limit_attached.html#sqlitelimitworkerthreads),...)接口的简单包装。

<h _id="pragma_trusted_schema" style="display:none">PRAGMA trusted_schema</h>

* * *

**PRAGMA trusted_schema;**

PRAGMA trusted_schema =** *boolean***;**

trusted_schema 设置是一个每个连接的布尔值，用于确定未经安全审计的 SQL 函数和虚拟表是否允许由视图、触发器或模式表达式（如 CHECK 约束、DEFAULT 子句、生成列、表达式索引和/或部分索引）运行。还可以使用 C 语言接口(db，SQLITE_DBCONFIG_TRUSTED_SCHEMA,...)控制此设置。

为了保持向后兼容性，默认情况下此设置为 ON。关闭它有利有弊，大多数应用程序关闭后不受影响。因此，建议所有应用程序在打开数据库连接时立即关闭此设置。

编译时选项-DSQLITE_TRUSTED_SCHEMA=0 将导致此设置默认为 OFF。 <h _id="pragma_user_version" style="display:none">PRAGMA user_version</h>

* * *

**PRAGMA** *schema.***user_version;

PRAGMA** *schema.***user_version =** *integer* **;**

user_version pragma 将获取或设置数据库头部[fileformat2.html#database_header]中偏移量 60 处的用户版本整数的值。用户版本是一个整数，应用程序可以根据需要自行使用。SQLite 本身不使用用户版本。

另请参阅 application_id pragma 和 schema_version pragma。 <h _id="pragma_vdbe_addoptrace" style="display:none">PRAGMA vdbe_addoptrace</h>

* * *

**PRAGMA vdbe_addoptrace =** *boolean***;**

如果 SQLite 使用 SQLITE_DEBUG 编译时选项编译，则 vdbe_addoptrace pragma 可用于在代码生成期间创建时显示完整的 VDBE 操作码。此功能用于调试 SQLite 本身。有关更多信息，请参阅 VDBE 文档。

这个 pragma 用于在调试 SQLite 本身时使用。只有在使用 SQLITE_DEBUG 编译时选项时才可用。

<h _id="pragma_vdbe_debug" style="display:none">PRAGMA vdbe_debug</h>

* * *

**PRAGMA vdbe_debug =** *boolean***;**

如果 SQLite 使用 SQLITE_DEBUG 编译时选项编译，则 vdbe_debug pragma 是三个仅供调试使用的快捷 pragma 的简写：vdbe_addoptrace、vdbe_listing 和 vdbe_trace。此功能用于调试 SQLite 本身。有关更多信息，请参阅 VDBE 文档。

这个 pragma 用于在调试 SQLite 本身时使用。只有在使用 SQLITE_DEBUG 编译时选项时才可用。

<h _id="pragma_vdbe_listing" style="display:none">PRAGMA vdbe_listing</h>

* * *

**PRAGMA vdbe_listing =** *boolean***;**

如果 SQLite 使用 SQLITE_DEBUG 编译选项编译，那么 vdbe_listing pragma 可以用来在每个语句执行时在标准输出中显示虚拟机操作码的完整列表。打开列表后，在开始执行之前会打印出整个程序的内容。此功能用于调试 SQLite 本身。更多信息请参阅 VDBE 文档。

这个 pragma 仅用于调试 SQLite 本身。只有在使用 SQLITE_DEBUG 编译选项时才可用。

<h _id="pragma_vdbe_trace" style="display:none">PRAGMA vdbe_trace</h>

* * *

**PRAGMA vdbe_trace =** *boolean***;**

如果 SQLite 使用 SQLITE_DEBUG 编译选项编译，那么 vdbe_trace pragma 可以用来在标准输出中打印虚拟机操作码的执行过程。这个特性用于调试 SQLite。更多信息请参阅 VDBE 文档。

这个 pragma 仅用于调试 SQLite 本身。只有在使用 SQLITE_DEBUG 编译选项时才可用。

<h _id="pragma_wal_autocheckpoint" style="display:none">PRAGMA wal_autocheckpoint</h>

* * *

**PRAGMA wal_autocheckpoint;

PRAGMA wal_autocheckpoint=***N***;**

这个 pragma 用于查询或设置预写式日志的自动 checkpoint 间隔。当启用预写式日志（通过 journal_mode pragma）时，每当预写式日志长度等于或超过*N*页时，就会自动运行一个 checkpoint。将自动 checkpoint 的大小设置为零或负数会关闭自动 checkpoint 功能。

这个 pragma 是 sqlite3_wal_autocheckpoint() C 接口的包装器。所有自动 checkpoint 都是 PASSIVE 的。

默认情况下，自动 checkpoint 的间隔为 1000 或 SQLITE_DEFAULT_WAL_AUTOCHECKPOINT。

<h _id="pragma_wal_checkpoint" style="display:none">PRAGMA wal_checkpoint</h>

* * *

**PRAGMA** *schema.***wal_checkpoint;**

**PRAGMA** *schema.***wal_checkpoint(PASSIVE);**

**PRAGMA** *schema.***wal_checkpoint(FULL);**

**PRAGMA** *schema.***wal_checkpoint(RESTART);**

**PRAGMA** *schema.***wal_checkpoint(TRUNCATE);**

如果启用了预写式日志（通过 journal_mode pragma），这个 pragma 会导致对数据库*database*进行 checkpoint 操作，如果省略了*database*，则会对所有附加数据库进行操作。如果禁用了预写式日志模式，这个 pragma 则是一个无害的空操作。

调用此编译指示而不带参数相当于调用 C 接口 sqlite3_wal_checkpoint()。

使用参数调用此编译指示相当于调用 C 接口 sqlite3_wal_checkpoint_v2()，并使用第三个参数对应于 3rd parameter：

被动

尽可能检查多个帧而不等待任何数据库读取器或写入器完成。如果日志中的所有帧都已检查点，则同步数据库文件。此模式与调用 C 接口 sqlite3_wal_checkpoint()相同。在此模式下，永远不会调用忙处理程序回调。

完全

此模式阻止（调用忙处理程序回调）直到没有数据库写入器和所有读取器从最新数据库快照中读取为止。然后，它检查点日志文件中的所有帧并同步数据库文件。完全在运行时阻止并发写入，但读取器可以继续进行。

重新开始

此模式的工作方式与完全相同，但在检查点日志文件后，它会阻塞（调用忙处理程序回调），直到所有读取器完成对日志文件的操作。这确保了下一个写入数据库文件的客户端从头重新开始日志文件。重新开始在运行时阻止并发写入，但允许读取器继续进行。

截断

此模式的工作方式与重新开始相同，但在成功完成后，WAL 文件将被截断为零字节。

wal_checkpoint 编译指示返回一个具有三个整数列的单行。第一列通常为 0，但如果由于其他线程或进程正在活动使用数据库而阻止完成，则为 1，例如，如果等效调用 sqlite3_wal_checkpoint_v2()将返回 SQLITE_OK 或 1 如果等效调用将返回 SQLITE_BUSY。第二列是已写入写前日志文件的修改页面数。第三列是在检查点结束时已成功移回数据库文件的写前日志文件中的页面数。如果没有写前日志，则第二列和第三列为-1，例如，如果在不在 WAL 模式的数据库连接上调用此编译指示。

<h _id="pragma_writable_schema" style="display:none">PRAGMA writable_schema</h>

* * *

**PRAGMA writable_schema =** *boolean***;**

**PRAGMA writable_schema = RESET**

当此编译指示开启，并且 SQLITE_DBCONFIG_DEFENSIVE 标志关闭时，sqlite_schema 表可以使用普通的 UPDATE、INSERT 和 DELETE 语句进行更改。如果参数为"RESET"，则禁用模式写入（类似于"PRAGMA writable_schema=OFF"），并且重新加载模式。**警告：**滥用此编译指示很容易导致数据库文件损坏。

* * *
