# 15.2.9 LOAD DATA 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/load-data.html`](https://dev.mysql.com/doc/refman/8.0/en/load-data.html)

```sql
LOAD DATA
    [LOW_PRIORITY | CONCURRENT] [LOCAL]
    INFILE '*file_name*'
    [REPLACE | IGNORE]
    INTO TABLE *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [CHARACTER SET *charset_name*]
    [{FIELDS | COLUMNS}
        [TERMINATED BY '*string*']
        [[OPTIONALLY] ENCLOSED BY '*char*']
        [ESCAPED BY '*char*']
    ]
    [LINES
        [STARTING BY '*string*']
        [TERMINATED BY '*string*']
    ]
    [IGNORE *number* {LINES | ROWS}]
    [(*col_name_or_user_var*
        [, *col_name_or_user_var*] ...)]
    [SET *col_name*={*expr* | DEFAULT}
        [, *col_name*={*expr* | DEFAULT}] ...]
```

`LOAD DATA` 语句以非常高的速度从文本文件中读取行到表中。文件可以从服务器主机或客户端主机读取，这取决于是否给出 `LOCAL` 修饰符。`LOCAL` 还会影响数据解释和错误处理。

`LOAD DATA` 是 `SELECT ... INTO OUTFILE` 的补充。（参见 Section 15.2.13.1, “SELECT ... INTO 语句”。）要将表中的数据写入文件，请使用 `SELECT ... INTO OUTFILE`。要将文件中的数据读回表中，请使用 `LOAD DATA`。这两个语句的 `FIELDS` 和 `LINES` 子句的语法相同。

**mysqlimport** 实用程序提供了另一种加载数据文件的方式；它通过向服务器发送 `LOAD DATA` 语句来操作。请参阅 Section 6.5.5, “mysqlimport — 数据导入程序”。

有关 `INSERT` 与 `LOAD DATA` 的效率以及加速 `LOAD DATA` 的信息，请参阅 Section 10.2.5.1, “优化 INSERT 语句”。

+   非 LOCAL 与 LOCAL 操作

+   输入文件字符集

+   输入文件位置

+   安全要求

+   重复键和错误处理

+   索引处理

+   字段和行处理

+   列列表规范

+   输入预处理

+   列值赋值

+   分区表支持

+   并发考虑

+   语句结果信息

+   复制考虑

+   杂项主题

#### 非 LOCAL 与 LOCAL 操作的区别

`LOCAL`修饰符影响`LOAD DATA`的这些方面，与非`LOCAL`操作相比：

+   它改变了输入文件的预期位置；参见输入文件位置。

+   它改变了语句的安全要求；参见安全要求。

+   它对输入文件内容的解释和错误处理具有与`IGNORE`修饰符相同的效果；参见重复键和错误处理，以及列值分配。

只有在服务器和客户端都配置为允许时，`LOCAL`才有效。例如，如果**mysqld**启动时禁用了`local_infile`系统变量，`LOCAL`会产生错误。参见第 8.1.6 节，“LOAD DATA LOCAL 的安全考虑”。

#### 输入文件字符集

文件名必须以文字字符串形式给出。在 Windows 上，路径名中的反斜杠应指定为正斜杠或双反斜杠。服务器使用由`character_set_filesystem`系统变量指示的字符集解释文件名。

默认情况下，服务器使用由`character_set_database`系统变量指示的字符集解释文件内容。如果文件内容使用与此默认值不同的字符集，最好通过使用`CHARACTER SET`子句指定该字符集。字符集为`binary`表示“无转换”。

`SET NAMES`和`character_set_client`的设置不影响文件内容的解释。

`LOAD DATA` 将文件中的所有字段解释为具有相同字符集，而不管字段值加载到的列的数据类型如何。为了正确解释文件，您必须确保它是用正确的字符集编写的。例如，如果您使用 **mysqldump -T** 写入数据文件，或者通过在 **mysql** 中发出 `SELECT ... INTO OUTFILE` 语句来写入数据文件，请务必使用 `--default-character-set` 选项以在加载文件时使用的字符集中写入输出。

