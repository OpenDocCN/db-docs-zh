# 15.2.7 插入语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/insert.html`](https://dev.mysql.com/doc/refman/8.0/en/insert.html)

15.2.7.1 插入...选择语句

15.2.7.2 插入...在重复键更新语句中

15.2.7.3 延迟插入语句

```sql
INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [(*col_name* [, *col_name*] ...)]
    { {VALUES | VALUE} (*value_list*) [, (*value_list*)] ... }
    [AS *row_alias*[(*col_alias* [, *col_alias*] ...)]]
    [ON DUPLICATE KEY UPDATE *assignment_list*]

INSERT [LOW_PRIORITY | DELAYED | HIGH_PRIORITY] [IGNORE]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    SET *assignment_list*
    [AS *row_alias*[(*col_alias* [, *col_alias*] ...)]]
    [ON DUPLICATE KEY UPDATE *assignment_list*]

INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [(*col_name* [, *col_name*] ...)]
    { SELECT ... 
      | TABLE *table_name* 
      | VALUES *row_constructor_list*
    }
    [ON DUPLICATE KEY UPDATE *assignment_list*]

*value*:
    {*expr* | DEFAULT}

*value_list*:
    *value* [, *value*] ...

*row_constructor_list*:
    ROW(*value_list*)[, ROW(*value_list*)][, ...]

*assignment*:
    *col_name* = 
          *value*
        | [*row_alias*.]*col_name*
        | [*tbl_name*.]*col_name*
        | [*row_alias*.]*col_alias*

*assignment_list*:
    *assignment* [, *assignment*] ...
```

`INSERT`将新行插入到现有表中。`INSERT ... VALUES`、`INSERT ... VALUES ROW()`和`INSERT ... SET`形式的语句根据明确指定的值插入行。`INSERT ... SELECT`形式从另一个表或表中选择的行插入。您还可以在 MySQL 8.0.19 及更高版本中使用`INSERT ... TABLE`从单个表中插入行。带有`ON DUPLICATE KEY UPDATE`子句的`INSERT`使得如果要插入的行会导致在`UNIQUE`索引或`PRIMARY KEY`中出现重复值，则现有行可以被更新。在 MySQL 8.0.19 及更高版本中，可以使用带有一个或多个可选列别名的行别名与`ON DUPLICATE KEY UPDATE`一起引用要插入的行。

有关`INSERT ... SELECT`和`INSERT ... ON DUPLICATE KEY UPDATE`的更多信息，请参见第 15.2.7.1 节，“插入...选择语句”和第 15.2.7.2 节，“插入...在重复键更新语句中”。

在 MySQL 8.0 中，服务器接受但忽略`DELAYED`关键字。关于这一点的原因，请参见第 15.2.7.3 节，“延迟插入语句”，

插入到表中需要表的`INSERT`权限。如果使用`ON DUPLICATE KEY UPDATE`子句，并且重复键导致执行`UPDATE`，则语句需要更新要更新的列的`UPDATE`权限。对于只读取但不修改的列，您只需要`SELECT`权限（例如，在`ON DUPLICATE KEY UPDATE`子句中仅在赋值的右侧引用的列）。

在插入分区表时，可以控制哪些分区和子分区接受新行。`PARTITION`子句接受表的一个或多个分区或子分区（或两者）的逗号分隔名称列表。如果给定`INSERT`语句要插入的任何行与列出的分区之一不匹配，则该`INSERT`语句将失败，并显示错误消息找到一个不匹配给定分区集的行。有关更多信息和示例，请参见第 26.5 节，“分区选择”。

*`tbl_name`*是应该插入行的表。按照以下方式指定语句提供值的列：

+   在表名后提供一个用逗号分隔的列名的括号列表。在这种情况下，每个命名列的值必须由`VALUES`列表、`VALUES ROW()`列表或`SELECT`语句提供。对于`INSERT TABLE`形式，源表中的列数必须与要插入的列数相匹配。

+   如果对于`INSERT ... VALUES`或`INSERT ... SELECT`没有指定列名列表，则必须通过`VALUES`列表、`SELECT`语句或`TABLE`语句为表中的每个列提供值。如果不知道表中列的顺序，请使用`DESCRIBE *`tbl_name`*`来查找。

+   `SET`子句通过列名明确指定，以及为每个列分配的值。

列值可以以多种方式给出：

+   如果未启用严格的 SQL 模式，则未明确给定值的任何列都将设置为其默认（显式或隐式）值。例如，如果指定的列列表未命名表中的所有列，则未命名列将设置为其默认值。默认值分配在第 13.6 节，“数据类型默认值”中描述。另请参阅第 1.6.3.3 节，“对无效数据的强制约束”。

    如果启用了严格的 SQL 模式，`INSERT`语句将在没有为没有默认值的每个列指定显式值时生成错误。参见第 7.1.11 节，“服务器 SQL 模式”。

