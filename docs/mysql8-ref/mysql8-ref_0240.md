# 7.1.11 服务器 SQL 模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/sql-mode.html`](https://dev.mysql.com/doc/refman/8.0/en/sql-mode.html)

MySQL 服务器可以在不同的 SQL 模式下运行，并且可以根据`sql_mode`系统变量的值为不同的客户端应用这些模式。数据库管理员可以设置全局 SQL 模式以匹配站点服务器的操作需求，每个应用程序可以将其会话 SQL 模式设置为自己的需求。

模式影响 MySQL 支持的 SQL 语法和执行的数据验证检查。这使得在不同环境中使用 MySQL 以及与其他数据库服务器一起使用 MySQL 变得更加容易。

+   设置 SQL 模式

+   最重要的 SQL 模式

+   SQL 模式的完整列表

+   组合 SQL 模式

+   严格 SQL 模式

+   IGNORE 关键字和严格 SQL 模式的比较

有关 MySQL 中服务器 SQL 模式经常被问到的问题的答案，请参阅 Section A.3, “MySQL 8.0 FAQ: Server SQL Mode”。

在使用`InnoDB`表时，还要考虑`innodb_strict_mode`系统变量。它为`InnoDB`表启用额外的错误检查。

#### 设置 SQL 模式

MySQL 8.0 中的默认 SQL 模式包括这些模式：`ONLY_FULL_GROUP_BY`, `STRICT_TRANS_TABLES`, `NO_ZERO_IN_DATE`, `NO_ZERO_DATE`, `ERROR_FOR_DIVISION_BY_ZERO`, 和 `NO_ENGINE_SUBSTITUTION`。

要在服务器启动时设置 SQL 模式，请在命令行上使用`--sql-mode="*`modes`*"`选项，或在选项文件（如`my.cnf`（Unix 操作系统）或`my.ini`（Windows））中使用`sql-mode="*`modes`*"`。*`modes`*是由逗号分隔的不同模式列表。要明确清除 SQL 模式，请使用`--sql-mode=""`在命令行上设置为空字符串，或在选项文件中使用`sql-mode=""`。

注意

MySQL 安装程序可能在安装过程中配置 SQL 模式。

如果 SQL 模式与默认值或您期望的值不同，请检查服务器在启动时读取的选项文件中的设置。

要在运行时更改 SQL 模式，请使用`SET`语句设置全局或会话`sql_mode`系统变量：

```sql
SET GLOBAL sql_mode = '*modes*';
SET SESSION sql_mode = '*modes*';
```

设置`GLOBAL`变量需要`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限），并影响从那时起连接的所有客户端的操作。设置`SESSION`变量仅影响当前客户端。每个客户端可以随时更改其会话`sql_mode`值。

要确定当前的全局或会话`sql_mode`设置，请选择其值：

```sql
SELECT @@GLOBAL.sql_mode;
SELECT @@SESSION.sql_mode;
```

重要

**SQL 模式和用户定义的分区。** 在创建并向分区表中插入数据后更改服务器 SQL 模式可能会导致这些表行为发生重大变化，并可能导致数据丢失或损坏。强烈建议您在创建使用用户定义分区的表后永远不要更改 SQL 模式。

在复制分区表时，源和副本上的不同 SQL 模式也可能导致问题。为获得最佳结果，您应始终在源和副本上使用相同的服务器 SQL 模式。

有关更多信息，请参见第 26.6 节，“分区的限制和限制”。

#### 最重要的 SQL 模式

最重要的`sql_mode`值可能是这些：

+   `ANSI`

    此模式改变了语法和行为，使其更符合标准 SQL。这是本节末尾列出的特殊组合模式之一。

+   `STRICT_TRANS_TABLES`

    如果无法将给定值插入事务表中，则中止该语句。对于非事务表，在单行语句中发生该值或多行语句的第一行中发生该值时，中止该语句。本节后面将详细介绍更多细节。

+   `TRADITIONAL`

    使 MySQL 表现得像一个“传统”的 SQL 数据库系统。简单描述这种模式是在向列插入不正确的值时“给出错误而不是警告”。这是本节末尾列出的特殊组合模式之一。

    注意

    启用`TRADITIONAL`模式后，`INSERT`或`UPDATE`在出现错误时会立即中止。如果使用非事务性存储引擎，则可能不是您想要的，因为错误之前所做的数据更改可能不会被回滚，导致“部分完成”更新。

当本手册提到“严格模式”时，指的是启用了`STRICT_TRANS_TABLES`或`STRICT_ALL_TABLES`中的一个或两个模式。

#### SQL 模式的完整列表

以下列表描述了所有支持的 SQL 模式：

+   `ALLOW_INVALID_DATES`

    不要对日期进行完整检查。只需检查月份在 1 到 12 的范围内，日期在 1 到 31 的范围内即可。这对于在 Web 应用程序中获取年、月和日分别存储用户插入的数据而不进行日期验证的情况可能很有用。此模式适用于`DATE`和`DATETIME`列。不适用于总是需要有效日期的`TIMESTAMP`列。

    禁用`ALLOW_INVALID_DATES`后，服务器要求月份和日期值合法，而不仅仅在 1 到 12 和 1 到 31 的范围内。在禁用严格模式的情况下，无效日期如`'2004-04-31'`会被转换为`'0000-00-00'`并生成警告。启用严格模式后，无效日期会生成错误。要允许这样的日期，请启用`ALLOW_INVALID_DATES`。

+   `ANSI_QUOTES`

    将`"`视为标识符引用字符（类似于启用此模式以使用```sql quote character) and not as a string quote character. You can still use ```引用标识符）。启用`ANSI_QUOTES`后，您不能使用双引号引用文本字符串，因为它们会被解释为标识符。