注意

不可能加载使用 `ucs2`、`utf16`、`utf16le` 或 `utf32` 字符集的数��文件。

#### 输入文件位置

这些规则确定了 `LOAD DATA` 输入文件的位置：

+   如果未指定 `LOCAL`，则文件必须位于服务器主机上。服务器直接读取文件，定位如下：

    +   如果文件名是绝对路径名，服务器将按照给定的路径使用它。

    +   如果文件名是带有前导组件的相对路径名，服务器将在其数据目录相对于文件查找。

    +   如果文件名没有前导组件，服务器将在默认数据库的数据库目录中查找文件。

+   如果指定了 `LOCAL`，则文件必须位于客户端主机上。客户端程序读取文件，定位如下：

    +   如果文件名是绝对路径名，客户端程序将按照给定的路径使用它。

    +   如果文件名是相对路径名，客户端程序将在其调用目录相对于文件查找。

    当使用 `LOCAL` 时，客户端程序读取文件并将其内容发送到服务器。服务器在存储临时文件的目录中创建文件的副本。请参阅 Section B.3.3.5, “Where MySQL Stores Temporary Files”。在此目录中没有足够空间来存储副本可能导致 `LOAD DATA LOCAL` 语句失败。

非 `LOCAL` 规则意味着服务器将相对于其数据目录读取名为 `./myfile.txt` 的文件，而将名为 `myfile.txt` 的文件从默认数据库的数据库目录中读取。例如，如果在 `db1` 是默认数据库的情况下执行以下 `LOAD DATA` 语句，服务器将从 `db1` 的数据库目录中读取文件 `data.txt`，即使该语句明确将文件加载到 `db2` 数据库中的表中：

```sql
LOAD DATA INFILE 'data.txt' INTO TABLE db2.my_table;
```

注意

服务器还使用非`LOCAL`规则来定位`IMPORT TABLE`语句的`.sdi`文件。

#### 安全要求

对于非`LOCAL`加载操作，服务器会读取位于服务器主机上的文本文件，因此必须满足以下安全要求：

+   您必须具有`FILE`权限。请参阅 Section 8.2.2, “MySQL 提供的权限”。

+   该操作受`secure_file_priv`系统变量设置的影响：

    +   如果变量值是非空目录名称，则文件必须位于该目录中。

    +   如果变量值为空（这是不安全的），文件只需对服务器可读。

对于`LOCAL`加载操作，客户端程序会读取位于客户端主机上的文本文件。由于文件内容通过客户端传输到服务器，使用`LOCAL`比服务器直接访问文件要慢一点。另一方面，您不需要`FILE`权限，并且文件可以位于客户端程序可以访问的任何目录中。

#### 重复键和错误处理

`REPLACE`和`IGNORE`修饰符控制对具有唯一键值（`PRIMARY KEY`或`UNIQUE`索引值）上现有表行重复的新（输入）行的处理：

+   使用`REPLACE`，具有与现有行中唯一键值相同值的新行将替换现有行。请参阅 Section 15.2.12, “REPLACE 语句”。

+   使用`IGNORE`，具有与唯一键值上现有行重复的新行将被丢弃。有关更多信息，请参阅 IGNORE 对语句执行的影响。

`LOCAL`修饰符与`IGNORE`具有相同的效果。这是因为服务器无法在操作中途停止文件的传输。

如果未指定`REPLACE`，`IGNORE`或`LOCAL`，当发现重复键值时会发生错误，并且文本文件的其余部分将被忽略。

除了影响刚刚描述的重复键处理之外，`IGNORE`和`LOCAL`还会影响错误处理：

+   没有`IGNORE`或`LOCAL`，数据解释错误会终止操作。

+   使用`IGNORE`或`LOCAL`，数据解释错误变为警告，并且加载操作会继续，即使 SQL 模式是限制性的。有关示例，请参阅列值分配。

