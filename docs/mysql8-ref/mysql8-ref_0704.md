# 12.9.8 在 3 字节和 4 字节 Unicode 字符集之间转换

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-unicode-conversion.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-unicode-conversion.html)

本节描述了在`utf8mb3`和`utf8mb4`字符集之间转换字符数据时可能遇到的问题。

注意

本讨论主要集中在`utf8mb3`和`utf8mb4`之间的转换，但类似的原则也适用于`ucs2`字符集与`utf16`或`utf32`等字符集之间的转换。

`utf8mb3`和`utf8mb4`字符集的区别如下：

+   `utf8mb3`仅支持基本多文种平面（BMP）中的字符。`utf8mb4`还支持位于 BMP 之外的补充字符。

+   `utf8mb3`每个字符最多使用三个字节。`utf8mb4`每个字符最多使用四个字节。

注意

本讨论提到`utf8mb3`和`utf8mb4`字符集名称，明确指代 3 字节和 4 字节 UTF-8 字符集数据。

从`utf8mb3`转换为`utf8mb4`的一个优势是，这使应用程序能够使用补充字符。一个权衡是这可能增加数据存储空间的需求。

在表内容方面，从`utf8mb3`转换为`utf8mb4`没有问题：

+   对于 BMP 字符，`utf8mb4`和`utf8mb3`具有相同的存储特性：相同的代码值，相同的编码，相同的长度。

+   对于补充字符，`utf8mb4`需要四个字节来存储它，而`utf8mb3`根本无法存储该字符。当将`utf8mb3`列转换为`utf8mb4`时，您不必担心转换补充字符，因为根本没有。

在表结构方面，这些是主要的潜在不兼容性：

+   对于可变长度字符数据类型（`VARCHAR` 和 `TEXT` 类型），在`utf8mb4`列中允许的最大字符长度比在`utf8mb3`列中要少。

+   对于所有字符数据类型（`CHAR`、`VARCHAR` 和 `TEXT` 类型），在`utf8mb4`列中可以索引的最大字符数比在`utf8mb3`列中要少。

因此，要将表从`utf8mb3`转换为`utf8mb4`，可能需要更改一些列或索引定义。

可以使用`ALTER TABLE`将表从`utf8mb3`转换为`utf8mb4`。假设一个表具有这样的定义：

```sql
CREATE TABLE t1 (
  col1 CHAR(10) CHARACTER SET utf8mb3 COLLATE utf8mb3_unicode_ci NOT NULL,
  col2 CHAR(10) CHARACTER SET utf8mb3 COLLATE utf8mb3_bin NOT NULL
) CHARACTER SET utf8mb3;
```

以下语句将`t1`转换为使用`utf8mb4`：

```sql
ALTER TABLE t1
  DEFAULT CHARACTER SET utf8mb4,
  MODIFY col1 CHAR(10)
    CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL,
  MODIFY col2 CHAR(10)
    CHARACTER SET utf8mb4 COLLATE utf8mb4_bin NOT NULL;
```

从`utf8mb3`转换为`utf8mb4`的关键在于列或索引键的最大长度在*字节*方面保持不变。因此，在*字符*方面较小，因为字符的最大长度为四个字节而不是三个字节。对于`CHAR`、`VARCHAR`和`TEXT`数据类型，在转换 MySQL 表时，请注意以下问题：

+   检查所有`utf8mb3`列的定义，并确保其不超过存储引擎的最大长度。

+   检查所有`utf8mb3`列上的索引，并确保其不超过存储引擎的最大长度。有时，由于存储引擎的增强，最大长度可能会发生变化。

如果满足上述条件，则必须减少列或索引的定义长度，或者继续使用`utf8mb3`而不是`utf8mb4`。

以下是可能需要进行结构更改的一些示例：

+   一个`TINYTEXT`列最多可以容纳 255 个字节，因此可以容纳 85 个 3 字节或 63 个 4 字节字符。假设您有一个使用`utf8mb3`但必须能够包含超过 63 个字符的`TINYTEXT`列。除非还将数据类型更改为更长的类型，如`TEXT`，否则无法将其转换为`utf8mb4`。

    同样，如果要将非常长的`VARCHAR`列从`utf8mb3`转换为`utf8mb4`，可能需要将其更改为更长的`TEXT`类型之一。

+   对于使用`COMPACT`或`REDUNDANT`行格式的表，`InnoDB`对于`utf8mb3`或`utf8mb4`列的最大索引长度为 767 个字节，因此对于分别最多可以索引 255 或 191 个字符的`utf8mb3`或`utf8mb4`列。如果您当前具有超过 191 个字符的`utf8mb3`列索引，则必须索引较少数量的字符。

    在使用`COMPACT`或`REDUNDANT`行格式的`InnoDB`表中，这些列和索引定义是合法的：

    ```sql
    col1 VARCHAR(500) CHARACTER SET utf8mb3, INDEX (col1(255))
    ```

    要改用`utf8mb4`，索引必须更小：

    ```sql
    col1 VARCHAR(500) CHARACTER SET utf8mb4, INDEX (col1(191))
    ```

    注意

    对于使用`COMPRESSED`或`DYNAMIC`行格式的`InnoDB`表，允许索引键前缀的长度超过 767 字节（最多 3072 字节）。使用这些行格式创建的表允许您为`utf8mb3`或`utf8mb4`列最多索引 1024 或 768 个字符。有关相关信息，请参见第 17.22 节，“InnoDB 限制”和 DYNAMIC 行格式。

前述类型的更改只有在您有非常长的列或索引时才可能需要。否则，您应该能够在不出现问题的情况下将表从`utf8mb3`转换为`utf8mb4`，如前所述使用`ALTER TABLE`。

以下项目总结了其他潜在的不兼容性：

+   `SET NAMES 'utf8mb4'`导致连接字符集使用 4 字节字符集。只要服务器不发送 4 字节字符，就不应该有问题。否则，期望每个字符接收最多三个字节的应用程序可能会出现问题。相反，期望发送 4 字节字符的应用程序必须确保服务器理解它们。

+   对于复制，如果要在源上使用支持补充字符的字符集，则所有副本必须也理解它们。

    此外，请记住一个一般原则，即如果表在源和副本上有不同的定义，这可能导致意外结果。例如，在源上使用`utf8mb3`和在副本上使用`utf8mb4`会导致最大索引键长度的差异，这样做是有风险的。

如果您已经转换为`utf8mb4`、`utf16`、`utf16le`或`utf32`，然后决定转换回`utf8mb3`或`ucs2`（例如，降级到 MySQL 的旧版本），则应考虑以下事项：

+   `utf8mb3`和`ucs2`数据应该没有问题。

+   服务器必须足够新，以识别引用正在转换的字符集的定义。

+   对于引用`utf8mb4`字符集的对象定义，您可以在降级之前使用**mysqldump**进行转储，编辑转储文件以将`utf8mb4`实例更改为`utf8`，然后在旧服务器中重新加载文件，只要数据中没有 4 字节字符。旧服务器在转储文件对象定义中看到`utf8`，并创建使用（3 字节）`utf8`字符集的新对象。