+   `ERROR_FOR_DIVISION_BY_ZERO`

    `ERROR_FOR_DIVISION_BY_ZERO`模式影响除零操作的处理，包括`MOD(*`N`*,0)`。对于数据更改操作（`INSERT`, `UPDATE`)，其效果还取决于是否启用了严格 SQL 模式。

    +   如果未启用此模式，除零操作会插入`NULL`且不会生成警告。

    +   如果启用了此模式，除零操作会插入`NULL`并生成警告。

    +   如果启用了此模式和严格模式，除非也给出 `IGNORE`，否则除以零会产生错误。对于 `INSERT IGNORE` 和 `UPDATE IGNORE`，除以零会插入 `NULL` 并产生警告。

    对于 `SELECT`，除以零返回 `NULL`。启用 `ERROR_FOR_DIVISION_BY_ZERO` 会导致产生警告，无论是否启用了严格模式。

    `ERROR_FOR_DIVISION_BY_ZERO` 已被弃用。`ERROR_FOR_DIVISION_BY_ZERO` 不是严格模式的一部分，但应与严格模式一起使用，并且默认启用。如果启用了 `ERROR_FOR_DIVISION_BY_ZERO` 而没有同时启用严格模式，或反之，则会产生警告。

    因为 `ERROR_FOR_DIVISION_BY_ZERO` 已被弃用，您应该期望在未来的 MySQL 发行版中将其作为单独的模式名称移除，并将其效果包含在严格 SQL 模式的效果中。

+   `HIGH_NOT_PRECEDENCE`

    `NOT` 运算符的优先级使得诸如 `NOT a BETWEEN b AND c` 的表达式被解析为 `NOT (a BETWEEN b AND c)`。在一些较旧版本的 MySQL 中，该表达式被解析为 `(NOT a) BETWEEN b AND c`。通过启用 `HIGH_NOT_PRECEDENCE` SQL 模式可以获得旧的更高优先级行为。

    ```sql
    mysql> SET sql_mode = '';
    mysql> SELECT NOT 1 BETWEEN -5 AND 5;
     -> 0
    mysql> SET sql_mode = 'HIGH_NOT_PRECEDENCE';
    mysql> SELECT NOT 1 BETWEEN -5 AND 5;
     -> 1
    ```

+   `IGNORE_SPACE`

    允许在函数名称和 `(` 字符之间有空格。这会导致内置函数名称被视为保留字。因此，与函数名称相同的标识符必须按照 第 11.2 节，“模式对象名称” 中描述的方式加引号。例如，因为有一个 `COUNT()` 函数，在以下语句中将 `count` 用作表名会导致错误：

    ```sql
    mysql> CREATE TABLE count (i INT);
    ERROR 1064 (42000): You have an error in your SQL syntax
    ```

    表名应该加引号：

    ```sql
    mysql> CREATE TABLE `count` (i INT);
    Query OK, 0 rows affected (0.00 sec)
    ```

    `IGNORE_SPACE` SQL 模式适用于内置函数，而不适用于可加载函数或存储函数。无论是否启用 `IGNORE_SPACE`，在可加载函数或存储函数名称后面都可以有空格。

    有关 `IGNORE_SPACE` 的进一步讨论，请参见 第 11.2.5 节，“函数名称解析和解析”。