+   如果列列表和`VALUES`列表都为空，`INSERT`将创建一行，其中每个列都设置为其默认值：

    ```sql
    INSERT INTO *tbl_name* () VALUES();
    ```

    如果未启用严格模式，MySQL 将对任何没有明确定义默认值的列使用隐式默认值。如果启用了严格模式，如果任何列没有默认值，则会发生错误。

+   使用关键字`DEFAULT`将列明确设置为其默认值。这样可以更轻松地编写`INSERT`语句，为除少数列外的所有列分配值，因为它使您可以避免编写不包括表中每列值的不完整`VALUES`列表。否则，您必须提供与`VALUES`列表中每个值对应的列名列表。

+   如果插入生成的列，则唯一允许的值是`DEFAULT`。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”。

+   在表达式中，您可以使用`DEFAULT(*`col_name`*)`来为列*`col_name`*生成默认值。

+   如果提供列值的表达式*`expr`*的数据类型与列数据类型不匹配，则可能发生类型转换。给定值的转换可能导致根据列类型而插入不同的值。例如，将字符串`'1999.0e-2'`插入到`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")、`FLOAT` - FLOAT, DOUBLE")、`DECIMAL(10,6)` - DECIMAL, NUMERIC")或`YEAR`列中，分别插入值`1999`、`19.9921`、`19.992100`或`1999`。存储在`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")和`YEAR`列中的值为`1999`，因为字符串转换为数字仅查看字符串的初始部分，该部分可能被视为有效整数或年份。对于`FLOAT` - FLOAT, DOUBLE")和`DECIMAL` - DECIMAL, NUMERIC")列，字符串转换为数字将整个字符串视为有效的数值。

+   表达式*`expr`*可以引用先前在值列表中设置的任何列。例如，您可以这样做，因为`col2`的值引用了先前分配的`col1`：

    ```sql
    INSERT INTO *tbl_name* (col1,col2) VALUES(15,col1*2);
    ```

    但是以下内容是不合法的，因为`col1`的值引用了在`col1`之后分配的`col2`：

    ```sql
    INSERT INTO *tbl_name* (col1,col2) VALUES(col2*2,15);
    ```

    对于包含`AUTO_INCREMENT`值的列会出现异常。因为`AUTO_INCREMENT`值是在其他值分配之后生成的，对`AUTO_INCREMENT`列的任何引用在赋值时返回`0`。

使用`VALUES`语法的`INSERT`语句可以插入多行。要做到这一点，包含多个逗号分隔的列值列表，列表用括号括起并用逗号分隔。示例：

```sql
INSERT INTO *tbl_name* (a,b,c)
    VALUES(1,2,3), (4,5,6), (7,8,9);
```

每个值列表必须包含与要插入的每行的值数量完全相同的值。以下语句是无效的，因为它包含一个包含九个值的列表，而不是三个包含三个值的列表：

```sql
INSERT INTO *tbl_name* (a,b,c) VALUES(1,2,3,4,5,6,7,8,9);
```

在这种情况下，`VALUE`是`VALUES`的同义词。两者都不暗示值列表的数量，也不暗示每个列表的值的数量。无论是单个值列表还是多个列表，以及每个列表中的值的数量，都可以使用任一项。

使用`VALUES ROW()`语法的`INSERT`语句也可以插入多行。在这种情况下，每个值列表必须包含在一个`ROW()`（行构造函数）中，就像这样：

```sql
INSERT INTO *tbl_name* (a,b,c)
    VALUES ROW(1,2,3), ROW(4,5,6), ROW(7,8,9);
```

可以使用`ROW_COUNT()` SQL 函数或`mysql_affected_rows()` C API 函数获取`INSERT`的受影响行数。请参阅第 14.15 节，“信息函数”和 mysql_affected_rows()。

如果您使用`INSERT ... VALUES`或`INSERT ... VALUES ROW()`插入多个值列表，或者`INSERT ... SELECT`或`INSERT ... TABLE`，则语句以以下格式返回信息字符串：

```sql
Records: *N1* Duplicates: *N2* Warnings: *N3*
```

如果您正在使用 C API，可以通过调用`mysql_info()`函数获取信息字符串。请参阅 mysql_info()。

`Records`表示语句处理的行数。（这不一定是实际插入的行数，因为`Duplicates`可能不为零。）`Duplicates`表示由于重复某些现有唯一索引值而无法插入的行数。`Warnings`表示尝试插入某种方式有问题的列值的次数。警告可能在以下任何条件下发生：

