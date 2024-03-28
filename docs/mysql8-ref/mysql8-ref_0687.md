# 12.7 列字符集转换

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-conversion.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-conversion.html)

要将二进制或非二进制字符串列转换为使用特定字符集，请使用`ALTER TABLE`。要成功转换，必须满足以下条件之一：

+   如果列具有二进制数据类型（`BINARY`, `VARBINARY`, `BLOB`)，则它包含的所有值必须使用单个字符集进行编码（您正在将列转换为的字符集）。如果使用二进制列存储多个字符集的信息，MySQL 无法知道哪些值使用哪个字符集，并且无法正确转换数据。

+   如果列具有非二进制数据类型（`CHAR`, `VARCHAR`, `TEXT`)，其内容应该以列字符集编码，而不是其他字符集。如果内容以不同的字符集编码，您可以先将列转换为使用二进制数据类型，然后再转换为具有所需字符集的非二进制列。

假设表`t`有一个名为`col1`的二进制列，定义为`VARBINARY(50)`。假设列中的信息使用单个字符集进行编码，您可以将其转换为具有该字符集的非二进制列。例如，如果`col1`包含代表`greek`字符集中字符的二进制数据，则可以按如下方式进行转换：

```sql
ALTER TABLE t MODIFY col1 VARCHAR(50) CHARACTER SET greek;
```

如果您的原始列类型为`BINARY(50)`，您可以将其转换为`CHAR(50)`，但结果值将在末尾填充`0x00`字节，这可能是不希望的。要删除这些字节，请使用`TRIM()`函数：

```sql
UPDATE t SET col1 = TRIM(TRAILING 0x00 FROM col1);
```

假设表`t`有一个名为`col1`的非二进制列，定义为`CHAR(50) CHARACTER SET latin1`，但您希望将其转换为使用`utf8mb4`，以便存储来自许多语言的值。以下语句可以实现这一目标：

```sql
ALTER TABLE t MODIFY col1 CHAR(50) CHARACTER SET utf8mb4;
```

如果列包含不在两个字符集中的字符，则转换可能会有损失。

如果您有 MySQL 4.1 之前的旧表，其中一个非二进制列包含实际上是使用与服务器默认字符集不同的字符集编码的值，则会出现特殊情况。例如，一个应用程序可能已经在列中存储了`sjis`值，尽管 MySQL 的默认字符集不同。可以将列转换为使用正确的字符集，但需要额外的步骤。假设服务器的默认字符集是`latin1`，`col1`被定义为`CHAR(50)`，但其内容是`sjis`值。第一步是将列转换为二进制数据类型，这将删除现有的字符集信息而不执行任何字符转换：

```sql
ALTER TABLE t MODIFY col1 BLOB;
```

下一步是将列转换为具有正确字符集的非二进制数据类型：

```sql
ALTER TABLE t MODIFY col1 CHAR(50) CHARACTER SET sjis;
```

在将 MySQL 升级到 4.1 或更高版本后，此过程要求表在使用`INSERT`或`UPDATE`等语句修改之前未被修改。在这种情况下，MySQL 将使用`latin1`存储新值在列中，并且该列将包含`sjis`和`latin1`值的混合，无法正确转换。

如果在创建列时指定了属性，那么在使用`ALTER TABLE`修改表时也应该指定这些属性。例如，如果您指定了`NOT NULL`和一个明确的`DEFAULT`值，那么在`ALTER TABLE`语句中也应该提供它们。否则，生成的列定义将不包括这些属性。

要转换表中的所有字符列，`ALTER TABLE ... CONVERT TO CHARACTER SET *`charset`*`语句可能会有用。请参阅第 15.1.9 节，“ALTER TABLE Statement”。