+   `NO_AUTO_VALUE_ON_ZERO`

    `NO_AUTO_VALUE_ON_ZERO`影响`AUTO_INCREMENT`列的处理。通常，通过插入`NULL`或`0`来生成列的下一个序列号。`NO_AUTO_VALUE_ON_ZERO`抑制了对`0`的这种行为，因此只有`NULL`会生成下一个序列号。

    如果在表的`AUTO_INCREMENT`列中存储了`0`，这种模式可能会有用（顺便说一句，存储`0`并不是一种推荐的做法）。例如，如果你使用**mysqldump**导出表然后重新加载，当遇到`0`值时，MySQL 通常会生成新的序列号，导致表的内容与导出时不同。在重新加载导出文件之前启用`NO_AUTO_VALUE_ON_ZERO`可以解决这个问题。因此，**mysqldump**在输出中自动包含一个启用`NO_AUTO_VALUE_ON_ZERO`的语句。

+   `NO_BACKSLASH_ESCAPES`

    启用此模式会禁用反斜杠字符（`\`）作为字符串和标识符中的转义字符。启用此模式后，反斜杠变成像其他字符一样的普通字符，并且`LIKE`表达式的默认转义序列被更改，不再使用转义字符。

+   `NO_DIR_IN_CREATE`

    创建表时，忽略所有`INDEX DIRECTORY`和`DATA DIRECTORY`指令。这个选项在复制服务器上很有用。

+   `NO_ENGINE_SUBSTITUTION`

    当语句（如`CREATE TABLE`或`ALTER TABLE`）指定一个被禁用或未编译的存储引擎时，控制默认存储引擎的自动替换。

    默认情况下，`NO_ENGINE_SUBSTITUTION`是启用的。

    因为存储引擎可以在运行时插拔，不可用的引擎会被同样对待：

    当`NO_ENGINE_SUBSTITUTION`被禁用时，对于`CREATE TABLE`，将使用默认引擎，如果所需引擎不可用，则会发出警告。对于`ALTER TABLE`，会发出警告并且表不会被修改。

    当`NO_ENGINE_SUBSTITUTION`被启用时，如果所需引擎不可用，将会发生错误并且表不会被创建或修改。

+   `NO_UNSIGNED_SUBTRACTION`

    整数值之间的减法，其中一个是`UNSIGNED`类型，默认会产生无符号结果。如果结果本应为负数，则会产生错误：

    ```sql
    mysql> SET sql_mode = '';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT CAST(0 AS UNSIGNED) - 1;
    ERROR 1690 (22003): BIGINT UNSIGNED value is out of range in '(cast(0 as unsigned) - 1)'
    ```

    如果启用了`NO_UNSIGNED_SUBTRACTION` SQL 模式，则结果为负数：

    ```sql
    mysql> SET sql_mode = 'NO_UNSIGNED_SUBTRACTION';
    mysql> SELECT CAST(0 AS UNSIGNED) - 1;
    +-------------------------+
    | CAST(0 AS UNSIGNED) - 1 |
    +-------------------------+
    |                      -1 |
    +-------------------------+
    ```

    如果此类操作的结果用于更新`UNSIGNED`整数列，则结果将被截断为列类型的最大值，或者如果启用了`NO_UNSIGNED_SUBTRACTION`，则截断为 0。启用严格 SQL 模式时，会发生错误，列保持不变��

    当启用`NO_UNSIGNED_SUBTRACTION`时，减法结果为有符号的，*即使任何操作数都是无符号的*。例如，比较表`t1`中列`c2`的类型与表`t2`中列`c2`的类型：

    ```sql
    mysql> SET sql_mode='';
    mysql> CREATE TABLE test (c1 BIGINT UNSIGNED NOT NULL);
    mysql> CREATE TABLE t1 SELECT c1 - 1 AS c2 FROM test;
    mysql> DESCRIBE t1;
    +-------+---------------------+------+-----+---------+-------+
    | Field | Type                | Null | Key | Default | Extra |
    +-------+---------------------+------+-----+---------+-------+
    | c2    | bigint(21) unsigned | NO   |     | 0       |       |
    +-------+---------------------+------+-----+---------+-------+

    mysql> SET sql_mode='NO_UNSIGNED_SUBTRACTION';
    mysql> CREATE TABLE t2 SELECT c1 - 1 AS c2 FROM test;
    mysql> DESCRIBE t2;
    +-------+------------+------+-----+---------+-------+
    | Field | Type       | Null | Key | Default | Extra |
    +-------+------------+------+-----+---------+-------+
    | c2    | bigint(21) | NO   |     | 0       |       |
    +-------+------------+------+-----+---------+-------+
    ```

    这意味着`BIGINT UNSIGNED`在所有情况下并非百分之百可用。请参阅第 14.10 节，“转换函数和运算符”。

+   `NO_ZERO_DATE`

    `NO_ZERO_DATE`模式影响服务器是否允许`'0000-00-00'`作为有效日期。其效果还取决于是否启用了严格 SQL 模式。

    +   如果未启用此模式，允许`'0000-00-00'`，并且插入不会产生警告。

    +   如果启用了此模式，允许`'0000-00-00'`，并且插入会产生警告。

    +   如果启用了此模式和严格模式，`'0000-00-00'`是不允许的，插入会产生错误，除非同时使用`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，允许`'0000-00-00'`，并且插入会产生警告。

    `NO_ZERO_DATE`已被弃用。`NO_ZERO_DATE`不是严格模式的一部分，但应与严格模式一起使用，并且默认情况下已启用。如果启用`NO_ZERO_DATE`而没有同时启用严格模式，或反之，则会发出警告。

    因为`NO_ZERO_DATE`已被弃用，您应该期望它在未来的 MySQL 发布中作为单独的模式名称被移除，并且其效果包含在严格 SQL 模式的效果中。

