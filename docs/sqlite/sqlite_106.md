# SQLite 的虚拟数据库引擎

> 原文：[`sqlite.com/vdbe.html`](https://sqlite.com/vdbe.html)
> 
> **过时文档警告：**本文档描述了 SQLite 版本 2.8.0 中使用的虚拟机。SQLite 版本 3.0 和 3.1 中的虚拟机在概念上类似，但现在是基于寄存器而不是基于堆栈的，每个操作码有五个操作数而不是三个，并且具有与下文所示不同的操作码集。请参阅虚拟机指令文档，了解当前的 VDBE 操作码集及其简要操作概述。本文档作为历史参考而保留。

如果您想了解 SQLite 库的内部工作原理，您需要从对虚拟数据库引擎或 VDBE 的扎实理解开始。VDBE 出现在处理流程的中间位置（参见架构图），因此似乎涉及到库的大部分部分。即使不直接与 VDBE 交互的代码部分通常也起支持作用。VDBE 确实是 SQLite 的核心。

本文简要介绍了 VDBE 的工作原理，特别是各种 VDBE 指令（在此处文档化）如何共同处理数据库以执行有用操作。风格为教程，从简单任务开始，逐步解决更复杂的问题。在此过程中，我们将访问 SQLite 库中的大多数子模块。完成本教程后，您应该对 SQLite 的工作原理有相当好的理解，并准备开始研究实际的源代码。

## 初步工作

VDBE 实现了一个虚拟计算机，以其虚拟机语言运行程序。每个程序的目标是查询或修改数据库。为此，VDBE 实现的机器语言专门设计用于搜索、读取和修改数据库。

VDBE 语言的每条指令包含一个操作码和三个操作数标签为 P1、P2 和 P3。操作数 P1 是任意整数。P2 是非负整数。P3 是指向数据结构或零结尾字符串的指针，可能为空。只有少数 VDBE 指令使用所有三个操作数。许多指令仅使用一个或两个操作数。大部分指令根本不使用操作数，而是在执行堆栈上接收其数据并存储其结果。每条指令的具体操作和使用的操作数在单独的操作码描述文档中有详细描述。

VDBE 程序从指令 0 开始执行，并继续执行直到遇到致命错误、执行 Halt 指令或将程序计数器推进到程序的最后一条指令之后。当 VDBE 完成执行时，所有打开的数据库游标都将关闭，所有内存将被释放，并且堆栈中的所有内容都将被弹出。因此，不必担心内存泄漏或未分配的资源。

如果您有过汇编语言编程经验或之前使用过任何抽象机器，那么所有这些细节对您来说应该很熟悉。所以让我们直接开始看一些代码。

## 将记录插入数据库

我们首先解决一个可以用几条指令解决的 VDBE 程序问题。假设我们有一个像这样创建的 SQL 表：

> ```sql
> CREATE TABLE examp(one text, two int);
> 
> ```

简单来说，我们有一个名为“examp”的数据库表，其中包含名为“one”和“two”的两列数据。现在假设我们想向此表中插入一条记录。如下所示：

> ```sql
> INSERT INTO examp VALUES('Hello, World!',99);
> 
> ```

我们可以通过**sqlite**命令行实用程序查看 SQLite 用来实现这个 INSERT 的 VDBE 程序。首先在一个新的空数据库上启动**sqlite**，然后创建表。接下来通过输入".explain"命令将**sqlite**的输出格式更改为与 VDBE 程序转储配合使用的形式。最后，在上面显示的[INSERT]语句前加上特殊关键字[EXPLAIN]。[EXPLAIN]关键字将导致**sqlite**打印 VDBE 程序而不是执行它。我们有：

> `$ **sqlite test_database_1** sqlite>

如上所示，我们的简单插入语句由 12 条指令实现。前 3 条和后 2 条指令是标准的序言和结尾，因此实际工作在中间的 7 条指令中完成。没有跳转，因此程序从头到尾执行一次。现在让我们详细看看每条指令。

> `0     Transaction   0      0                                          1     VerifyCookie  0      81                                         2     Transaction   1      0`

指令 Transaction 开始一个事务。遇到 Commit 或 Rollback 操作码时事务结束。P1 是启动事务的数据库文件的索引。索引 0 是主数据库文件。启动事务时会获得数据库文件的写锁。在事务进行期间，没有其他进程可以读取或写入该文件。开始事务还会创建一个回滚日志。在对数据库进行任何更改之前必须启动事务。

指令 VerifyCookie 检查 cookie 0（数据库模式版本）是否等于 P2（最后读取数据库模式时的值）。P1 是数据库编号（主数据库为 0）。这是为了确保数据库模式没有被其他线程更改，如果被更改则需要重新读取。

第二个 Transaction 指令开始一个事务，并为数据库 1 开始一个回滚日志，该数据库用于临时表使用。

> `3     Integer       0      0                                     4     OpenWrite     0      3      examp`

指令 Integer 将整数值 P1 (0) 推入堆栈。这里的 0 是在接下来的 OpenWrite 指令中要使用的数据库编号。如果 P3 不为 NULL，则它是相同整数的字符串表示。之后堆栈的情况如下：

> | (integer) 0 |
> | --- |

指令 OpenWrite 在表 "examp" 上打开一个新的读/写游标，其句柄为 P1（在本例中为 0），该表的根页面为 P2（在这个数据库文件中为 3）。游标句柄可以是任何非负整数，但 VDBE 会在一个数组中分配游标，数组的大小比最大游标大一。因此，为了节省内存，最好从零开始顺序递增地使用句柄。在这里，P3（"examp"）是要打开的表的名称，但是这是未使用的，仅用于使代码更易于阅读。此指令从堆栈顶部弹出要使用的数据库编号（主数据库为 0），因此执行后堆栈再次为空。

> `5     NewRecno      0      0`

指令 NewRecno 为游标 P1 指向的表创建一个新的整数记录编号。记录编号是表中当前未用作键的一个。新的记录编号被推入堆栈。执行后堆栈如下所示：

> | (integer) new record key |
> | --- |
> 
> `6     String        0      0      Hello, World!`

指令 String 将其 P3 操作数推入堆栈。执行后堆栈如下所示：

> | (string) "Hello, World!" |
> | --- |
> | (integer) new record key |
> 
> `7     Integer       99     0      99`

指令 Integer 将其 P1 操作数 (99) 推入堆栈。执行后堆栈如下所示：

> | (integer) 99 |
> | --- |
> | (string) "Hello, World!" |
> | (integer) new record key |
> 
> `8     MakeRecord    2      0`

指令 MakeRecord 弹出堆栈顶部的 P1 元素（在本例中为 2 个），并将它们转换为数据库文件中存储记录的二进制格式。新生成的记录由 MakeRecord 指令推回到堆栈上。执行后堆栈如下所示：

> | (record) "Hello, World!", 99 |
> | --- |
> | （整数）新记录键 |
> 
> `9     PutIntKey     0      1`

指令 PutIntKey 使用栈顶的两个条目将一个条目写入由游标 P1 指向的表中。如果条目不存在，则创建新条目；如果存在，则覆盖现有条目的数据。记录数据是栈顶条目，键是下面的条目。该指令使堆栈弹出两次。由于操作数 P2 为 1，行更改计数增加，并且行 ID 被存储，以便稍后由 sqlite_last_insert_rowid()函数返回。如果 P2 为 0，则行更改计数不会改变。这条指令实际上是插入操作的地方。

> `10    Close         0      0`

指令 Close 关闭先前作为 P1（0，唯一打开的游标）打开的游标。如果 P1 当前未打开，则此指令无效。

> `11    Commit        0      0`

指令 Commit 导致自上次事务以来对数据库的所有修改实际生效。在启动另一个事务之前，不允许进行其他修改。Commit 指令删除日志文件并释放对数据库的写锁。如果仍然有游标打开，则继续保持读锁。

> `12    Halt          0      0`

指令 Halt 使 VDBE 引擎立即退出。所有打开的游标、列表、排序等都将自动关闭。P1 是由 sqlite_exec()返回的结果代码。对于正常的停止，这应该是 SQLITE_OK（0）。对于错误，可能是其他值。只有在有错误时才使用操作数 P2。每个程序的末尾都有一个隐含的"Halt 0 0 0"指令，VDBE 在准备程序运行时附加它。

## 追踪 VDBE 程序执行

如果 SQLite 库在编译时没有使用 NDEBUG 预处理宏，则 PRAGMA vdbe_trace 会导致 VDBE 追踪程序的执行。尽管此功能最初用于测试和调试，但也可用于了解 VDBE 的操作方式。使用 "`PRAGMA vdbe_trace=ON;`" 开启追踪，并使用 "`PRAGMA vdbe_trace=OFF`" 关闭追踪。示例如下：

> `sqlite> **PRAGMA vdbe_trace=ON;**    0 Halt            0    0 sqlite> **INSERT INTO examp VALUES('Hello, World!',99);**    0 Transaction     0    0    1 VerifyCookie    0   81    2 Transaction     1    0    3 Integer         0    0 栈：i:0    4 OpenWrite       0    3 examp    5 NewRecno        0    0 栈：i:2    6 String          0    0 Hello, World! 栈：t[Hello,.World!] i:2    7 Integer        99    0 99 栈：si:99 t[Hello,.World!] i:2    8 MakeRecord      2    0 栈：s[...Hello,.World!.99] i:2    9 PutIntKey       0    1   10 Close           0    0   11 Commit          0    0   12 Halt            0    0`

在追踪模式下，VDBE 在执行每条指令之前打印该指令。执行指令后，显示栈顶部几个条目。如果栈为空，则省略栈显示。

在堆栈显示中，大多数条目都带有前缀，用于指示该堆栈条目的数据类型。整数以 "`i:`" 开头。浮点数值以 "`r:`" 开头（其中 "r" 表示 "real-number"）。字符串以 "`s:`"、"`t:`"、"`e:`" 或 "`z:`" 开头。字符串前缀的区别是由它们的内存分配方式引起的。`z:` 字符串存储在通过 **malloc()** 获得的内存中。`t:` 字符串是静态分配的。`e:` 字符串是短暂的。所有其他字符串都带有 `s:` 前缀。这对于观察者（你）并无差别，但对于 VDBE 来说非常重要，因为 `z:` 字符串在弹出时需要传递给 **free()** 以避免内存泄漏。注意，只显示字符串值的前 10 个字符，并且二进制值（如 MakeRecord 指令的结果）被视为字符串。VDBE 堆栈上唯一可以存储的其他数据类型是 NULL，它以简单的 "`NULL`" 形式显示，没有前缀。如果整数既以整数形式又以字符串形式放置在堆栈上，则其前缀为 "`si:`"。

## 简单查询

现在，你应该了解 VDBE 如何向数据库写入的基础知识了。现在让我们看看它如何执行查询。我们将使用以下简单的 SELECT 语句作为示例：

> ```sql
> SELECT * FROM examp;
> 
> ```

为此 SQL 语句生成的 VDBE 程序如下所示：

> `sqlite> **EXPLAIN SELECT * FROM examp;** addr  opcode        p1     p2     p3                                  ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      one                                 1     ColumnName    1      0      two                                 2     Integer       0      0                                          3     OpenRead      0      3      examp                               4     VerifyCookie  0      81                                         5     Rewind        0      10                                         6     Column        0      0                                          7     Column        0      1                                          8     Callback      2      0                                          9     Next          0      6                                          10    Close         0      0                                          11    Halt          0      0`

在我们开始查看这个问题之前，让我们简要回顾一下 SQLite 中查询的工作方式，这样我们就会知道我们要达成什么目标。对于查询结果中的每一行，SQLite 将调用一个具有以下原型的回调函数：

> ```sql
> int Callback(void *pUserData, int nColumn, char *azData[], char *azColumnName[]);
> 
> ```

SQLite 库向 VDBE 提供了指向回调函数和 **pUserData** 指针。（回调函数和用户数据最初作为参数传递给 **sqlite_exec()** API 函数。）VDBE 的任务是为 **nColumn**、**azData[]** 和 **azColumnName[]** 提供值。**nColumn** 当然是结果中的列数。**azColumnName[]** 是一个字符串数组，其中每个字符串是一个结果列的名称。**azData[]** 是一个字符串数组，其中包含实际数据。

> `0     ColumnName    0      0      one                                 1     ColumnName    1      0      two`

对于我们查询的 VDBE 程序中的前两条指令，主要是设置**azColumn**的值。ColumnName 指令告诉 VDBE 要为**azColumnName[]**数组的每个元素填入什么值。每个查询都将以一个 ColumnName 指令开始，对应于结果中的每一列，而后面的每个查询中将会有一个匹配的 Column 指令。

> `2     Integer       0      0                                          3     OpenRead      0      3      examp                               4     VerifyCookie  0      81`

指令 2 和 3 在数据库表上打开了一个读取游标，这与插入示例中的 OpenWrite 指令相同，不同之处在于这次游标是用于读取的而不是写入。指令 4 验证了数据库模式，就像插入示例中那样。

> `5     Rewind        0      10`

倒带指令初始化一个循环，迭代“examp”表。它将游标 P1 倒回到其表中的第一个条目。这是列和下一步指令所必需的，它们使用游标来遍历表格。如果表格为空，则跳到 P2（10），即循环结束后的指令。如果表格不为空，则顺序执行以下指令 6，这是循环体的开始。

> `6     Column        0      0                                          7     Column        0      1                                          8     Callback      2      0`

指令 6 到 8 形成了循环体，每个指令对数据库文件中的每条记录执行一次。地址为 6 和 7 的 Column 指令分别从游标 P1 处获取第 P2 列，并将其推送到堆栈上。在这个例子中，第一个 Column 指令将列“one”的值推送到堆栈上，第二个 Column 指令则是将列“two”的值推送上去。地址为 8 的 Callback 指令调用了 callback() 函数。Callback 的操作数 P1 成为了**nColumn**的值。Callback 指令从堆栈中弹出 P1 个值，并用它们填充**azData[]**数组。

> `9     Next          0      6`

指令 9 实现了循环的分支部分。与地址 5 处的 Rewind 一起，形成了循环逻辑。这是一个你应该特别注意的关键概念。Next 指令将游标 P1 推进到下一条记录。如果游标推进成功，立即跳转到 P2（6，循环体的开始）。如果游标已经到达末尾，则继续执行下一条指令，结束循环。

> `10    Close         0      0                                          11    Halt          0      0`

程序末尾的 Close 指令关闭了指向表“examp”的游标。在程序停止时，VDBE 会自动关闭所有游标，因此其实不需要在此处调用 Close。但我们需要一个 Rewind 跳转到的指令，所以我们可以让该指令做一些实际有用的事情。Halt 指令结束了 VDBE 程序。

请注意，此 SELECT 查询的程序中未包含用于 INSERT 示例中的 Transaction 和 Commit 指令。因为 SELECT 是一个读操作，不会改变数据库，所以不需要事务。

## 稍微复杂一点的查询

前一个示例的关键点在于使用 Callback 指令调用回调函数，并使用 Next 指令实现对数据库文件所有记录的循环。这个示例试图通过展示一个稍微复杂的查询来进一步阐明这些思想，该查询涉及更多的输出列，其中一些是计算值，并且有一个 WHERE 子句来限制哪些记录实际上传递到回调函数。考虑以下查询：

> ```sql
> SELECT one, two, one || two AS 'both'
> FROM examp
> WHERE one LIKE 'H%'
> 
> ```

这个查询可能有些牵强，但它确实能够说明我们的观点。结果将有三列，名称分别为"one"、"two"和"both"。前两列直接复制自表格中的两列，第三列的结果是通过连接表格的第一列和第二列形成的字符串。最后，WHERE 子句表示我们只选择以"H"开头的行作为结果。下面是此查询的 VDBE 程序示例：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      one 1     ColumnName    1      0      two 2     ColumnName    2      0      both 3     Integer       0      0 4     OpenRead      0      3      examp 5     VerifyCookie  0      81 6     Rewind        0      18 7     String        0      0      H%                                       8     Column        0      0 9     Function      2      0      ptr(0x7f1ac0) 10    IfNot         1      17 11    Column        0      0 12    Column        0      1 13    Column        0      0 14    Column        0      1 15    Concat        2      0 16    Callback      3      0 17    Next          0      7 18    Close         0      0 19    Halt          0      0`

除了 WHERE 子句之外，此示例的程序结构与先前的示例非常相似，只是多了一列。现在有 3 列，而不是之前的 2 列，并且有三个 ColumnName 指令。使用 OpenRead 指令打开游标，就像先前的示例一样。地址 6 处的 Rewind 指令和地址 17 处的 Next 形成了对表中所有记录的循环。末尾的 Close 指令是为了在完成时为 Rewind 指令提供跳转目标。所有这些都与第一个查询演示中的情况完全相同。

在这个例子中，回调指令需要生成三个结果列的数据，而不是两个，但除此之外与第一个查询中的相同。当调用回调指令时，结果的最左列应为堆栈中的最低值，而最右列应为堆栈的顶部值。我们可以看到在地址 11 到 15 之间堆栈被设置成这种方式。地址 11 和 12 处的列指令推送结果的前两列的值。地址 13 和 14 处的两个列指令获取计算第三个结果列所需的值，地址 15 处的 Concat 指令将它们连接成堆栈上的单个条目。

当前示例真正新的是实现 WHERE 子句的部分，其指令在地址 7 到 10 中实现。地址 7 和 8 的指令将表中的 "one" 列的值和字面字符串 "H%" 推送到堆栈上。地址 9 的 Function 指令从堆栈中弹出这两个值，并将 LIKE() 函数的结果再次推送到堆栈上。地址 10 的 IfNot 指令弹出堆栈的顶部值，并且如果顶部值为假（*不*像字面字符串 "H%" ），则立即跳转到下一条指令。执行这一跳转会有效地跳过回调，这正是 WHERE 子句的意义所在。如果比较的结果为真，则不执行跳转，控制将流转到下面的回调指令。

注意 LIKE 操作符的实现方式。在 SQLite 中，它是一个用户定义的函数，因此其函数定义的地址在 P3 中指定。操作数 P1 是从堆栈中取出的参数数量。在这种情况下，LIKE() 函数需要 2 个参数。参数是以逆序（从右到左）从堆栈中取出的，因此要匹配的模式是堆栈顶部的元素，下一个元素是要比较的数据。返回值被推送到堆栈上。

## SELECT 程序的模板

前两个查询示例展示了每个 SELECT 程序都会遵循的一种模板。基本上，我们有：

1.  初始化回调的 **azColumnName[]** 数组。

1.  打开一个游标进入要查询的表中。

1.  对于表中的每条记录，执行：

    1.  如果 WHERE 子句评估为 FALSE，则跳过后续步骤并继续到下一条记录。

    1.  计算结果的当前行的所有列。

    1.  调用当前结果行的回调函数。

1.  关闭游标。

随着我们考虑到诸如连接、复合选择、使用索引加速搜索、排序以及带有和不带有 GROUP BY 和 HAVING 子句的聚合函数等额外复杂情况，此模板将被大幅扩展。但是相同的基本思想仍然适用。

## 更新和删除语句

更新和删除语句使用的模板与 SELECT 语句模板非常相似。当然，主要区别在于最终的动作是修改数据库而不是调用回调函数。因为它修改数据库，所以也会使用事务。让我们从查看 DELETE 语句开始：

> ```sql
> DELETE FROM examp WHERE two<50;
> 
> ```

这条 DELETE 语句将从“examp”表中删除所有“two”列小于 50 的记录。生成的代码如下：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0 1     Transaction   0      0 2     VerifyCookie  0      178 3     Integer       0      0 4     OpenRead      0      3      examp 5     Rewind        0      12 6     Column        0      1 7     Integer       50     0      50 8     Ge            1      11 9     Recno         0      0 10    ListWrite     0      0 11    Next          0      6 12    Close         0      0 13    ListRewind    0      0 14    Integer       0      0 15    OpenWrite     0      3 16    ListRead      0      20 17    NotExists     0      19 18    Delete        0      1 19    Goto          0      16 20    ListReset     0      0 21    Close         0      0 22    Commit        0      0 23    Halt          0      0`

程序必须执行以下操作。首先，它必须定位表 "examp" 中所有将被删除的记录。这通过一个循环来完成，其结构与上述 SELECT 示例中使用的循环非常相似。一旦找到所有记录，我们就可以逐个删除它们。请注意，我们不能在找到记录后立即删除每条记录。我们必须先找到所有记录，然后再进行删除。这是因为 SQLite 数据库后端在删除操作后可能会改变扫描顺序。如果扫描顺序在扫描中间改变，某些记录可能会被多次访问，而其他记录可能根本不会被访问。

因此，DELETE 的实现实际上分为两个循环。第一个循环（指令 5 到 11）定位将要删除的记录，并将它们的键保存到一个临时列表中，第二个循环（指令 16 到 19）使用键列表逐个删除记录。

> `0     Transaction   1      0 1     Transaction   0      0 2     VerifyCookie  0      178 3     Integer       0      0 4     OpenRead      0      3      examp`

指令 0 到 4 与 INSERT 示例中一样。它们为主数据库和临时数据库启动事务，验证主数据库的数据库模式，并在 "examp" 表上打开读取游标。请注意，游标仅用于读取，而不是写入。在程序的这个阶段，我们只会扫描表，而不会修改它。我们将在指令 15 后重新打开相同的表进行写入。

> `5     Rewind        0      12`

如同 SELECT 示例中一样，倒带 指令将游标倒回表的开头，为循环体做好准备。

> `6     Column        0      1 7     Integer       50     0      50 8     Ge            1      11`

WHERE 子句由指令 6 到 8 实现。WHERE 子句的作用是在“two”列（由列指令提取）大于或等于 50 时跳过 ListWrite。为此，它会跳转到 Next 指令。

与之前一样，Column 指令使用游标 P1，并将列 P2（1，列“two”）的数据记录推送到堆栈中。Integer 指令将值 50 推送到堆栈的顶部。执行完这两个指令后，堆栈如下所示：

> | (整数) 50 |
> | --- |
> | (记录) 列“two”的当前记录 |

Ge 运算符比较堆栈顶部的两个元素，将它们弹出，并根据比较结果进行分支。如果第二个元素大于等于顶部元素，则跳转到地址 P2（循环结束时的 Next 指令）。因为 P1 为真，如果任一操作数为 NULL（因此结果为 NULL），则执行跳转。如果不跳转，则继续执行下一条指令。

> `9     Recno         0      0 10    ListWrite     0      0`

Recno 指令将整数推送到堆栈中，这个整数是指针 P1 指向的表的当前条目的键的前 4 个字节。ListWrite 指令将堆栈顶部的整数写入临时存储列表，并弹出顶部元素。这是循环的重要工作，用于存储待删除记录的键，以便在第二个循环中删除它们。执行完这个 ListWrite 指令后，堆栈再次为空。

> `11    Next          0      6 12    Close         0      0`

下一条指令将游标增加到由游标 P0 指向的表中的下一个元素，并且如果成功则跳转到 P2（即循环体的开始）。Close 指令关闭了游标 P1。它不影响临时存储列表，因为它与游标 P1 无关；相反，它是一个全局的工作列表（可以通过 ListPush 保存）。

> `13    ListRewind    0      0`

ListRewind 指令将临时存储列表倒回到开始位置，为第二个循环做准备。

> `14    Integer       0      0 15    OpenWrite     0      3`

就像 INSERT 示例中一样，我们将数据库编号 P1（主数据库，编号为 0）推送到堆栈上，并使用 OpenWrite 打开表 P2（基页 3，"examp"）上的游标 P1 以进行修改。

> `16    ListRead      0      20 17    NotExists     0      19 18    Delete        0      1 19    Goto          0      16`

这个循环执行实际的删除操作。它的组织方式与 UPDATE 示例中的循环不同。ListRead 指令在 INSERT 循环中扮演 Next 指令的角色，但由于在失败时跳转到 P2，而 Next 在成功时跳转，所以我们将其放在循环的开始而不是结束。这意味着我们必须在循环末尾放置一个 Goto 来跳回循环测试的开始。因此，这个循环的形式类似于 C 语言的 while(){...} 循环，而 INSERT 示例中的循环形式类似于 do{...}while() 循环。Delete 指令填补了前面示例中回调函数的角色。

ListRead 指令从临时存储列表中读取一个元素并将其推送到堆栈上。如果成功，则继续执行下一条指令。如果因为列表为空而失败，则跳转到 P2，这是循环后的第一条指令。之后，堆栈的状态如下：

> | （整数）当前记录的键 |
> | --- |

注意 ListRead 和 Next 指令之间的相似性。这两个操作都遵循这个规则：

> 将下一个“thing”推送到栈上并穿过或者跳转到 P2，具体取决于是否有下一个“thing”可以推送。

Next 和 ListRead 之间的一个区别在于它们对“thing”的理解。Next 指令中的“things”是数据库文件中的记录。“Things” 对于 ListRead 来说是列表中的整数键。另一个区别是如果没有下一个“thing”时是跳转还是穿过。在这种情况下，Next 是穿过，而 ListRead 是跳转。稍后我们将看到其他使用相同原则的循环指令（NextIdx 和 SortNext）。

不存在 指令弹出栈顶的元素，并将其用作整数键。如果表 P1 中不存在具有该键的记录，则跳转到 P2。如果记录存在，则继续执行下一个指令。在这种情况下，P2 让我们跳到循环结尾的 Goto 处，它会跳回到循环开始的 ListRead。这本可以编码为将 P2 设为 16，即循环开始的 ListRead，但生成这段代码的 SQLite 解析器并没有进行这种优化。

删除 指令完成了这个循环的工作；它从栈上弹出一个整数键（由前面的 ListRead 放置），并删除具有该键的游标 P1 的记录。因为 P2 是真的，所以行更改计数器被递增。

跳转 指令跳回到循环的开头。这是循环的结束。

> `20    ListReset     0      0 21    Close         0      0 22    Commit        0      0 23    Halt          0      0`

这一段指令块清理了 VDBE 程序。这些指令中有三个实际上并不需要，但是它们是由 SQLite 解析器从其代码模板生成的，这些模板设计用于处理更复杂的情况。

ListReset 指令清空临时存储列表。当 VDBE 程序终止时，此列表会自动清空，因此在这种情况下是不必要的。Close 指令关闭游标 P1。同样，在 VDBE 引擎完成运行此程序时会执行此操作。Commit 成功结束当前事务，并导致此事务中发生的所有更改保存到数据库。最后的 Halt 也是不必要的，因为它在每个准备运行的 VDBE 程序中都会添加。

UPDATE 语句的工作方式与 DELETE 语句非常相似，只是不删除记录而是用新记录替换它。考虑以下示例：

> ```sql
> UPDATE examp SET one= '(' || one || ')' WHERE two < 50;
> 
> ```

而不是删除“two”列小于 50 的记录，该语句只是将“one”列放在括号中。实现此语句的 VDBE 程序如下：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0                                          1     Transaction   0      0                                          2     VerifyCookie  0      178                                             3     Integer       0      0                                          4     OpenRead      0      3      examp                               5     Rewind        0      12                                         6     Column        0      1                                          7     Integer       50     0      50                                  8     Ge            1      11                                         9     Recno         0      0                                          10    ListWrite     0      0                                          11    Next          0      6                                               12    Close         0      0                                          13    Integer       0      0                                          14    OpenWrite     0      3                                               15    ListRewind    0      0                                          16    ListRead      0      28                                              17    Dup           0      0                                -   链接（[地址](https://example.org)）需要翻译其内容，但保留网址

这个程序与 DELETE 程序基本相同，只是第二个循环体的主体被一系列指令（地址为 17 到 26）所替代，这些指令用于更新记录而不是删除它。这个指令序列中的大部分内容您应该已经很熟悉了，但是还有一些小的变化，因此我们会简要地过一下。同时注意在第二个循环体之前和之后一些指令的顺序已经改变。这只是 SQLite 解析器使用不同模板输出代码的方式。

进入第二个循环体内部（在指令 17 处），栈内包含一个单独的整数，这个整数是我们要修改的记录的键。我们需要使用这个键两次：一次是获取记录的旧值，另一次是写入修改后的记录。因此，第一条指令是 Dup，用于复制栈顶的键。Dup 指令可以复制栈中的任何元素，而不仅仅是顶部元素。您可以使用 P1 操作数指定要复制的元素。当 P1 为 0 时，复制栈顶元素；当 P1 为 1 时，复制栈中下一个元素，以此类推。

在复制完键之后，接下来的指令是 NotExists，它将栈顶的值弹出并将其作为键来检查数据库文件中记录的存在性。如果不存在该键对应的记录，则跳回到 ListRead 来获取另一个键。

指令 19 到 25 用于构建一个新的数据库记录，用于替换现有的记录。这与 INSERT 中描述的代码类型相同，这里不再详细描述。执行完第 25 条指令后，栈的情况如下：

> | （记录）新数据记录 |
> | --- |
> | （整数）键 |

PutIntKey 指令（在讨论 INSERT 时也有描述）将一个条目写入数据库文件，其数据为堆栈顶部的内容，键为堆栈中的下一个内容，然后将堆栈弹出两次。PutIntKey 指令将用新数据覆盖具有相同键的现有记录，这正是我们所希望的。在 INSERT 时不存在覆盖的问题，因为键是由 NewRecno 指令生成的，该指令保证提供一个之前未使用的键。

## CREATE 和 DROP

使用 CREATE 或 DROP 命令来创建或销毁表格或索引，在 VDBE 的视角下与从特殊的"sqlite_master"表中进行 INSERT 或 DELETE 操作基本相同。sqlite_master 表是每个 SQLite 数据库自动创建的特殊表，其结构如下：

> ```sql
> CREATE TABLE sqlite_master (
>   type      TEXT,    -- either "table" or "index"
>   name      TEXT,    -- name of this table or index
>   tbl_name  TEXT,    -- for indices: name of associated table
>   sql       TEXT     -- SQL text of the original CREATE statement
> )
> 
> ```

每个 SQLite 数据库中的每个表（除了"sqlite_master"表本身）和每个命名索引都在 sqlite_master 表中有一个条目。你可以使用 SELECT 语句查询这个表，就像查询任何其他表一样。但是你不允许直接使用 UPDATE、INSERT 或 DELETE 来更改表。必须使用 CREATE 和 DROP 命令对 sqlite_master 进行更改，因为 SQLite 在添加或销毁表和索引时也需要更新其一些内部数据结构。

但从 VDBE 的视角来看，CREATE 几乎就像是 INSERT，而 DROP 则像 DELETE。当 SQLite 库打开现有数据库时，它首先执行 SELECT 以读取 sqlite_master 表的所有条目的"sql"列。"sql"列包含最初生成索引或表的 CREATE 语句的完整 SQL 文本。此文本被反馈到 SQLite 解析器中，并用于重建描述索引或表的内部数据结构。

## 使用索引加速搜索

在上述示例查询中，被查询的表的每一行都必须从磁盘加载并检查，即使只有很小一部分行最终出现在结果中。对于大表来说，这可能需要很长时间。为了加快速度，SQLite 可以使用索引。

对于 SQLite 文件，将键与一些数据关联起来。对于 SQLite 表，数据库文件被设置为键是整数，数据是表的一行的信息。SQLite 中的索引颠倒了这种安排。索引键是存储的一些信息，索引数据是整数。要访问具有特定内容的表行，我们首先在索引表中查找内容以找到其整数索引，然后使用该整数来查找表中的完整记录。

注意 SQLite 使用了 b 树，这是一种排序数据结构，因此当 SELECT 语句的 WHERE 子句包含对等式或不等式的测试时，可以使用索引。以下示例查询如果有索引可用则可以使用：

> ```sql
> SELECT * FROM examp WHERE two==50;
> SELECT * FROM examp WHERE two<50;
> SELECT * FROM examp WHERE two IN (50, 100);
> 
> ```

如果存在一个将 "examp" 表的 "two" 列映射到整数的索引，则 SQLite 将使用该索引来查找具有列二值为 50 的所有行的整数键，或者所有小于 50 的行等。但以下查询无法使用索引：

> ```sql
> SELECT * FROM examp WHERE two%50 == 10;
> SELECT * FROM examp WHERE two&127 == 3;
> 
> ```

注意 SQLite 解析器不总是会生成代码来使用索引，即使可以这样做。以下查询目前不会使用索引：

> ```sql
> SELECT * FROM examp WHERE two+10 == 50;
> SELECT * FROM examp WHERE two==50 OR two==100;
> 
> ```

要更好地理解索引的工作原理，让我们首先看看它们是如何创建的。我们继续在 examp 表的两列上建立索引。我们有：

> ```sql
> CREATE INDEX examp_idx1 ON examp(two);
> 
> ```

上述语句生成的 VDBE 代码如下所示：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0                                          1     Transaction   0      0                                          2     VerifyCookie  0      178                                             3     Integer       0      0                                          4     OpenWrite     0      2                                          5     NewRecno      0      0                                          6     String        0      0      index                               7     String        0      0      examp_idx1                          8     String        0      0      examp                               9     CreateIndex   0      0      ptr(0x791380)                       10    Dup           0      0                                          11    Integer       0      0                                          12    OpenWrite     1      0                                          13    String        0      0      CREATE INDEX examp_idx1 ON examp(tw 14    MakeRecord    5      0                                          15    PutIntKey     0      0                                          16    Integer       0      0                                          17    OpenRead      2      3      examp                               18    Rewind        2      24                                              19    Recno         2      0                -   `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0                                          1     Transaction   0      0                                          2     VerifyCookie  0      178                                             3     Integer       0      0                                          4     OpenWrite     0      2                                          5     NewRecno      0      0                                          6     String        0      0      index                               7     String        0      0      examp_idx1                          8     String        0      0      examp                               9     CreateIndex   0      0      ptr(0x791380)                       10    Dup           0      0                                          11    Integer       0      0                                          12    OpenWrite     1      0                                          13    String        0      0      CREATE INDEX examp_idx1 ON examp(tw 14    MakeRecord    5      0                                          15    PutIntKey     0      0                                          16    Integer       0      0                                          17    OpenRead      2      3      examp                               18    Rewind        2      24                                              19    Recno         2      0                -   `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0                                          1     Transaction   0      0                                          2     VerifyCookie  0      178                                             3     Integer       0      0                                          4     OpenWrite     0      2                                          5     NewRecno      0      0                                          6     String        0      0      index                               7     String        0      0      examp_idx1                          8     String        0      0      examp                               9     CreateIndex   0      0      ptr(0x791380)                       10    Dup           0      0                                          11    Integer       0      0                                          12    OpenWrite     1      0                                          13    String        0      0      CREATE INDEX examp_idx1 ON examp(tw 14    MakeRecord    5      0                                          15    PutIntKey     0      0                                          16    Integer       0      0                                          17    OpenRead      2      3      examp                               18    Rewind        2      24                                              19    Recno         2      0

请记住，除了 sqlite_master 表之外的每个表和每个命名索引都在 sqlite_master 表中有一个条目。因为我们正在创建一个新的索引，所以我们必须向 sqlite_master 表中添加一个新的条目。这由第 3 到 15 条指令处理。向 sqlite_master 添加条目的过程就像任何其他 INSERT 语句一样，所以我们在这里不再多说。在这个示例中，我们希望专注于使用有效数据填充新索引，这发生在第 16 到 23 条指令之间。

> `16    Integer       0      0                                          17    OpenRead      2      3      examp`

首先要做的是打开用于读取的正在索引的表。为了为表构建索引，我们必须知道该表中包含什么。使用指令 3 和 4 已经打开了用于写入的索引。

> `18    Rewind        2      24                                              19    Recno         2      0                                          20    Column        2      1                                          21    MakeIdxKey    1      0      n                                   22    IdxPut        1      0      indexed columns are not unique      23    Next          2      19`

指令 18 到 23 实现了对正在被索引的表的每一行进行循环。对于每一行表格，我们首先使用指令 19 中的 Recno 提取该行的整数键，然后使用指令 20 中的 Column 获取“two”列的值。第 21 条指令 MakeIdxKey 将来自堆栈顶部的“two”列数据转换为有效的索引键。对于单列索引，这基本上是一个空操作。但是如果 MakeIdxKey 的 P1 操作数大于一，多个条目将从堆栈中弹出并转换为单个索引键。第 22 条指令 IdxPut 实际上创建索引条目。IdxPut 从堆栈中弹出两个元素。堆栈顶部用作键来从索引表中获取条目。然后将第二个在堆栈上的整数添加到该索引的整数集合中，并将新记录写回数据库文件。请注意，如果两个或多个表条目在“two”列上具有相同的值，同一个索引条目可以存储多个整数。

现在让我们看看这个索引将如何被使用。考虑以下查询：

> ```sql
> SELECT * FROM examp WHERE two==50;
> 
> ```

SQLite 生成以下 VDBE 代码来处理此查询：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      one                                 1     ColumnName    1      0      two                                 2     Integer       0      0                                          3     OpenRead      0      3      examp                               4     VerifyCookie  0      256                                             5     Integer       0      0                                          6     OpenRead      1      4      examp_idx1                          7     Integer       50     0      50                             8     MakeKey       1      0      n                                   9     MemStore      0      0                                          10    MoveTo        1      19                                              11    MemLoad       0      0                                          12    IdxGT         1      19                                              13    IdxRecno      1      0                                          14    MoveTo        0      0                                          15    Column        0      0                                          16    Column        0      1                                          17    Callback      2      0                                -   `地址   操作码       参数 1   参数 2   参数 3                                         ----  ------------  -----  -----  ----------------------------------- 0     列名          0      0      one                                 1     列名          1      0      two                                 2     整数          0      0                                          3     打开读取      0      3      examp                               4     验证 Cookie   0      256                                             5     整数          0      0                                          6     打开读取      1      4      examp_idx1                          7     整数          50     0      50                             8     生成键        1      0      n                                   9     内存存储      0      0                                          10    移动到        1      19                                              11    内存加载      0      0                                          12    索引大于      1      19                                              13    索引记录号    1      0                                          14    移动到        0      0                                          15    列            0      0                                          16    列            0      1                                -                                           17    回调         2      0

SELECT 查询以熟悉的方式开始。首先初始化列名并打开正在查询的表。从第 5 和第 6 条指令开始，情况有所不同，索引文件也被打开。指令 7 和 8 创建一个值为 50 的键。地址 9 处的 MemStore 指令将索引键存储在 VDBE 内存位置 0 中。VDBE 内存用于避免从栈深处获取值，尽管这是可行的，但会使程序更难生成。接下来的指令，地址 10 处的 MoveTo，弹出栈中的键并将索引游标移动到具有该键的索引的第一行。这初始化了用于以下循环中的游标。

指令 11 至 18 实现了对由地址 8 处获取的索引键对应的所有索引记录的循环。所有具有此键的索引记录将在索引表中连续存放，因此我们遍历它们，并从索引中获取相应的表键。然后，使用此表键将游标移动到表中的那一行。循环的其余部分与非索引 SELECT 查询的循环相同。

循环以地址 11 处的 MemLoad 指令开始，将索引键的副本推送回栈中。地址 12 处的 IdxGT 指令将当前游标 P1 指向的索引记录中的键与我们正在查找的键进行比较。如果当前游标位置的索引键大于我们要查找的索引键，则跳出循环。

指令 IdxRecno 在 13 处将索引中的表记录编号推送到堆栈上。接下来的 MoveTo 指令将其弹出并将表游标移动到该行。接下来的 3 条指令以与非索引情况相同的方式选择列数据。Column 指令提取列数据并调用回调函数。最后的 Next 指令将索引游标前进到下一行，而不是表游标，并且如果索引中还有记录，将分支返回到循环的起点。

由于索引用于查找表中的值，因此保持索引与表一致非常重要。现在在 examp 表上有了索引，我们将不得不在 examp 表中插入、删除或更改数据时更新该索引。记住上面的第一个示例，我们能够使用 12 条 VDBE 指令将新行插入到 "examp" 表中。现在，这张表已经建立了索引，需要 19 条指令。SQL 语句如下所示：

> ```sql
> INSERT INTO examp VALUES('Hello, World!',99);
> 
> ```

并且生成的代码如下所示：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     Transaction   1      0                                          1     Transaction   0      0                                          2     VerifyCookie  0      256                                             3     Integer       0      0                                          4     OpenWrite     0      3      examp                               5     Integer       0      0                                          6     OpenWrite     1      4      examp_idx1                          7     NewRecno      0      0                                          8     String        0      0      Hello, World!                       9     Integer       99     0      99                                  10    Dup           2      1                                          11    Dup           1      1                                          12    MakeIdxKey    1      0      n                                   13    IdxPut        1      0                                          14    MakeRecord    2      0                                          15    PutIntKey     0      1                                          16    Close         0      0                                          17    Close         1      0                                          18    Commit        0      0                                抱歉，我漏掉了最后一行原文的内容，请允许我继续完成翻译：

到目前为止，你应该已经足够理解 VDBE，能够独立推断上述程序的工作原理。因此，在本文中我们将不再进一步讨论它。

## 连接

在连接操作中，两个或多个表被合并以生成单个结果。结果表包含来自被连接表的每个可能的行组合。实现这一点最简单和自然的方法是使用嵌套循环。

回想一下上面讨论过的查询模板，在那里有一个单独的循环搜索表的每条记录。在连接中，我们基本上有相同的东西，只是有嵌套循环。例如，要连接两个表，查询模板可能如下所示：

1.  初始化**azColumnName[]**数组以供回调使用。

1.  打开两个游标，一个用于查询的每个表。

1.  对于第一个表中的每条记录，执行以下操作：

    1.  对于第二个表中的每条记录，执行以下操作：

        1.  如果 WHERE 子句评估为假，则跳过后续步骤并继续下一条记录。

        1.  计算当前结果行的所有列。

        1.  调用当前结果行的回调函数。

1.  关闭两个游标。

这个模板会起作用，但可能会很慢，因为我们现在处理的是一个 O(N²) 循环。但通常情况下，WHERE 子句可以被分解为项，并且其中一个或多个项将仅涉及第一个表中的列。当发生这种情况时，我们可以将部分 WHERE 子句测试移出内部循环，并获得很高的效率。因此，一个更好的模板可能是这样的：

1.  初始化**azColumnName[]**数组以供回调使用。

1.  打开两个游标，一个用于查询的每个表。

1.  对于第一个表中的每条记录，执行以下操作：

    1.  评估仅涉及第一个表列的 WHERE 子句项。如果任何项为假（意味着整个 WHERE 子句必须为假），则跳过此循环的其余部分并继续下一条记录。

    1.  对于第二个表中的每条记录，执行以下操作：

        1.  如果 WHERE 子句评估为假，则跳过后续步骤并继续下一条记录。

        1.  计算当前结果行的所有列。

        1.  调用当前结果行的回调函数。

1.  关闭两个游标。

如果可以使用索引来加速两个循环中的搜索，则可以进行额外的加速。

SQLite 总是按照 SELECT 语句的 FROM 子句中表的出现顺序构建循环。最左边的表成为外部循环，最右边的表成为内部循环。理论上，在某些情况下可以重新排序循环以加快连接操作的评估速度。但 SQLite 不会尝试这种优化。

你可以通过以下示例看到 SQLite 如何构建嵌套循环：

> ```sql
> CREATE TABLE examp2(three int, four int);
> SELECT * FROM examp, examp2 WHERE two<50 AND four==two;
> 
> ```
> 
> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      examp.one                           1     ColumnName    1      0      examp.two                           2     ColumnName    2      0      examp2.three                        3     ColumnName    3      0      examp2.four                         4     Integer       0      0                                          5     OpenRead      0      3      examp                               6     VerifyCookie  0      909                                             7     Integer       0      0                                          8     OpenRead      1      5      examp2                              9     Rewind        0      24                                              10    Column        0      1                                          11    Integer       50     0      50                                  12    Ge            1      23                                              13    Rewind        1      23                                              14    Column        1      1                                          15    Column        0      1                                          16    Ne            1      22                                         17    Column        0      0                                          18    Column        0      1                                          19    Column        1      -   `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      examp.one                           1     ColumnName    1      0      examp.two                           2     ColumnName    2      0      examp2.three                        3     ColumnName    3      0      examp2.four                         4     Integer       0      0                                          5     OpenRead      0      3      examp                               6     VerifyCookie  0      909                                             7     Integer       0      0                                          8     OpenRead      1      5      examp2                              9     Rewind        0      24                                              10    Column        0      1                                          11    Integer       50     0      50                                  12    Ge            1      23                                              13    Rewind        1      23                                              14    Column        1      1                                          15    Column        0      1                                          16    Ne            1      22                                         17    Column        0      0                                          18    Column        0      1                                          19    Column        1      0                                          20    Column        1      1                                          21    Callback      4      0                                          22    Next          1      14                                              23    Next          0      10                                         24    Close         0      0                                          25    Close         1      0                                          26    Halt          0      0

表格 examp 上的外层循环由第 7 到 23 条指令实现。内部循环由第 13 到 22 条指令实现。注意，WHERE 表达式中的"two<50"条件仅涉及第一个表的列，并且可以从内部循环中提取出来。SQLite 执行此操作，并在第 10 到 12 条指令中实现"two<50"测试。"four==two"测试在内部循环的第 14 到 16 条指令中实现。

SQLite 在联接表格时不对表格施加任何任意限制。它还允许一个表格与自身联接。

## ORDER BY 子句

基于历史原因和效率考虑，当前所有排序都是在内存中完成的。

SQLite 使用一组特殊的指令来实现 ORDER BY 子句，以控制一个称为排序器的对象。在查询的最内层循环中，通常会有一个回调指令，但这里会构建一个记录，包含回调参数和一个键。这个记录被添加到排序器中（以链表形式）。查询循环结束后，记录列表被排序，并遍历这个列表。对于列表中的每条记录，调用回调函数。最后，关闭排序器并释放内存。

我们可以在以下查询中看到这个过程的实际操作：

> ```sql
> SELECT * FROM examp ORDER BY one DESC, two;
> 
> ```
> 
> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      one                                 1     ColumnName    1      0      two                                 2     Integer       0      0                                          3     OpenRead      0      3      examp                               4     VerifyCookie  0      909                                             5     Rewind        0      14                                              6     Column        0      0                                          7     Column        0      1                                          8     SortMakeRec   2      0                                               9     Column        0      0                                          10    Column        0      1                                          11    SortMakeKey   2      0      D+                                  12    SortPut       0      0                                               13    Next          0      6                                               14    Close         0      0                                               15    Sort          0      0                                               16    SortNext      0      19                                              17    SortCallback  2      0                                              18    Goto          0      16                                              19    SortReset     0      0                                          20    Halt          0      0`

只有一个排序器对象，因此没有打开或关闭它的指令。在需要时它会自动打开，并且在 VDBE 程序停止时关闭。

查询循环是由第 5 至 13 条指令构建的。第 6 至 8 条指令构建一个记录，该记录包含单次回调调用的 azData[] 值。排序键由第 9 至 11 条指令生成。第 12 条指令将调用记录和排序键组合成单个条目，并将该条目放入排序列表中。

第 11 条指令的 P3 参数尤为重要。排序关键字由将 P3 的一个字符前置到每个字符串并连接所有字符串而形成。排序比较函数将检查此字符以确定排序顺序是升序还是降序，以及是否按字符串或数字排序。在此示例中，第一列应按字符串降序排序，因此其前缀为 "D"，第二列应按数字升序排序，因此其前缀为 "+"。升序字符串排序使用 "A"，降序数字排序使用 "-"。

查询循环结束后，在第 14 条指令关闭正在查询的表。这是为了允许其他进程或线程访问该表，如果需要的话。在第 15 条指令中，已建立的记录列表按照排序。第 16 至 18 条指令遍历排序后的记录列表，并对每条记录调用回调函数一次。最后，在第 19 条指令中关闭排序器。

## 聚合函数和 GROUP BY 及 HAVING 子句

要计算聚合函数，VDBE 实现了一种特殊的数据结构和控制该数据结构的指令。数据结构是一个无序的桶集合，每个桶都有一个键和一个或多个内存位置。在查询循环中，GROUP BY 子句用于构建一个键，并将具有该键的桶置于焦点。如果之前不存在，则创建一个新的桶与该键。一旦桶处于焦点状态，桶的内存位置用于累积各种聚合函数的值。查询循环结束后，每个桶被访问一次，以生成结果的单行。

一个例子将有助于澄清这个概念。考虑以下查询：

> ```sql
> SELECT three, min(three+four)+avg(four) 
> FROM examp2
> GROUP BY three;
> 
> ```

为此查询生成的 VDBE 代码如下：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      three                               1     ColumnName    1      0      min(three+four)+avg(four)           2     AggReset      0      3                                               3     AggInit       0      1      ptr(0x7903a0)                       4     AggInit       0      2      ptr(0x790700)                       5     Integer       0      0                                          6     OpenRead      0      5      examp2                              7     VerifyCookie  0      909                                             8     Rewind        0      23                                              9     Column        0      0                                          10    MakeKey       1      0      n                                   11    AggFocus      0      14                                              12    Column        0      0                                          13    AggSet        0      0                                          14    Column        0      0                                          15    Column        0      1                                          16    Add           0      0                                          17    Integer       1      0                                          18    AggFunc       0      1      ptr(0x7903a0)                       19    Column        0      1                        -   **addr**  **opcode**        **p1**     **p2**     **p3**                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      three                               1     ColumnName    1      0      min(three+four)+avg(four)           2     AggReset      0      3                                               3     AggInit       0      1      ptr(0x7903a0)                       4     AggInit       0      2      ptr(0x790700)                       5     Integer       0      0                                          6     OpenRead      0      5      examp2                              7     VerifyCookie  0      909                                             8     Rewind        0      23                                              9     Column        0      0                                          10    MakeKey       1      0      n                                   11    AggFocus      0      14                                              12    Column        0      0                                          13    AggSet        0      0                                          14    Column        0      0                                          15    Column        0      1                                          16    Add           0      0                                          17    Integer       1      0                                          18    AggFunc       0      1      ptr(0x7903a0)                       19    Column        0      1

第一个感兴趣的指令是在 AggReset 处于 2。AggReset 指令将初始化桶集合为空集，并指定每个桶中可用的内存槽数量为 P2。在这个例子中，每个桶将包含 3 个内存槽。虽然不明显，但如果你仔细看程序的其余部分，你可以弄清楚每个这些槽位的用途。

> | Memory Slot | 此内存槽的预期用途 |
> | --- | --- |
> | 0 | “three”列 -- 桶的关键字 |
> | 1 | 最小的“three+four”值 |
> | 2 | 所有“four”值的总和。这用于计算“avg(four)”。 |

查询循环由指令 8 到 22 实现。由 GROUP BY 子句指定的聚合键由指令 9 和 10 计算。指令 11 使得相应的桶成为焦点。如果具有给定键的桶尚不存在，则创建一个新的桶，并且控制流通过到指令 12 和 13 来初始化桶。如果桶已经存在，则跳转到指令 14。聚合函数的值由指令 11 到 21 之间的指令更新。指令 14 到 18 更新内存槽 1 以保存下一个值“min(three+four)” 。然后指令 19 到 21 更新“four”列的总和。

查询循环完成后，在指令 23 关闭表“examp2”，以释放其锁，并且可以被其他线程或进程使用。接下来的步骤是循环遍历所有聚合桶，并为每个桶输出一个结果行。这由指令 24 到 30 的循环完成。AggNext 指令在 24 处将下一个桶置为焦点，或者如果已经检查过所有桶，则跳转到循环的结尾。结果的 3 列按顺序从聚合器桶中获取，分别在指令 25 到 27 处。最后，在指令 29 处调用回调函数。

因此，任何带有聚合函数的查询都是由两个循环实现的。第一个循环扫描输入表并将聚合信息计算到桶中，第二个循环遍历所有桶以计算最终结果。

确定一个聚合查询实际上是两个连续的循环，这使得理解 SQL 查询语句中 WHERE 子句和 HAVING 子句的区别变得更容易。WHERE 子句是对第一个循环的限制，而 HAVING 子句是对第二个循环的限制。通过在我们的示例查询中添加 WHERE 和 HAVING 子句，您可以看到这一点：

> ```sql
> SELECT three, min(three+four)+avg(four) 
> FROM examp2
> WHERE three>four
> GROUP BY three
> HAVING avg(four)<10;
> 
> ```
> 
> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     ColumnName    0      0      three                               1     ColumnName    1      0      min(three+four)+avg(four)           2     AggReset      0      3                                               3     AggInit       0      1      ptr(0x7903a0)                       4     AggInit       0      2      ptr(0x790700)                       5     Integer       0      0                                          6     OpenRead      0      5      examp2                              7     VerifyCookie  0      909                                             8     Rewind        0      26                                              9     Column        0      0                                          10    Column        0      1                                          11    Le            1      25                                              12    Column        0      0                                          13    MakeKey       1      0      n                                   14    AggFocus      0      17                                              15    Column        0      0                                          16    AggSet        0      0                                          17    Column        0      0                                          18    Column        0      1                                          19    Add           0      -   **addr**  **opcode**        **p1**     **p2**     **p3**                                       ----  ------------  -----  -----  ----------------------------------- 0     **ColumnName**    0      0      **three**                               1     **ColumnName**    1      0      **min(three+four)+avg(four)**           2     **AggReset**      0      3                                               3     **AggInit**       0      1      **ptr(0x7903a0)**                       4     **AggInit**       0      2      **ptr(0x790700)**                       5     **Integer**       0      0                                          6     **OpenRead**      0      5      **examp2**                              7     **VerifyCookie**  0      909                                             8     **Rewind**        0      26                                              9     **Column**        0      0                                          10    **Column**        0      1                                          11    **Le**            1      25                                              12    **Column**        0      0                                          13    **MakeKey**       1      0      **n**                               14    **AggFocus**      0      17                                              15    **Column**        0      0                                          16    **AggSet**        0      0                                          17    **Column**        0      0                                          18    **Column**        0      1                                          19    **Add**           0

这个最后示例生成的代码与前一个示例相同，只是增加了两个条件跳转用于实现额外的 WHERE 和 HAVING 子句。WHERE 子句由查询循环中的第 9 至 11 条指令实现。HAVING 子句由输出循环中的第 28 至 30 条指令实现。

## 将 SELECT 语句用作表达式中的项

"结构化查询语言"这个名字告诉我们，SQL 应该支持嵌套查询。事实上，SQL 支持两种不同的嵌套方式。任何返回单行单列结果集的 SELECT 语句都可以作为另一个 SELECT 语句中表达式的项。而返回单列多行结果集的 SELECT 语句可以作为 IN 和 NOT IN 运算符右操作数的项。我们将从第一种嵌套类型的示例开始，即使用单行单列 SELECT 作为另一个 SELECT 表达式中的项。以下是我们的示例：

> ```sql
> SELECT * FROM examp
> WHERE two!=(SELECT three FROM examp2
>             WHERE four=5);
> 
> ```

SQLite 处理这个问题的方式是首先运行内部 SELECT（针对 examp2 的那个），并将其结果存储在一个私有内存单元中。SQLite 在评估外部 SELECT 时，将这个私有内存单元的值替换为内部 SELECT 的值。代码看起来像这样：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     String        0      0                                          1     MemStore      0      1                                          2     Integer       0      0                                          3     OpenRead      1      5      examp2                              4     VerifyCookie  0      909                                             5     Rewind        1      13                                              6     Column        1      1                                          7     Integer       5      0      5                                   8     Ne            1      12                                         9     Column        1      0                                          10    MemStore      0      1                                          11    Goto          0      13                                              12    Next          1      6                                               13    Close         1      0                                          14    ColumnName    0      0      one                                 15    ColumnName    1      0      two                                 16    Integer       0      0                                          17    OpenRead      -   `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     String        0      0                                          1     MemStore      0      1                                          2     Integer       0      0                                          3     OpenRead      1      5      examp2                              4     VerifyCookie  0      909                                             5     Rewind        1      13                                              6     Column        1      1                                          7     Integer       5      0      5                                   8     Ne            1      12                                         9     Column        1      0                                          10    MemStore      0      1                                          11    Goto          0      13                                              12    Next          1      6                                               13    Close         1      0                                          14    ColumnName    0      0      one                                 15    ColumnName    1      0      two                                 16    Integer       0      0                                          17    OpenRead      0      3      examp                               18    Rewind        0      26                                              19    Column        0      1                                          20    MemLoad       0      0                                          21    Eq            1      25                                              22    Column        0      0                                          23    Column        0      1                                          24    Callback      2      0                                          25    Next          0      19                                              26    Close         0      0                                          27    Halt          0      0`

首两条指令将私有内存单元初始化为 NULL。指令 2 到 13 实现对 examp2 表的内部 SELECT 语句。请注意，与将结果发送到回调或将结果存储在排序器中不同，查询结果通过指令 10 推送到内存单元，并且在指令 11 处的跳转中退出循环。指令 11 处的跳转是遗留的，永远不会执行。

外部 SELECT 由指令 14 到 25 实现。特别是包含嵌套选择的 WHERE 子句由指令 19 到 21 实现。您可以看到，内部选择的结果通过指令 20 加载到堆栈上，并在 21 处的条件跳转中使用。

当子查询的结果是标量时，可以使用单个私有内存单元，就像前面的例子所示。但是，当子查询的结果是向量时，例如当子查询是 IN 或 NOT IN 的右操作数时，就需要采用不同的方法。在这种情况下，子查询的结果存储在一个临时表中，并使用 Found 或 NotFound 运算符测试该表的内容。考虑以下示例：

> ```sql
> SELECT * FROM examp
> WHERE two IN (SELECT three FROM examp2);
> 
> ```

实现此最后查询所生成的代码如下：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     OpenTemp      1      1                                          1     Integer       0      0                                          2     OpenRead      2      5      examp2                              3     VerifyCookie  0      909                                             4     Rewind        2      10                                         5     Column        2      0                                          6     IsNull        -1     9                                               7     String        0      0                                          8     PutStrKey     1      0                                          9     Next          2      5                                               10    Close         2      0                                          11    ColumnName    0      0      one                                 12    ColumnName    1      0      two                                 13    Integer       0      0                                          14    OpenRead      0      3      examp                               15    Rewind        0      25                                              16    Column        0      1                                          17    NotNull       -1     20                                        -   `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     OpenTemp      1      1                                          1     Integer       0      0                                          2     OpenRead      2      5      examp2                              3     VerifyCookie  0      909                                             4     Rewind        2      10                                         5     Column        2      0                                          6     IsNull        -1     9                                               7     String        0      0                                          8     PutStrKey     1      0                                          9     Next          2      5                                               10    Close         2      0                                          11    ColumnName    0      0      one                                 12    ColumnName    1      0      two                                 13    Integer       0      0                                          14    OpenRead      0      3      examp                               15    Rewind        0      25                                              16    Column        0      1                                          17    NotNull       -1     20

存储内部 SELECT 结果的临时表是由 0 号 OpenTemp 指令创建的。该操作码仅用于仅在单个 SQL 语句期间存在的表。即使主数据库是只读的，临时游标也始终以读/写方式打开。当游标关闭时，临时表会自动删除。P2 值为 1 表示游标指向 BTree 索引，该索引没有数据，但可以有任意键。

内部 SELECT 语句由指令 1 到 10 实现。所有这些代码所做的只是为 examp2 表中"three"列的每一行非空值创建临时表条目。每个临时表条目的键是 examp2 的"three"列，数据是空字符串，因为它从未被使用。

外部 SELECT 由指令 11 到 25 实现。特别是包含 IN 运算符的 WHERE 子句由 16、17 和 20 号指令实现。指令 16 将当前行的"two"列值推送到堆栈上，并检查其是否非空。如果成功，执行跳转到 20，其中检查堆栈顶部是否与临时表中的任何键匹配。其余代码与之前显示的代码相同。

## 复合 SELECT 语句

SQLite 还允许使用 UNION、UNION ALL、INTERSECT 和 EXCEPT 运算符将两个或多个 SELECT 语句作为对等体连接。这些复合选择语句使用临时表实现。每个运算符的实现略有不同，但基本思想是相同的。我们将使用 EXCEPT 运算符作为示例。

> ```sql
> SELECT two FROM examp
> EXCEPT
> SELECT four FROM examp2;
> 
> ```

最后一个示例的结果应该是 examp 表中"two"列的每个唯一值，但删除了 examp2 的"four"列中的任何值。实现此查询的代码如下：

> `addr  opcode        p1     p2     p3                                       ----  ------------  -----  -----  ----------------------------------- 0     OpenTemp      0      1                                          1     KeyAsData     0      1                                               2     Integer       0      0                                          3     OpenRead      1      3      examp                               4     VerifyCookie  0      909                                             5     Rewind        1      11                                         6     Column        1      1                                          7     MakeRecord    1      0                                          8     String        0      0                                          9     PutStrKey     0      0                                          10    Next          1      6                                               11    Close         1      0                                          12    Integer       0      0                                          13    OpenRead      2      5      examp2                              14    Rewind        2      20                                         15    Column        2      1                                          16    MakeRecord    1      0                                          17    NotFound      0      19                -   `未找到      18    Delete        0      0                                          19    Next          2      15                                              20    Close         2      0                                          21    ColumnName    0      0      four                                22    Rewind        0      26                                              23    Column        0      0                                          24    Callback      1      0                                          25    Next          0      23                                              26    Close         0      0                                          27    Halt          0      0`

结果构建的临时表是由指令 0 创建的。然后跟随三个循环。指令 5 到 10 的循环实现了第一个 SELECT 语句。指令 14 到 19 的循环实现了第二个 SELECT 语句。最后，指令 22 到 25 的循环读取临时表，并为结果中的每一行调用回调函数一次。

在本示例中，指令 1 尤为重要。通常，Column 指令从 SQLite 文件条目的数据中提取列的值。指令 1 设置了临时表上的一个标志，使得 Column 可以将 SQLite 文件条目的键视为数据，并从键中提取列信息。

这里将会发生以下事情：第一个 SELECT 语句将构建结果的行，并将每行保存为临时表中条目的键。临时表中每个条目的数据从未被使用，因此我们用空字符串填充它。第二个 SELECT 语句同样构建行，但是第二个 SELECT 构建的行会从临时表中移除。这就是为什么我们希望行存储在 SQLite 文件的键中，而不是数据中 -- 这样它们可以轻松定位并删除。

让我们更仔细地看看这里发生了什么。第一个 SELECT 由 5 到 10 行指令的循环实现。第 5 行指令通过倒回其光标来初始化循环。第 6 行指令从"examp"中提取"two"列的值，第 7 行将其转换为一行。第 8 行将一个空字符串推入堆栈。最后，第 9 行将该行写入临时表中。但请记住，PutStrKey 操作码使用堆栈顶部作为记录数据，堆栈中的下一个作为键。对于 INSERT 语句，MakeRecord 操作码生成的行是记录数据，而 NewRecno 操作码创建的整数是记录键。但在这里，角色被颠倒了，MakeRecord 生成的行成为记录键，而记录数据只是一个空字符串。

第二个 SELECT 由 14 到 19 行指令的循环实现。第 14 行指令通过倒回其光标来初始化循环。指令 15 和 16 从表"examp2"的"four"列创建一个新的结果行。但是，与其使用 PutStrKey 将这一新行写入临时表，我们选择调用 Delete 来将其从临时表中删除（如果存在）。

复合选择的结果由 22 到 25 行指令的循环发送给回调例程。这个循环本身并没有什么新鲜或引人注目的地方，除了在第 23 行的 Column 指令将会从记录键中提取一列，而不是记录数据。

## 摘要

本文已审阅了 SQLite 的 VDBE 用于实现 SQL 语句的所有主要技术。未展示的是，大多数这些技术可以结合使用，以生成适当复杂的查询语句的代码。例如，我们展示了如何在简单查询上完成排序，以及如何实现复合查询。但我们没有给出复合查询中排序的示例。这是因为在复合查询中进行排序并未引入任何新概念：它只是在同一个 VDBE 程序中结合了两个先前的概念（排序和复合）。

要获取 SQLite 库功能的更多信息，请读者直接查看 SQLite 源代码。如果你理解了本文中的内容，应该不会有太多难度来追踪源代码。SQLite 内部的认真学习者可能还想仔细研究 VDBE 操作码的文档，如这里所述。大多数操作码文档都是使用脚本从源代码的注释中提取的，因此您也可以直接从**vdbe.c**源文件中获取有关各种操作码的信息。如果您成功阅读到这里，理解其余部分应该不会有太大困难。

如果你发现文档或代码中有错误，请随时修正并/或联系作者，邮箱为 drh@hwaci.com。欢迎您的 Bug 修复或建议。