#### 索引处理

在加载操作期间忽略外键约束，请在执行`LOAD DATA`之前执行`SET foreign_key_checks = 0`语句。

如果在空的`MyISAM`表上使用`LOAD DATA`，则所有非唯一索引将在单独的批处理中创建（与`REPAIR TABLE`一样）。通常，当您有许多索引时，这使得`LOAD DATA`速度更快。在一些极端情况下，您可以通过在将文件加载到表中之前使用`ALTER TABLE ... DISABLE KEYS`关闭它们来更快地创建索引，并在加载文件后使用`ALTER TABLE ... ENABLE KEYS`重新创建索引。请参见第 10.2.5.1 节，“优化 INSERT 语句”。

#### 字段和行处理

对于`LOAD DATA`和`SELECT ... INTO OUTFILE`语句，`FIELDS`和`LINES`子句的语法相同。两个子句都是可选的，但如果两者都指定，则`FIELDS`必须在`LINES`之前。

如果指定了`FIELDS`子句，则其每个子句（`TERMINATED BY`，`[OPTIONALLY] ENCLOSED BY`和`ESCAPED BY`）也是可选的，除非您必须至少指定其中一个。这些子句的参数只允许包含 ASCII 字符。

如果不指定`FIELDS`或`LINES`子句，则默认值与您编写以下内容相同：

```sql
FIELDS TERMINATED BY '\t' ENCLOSED BY '' ESCAPED BY '\\'
LINES TERMINATED BY '\n' STARTING BY ''
```

在 SQL 语句中的字符串中，反斜杠是 MySQL 转义字符。因此，要指定一个字面上的反斜杠，您必须为解释为单个反斜杠的值指定两个反斜杠。转义序列`'\t'`和`'\n'`分别指定制表符和换行符。

换句话说，当读取输入时，`LOAD DATA`的默认值如下：

+   在换行符处查找行边界。

+   不要跳过任何行前缀。

+   在制表符处将行分割为字段。

+   不要期望字段被包含在任何引号字符内。

+   将由转义字符`\`引导的字符解释为转义序列。例如，`\t`，`\n`和`\\`分别表示制表符，换行符和反斜杠。有关转义序列的完整列表，请参见稍后讨论的`FIELDS ESCAPED BY`。

相反，当写入输出时，`SELECT ... INTO OUTFILE`的默认值如下：

+   在字段之间写入制表符。

+   不要在任何引号字符内包含字段。

+   使用`\`来转义字段值中出现的制表符，换行符或`\`。

+   在行末写入换行符。

注意

对于在 Windows 系统上生成的文本文件，正确的文件读取可能需要`LINES TERMINATED BY '\r\n'`，因为 Windows 程序通常使用两个字符作为行终止符。一些程序，如**WordPad**，在写文件时可能使用`\r`作为行终止符。要读取这样的文件，请使用`LINES TERMINATED BY '\r'`。

如果所有输入行都有一个要忽略的公共前缀，你可以使用`LINES STARTING BY '*`prefix_string`*'`来跳过前缀*及其之前的任何内容*。如果一行不包含前缀，则整行将被跳过。假设你发出以下语句：

```sql
LOAD DATA INFILE '/tmp/test.txt' INTO TABLE test
  FIELDS TERMINATED BY ','  LINES STARTING BY 'xxx';
```

如果数据文件看起来像这样：

```sql
xxx"abc",1
something xxx"def",2
"ghi",3
```

结果行是`("abc",1)`和`("def",2)`。文件中的第三行被跳过，因为它不包含前缀。

`IGNORE *`number`* LINES`子句可用于忽略文件开头的行。例如，你可以使用`IGNORE 1 LINES`跳过包含列名的初始标题行：

```sql
LOAD DATA INFILE '/tmp/test.txt' INTO TABLE test IGNORE 1 LINES;
```

当你使用`SELECT ... INTO OUTFILE`与`LOAD DATA`结合，将数据库中的数据写入文件，然后稍后将文件中的数据读回数据库时，两个语句的字段和行处理选项必须匹配。否则，`LOAD DATA`无法正确解释文件的内容。假设你使用`SELECT ... INTO OUTFILE`来写一个以逗号分隔字段的文件：

```sql
SELECT * INTO OUTFILE 'data.txt'
  FIELDS TERMINATED BY ','
  FROM table2;