+   `NO_ZERO_IN_DATE`

    `NO_ZERO_IN_DATE`模式影响服务器是否允许日期中年份部分为非零但月份或日期部分为 0 的情况。（此模式影响诸如`'2010-00-01'`或`'2010-01-00'`之类的日期，但不影响`'0000-00-00'`。要控制服务器是否允许`'0000-00-00'`，请使用`NO_ZERO_DATE`模式。）`NO_ZERO_IN_DATE`的效果还取决于是否启用了严格 SQL 模式。

    +   如果未启用此模式，则允许具有零部分的日期，并且插入不会产生警告。

    +   如果启用了此模式，具有零部分的日期将被插入为`'0000-00-00'`并产生警告。

    +   如果启用了此模式和严格模式，将不允许具有零部分的日期，并且插入将产生错误，除非同时使用`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，具有零部分的日期将被插入为`'0000-00-00'`并产生警告。

    `NO_ZERO_IN_DATE`已被弃用。`NO_ZERO_IN_DATE`不是严格模式的一部分，但应与严格模式一起使用，并且默认情况下已启用。如果启用了`NO_ZERO_IN_DATE`而没有同时启用严格模式，或反之，则会发出警告。

    因为`NO_ZERO_IN_DATE`已被弃用，您应该预期在未来的 MySQL 版本中将其作为单独的模式名称移除，并将其效果包含在严格 SQL 模式的效果中。

+   `ONLY_FULL_GROUP_BY`

    拒绝查询，其中选择列表、`HAVING`条件或`ORDER BY`列表引用非聚合列，这些列既不在`GROUP BY`子句中命名，也不是功能上依赖于（由`GROUP BY`���唯一确定的）`GROUP BY`列。

    MySQL 对标准 SQL 的扩展允许`HAVING`子句中引用选择列表中的别名表达式。`HAVING`子句可以引用别名，无论是否启用了`ONLY_FULL_GROUP_BY`。

    有关更多讨论和示例，请参见 Section 14.19.3, “MySQL Handling of GROUP BY”。

+   `PAD_CHAR_TO_FULL_LENGTH`

    默认情况下，从`CHAR`列值中检索时会删除尾随空格。如果启用了`PAD_CHAR_TO_FULL_LENGTH`，则不会发生修剪，并且检索的`CHAR`值将填充到其完整长度。此模式不适用于`VARCHAR`列，对于这些列，在检索时会保留尾随空格。

    注意

    从 MySQL 8.0.13 开始，`PAD_CHAR_TO_FULL_LENGTH`已被弃用。预计在未来的 MySQL 版本中将其移除。

    ```sql
    mysql> CREATE TABLE t1 (c1 CHAR(10));
    Query OK, 0 rows affected (0.37 sec)

    mysql> INSERT INTO t1 (c1) VALUES('xy');
    Query OK, 1 row affected (0.01 sec)

    mysql> SET sql_mode = '';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT c1, CHAR_LENGTH(c1) FROM t1;
    +------+-----------------+
    | c1   | CHAR_LENGTH(c1) |
    +------+-----------------+
    | xy   |               2 |
    +------+-----------------+
    1 row in set (0.00 sec)

    mysql> SET sql_mode = 'PAD_CHAR_TO_FULL_LENGTH';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT c1, CHAR_LENGTH(c1) FROM t1;
    +------------+-----------------+
    | c1         | CHAR_LENGTH(c1) |
    +------------+-----------------+
    | xy         |              10 |
    +------------+-----------------+
    1 row in set (0.00 sec)
    ```

+   `PIPES_AS_CONCAT`

    将`||`视为字符串连接运算符（与`CONCAT()`相同），而不是`OR`的同义词。

+   `REAL_AS_FLOAT`

    将`REAL`视为`FLOAT`的同义词。默认情况下，MySQL 将`REAL`视为`DOUBLE`的同义词。

+   `STRICT_ALL_TABLES`

    启用所有存储引擎的严格 SQL 模式。无效数据值将被拒绝。详情请参见严格 SQL 模式。

+   `STRICT_TRANS_TABLES`

    为事务性存储引擎启用严格 SQL 模式，并在可能的情况下为非事务性存储引擎启用。详情请参见严格 SQL 模式。

+   `TIME_TRUNCATE_FRACTIONAL`

    控制在将带有小数秒部分的`TIME`, `DATE`或`TIMESTAMP`值插入到具有相同类型但小数位数较少的列时发生四舍五入或截断。默认行为是使用四舍五入。如果启用了此模式，则将发生截断。以下语句序列说明了差异：

    ```sql
    CREATE TABLE t (id INT, tval TIME(1));
    SET sql_mode='';
    INSERT INTO t (id, tval) VALUES(1, 1.55);
    SET sql_mode='TIME_TRUNCATE_FRACTIONAL';
    INSERT INTO t (id, tval) VALUES(2, 1.55);
    ```

    结果表内容如下，第一个值经过四舍五入，第二个值经过截断：

    ```sql
    mysql> SELECT id, tval FROM t ORDER BY id;
    +------+------------+
    | id   | tval       |
    +------+------------+
    |    1 | 00:00:01.6 |
    |    2 | 00:00:01.5 |
    +------+------------+
    ```

    另请参见第 13.2.6 节，“时间值中的小数秒”。

#### 组合 SQL 模式

以下特殊模式提供了前述列表中模式值的组合的简写。

+   `ANSI`

    等同于`REAL_AS_FLOAT`, `PIPES_AS_CONCAT`, `ANSI_QUOTES`, `IGNORE_SPACE`和`ONLY_FULL_GROUP_BY`。

    `ANSI`模式还会导致服务器对查询返回错误，其中一个带有外部引用`*`S`*(*`outer_ref`*)`的集合函数*`S`*无法在已解析外部引用的外部查询中进行聚合。这是这样一个查询：

    ```sql
    SELECT * FROM t1 WHERE t1.a IN (SELECT MAX(t1.b) FROM t2 WHERE ...);
    ```

    在这里，`MAX(t1.b)`不能在外部查询中聚合，因为它出现在该查询的`WHERE`子句中。标准 SQL 要求在这种情况下出错。如果未启用`ANSI`模式，服务器将在这些查询中以与解释`*`S`*(*`const`*)`相同的方式解释`*`S`*(*`outer_ref`*)`。

    参见第 1.6 节，“MySQL 标准兼容性”。

+   `TRADITIONAL`

    `TRADITIONAL`等同于`STRICT_TRANS_TABLES`、`STRICT_ALL_TABLES`、`NO_ZERO_IN_DATE`、`NO_ZERO_DATE`、`ERROR_FOR_DIVISION_BY_ZERO`和`NO_ENGINE_SUBSTITUTION`。

#### 严格 SQL 模式

严格模式控制 MySQL 如何处理数据更改语句中的无效或缺失值，例如`INSERT`或`UPDATE`。值可能无效的原因有多种。例如，它可能对于列的数据类型错误，或者超出范围。当要插入的新行不包含非`NULL`列的值且该列在定义中没有显式的`DEFAULT`子句时，值就会缺失。（对于`NULL`列，如果值缺失，则插入`NULL`。）严格模式还影响 DDL 语句，例如`CREATE TABLE`。

如果严格模式未生效，MySQL 会为无效或缺失的值插入调整后的值，并生成警告（参见第 15.7.7.42 节，“SHOW WARNINGS 语句”）。在严格模式下，您可以通过使用`INSERT IGNORE`或`UPDATE IGNORE`来产生这种行为。

对于诸如`SELECT`这样不改变数据的语句，在严格模式下，无效值会生成警告，而不是错误。

严格模式会在尝试创建超过最大键长度的键时产生错误。当未启用严格模式时，这会导致警告并将键截断为最大键长度。

严格模式不影响外键约束的检查。可以使用`foreign_key_checks`进行检查。（参见第 7.1.8 节，“服务器系统变量”。）

如果启用了`STRICT_ALL_TABLES`或`STRICT_TRANS_TABLES`，则严格 SQL 模式生效，尽管这些模式的影响略有不同：

+   对于事务表，在数据更改语句中出现无效或缺失值时，当`STRICT_ALL_TABLES`或`STRICT_TRANS_TABLES`启用时会发生错误。该语句将被中止并回滚。

+   对于非事务表，如果坏值出现在要插入或更新的第一行中，无论哪种模式，行为都是相同的：语句会被中止，表保持不变。如果语句插入或修改多行，且坏值出现在第二行或之后的行中，结果取决于启用了哪种严格模式：

    +   对于`STRICT_ALL_TABLES`，MySQL 会返回错误并忽略其余行。然而，由于较早的行已被插入或更新，结果是部分更新。为避免这种情况，请使用单行语句，可以在不更改表的情况下中止。

    +   对于`STRICT_TRANS_TABLES`，MySQL 会将无效值转换为列的最接近有效值并插入调整后的值。如果值缺失，MySQL 会插入列数据类型的隐式默认值。在任一情况下，MySQL 会生成警告而不是错误，并继续处理语句。隐式默认值在第 13.6 节，“数据类型默认值”中描述。

严格模式影响对零除法、零日期和日期中的零的处理如下：

+   严格模式影响对零除法的处理，包括`MOD(*`N`*,0)`：

    对于数据更改操作（`INSERT`，`UPDATE`)：

    +   如果未启用严格模式，零除法会插入`NULL`并不会产生警告。

    +   如果启用了严格模式，零除法会产生错误，除非也给出了`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，零除法会插入`NULL`并生成警告。

    对于`SELECT`，零除法返回`NULL`。启用严格模式会产生警告。

