> 原文：[`dev.mysql.com/doc/refman/8.0/en/select-into.html`](https://dev.mysql.com/doc/refman/8.0/en/select-into.html)

#### 15.2.13.1 SELECT ... INTO Statement

`SELECT ... INTO`形式的`SELECT`允许将查询结果存储在变量中或写入文件：

+   `SELECT ... INTO *`var_list`*`选择列值并将其存储到变量中。

+   `SELECT ... INTO OUTFILE`将所选行写入文件。可以指定列和行终止符以生成特定的输出格式。

+   `SELECT ... INTO DUMPFILE`将一行数据写入文件，不进行任何格式化。

给定的`SELECT`语句最多可以包含一个`INTO`子句，尽管如`SELECT`语法描述所示（参见第 15.2.13 节，“SELECT Statement”），`INTO`可以出现在不同的位置：

+   在`FROM`之前。示例：

    ```sql
    SELECT * INTO @myvar FROM t1;
    ```

+   在尾随锁定子句之前。示例：

    ```sql
    SELECT * FROM t1 INTO @myvar FOR UPDATE;
    ```

+   在`SELECT`的末尾。示例：

    ```sql
    SELECT * FROM t1 FOR UPDATE INTO @myvar;
    ```

在 MySQL 8.0.20 中支持语句末尾的`INTO`位置，并且是首选位置。在 MySQL 8.0.20 中，位于锁定子句之前的位置已被弃用；预计在未来的 MySQL 版本中将删除对其的支持。换句话说，`INTO`在`FROM`之后但不在`SELECT`的末尾会产生警告。

不应在嵌套的`SELECT`中使用`INTO`子句，因为这样的`SELECT`必须将其结果返回给外部上下文。在`UNION`语句中对`INTO`的使用也受到约束；请参见第 15.2.18 节，“UNION Clause”。

对于`INTO *`var_list`*`变体：

+   *`var_list`*命名一个或多个变量的列表，每个变量可以是用户定义的变量、存储过程或函数参数，或存储程序本地变量。（在准备的`SELECT ... INTO *`var_list`*`语句中，只允许使用用户定义的变量；请参见第 15.6.4.2 节，“本地变量范围和解析”。）

+   所选值被分配给变量。变量的数量必须与列的数量匹配。查询应返回一行数据。如果查询未返回任何行，则会出现带有错误代码 1329 的警告（`No data`），并且变量值保持不变。如果查询返回多行数据，则会出现错误 1172（`Result consisted of more than one row`）。如果可能语句可能检索多行数据，可以使用`LIMIT 1`将结果集限制为一行。

    ```sql
    SELECT id, data INTO @x, @y FROM test.t1 LIMIT 1;
    ```

`INTO *`var_list`*`也可以与`TABLE`语句一起使用，但受到以下限制：

+   变量的数量必须与表中的列数相匹配。

+   如果表包含多行，则必须使用`LIMIT 1`将结果集限制为单行。`LIMIT 1`必须在`INTO`关键字之前。

这里显示了这种语句的一个示例：

```sql
TABLE employees ORDER BY lname DESC LIMIT 1
    INTO @id, @fname, @lname, @hired, @separated, @job_code, @store_id;
```

您还可以从生成单行的`VALUES`语句中选择值到一组用户变量中。在这种情况下，您必须使用表别名，并且必须将值列表中的每个值分配给一个变量。这里显示的两个语句中的每一个都等同于`SET @x=2, @y=4, @z=8`：

```sql
SELECT * FROM (VALUES ROW(2,4,8)) AS t INTO @x,@y,@z;

SELECT * FROM (VALUES ROW(2,4,8)) AS t(a,b,c) INTO @x,@y,@z;
```

用户变量名称不区分大小写。请参阅第 11.4 节，“用户定义变量”。

`SELECT ... INTO OUTFILE '*`file_name`*'`形式的`SELECT`将所选行写入文件。文件在服务器主机上创建，因此您必须具有`FILE`权限才能使用此语法。*`file_name`*不能是现有文件，这样可以防止修改文件，例如`/etc/passwd`和数据库表。`character_set_filesystem`系统变量控制文件名的解释。

`SELECT ... INTO OUTFILE`语句旨在使在服务器主机上将表转储到文本文件成为可能。要在其他主机上创建结果文件，通常使用`SELECT ... INTO OUTFILE`是不合适的，因为无法相对于服务器主机文件系统写入文件路径，除非可以使用服务器主机文件系统上的网络映射路径访问远程主机上文件的位置。

或者，如果远程主机上安装了 MySQL 客户端软件，您可以使用客户端命令，例如`mysql -e "SELECT ..." > *`file_name`*`在该主机上生成文件。

`SELECT ... INTO OUTFILE`是`LOAD DATA`的补充。列值被写入并转换为`CHARACTER SET`子句中指定的字符集。如果没有这样的子句，值将使用`binary`字符集进行转储。实际上，没有字符集转换。如果结果集包含几种字符集的列，则输出数据文件也是如此，可能无法正确重新加载文件。

语句中 *`export_options`* 部分的语法由与 `LOAD DATA` 语句一起使用的相同 `FIELDS` 和 `LINES` 子句组成。有关 `FIELDS` 和 `LINES` 子句的信息，包括它们的默认值和允许的值，请参阅 Section 15.2.9, “LOAD DATA Statement”。

`FIELDS ESCAPED BY` 控制如何写入特殊字符。如果 `FIELDS ESCAPED BY` 字符不为空，则在必要时用作前缀，避免输出时后续字符的歧义：

+   `FIELDS ESCAPED BY` 字符

+   `FIELDS [OPTIONALLY] ENCLOSED BY` 字符

+   `FIELDS TERMINATED BY` 和 `LINES TERMINATED BY` 值的第一个字符

+   ASCII `NUL`（零值字节；实际写入转义字符后的内容是 ASCII `0`，而不是零值字节）

`FIELDS TERMINATED BY`、`ENCLOSED BY`、`ESCAPED BY` 或 `LINES TERMINATED BY` 字符 *必须* 转义，以便您可以可靠地读取文件。ASCII `NUL` 被转义，以便在某些分页器中更容易查看。

生成的文件不需要符合 SQL 语法，因此不需要转义其他内容。

如果 `FIELDS ESCAPED BY` 字符为空，则不会转义任何字符，`NULL` 输出为 `NULL`，而不是 `\N`。如果你的数据中的字段值包含刚才列出的字符之一，可能不是一个好主意指定一个空的转义字符。

当您想要将表的所有列转储到文本文件时，也可以使用 `INTO OUTFILE` 与 `TABLE` 语句。在这种情况下，可以使用 `ORDER BY` 和 `LIMIT` 控制排序和行数；这些子句必须在 `INTO OUTFILE` 之前。`TABLE ... INTO OUTFILE` 支持与 `SELECT ... INTO OUTFILE` 相同的 *`export_options`*，并且受到写入文件系统的相同限制。这里展示了这种语句的一个示例：

```sql
TABLE employees ORDER BY lname LIMIT 1000
    INTO OUTFILE '/tmp/employee_data_1.txt'
    FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"', ESCAPED BY '\'
    LINES TERMINATED BY '\n';
```

你也可以使用 `SELECT ... INTO OUTFILE` 与 `VALUES` 语句将值直接写入文件。这里有一个示例：

```sql
SELECT * FROM (VALUES ROW(1,2,3),ROW(4,5,6),ROW(7,8,9)) AS t
    INTO OUTFILE '/tmp/select-values.txt';
```

您必须使用表别名；列别名也受支持，并且可以选择性地用于仅从所需列中写入值。您还可以使用 `SELECT ... INTO OUTFILE` 支持的任何或所有导出选项来将输出格式化到文件中。

这里有一个示例，生成一个以逗号分隔值（CSV）格式的文件，许多程序使用这种格式：

```sql
SELECT a,b,a+b INTO OUTFILE '/tmp/result.txt'
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  FROM test_table;
```

如果使用 `INTO DUMPFILE` 而不是 `INTO OUTFILE`，MySQL 只会将一行写入文件，不会有任何列或行终止，并且不执行任何转义处理。这对于选择 `BLOB` 值并将其存储在文件中很有用。

`TABLE`也支持`INTO DUMPFILE`。 如果表包含多行，则还必须使用`LIMIT 1`将输出限制为单行。 `INTO DUMPFILE`也可以与`SELECT * FROM (VALUES ROW()[, ...]) AS *`table_alias`* [LIMIT 1]`一起使用。请参阅 Section 15.2.19，“VALUES Statement”。

注意

由`INTO OUTFILE`或`INTO DUMPFILE`创建的任何文件都归属于运行**mysqld**的操作系统用户。 （出于这个和其他原因，您*绝对不应该*以`root`身份运行**mysqld**。）从 MySQL 8.0.17 开始，文件创建的 umask 为 0640；您必须具有足够的访问权限来操作文件内容。在 MySQL 8.0.17 之前，umask 为 0666，文件可被服务器主机上的所有用户写入。

如果`secure_file_priv`系统变量设置为非空目录名称，则要写入的文件必须位于该目录中。

在由事件调度程序执行的事件的一部分中发生的`SELECT ... INTO`语句的上下文中，诊断消息（不仅是错误，还包括警告）将被写入错误日志，并且在 Windows 上，将被写入应用程序事件日志。有关更多信息，请参阅 Section 27.4.5，“事件调度程序状态”。

从 MySQL 8.0.22 开始，支持定期同步由`SELECT INTO OUTFILE`和`SELECT INTO DUMPFILE`写入的输出文件，通过设置在该版本中引入的`select_into_disk_sync`服务器系统变量来启用。可以使用`select_into_buffer_size`和`select_into_disk_sync_delay`分别设置输出缓冲区大小和可选延迟。有关更多信息，请参阅这些系统变量的描述。