+   将`NULL`插入已声明为`NOT NULL`的列。对于多行`INSERT`语句或`INSERT INTO ... SELECT`语句，该列设置为列数据类型的隐式默认值。对于数值类型，这是`0`，对于字符串类型，这是空字符串（`''`），对于日期和时间类型，这是“零”值。由于服务器不会检查`SELECT`的结果集是否返回单行，因此`INSERT INTO ... SELECT`语句与多行插入处理方式相同。 （对于单行`INSERT`，当将`NULL`插入到`NOT NULL`列时，不会发出警告。相反，该语句将因错误而失败。）

+   将数值列设置为超出列范围的值。该值被截断为范围的最近端点。

+   将值赋给数值列，例如`'10.34 a'`。尾随的非数字文本被剥离，剩余的数值部分被插入。如果字符串值没有前导数值部分，则该列设置为`0`。

+   将字符串插入字符串列（`CHAR`、`VARCHAR`、`TEXT`或`BLOB`），其超过列的最大长度。该值被截断为列的最大长度。

+   将值插入日期或时间列，该值对于数据类型是非法的。该列设置为该类型的适当零值。

+   有关涉及`AUTO_INCREMENT`列值的`INSERT`示例，请参见第 5.6.9 节“使用 AUTO_INCREMENT”。

    如果`INSERT`向具有`AUTO_INCREMENT`列的表中插入一行，则可以使用`LAST_INSERT_ID()` SQL 函数或`mysql_insert_id()` C API 函数找到用于该列的值。

    注意

    这两个函数的行为并不总是相同。关于与`AUTO_INCREMENT`列有关的`INSERT`语句的行为在第 14.15 节“信息函数”和 mysql_insert_id()中进一步讨论。

`INSERT`语句支持以下修饰符：

+   如果使用`LOW_PRIORITY`修饰符，`INSERT`的执行将延迟，直到没有其他客户端从表中读取数据。这包括在现有客户端正在读取数据时开始读取数据的其他客户端，以及`INSERT LOW_PRIORITY`语句正在等待的情况。因此，发出`INSERT LOW_PRIORITY`语句的客户端可能需要等待很长时间。

    `LOW_PRIORITY`仅影响仅使用表级锁定的存储引擎（如`MyISAM`，`MEMORY`和`MERGE`）。

    注意

    `LOW_PRIORITY`通常不应与`MyISAM`表一起使用，因为这样做会禁用并发插入。参见第 10.11.3 节，“并发插入”。

+   如果指定了`HIGH_PRIORITY`，它会覆盖服务器在启动时使用`--low-priority-updates`选项的效果。它还会导致不使用并发插入。参见第 10.11.3 节，“并发插入”。

    `HIGH_PRIORITY`仅影响仅使用表级锁定的存储引擎（如`MyISAM`，`MEMORY`和`MERGE`）。

+   如果使用`IGNORE`修饰符，执行`INSERT`语句时发生的可忽略错误将被忽略。例如，如果没有`IGNORE`，在表中重复现有`UNIQUE`索引或`PRIMARY KEY`值的行会导致重复键错误并中止语句。使用`IGNORE`，该行将被丢弃，不会发生错误。被忽略的错误会生成警告。

    `IGNORE`对于插入到未找到匹配给定值的分区表具有类似的效果。如果没有`IGNORE`，这样的`INSERT`语句将因错误而中止。当使用`INSERT IGNORE`时，对于包含不匹配值的行，插入操作会悄悄失败，但会插入匹配的行。例如，请参见第 26.2.2 节，“LIST 分区”。

    如果未指定`IGNORE`，会触发错误的数据转换将中止语句。使用`IGNORE`，无效值将调整为最接近的值并插入；会产生警告，但语句不会中止。您可以使用`mysql_info()` C API 函数确定实际插入表中的行数。

    有关更多信息，请参见 IGNORE 对语句执行的影响。

    你可以使用`REPLACE`来覆盖旧行，而不是使用`INSERT`。`REPLACE`是对待包含重复旧行的唯一键值的新行的处理的对应项：新行取代旧行而不是被丢弃。参见 Section 15.2.12, “REPLACE Statement”。

+   如果你指定了`ON DUPLICATE KEY UPDATE`，并且插入了一行会导致在`UNIQUE`索引或`PRIMARY KEY`中出现重复值的情况，旧行将被`UPDATE`。每行的受影响行数为 1，如果该行被插入为新行，则为 2，如果更新了现有行，则为 0。如果在连接到**mysqld**时，通过在`mysql_real_connect()` C API 函数中指定`CLIENT_FOUND_ROWS`标志，受影响行数为 1（而不是 0），如果现有行被设置为其当前值。参见 Section 15.2.7.2, “INSERT ... ON DUPLICATE KEY UPDATE Statement”。

+   `INSERT DELAYED`在 MySQL 5.6 中已被弃用，并计划最终移除。在 MySQL 8.0 中，`DELAYED`修饰符被接受但被忽略。请改用`INSERT`（不带`DELAYED`）。参见 Section 15.2.7.3, “INSERT DELAYED Statement”。