+   严格模式影响服务器是否允许`'0000-00-00'`作为有效日期：

    +   如果未启用严格模式，允许`'0000-00-00'`并且插入不会产生警告。

    +   如果启用了严格模式，不允许`'0000-00-00'`，并且插入会产生错误，除非也给出了`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，`'0000-00-00'`是允许的，并且插入会生成警告。

+   严格模式影响服务器是否允许年份部分为非零但月份或日期部分为 0 的日期（例如 `'2010-00-01'` 或 `'2010-01-00'`）：

    +   如果未启用严格模式，允许具有零部分的日期，并且插入不会产生警告。

    +   如果启用了严格模式，不允许具有零部分的日期，并且插入会产生错误，除非也给出了`IGNORE`。对于`INSERT IGNORE`和`UPDATE IGNORE`，具有零部分的日期会插入为`'0000-00-00'`（在`IGNORE`情况下被视为有效）并生成警告。

有关`IGNORE`与严格模式的更多信息，请参阅 IGNORE 关键字和严格 SQL 模式的比较。

严格模式影响对零除法、零日期和日期中的零的处理，与`ERROR_FOR_DIVISION_BY_ZERO`、`NO_ZERO_DATE`和`NO_ZERO_IN_DATE`模式一起。

#### `IGNORE`关键字和严格 SQL 模式的比较

本节比较了`IGNORE`关键字（将错误降级为警告）和严格 SQL 模式（将警告升级为错误）对语句执行的影响。它描述了它们影响的语句以及它们适用的错误。

以下表格总结了当默认为产生错误与警告时语句行为的比较。当默认为产生错误时的示例是将`NULL`插入`NOT NULL`列。当默认为产生警告时的示例是将错误数据类型的值插入列（例如将字符串`'abc'`插入整数列）。

| 操作模式 | 当语句默认为错误时 | 当语句默认为警告时 |
| --- | --- | --- |
| 没有`IGNORE`或严格 SQL 模式 | 错误 | 警告 |
| 使用`IGNORE` | 警告 | 警告（与没有`IGNORE`或严格 SQL 模式相同） |
| 使用严格的 SQL 模式 | 错误（与没有`IGNORE`或严格的 SQL 模式相同） | 错误 |
| 使用`IGNORE`和严格 SQL 模式 | 警告 | 警告 |

从表中可以得出一个结论，即当`IGNORE`关键字和严格的 SQL 模式同时生效时，`IGNORE`优先。这意味着，尽管`IGNORE`和严格的 SQL 模式在错误处理方面可以被认为具有相反的效果，但在一起使用时并不会被取消。

+   IGNORE 对语句执行的影响

+   严格 SQL 模式对语句执行的影响

##### IGNORE 对语句执行的影响

MySQL 中的几个语句支持可选的`IGNORE`关键字。此关键字导致服务器将某些类型的错误降级并生成警告。对于多行语句，将错误降级为警告可能会使一行得以处理。否则，`IGNORE`会导致语句跳到下一行而不是中止。（对于不可忽略的错误，无论有无`IGNORE`关键字，都会发生错误。）

例如：如果表`t`具有包含唯一值的主键列`i`，尝试将相同值的`i`插入多行通常会产生重复键错误：

```sql
mysql> CREATE TABLE t (i INT NOT NULL PRIMARY KEY);
mysql> INSERT INTO t (i) VALUES(1),(1);
ERROR 1062 (23000): Duplicate entry '1' for key 't.PRIMARY'
```

使用`IGNORE`，包含重复键的行仍然不会被插入，但会产生警告而不是错误：

```sql
mysql> INSERT IGNORE INTO t (i) VALUES(1),(1);
Query OK, 1 row affected, 1 warning (0.01 sec)
Records: 2  Duplicates: 1  Warnings: 1