```

要读取逗号分隔的文件，正确的语句是：

```sql
LOAD DATA INFILE 'data.txt' INTO TABLE table2
  FIELDS TERMINATED BY ',';
```

如果你尝试用下面显示的语句读取文件，它将无法工作，因为它指示`LOAD DATA`在字段之间查找制表符：

```sql
LOAD DATA INFILE 'data.txt' INTO TABLE table2
  FIELDS TERMINATED BY '\t';
```

可能的结果是每个输入行被解释为单个字段。

`LOAD DATA`可用于读取从外部来源获取的文件。例如，许多程序可以以逗号分隔值（CSV）格式导出数据，使得行由逗号分隔的字段并用双引号括起，带有列名的初始行。如果这样的文件中的行以回车/换行对终止，那么这里显示的语句说明了你将用于加载文件的字段和行处理选项：

```sql
LOAD DATA INFILE 'data.txt' INTO TABLE *tbl_name*
  FIELDS TERMINATED BY ',' ENCLOSED BY '"'
  LINES TERMINATED BY '\r\n'
  IGNORE 1 LINES;
```

如果输入值不一定被引号括起来，可以在`ENCLOSED BY`选项之前使用`OPTIONALLY`。

任何字段或行处理选项都可以指定空字符串（`''`）。如果不为空，则`FIELDS [OPTIONALLY] ENCLOSED BY`和`FIELDS ESCAPED BY`值必须是单个字符。`FIELDS TERMINATED BY`、`LINES STARTING BY`和`LINES TERMINATED BY`值可以是多个字符。例如，要写入由回车/换行对终止的行，或者读取包含这些行的文件，请指定`LINES TERMINATED BY '\r\n'`子句。

要读取包含以`%%`为分隔的笑话的文件，可以这样做

```sql
CREATE TABLE jokes
  (a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  joke TEXT NOT NULL);
LOAD DATA INFILE '/tmp/jokes.txt' INTO TABLE jokes
  FIELDS TERMINATED BY ''
  LINES TERMINATED BY '\n%%\n' (joke);
