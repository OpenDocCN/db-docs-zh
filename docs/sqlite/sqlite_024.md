# SQLite 库的 Tcl 接口

> 原文：[`sqlite.com/tclsqlite.html`](https://sqlite.com/tclsqlite.html)

SQLite 库被设计为从[Tcl 或 Tcl/Tk](http://www.tcl-lang.org)脚本非常容易使用。 SQLite 起初是作为[Tcl 扩展](http://www.tcl-lang.org/doc/tea/)，并且其主要测试套件是用 TCL 编写的。 SQLite 可以与任何编程语言一起使用，但它与 TCL 的连接非常深入。

本文档概述了 SQLite 的 Tcl 编程接口。

### API

SQLite 库的接口由一个名为**sqlite3**的单个 tcl 命令组成，因为只有这一个命令，所以接口没有放置在单独的命名空间中。

**sqlite3**命令主要用于以下方式来打开或创建数据库：

> **sqlite3**  *dbcmd  ?database-name?  ?options?*

要仅获取信息，**sqlite3**命令可以提供恰好一个参数，即"-version"、"-sourceid"或"-has-codec"，这将返回指定的数据，而没有其他效果。

有其他参数时，**sqlite3**命令会打开第二个非选项参数中指定的数据库，如果没有则创建一个名为""的数据库。如果打开成功，将创建一个由第一个参数命名的新 Tcl 命令，并返回""。（这种方法类似于在 Tk 中创建小部件。）如果打开失败，则会引发错误而不创建 Tcl 命令，并返回一个错误消息字符串。

如果数据库不存在，默认行为是自动创建它（尽管可以通过使用 "**-create *false***" 选项更改这一行为）。

数据库的名称通常只是存储数据库的磁盘文件的名称。如果数据库的名称是特殊名称":memory:"，则在内存中创建一个新数据库。如果数据库的名称是空字符串，则在关闭数据库连接时自动删除空文件中创建数据库。如果在**sqlite3**命令上提供了"**-uri yes**"选项，则可以使用 URI 文件名。

**sqlite3**命令理解的选项包括：

> **-create** *BOOLEAN*
> 
> 如果为真，则如果不存在，则创建一个新数据库。如果为假，则尝试打开先前不存在的数据库文件会引发错误。默认行为为"true"。
> 
> **-nomutex** *BOOLEAN*
> 
> 如果为真，则禁用数据库连接的所有互斥体，这在单线程应用程序中提供了小幅性能提升。
> 
> **-readonly** *BOOLEAN*
> 
> 如果为真，则以只读模式打开数据库文件。如果为假，则如果文件系统权限允许，则以读写模式打开数据库，或者如果操作系统通过文件系统写入权限拒绝，则仅以只读模式打开。默认设置为"false"。请注意，如果先前用于数据库的进程未干净退出并留下热日志，则在打开后需要写权限来恢复数据库，并且数据库不能以只读方式打开。
> 
> **-uri** *BOOLEAN*
> 
> 如果为真，则将文件名参数解释为 URI 文件名。如果为假，则参数为字面文件名。默认值为"false"。
> 
> **-vfs** *VFSNAME*
> 
> 使用替代的 VFS 根据参数命名。
> 
> **-fullmutex** *BOOLEAN*
> 
> 如果为真，则多个线程可以安全地尝试使用数据库。如果为假，则此类尝试是不安全的。默认值取决于扩展构建方式。
> 
> **-nofollow** *BOOLEAN*
> 
> 如果为真，并且数据库名称是符号链接，则不会跟随以打开真实数据库文件。如果为假，则会跟随符号链接。默认为"false"。

一旦 SQLite 数据库打开，可以使用*dbcmd*的方法进行控制。当前定义了 40 种方法。

|

+   authorizer

+   backup

+   bind_fallback

+   busy

+   cache

+   changes

+   close

+   collate

+   collation_needed

+   commit_hook

+   complete

+   config

+   copy

+   deserialize

|

+   enable_load_extension

+   errorcode

+   eval

+   exists

+   function

+   incrblob

+   interrupt

+   last_insert_rowid

+   nullvalue

+   onecolumn

+   preupdate

+   profile

+   progress

+   restore

|

+   rollback_hook

+   serialize

+   status

+   timeout

+   total_changes

+   trace

+   trace_v2

+   transaction

+   unlock_notify

+   update_hook

+   version

+   wal_hook

|

将在续集中解释每个方法的使用，尽管不按照上述顺序。

### "eval"方法

最有用的*dbcmd*方法是"eval"。eval 方法用于在数据库上执行 SQL。eval 方法的语法如下：

> *dbcmd*  **eval**  ?**-withoutnulls**?  *sql*   ?*array-name*?  ?*script*?

eval 方法的作用是执行第二个参数中给出的 SQL 语句或语句。例如，要在数据库中创建一个新表，可以这样做：

> **sqlite3 db1 ./testdb
> 
> db1 eval {CREATE TABLE t1(a int, b text)}**

上述代码创建了一个名为**t1**，带有列**a**和**b**的新表。有什么比这更简单的呢？

查询结果作为列值列表返回。如果查询请求 2 列，并且有 3 行匹配查询，则返回的列表将包含 6 个元素。例如：

> **db1 eval {INSERT INTO t1 VALUES(1,'hello')}
> 
> db1 eval {INSERT INTO t1 VALUES(2,'goodbye')}
> 
> db1 eval {INSERT INTO t1 VALUES(3,'howdy!')}
> 
> set x [db1 eval {SELECT * FROM t1 ORDER BY a}]**

变量 **$x** 由上述代码设置为

> **1 hello 2 goodbye 3 howdy!**

您还可以通过指定数组变量的名称和跟随 SQL 代码的脚本来逐行处理查询结果。对于查询结果的每一行，所有列的值将插入到数组变量中，并且将执行脚本。例如：

> **db1 eval {SELECT * FROM t1 ORDER BY a} values {
> 
> parray values
> 
> puts ""
> 
> }**

这段代码将输出以下内容：

> **values(*) = a b
> 
> values(a) = 1
> 
> values(b) = hello**
> 
> values(*) = a b
> 
> values(a) = 2
> 
> values(b) = goodbye
> 
> values(*) = a b
> 
> values(a) = 3
> 
> values(b) = howdy!

对于结果的每行中的每列，该列的名称用作数组中的索引，并且列的值存储在相应的数组条目中。（注意：如果查询结果集中的两个或更多列具有相同的名称，则具有该名称的最后一列将覆盖先前的值，并且先前具有相同名称的较早列将无法访问。）特殊数组索引 * 用于按它们出现的顺序存储列名列表。

通常，NULL SQL 结果使用 [nullvalue](https://wiki.example.org/nullvalue) 设置存储在数组中。但是，如果使用了 **-withoutnulls** 选项，则 NULL SQL 值将导致相应的数组元素被取消设置。

如果数组变量名被省略或为空字符串，则每列的值将存储在与列名相同的变量中。例如：

> **db1 eval {SELECT * FROM t1 ORDER BY a} {
> 
> puts "a=$a b=$b"
> 
> }**

由此我们得到以下输出

> **a=1 b=hello
> 
> a=2 b=goodbye
> 
> a=3 b=howdy!**

Tcl 变量名称可以出现在第二个参数的 SQL 语句中的任何位置，合法的地方是在字符串或数字字面量中。变量的值将替换为变量名。如果变量不存在，则使用 NULL 值。例如：

> **db1 eval {INSERT INTO t1 VALUES(5,$bigstring)}**

注意，不需要对 $bigstring 值加引号。这是自动完成的。如果 $bigstring 是一个大字符串或二进制对象，则此技术不仅更容易编写，而且效率更高，因为它避免了复制 $bigstring 内容。

如果 $bigstring 变量既有字符串又有 "bytearray" 表示，则 TCL 将其值插入为字符串。如果它只有 "bytearray" 表示，则将该值插入为 BLOB。要强制将值插入为 BLOB，即使它也有文本表示，请在 "$" 的位置使用 "@" 字符。像这样：

> **db1 eval {INSERT INTO t1 VALUES(5,@bigstring)}**

如果变量没有 bytearray 表示形式，则 "@" 就像 "$" 一样工作。请注意，":" 在所有情况下都像 "$" 一样工作，因此以下是表示相同语句的另一种方式：

> **db1 eval {INSERT INTO t1 VALUES(5,:bigstring)}**

在变量名之前使用 ":" 而不是 "$" 有时可以很有用，如果 SQL 文本是用双引号 "..." 而不是大括号 {...} 括起来的话。当 SQL 包含在双引号 "..." 中时，TCL 将对 $-变量进行替换，如果不小心使用会导致 SQL 注入。但是 TCL 永远不会替换 :-变量，无论 SQL 是用双引号 "..." 还是大括号 {...} 括起来，因此使用 :-变量可以额外增加防止 SQL 注入的措施。

### "close" 方法

如其名称所示，"close" 方法关闭 SQLite 数据库，这会删除 *dbcmd* Tcl 命令。以下是打开然后立即关闭数据库的示例：

> **sqlite3 db1 ./testdb
> 
> db1 close**

如果直接删除*dbcmd*，这与调用 "close" 方法具有相同的效果。因此，下面的代码等同于之前的代码：

> **sqlite3 db1 ./testdb
> 
> rename db1 {}**

### "transaction" 方法

"transaction" 方法用于在 SQLite 数据库事务内执行 TCL 脚本。当脚本完成时，事务将提交，如果脚本失败则回滚。如果事务发生在另一个事务内（即使手动使用 BEGIN 启动的事务），则它是无操作的。

事务命令可用于以安全方式组合几个 SQLite 命令。当然，您始终可以手动启动事务使用 BEGIN。但是，如果发生错误导致 COMMIT 或 ROLLBACK 永远不会运行，则数据库将无限期保持锁定状态。此外，BEGIN 不支持嵌套，因此在启动新事务之前必须确保没有其他事务处于活动状态。 "transaction" 方法会自动处理所有这些细节。

语法如下：

> *dbcmd*  **事务**  *?事务类型?*   *脚本*

*事务类型* 可以是 **deferred**，**exclusive** 或 **immediate** 中的一种。默认值为 deferred。

### "cache" 方法

上面描述的 "eval" 方法保留最近评估的 SQL 命令的 prepared statements 缓存。 "cache" 方法用于控制此缓存。此命令的第一种形式如下：

> *dbcmd*  **缓存大小**  *N*

这设置了可以缓存的语句的最大数量。上限是 100。默认值是 10。如果将缓存大小设置为 0，则不会进行缓存。

此命令的第二种形式如下：

> *dbcmd*  **缓存刷新**

缓存刷新方法 finalizes 目前缓存中的所有准备好的语句。

### "complete" 方法

"complete" 方法以所谓的 SQL 字符串作为其唯一参数。如果该字符串是完整的 SQL 语句，则返回 TRUE；如果还有待输入，则返回 FALSE。

在构建交互式应用程序时，"complete" 方法非常有用，用于判断用户是否已完成输入一行 SQL 代码。这实际上只是 **sqlite3_complete()** C 函数的一个接口。

### "config" 方法

"config" 方法使用 sqlite3_db_config() 接口来查询或更改数据库连接的某些配置设置。不带参数运行此方法以获取可用配置设置及其当前值的 TCL 列表：

> *dbcmd*  **config**

以上将返回类似于以下内容：

> defensive 0 dqs_ddl 1 dqs_dml 1 enable_fkey 0 enable_qpsg 0 enable_trigger 1 enable_view 1 fts3_tokenizer 1 legacy_alter_table 0 legacy_file_format 0 load_extension 0 no_ckpt_on_close 0 reset_database 0 trigger_eqp 0 trusted_schema 1 writable_schema 0

添加单个配置设置的名称以查询该设置的当前值。可选地添加布尔值以更改设置。

建议进行以下四项配置更改以实现最大的应用程序安全性。关闭 trust_schema 设置可以防止在触发器、视图、CHECK 约束、生成列和表达式索引内使用虚拟表和不安全的 SQL 函数。关闭 dqs_dml 和 dqs_ddl 设置可以防止使用双引号字符串。打开 defensive 可以防止直接写入到影子表。

> ```sql
> db config trusted_schema 0
> db config defensive 1
> db config dqs_dml 0
> db config dqs_ddl 0
> 
> ```

### "copy" 方法

"copy" 方法将数据从文件复制到表中。它返回从文件成功处理的行数。copy 方法的语法如下：

> *dbcmd*  **copy**  *conflict-algorithm*   *table-name *  *file-name *     ?*column-separator*?   ?*null-indicator*?

冲突算法必须是 INSERT 语句的 SQLite 冲突算法之一：*rollback*、*abort*、*fail*、*ignore* 或 *replace*。有关更多信息，请参阅 SQLite 语言部分的 ON CONFLICT。冲突算法必须以小写指定。

Table-name 必须已存在作为一个表。File-name 必须存在，并且文件中的每一行必须包含与表定义的列数相同的列数。如果文件中的某行包含的列数多于或少于表定义的列数，copy 方法将回滚任何插入，并返回错误。

Column-separator 是一个可选的列分隔符字符串。默认为 ASCII 制表符 \t。

Null-indicator 是一个可选字符串，表示列值为 null。默认为空字符串。请注意，column-separator 和 null-indicator 是可选的位置参数；如果指定了 null-indicator，则必须先指定 column-separator 参数。

"copy" 方法实现了与 **.import** SQLite shell 命令类似的功能。

### "timeout" 方法

"timeout" 方法用于控制 SQLite 库在放弃数据库事务之前等待锁定清除的时间。默认超时时间为 0 毫秒（换句话说，默认行为是不等待）。

SQLite 数据库允许多个同时读取器或单个写入器，但不能同时进行读写。如果任何进程正在向数据库写入，则不允许其他进程读取或写入。如果任何进程正在读取数据库，则允许其他进程读取但不允许写入。整个数据库共享一个锁。

当 SQLite 尝试打开数据库并发现它被锁定时，它可以选择延迟一小段时间然后再次尝试打开文件。此过程重复，直到查询超时并且 SQLite 返回失败。超时时间可调整。默认情况下设置为 0，因此如果数据库被锁定，则 SQL 语句立即失败。但是您可以使用 "timeout" 方法将超时值更改为正数。例如：

> **db1 timeout 2000**

timeout 方法的参数是等待锁定清除的最大毫秒数。因此，在上面的示例中，最大延迟将为 2 秒。

### "busy" 方法

"busy" 方法与 "timeout" 类似，仅在数据库被锁定时起作用。但是 "busy" 方法允许程序员更多地控制要采取的操作。"busy" 方法指定一个回调 Tcl 过程，每当 SQLite 尝试打开一个锁定的数据库时调用该过程。在调用回调之前，会附加一个整数参数，该参数是当前锁定事件的先前调用次数。回调应该在短时间内完成一些其他有用的工作（如服务 GUI 事件），然后返回，以便可以再次尝试锁定。如果回调过程希望 SQLite 再次尝试打开数据库，则应返回 "0"；如果希望 SQLite 放弃当前操作，则应返回 "1"。

如果没有参数调用 busy 方法，则返回最后由 busy 方法设置的回调过程的名称。如果未设置任何回调过程，则返回空字符串。

### "enable_load_extension" 方法

SQLite 的扩展加载机制（使用 load_extension() SQL 函数访问）默认关闭。这是一种安全预防措施。如果应用程序希望使用 load_extension() 函数，则必须首先使用此方法打开此功能。

此方法接受一个布尔参数，用于打开或关闭扩展加载功能。

出于最佳安全考虑，除非确实需要，否则不要使用此方法，并在调用此方法之前运行 PRAGMA trusted_schema=OFF 或 "db config trusted_schema 0" 方法。

此方法映射到 sqlite3_enable_load_extension() C/C++ 接口。

### **exists** 方法

**exists** 方法类似于 "onecolumn" 和 "eval"，它执行 SQL 语句。不同之处在于，"exists" 方法始终返回布尔值，如果它执行的 SQL 语句返回一个或多个行，则返回 TRUE，如果 SQL 返回空集，则返回 FALSE。

**exists** 方法经常用于测试表中是否存在行。例如：

> **if {[db exists {SELECT 1 FROM table1 WHERE user=$user}]} {
> 
> # 如果 $user 存在则处理
> 
> } else {
> 
> # 如果 $user 不存在则处理
> 
> }**

### **last_insert_rowid** 方法

**last_insert_rowid** 方法返回一个整数，该整数是最近插入的数据库行的 ROWID。

### **function** 方法

**function** 方法向 SQLite 引擎注册新的 SQL 函数。参数是新 SQL 函数的名称以及实现该函数的 TCL 命令。在调用之前，函数的参数将追加到 TCL 命令中。

出于安全考虑，建议应用程序在使用此方法之前首先设置 PRAGMA trusted_schema=OFF 或运行 "db config trusted_schema 0" 方法。

语法如下：

> *dbcmd*  **function**   *sql-name*   *?options?*   *script*

以下示例创建一个名为 "hex" 的新 SQL 函数，该函数将其数值参数转换为十六进制编码字符串：

> **db 函数 hex {format 0x%X}**

**function** 方法接受以下选项：

> **-argcount** *INTEGER*
> 
> 指定 SQL 函数接受的参数数量。默认值为 -1，表示任意数量的参数。
> 
> **-deterministic**
> 
> 此选项指示函数在给定相同参数值时将始终返回相同的答案。SQLite 查询优化器使用此信息来缓存具有恒定输入的函数调用的答案，并重用结果，而不是重复调用函数。
> 
> **-directonly**
> 
> 此选项限制函数仅可通过直接的顶层 SQL 语句使用。该函数将不可供触发器、视图、CHECK 约束、生成列或索引表达式访问。此选项建议用于所有应用定义的 SQL 函数，并强烈建议用于具有副作用或者揭示应用程序内部状态的任何 SQL 函数。
> 
> **安全警告：** 如果没有此开关，攻击者可能能够修改数据库文件的模式，将新函数包含在触发器、视图或 CHECK 约束中，从而欺骗应用程序使用攻击者选择的参数运行函数。因此，如果新函数具有副作用或揭示应用程序内部状态，并且未使用 -directonly 选项，则存在潜在的安全漏洞。
> 
> **-innocuous**
> 
> 此选项指示函数没有副作用，并且不泄漏任何无法直接从其输入参数计算的信息。当指定此选项时，即使 PRAGMA trusted_schema=OFF，该函数也可以在触发器、视图、CHECK 约束、生成的列和/或索引表达式中使用。除非确实需要，否则不建议使用此选项。
> 
> **-returntype integer|real|text|blob|any**
> 
> 此选项用于配置函数返回的结果类型。如果将此选项设置为 "any"（默认值），SQLite 会基于 Tcl 值的内部类型尝试确定函数实现返回值的类型。或者，如果设置为 "text" 或 "blob"，则返回值始终为文本或 blob 值。如果将此选项设置为 "integer"，SQLite 尝试将函数返回的值强制转换为整数。如果无法在不丢失数据的情况下进行此操作，则尝试将其转换为实数值，最后回退到文本。如果将此选项设置为 "real"，则尝试返回实数值，如果无法实现，则回退到文本。

### "nullvalue" 方法

"nullvalue" 方法改变了作为 "eval" 方法结果返回的 NULL 的表示方式。

> **db1 nullvalue NULL**

"nullvalue" 方法在 Tcl 中缺少 NULL 表示时非常有用，用于区分 NULL 和空列值。NULL 值的默认表示是空字符串。

### "onecolumn" 方法

"onecolumn" 方法类似于 "eval"，它评估作为参数给出的 SQL 查询语句。不同之处在于，"onecolumn" 返回一个单一元素，即查询结果的第一行的第一列。

这是一个方便的方法。它保存用户免去了对 "eval" 结果进行 "`[lindex ... 0]`" 提取单列结果的步骤。

### "changes" 方法

"changes" 方法返回一个整数，该整数表示数据库中由最近的 "eval" 方法插入、删除和/或修改的行数。

### "total_changes" 方法

"total_changes" 方法返回一个整数，该整数表示自打开当前数据库连接以来插入、删除和/或修改的行数。

### "authorizer" 方法

"authorizer" 方法提供对 sqlite3_set_authorizer C/C++ 接口的访问。authorizer 的参数是一个过程的名称，当编译 SQL 语句以授权特定操作时会调用该过程。回调过程接受 5 个描述编码操作的参数。如果回调返回文本字符串 "SQLITE_OK"，则允许操作。如果返回 "SQLITE_IGNORE"，则操作会被静默禁用。如果返回 "SQLITE_DENY"，则编译将失败并显示错误。

如果参数是空字符串，则禁用授权者。如果省略参数，则返回当前授权者。

### "bind_fallback" 方法

"bind_fallback" 方法允许应用程序控制在没有匹配参数名称的 TCL 变量时如何处理参数绑定。

当 eval 方法 在 SQL 语句中看到命名的 SQL 参数（如 "$abc" 或 ":def" 或 "@ghi"），它尝试查找相同名称的 TCL 变量，并将该 TCL 变量的值绑定到 SQL 参数。如果没有这样的 TCL 变量存在，默认行为是将 SQL NULL 值绑定到参数。但是，如果指定了 bind_fallback proc，则调用该 proc 并将 proc 的返回值绑定到 SQL 参数。如果 proc 返回错误，则 SQL 语句中止并显示该错误。如果 proc 返回除 TCL_OK 或 TCL_ERROR 之外的其他代码，则将 SQL 参数绑定为 NULL，就像默认情况下一样。

"bind_fallback" 方法有一个可选参数。如果参数是空字符串，则取消 bind_fallback 并恢复默认行为。如果参数是非空字符串，则参数是要调用的 TCL 命令（通常是 proc 的名称），每当看到不匹配任何 TCL 变量的 SQL 参数时调用该命令。如果 "bind_fallback" 方法没有参数，则返回当前的 bind_fallback 命令。

例如，以下设置会导致 TCL 在 SQL 语句中包含不匹配任何全局 TCL 变量的参数时抛出错误：

> ```sql
>  proc bind_error {nm} {
>   error "no such variable: $nm"
> }
> db bind_fallback bind_error 
> ```

### "progress" 方法

此方法注册一个回调函数，它在查询处理期间定期调用。有两个参数：在调用之间的 SQLite 虚拟机操作码数，以及要调用的 TCL 命令。将进度回调设置为空字符串会禁用它。

进度回调可以用于显示长查询的状态或在长查询期间处理 GUI 事件。

### "collate" 方法

此方法注册新的文本排序序列。有两个参数：排序序列的名称和实现排序函数的 TCL 过程的名称。

例如，以下代码实现了一个称为 "NOCASE" 的排序序列，按文本顺序排序而不考虑大小写：

> ```sql
>  proc nocase_compare {a b} {
>   return [string compare [string tolower $a] [string tolower $b]]
> }
> db collate NOCASE nocase_compare 
> ```

### "collation_needed" 方法

此方法注册一个回调函数，当 SQLite 引擎需要特定排序序列但未注册该排序序列时调用该回调。回调函数会注册排序序列，参数是所需排序序列的名称。

### "commit_hook" 方法

此方法注册一个回调例程，该例程在 SQLite 尝试提交对数据库的更改之前调用。如果回调抛出异常或返回非零结果，则事务回滚而不是提交。

### "rollback_hook" 方法

此方法注册一个回调例程，该例程在 SQLite 尝试执行回滚之前调用。脚本参数不会改变。

### "status" 方法

此方法返回最近评估的 SQL 语句的状态信息。状态方法接受一个参数，应为"steps"或"sorts"之一。如果参数为"steps"，则该方法返回前一个 SQL 语句评估的全表扫描步骤数。如果参数为"sorts"，则该方法返回排序操作的数量。此信息可用于检测未使用索引以加快搜索或排序的查询。

状态方法基本上是 sqlite3_stmt_status() C 语言接口的包装器。

### "update_hook" 方法

此方法注册一个回调例程，该例程在每次通过 UPDATE、INSERT 或 DELETE 语句修改每行后立即调用。在调用回调之前，会附加四个参数：

+   "INSERT"、"UPDATE" 或 "DELETE" 等关键字，视情况而定

+   正在更改的数据库的名称

+   正在更改的表

+   正在更改的表中行的 rowid

回调不能执行任何会修改调用更新钩子的数据库连接的操作，如运行查询。

### "preupdate" 方法

此方法要么注册一个回调例程，该例程在每次通过 UPDATE、INSERT 或 DELETE 语句修改每行前调用，要么执行与即将进行的更新相关的某些操作。

要注册或移除预更新回调，请使用以下语法：

> *dbcmd*  **preupdate  hook**  *?SCRIPT?*

注册预更新回调时，每次行修改之前，会使用以下参数运行回调：

+   "INSERT"、"UPDATE" 或 "DELETE" 等关键字，视情况而定

+   正在更改的数据库的名称

+   正在更改的表

+   正在更改的表中行的原始 rowid

+   正在更改的表中行的新 rowid（如果有）

回调不能执行任何会修改调用预更新钩子的数据库连接的操作，如运行查询。

当回调正在执行时，并且仅在这时，可以通过使用指定的语法执行这些预更新操作：

> *dbcmd*  **preupdate  count**
> 
> *dbcmd*  **preupdate  depth**
> 
> *dbcmd*  **preupdate  new**  *INDEX*
> 
> *dbcmd*  **preupdate  old**  *INDEX*

**count** 子方法返回正在插入、更新或删除的行中的列数。

**depth** 子方法返回更新嵌套深度。对于直接的插入、更新或删除操作，深度为 0；对于由顶层触发器调用的插入、更新或删除，深度为 1；或者对于由触发器调用的触发器引起的更改，深度更高。

**old** 和 **new** 子方法分别返回正在更新的行的原始或更改后的列值。

请注意，Tcl 接口（和底层的 SQLite 库）必须使用预处理器符号 SQLITE_ENABLE_PREUPDATE_HOOK 定义才能使用**preupdate**方法。

### "wal_hook" 方法

此方法注册一个回调例程，该例程在数据库处于 WAL 模式时在事务提交后调用。在调用前，将向回调命令附加两个参数：

+   提交事务的数据库的名称

+   该数据库的写入前日志（WAL）文件中的条目数

此方法可以决定是否运行一个 checkpoint，可以是自身或后续的空闲回调。请注意，SQLite 仅允许一个 WAL 挂钩。默认情况下，此单个 WAL 挂钩用于自动检查点。如果设置了显式 WAL 挂钩，则该 WAL 挂钩必须确保发生检查点，因为自动检查点机制将被禁用。

此方法应返回一个整数值，该值等效于 SQLite 错误代码（成功情况下通常为 SQLITE_OK 的 0 或者如果发生某些错误则为 SQLITE_ERROR 的 1）。如同 sqlite3_wal_hook()，如果脚本返回的值无法解释为整数值，或者脚本引发 Tcl 异常，则 SQLite 不返回错误，但会引发 Tcl 后台错误。

### "incrblob" 方法

此方法打开一个 TCL 通道，可用于读取或写入数据库中预先存在的 BLOB。语法如下：

> *dbcmd*  **incrblob**  **?-readonly?**   *?DB?  TABLE  COLUMN  ROWID*

该命令返回一个新的 TCL 通道，用于读取或写入 BLOB。使用 TCL 的**close**命令关闭通道。

### "errorcode" 方法

此方法返回由最近的 SQLite 操作产生的数值错误代码。

### "trace" 方法

"trace" 方法注册一个回调，该回调在编译每个 SQL 语句时调用。SQL 的文本作为一个字符串附加到调用前的命令。可以使用此功能（例如）来记录应用程序执行的所有 SQL 操作。

### "trace_v2" 方法

"trace_v2" 方法注册一个回调，该回调在编译每个 SQL 语句时调用。语法如下：

> *dbcmd*  **trace_v2**  ?*callback*?  ?*mask*?

此命令导致在发生特定条件时调用"callback"脚本。条件由*mask*参数确定，*mask*应为零个或多个以下关键字的 TCL 列表：

+   **statement**

+   **profile**

+   **row**

+   **close**

**statement**跟踪会在运行新的 SQL 语句时调用回调函数，带有两个参数。第一个参数是一个整数，表示指向底层 sqlite3_stmt 对象的指针值。此整数可用于将 SQL 语句文本与**profile**或**row**回调的结果关联起来。第二个参数是 SQL 语句运行时的未展开文本。所谓未展开是指文本中的变量替换未展开成变量值。这与"trace"方法的行为不同，后者会展开变量替换。

**profile**跟踪会在每个 SQL 语句完成时调用带有两个参数的回调函数。第一个参数是底层 sqlite3_stmt 对象的值。第二个参数是语句的大致运行时间（以纳秒为单位）。运行时间是根据应用程序运行平台的能力提供的最佳估计。

**row**跟踪会在每次从 SQL 语句中获取新结果行时调用回调函数，带有一个参数。该参数是底层 sqlite3_stmt 对象指针的值。

关闭时的**close**跟踪会调用带有一个参数的回调函数，因为数据库连接正在关闭。该参数是一个整数，表示正在关闭的底层 sqlite3 对象的指针值。

只能在数据库连接上注册一个单一的跟踪回调。每次使用"trace"或"trace_v2"都会取消先前的所有跟踪回调。

### "backup"方法

"backup"方法创建一个活动数据库的备份副本。命令语法如下：

> *dbcmd*  **backup**  ?*source-database*?  *backup-filename*

可选的*source-database*参数指定应备份当前连接中的哪个数据库。默认值为**main**（或者说是主数据库文件）。要备份 TEMP 表，请使用**temp**。要备份通过 ATTACH 命令添加到连接中的辅助数据库，请使用在 ATTACH 命令中分配的数据库名称。

*backup-filename*是备份写入的文件名。*Backup-filename*不需要事先存在，但如果存在，必须是一个格式良好的 SQLite 数据库。

### "restore"方法

"restore"方法将内容从单独的数据库文件复制到当前数据库连接中，覆盖任何现有内容。命令语法如下：

> *dbcmd*  **restore**  ?*target-database*?  *source-filename*

可选的 *target-database* 参数告知当前连接中应该使用新内容覆盖的数据库。默认值是 **main**（或者说，主数据库文件）。要重新填充 TEMP 表，请使用 **temp**。要覆盖使用 ATTACH 命令添加到连接中的辅助数据库，请使用该命令中分配的数据库名称。

*source-filename* 是现有的格式良好的 SQLite 数据库文件的名称，从中提取内容。

### "serialize" 方法

"serialize" 方法创建一个 BLOB，它是底层数据库的完整副本。语法如下：

> *dbcmd*  **serialize**  ?*database*?

可选参数是要序列化的模式或数据库的名称。默认值是 "main"。

此例程返回一个 TCL 字节数组，该数组是指定数据库的完整内容。此字节数组可写入文件，然后像普通的 SQLite 数据库一样使用，或者可通过 TCP/IP 连接发送给其他应用程序，或传递给另一个数据库连接的 "deserialize" 方法。

此方法仅在 SQLite 编译时使用 -DSQLITE_ENABLE_DESERIALIZE 时有效

### "deserialize" 方法

"deserialize" 方法接受一个包含 SQLite 数据库文件的 TCL 字节数组，并将其添加到数据库连接中。语法如下：

> *dbcmd*  **deserialize**  ?*database*?  *value*

可选的 *database* 参数标识应该接收反序列化的附加数据库。默认值为 "main"。

此命令导致 SQLite 断开与先前数据库的连接，并重新连接到一个内存数据库，其内容为 *value*。如果 *value* 不是包含良好定义的 SQLite 数据库的字节数组，则后续命令可能会返回 SQLITE_CORRUPT 错误。

此方法仅在 SQLite 编译时使用 -DSQLITE_ENABLE_DESERIALIZE 时有效

### "interrupt" 方法

"interrupt" 方法调用 sqlite3_interrupt() 接口，导致任何待处理的查询中止。

### "version" 方法

返回当前库的版本。例如，"3.23.0"。

### "profile" 方法

此方法用于分析应用程序运行的 SQL 语句的执行情况。语法如下：

> *dbcmd*  **profile**  ?*script*?

除非 *script* 是空字符串，否则此方法会安排在每个 SQL 语句执行后评估 *script*。在调用之前会附加两个参数到 *script*：执行的 SQL 语句的文本和执行该语句所用的时间（以纳秒计）。

数据库句柄一次只能注册一个配置文件脚本。如果在调用配置文件方法时已经注册了脚本，则先前的配置文件脚本将被新的脚本替换。如果 *script* 参数是空字符串，则取消任何先前注册的配置文件回调，但不注册新的配置文件脚本。

### "unlock_notify" 方法

`unlock_notify` 方法用于访问 sqlite3_unlock_notify() 接口，用于 SQLite 核心库的测试目的。不建议应用程序使用此方法。
