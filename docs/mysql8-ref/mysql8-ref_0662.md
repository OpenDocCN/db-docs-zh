# 11.2.5 函数名称解析和解析

> 原文：[`dev.mysql.com/doc/refman/8.0/en/function-resolution.html`](https://dev.mysql.com/doc/refman/8.0/en/function-resolution.html)

MySQL 支持内置（本地）函数、可加载函数和存储函数。本节描述了服务器如何识别内置函数的名称是作为函数调用还是作为标识符使用，以及在存在具有给定名称的不同类型函数时服务器如何确定使用哪个函数。

+   内置函数名称解析

+   函数名称解析

#### 内置函数名称解析

解析器使用默认规则解析内置函数的名称。这些规则可以通过启用`IGNORE_SPACE` SQL 模式来更改。

当解析器遇到内置函数名称时，必须确定该名称是表示函数调用还是非表达式引用标识符（例如表或列名称）。例如，在以下语句中，对`count`的第一个引用是函数调用，而第二个引用是表名：

```sql
SELECT COUNT(*) FROM mytable;
CREATE TABLE count (i INT);
```

解析器应仅在解析预期为表达式的内容时将内置函数名称识别为表示函数调用。也就是说，在非表达式上下文中，函数名称允许作为标识符。

然而，一些内置函数具有特殊的解析或实现考虑因素，因此解析器默认使用以下规则来区分它们的名称是作为函数调用还是作为非表达式上下文中的标识符：

+   要在表达式中使用名称作为函数调用，名称和后面的`(`括号字符之间不能有空格。

+   相反，要将函数名称用作标识符，必须不紧跟在括号后面。

要求函数调用在名称和括号之间没有空格的规定仅适用于具有特殊考虑因素的内置函数。`COUNT`就是这样一个名称。`sql/lex.h`源文件列出了这些特殊函数的名称，后续空格决定了它们的解释：由`symbols[]`数组中的`SYM_FN()`宏定义的名称。

下面列出了在 MySQL 8.0 中受`IGNORE_SPACE`设置影响并在`sql/lex.h`源文件中列为特殊的函数列表。您可能会发现将无空格要求视为适用于所有函数调用最容易。

+   `ADDDATE`

+   `BIT_AND`

+   `BIT_OR`

+   `BIT_XOR`

+   `CAST`

+   `COUNT`

+   `CURDATE`

+   `CURTIME`

+   `DATE_ADD`

+   `DATE_SUB`

+   `EXTRACT`

+   `GROUP_CONCAT`

+   `MAX`

+   `MID`

+   `MIN`

+   `NOW`

+   `POSITION`

+   `SESSION_USER`

+   `STD`

+   `STDDEV`

+   `STDDEV_POP`

+   `STDDEV_SAMP`

+   `SUBDATE`

+   `SUBSTR`

+   `SUBSTRING`

+   `SUM`

+   `SYSDATE`

+   `SYSTEM_USER`

+   `TRIM`

+   `VARIANCE`

+   `VAR_POP`

+   `VAR_SAMP`

对于在`sql/lex.h`中未列为特殊的函数，空格并不重要。只有在表达式上下文中使用时，它们才会被解释为函数调用，否则可以自由地用作标识符。`ASCII`就是这样一个名称。然而，对于这些未受影响的函数名称，在表达式上下文中的解释可能有所不同：如果存在具有给定名称的内置函数，则`*`func_name`* ()`将被解释为内置函数；如果没有，则`*`func_name`* ()`将被解释为可加载函数或存储函数（如果存在具有该名称的函数）。

`IGNORE_SPACE` SQL 模式可用于修改解析器对对空格敏感的函数名称的处理方式：

+   禁用`IGNORE_SPACE`后，当函数名称和后续括号之间没有空格时，解析器将名称解释为函数调用。即使在非表达式上下文中使用函数名称也会发生这种情况：

    ```sql
    mysql> CREATE TABLE count(i INT);
    ERROR 1064 (42000): You have an error in your SQL syntax ...
    near 'count(i INT)'
    ```

    为了消除错误并使名称被视为标识符，可以在名称后面使用空格或将其写为带引号的标识符（或两者都使用）：

    ```sql
    CREATE TABLE count (i INT);
    CREATE TABLE `count`(i INT);
    CREATE TABLE `count` (i INT);
    ```

+   启用`IGNORE_SPACE`后，解析器放宽了函数名称和后续括号之间不能有空格的要求。这样可以更灵活地编写函数调用。例如，以下任一函数调用都是合法的：

    ```sql
    SELECT COUNT(*) FROM mytable;
    SELECT COUNT (*) FROM mytable;
    ```

    然而，启用`IGNORE_SPACE`也会导致解析器将受影响的函数名称视为保留字（请参阅第 11.3 节，“关键字和保留字”）。这意味着名称后面的空格不再表示其用作标识符。该名称可以在带有或不带有后续空格的函数调用中使用，但在非表达式上下文中除非加引号，否则会导致语法错误。例如，启用`IGNORE_SPACE`后，以下两个语句都会因解析器将`count`解释为保留字而导致语法错误：

    ```sql
    CREATE TABLE count(i INT);
    CREATE TABLE count (i INT);
    ```

    要在非表达式上下文中使用函数名称，请将其写为带引号的标识符：

    ```sql
    CREATE TABLE `count`(i INT);
    CREATE TABLE `count` (i INT);
    ```

要启用`IGNORE_SPACE` SQL 模式，请使用以下语句：

```sql
SET sql_mode = 'IGNORE_SPACE';
```

`IGNORE_SPACE`也被某些其他复合模式启用，例如`ANSI`将其包含在其值中：

```sql
SET sql_mode = 'ANSI';
```

查看第 7.1.11 节，“服务器 SQL 模式”，以查看哪些复合模式启用了`IGNORE_SPACE`。

为了最大程度地减少 SQL 代码对`IGNORE_SPACE`设置的依赖，请遵循以下准则：

+   避免创建与内置函数同名的���加载函数或存储函数。

+   避免在非表达式上下文中使用函数名。例如，这些语句使用了 `count`（受 `IGNORE_SPACE` 影响的函数名之一），因此如果启用了 `IGNORE_SPACE`，无论名称后是否有空格，它们都会失败：

    ```sql
    CREATE TABLE count(i INT);
    CREATE TABLE count (i INT);
    ```

    如果必须在非表达式上下文中使用函数名，请将其写为引号标识符：

    ```sql
    CREATE TABLE `count`(i INT);
    CREATE TABLE `count` (i INT);
    ```

#### 函数名解析

以下规则描述了服务器如何解析函数名的引用以进行函数创建和调用：

+   内置函数和可加载函数

    如果尝试创建与内置函数同名的可加载函数，则会出现错误。

    `IF NOT EXISTS`（从 MySQL 8.0.29 开始提供）在这种情况下没有效果。有关更多信息，请参阅 Section 15.7.4.1, “CREATE FUNCTION Statement for Loadable Functions”。

+   内置函数和存储函数

    可以创建与内置函数同名的存储函数，但要调用存储函数，必须使用模式名称进行限定。例如，如果在 `test` 模式中创建了名为 `PI` 的存储函数，则调用时应为 `test.PI()`，因为服务器会将不带限定符的 `PI()` 解析为对内置函数的引用。如果存储函数名与内置函数名冲突，服务器会生成警告。可以使用 `SHOW WARNINGS` 显示警告。

    `IF NOT EXISTS`（MySQL 8.0.29 及更高版本）在这种情况下没有效果；请参阅 Section 15.1.17, “CREATE PROCEDURE and CREATE FUNCTION Statements”。

+   可加载函数和存储函数

    可以创建与现有可加载函数同名的存储函数，反之亦然。如果建议的存储函数名与现有可加载函数名冲突，或者建议的可加载函数名与现有存储函数名相同，则服务器会生成警告。在这两种情况下，一旦两个函数都存在，以后在调用存储函数时必须使用模式名称进行限定；在这种情况下，服务器假定未限定的名称指的是可加载函数。

    从 MySQL 8.0.29 开始，`IF NOT EXISTS` 在 `CREATE FUNCTION` 语句中得到支持，但在这种情况下没有效果。

    在 MySQL 8.0.28 之前，可以创建与现有可加载函数同名的存储函数，但反之则不行（Bug #33301931）。

前述函数名解析规则对于升级到实现新内置函数的 MySQL 版本具有影响：

+   如果您已经创建了一个具有特定名称的可加载函数，并将 MySQL 升级到实现了同名新内置函数的版本，则该可加载函数将变得无法访问。为了纠正这个问题，使用 `DROP FUNCTION` 删除可加载函数，然后使用 `CREATE FUNCTION` 以不冲突的不同名称重新创建可加载函数。然后修改任何受影响的代码以使用新名称。

+   如果 MySQL 的新版本实现了一个与现有存储函数同名的内置函数或可加载函数，你有两个选择：将存储函数重命名为不冲突的名称，或者修改任何尚未这样做的对该函数的调用以使用模式限定符（`*`schema_name`*.*`func_name`*()` 语法）。在任何情况下，相应地修改任何受影响的代码。