```

`FIELDS [OPTIONALLY] ENCLOSED BY`控制字段的引用。对于输出（`SELECT ... INTO OUTFILE`），如果省略`OPTIONALLY`这个词，所有字段都将被`ENCLOSED BY`字符包围。这里展示了这种输出的示例（使用逗号作为字段分隔符）：

```sql
"1","a string","100.20"
"2","a string containing a , comma","102.20"
"3","a string containing a \" quote","102.20"
"4","a string containing a \", quote and comma","102.20"
```

如果指定了`OPTIONALLY`，则`ENCLOSED BY`字符仅用于封装具有字符串数据类型的列的值（例如`CHAR`、`BINARY`、`TEXT`或`ENUM`)：

```sql
1,"a string",100.20
2,"a string containing a , comma",102.20
3,"a string containing a \" quote",102.20
4,"a string containing a \", quote and comma",102.20
```

字段值内部的`ENCLOSED BY`字符通过在其前面加上`ESCAPED BY`字符来转义。此外，如果指定空的`ESCAPED BY`值，则可能会无意中生成无法被`LOAD DATA`正确读取的输出。例如，如果转义字符为空，则前面显示的输出将如下所示。请注意，第四行中的第二个字段包含在引号后面的逗号，这（错误地）似乎终止了该字段：

```sql
1,"a string",100.20
2,"a string containing a , comma",102.20
3,"a string containing a " quote",102.20
4,"a string containing a ", quote and comma",102.20
```

对于输入，如果存在`ENCLOSED BY`字符，则会从字段值的两端剥离该字符。（无论是否指定`OPTIONALLY`，都会这样处理；`OPTIONALLY`对输入解释没有影响。）由`ESCAPED BY`字符引导的`ENCLOSED BY`字符的出现被解释为当前字段值的一部分。

如果字段以`ENCLOSED BY`字符开头，则只有在该字符后跟字段或行`TERMINATED BY`序列时，才会识别该字符的实例作为终止字段值。为避免歧义，在字段值内部的`ENCLOSED BY`字符的出现可以加倍，并且被解释为字符的单个实例。例如，如果指定了`ENCLOSED BY '"'`，则引号将被处理如下：

```sql
"The ""BIG"" boss"  -> The "BIG" boss
The "BIG" boss      -> The "BIG" boss
The ""BIG"" boss    -> The ""BIG"" boss
```

`FIELDS ESCAPED BY`控制如何读取或写入特殊字符：

+   对于输入，如果`FIELDS ESCAPED BY`字符不为空，则会剥离该字符的出现，并且以下字符会被视为字段值的一部分。有一些例外的两字符序列，第一个字符是转义字符。这些序列在下表中显示（使用`\`表示转义字符）。有关`NULL`处理规则的描述稍后在本节中描述。

    | 字符 | 转义序列 |
    | --- | --- |
    | `\0` | ASCII NUL（`X'00'`）字符 |
    | `\b` | 退格字符 |
    | `\n` | 换行（换行）字符 |
    | `\r` | 回车字符 |
    | `\t` | 制表符。 |
    | `\Z` | ASCII 26（Control+Z） |
    | `\N` | 空值 |

    有关`\`转义语法的更多信息，请参见第 11.1.1 节，“字符串文字”。

    如果`FIELDS ESCAPED BY`字符为空，则不会发生转义序列解释。

+   对于输出，如果`FIELDS ESCAPED BY`字符不为空，则用于在输出时前缀以下字符：

    +   `FIELDS ESCAPED BY`字符。

    +   `FIELDS [OPTIONALLY] ENCLOSED BY`字符。

    +   `FIELDS TERMINATED BY`和`LINES TERMINATED BY`值的第一个字符，如果`ENCLOSED BY`字符为空或未指定。

    +   ASCII `0`（实际上在转义字符后写的是 ASCII `0`，而不是零值字节）。

    如果`FIELDS ESCAPED BY`字符为空，则不会转义任何字符，`NULL`会输出为`NULL`，而不是`\N`。如果您的数据中的字段值包含刚才列出的任何字符，可能不是一个好主意指定一个空的转义字符。

在某些情况下，字段和行处理选项会相互作用：

+   如果`LINES TERMINATED BY`是空字符串且`FIELDS TERMINATED BY`不为空，则行也以`FIELDS TERMINATED BY`结束。

+   如果`FIELDS TERMINATED BY`和`FIELDS ENCLOSED BY`值都为空（`''`），则使用固定行（非分隔）格式。使用固定行格式，字段之间不使用分隔符（但仍然可以有行终止符）。相反，列值使用足够宽度的字段宽度读取和写入以容纳字段中的所有值。对于`TINYINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")、`SMALLINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")、`MEDIUMINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")、`INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")和`BIGINT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")，字段宽度分别为 4、6、8、11 和 20，无论声明的显示宽度是多少。

    `LINES TERMINATED BY`仍用于分隔行。如果一行不包含所有字段，则其余列将设置为默认值。如果您没有行终止符，应将其设置为`''`。在这种情况下，文本文件必须为每行包含所有字段。

    固定行格式还会影响`NULL`值的处理，如后面所述。

    注意

    如果使用多字节字符集，则固定大小格式无法工作。

根据使用的`FIELDS`和`LINES`选项，`NULL`值的处理方式会有所不同：

