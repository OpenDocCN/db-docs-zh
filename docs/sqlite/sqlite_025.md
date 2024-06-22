# 1\. SQLite 中的数据类型

> 原文：[`sqlite.com/datatype3.html`](https://sqlite.com/datatype3.html)

大多数 SQL 数据库引擎（我们所知道的除 SQLite 外的每个 SQL 数据库引擎）使用静态、严格的类型。在静态类型中，值的数据类型由其容器确定 - 值存储在的特定列。

SQLite 使用更通用的动态类型系统。在 SQLite 中，值的数据类型与值本身关联，而不是与其容器关联。SQLite 的动态类型系统与其他数据库引擎中更常见的静态类型系统向后兼容，这意味着在 SQLite 中可以像在静态类型数据库中一样使用 SQL 语句。然而，SQLite 中的动态类型使其能够执行传统上无法在严格类型数据库中实现的功能。灵活的类型是 SQLite 的特点，而非 bug。

更新：从版本 3.37.0 开始（2021-11-27），SQLite 提供了 STRICT 表，用于那些喜欢这种类型强制的开发者。

# 2\. 存储类和数据类型

SQLite 数据库中存储（或由数据库引擎操作）的每个值都有以下存储类之一：

+   **NULL**. 这个值是 NULL 值。

+   **INTEGER**. 这个值是有符号整数，存储在 0、1、2、3、4、6 或 8 个字节中，具体取决于值的大小。

+   **REAL**. 这个值是一个浮点数值，以 8 字节 IEEE 浮点数格式存储。

+   **TEXT**. 这个值是文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储。

+   **BLOB**. 这个值是一个数据块，按照输入时的精确格式存储。

存储类比数据类型更一般化。例如，INTEGER 存储类包括 7 种不同长度的整数数据类型。这在磁盘上有所不同。但是一旦 INTEGER 值从磁盘读入内存进行处理，它们将转换为最通用的数据类型（8 字节有符号整数）。因此，在大多数情况下，“存储类”与“数据类型”几乎没有区别，两个术语可以互换使用。

在 SQLite 版本 3 数据库中，除了 INTEGER PRIMARY KEY 列之外，任何列都可以用来存储任何存储类的值。

SQL 语句中的所有值，无论是嵌入 SQL 语句文本中的字面量还是绑定到预编译 SQL 语句的参数，都有一个隐式的存储类。在下文描述的情况下，数据库引擎可能在查询执行期间在数值存储类（INTEGER 和 REAL）和 TEXT 之间转换值。

## 2.1\. 布尔数据类型

SQLite 没有单独的布尔存储类。相反，布尔值以整数 0（false）和 1（true）存储。

SQLite 自版本 3.23.0（2018-04-02）起，识别关键字 "TRUE" 和 "FALSE"，但这些关键字实际上只是整数字面值 1 和 0 的替代拼写。

## 2.2\. 日期和时间数据类型

SQLite 没有专门用于存储日期和/或时间的存储类。相反，SQLite 的内置 日期和时间函数 能够将日期和时间存储为 TEXT、REAL 或 INTEGER 值：

+   **TEXT** 作为 ISO8601 字符串（"YYYY-MM-DD HH:MM:SS.SSS"）。

+   **REAL** 作为儒略日数，即格林尼治时间 4714 年 11 月 24 日中午以来的天数，根据推测的格里高利历。

+   **INTEGER** 作为 Unix 时间，即从 1970-01-01 00:00:00 UTC 起的秒数。

应用程序可以选择使用这些格式存储日期和时间，并使用内置的 日期和时间函数 在这些格式之间自由转换。

# 3\. 类型亲和力

使用严格类型的 SQL 数据库引擎通常会尝试自动将值转换为适当的数据类型。考虑以下示例：

> ```sql
> CREATE TABLE t1(a INT, b VARCHAR(10));
> INSERT INTO t1(a,b) VALUES('123',456);
> 
> ```

严格类型化的数据库将在执行插入之前，将字符串 '123' 转换为整数 123，将整数 456 转换为字符串 '456'。

为了最大化 SQLite 与其他数据库引擎之间的兼容性，并使上述示例在 SQLite 上与其他 SQL 数据库引擎一样有效，SQLite 支持列的 "类型亲和力" 概念。列的类型亲和力是存储在该列中的数据的推荐类型。这里的重要思想是类型是推荐的，而不是必须的。任何列仍然可以存储任何类型的数据。只是，某些列在选择时更倾向于使用一个存储类而不是另一个。列的首选存储类称为其 "亲和力"。

SQLite 3 数据库中的每列被分配以下类型亲和力之一：

+   **TEXT**

+   **NUMERIC**

+   **INTEGER**

+   **REAL**

+   **BLOB**

（历史注："BLOB" 类型亲和力曾被称为 "NONE"。但该术语易于与 "无亲和力" 混淆，因此改名。）

具有 TEXT 亲和力的列使用 NULL、TEXT 或 BLOB 存储所有数据。如果将数值数据插入具有 TEXT 亲和力的列中，则在存储之前将其转换为文本形式。

一个具有 NUMERIC 亲和性的列可以包含使用所有五种存储类别的值。当将文本数据插入 NUMERIC 列时，如果文本是一个格式良好的整数或实数文字字面值，则文本的存储类别将分别转换为 INTEGER 或 REAL（优先顺序）。如果 TEXT 值是一个格式良好的整数文字字面值，但太大而无法适应 64 位有符号整数，则将其转换为 REAL。在 TEXT 和 REAL 存储类别之间进行转换时，只保留数字的前 15 个有效十进制数字。如果 TEXT 值不是格式良好的整数或实数文字字面值，则该值将作为 TEXT 存储。在本段的目的下，十六进制整数文字字面值不被视为格式良好，并且将作为 TEXT 存储（这是为了与 SQLite 3.8.6 版本 2014-08-15 之前的版本的历史兼容性而执行的）。如果可以准确表示为整数的浮点值插入具有 NUMERIC 亲和性的列中，则该值将转换为整数。不会尝试转换 NULL 或 BLOB 值。

字符串可能看起来像带有小数点和/或指数符号的浮点文字字面值，但只要该值可以表示为整数，NUMERIC 亲和性将其转换为整数。因此，字符串'3.0e+5'在具有 NUMERIC 亲和性的列中存储为整数 300000，而不是浮点值 300000.0。

使用 INTEGER 亲和性的列与使用 NUMERIC 亲和性的列的行为相同。INTEGER 和 NUMERIC 亲和性之间的区别仅在于 CAST 表达式中明显：表达式"CAST(4.0 AS INT)"返回整数 4，而"CAST(4.0 AS NUMERIC)"将值保留为浮点数 4.0。

具有 REAL 亲和性的列的行为类似于具有 NUMERIC 亲和性的列，但是它会强制整数值转换为浮点表示。（作为内部优化，没有分数部分的小浮点值并且存储在具有 REAL 亲和性的列中的整数被写入磁盘以占用更少空间，并且在值读取时会自动转换回浮点值。这种优化在 SQL 级别完全不可见，只能通过检查数据库文件的原始位来检测到。）

具有 BLOB 亲和性的列不偏好一种存储类别，也不会尝试将数据从一种存储类别强制转换为另一种存储类别。

## 3.1\. 确定列亲和性

对于未声明为 STRICT 的表，列的亲和性由列的声明类型决定，根据以下规则按照显示的顺序：

1.  如果声明类型包含字符串"INT"，则分配 INTEGER 亲和性。

1.  If the declared type of the column contains any of the strings "CHAR", "CLOB", or "TEXT" then that column has TEXT affinity. Notice that the type VARCHAR contains the string "CHAR" and is thus assigned TEXT affinity.

1.  If the declared type for a column contains the string "BLOB" or if no type is specified then the column has affinity BLOB.

1.  If the declared type for a column contains any of the strings "REAL", "FLOA", or "DOUB" then the column has REAL affinity.

1.  Otherwise, the affinity is NUMERIC.

Note that the order of the rules for determining column affinity is important. A column whose declared type is "CHARINT" will match both rules 1 and 2 but the first rule takes precedence and so the column affinity will be INTEGER.

### 3.1.1\. Affinity Name Examples

The following table shows how many common datatype names from more traditional SQL implementations are converted into affinities by the five rules of the previous section. This table shows only a small subset of the datatype names that SQLite will accept. Note that numeric arguments in parentheses that following the type name (ex: "VARCHAR(255)") are ignored by SQLite - SQLite does not impose any length restrictions (other than the large global SQLITE_MAX_LENGTH limit) on the length of strings, BLOBs or numeric values.

> | Example Typenames From The CREATE TABLE Statement
> 
> or CAST Expression | Resulting Affinity | Rule Used To Determine Affinity |
> 
> | INT INTEGER
> 
> TINYINT
> 
> SMALLINT
> 
> MEDIUMINT
> 
> BIGINT
> 
> UNSIGNED BIG INT
> 
> INT2
> 
> INT8 | INTEGER | 1 |
> 
> | CHARACTER(20) VARCHAR(255)
> 
> VARYING CHARACTER(255)
> 
> NCHAR(55)
> 
> NATIVE CHARACTER(70)
> 
> NVARCHAR(100)
> 
> TEXT
> 
> CLOB | TEXT | 2 |
> 
> | BLOB *no datatype specified* | BLOB | 3 |
> | --- | --- | --- |
> 
> | REAL DOUBLE
> 
> DOUBLE PRECISION
> 
> FLOAT | REAL | 4 |
> 
> | NUMERIC DECIMAL(10,5)
> 
> BOOLEAN
> 
> DATE
> 
> DATETIME | NUMERIC | 5 |

Note that a declared type of "FLOATING POINT" would give INTEGER affinity, not REAL affinity, due to the "INT" at the end of "POINT". And the declared type of "STRING" has an affinity of NUMERIC, not TEXT.

## 3.2\. Affinity Of Expressions

Every table column has a type affinity (one of BLOB, TEXT, INTEGER, REAL, or NUMERIC) but expressions do not necessarily have an affinity.

Expression affinity is determined by the following rules:

+   The right-hand operand of an IN or NOT IN operator has no affinity if the operand is a list, or has the same affinity as the affinity of the result set expression if the operand is a SELECT.

+   When an expression is a simple reference to a column of a real table (not a VIEW or subquery) then the expression has the same affinity as the table column.

    +   Parentheses around the column name are ignored. Hence if X and Y.Z are column names, then (X) and (Y.Z) are also considered column names and have the affinity of the corresponding columns.

    +   应用于列名的任何运算符（包括无操作符的一元"+"运算符）将列名转换为总是没有亲和力的表达式。因此，即使 X 和 Y.Z 是列名，表达式+X 和+Y.Z 也不是列名，也没有亲和力。

+   形式为"CAST(*expr* AS *type*)"的表达式具有与声明类型为"*type*"的列相同的亲和力。

+   COLLATE 运算符具有与其左侧操作数相同的亲和力。

+   否则，表达式没有亲和力。

## 3.3\. 视图和子查询的列亲和力

视图或 FROM 子查询的“列”实际上是实现视图或子查询的 SELECT 语句结果集中的表达式。因此，视图或子查询的列的亲和力由上述表达式亲和力规则确定。考虑一个例子：

> ```sql
> CREATE TABLE t1(a INT, b TEXT, c REAL);
> CREATE VIEW v1(x,y,z) AS SELECT b, a+c, 42 FROM t1 WHERE b!=11;
> 
> ```

v1.x 列的亲和力将与 t1.b(TEXT)的亲和力相同，因为 v1.x 直接映射到 t1.b。但 v1.y 和 v1.z 列都没有亲和力，因为这些列映射到表达式 a+c 和 42，而表达式总是没有亲和力的。

### 3.3.1\. 复合视图的列亲和力

当实现一个视图或 FROM 子查询的 SELECT 语句是一个复合 SELECT 时，视图或子查询的每一列的亲和力将是组成复合 SELECT 的各个单独 SELECT 语句的相应结果列的亲和力。然而，确定使用哪个 SELECT 语句来确定亲和力是不确定的。在查询评估过程中，不同的组成 SELECT 语句可能在不同的时间确定亲和力。这个选择可能会在不同版本的 SQLite 中有所不同。这个选择可能会在同一版本的 SQLite 中的不同查询之间改变。在同一查询的不同时间，这个选择可能会有所不同。因此，你永远无法确定复合 SELECT 的列的亲和力将使用哪个版本中具有不同亲和力的子查询。

最佳实践是避免在复合 SELECT 中混合使用亲和力，如果您关心结果的数据类型。在复合 SELECT 中混合使用亲和力可能会导致令人惊讶和不直观的结果。例如，参见[论坛帖子 02d7be94d7](https://sqlite.org/forum/forumpost/02d7be94d7)。

## 3.4\. 列亲和力行为示例

以下 SQL 演示了 SQLite 在将值插入表时如何使用列亲和力进行类型转换。

> ```sql
> CREATE TABLE t1(
>     t  TEXT,     -- text affinity by rule 2
>     nu NUMERIC,  -- numeric affinity by rule 5
>     i  INTEGER,  -- integer affinity by rule 1
>     r  REAL,     -- real affinity by rule 4
>     no BLOB      -- no affinity by rule 3
> );
> 
> -- Values stored as TEXT, INTEGER, INTEGER, REAL, TEXT.
> INSERT INTO t1 VALUES('500.0', '500.0', '500.0', '500.0', '500.0');
> SELECT typeof(t), typeof(nu), typeof(i), typeof(r), typeof(no) FROM t1;
> text|integer|integer|real|text
> 
> -- Values stored as TEXT, INTEGER, INTEGER, REAL, REAL.
> DELETE FROM t1;
> INSERT INTO t1 VALUES(500.0, 500.0, 500.0, 500.0, 500.0);
> SELECT typeof(t), typeof(nu), typeof(i), typeof(r), typeof(no) FROM t1;
> text|integer|integer|real|real
> 
> -- Values stored as TEXT, INTEGER, INTEGER, REAL, INTEGER.
> DELETE FROM t1;
> INSERT INTO t1 VALUES(500, 500, 500, 500, 500);
> SELECT typeof(t), typeof(nu), typeof(i), typeof(r), typeof(no) FROM t1;
> text|integer|integer|real|integer
> 
> -- BLOBs are always stored as BLOBs regardless of column affinity.
> DELETE FROM t1;
> INSERT INTO t1 VALUES(x'0500', x'0500', x'0500', x'0500', x'0500');
> SELECT typeof(t), typeof(nu), typeof(i), typeof(r), typeof(no) FROM t1;
> blob|blob|blob|blob|blob
> 
> -- NULLs are also unaffected by affinity
> DELETE FROM t1;
> INSERT INTO t1 VALUES(NULL,NULL,NULL,NULL,NULL);
> SELECT typeof(t), typeof(nu), typeof(i), typeof(r), typeof(no) FROM t1;
> null|null|null|null|null
> 
> ```

# 4\. 比较表达式

SQLite 版本 3 具有通常的 SQL 比较运算符集，包括"="、"=="、"<"、"<="、">"、">="、"!="、""、"IN"、"NOT IN"、"BETWEEN"、"IS"和"IS NOT"。

## 4.1\. 排序顺序

比较的结果取决于操作数的存储类，根据以下规则：

+   带有存储类 NULL 的值被视为小于任何其他值（包括具有存储类 NULL 的另一个值）。

+   INTEGER 或 REAL 值小于任何 TEXT 或 BLOB 值。当 INTEGER 或 REAL 与另一个 INTEGER 或 REAL 进行比较时，执行数值比较。

+   TEXT 值小于 BLOB 值。当比较两个 TEXT 值时，使用适当的排序序列确定结果。

+   当比较两个 BLOB 值时，结果使用 memcmp() 决定。

## 4.2\. 类型转换优先于比较

在执行比较之前，SQLite 可能会尝试在存储类 INTEGER、REAL 和/或 TEXT 之间转换值。在进行比较之前是否尝试进行任何转换取决于操作数的类型亲和性。

"应用亲和性" 意味着仅当转换不会丢失重要信息时，将操作数转换为特定的存储类。数值始终可以转换为 TEXT。如果文本内容是格式良好的整数或实数文字，TEXT 值可以转换为数值，但不能是十六进制整数文字。通过简单地解释二进制 BLOB 内容作为当前数据库编码中的文本字符串，将 BLOB 值转换为 TEXT 值。

在比较操作数之前，根据以下规则按照所示顺序应用亲和性：

+   如果一个操作数具有 INTEGER、REAL 或 NUMERIC 亲和性，另一个操作数具有 TEXT 或 BLOB 或没有亲和性，则将 NUMERIC 亲和性应用于另一个操作数。

+   如果一个操作数具有 TEXT 亲和性，另一个操作数没有亲和性，则将 TEXT 亲和性应用于另一个操作数。

+   否则，不应用任何亲和性，并且两个操作数按原样进行比较。

表达式 "a BETWEEN b AND c" 被视为两个独立的二进制比较 "a >= b AND a <= c"，即使这意味着在每个比较中 'a' 应用不同的亲和性。在类似 "x IN (SELECT y ...)" 的比较中，数据类型转换被处理为实际上是 "x=y" 的比较。表达式 "a IN (x, y, z, ...)" 等效于 "a = +x OR a = +y OR a = +z OR ..."。换句话说，IN 操作符右侧的值（此示例中的 "x"、"y" 和 "z" 值）被认为没有亲和性，即使它们恰好是列值或 CAST 表达式。

## 4.3\. 比较示例

> ```sql
> CREATE TABLE t1(
>     a TEXT,      -- text affinity
>     b NUMERIC,   -- numeric affinity
>     c BLOB,      -- no affinity
>     d            -- no affinity
> );
> 
> -- Values will be stored as TEXT, INTEGER, TEXT, and INTEGER respectively
> INSERT INTO t1 VALUES('500', '500', '500', 500);
> SELECT typeof(a), typeof(b), typeof(c), typeof(d) FROM t1;
> text|integer|text|integer
> 
> -- Because column "a" has text affinity, numeric values on the
> -- right-hand side of the comparisons are converted to text before
> -- the comparison occurs.
> SELECT a < 40,   a < 60,   a < 600 FROM t1;
> 0|1|1
> 
> -- Text affinity is applied to the right-hand operands but since
> -- they are already TEXT this is a no-op; no conversions occur.
> SELECT a < '40', a < '60', a < '600' FROM t1;
> 0|1|1
> 
> -- Column "b" has numeric affinity and so numeric affinity is applied
> -- to the operands on the right.  Since the operands are already numeric,
> -- the application of affinity is a no-op; no conversions occur.  All
> -- values are compared numerically.
> SELECT b < 40,   b < 60,   b < 600 FROM t1;
> 0|0|1
> 
> -- Numeric affinity is applied to operands on the right, converting them
> -- from text to integers.  Then a numeric comparison occurs.
> SELECT b < '40', b < '60', b < '600' FROM t1;
> 0|0|1
> 
> -- No affinity conversions occur.  Right-hand side values all have
> -- storage class INTEGER which are always less than the TEXT values
> -- on the left.
> SELECT c < 40,   c < 60,   c < 600 FROM t1;
> 0|0|0
> 
> -- No affinity conversions occur.  Values are compared as TEXT.
> SELECT c < '40', c < '60', c < '600' FROM t1;
> 0|1|1
> 
> -- No affinity conversions occur.  Right-hand side values all have
> -- storage class INTEGER which compare numerically with the INTEGER
> -- values on the left.
> SELECT d < 40,   d < 60,   d < 600 FROM t1;
> 0|0|1
> 
> -- No affinity conversions occur.  INTEGER values on the left are
> -- always less than TEXT values on the right.
> SELECT d < '40', d < '60', d < '600' FROM t1;
> 1|1|1
> 
> ```

如果对形式为 "a<40" 的表达式重写为 "40>a"，则示例中的所有结果都相同。

# 5\. 操作符

数学运算符（+、-、*、/、%、<<、>>、& 和 |）会将两个操作数解释为数值。字符串或 BLOB 操作数会自动转换为 REAL 或 INTEGER 值。如果字符串或 BLOB 看起来像是一个实数（如果有小数点或指数）或者其值超出了可以表示为 64 位有符号整数的范围，那么它会被转换为 REAL。否则，操作数会转换为 INTEGER。数学操作数的隐含类型转换与转换为 NUMERIC 略有不同，即看起来像实数但没有小数部分的字符串和 BLOB 值会保持为 REAL，而不会像转换为 NUMERIC 那样转换为 INTEGER。即使这种从 STRING 或 BLOB 到 REAL 或 INTEGER 的转换是有损且不可逆的，也会执行。某些数学运算符（%，<<、>>、& 和 |）需要 INTEGER 操作数。对于这些运算符，REAL 操作数会像转换为 INTEGER 一样转换为 INTEGER。<<、>>、& 和 | 运算符始终返回 INTEGER（或 NULL）结果，但 % 运算符根据其操作数的类型返回 INTEGER 或 REAL（或 NULL）结果。在数学运算符上的 NULL 操作数会产生 NULL 结果。在数学运算符上的操作数如果看起来根本不是数值且不为 NULL，则会被转换为 0 或 0.0。除以零会得到 NULL 结果。

# 6\. 排序、分组和复合 SELECT

当通过 ORDER BY 子句对查询结果进行排序时，存储类别为 NULL 的值首先出现，然后是以数值顺序交错出现的 INTEGER 和 REAL 值，然后是按排序序列顺序出现的 TEXT 值，最后是按 memcmp() 顺序出现的 BLOB 值。在排序之前不会发生存储类别转换。

在 GROUP BY 子句中对值进行分组时，具有不同存储类别的值被视为不同，除非它们在数值上相等，INTEGER 和 REAL 值在数值上相等时会被视为相等。在 GROUP BY 子句的结果上不会应用任何亲和力。

复合 SELECT 操作符 UNION、INTERSECT 和 EXCEPT 在值之间执行隐式比较。对于与 UNION、INTERSECT 或 EXCEPT 关联的隐式比较，不会应用任何亲和力到比较操作数上 - 值会按原样进行比较。

# 7\. 排序序列

当 SQLite 比较两个字符串时，它使用一个排序序列或排序函数（这两个术语是相同的）来确定哪个字符串更大或两个字符串是否相等。SQLite 内置了三个排序函数：BINARY、NOCASE 和 RTRIM。

+   **BINARY** - 使用 memcmp() 比较字符串数据，无论文本编码如何。

+   **NOCASE** - 与 binary 相似，但使用 sqlite3_strnicmp()进行比较。因此，在执行比较之前，ASCII 的 26 个大写字符被折叠成它们的小写等价字符。请注意，只有 ASCII 字符会被折叠大小写。由于表的尺寸，SQLite 不尝试进行完整的 UTF 大小写折叠。还请注意，字符串中的任何 U+0000 字符都被视为比较目的的字符串终止符。

+   **RTRIM** - 与 binary 相同，但忽略尾随空格字符。

应用程序可以使用 sqlite3_create_collation()接口注册额外的排序函数。

排序函数仅在比较字符串值时才重要。数字值始终以数值方式比较，而 BLOB 始终使用 memcmp()逐字节比较。

## 7.1. 从 SQL 分配排序序列

每个表的每个列都有一个关联的排序函数。如果未明确定义排序函数，则默认为 BINARY。列定义的 COLLATE 子句用于为列定义替代排序函数。

用于二进制比较运算符（=，<，>，<=，>=，!=，IS 和 IS NOT）确定使用哪个排序函数的规则如下：

1.  如果任一操作数使用后缀 COLLATE operator 具有显式的排序函数分配，则将使用显式的排序函数进行比较，优先于左操作数的排序函数。

1.  如果任一操作数是列，则使用该列的排序函数，左操作数优先。对于前述句子，以一个或多个一元“+”运算符和/或 CAST 运算符前缀的列名仍然被视为列名。

1.  否则，将使用 BINARY 排序函数进行比较。

如果比较的操作数的任何子表达式使用后缀 COLLATE operator，则该操作数被认为有显式的排序函数分配（规则 1）。因此，如果在比较表达式的任何地方使用 COLLATE operator，则该操作数的排序函数定义将用于字符串比较，而不管该表的列可能是该表达式的一部分。如果在比较中的任何地方出现两个或更多的 COLLATE operator 子表达式，则无论 COLLATE 运算符在表达式中嵌套多深，以及表达式如何括号化，最左边的显式排序函数都将被使用。

表达式"x BETWEEN y and z"在逻辑上等同于两个比较"x >= y AND x <= z"，并且在处理排序函数时，它会被视为两个独立的比较。表达式"x IN (SELECT y ...)"在确定排序顺序时与表达式"x = y"处理方式相同。对于形式为"x IN (y, z, ...)"的表达式，使用的排序顺序是 x 的排序顺序。如果在 IN 运算符上需要明确的排序顺序，则应将其应用于左操作数，如下所示："x COLLATE nocase IN (y,z, ...)"。

SELECT 语句的 ORDER BY 子句的项可以使用 COLLATE 运算符分配排序顺序，这样指定的排序函数将用于排序。否则，如果由 ORDER BY 子句排序的表达式是列，则使用该列的排序顺序确定排序顺序。如果表达式不是列并且没有 COLLATE 子句，则使用 BINARY 排序顺序。

## 7.2\. 排序顺序示例

下面的示例确定了可能由各种 SQL 语句执行的文本比较的排序顺序。请注意，在数字、blob 或 NULL 值的情况下可能不需要文本比较，并且不使用排序顺序。

> ```sql
> CREATE TABLE t1(
>     x INTEGER PRIMARY KEY,
>     a,                 /* collating sequence BINARY */
>     b COLLATE BINARY,  /* collating sequence BINARY */
>     c COLLATE RTRIM,   /* collating sequence RTRIM  */
>     d COLLATE NOCASE   /* collating sequence NOCASE */
> );
>                    /* x   a     b     c       d */
> INSERT INTO t1 VALUES(1,'abc','abc', 'abc  ','abc');
> INSERT INTO t1 VALUES(2,'abc','abc', 'abc',  'ABC');
> INSERT INTO t1 VALUES(3,'abc','abc', 'abc ', 'Abc');
> INSERT INTO t1 VALUES(4,'abc','abc ','ABC',  'abc');
>  
> /* Text comparison a=b is performed using the BINARY collating sequence. */
> SELECT x FROM t1 WHERE a = b ORDER BY x;
> --result 1 2 3
> 
> /* Text comparison a=b is performed using the RTRIM collating sequence. */
> SELECT x FROM t1 WHERE a = b COLLATE RTRIM ORDER BY x;
> --result 1 2 3 4
> 
> /* Text comparison d=a is performed using the NOCASE collating sequence. */
> SELECT x FROM t1 WHERE d = a ORDER BY x;
> --result 1 2 3 4
> 
> /* Text comparison a=d is performed using the BINARY collating sequence. */
> SELECT x FROM t1 WHERE a = d ORDER BY x;
> --result 1 4
> 
> /* Text comparison 'abc'=c is performed using the RTRIM collating sequence. */
> SELECT x FROM t1 WHERE 'abc' = c ORDER BY x;
> --result 1 2 3
> 
> /* Text comparison c='abc' is performed using the RTRIM collating sequence. */
> SELECT x FROM t1 WHERE c = 'abc' ORDER BY x;
> --result 1 2 3
> 
> /* Grouping is performed using the NOCASE collating sequence (Values
> ** 'abc', 'ABC', and 'Abc' are placed in the same group). */
> SELECT count(*) FROM t1 GROUP BY d ORDER BY 1;
> --result 4
> 
> /* Grouping is performed using the BINARY collating sequence.  'abc' and
> ** 'ABC' and 'Abc' form different groups */
> SELECT count(*) FROM t1 GROUP BY (d || '') ORDER BY 1;
> --result 1 1 2
> 
> /* Sorting or column c is performed using the RTRIM collating sequence. */
> SELECT x FROM t1 ORDER BY c, x;
> --result 4 1 2 3
> 
> /* Sorting of (c||'') is performed using the BINARY collating sequence. */
> SELECT x FROM t1 ORDER BY (c||''), x;
> --result 4 2 3 1
> 
> /* Sorting of column c is performed using the NOCASE collating sequence. */
> SELECT x FROM t1 ORDER BY c COLLATE NOCASE, x;
> --result 2 4 3 1
> 
> ```