mysql> SHOW WARNINGS;
+---------+------+-----------------------------------------+
| Level   | Code | Message                                 |
+---------+------+-----------------------------------------+
| Warning | 1062 | Duplicate entry '1' for key 't.PRIMARY' |
+---------+------+-----------------------------------------+
1 row in set (0.00 sec)
```

示例：如果表`t2`有一个`NOT NULL`列`id`，尝试插入`NULL`在严格 SQL 模式下会产生错误：

```sql
mysql> CREATE TABLE t2 (id INT NOT NULL);
mysql> INSERT INTO t2 (id) VALUES(1),(NULL),(3);
ERROR 1048 (23000): Column 'id' cannot be null
mysql> SELECT * FROM t2;
Empty set (0.00 sec)
```

如果 SQL 模式不是严格的，`IGNORE`会将`NULL`插入为列的隐式默认值（在本例中为 0），从而使行能够在不跳过的情况下处理：

```sql
mysql> INSERT INTO t2 (id) VALUES(1),(NULL),(3);
mysql> SELECT * FROM t2;
+----+
| id |
+----+
|  1 |
|  0 |
|  3 |
+----+
```

这些语句支持`IGNORE`关键字：

+   `CREATE TABLE ... SELECT`: `IGNORE`不适用于语句的`CREATE TABLE`或`SELECT`部分，而适用于由`SELECT`生成的行插入到表中。重复现有唯一键值的行将被丢弃。

+   `DELETE`: `IGNORE`使 MySQL 在删除行的过程中忽略错误。

+   `INSERT`: 使用`IGNORE`，重复现有唯一键值的行将被丢弃。将导致数据转换错误的行设置为最接近的有效值。

    对于分区表中找不到与给定值匹配的分区的情况，`IGNORE`会导致包含不匹配值的行的插入操作静默失败。

+   `LOAD DATA`, `LOAD XML`: 使用`IGNORE`，重复现有唯一键值的行将被丢弃。

+   `UPDATE`: 使用`IGNORE`时，对于唯一键值发生重复键冲突的行不会被更新。将导致数据转换错误的行更新为最接近的有效值。

`IGNORE`关键字适用于以下可忽略的错误：

```sql
ER_BAD_NULL_ERROR
ER_DUP_ENTRY
ER_DUP_ENTRY_WITH_KEY_NAME
ER_DUP_KEY
ER_NO_PARTITION_FOR_GIVEN_VALUE
ER_NO_PARTITION_FOR_GIVEN_VALUE_SILENT
ER_NO_REFERENCED_ROW_2
ER_ROW_DOES_NOT_MATCH_GIVEN_PARTITION_SET
ER_ROW_IS_REFERENCED_2
ER_SUBQUERY_NO_1_ROW
ER_VIEW_CHECK_FAILED
```

##### 严格 SQL 模式对语句执行的影响

MySQL 服务器可以在不同的 SQL 模式下运行，并且可以根据`sql_mode`系统变量的值为不同的客户端应用这些模式。在“严格”SQL 模式下，服务器将某些警告升级为错误。

例如，在非严格 SQL 模式下，将字符串`'abc'`插入整数列会将该值转换为 0 并产生警告：

```sql
mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t (i) VALUES('abc');
Query OK, 1 row affected, 1 warning (0.01 sec)

