# 1\. 概述

> 原文：[`sqlite.com/lang_corefunc.html`](https://sqlite.com/lang_corefunc.html)

下面列出的核心函数默认可用。日期和时间函数、聚合函数、窗口函数、数学函数和 JSON 函数有单独的文档。应用程序可以定义使用 sqlite3_create_function() API 添加到数据库引擎的其他以 C 编写的函数。

**simple-function-invocation:**

<svg class="pikchr" viewBox="0 0 414.49 126.792"><text x="86" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">simple-func</text> <text x="179" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">(</text> <text x="273" y="55" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">表达式</text> <text x="366" y="55" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">)</text> <text x="273" y="17" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">,</text> <text x="273" y="109" text-anchor="middle" font-weight="bold" fill="rgb(0,0,0)" dominant-baseline="central">*</text></svg>

详见表达式中的函数文档，了解更多关于 SQL 函数调用如何适用于 SQL 表达式的信息。

# 2\. 核心函数列表

+   abs(X)

+   changes()

+   char(X1,X2,...,XN)

+   coalesce(X,Y,...)

+   concat(X,...)

+   concat_ws(SEP,X,...)

+   format(FORMAT,...)

+   glob(X,Y)

+   hex(X)

+   ifnull(X,Y)

+   iif(X,Y,Z)

+   instr(X,Y)

+   last_insert_rowid()

+   length(X)

+   like(X,Y)

+   like(X,Y,Z)

+   likelihood(X,Y)

+   likely(X)

+   load_extension(X)

+   load_extension(X,Y)

+   lower(X)

+   ltrim(X)

+   ltrim(X,Y)

+   max(X,Y,...)

+   min(X,Y,...)

+   nullif(X,Y)

+   octet_length(X)

+   printf(FORMAT,...)

+   quote(X)

+   random()

+   randomblob(N)

+   replace(X,Y,Z)

+   round(X)

+   round(X,Y)

+   rtrim(X)

+   rtrim(X,Y)

+   sign(X)

+   soundex(X)

+   sqlite_compileoption_get(N)

+   sqlite_compileoption_used(X)

+   sqlite_offset(X)

+   sqlite_source_id()

+   sqlite_version()

+   substr(X,Y)

+   substr(X,Y,Z)

+   substring(X,Y)

+   substring(X,Y,Z)

+   total_changes()

+   trim(X)

+   trim(X,Y)

+   typeof(X)

+   unhex(X)

+   unhex(X,Y)

+   unicode(X)

+   unlikely(X)

+   upper(X)

+   zeroblob(N)

# 3\. 内置标量 SQL 函数的描述

**abs(*X*)**

`abs(X)` 函数返回数值参数 X 的绝对值。如果 X 为 NULL，则 `abs(X)` 返回 NULL。如果 X 是无法转换为数值的字符串或 blob，则 `abs(X)` 返回 0.0。如果 X 是整数 -9223372036854775808，则 `abs(X)` 抛出整数溢出错误，因为没有等效的正 64 位二进制补码值。

**changes()**

`changes()` 函数返回最近完成的 INSERT、DELETE 或 UPDATE 语句更改或插入或删除的数据库行数，不包括较低级触发器中的语句。`changes()` SQL 函数是对 sqlite3_changes64() C/C++ 函数的封装，因此遵循相同的更改计数规则。

**char(*X1*,*X2*,...,*XN*)**

`char(X1,X2,...,XN)` 函数返回由整数 X1 到 XN 的 Unicode 代码点值组成的字符串。

**coalesce(*X*,*Y*,...)**

`coalesce()` 函数返回其第一个非 NULL 参数的副本，如果所有参数都为 NULL，则返回 NULL。`coalesce()` 必须至少有 2 个参数。

**concat(*X*,...)**

`concat(...)` 函数返回一个字符串，该字符串是其所有非 NULL 参数的字符串表示的串联。如果所有参数都为 NULL，则 `concat()` 返回空字符串。

**concat_ws(*SEP*,*X*,...)**

`concat_ws(SEP,...)` 函数返回一个字符串，它是除第一个参数外所有非空参数的串联，使用第一个参数的文本值作为分隔符。如果第一个参数为 NULL，则 `concat_ws()` 返回 NULL。如果除第一个参数外的所有参数都为 NULL，则 `concat_ws()` 返回空字符串。

**format(*FORMAT*,...)**

SQL 函数 format(FORMAT,...) 的工作方式类似于 sqlite3_mprintf() C 语言函数和标准 C 库中的 printf() 函数。第一个参数是格式字符串，指定如何使用后续参数的值构造输出字符串。如果缺少或为 NULL 的 FORMAT 参数，则结果为 NULL。格式 %n 被静默忽略且不消耗参数。格式 %p 是 %X 的别名。格式 %z 可以与 %s 互换使用。如果参数列表中的参数不足，则假定缺少的参数具有 NULL 值，这将被转换为数值格式的 0 或 0.0，或者字符串格式的空字符串 %s。有关更多信息，请参阅 内建 printf() 文档。

**glob(*X*,*Y*)**

函数 **glob(X,Y)** 等价于表达式 "**Y GLOB X**"。请注意，在 glob() 函数中，X 和 Y 参数与中缀 GLOB 运算符相对位置相反。其中，Y 是字符串，X 是模式。因此，例如，以下表达式是等价的：

> ```sql
>      name GLOB '*helium*'
>      glob('*helium*',name)
>   
> ```

如果使用 sqlite3_create_function() 接口来覆盖 glob(X,Y) 函数的默认实现，则 GLOB 操作符将调用该替代实现。

**hex(*X*)**

函数 hex() 将其参数解释为 BLOB，并返回该 BLOB 内容的大写十六进制表示的字符串。

如果 "hex(*X*)" 中的参数 *X* 是整数或浮点数，则 "interprets its argument as a BLOB" 意味着二进制数首先转换为 UTF8 文本表示，然后该文本被解释为 BLOB。因此，"hex(12345678)" 渲染为 "3132333435363738" 而不是整数值 "0000000000BC614E" 的二进制表示。

另请参见：unhex()

**ifnull(*X*,*Y*)**

函数 ifnull() 返回其第一个非 NULL 参数的副本，如果两个参数均为 NULL 则返回 NULL。ifnull() 必须有精确的 2 个参数。函数 ifnull() 等效于 coalesce() 的两个参数形式。

**iif(*X*,*Y*,*Z*)**

函数 iif(X,Y,Z) 如果 X 为真则返回值 Y，否则返回值 Z。函数 iif(X,Y,Z) 在逻辑上等效于并生成与 CASE 表达式 "CASE WHEN X THEN Y ELSE Z END" 相同的 字节码。

**instr(*X*,*Y*)**

函数 instr(X,Y) 在字符串 X 中查找字符串 Y 的第一个出现位置，并返回前导字符数加 1，如果 Y 在 X 中找不到则返回 0。或者，如果 X 和 Y 都是 BLOB，则 instr(X,Y) 返回第一个 Y 出现位置之前的字节数加 1，如果 Y 在 X 中找不到则返回 0。如果 instr(X,Y) 的参数 X 和 Y 都非 NULL 且不是 BLOB，则都被解释为字符串。如果 instr(X,Y) 中的 X 或 Y 为 NULL，则结果为 NULL。

**last_insert_rowid()**

last_insert_rowid() 函数返回从调用函数的数据库连接中插入的最后一行的 ROWID。last_insert_rowid() SQL 函数是 sqlite3_last_insert_rowid() C/C++ 接口函数的包装器。

**length(*X*)**

对于字符串值 X，length(X) 函数返回在第一个 NUL 字符之前的字符数（而非字节）。由于 SQLite 字符串通常不包含 NUL 字符，length(X) 函数通常返回字符串 X 中的字符总数。对于 blob 值 X，length(X) 返回 blob 中的字节数。如果 X 为 NULL，则 length(X) 也为 NULL。如果 X 是数字，则 length(X) 返回 X 的字符串表示的长度。

注意，对于字符串，length(X) 函数返回字符串的 *字符* 长度，而不是字节长度。字符长度是字符串中的字符数。对于 UTF-16 字符串，字符长度始终不同于字节长度，并且对于 UTF-8 字符串，如果字符串包含多字节字符，则字符长度可能与字节长度不同。使用 octet_length() 函数查找字符串的字节长度。

对于 BLOB 值，length(X) 总是返回 BLOB 的字节长度。

对于字符串值，length(X) 必须读取整个字符串到内存中以计算字符长度。但是对于 BLOB 值，由于 SQLite 知道 BLOB 中有多少字节，因此这是不必要的。因此，对于多兆字节值，length(X) 函数通常对于 BLOB 来说比对字符串更快，因为它不需要将值加载到内存中。

**like(*X*,*Y*)

like(*X*,*Y*,*Z*)**

like() 函数用于实现 "**Y LIKE X [ESCAPE Z]**" 表达式。如果存在可选的 ESCAPE 子句，则 like() 函数将使用三个参数调用。否则，只使用两个参数调用。注意，相对于中缀 LIKE 运算符，like() 函数中的 X 和 Y 参数是颠倒的。X 是模式，Y 是要与该模式匹配的字符串。因此，以下表达式是等效的：

> ```sql
>      name LIKE '%neon%'
>      like('%neon%',name)
>   
> ```

可以使用 sqlite3_create_function() 接口来重写 like() 函数，从而改变 LIKE 运算符的操作方式。在重写 like() 函数时，重写两个参数版本和三个参数版本的 like() 函数可能很重要。否则，根据是否指定了 ESCAPE 子句，将调用不同的代码来实现 LIKE 运算符。

**likelihood(*X*,*Y*)**

likelihood(X,Y) 函数返回参数 X 本身。likelihood(X,Y) 中的 Y 必须是介于 0.0 和 1.0 之间的浮点常数。likelihood(X) 函数是一个无操作函数，代码生成器会优化掉它，因此在运行时（即在调用 sqlite3_step() 时）不会消耗 CPU 周期。likelihood(X,Y) 函数的目的是为了向查询优化器提供提示，说明参数 X 是一个概率大约为 Y 的布尔值。unlikely(X) 函数是 likelihood(X,0.0625) 的简写。likely(X) 函数是 likelihood(X,0.9375) 的简写。

**likely(*X*)**

likely(X) 函数返回参数 X 本身。likely(X) 函数是一个无操作函数，代码生成器会优化掉它，因此在运行时（即在调用 sqlite3_step() 时）不会消耗 CPU 周期。likely(X) 函数的目的是为了向查询优化器提供提示，说明参数 X 是一个通常为真的布尔值。likely(X) 函数等效于 likelihood(X,0.9375)。另请参见：unlikely(X)。

**load_extension(X)**

load_extension(*X*,*Y*)**

load_extension(X,Y) 函数加载名为 X 的共享库文件中的 SQLite 扩展，使用入口点 Y。load_extension() 的结果始终为 NULL。如果省略 Y，则使用默认的入口点名称。如果扩展加载或初始化失败，load_extension() 函数会引发异常。

如果扩展尝试修改或删除 SQL 函数或排序序列，则 load_extension() 函数将失败。扩展可以添加新的函数或排序序列，但不能修改或删除当前运行的 SQL 语句中的现有函数或排序序列，因为这些函数和/或排序序列可能在其他地方使用。要加载更改或删除函数或排序序列的扩展，请使用 sqlite3_load_extension() C 语言 API。

由于安全原因，默认情况下禁用扩展加载，并且必须通过先前调用 sqlite3_enable_load_extension() 来启用。

**lower(X)**

lower(X) 函数返回字符串 X 的副本，其中所有 ASCII 字符转换为小写。默认的内置 lower() 函数仅适用于 ASCII 字符。要对非 ASCII 字符进行大小写转换，请加载 ICU 扩展。

**ltrim(X)**

ltrim(X,Y)**

ltrim(X,Y) 函数返回一个字符串，该字符串由从 X 的左侧删除所有出现在 Y 中的字符形成。如果省略 Y 参数，则 ltrim(X) 会从 X 的左侧删除空格。

**max(*X*,*Y*,...)**

多参数的 max()函数返回具有最大值的参数，如果任何参数为 NULL 则返回 NULL。多参数的 max()函数从左到右搜索其参数，以找到定义排序函数的参数，并对所有字符串比较使用该排序函数。如果 max()的参数中没有一个定义排序函数，则使用 BINARY 排序函数。请注意，**max()**在有 2 个或更多参数时是一个简单的函数，但如果只有一个参数，则作为聚合函数运行。

**min(*X*,*Y*,...)**

多参数的 min()函数返回具有最小值的参数。多参数的 min()函数从左到右搜索其参数，以找到定义排序函数的参数，并对所有字符串比较使用该排序函数。如果 min()的参数中没有一个定义排序函数，则使用 BINARY 排序函数。请注意，**min()**在有 2 个或更多参数时是一个简单的函数，但如果只有一个参数，则作为聚合函数运行。

**nullif(*X*,*Y*)**

nullif(X,Y)函数如果参数不同则返回其第一个参数，如果参数相同则返回 NULL。nullif(X,Y)函数从左到右搜索其参数，以找到定义排序函数的参数，并对所有字符串比较使用该排序函数。如果 nullif()的两个参数都不定义排序函数，则使用 BINARY 排序函数。

**octet_length(*X*)**

octet_length(X)函数返回文本字符串 X 编码中的字节数。如果 X 为 NULL，则 octet_length(X)返回 NULL。如果 X 为 BLOB 值，则 octet_length(X)与 length(X)相同。如果 X 为数值，则 octet_length(X)返回该数字的文本呈现的字节数。

因为 octet_length(X)返回 X 中的字节数而不是字符数，所以返回的值取决于数据库编码。如果数据库编码为 UTF16 而不是 UTF8，则 octet_length()函数对于相同的输入字符串可能返回不同的答案。

如果参数 X 是表列，并且值为文本或 blob 类型，则 octet_length(X)避免从磁盘读取 X 的内容，因为可以从元数据计算出字节长度。因此，即使 X 是包含多兆字节文本或 blob 值的列，octet_length(X)也是高效的。

**printf(*FORMAT*,...)**

printf() SQL 函数是 format() SQL 函数的别名。format() SQL 函数最初命名为 printf()，但后来为了与其他数据库引擎兼容而更改为 format()。保留 printf()名称作为别名，以免破坏旧代码。

**quote(*X*)**

函数 `quote(X)` 返回一个 SQL 文字的文本，它是其参数的值，适合包含到 SQL 语句中。字符串由单引号包围，内部引号根据需要进行转义。BLOBs 被编码为十六进制文字。包含嵌入 NUL 字符的字符串不能作为 SQL 字符串字面量表示，因此返回的字符串字面量在第一个 NUL 之前被截断。

**random()**

函数 `random()` 返回一个介于 -9223372036854775808 到 +9223372036854775807 之间的伪随机整数。

**randomblob(*N*)**

函数 `randomblob(N)` 返回一个包含 N 字节伪随机字节的 blob。如果 N 小于 1，则返回一个 1 字节的随机 blob。

提示：应用程序可以使用此函数与 hex() 和/或 lower() 生成全局唯一标识符，例如：

> `hex(randomblob(16))`
> 
> `lower(hex(randomblob(16)))`

**replace(*X*,*Y*,*Z*)**

函数 `replace(X,Y,Z)` 返回一个字符串，通过将字符串 X 中每个字符串 Y 的出现替换为字符串 Z 形成。用于比较的是 BINARY 排序序列。如果 Y 是空字符串，则返回不变的 X。如果 Z 最初不是字符串，则在处理之前将其强制转换为 UTF-8 字符串。

**round(*X*)**

round(*X*,*Y*)**

函数 `round(X,Y)` 返回将 X 四舍五入到小数点右侧 Y 位的浮点值。如果省略或为负的 Y 参数，则被视为 0。

**rtrim(*X*)**

rtrim(*X*,*Y*)**

函数 `rtrim(X,Y)` 返回一个字符串，通过从 X 的右侧移除在 Y 中出现的任何字符形成。如果省略 Y 参数，则 rtrim(X) 从 X 的右侧移除空格。

**sign(*X*)**

函数 `sign(X)` 如果参数 X 是负数则返回 -1，零则返回 0，正数则返回 +1。如果给 sign(X) 的参数为 NULL 或者是无法无损转换为数字的字符串或 blob，则 sign(X) 返回 NULL。

**soundex(*X*)**

函数 `soundex(X)` 返回字符串 X 的 soundex 编码的字符串。如果参数为 NULL 或不包含 ASCII 字母字符，则返回字符串 "?000"。默认情况下 SQLite 不包括此函数。只有在构建 SQLite 时使用了 SQLITE_SOUNDEX 编译选项时，此函数才可用。

**sqlite_compileoption_get(*N*)**

SQL 函数 `sqlite_compileoption_get()` 是 sqlite3_compileoption_get() C/C++ 函数的包装器。此例程返回用于构建 SQLite 的第 N 个编译时选项，如果 N 超出范围则返回 NULL。另请参阅 compile_options pragma。

**sqlite_compileoption_used(*X*)**

sqlite_compileoption_used() SQL 函数是 sqlite3_compileoption_used() C/C++ 函数的封装。当参数 X 是编译时选项的名称的字符串时，此例程根据是否在构建过程中使用该选项返回 true（1）或 false（0）。

**sqlite_offset(*X*)**

sqlite_offset(X) 函数返回从数据库文件中读取值的记录开始处的字节偏移量。如果 X 不是普通表中的列，则 sqlite_offset(X) 返回 NULL。sqlite_offset(X) 返回的值可能引用原始表或索引，具体取决于查询。如果值 X 通常从索引中提取，则 sqlite_offset(X) 返回相应索引记录的偏移量。如果值 X 应从原始表中提取，则 sqlite_offset(X) 返回表记录的偏移量。

如果 SQLite 使用了 -DSQLITE_ENABLE_OFFSET_SQL_FUNC 编译时选项，则 sqlite_offset(X) SQL 函数可用。

**sqlite_source_id()**

sqlite_source_id() 函数返回一个字符串，用于标识用于构建 SQLite 库的特定版本的源代码。sqlite_source_id() 返回的字符串是源代码被检入的日期和时间，后跟该检入的 SHA3-256 哈希值。此函数是 sqlite3_sourceid() C 接口的 SQL 封装。

**sqlite_version()**

sqlite_version() 函数返回正在运行的 SQLite 库的版本字符串。此函数是 sqlite3_libversion() C 接口的 SQL 封装。

**substr(*X*,*Y*,*Z*)**

substr(*X*,*Y*)

substring(*X*,*Y*,*Z*)

substring(*X*,*Y*)**

substr(X,Y,Z) 函数返回输入字符串 X 的子串，从第 Y 个字符开始，长度为 Z 个字符。如果省略 Z，则 substr(X,Y) 返回从第 Y 个字符开始到字符串 X 结尾的所有字符。X 最左边的字符为编号 1。如果 Y 为负数，则子串的第一个字符从右边开始计数而不是从左边开始。如果 Z 为负数，则返回前 abs(Z) 个字符，这些字符位于第 Y 个字符之前。如果 X 是字符串，则字符索引引用实际的 UTF-8 字符。如果 X 是 BLOB，则索引引用字节。

"substring()" 是从 SQLite 版本 3.34 开始的 "substr()" 的别名。

**total_changes()**

total_changes() 函数返回自打开当前数据库连接以来由 INSERT、UPDATE 或 DELETE 语句引起的行更改次数。此函数是 sqlite3_total_changes64() C/C++ 接口的封装。

**trim(*X*)**

trim(*X*,*Y*)**

trim(X,Y) 函数返回一个字符串，通过从 X 的两端删除 Y 中出现的所有字符形成。如果省略 Y 参数，则 trim(X) 从 X 的两端删除空格。

**typeof(*X*)**

typeof(X) 函数返回一个字符串，指示表达式 X 的数据类型："null"、"integer"、"real"、"text" 或 "blob"。

**unhex(*X*)**

**unhex(*X*,*Y*)**

unhex(X,Y) 函数返回一个 BLOB 值，它是十六进制字符串 X 的解码。如果 X 包含任何非十六进制数字且不在 Y 中，则 unhex(X,Y) 返回 NULL。如果省略 Y，则理解为空字符串，因此 X 必须是一个纯十六进制字符串。X 中的所有十六进制数字必须成对出现，每对的两个数字紧邻相邻，否则 unhex(X,Y) 返回 NULL。如果参数 X 或 Y 是 NULL，则 unhex(X,Y) 返回 NULL。输入 X 可以包含任意混合大小写的十六进制数字。Y 中的十六进制数字对 X 的翻译没有影响。Y 中不是十六进制数字的字符在 X 中被忽略。

参见：hex()

**unicode(*X*)**

unicode(X) 函数返回与字符串 X 的第一个字符对应的数值 unicode 代码点。如果 unicode(X) 的参数不是字符串，则结果是未定义的。

**unlikely(*X*)**

unlikely(X) 函数返回参数 X 本身。unlikely(X) 函数是一个无操作函数，在代码生成器中进行优化，以便在运行时不消耗 CPU 周期（即在调用 sqlite3_step()时）。unlikely(X) 函数的目的是向查询规划器提供一个提示，表明参数 X 是一个通常为假的布尔值。unlikely(X) 函数等效于 likelihood(X, 0.0625)。

**upper(*X*)**

upper(X) 函数返回输入字符串 X 的副本，其中所有小写 ASCII 字符转换为它们的大写等效字符。

**zeroblob(*N*)**

zeroblob(N) 函数返回一个由 N 字节 0x00 组成的 BLOB。SQLite 非常有效地管理这些 zeroblob。Zeroblob 可用于预留一个 BLOB 的空间，稍后使用增量 BLOB I/O 进行写入。这个 SQL 函数是使用 C/C++ 接口中的 sqlite3_result_zeroblob()例程实现的。