+   对于默认的`FIELDS`和`LINES`值，`NULL`被写为输出的字段值`\N`，并且输入时字段值`\N`被读取为`NULL`（假设`ESCAPED BY`字符为`\`）。

+   如果`FIELDS ENCLOSED BY`不为空，则包含字面单词`NULL`的字段被读取为`NULL`值。这与包含在`FIELDS ENCLOSED BY`字符中的单词`NULL`不同，后者被读取为字符串`'NULL'`。

+   如果`FIELDS ESCAPED BY`为空，则`NULL`会被写为单词`NULL`。

+   使用固定行格式（当`FIELDS TERMINATED BY`和`FIELDS ENCLOSED BY`均为空时使用）时，`NULL`被写为空字符串。这导致表中的`NULL`值和空字符串在写入文件时无法区分，因为两者都被写为空字符串。如果您需要在读取文件时能够区分这两者，请不要使用固定行格式。

尝试将`NULL`加载到`NOT NULL`列中会根据列值分配中描述的规则产生警告或错误。

一些情况不受`LOAD DATA`支持：

+   固定大小行（`FIELDS TERMINATED BY`和`FIELDS ENCLOSED BY`均为空）以及`BLOB`或`TEXT`列。

+   如果您指定一个与另一个相同或前缀相同的分隔符，`LOAD DATA`无法正确解释输入。例如，以下`FIELDS`子句会导致问题：

    ```sql
    FIELDS TERMINATED BY '"' ENCLOSED BY '"'
    ```

+   如果`FIELDS ESCAPED BY`为空，则包含`FIELDS ENCLOSED BY`或`LINES TERMINATED BY`后跟`FIELDS TERMINATED BY`值的字段值会导致`LOAD DATA`过早停止读取字段或行。这是因为`LOAD DATA`无法正确确定字段或行值的结束位置。

#### 列列表规范

以下示例加载`persondata`表的所有列：

```sql
LOAD DATA INFILE 'persondata.txt' INTO TABLE persondata;
```

默认情况下，在`LOAD DATA`语句末尾未提供列列表时，预期输入行包含每个表列的字段。如果您只想加载表的部分列，请指定列列表：

```sql
LOAD DATA INFILE 'persondata.txt' INTO TABLE persondata
(*col_name_or_user_var* [, *col_name_or_user_var*] ...);
```

如果输入文件中字段的顺序与表中列的顺序不同，则还必须指定列列表。否则，MySQL 无法确定如何将输入字段与表列匹配。

#### 输入预处理

`LOAD DATA`语法中的每个*`col_name_or_user_var`*实例都是列名或用户变量。使用用户变量，`SET`子句使您能够在将结果分配给列之前对其值执行预处理转换。

`SET`子句中的用户变量可以以多种方式使用。以下示例直接使用第一个输入列作为`t1.column1`的值，并将第二个输入列分配给一个用户变量，该用户变量在用于`t1.column2`的值之前经历了除法操作：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, @var1)
  SET column2 = @var1/100;
```

`SET`子句可用于提供不是从输入文件派生的值。以下语句将`column3`设置为当前日期和时间：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, column2)
  SET column3 = CURRENT_TIMESTAMP;
```

你也可以通过将其分配给用户变量并不将变量分配给任何表列来丢弃输入值：

```sql
LOAD DATA INFILE 'file.txt'
  INTO TABLE t1
  (column1, @dummy, column2, @dummy, column3);
