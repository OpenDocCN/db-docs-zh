# 1\. 概述

> 原文：[`sqlite.com/quirks.html`](https://sqlite.com/quirks.html)

SQL 语言是一种“标准”。即便如此，没有两个 SQL 数据库引擎的工作方式完全相同。每个 SQL 实现都有其自己的特殊之处和怪癖，SQLite 也不例外。

本文旨在突出 SQLite 与其他 SQL 实现之间的主要差异，作为帮助那些正在将系统迁移到 SQLite 或从 SQLite 迁移的开发人员的辅助资料，或者是试图在多个数据库引擎之间构建系统的开发人员。

如果您是一个 SQLite 用户，遇到了此处未提及的某些 SQLite 怪癖，请通过在[SQLite 论坛](https://sqlite.org/forum/forum)上发布简短消息告知开发人员。

# 2\. SQLite 是嵌入式的，而不是客户端-服务器

每当将 SQLite 与 SQL Server、PostgreSQL、MySQL 或 Oracle 等其他 SQL 数据库引擎进行比较时，首先重要的是意识到 SQLite 并不打算取代或竞争这些系统中的任何一个。SQLite 是无服务器的。没有单独的服务器进程来管理数据库。应用程序通过函数调用与数据库引擎交互，而不是通过向单独的进程或线程发送消息。

SQLite 作为嵌入式和非服务器的特性，而不是客户端/服务器，这是一个特点，而不是一个缺陷。

像 MySQL、PostgreSQL、SQL Server、Oracle 等客户端/服务器数据库是现代系统中的重要组成部分。这些系统解决了一个重要问题。但是 SQLite 解决了不同的问题。SQLite 和客户端/服务器数据库都有其作用。将 SQLite 与其他 SQL 数据库引擎进行比较的开发人员需要清楚地理解这一区别。

参见 SQLite 适当使用场景文档以获取更多信息。

# 3\. 灵活的类型

SQLite 在数据类型方面非常灵活。数据类型是建议性的而不是强制性的。

一些评论者说 SQLite 是“弱类型”的，而其他 SQL 数据库是“强类型”的。我们认为这些术语不准确甚至带有贬义。我们更倾向于说 SQLite 是“灵活类型”的，而其他 SQL 数据库引擎是“严格类型”的。

请参阅 SQLite 中的数据类型文档，详细讨论 SQLite 中的类型系统。

关键点在于 SQLite 非常宽容于放入数据库中的数据类型。例如，如果一个列的数据类型是“INTEGER”，而应用程序插入了一个文本字符串到该列中，SQLite 会首先尝试将文本字符串转换为整数，就像每个其他 SQL 数据库引擎一样。因此，如果将**'1234'**插入到 INTEGER 列中，该值将被转换为整数 1234 并存储。但是，如果将非数值字符串如**'wxyz'**插入到 INTEGER 列中，与其他 SQL 数据库不同，SQLite 不会抛出错误。相反，SQLite 会将实际的字符串值存储在列中。

类似地，SQLite 允许您将一个 2000 字符的字符串存储到 VARCHAR(50)类型的列中。其他 SQL 实现要么会抛出错误，要么会截断字符串。SQLite 存储整个 2000 字符的字符串，没有信息丢失，也没有投诉。

这个问题的根源在于开发人员在使用 SQLite 做一些初始编码工作并使其应用程序正常运行后，尝试将其转换到像 PostgreSQL 或 SQL Server 这样的另一个数据库时。如果应用程序最初利用了 SQLite 的灵活类型，那么当转移到对数据类型更加严格的其他数据库时，它将失败。

灵活类型是 SQLite 的一个特性，而不是一个 bug。灵活类型是关于自由的。尽管如此，我们承认，这个特性有时会导致习惯于使用其他更严格的数据类型规则的开发人员感到困惑。回顾起来，也许如果 SQLite 仅实现了一个 ANY 数据类型，开发人员可以在需要使用灵活类型时明确说明，而不是将灵活类型作为默认选项，可能会减少困惑。作为对那些期望严格类型的人的一种适应，SQLite 版本 3.37.0（2021-11-27）引入了 STRICT tables 的选项。这些选项要么强制实施其他 SQL 数据库引擎中发现的强制数据类型约束，要么允许使用显式的 ANY 数据类型保留 SQLite 的灵活类型。

## 3.1\. 没有单独的 BOOLEAN 数据类型

与大多数其他 SQL 实现不同，SQLite 没有单独的 BOOLEAN 数据类型。相反，TRUE 和 FALSE 分别（通常）表示为整数 1 和 0。这似乎并没有引起太多问题，因为我们很少收到关于它的投诉。但是这一点很重要要认识到。

从 SQLite 版本 3.23.0（2018-04-02）开始，SQLite 还识别 TRUE 和 FALSE 关键字作为整数值 1 和 0 的别名。这提供了与其他 SQL 实现更好的兼容性。但为了向后兼容性，如果有名为 TRUE 或 FALSE 的列，则这些关键字将被视为引用这些列的标识符，而不是 BOOLEAN 文字。

## 3.2\. 没有单独的 DATETIME 数据类型

SQLite 没有 DATETIME 数据类型。相反，日期和时间可以以以下任何一种方式存储：

+   作为 ISO-8601 格式的文本字符串。例如：'2018-04-02 12:13:46'。

+   作为自 1970 年以来的整数秒数（也称为“unix 时间”）。

+   作为小数部分的实数值，即[儒略日数](https://en.wikipedia.org/wiki/Julian_day)。

SQLite 的内置 date and time functions 了解所有上述格式的日期/时间，并可以在它们之间自由变换。使用哪种格式完全取决于您的应用程序。

## 3.3\. 数据类型是可选的

因为 SQLite 在数据类型方面很灵活且宽容，所以可以创建没有指定数据类型的表列。例如：

```sql
CREATE TABLE t1(a,b,c,d);

```

表 "t1" 有四列 "a"、"b"、"c" 和 "d"，没有特定的数据类型。你可以在这些列中存储任何你想要的数据。

# 4\. 默认情况下，外键强制执行是关闭的

SQLite 早在很久以前就解析了外键约束，但是直到后来的 版本 3.6.19 (2009-10-14) 才增加了实际强制执行这些约束的能力。在增加外键约束强制执行之时，已经有数以百万计的数据库包含了某些不正确的外键约束。为了避免破坏那些遗留数据库，SQLite 默认情况下关闭了外键约束强制执行。

应用程序可以通过在运行时使用 PRAGMA foreign_keys 语句来激活外键强制执行。或者，可以通过 -DSQLITE_DEFAULT_FOREIGN_KEYS=1 编译时选项在编译时激活外键强制执行。

# 5\. 主键有时可以包含 NULL 值

在 SQLite 表中，主键通常只是一个唯一约束。由于一个历史性的疏忽，主键的列值允许为 NULL。这是一个 bug，但是当问题被发现时，已经有很多依赖于这个 bug 的数据库在流通中了，所以决定继续支持这种有问题的行为。你可以通过在每个主键列上添加 NOT NULL 约束来解决这个问题。

例外情况：

+   INTEGER PRIMARY KEY 列的值必须始终是非 NULL 整数，因为 INTEGER PRIMARY KEY 是 ROWID 的别名。如果尝试向 INTEGER PRIMARY KEY 列插入 NULL，则 SQLite 会自动将 NULL 转换为唯一的整数。

+   这个 bug 被发现后，添加了 WITHOUT ROWID 和 STRICT 特性，所以 WITHOUT ROWID 和 STRICT 表工作得很正常：它们不允许在主键中存在 NULL 值。

# 6\. 聚合查询可以包含非聚合结果列，这些列不在 GROUP BY 子句中

在大多数 SQL 实现中，聚合查询的输出列只能引用聚合函数或在 GROUP BY 子句中命名的列。在聚合查询中引用普通列并不明智，因为每个输出行可能由输入表中的两个或更多行组成。

SQLite 不强制执行这个限制。聚合查询的输出列可以是包含不在 GROUP BY 子句中的列的任意表达式。这个特性有两个用途：

1.  对于 SQLite（但不适用于我们所知的任何其他 SQL 实现），如果聚合查询包含单个 min()或 max()函数，则输出中使用的列值将来自满足 min()或 max()值的行。如果两个或更多行具有相同的 min()或 max()值，则列值将从这些行中的任意一行任意选择。

    例如，要查找薪水最高的员工：

    ```sql
    SELECT max(salary), first_name, last_name FROM employee;

    ```

    在上面的查询中，first_name 和 last_name 列的值将对应于满足 max(salary)条件的行。

1.  如果查询根本不包含任何聚合函数，则可以添加 GROUP BY 子句作为 DISTINCT ON 子句的替代。换句话说，输出行被过滤，以便每个 GROUP BY 列值集合中只显示一行。如果两个或更多输出行在 GROUP BY 列中具有相同的值集合，则任意选择其中一行。（SQLite 支持 DISTINCT 但不支持 DISTINCT ON，其功能由 GROUP BY 提供。）

# 7\. SQLite 默认情况下不进行完全的 Unicode 大小写折叠。

SQLite 对于所有 Unicode 字符并不区分大小写。像 upper()和 lower()这样的 SQL 函数只适用于 ASCII 字符。这有两个原因：

1.  虽然现在很稳定，但当 SQLite 首次设计时，Unicode 大小写折叠规则仍在变动中。这意味着行为可能会随着每个新的 Unicode 发布而改变，从而破坏应用程序并损坏索引。

1.  用于执行完全和正确的 Unicode 大小写折叠的表比整个 SQLite 库还要大。

如果 SQLite 编译时启用了-DSQLITE_ENABLE_ICU 选项并链接了[国际 Unicode 组件](https://icu.unicode.org)库，它就支持完全的 Unicode 大小写折叠。

# 8\. 双引号字符串字面量是被接受的

SQL 标准要求标识符要用双引号括起来，字符串字面量要用单引号括起来。例如：

+   `"this is a legal SQL column name"`

+   `'this is an SQL string literal'`

SQLite 接受上述两种情况。但是为了兼容 MySQL 3.x（当 SQLite 首次设计时，MySQL 3.x 是最广泛使用的关系数据库管理系统之一），SQLite 还将解释为字符串字面量，如果它不匹配任何有效标识符。

这个设计缺陷意味着，一个拼错的双引号标识符会被解释为一个字符串字面量，而不是生成一个错误。它还会诱使刚接触 SQL 语言的开发者养成错误使用双引号字符串字面量的坏习惯，而他们真正需要学会使用正确的单引号字符串字面量形式。

事后看来，我们不应该尝试使 SQLite 接受 MySQL 3.x 语法，并且永远不应该允许双引号字符串文字。然而，有无数应用程序使用双引号字符串文字，因此我们继续支持这种功能，以避免破坏传统。

截至 SQLite 3.27.0（2019-02-07），使用双引号字符串文字会导致向错误日志发送警告消息。

截至 SQLite 3.29.0（2019-07-10），可以使用 SQLITE_DBCONFIG_DQS_DDL 和 SQLITE_DBCONFIG_DQS_DML 操作通过 sqlite3_db_config()在运行时禁用双引号字符串文字。可以使用-DSQLITE_DQS=*N*编译选项在编译时更改默认设置。建议应用开发人员使用-DSQLITE_DQS=0 编译，以便默认禁用双引号字符串文字。如果不可能，则可以使用像下面这样的 C 代码为单个数据库连接禁用双引号字符串文字：

> ```sql
> sqlite3_db_config(db, SQLITE_DBCONFIG_DQS_DDL, 0, (void*)0);
> sqlite3_db_config(db, SQLITE_DBCONFIG_DQS_DML, 0, (void*)0);
> 
> ```

或者，如果默认情况下禁用了双引号字符串文字，但需要为一些历史数据库连接有选择地启用它们，则可以使用与上面显示的相同的 C 代码，只是将第三个参数从 0 更改为 1。

截至 SQLite 3.41.0（2023-02-21），默认情况下在 CLI 中禁用了 SQLITE_DBCONFIG_DQS_DDL 和 SQLITE_DBCONFIG_DQS_DML。如果需要，可以使用".dbconfig" dot-command 重新启用传统行为。

# 9\. 关键字通常可以用作标识符

SQL 语言中关键字丰富。大多数 SQL 实现不允许将关键字用作标识符（表名或列名）的名称，除非将其括在双引号中。但是 SQLite 更为灵活。许多关键字可以在不需要引号的情况下用作标识符，只要这些关键字在使用上下文中明确表明它们是标识符即可。

例如，在 SQLite 中，以下语句是有效的：

```sql
CREATE TABLE union(true INT, with BOOLEAN);

```

我们知道，其他所有 SQL 实现上述相同的 SQL 语句将失败，因为它使用关键字"union"、"true"和"with"作为标识符。

使用关键字作为标识符的能力促进了向后兼容性。随着添加新关键字，仅偶尔使用这些关键字作为表或列名的旧架构继续工作。然而，使用关键字作为标识符的能力有时会导致意外的结果。例如：

```sql
CREATE TRIGGER AFTER INSERT ON tableX BEGIN
  INSERT INTO tableY(b) VALUES(new.a);
END;

```

前面语句创建的触发器名为"AFTER"，它是一个"BEFORE"触发器。"AFTER"令牌被用作标识符而不是关键字，因此这是解析语句的唯一方法。另一个例子：

```sql
CREATE TABLE tableZ(INTEGER PRIMARY KEY);

```

tableZ 表只有一个名为"INTEGER"的列。该列没有指定数据类型，但是它是主键。该列*不是*表的 INTEGER PRIMARY KEY，因为它没有数据类型。"INTEGER"标记用作列名的标识符，而不是数据类型关键字。

# 10\. 可以在没有任何错误或警告的情况下允许可疑的 SQL。

SQLite 的最初实现旨在遵循[波斯特尔法则](https://zh.wikipedia.org/wiki/%E7%BD%91%E7%BB%9C%E8%A7%84%E8%8C%83)的部分内容，“在接受输入时要宽容”。这曾被认为是良好的设计——系统会接受可疑的输入并尽量做到最好而不会抱怨太多。但近来人们已经意识到，有时严格接受输入会更好，因为这样更容易找到输入中的错误。

# 11\. AUTOINCREMENT 与 MySQL 不同

SQLite AUTOINCREMENT 特性在 SQLite 中的工作方式与 MySQL 不同。这经常会让那些最初在 MySQL 上学习 SQL，然后开始使用 SQLite 并期望两个系统工作方式相同的人感到困惑。

详细了解 SQLite 中 AUTOINCREMENT 的功能及其作用，请参见 SQLite AUTOINCREMENT 文档。

# 12\. 文本字符串中允许存在 NUL 字符

NUL 字符（ASCII 码 0x00 和 Unicode \u0000）可能出现在 SQLite 字符串的中间。这可能导致意外的行为。有关更多信息，请参阅"字符串中的 NUL 字符"文档。

# 13\. SQLite 区分整数和文本文字面量。

SQLite 表示以下查询返回 false：

```sql
SELECT 1='1';

```

它这样做是因为整数不是字符串。其他所有主要的 SQL 数据库引擎都认为这是真的，原因是 SQLite 的创建者不理解。

# 14\. SQLite 错误地解析逗号连接的优先级

SQLite 给予所有连接操作符相等的优先级，并从左到右处理它们。但这并不完全正确。应该是逗号连接比其他所有连接操作符的优先级都低。换句话说，像这样的 FROM 子句：

> ... FROM a, b RIGHT JOIN c, d ...

应该解析如下：

<svg class="pikchr" viewBox="0 0 153.328 245.544"><text x="93" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">JOIN</text> <text x="53" y="87" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">JOIN</text> <text x="134" y="87" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">D</text> <text x="93" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">RIGHT JOIN</text> <text x="13" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">A</text> <text x="53" y="228" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">B</text> <text x="134" y="228" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">C</text></svg>

但是 SQLite 实际上会像这样解析 FROM 子句：

<svg class="pikchr" viewBox="0 0 188.691 245.544"><text x="134" y="17" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接</text> <text x="93" y="87" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">右连接</text> <text x="174" y="87" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">D</text> <text x="53" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">连接</text> <text x="134" y="157" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">C</text> <text x="13" y="228" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">A</text> <text x="93" y="228" text-anchor="middle" fill="rgb(0,0,0)" dominant-baseline="central">B</text></svg>

使用右外连接或全外连接在同一 FROM 子句中与逗号连接一起使用时，只有在极少数情况下才会在结果中产生差异。可以轻松通过在 FROM 子句中使用括号来解决这个问题：

> ... FROM a, (b 右连接 c), d ...
