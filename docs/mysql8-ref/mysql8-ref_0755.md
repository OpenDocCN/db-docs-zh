# 13.3.4 BLOB 和 TEXT 类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/blob.html`](https://dev.mysql.com/doc/refman/8.0/en/blob.html)

`BLOB`是可以容纳可变数量数据的二进制大对象。四种`BLOB`类型分别是`TINYBLOB`、`BLOB`、`MEDIUMBLOB`和`LONGBLOB`。它们仅在可以容纳的值的最大长度上有所不同。四种`TEXT`类型是`TINYTEXT`、`TEXT`、`MEDIUMTEXT`和`LONGTEXT`。这些与四种`BLOB`类型对应，并具有相同的最大长度和存储要求。参见第 13.7 节，“数据类型存储要求”。

`BLOB`值被视为二进制字符串（字节字符串）。它们具有`binary`字符集和排序规则，比较和排序基于列值中字节的数值。`TEXT`值被视为非二进制字符串（字符字符串）。它们具有除`binary`之外的字符集，并且根据字符集的排序规则进行排序和比较。

如果未启用严格的 SQL 模式并且将值分配给超出列的最大长度的`BLOB`或`TEXT`列，则该值将被截断以适应并生成警告。对于非空格字符的截断，您可以通过使用严格的 SQL 模式导致错误发生（而不是警告）并抑制值的插入。参见第 7.1.11 节，“服务器 SQL 模式”。

要插入`TEXT`列的值中多余的尾随空格截断总是生成警告，无论 SQL 模式如何。

对于`TEXT`和`BLOB`列，在插入时不会填充，选择时也不会去除任何字节。

如果对`TEXT`列进行索引，索引条目比较在末尾填充空格。这意味着，如果索引需要唯一值，则对于仅在尾随空格数量上不同的值会导致重复键错误。例如，如果表包含`'a'`，则尝试存储`'a '`会导致重复键错误。对于`BLOB`列则不是这样。

在大多数方面，您可以将`BLOB`列视为可以任意大的`VARBINARY`列。同样，您可以将`TEXT`列视为`VARCHAR`列。`BLOB`和`TEXT`与`VARBINARY`和`VARCHAR`在以下方面有所不同：

+   对于`BLOB`和`TEXT`列上的索引，必须指定索引前缀长度。对于`CHAR`和`VARCHAR`，前缀长度是可选的。参见第 10.3.5 节，“列索引”。

+   `BLOB`和`TEXT`列不能有`DEFAULT`值。

如果在`TEXT`数据类型中使用`BINARY`属性，则该列将被分配为列字符集的二进制(`_bin`)排序规则。

`LONG`和`LONG VARCHAR`映射到`MEDIUMTEXT`数据类型。这是一个兼容性特性。

MySQL Connector/ODBC 将`BLOB`值定义为`LONGVARBINARY`，将`TEXT`值定义为`LONGVARCHAR`。

因为`BLOB`和`TEXT`值可能非常长，所以在使用它们时可能会遇到一些限制：

+   在排序时只使用列的前`max_sort_length`字节。`max_sort_length`的默认值为 1024。您可以通过在服务器启动或运行时增加`max_sort_length`的值来使更多字节在排序或分组中起作用。任何客户端都可以更改其会话`max_sort_length`变量的值：

    ```sql
    mysql> SET max_sort_length = 2000;
    mysql> SELECT id, comment FROM t
     -> ORDER BY comment;
    ```

+   在使用临时表处理的查询结果中存在`BLOB`或`TEXT`列的实例会导致服务器在磁盘上而不是内存中使用表，因为`MEMORY`存储引擎不支持这些数据类型（参见第 10.4.4 节，“MySQL 中的内部临时表使用”）。使用磁盘会导致性能损失，因此只有在真正需要时才在查询结果中包含`BLOB`或`TEXT`列。例如，避免使用`SELECT *`，它会选择所有列。

+   `BLOB`或`TEXT`对象的最大大小由其类型确定，但实际上您可以在客户端和服务器之间传输的最大值取决于可用内存量和通信缓冲区的大小。您可以通过更改`max_allowed_packet`变量的值来更改消息缓冲区大小，但必须同时为服务器和客户端程序执行此操作。例如，**mysql**和**mysqldump**都允许您更改客户端端的`max_allowed_packet`值。请参阅第 7.1.1 节，“配置服务器”，第 6.5.1 节，“mysql — MySQL 命令行客户端”和第 6.5.4 节，“mysqldump — 数据库备份程序”。您可能还想比较数据包大小和您存储的数据对象的大小与存储要求的大小，参见第 13.7 节，“数据类型存储要求”

每个`BLOB`或`TEXT`值在内部由单独分配的对象表示。这与所有其他数据类型形成对比，其他数据类型在打开表时为每列分配存储空间。

在某些情况下，将二进制数据（如媒体文件）存储在`BLOB`或`TEXT`列中可能是可取的。您可能会发现 MySQL 的字符串处理函数在处理此类数据时非常有用。请参阅第 14.8 节，“字符串函数和运算符”。出于安全和其他原因，通常最好使用应用程序代码来处理此类数据，而不是给予应用程序用户`FILE`权限。您可以在 MySQL 论坛（[`forums.mysql.com/`](http://forums.mysql.com/)）讨论各种语言和平台的具体情况。

注意

在**mysql**客户端中，二进制字符串以十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参阅第 6.5.1 节，“mysql — MySQL 命令行客户端”。