```

列/变量列表和`SET`子句的使用受以下限制：

+   `SET`子句中的赋值应该只在赋值运算符的左侧具有列名。

+   您可以在`SET`赋值的右侧使用子查询。返回要分配给列的值的子查询只能是标量子查询。此外，您不能使用子查询从正在加载的表中进行选择。

+   被`IGNORE *`number`* LINES`子句忽略的行不会被处理为列/变量列表或`SET`子句。

+   当使用固定行格式加载数据时，不能使用用户变量，因为用户变量没有显示宽度。

#### 列值分配

要处理输入行，`LOAD DATA`将其拆分为字段，并根据列/变量列表和`SET`子句中的值使用这些值，如果它们存在的话。然后将生成的行插入表中。如果表中有`BEFORE INSERT`或`AFTER INSERT`触发器，则在插入行之前或之后分别激活它们。

字段值的解释和分配到表列取决于这些因素：

+   SQL 模式（`sql_mode`系统变量的值）。该模式可以以各种方式是非限制性的，或者是限制性的。例如，可以启用严格的 SQL 模式，或者模式可以包括值，如`NO_ZERO_DATE`或`NO_ZERO_IN_DATE`。

+   `IGNORE`和`LOCAL`修饰符的存在或缺失。

这些因素结合起来产生了由`LOAD DATA`进行的限制性或非限制性数据解释：

+   如果 SQL 模式是限制性的，且未指定 `IGNORE` 或 `LOCAL` 修饰符，则数据解释是限制性的。错误会终止加载操作。

+   如果 SQL 模式是非限制性的，或者指定了 `IGNORE` 或 `LOCAL` 修饰符，则数据解释是非限制性的。（特别是，如果未指定 `REPLACE` 修饰符，则指定任一修饰符 *覆盖* 了限制性 SQL 模式。）错误变为警告，加载操作继续进行。

限制性数据解释使用以下规则：

+   字段过多或过少会导致错误。

+   将`NULL`（即`\N`）赋给非`NULL`列会导致错误。

+   超出列数据类型范围的值会导致错误。

+   无效值产生错误。例如，数值列的值如 `'x'` 会导致错误，而不是转换为 0。

相比之下，非限制性数据解释使用以下规则：

+   如果输入行字段太多，则额外字段将被忽略，并且警告数量会增加。

+   如果输入行字段太少，则为缺少输入字段的列分配它们的默认值。默认值分配在第 13.6 节，“数据类型默认值”中描述。

+   将`NULL`（即`\N`）赋给非`NULL`列会将隐式默认值分配给列数据类型。隐式默认值在第 13.6 节，“数据类型默认值”中描述。

+   无效值产生警告而不是错误，并转换为列数据类型的“最接近”有效值。例如：

    +   对于数值列的值如 `'x'` 会转换为 0。

    +   超出范围的数值或时间值将被剪切到列数据类型范围的最近端点。

    +   对于 `DATETIME`、`DATE` 或 `TIME` 列的无效值将插入为隐式默认值，无论 SQL 模式 `NO_ZERO_DATE` 设置如何。隐式默认值是该类型的适当“零”值（`'0000-00-00 00:00:00'`、`'0000-00-00'` 或 `'00:00:00'`）。请参见第 13.2 节，“日期和时间数据类型”。

+   `LOAD DATA` 对待空字段值与缺少字段不同：

    +   对于字符串类型，该列被设置为空字符串。

    +   对于数值类型，该列被设置为`0`。

    +   对于日期和时间类型，该列被设置为该类型的适当“零”值。请参见第 13.2 节，“日期和时间数据类型”。

    这些是在 `INSERT` 或 `UPDATE` 语句中明确将空字符串分配给字符串、数值、日期或时间类型时的结果。

只有当列中存在`NULL`值（即`\N`）且该列未声明允许`NULL`值，或者`TIMESTAMP` 列的默认值为当前时间戳且在指定字段列表时被省略时，`TIMESTAMP` 列才设置为当前日期和时间。

`LOAD DATA` 将所有输入视为字符串，因此不能像使用`INSERT` 语句那样在`ENUM` 或`SET` 列中使用数值。所有`ENUM` 和`SET` 值必须指定为字符串。

不能直接使用二进制表示法（例如，`b'011010'`）加载`BIT` 值。为解决此问题，请使用`SET`子句去除前导的`b'`和尾随的`'`，并执行二进制到十进制的转换，以便 MySQL 正确将值加载到`BIT` 列中：

```sql
$> cat /tmp/bit_test.txt
b'10'
b'1111111'
$> mysql test
mysql> LOAD DATA INFILE '/tmp/bit_test.txt'
       INTO TABLE bit_test (@var1)
       SET b = CAST(CONV(MID(@var1, 3, LENGTH(@var1)-3), 2, 10) AS UNSIGNED);
