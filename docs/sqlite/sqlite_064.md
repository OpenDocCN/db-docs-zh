# 1\. 入门指南

> 原文：[`sqlite.com/cli.html`](https://sqlite.com/cli.html)

SQLite 项目提供了一个名为**sqlite3**（或 Windows 上的**sqlite3.exe**）的简单命令行程序，允许用户手动输入并执行 SQL 语句，针对 SQLite 数据库或 Z​​IP 存档。本文提供了如何使用**sqlite3**程序的简要介绍。

通过在命令提示符下键入"sqlite3"启动**sqlite3**程序，可选择性地跟上保存 SQLite 数据库（或 Z​​IP 存档）的文件名。如果指定的文件不存在，则将自动创建一个具有给定名称的新数据库文件。如果在命令行中未指定数据库文件，则将创建一个临时数据库，并在"sqlite3"程序退出时自动删除。

启动时，**sqlite3**程序将显示一个简短的横幅消息，然后提示您输入 SQL。键入 SQL 语句（以分号结尾），按"Enter"执行 SQL 语句。

例如，要创建名为"ex1"的新 SQLite 数据库，并包含名为"tbl1"的单个表，您可以这样做：

```sql
$ sqlite3 ex1
SQLite version 3.36.0 2021-06-18 18:36:39
Enter ".help" for usage hints.
sqlite> create table tbl1(one text, two int);
sqlite> insert into tbl1 values('hello!',10);
sqlite> insert into tbl1 values('goodbye', 20);
sqlite> select * from tbl1;
hello!|10
goodbye|20
sqlite>

```

终止 sqlite3 程序时，请键入系统的文件结束符（通常是 Control-D）。使用中断字符（通常是 Control-C）来停止长时间运行的 SQL 语句。

确保每个 SQL 命令的末尾都键入分号！sqlite3 程序会查找分号以确定您的 SQL 命令何时完成。如果省略分号，sqlite3 将给出一个继续提示，并等待您输入更多文本以完成 SQL 命令。此功能允许您输入跨多行的 SQL 命令。例如：

```sql
sqlite> CREATE TABLE tbl2 (
   ...>   f1 varchar(30) primary key,
   ...>   f2 text,
   ...>   f3 real
   ...> );
sqlite>

```

# 2\. 在 Windows 上双击启动

Windows 用户可以双击**sqlite3.exe**图标，使命令行 shell 弹出运行 SQLite 的终端窗口。但是，因为双击启动 sqlite3.exe 时没有命令行参数，所以不会指定任何数据库文件，因此 SQLite 将使用一个在会话退出时删除的临时数据库。要将持久磁盘文件用作数据库，请在终端窗口启动后立即输入".open"命令：

```sql
SQLite version 3.36.0 2021-06-18 18:36:39
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> .open ex1.db
sqlite>

```

上述示例导致名为"ex1.db"的数据库文件被打开和使用。如果之前不存在"ex1.db"文件，则会创建该文件。您可能希望使用完整路径名来确保文件位于您认为的目录中。请使用斜杠作为目录分隔符。换句话说，请使用"c:/work/ex1.db"，而不是"c:\work\ex1.db"。

或者，您可以使用默认临时存储创建一个新的数据库，然后使用".save"命令将该数据库保存到磁盘文件中：

```sql
SQLite version 3.36.0 2021-06-18 18:36:39
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite> *... many SQL commands omitted ...*
sqlite> .save ex1.db
sqlite>

```

使用".save"命令时要小心，因为它将覆盖任何具有相同名称的现有数据库文件而不会提示确认。与".open"命令一样，您可能希望使用带有斜杠目录分隔符的完整路径名，以避免歧义。

# 3\. 特殊命令到 sqlite3（点命令）

大多数时候，sqlite3 只是读取输入行并将其传递给 SQLite 库进行执行。但以点号 (".") 开头的输入行会被 sqlite3 程序拦截并解释。这些 "点命令" 通常用于更改查询的输出格式，或执行某些预打包的查询语句。最初只有几个点命令，但多年来积累了许多新功能，因此今天有超过 60 个点命令。

要查看可用的点命令列表，可以输入不带参数的".help"。或输入".help TOPIC"以获取有关 TOPIC 的详细信息。可用点命令的列表如下：

```sql
sqlite> .help
.archive ...             Manage SQL archives
.auth ON|OFF             Show authorizer callbacks
.backup ?DB? FILE        Backup DB (default "main") to FILE
.bail on|off             Stop after hitting an error.  Default OFF
.cd DIRECTORY            Change the working directory to DIRECTORY
.changes on|off          Show number of rows changed by SQL
.check GLOB              Fail if output since .testcase does not match
.clone NEWDB             Clone data into NEWDB from the existing database
.connection [close] [#]  Open or close an auxiliary database connection
.crnl on|off             Translate \n to \r\n.  Default ON
.databases               List names and files of attached databases
.dbconfig ?op? ?val?     List or change sqlite3_db_config() options
.dbinfo ?DB?             Show status information about the database
.dump ?OBJECTS?          Render database content as SQL
.echo on|off             Turn command echo on or off
.eqp on|off|full|...     Enable or disable automatic EXPLAIN QUERY PLAN
.excel                   Display the output of next command in spreadsheet
.exit ?CODE?             Exit this program with return-code CODE
.expert                  EXPERIMENTAL. Suggest indexes for queries
.explain ?on|off|auto?   Change the EXPLAIN formatting mode.  Default: auto
.filectrl CMD ...        Run various sqlite3_file_control() operations
.fullschema ?--indent?   Show schema and the content of sqlite_stat tables
.headers on|off          Turn display of headers on or off
.help ?-all? ?PATTERN?   Show help text for PATTERN
.import FILE TABLE       Import data from FILE into TABLE
.indexes ?TABLE?         Show names of indexes
.limit ?LIMIT? ?VAL?     Display or change the value of an SQLITE_LIMIT
.lint OPTIONS            Report potential schema issues.
.load FILE ?ENTRY?       Load an extension library
.log FILE|on|off         Turn logging on or off.  FILE can be stderr/stdout
.mode MODE ?OPTIONS?     Set output mode
.nonce STRING            Suspend safe mode for one command if nonce matches
.nullvalue STRING        Use STRING in place of NULL values
.once ?OPTIONS? ?FILE?   Output for the next SQL command only to FILE
.open ?OPTIONS? ?FILE?   Close existing database and reopen FILE
.output ?FILE?           Send output to FILE or stdout if FILE is omitted
.parameter CMD ...       Manage SQL parameter bindings
.print STRING...         Print literal STRING
.progress N              Invoke progress handler after every N opcodes
.prompt MAIN CONTINUE    Replace the standard prompts
.quit                    Stop interpreting input stream, exit if primary.
.read FILE               Read input from FILE or command output
.recover                 Recover as much data as possible from corrupt db.
.restore ?DB? FILE       Restore content of DB (default "main") from FILE
.save ?OPTIONS? FILE     Write database to FILE (an alias for .backup ...)
.scanstats on|off|est    Turn sqlite3_stmt_scanstatus() metrics on or off
.schema ?PATTERN?        Show the CREATE statements matching PATTERN
.separator COL ?ROW?     Change the column and row separators
.session ?NAME? CMD ...  Create or control sessions
.sha3sum ...             Compute a SHA3 hash of database content
.shell CMD ARGS...       Run CMD ARGS... in a system shell
.show                    Show the current values for various settings
.stats ?ARG?             Show stats or turn stats on or off
.system CMD ARGS...      Run CMD ARGS... in a system shell
.tables ?TABLE?          List names of tables matching LIKE pattern TABLE
.timeout MS              Try opening locked tables for MS milliseconds
.timer on|off            Turn SQL timer on or off
.trace ?OPTIONS?         Output each SQL statement as it is run
.version                 Show source, library and compiler versions
.vfsinfo ?AUX?           Information about the top-level VFS
.vfslist                 List all available VFSes
.vfsname ?AUX?           Print the name of the VFS stack
.width NUM1 NUM2 ...     Set minimum column widths for columnar output
sqlite>

```

# 4\. “点命令”、SQL 和更多规则

## 4.1\. 行结构

CLI 的输入被解析为一个序列，其中包括：

+   SQL 语句；

+   点命令；或

+   CLI 注释

SQL 语句是自由格式的，可以跨多行分布，任何位置嵌入空白或 SQL 注释。它们以输入行末尾的 ';' 字符或 '/' 字符或单独一行上的单词 "go" 终止。用于终止的目的忽略尾随空白字符。

点命令具有更严格的结构：

+   它必须以其 "." 开头，左边距没有前导空白字符。

+   它必须完全包含在单个输入行中。

+   它不能出现在普通 SQL 语句的中间。换句话说，它不能出现在继续提示符处。

+   点命令没有注释语法。

CLI 还接受以 '#' 字符开头并延伸到行尾的整行注释。在 '#' 前不能有空白字符。

## 4.2\. 点命令参数

传递给点命令的参数根据以下规则从命令尾部解析：

1.  末尾的换行符和任何其他尾随空白字符都被丢弃；

1.  紧跟在点命令名称后面的空白字符，或者任何参数输入结束界定符被丢弃；

1.  参数输入以任何非空白字符开头；

1.  参数输入以其前导字符取决于其结束字符：

+   对于前导单引号（'），单引号充当结束定界符；

+   对于前导双引号（"），未转义的双引号充当结束定界符；

+   对于任何其他前导字符，结束定界符是任何空白字符；和

+   命令尾端作为任何参数的结束定界符；

1.  在双引号参数输入内，反斜杠转义的双引号是参数的一部分而不是其终止引号；

1.  在双引号参数内，传统的 C 字符串文字，反斜杠转义序列被转换；和

1.  参数输入分隔符（界定引号或空白字符）被丢弃以得到传递的参数。

## 4.3\. 点命令执行

".separator" dot 命令由 sqlite3.exe 命令行程序解释，而不是 SQLite 本身。因此，所有 dot 命令都不能作为 SQLite 接口（如 sqlite3_prepare() 或 sqlite3_exec()）的参数使用。

# 5\. 更改输出格式

sqlite3 程序能够以 14 种不同的输出格式显示查询结果：

+   ascii

+   box

+   csv

+   column

+   html

+   insert

+   json

+   line

+   list

+   markdown

+   quote

+   table

+   tabs

+   tcl

可以使用 ".mode" dot 命令在这些输出格式之间切换。默认输出模式为 "list"。在 list 模式下，查询结果的每一行写在输出的一行上，每个行内的列由特定分隔字符串分隔。默认分隔符是管道符号（"|"）。当将查询输出发送到另一个程序（如 AWK）进行额外处理时，list 模式特别有用。

```sql
sqlite> .mode list
sqlite> select * from tbl1;
hello!|10
goodbye|20
sqlite>

```

输入 ".mode" 且不带参数以显示当前模式：

```sql
sqlite> .mode
current output mode: list
sqlite>

```

使用 ".separator" dot 命令来更改分隔符。例如，要将分隔符更改为逗号和空格，可以这样做：

```sql
sqlite> .separator ", "
sqlite> select * from tbl1;
hello!, 10
goodbye, 20
sqlite>

```

下一个 ".mode" 命令可能会将 ".separator" 重置回某个默认值（取决于其参数）。因此，如果要继续使用非标准分隔符，每次更改模式后可能需要重复 ".separator" 命令。

在 "quote" 模式下，输出格式为 SQL 字面量。字符串用单引号括起来，内部单引号通过双重转义。Blobs 以十六进制 blob 字面量表示（例如：x'abcd'）。数字显示为 ASCII 文本，NULL 值显示为 "NULL"。所有列都用逗号（或使用 ".separator" 选择的其他字符）分隔。

```sql
sqlite> .mode quote
sqlite> select * from tbl1;
'hello!',10
'goodbye',20
sqlite>

```

在 "line" 模式下，数据库行中的每一列都显示在单独的一行上。每行由列名、等号和列数据组成。连续的记录之间用空行分隔。以下是 line 模式输出的示例：

```sql
sqlite> .mode line
sqlite> select * from tbl1;
one = hello!
two = 10

one = goodbye
two = 20
sqlite>

```

在列模式下，每条记录都显示在单独的一行上，数据在列中对齐。例如：

```sql
sqlite> .mode column
sqlite> select * from tbl1;
one       two
--------  ---
hello!    10
goodbye   20
sqlite>

```

在 "column" 模式（以及 "box"、"table" 和 "markdown" 模式）中，列的宽度会自动调整。但您可以使用 ".width" 命令为每个列指定特定的宽度。".width" 的参数是整数，表示每列要占用的字符数。负数表示右对齐。因此：

```sql
sqlite> .width 12 -6
sqlite> select * from tbl1;
one              two
------------  ------
hello!            10
goodbye           20
sqlite>

```

宽度为 0 表示列宽度自动选择。未指定的列宽度变为零。因此，不带参数的 ".width" 命令会将所有列宽度重置为零，导致所有列宽度自动确定。

"column" 模式是一个表格输出格式。其他表格输出格式包括 "box"、"markdown" 和 "table"：

```sql
sqlite> .width
sqlite> .mode markdown
sqlite> select * from tbl1;
|   one   | two |
|---------|-----|
| hello!  | 10  |
| goodbye | 20  |
sqlite> .mode table
sqlite> select * from tbl1;
+---------+-----+
|   one   | two |
+---------+-----+
| hello!  | 10  |
| goodbye | 20  |
+---------+-----+
sqlite> .mode box
sqlite> select * from tbl1;
┌─────────┬─────┐
│   one   │ two │
├─────────┼─────┤
│ hello!  │ 10  │
│ goodbye │ 20  │
└─────────┴─────┘
sqlite>

```

列模式接受一些额外的选项来控制格式化。"--wrap *N*" 选项（*N* 为整数）会导致列在超过 N 个字符的文本处换行。如果 N 为零，则禁用换行。

```sql
sqlite> insert into tbl1 values('The quick fox jumps over a lazy brown dog.',90);
sqlite> .mode box --wrap 30
sqlite> select * from tbl1 where two>50;
┌────────────────────────────────┬─────┐
│              one               │ two │
├────────────────────────────────┼─────┤
│ The quick fox jumps over a laz │ 90  │
│ y brown dog.                   │     │
└────────────────────────────────┴─────┘
sqlite>

```

换行发生在正好 *N* 个字符之后，这可能发生在单词中间。要在单词边界处换行，请添加 "--wordwrap on" 选项（或简写为 "-ww"）：

```sql
sqlite> .mode box --wrap 30 -ww
sqlite> select * from tbl1 where two>50;
┌─────────────────────────────┬─────┐
│             one             │ two │
├─────────────────────────────┼─────┤
│ The quick fox jumps over a  │ 90  │
│ lazy brown dog.             │     │
└─────────────────────────────┴─────┘
sqlite>

```

"--quote" 选项会导致每列中的结果像 SQL 字面量一样带引号，就像 "quote" 模式一样。有关其他选项，请参阅在线帮助。

命令 ".mode box --wrap 60 --quote" 在通用数据库查询中非常有用，因此它具有自己的别名。您可以直接使用 ".mode qbox" 而不是输入整个 27 个字符的命令。

另一种有用的输出模式是 "insert"。在插入模式下，输出格式化为类似 SQL INSERT 语句的形式。使用插入模式生成文本，以便稍后将数据输入到其他数据库中。

在指定插入模式时，必须提供额外的参数，即要插入的表名。例如：

```sql
sqlite> .mode insert new_table
sqlite> select * from tbl1 where two<50;
INSERT INTO "new_table" VALUES('hello',10);
INSERT INTO "new_table" VALUES('goodbye',20);
sqlite>

```

其他输出模式包括 "csv"、"json" 和 "tcl"。请自行尝试以查看它们的功能。

# 6\. 查询数据库模式

sqlite3 程序提供了几个方便的命令，用于查看数据库的架构。这些命令提供了一些其他方法无法完成的便利。这些命令纯粹是为了快捷起见而提供的。

例如，要查看数据库中表的列表，可以输入 ".tables"。

```sql
sqlite> .tables
tbl1 tbl2
sqlite>

```

".tables" 命令类似于设置列表模式，然后执行以下查询：

```sql
SELECT name FROM sqlite_schema
WHERE type IN ('table','view') AND name NOT LIKE 'sqlite_%'
ORDER BY 1

```

但是 ".tables" 命令功能更强大。它查询 sqlite_schema 表中所有 attached 数据库，而不仅限于主数据库。并将其输出整齐地排列成列。

".indexes" 命令的工作方式类似于列出所有索引。如果 ".indexes" 命令带有表名作为参数，则只显示该表上的索引。

".schema" 命令显示数据库的完整模式，或者如果提供了可选的表名参数，则显示单个表的模式：

```sql
sqlite> .schema
create table tbl1(one varchar(10), two smallint)
CREATE TABLE tbl2 (
  f1 varchar(30) primary key,
  f2 text,
  f3 real
);
sqlite> .schema tbl2
CREATE TABLE tbl2 (
  f1 varchar(30) primary key,
  f2 text,
  f3 real
);
sqlite>

```

".schema" 命令与设置列表模式然后输入以下查询大致相同：

```sql
SELECT sql FROM sqlite_schema
ORDER BY tbl_name, type DESC, name

```

与 ".tables" 类似，".schema" 命令显示所有 attached 数据库的模式。如果只想查看单个数据库（比如 "main"）的模式，则可以在 ".schema" 后加上参数来限制其输出：

```sql
sqlite> .schema main.*

```

".schema" 命令可以使用 "--indent" 选项增强，这样它会尝试重新格式化模式的各种 CREATE 语句，使其更易于人类阅读。

".databases" 命令显示当前连接中所有打开的数据库列表。至少会有两个数据库。第一个是 "main"，最初打开的数据库。第二个是 "temp"，用于临时表的数据库。可能会列出使用 ATTACH 语句附加的其他数据库。第一个输出列是数据库的附加名称，第二个结果列是外部文件的文件名。可能会有第三个结果列，其内容将是 "'r/o'" 或 "'r/w'"，取决于数据库文件是只读还是读写。还可能会有第四个结果列，显示该数据库文件的 sqlite3_txn_state() 的结果。

```sql
sqlite> .databases

```

".fullschema" 命令类似于 ".schema" 命令，显示整个数据库架构。但 ".fullschema" 还包括统计表 "sqlite_stat1"、"sqlite_stat3" 和 "sqlite_stat4" 的转储，如果存在的话。".fullschema" 命令通常提供了重新创建特定查询的查询计划所需的所有信息。在向 SQLite 开发团队报告 SQLite 查询规划器可能存在的问题时，开发人员被要求提供完整的 ".fullschema" 输出作为问题报告的一部分。请注意，sqlite_stat3 和 sqlite_stat4 表包含索引条目的样本，因此可能包含敏感数据，因此不要通过公共频道发送专有数据库的 ".fullschema" 输出。

# 7\. 打开数据库文件

".open" 命令打开一个新的数据库连接，在先关闭先前打开的数据库命令之后。最简单的形式下，".open" 命令仅在其参数命名的文件上调用 sqlite3_open()。使用名称 ":memory:" 可以打开一个新的内存数据库，当 CLI 退出或再次运行 ".open" 命令时会消失。或者不使用名称来打开一个私有的临时磁盘数据库，也会在退出或使用 ".open" 后消失。

如果在 ".open" 中包含 --new 选项，则在打开之前将重置数据库。任何先前的数据都将被销毁。这是一次破坏性的覆盖先前数据操作，不会请求确认，请谨慎使用该选项。

--readonly 选项以只读模式打开数据库。写操作将被禁止。

--deserialize 选项会导致将磁盘文件的整个内容读入内存，然后使用 sqlite3_deserialize() 接口作为内存数据库打开。如果有大型数据库，这当然会消耗大量内存。此外，除非显式使用 ".save" 或 ".backup" 命令保存更改，否则对数据库所做的任何更改都不会保存回磁盘。

--append 选项使 SQLite 数据库附加到现有文件而不是作为独立文件工作。有关更多信息，请参阅 [appendvfs 扩展](https://www.sqlite.org/src/file/ext/misc/appendvfs.c)。

--zip 选项导致指定的输入文件被解释为 ZIP 存档，而不是 SQLite 数据库文件。

--hexdb 选项使数据库内容以十六进制格式从后续输入的行中读取，而不是从磁盘上的单独文件中读取。"dbtotxt" 命令行工具可用于为数据库生成适当的文本。--hexdb 选项旨在供 SQLite 开发人员用于测试目的。我们不知道在内部 SQLite 测试和开发之外有任何用例。

# 8\. 重定向 I/O

## 8.1\. 将结果写入文件

默认情况下，sqlite3 将查询结果发送到标准输出。您可以使用 ".output" 和 ".once" 命令更改此行为。只需将输出文件的名称作为 .output 的参数，所有随后的查询结果将被写入该文件。或者使用 .once 命令而不是 .output，在单个下一个命令之前重定向输出，然后恢复到控制台。使用没有参数的 .output 来重新开始写入标准输出。例如：

```sql
sqlite> .mode list
sqlite> .separator |
sqlite> .output test_file_1.txt
sqlite> select * from tbl1;
sqlite> .exit
$ cat test_file_1.txt
hello|10
goodbye|20
$

```

如果 ".output" 或 ".once" 文件名的第一个字符是管道符号（"|"），则剩余的字符被视为命令，并将输出发送到该命令。这使得可以轻松地将查询结果导入到其他进程中。例如，在 Mac 上，"open -f" 命令打开一个文本编辑器来显示从标准输入读取的内容。因此，要在文本编辑器中查看查询结果，可以输入：

```sql
sqlite> .once | open -f
sqlite> SELECT * FROM bigTable;

```

如果 ".output" 或 ".once" 命令带有 "-e" 参数，则输出会被收集到一个临时文件中，并调用系统文本编辑器打开该文本文件。因此，命令 ".once -e" 实现的效果与 ".once '|open -f'" 相同，但在所有系统上都是可移植的。

如果 ".output" 或 ".once" 命令带有 "-x" 参数，这会导致它们将输出累积为逗号分隔值（CSV）在一个临时文件中，然后调用默认的系统实用程序来查看 CSV 文件（通常是电子表格程序）。这是将查询结果快速发送到电子表格以便轻松查看的一种方法：

```sql
sqlite> .once -x
sqlite> SELECT * FROM bigTable;

```

".excel" 命令是 ".once -x" 的别名，功能完全相同。

## 8.2\. 从文件中读取 SQL

在交互模式下，sqlite3 从键盘读取输入文本（可以是 SQL 语句或 dot-commands）。当然，您也可以在启动 sqlite3 时从文件重定向输入，但这样就无法与程序交互。有时从命令行运行包含在文件中的 SQL 脚本并输入其他命令是有用的。为此，提供了 ".read" dot-command。

“.read” 命令接受一个参数，该参数通常是要读取输入文本的文件名。

```sql
sqlite> .read myscript.sql

```

“.read” 命令临时停止从键盘读取，并从指定的文件名中读取输入。在达到文件末尾后，输入会恢复到键盘。脚本文件可以包含点命令，就像普通交互式输入一样。

如果“.read”命令的参数以“|”字符开头，则不会打开参数作为文件，而是将参数（去掉前导“|”）作为命令运行，然后使用该命令的输出作为其输入。因此，如果您有一个生成 SQL 的脚本，可以使用类似以下命令直接执行该 SQL：

```sql
sqlite> .read |myscript.bat

```

## 8.3\. 文件 I/O 函数

命令行 shell 添加了两个 应用定义的 SQL 函数，分别用于从文件中读取内容到表列中，以及将列的内容写入文件。

readfile(X) SQL 函数读取名为 X 的文件的整个内容，并将该内容作为 BLOB 返回。这可以用于将内容加载到表中。例如：

```sql
sqlite> CREATE TABLE images(name TEXT, type TEXT, img BLOB);
sqlite> INSERT INTO images(name,type,img)
   ...>   VALUES('icon','jpeg',readfile('icon.jpg'));

```

writefile(X,Y) SQL 函数将 blob Y 写入名为 X 的文件，并返回写入的字节数。使用此函数将单个表列的内容提取到文件中。例如：

```sql
sqlite> SELECT writefile('icon.jpg',img) FROM images WHERE name='icon';

```

请注意，readfile(X) 和 writefile(X,Y) 函数是扩展函数，并不是内置于核心 SQLite 库中的。这些例程作为 可加载扩展 提供，源文件位于 SQLite 源代码库 中的 [ext/misc/fileio.c](https://www.sqlite.org/src/artifact?ci=trunk&filename=ext/misc/fileio.c)。

## 8.4\. 编辑函数 edit()

CLI 还有一个内置的 SQL 函数名为 edit()。Edit() 接受一到两个参数。第一个参数是一个值 - 通常是要编辑的大多行字符串。第二个参数是调用文本编辑器的命令（可能包含影响编辑器行为的选项）。如果省略第二个参数，则使用 VISUAL 环境变量。edit() 函数将其第一个参数写入临时文件，在编辑器完成后重新读取文件到内存，然后返回编辑后的文本。

edit() 函数可用于对大文本值进行更改。例如：

```sql
sqlite> UPDATE docs SET body=edit(body) WHERE name='report-15';

```

在此示例中，“report-15” 的 docs.name 字段的 docs.body 内容将被发送到编辑器。编辑器返回后，结果将被写回 docs.body 字段中。

edit() 的默认操作是调用文本编辑器。但是，通过在第二个参数中使用替代的编辑程序，也可以编辑图像或其他非文本资源。例如，如果您想修改存储在表字段中的 JPEG 图像，可以运行：

```sql
sqlite> UPDATE pics SET img=edit(img,'gimp') WHERE id='pic-1542';

```

编辑程序也可用作查看器，只需忽略返回值即可。例如，要仅查看上面的图像，您可以运行：

```sql
sqlite> SELECT length(edit(img,'gimp')) WHERE id='pic-1542';

```

## 8.5\. 将文件导入为 CSV 或其他格式

使用".import"命令将 CSV（逗号分隔值）或类似分隔的数据导入 SQLite 表格。".import"命令接受两个参数：数据来源和要插入数据的 SQLite 表格名称。数据来源可以是要读取的文件名称，或者如果以"|"字符开头，则指定一个将生成输入数据的命令。

在运行".import"命令之前，设置"mode"可能非常重要。这样做是为了防止命令行解释器尝试将输入文件文本解释为除文件结构外的其他格式。如果使用了--csv 或--ascii 选项，则它们会控制导入输入分隔符。否则，分隔符将按照当前输出模式的设置来确定。

要将数据导入到不在"main"模式下的表中，可以使用--schema 选项指定表位于其他模式中。这对于 ATTACH 的数据库或导入到 TEMP 表格非常有用。

当运行.import 时，其对第一个输入行的处理取决于目标表是否已存在。如果不存在，则自动创建表，并使用第一个输入行的内容设置表中所有列的名称。在这种情况下，表格数据内容来自第二行及其后续的输入行。如果目标表已存在，则将输入文件的每一行（包括第一行）视为实际数据内容。如果输入文件包含初始的列标签行，您可以使用"--skip 1"选项让.import 命令跳过该初始行。

这是一个示例用法，从包含列名的 CSV 文件中加载预先存在的临时表格的内容：

```sql
sqlite> .import --csv --skip 1 --schema temp C:/work/somedata.csv tab1

```

在读取非'ascii'模式的输入数据时，".import"将根据 RFC 4180 规范将输入解释为由字段组成的记录，但有一个例外：输入记录和字段分隔符由模式设置或使用.separator 命令设置。字段始终要经过引号删除，以撤消 RFC 4180 中执行的引号处理，但在 ascii 模式下除外。

要导入具有任意分隔符和无引号的数据，请先设置 ascii 模式（".mode ascii"），然后使用".separator"命令设置字段和记录分隔符。这将禁止去引号。在".import"时，数据将根据指定的分隔符拆分为字段和记录。

## 8.6\. 导出为 CSV

要将 SQLite 表格（或表格的一部分）导出为 CSV，只需将"mode"设置为"csv"，然后运行查询以提取表格中所需的行。输出将按照 RFC 4180 的 CSV 格式进行格式化。

```sql
sqlite> .headers on
sqlite> .mode csv
sqlite> .once c:/work/dataout.csv
sqlite> SELECT * FROM tab1;
sqlite> .system c:/work/dataout.csv

```

在上面的示例中，".headers on" 行会导致列标签被打印为输出的第一行。这意味着生成的 CSV 文件的第一行将包含列标签。如果不需要列标签，请设置 ".headers off"。 （".headers off" 设置是默认设置，如果未曾打开过标题，则可以省略。）

".once *FILENAME*" 行会导致所有查询输出写入指定文件，而不是在控制台上打印。在上述示例中，该行会将 CSV 内容写入名为 "C:/work/dataout.csv" 的文件中。

示例的最后一行（".system c:/work/dataout.csv"）具有与在 Windows 上双击 c:/work/dataout.csv 文件相同的效果。这通常会打开一个电子表格程序来显示 CSV 文件。

该命令仅在 Windows 上如原样写入才有效。在 Mac 上等效的命令行是：

```sql
sqlite> .system open dataout.csv

```

在 Linux 和其他 Unix 系统上，您需要输入类似以下内容：

```sql
sqlite> .system xdg-open dataout.csv

```

### 8.6.1\. 导出到 Excel

为了简化导出到电子表格，CLI 提供了 ".excel" 命令，它捕获单个查询的输出，并将该输出发送到主机计算机上的默认电子表格程序。使用方式如下：

```sql
sqlite> .excel
sqlite> SELECT * FROM tab;

```

上述命令会将查询的输出作为 CSV 写入一个临时文件，调用 CSV 文件的默认处理程序（通常是首选的电子表格程序，如 Excel 或 LibreOffice），然后删除临时文件。这本质上是执行上述 ".csv"、".once" 和 ".system" 命令序列的简便方法。

".excel" 命令实际上是 ".once -x" 的别名。".once" 的 -x 选项会将结果作为 CSV 写入一个临时文件，并以 ".csv" 后缀命名，然后调用系统的默认 CSV 文件处理程序。

也有一个 ".once -e" 命令，其工作方式类似，但是它会以 ".txt" 后缀命名临时文件，这样系统的默认文本编辑器将被调用，而不是默认的电子表格程序。

### 8.6.2\. 导出到 TSV（制表符分隔值）

在运行查询之前输入 ".mode tabs" 可以导出纯 TSV，不带任何字段引用。但是，如果输出包含双引号字符，在 tabs 模式下由 ".import" 命令读取时，输出将无法正确读取。为了按照 RFC 4180 对 TSV 进行引用，使其可以在 tabs 模式下由 ".import" 输入，首先输入 ".mode csv"，然后在运行查询之前输入 '.separator "\t"'。

# 9\. 访问 ZIP 归档作为数据库文件

除了读写 SQLite 数据库文件外，**sqlite3** 程序还可以读写 ZIP 归档文件。只需在初始命令行中指定 ZIP 归档文件名，或在 ".open" 命令中指定 ZIP 归档文件名，**sqlite3** 将自动检测文件是否为 ZIP 归档文件而不是 SQLite 数据库文件，并相应地打开它。这适用于任何文件后缀。因此，您可以打开 JAR、DOCX 和 ODP 文件，以及任何其他实际上是 ZIP 归档文件的文件格式，SQLite 都可以为您读取。

ZIP 归档看起来像一个包含以下架构的数据库：

```sql
CREATE TABLE zip(
  name,     // Name of the file
  mode,     // Unix-style file permissions
  mtime,    // Timestamp, seconds since 1970
  sz,       // File size after decompression
  rawdata,  // Raw compressed file data
  data,     // Uncompressed file content
  method    // ZIP compression method code
);

```

因此，例如，如果您想要查看 ZIP 归档文件中所有文件的压缩效率（表示为压缩内容大小与原始未压缩文件大小的比率），从最高压缩到最低压缩，您可以运行如下查询：

```sql
sqlite> SELECT name, (100.0*length(rawdata))/sz FROM zip ORDER BY 2;

```

或者使用 文件 I/O 函数，您可以提取 ZIP 归档文件的元素：

```sql
sqlite> SELECT writefile(name,content) FROM zip
   ...> WHERE name LIKE 'docProps/%';

```

## 9.1\. ZIP 归档访问的实现方式

命令行 shell 使用 Zipfile 虚拟表 来访问 ZIP 归档文件。当 ZIP 归档文件打开时，您可以通过运行 ".schema" 命令来查看：

```sql
sqlite> .schema
CREATE VIRTUAL TABLE zip USING zipfile('document.docx')
/* zip(name,mode,mtime,sz,rawdata,data,method) */;

```

当打开文件时，如果命令行客户端发现文件是 ZIP 归档文件而不是 SQLite 数据库，则会实际打开一个 内存数据库，然后在该内存数据库中创建与 ZIP 归档相关联的 Zipfile 虚拟表 实例。

打开 ZIP 归档文件的特殊处理是命令行 shell 的一个技巧，而不是核心 SQLite 库的一部分。因此，如果您想在应用程序中将 ZIP 归档文件作为数据库打开，您需要激活 Zipfile 虚拟表 模块，然后运行适当的 CREATE VIRTUAL TABLE 声明。

# 10\. 将整个数据库转换为文本文件

使用 ".dump" 命令将整个数据库内容转换为单个 UTF-8 文本文件。通过将其通过管道输入回 **sqlite3**，可以将此文件转换回数据库。

制作数据库的归档副本的好方法是：

```sql
$ sqlite3 ex1 .dump | gzip -c >ex1.dump.gz

```

这将生成名为 **ex1.dump.gz** 的文件，其中包含您重建数据库所需的所有内容，可以在以后的时间或其他机器上重建数据库。要重建数据库，只需键入：

```sql
$ zcat ex1.dump.gz | sqlite3 ex2

```

文本格式是纯 SQL，因此您也可以使用 .dump 命令将 SQLite 数据库导出到其他流行的 SQL 数据库引擎。例如：

```sql
$ createdb ex2
$ sqlite3 ex1 .dump | psql ex2

```

# 11\. 从损坏的数据库中恢复数据

与 ".dump" 命令类似，".recover" 尝试将数据库文件的整个内容转换为文本。不同之处在于，".recover" 尝试根据直接从尽可能多的数据库页面中提取的数据重新组装数据库，而不是使用正常的 SQL 数据库接口读取数据。如果数据库损坏，".recover" 通常能够从数据库的所有未损坏部分中恢复数据，而 ".dump" 在遇到第一个损坏迹象时停止。

如果 ".recover" 命令恢复了一个或多个无法归因于任何数据库表的行，则输出脚本将创建一个 "lost_and_found" 表来存储孤立的行。lost_and_found 表的模式如下所示：

```sql
CREATE TABLE lost_and_found(
    rootpgno INTEGER,             -- root page of tree pgno is a part of
    pgno INTEGER,                 -- page number row was found on
    nfield INTEGER,               -- number of fields in row
    id INTEGER,                   -- value of rowid field, or NULL
    c0, c1, c2, c3...             -- columns for fields of row
);

```

"lost_and_found" 表包含从数据库中恢复的每个孤立行的一行。此外，还有一行用于无法归因于任何 SQL 索引的每个恢复的索引条目。这是因为在 SQLite 数据库中，用于存储 SQL 索引条目和 WITHOUT ROWID 表条目的格式相同。

| 列 | 内容 |
| --- | --- |
| rootpgno | 即使可能无法将行归因于特定的数据库表，它可能是数据库文件中树结构的一部分。在这种情况下，该树结构的根页号存储在此列中。或者，如果找到行的页面不是树结构的一部分，则此列存储“pgno”列中值的副本 - 找到行的页面的页号。在许多情况下，虽然不是所有情况，lost_and_found 表中具有相同值的所有行都属于同一表。 |
| pgno | 找到该行的页面的页号。 |
| nfield | 此行中字段的数量。 |
| id | 如果行来自 WITHOUT ROWID 表，则此列包含 NULL。否则，它包含行的 64 位整数 rowid 值。 |
| c0, c1, c2... | 行的每个列的值存储在这些列中。".recover" 命令根据最长的孤立行所需的列数创建 lost_and_found 表。 |

如果恢复的数据库模式已经包含名为 "lost_and_found" 的表，则 ".recover" 命令使用名称 "lost_and_found0"。如果名称 "lost_and_found0" 也已经被使用，则使用 "lost_and_found1"，依此类推。可以通过以带 --lost-and-found 开关调用 ".recover" 来覆盖默认名称 "lost_and_found"。例如，要让输出脚本称表为 "orphaned_rows"：

```sql
sqlite> .recover --lost-and-found orphaned_rows

```

# 12\. 加载扩展

您可以通过 ".load" 命令在运行时向命令行 shell 中添加新的自定义 应用定义的 SQL 函数，排序序列，虚拟表，以及 VFS。首先，将扩展构建为 DLL 或共享库（如 运行时可加载扩展 文档中描述的那样），然后输入以下内容：

```sql
sqlite> .load /path/to/my_extension

```

请注意，SQLite 自动添加适当的扩展后缀（".dll" 在 Windows 上，".dylib" 在 Mac 上，大多数其他 Unix 系统上为 ".so"）到扩展文件名中。通常最好指定扩展的完整路径名。

SQLite 根据扩展文件名计算扩展的入口点。要覆盖此选择，只需将扩展名作为 ".load" 命令的第二个参数添加即可。

多个有用扩展的源代码可以在 SQLite 源代码树的[ext/misc](https://www.sqlite.org/src/tree?name=ext/misc&ci=trunk)子目录中找到。你可以直接使用这些扩展，或者以它们为基础创建你自己的定制扩展，以满足特定需求。

# 13\. 数据库内容的加密哈希

".sha3sum" dot-command 计算数据库内容的[SHA3](https://en.wikipedia.org/wiki/SHA-3)哈希。明确地说，哈希是在数据库内容上计算的，而不是在磁盘上它的表示上。这意味着，例如，VACUUM 或类似的数据保留转换不会更改哈希。

".sha3sum" 命令支持选项 "--sha3-224"、"--sha3-256"、"--sha3-384" 和 "--sha3-512"，用于定义要使用的 SHA3 的种类。默认为 SHA3-256。

数据库模式（在 sqlite_schema 表中）通常不包含在哈希中，但可以通过 "--schema" 选项添加。

".sha3sum" 命令接受一个可选参数，这个参数是一个 LIKE 模式。如果存在这个选项，只有表名与 LIKE 模式匹配的表会被哈希。

".sha3sum" 命令是通过命令行 shell 包含的[扩展函数 "sha3_query()"](https://www.sqlite.org/src/file/ext/misc/shathree.c)来实现的。

# 14\. 数据库内容自测

".selftest" 命令尝试验证数据库是否完整且未损坏。该命令查找名为 "selftest" 的模式中的表，并定义如下：

```sql
CREATE TABLE selftest(
  tno INTEGER PRIMARY KEY,  -- Test number
  op TEXT,                  -- 'run' or 'memo'
  cmd TEXT,                 -- SQL command to run, or text of "memo"
  ans TEXT                  -- Expected result of the SQL command
);

```

".selftest" 命令按照 selftest.tno 排序读取 selftest 表的行。对于每个 'memo' 行，它将 'cmd' 中的文本写入输出。对于每个 'run' 行，它运行 'cmd' 文本作为 SQL，并将结果与 'ans' 中的值进行比较，如果结果不同则显示错误消息。

如果没有 selftest 表，则 ".selftest" 命令运行 PRAGMA integrity_check。

".selftest --init" 命令如果不存在则创建 selftest 表，然后追加条目以检查所有表内容的 SHA3 哈希。随后的 ".selftest" 运行将验证数据库未被任何方式更改。要生成用于验证子集表未更改的测试，只需运行 ".selftest --init" 然后 DELETE selftest 行，这些行引用未被改变的表。

# 15\. SQLite 归档支持

".archive" 点命令和 "-A" 命令行选项为 SQLite Archive format 提供了内置支持。其接口类似于 Unix 系统上的 "tar" 命令。每次 ".ar" 命令的调用必须指定一个单一的命令选项。以下命令适用于 ".archive"：

| Option | Long Option | Purpose |
| --- | --- | --- |
| -c | --create | 创建包含指定文件的新存档。 |
| -x | --extract | 从存档中提取指定的文件。 |
| -i | --insert | 添加文件到现有存档。 |
| -r | --remove | 从存档中删除文件。 |
| -t | --list | 列出存档中的文件。 |
| -u | --update | 如果文件发生更改，则添加到现有存档中。 |

以及命令选项，每次调用 ".ar" 可能指定一个或多个修饰选项。某些修饰选项需要一个参数，某些则不需要。以下修饰选项可用：

| Option | Long Option | Purpose |
| --- | --- | --- |
| -v | --verbose | 处理每个文件时列出其名称。 |
| -f FILE | --file FILE | 如果指定了，使用文件 FILE 作为存档。否则，假定当前的 "main" 数据库是要操作的存档。 |
| -a FILE | --append FILE | 类似于 --file，使用文件 FILE 作为存档，但使用 [apndvfs VFS](https://sqlite.org/src/file/ext/misc/appendvfs.c) 打开文件，以便如果 FILE 已存在，则将存档追加到 FILE 的末尾。 |
| -C DIR | --directory DIR | 如果指定了，将所有相对路径解释为相对于 DIR 而不是当前工作目录。 |
| -g | --glob | 使用 glob(*Y*,*X*) 匹配存档中的名称与参数。 |
| -n | --dryrun | 显示将运行的 SQL 来执行存档操作，但实际上不改变任何内容。 |
| -- | -- | 所有后续的命令行单词都是命令参数，而不是选项。 |

对于命令行使用，立即在 "-A" 后添加短样式命令行选项，没有空格。所有后续参数都被视为 .archive 命令的一部分。例如，以下命令是等效的：

```sql
sqlite3 new_archive.db -Acv file1 file2 file3
sqlite3 new_archive.db ".ar -cv file1 file2 file3"

```

长式和短式选项可以混合使用。例如，以下是等效的：

```sql
*-- Two ways to create a new archive named "new_archive.db" containing*
*-- files "file1", "file2" and "file3".*
.ar -c --file new_archive.db file1 file2 file3
.ar -f new_archive.db --create file1 file2 file3

```

或者，".ar" 后的第一个参数可以是所有必需选项的短形式的连接（不带 "-" 字符）。在这种情况下，需要它们的选项的参数从命令行中读取，任何剩余的单词都被视为命令参数。例如：

```sql
*-- Create a new archive "new_archive.db" containing files "file1" and*
*-- "file2" from directory "dir1".*
.ar cCf dir1 new_archive.db file1 file2 file3

```

## 15.1\. SQLite Archive Create Command

创建一个新的存档，覆盖任何现有的存档（无论是在当前的 "main" 数据库中还是在由 --file 选项指定的文件中）。每个选项后面的参数都是要添加到存档中的文件。目录会递归导入。请参阅上面的示例。

## 15.2\. SQLite Archive Extract Command

从归档中提取文件（提取到当前工作目录或由 --directory 选项指定的目录）。与 --glob 选项影响的参数匹配的文件或目录将被提取。或者，如果选项后没有指定参数，则将提取所有文件和目录。如果在归档中找不到任何指定的名称或匹配模式，则会报错。

```sql
*-- Extract all files from the archive in the current "main" db to the*
*-- current working directory. List files as they are extracted.* 
.ar --extract --verbose

*-- Extract file "file1" from archive "ar.db" to directory "dir1".*
.ar fCx ar.db dir1 file1

*-- Extract files with ".h" extension to directory "headers".*
.ar -gCx headers *.h

```

## 15.3\. SQLite 归档列表命令

列出归档的内容。如果未指定任何参数，则会列出所有文件。否则，只列出与 --glob 选项影响的参数匹配的文件。当前，--verbose 选项不会改变此命令的行为。未来可能会改变。

```sql
*-- List contents of archive in current "main" db.*.
.ar --list

```

## 15.4\. SQLite 归档插入和更新命令

--update 和 --insert 命令的工作方式类似于 --create 命令，但它们在开始之前不会删除当前归档。文件的新版本将以静默方式替换同名的现有文件，但否则归档的初始内容（如果有）将保持不变。

对于 --insert 命令，将列出的所有文件插入到归档中。对于 --update 命令，仅当文件在归档中不存在，或者它们的 "mtime" 或 "mode" 与当前归档中的不同时才会插入。

兼容性注意事项：在 SQLite 版本 3.28.0（2019-04-16）之前，仅支持 --update 选项，但该选项的工作方式类似于 --insert，即始终重新插入每个文件，无论它们是否已更改。

## 15.5\. SQLite 归档删除命令

--remove 命令将删除与提供的参数（如果有）匹配的文件和目录，这受到 --glob 选项的影响。如果提供的参数与归档中的任何内容都不匹配，则会报错。

## 15.6\. ZIP 归档操作

如果 FILE 是一个 ZIP 归档文件而不是 SQLite 归档文件，则 ".archive" 命令和 "-A" 命令行选项仍然有效。这是通过 zipfile 扩展实现的。因此，以下命令在输出格式方面略有不同，但在功能上大致相当：

| 传统命令 | 等效的 sqlite3.exe 命令 |
| --- | --- |
| unzip archive.zip | sqlite3 -Axf archive.zip |
| unzip -l archive.zip | sqlite3 -Atvf archive.zip |
| zip -r archive2.zip dir | sqlite3 -Acf archive2.zip dir |

## 15.7\. 实现 SQLite 归档操作的 SQL

各种 SQLite 归档命令是使用 SQL 语句实现的。应用程序开发人员可以通过运行适当的 SQL 轻松为自己的项目添加 SQLite 归档的读写支持。

要查看用于实现 SQLite 归档操作的 SQL 语句，添加 --dryrun 或 -n 选项。这会显示 SQL 语句，但不会执行它们。

用于实现 SQLite 存档操作的 SQL 语句利用了各种可加载扩展。所有这些扩展都可以在[SQLite 源代码树](https://sqlite.org/src)中的[ext/misc/子文件夹](https://sqlite.org/src/file/ext/misc)找到。完全支持 SQLite 存档所需的扩展包括：

1.  [fileio.c](https://sqlite.org/src/file/ext/misc/fileio.c) — 这个扩展添加了 SQL 函数 readfile()和 writefile()，用于从磁盘上的文件读取和写入内容。fileio.c 扩展还包括了 fsdir()表值函数，用于列出目录内容，以及 lsmode()函数，用于将从 stat()系统调用中获取的数值 st_mode 整数转换为类似于"ls -l"命令的可读字符串。

1.  [sqlar.c](https://sqlite.org/src/file/ext/misc/sqlar.c) — 这个扩展添加了 sqlar_compress()和 sqlar_uncompress()函数，用于在插入和提取 SQLite 存档中的文件内容时进行压缩和解压缩操作。

1.  zipfile.c — 这个扩展实现了"zipfile(FILE)"表值函数，用于读取 ZIP 存档。仅在读取 ZIP 存档而不是 SQLite 存档时才需要此扩展。

1.  [appendvfs.c](https://sqlite.org/src/file/ext/misc/appendvfs.c) — 这个扩展实现了一个新的 VFS，允许将 SQLite 数据库附加到其他文件，例如可执行文件。仅在使用.archive 命令的--append 选项时才需要此扩展。

# 16\. SQL 参数

SQLite 允许在 SQL 语句中的任何允许字面值的地方使用绑定参数。这些参数的值可以使用 sqlite3_bind_...()系列 API 进行设置。

参数可以是命名或未命名的。未命名参数是一个问号("?")。命名参数是一个问号后面紧跟一个数字（例如"?15"或"?123"）或一个"$"、":"或"@"字符后面跟着一个字母数字名称（例如"$var1"、":xyz"、"@bingo"）。

此命令行 shell 将未命名的参数保持未绑定状态，意味着它们将具有 SQL NULL 的值，但命名参数可能会被赋值。如果存在一个命名为"sqlite_parameters"的 TEMP 表，其模式如下：

```sql
CREATE TEMP TABLE sqlite_parameters(
  key TEXT PRIMARY KEY,
  value
) WITHOUT ROWID;

```

如果在该表中有一条记录的键列与参数名称完全匹配（包括初始的"?"、"$"、":"或"@"字符），则参数被赋予值列的值。如果不存在条目，则参数默认为 NULL。

".parameter" 命令存在是为了简化管理此表。".parameter init" 命令（通常简写为".param init"）如果该表尚不存在，则创建 temp.sqlite_parameters 表。".param list" 命令显示 temp.sqlite_parameters 表中的所有条目。".param clear" 命令删除 temp.sqlite_parameters 表。".param set KEY VALUE" 和 ".param unset KEY" 命令分别在 temp.sqlite_parameters 表中创建或删除条目。

传递给".param set KEY VALUE"的 VALUE 可以是 SQL 字面量，也可以是任何其他可以评估以产生值的 SQL 表达式或查询。这允许设置不同类型的值。如果此类评估失败，则提供的 VALUE 将被引用并插入为文本。由于此类初始评估可能会根据 VALUE 内容成功或失败，获取文本值的可靠方法是用单引号括起来，以免受到上述命令尾部解析的保护。例如，（除非打算设置值为-1365）：

```sql
.parameter init
.parameter set @phoneNumber "'202-456-1111'"

```

请注意，双引号用于保护单引号，并确保引用文本被解析为一个参数。

temp.sqlite_parameters 表仅为命令行 Shell 提供参数值。temp.sqlite_parameter 表对使用 SQLite C 语言 API 直接运行的查询没有影响。预计各个应用程序将实施其自己的参数绑定。您可以在[命令行 Shell 源代码](https://sqlite.org/src/file/src/shell.c.in)中搜索"sqlite_parameters"，了解命令行 Shell 如何进行参数绑定，并将其用作实现的提示。

# 17\. 索引建议（SQLite 专家）

**注意：此命令属于实验性质。在未来某个时候，可能会删除该命令或修改其接口，不向后兼容。**

对于大多数非平凡的 SQL 数据库，性能的关键在于创建正确的 SQL 索引。在这个上下文中，“正确的 SQL 索引”意味着那些能够使应用程序需要优化的查询快速运行的索引。".expert" 命令可以通过建议可能有助于特定查询的索引来提供帮助，如果这些索引存在于数据库中的话。

首先发出".expert"命令，然后在单独的行上跟随 SQL 查询。例如，请考虑以下会话：

```sql
sqlite> CREATE TABLE x1(a, b, c);                  *-- Create table in database* 
sqlite> .expert
sqlite> SELECT * FROM x1 WHERE a=? AND b>?;        *-- Analyze this SELECT* 
CREATE INDEX x1_idx_000123a7 ON x1(a, b);

0|0|0|SEARCH TABLE x1 USING INDEX x1_idx_000123a7 (a=? AND b>?)

sqlite> CREATE INDEX x1ab ON x1(a, b);             *-- Create the recommended index* 
sqlite> .expert
sqlite> SELECT * FROM x1 WHERE a=? AND b>?;        *-- Re-analyze the same SELECT* 
(no new indexes)

0|0|0|SEARCH TABLE x1 USING INDEX x1ab (a=? AND b>?)

```

在上述例子中，用户创建了数据库模式（一个名为"x1"的单表），然后使用".expert"命令来分析查询，例如"SELECT * FROM x1 WHERE a=? AND b>?"。Shell 工具建议用户创建一个新索引（索引"x1_idx_000123a7"）并以 EXPLAIN QUERY PLAN 格式输出该查询将使用的计划。然后用户创建具有等效模式的索引，并再次对相同的查询运行分析。这次 Shell 工具不建议创建任何新索引，并输出 SQLite 将根据现有索引为查询使用的计划。

".expert" 命令接受以下选项：

| 选项 | 目的 |
| --- | --- |
| ‑‑verbose | 如果存在，为每个分析的查询输出更详细的报告。 |
| ‑‑sample PERCENT | 此参数默认为 0，导致".expert"命令仅基于查询和数据库架构推荐索引。这类似于 SQLite 查询规划器在用户未对数据库运行 ANALYZE 命令生成数据分布统计时为查询选择索引的方式。如果此选项传递了非零参数，则".expert"命令会为考虑的所有索引生成类似的数据分布统计，基于每个数据库表当前存储的 PERCENT 百分比的行数。对于具有不寻常数据分布的数据库，这可能会导致更好的索引推荐，特别是如果应用程序打算运行 ANALYZE。对于小型数据库和现代 CPU，通常没有理由不传递"--sample 100"。然而，对于大型数据库表来说，收集数据分布统计可能会很昂贵。如果操作太慢，请尝试传递更小的"--sample"选项值。 |

本节描述的功能可以通过[SQLite 专家扩展](https://www.sqlite.org/src/dir?ci=trunk&name=ext/expert)代码集成到其他应用程序或工具中。

使用 SQL 自定义函数制定的数据库架构，通过扩展加载机制提供的可能需要特殊配置以与.expert 功能一起工作。因为该功能使用额外的连接来实现其功能，所以这些自定义函数必须在这些额外的连接中提供。可以通过在自动加载静态链接扩展和持久加载扩展描述的扩展加载/使用选项来完成。

# 18\. 处理多个数据库连接

从版本 3.37.0（2021-11-27）开始，CLI 具有能力同时保持多个数据库连接处于打开状态。一次只能活动一个数据库连接。非活动连接仍然打开但处于空闲状态。

使用".connection"点命令（通常简写为".conn"）来查看数据库连接列表，并指示当前活动的连接。每个数据库连接由 0 到 9 之间的整数标识。（最多可以同时打开 10 个连接。）通过键入".conn"命令后跟其编号，可以切换到另一个数据库连接，如果不存在则创建。通过键入".conn close N"关闭数据库连接，其中 N 是连接号。

虽然底层的 SQLite 数据库连接是彼此完全独立的，但许多 CLI 设置（如输出格式）在所有数据库连接之间共享。因此，在一个连接中更改 输出模式 将会在所有连接中进行更改。另一方面，某些 dot-commands 如 .open 仅影响当前连接。

# 19\. 杂项扩展功能

CLI 是由几个 SQLite 扩展构建的，这些扩展不包含在 SQLite 库中。其中一些扩展提供了前面章节中未描述的功能，特别是：

+   UINT 排序序列将嵌入在文本中的无符号整数按其值及其他文本进行排序；

+   十进制算术由 decimal extension 提供；

+   generate_series() 表值函数；

+   base64() 和 base85() 函数用于将 blob 编码为 base64 或 base85 文本，或将相同的内容解码为 blob；

+   支持 POSIX 扩展正则表达式，绑定到 REGEXP 运算符。

# 20\. 其他 Dot Commands

命令行 shell 中还有许多其他的 dot-commands 可用。查看 ".help" 命令以获取 SQLite 特定版本和构建的完整列表。

# 21\. 在 shell 脚本中使用 sqlite3

在 shell 脚本中使用 sqlite3 的一种方法是使用 "echo" 或 "cat" 在文件中生成一系列命令，然后在重定向输入的同时调用 sqlite3。这种方法在许多情况下都很有效。但作为额外的便利，sqlite3 允许将单个 SQL 命令作为第二个参数输入到命令行中。当 sqlite3 程序以两个参数启动时，第二个参数将传递给 SQLite 库进行处理，查询结果将以列表模式打印到标准输出，并退出程序。此机制旨在使 sqlite3 在与 "awk" 等程序配合使用时更加方便。例如：

```sql
$ sqlite3 ex1 'select * from tbl1' \
>  | awk '{printf "<tr><td>%s<td>%s\n",$1,$2 }'
<tr><td>hello<td>10
<tr><td>goodbye<td>20
$

```

# 22\. 标记 SQL 语句的结束

SQLite 命令通常以分号结尾。在 CLI 中，你还可以使用单独一行上的单词 "GO"（大小写不敏感）或斜杠字符 "/" 来结束命令。这些分别被 SQL Server 和 Oracle 使用，并且 SQLite CLI 支持以保持兼容性。这些在 **sqlite3_exec()** 中不起作用，因为 CLI 在将它们传递到 SQLite 核心之前会将这些输入翻译成分号。

# 23\. 命令行选项

CLI 提供了许多命令行选项。使用 --help 命令行选项查看列表：

```sql
$ sqlite3 --help
Usage: ./sqlite3 [OPTIONS] FILENAME [SQL]
FILENAME is the name of an SQLite database. A new database is created
if the file does not previously exist. Defaults to :memory:.
OPTIONS include:
   --                   treat no subsequent arguments as options
   -A ARGS...           run ".archive ARGS" and exit
   -append              append the database to the end of the file
   -ascii               set output mode to 'ascii'
   -bail                stop after hitting an error
   -batch               force batch I/O
   -box                 set output mode to 'box'
   -column              set output mode to 'column'
   -cmd COMMAND         run "COMMAND" before reading stdin
   -csv                 set output mode to 'csv'
   -deserialize         open the database using sqlite3_deserialize()
   -echo                print inputs before execution
   -init FILENAME       read/process named file
   -[no]header          turn headers on or off
   -help                show this message
   -html                set output mode to HTML
   -interactive         force interactive I/O
   -json                set output mode to 'json'
   -line                set output mode to 'line'
   -list                set output mode to 'list'
   -lookaside SIZE N    use N entries of SZ bytes for lookaside memory
   -markdown            set output mode to 'markdown'
   -maxsize N           maximum size for a --deserialize database
   -memtrace            trace all memory allocations and deallocations
   -mmap N              default mmap size set to N
   -newline SEP         set output row separator. Default: '\n'
   -nofollow            refuse to open symbolic links to database files
   -nonce STRING        set the safe-mode escape nonce
   -nullvalue TEXT      set text string for NULL values. Default ''
   -pagecache SIZE N    use N slots of SZ bytes each for page cache memory
   -pcachetrace         trace all page cache operations
   -quote               set output mode to 'quote'
   -readonly            open the database read-only
   -safe                enable safe-mode
   -separator SEP       set output column separator. Default: '|'
   -stats               print memory stats before each finalize
   -table               set output mode to 'table'
   -tabs                set output mode to 'tabs'
   -unsafe-testing      allow unsafe commands and modes for testing
   -version             show SQLite version
   -vfs NAME            use NAME as the default VFS
   -zip                 open the file as a ZIP Archive

```

CLI 在命令行选项格式上非常灵活。允许使用一个或两个前导 "-" 字符。因此，"-box" 和 "--box" 表示相同的意思。命令行选项从左向右处理。因此，"--box" 选项将覆盖先前的 "--quote" 选项。

大多数命令行选项都不言自明，但有几个值得在下面进一步讨论。

## 23.1\. --safe 命令行选项

--safe 命令行选项尝试禁用 CLI 的所有可能导致除指定在命令行中命名的特定数据库文件以外的主机计算机更改的功能。其想法是，如果您从一个未知或不信任的源接收到一个大型 SQL 脚本，您可以运行该脚本以查看其操作，而不用担心通过使用--safe 选项来避免被利用。--safe 选项禁用（除其他外）：

+   .open 命令，除非使用--hexdb 选项或文件名为":memory:"。这可以防止脚本读取或写入未在原始命令行中命名的任何数据库文件。

+   ATTACH SQL 命令。

+   可能具有有害副作用的 SQL 函数，如 edit()、fts3_tokenizer()、load_extension()、readfile()和 writefile()。

+   .archive 命令。

+   .backup 和.save 命令。

+   .import 命令。

+   .load 命令。

+   .log 命令。

+   .shell 和.system 命令。

+   .excel、.once 和.output 命令。

+   其他可能具有有害副作用的命令。

基本上，CLI 的任何读取或写入除主数据库文件以外的磁盘文件的功能都将被禁用。

### 23.1.1\. 为特定命令绕过--safe 限制

如果命令行中还包括"--nonce NONCE"选项，其中 NONCE 是一个大而随意的字符串，那么带有相同大随机数的".nonce NONCE"命令将允许下一个 SQL 语句或点命令绕过--safe 限制。

假设您想运行一个可疑的脚本，并且该脚本需要--safe 通常禁用的一个或两个功能。例如，假设它需要附加一个额外的数据库。或者假设脚本需要加载一个特定的扩展。可以通过在适当的".nonce"命令之前放置（经过仔细审核的）ATTACH 语句或".load"命令，并使用"--nonce"命令行选项提供相同的 nonce 值来实现这一点。这些特定命令将允许正常执行，但所有其他不安全的命令仍将受到限制。

在某种意义上，使用".nonce"是危险的，因为一个错误可能允许敌意脚本损坏您的系统。因此，在没有其他方法可以在--safe 模式下运行脚本时，请谨慎、节约地使用".nonce"，并作为最后的手段。

## 23.2\. --unsafe-testing 命令行选项

--unsafe-testing 命令行选项支持用于 SQLite 库的内部测试 CLI。它对于创建、修改或查询 SQLite 数据库的实用程序使用不需要也不有用。其预期用途是允许使用直接模式测试的脚本化测试，防御措施被打败，以及启用某些特殊用途、未记录的、面向测试的点命令。

通常情况下，需要通过 --unsafe-testing 选项诱导的行为通常不会单独因此原因被视为错误。CLI 在 --unsafe-testing 下的行为不受支持或定义。

## 23.3\. --no-utf8 和 --utf8 命令行选项

在 Windows 平台上，当控制台用于输入或输出时，需要在可从控制台获得或发送的字符编码与 CLI 的内部 UTF-8 文本表示之间进行转换。CLI 的早期版本接受这些选项，以启用或禁用依赖于 Windows 控制台功能的转换，在现代 OS 版本上它能够产生或接受 UTF-8。

现有的 CLI 版本（3.44.1 或更高版本）通过从/向 Windows 控制台 API 读取或写入 UTF-16 来进行控制台 I/O。即使在支持 Windows 2000 的 Windows 版本上也能正常运行，因此不再需要这些选项。它们仍然被接受，但没有效果。

在所有情况下，非控制台文本 I/O 均为 UTF-8 编码。

在非 Windows 平台上，这些选项也将被忽略。

# 24\. 从源代码编译 sqlite3 程序

要在 Unix 系统和 Windows 的 MinGW 上编译命令行 shell，通常的 configure-make 命令适用：

```sql
sh configure; make

```

无论您是从源树的规范源代码构建，还是从集成包构建，都可以使用 configure-make 进行工作。依赖关系很少。在从规范源代码构建时，需要一个可用的 [tclsh](https://www.tcl.tk/man/tcl8.3/UserCmd/tclsh.htm)。如果使用集成包，所有通常由 tclsh 完成的预处理工作将已经完成，只需要常规的构建工具。

为了使 .archive 命令 正常运行，需要一个可用的 [zlib 压缩库](https://zlib.net)。

在 Windows 上使用 MSVC，使用 Makefile.msc 的 nmake：

```sql
nmake /f Makefile.msc

```

为了正确运行 .archive 命令，请将 [zlib 源代码](https://zlib.net) 的副本复制到源代码树的 compat/zlib 子目录中，并以以下方式进行编译：

```sql
nmake /f Makefile.msc USE_ZLIB=1

```

## 24.1\. 自行构建

SQLite3 命令行界面的源代码位于名为 "shell.c" 的单个文件中。shell.c 源文件是从其他源文件生成的，但是大部分 shell.c 的代码可以在 [src/shell.c.in](https://sqlite.org/src/file/src/shell.c.in) 找到。（通过在规范源代码树中输入 "make shell.c" 来重新生成 shell.c。）编译 shell.c 文件（连同 sqlite3 库源代码）以生成可执行文件。例如：

```sql
gcc -o sqlite3 shell.c sqlite3.c -ldl -lpthread -lz -lm

```

建议使用以下额外的编译时选项以提供全功能的命令行 shell：

+   -DSQLITE_THREADSAFE=0

+   -DSQLITE_ENABLE_EXPLAIN_COMMENTS

+   -DSQLITE_HAVE_ZLIB

+   -DSQLITE_INTROSPECTION_PRAGMAS

+   -DSQLITE_ENABLE_UNKNOWN_SQL_FUNCTION

+   -DSQLITE_ENABLE_STMTVTAB

+   -DSQLITE_ENABLE_DBPAGE_VTAB

+   -DSQLITE_ENABLE_DBSTAT_VTAB

+   -DSQLITE_ENABLE_OFFSET_SQL_FUNC

+   -DSQLITE_ENABLE_JSON1

+   -DSQLITE_ENABLE_RTREE

+   -DSQLITE_ENABLE_FTS4

+   -DSQLITE_ENABLE_FTS5
