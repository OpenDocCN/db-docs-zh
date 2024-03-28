# 12.2.2 UTF-8 for Metadata

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-metadata.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-metadata.html)

元数据是“关于数据的数据”。任何*描述*数据库的东西——而不是数据库的*内容*——都是元数据。因此，列名、数据库名、用户名称、版本名称以及`SHOW`的大部分字符串结果都是元数据。这也适用于`INFORMATION_SCHEMA`中表的内容，因为这些表定义上包含有关数据库对象的信息。

元数据的表示必须满足这些要求：

+   所有元数据必须使用相同的字符集。否则，`SHOW`语句或`INFORMATION_SCHEMA`中表的`SELECT`语句将无法正常工作，因为这些操作的结果中同一列中的不同行将使用不同的字符集。

+   元数据必须包含所有语言中的所有字符。否则，用户将无法使用自己的语言命名列和表。

为了满足这两个要求，MySQL 将元数据存储在 Unicode 字符集中，即 UTF-8。如果您从不使用带重音或非拉丁字符，这不会造成任何干扰。但如果您使用了，您应该知道元数据是 UTF-8 的。

元数据要求意味着`USER()`、`CURRENT_USER()`、`SESSION_USER()`、`SYSTEM_USER()`、`DATABASE()`和`VERSION()`函数的返回值默认为 UTF-8 字符集。

服务器将`character_set_system`系统变量设置为元数据字符集的名称：

```sql
mysql> SHOW VARIABLES LIKE 'character_set_system';
+----------------------+---------+
| Variable_name        | Value   |
+----------------------+---------+
| character_set_system | utf8mb3 |
+----------------------+---------+
```

使用 Unicode 存储元数据*不*意味着服务器默认以`character_set_system`字符集返回列标题和`DESCRIBE`函数的结果。当您使用`SELECT column1 FROM t`时，服务器将`column1`名称本身以由`character_set_results`系统变量的值确定的字符集返回给客户端，该系统变量的默认值为`utf8mb4`。如果您希望服务器以不同的字符集传递元数据结果，请使用`SET NAMES`语句强制服务器执行字符集转换。`SET NAMES`设置`character_set_results`和其他相关系统变量。（请参阅第 12.4 节，“连接字符集和校对规则”。）或者，客户端程序可以在从服务器接收结果后执行转换。客户端执行转换更有效，但并非所有客户端都能使用此选项。

如果`character_set_results`设置为`NULL`，则不执行任何转换，服务器将使用其原始字符集返回元数据（由`character_set_system`指示的字符集）。

从服务器返回给客户端的错误消息会自动转换为客户端字符集，与元数据一样。

如果您在单个语句中使用（例如）`USER()`函数进行比较或赋值，不用担心。MySQL 会为您执行一些自动转换。

```sql
SELECT * FROM t1 WHERE USER() = latin1_column;
```

这是因为在比较之前，`latin1_column`的内容会自动转换为 UTF-8。

```sql
INSERT INTO t1 (latin1_column) SELECT USER();
```

这是因为在赋值之前，`USER()`的内容会自动转换为`latin1`。

尽管自动转换不符合 SQL 标准，但标准确实指出每个字符集（在支持的字符方面）都是 Unicode 的“子集”。因为“适用于超集的内容也适用于子集”是一个众所周知的原则，我们认为 Unicode 的排序规则可以用于与非 Unicode 字符串的比较。有关字符串强制转换的更多信息，请参阅第 12.8.4 节，“表达式中的排序规则可强制性”。