Query OK, 2 rows affected (0.00 sec)
Records: 2  Deleted: 0  Skipped: 0  Warnings: 0

mysql> SELECT BIN(b+0) FROM bit_test;
+----------+
| BIN(b+0) |
+----------+
| 10       |
| 1111111  |
+----------+
2 rows in set (0.00 sec)
```

对于以`0b`二进制表示法（例如，`0b011010`）的`BIT` 值，请改用以下`SET`子句以去除前导的`0b`：

```sql
SET b = CAST(CONV(MID(@var1, 3, LENGTH(@var1)-2), 2, 10) AS UNSIGNED)
```

#### 分区表支持

`LOAD DATA` 支持使用`PARTITION`子句显式选择分区，其中包含一个或多个逗号分隔的分区、子分区或两者的名称列表。当使用此子句时，如果文件中的任何行无法插入到列表中命名的任何分区或子分区中，则该语句将失败，并显示错误消息“找到一个不匹配给定分区集的行”。有关更多信息和示例，请参见第 26.5 节，“分区选择”。

#### 并发考虑事项

使用`LOW_PRIORITY`修饰符，`LOAD DATA` 语句的执行将延迟，直到没有其他客户端从表中读取数据。这仅影响仅使用表级锁定的存储引擎（如`MyISAM`、`MEMORY`和`MERGE`）。

使用`CONCURRENT`修饰符和满足并发插入条件的`MyISAM`表（即中间不包含空闲块），其他线程可以在`LOAD DATA` 执行时从表中检索数据。即使没有其他线程同时使用表，此修饰符也会稍微影响`LOAD DATA` 的性能。

#### 语句结果信息

当`LOAD DATA` 语句完成时，将以以下格式返回信息字符串：

```sql
Records: 1  Deleted: 0  Skipped: 0  Warnings: 0
```

警告发生的情况与使用`INSERT`语句插入值时相同（参见 Section 15.2.7, “INSERT Statement”），只是当输入行中字段过少或过多时，`LOAD DATA`也会生成警告。

你可以使用`SHOW WARNINGS`获取前`max_error_count`个警告的列表，以了解出了什么问题。请参见 Section 15.7.7.42, “SHOW WARNINGS Statement”。

如果你正在使用 C API，可以通过调用`mysql_info()`函数获取有关语句的信息。请参见 mysql_info()。

#### 复制注意事项

`LOAD DATA`被认为在基于语句的复制中是不安全的。如果你在`binlog_format=STATEMENT`下使用`LOAD DATA`，每个要应用更改的副本都会创建一个包含数据的临时文件。即使源端启用了二进制日志加密，这个临时文件也不会被加密。如果需要加密，应该使用基于行或混合的二进制日志格式，副本不会创建临时文件。关于`LOAD DATA`和复制之间的交互更多信息，请参见 Section 19.5.1.19, “Replication and LOAD DATA”。

#### 杂项主题

在 Unix 上，如果需要`LOAD DATA`从管道中读取数据，可以使用以下技术（示例将`/`目录的列表加载到表`db1.t1`中）：

```sql
mkfifo /mysql/data/db1/ls.dat
chmod 666 /mysql/data/db1/ls.dat
find / -ls > /mysql/data/db1/ls.dat &
mysql -e "LOAD DATA INFILE 'ls.dat' INTO TABLE t1" db1
```

在这里，你必须在生成要加载的数据的命令和**mysql**命令上分别运行在不同的终端，或者在后台运行数据生成过程（如前面的示例所示）。如果不这样做，管道会一直阻塞，直到**mysql**进程读取数据。