mysql> SHOW WARNINGS;
+---------+------+--------------------------------------------------------+
| Level   | Code | Message                                                |
+---------+------+--------------------------------------------------------+
| Warning | 1366 | Incorrect integer value: 'abc' for column 'i' at row 1 |
+---------+------+--------------------------------------------------------+
1 row in set (0.00 sec)
```

在严格 SQL 模式下，无效值将被拒绝并产生错误：

```sql
mysql> SET sql_mode = 'STRICT_ALL_TABLES';
Query OK, 0 rows affected (0.00 sec)

mysql> INSERT INTO t (i) VALUES('abc');
ERROR 1366 (HY000): Incorrect integer value: 'abc' for column 'i' at row 1
```

有关`sql_mode`系统变量可能设置的更多信息，请参见第 7.1.11 节，“服务器 SQL 模式”。

严格的 SQL 模式适用于以下语句，在某些值可能超出范围或将无效行插入或删除表时：

+   `ALTER TABLE`

+   `CREATE TABLE`

+   `CREATE TABLE ... SELECT`

+   `DELETE`（单表和多表）

+   `INSERT`

+   `LOAD DATA`

+   `LOAD XML`

+   `SELECT SLEEP()`

+   `UPDATE`（单表和多表）

在存储程序中，如果在严格模式下定义程序，则刚才列出的类型的单个语句将以严格的 SQL 模式执行。

严格的 SQL 模式适用于以下错误，这些错误代表一类错误，其中输入值无效或缺失。 如果值的数据类型对于列来说是错误的或可能超出范围，则该值无效。 如果要插入的新行不包含在其定义中没有显式`DEFAULT`子句的`NOT NULL`列的值，则该值缺失。

```sql
ER_BAD_NULL_ERROR
ER_CUT_VALUE_GROUP_CONCAT
ER_DATA_TOO_LONG
ER_DATETIME_FUNCTION_OVERFLOW
ER_DIVISION_BY_ZERO
ER_INVALID_ARGUMENT_FOR_LOGARITHM
ER_NO_DEFAULT_FOR_FIELD
ER_NO_DEFAULT_FOR_VIEW_FIELD
ER_TOO_LONG_KEY
ER_TRUNCATED_WRONG_VALUE
ER_TRUNCATED_WRONG_VALUE_FOR_FIELD
ER_WARN_DATA_OUT_OF_RANGE
ER_WARN_NULL_TO_NOTNULL
ER_WARN_TOO_FEW_RECORDS
ER_WRONG_ARGUMENTS
ER_WRONG_VALUE_FOR_TYPE
WARN_DATA_TRUNCATED
```

注意

由于持续的 MySQL 开发定义了新的错误，可能存在不在前述列表中的错误，严格的 SQL 模式适用于这些错误。
