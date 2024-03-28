# 12.4 连接字符集和排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-connection.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-connection.html)

“连接”是客户端程序连接到服务器时建立的会话，通过该会话与服务器进行交互。客户端通过会话连接发送 SQL 语句，如查询。服务器通过连接向客户端发送响应，如结果集或错误消息。

+   连接字符集和排序规则系统变量

+   不允许的客户端字符集

+   客户端程序连接字符集配置

+   连接字符集配置的 SQL 语句

+   连接字符集错误处理

### 连接字符集和排序规则系统变量

几个字符集和排序规则系统变量与客户端与服务器的交互相关。其中一些在前面的章节中已经提到：

+   `character_set_server` 和 `collation_server` 系统变量指示服务器的字符集和排序规则。参见 第 12.3.2 节，“服务器字符集和排序规则”。

+   `character_set_database` 和 `collation_database` 系统变量指示默认数据库的字符集和排序规则。参见 第 12.3.3 节，“数据库字符集和排序规则”。

处理客户端与服务器之间连接流量涉及到其他字符集和排序规则系统变量。每个客户端都有特定于会话的连接相关字符集和排序规则系统变量。这些会话系统变量值在连接时初始化，但可以在会话中更改。

关于客户端连接的字符集和排序规则处理的几个问题可以通过系统变量来回答：

+   当语句离开客户端时，它们处于什么字符集？

    服务器将`character_set_client`系统变量视为客户端发送的语句的字符集。

    注意

    一些字符集不能用作客户端字符集。请参阅不允许的客户端字符集。

+   服务器在接收语句后应将其转换为哪种字符集？

    为了确定这一点，服务器使用`character_set_connection`和`collation_connection`系统变量：

    +   服务器将客户端发送的语句从`character_set_client`转换为`character_set_connection`。例外：对于具有诸如`_utf8mb4`或`_latin2`之类的引导符的字符串文字，引导符确定字符集。请参阅第 12.3.8 节，“字符集引导符”。

    +   `collation_connection`对于文字字符串的比较很重要。对于与列值进行比较的字符串，`collation_connection`并不重要，因为列有自己的排序规则，其具有更高的排序规则优先级（参见第 12.8.4 节，“表达式中的排序规则可强制性”）。

+   服务器在将查询结果返回给客户端之前应将其转换为哪种字符集？

    `character_set_results`系统变量指示服务器将查询结果返回给客户端的字符集。这包括结果数据，如列值，结果元数据，如列名，以及错误消息。

    要告诉服务器不要对结果集或错误消息进行任何转换，请将`character_set_results`设置为`NULL`或`binary`：

    ```sql
    SET character_set_results = NULL;
    SET character_set_results = binary;
    ```

    有关字符集和错误消息的更多信息，请参阅第 12.6 节，“错误消息字符集”。

要查看适用于当前会话的字符集和排序规则系统变量的值，请使用此语句：

```sql
SELECT * FROM performance_schema.session_variables
WHERE VARIABLE_NAME IN (
  'character_set_client', 'character_set_connection',
  'character_set_results', 'collation_connection'
) ORDER BY VARIABLE_NAME;
```

以下更简单的语句也显示连接变量，但同时包括其他相关变量。它们可以用于查看*所有*字符集和排序规则系统变量：

```sql
SHOW SESSION VARIABLES LIKE 'character\_set\_%';
SHOW SESSION VARIABLES LIKE 'collation\_%';
```

客户端可以微调这些变量的设置，或依赖默认值（在这种情况下，您可以跳过本节的其余部分）。 如果不使用默认值，则必须为*每个连接到服务器的连接*更改字符设置。

### 不允许的客户端字符集

`character_set_client` 系统变量不能设置为某些字符集：

```sql
ucs2
utf16
utf16le
utf32
```

尝试将任何这些字符集用作客户端字符集会产生错误：

```sql
mysql> SET character_set_client = 'ucs2';
ERROR 1231 (42000): Variable 'character_set_client'
can't be set to the value of 'ucs2'
```

如果在以下情况下使用任何这些字符集，都会发生相同的错误，所有这些情况都会尝试将`character_set_client`设置为指定的字符集：

+   MySQL 客户端程序（如**mysql**和**mysqladmin**）使用的`--default-character-set=*`charset_name`*` 命令选项。

+   `SET NAMES '*`charset_name`*'` 语句。

+   `SET CHARACTER SET '*`charset_name`*'` 语句。

### 客户端程序连接字符集配置

当客户端连接到服务器时，它指示要用于与服务器通信的字符集。 （实际上，客户端指示该字符集的默认排序规则，从中服务器可以确定字符集。）服务器使用此信息将`character_set_client`、`character_set_results`、`character_set_connection` 系统变量设置为字符集，并将`collation_connection` 设置为字符集的默认排序规则。 实际上，服务器执行了一个等效的`SET NAMES` 操作。

如果服务器不支持请求的字符集或排序规则，它将回退到使用服务器字符集和排序规则来配置连接。有关此回退行为的更多详细信息，请参阅连接字符集错误处理。

**mysql**, **mysqladmin**, **mysqlcheck**, **mysqlimport**, 和 **mysqlshow** 客户端程序确定要使用的默认字符集如下：

+   在没有其他信息的情况下，每个客户端使用编译时的默认字符集，通常为`utf8mb4`。

+   每个客户端可以根据操作系统设置自动检测要使用的字符集，例如在 Unix 系统上的`LANG`或`LC_ALL`区域设置环境变量的值，或 Windows 系统上的代码页设置。对于从操作系统获取区域设置的系统，客户端使用它来设置默认字符集，而不是使用编译时的默认值。例如，将`LANG`设置为`ru_RU.KOI8-R`会导致使用`koi8r`字符集。因此，用户可以配置其环境中的区域设置供 MySQL 客户端使用。

    如果没有完全匹配，操作系统字符集将映射到最接近的 MySQL 字符集。如果客户端不支持匹配的字符集，则使用编译时的默认值。例如，`utf8`和`utf-8`映射到`utf8mb4`，而`ucs2`不支持作为连接字符集，因此映射到编译时的默认值。

    C 应用程序可以根据操作系统设置使用字符集自动检测，方法是在连接到服务器之前调用`mysql_options()`如下：

    ```sql
    mysql_options(mysql,
                  MYSQL_SET_CHARSET_NAME,
                  MYSQL_AUTODETECT_CHARSET_NAME);
    ```

+   每个客户端支持`--default-character-set`选项，允许用户显式指定字符集以覆盖客户端否则确定的默认值。

    注意

    有些字符集不能用作客户端字符集。尝试使用它们与`--default-character-set`会产生错误。参见不允许的客户端字符集。

使用**mysql**客户端，如果要使用与默认字符集不同的字符集，您可以在每次连接到服务器时显式执行一个`SET NAMES`语句（参见客户端程序连接字符集配置）。为了更轻松地实现相同的结果，可以在选项文件中指定字符集。例如，以下选项文件设置在每次调用**mysql**时将三个与连接相关的字符集系统变量设置为`koi8r`：

```sql
[mysql]
default-character-set=koi8r
```

如果您正在使用启用了自动重新连接的**mysql**客户端（不建议），最好使用`charset`命令而不是`SET NAMES`。例如：

```sql
mysql> charset koi8r
Charset changed
```

`charset`命令发出一个`SET NAMES`语句，并在连接断开后重新连接时更改**mysql**使用的默认字符集。

在配置客户端程序时，还必须考虑它们执行的环境。参见第 12.5 节，“配置应用程序字符集和排序规则”。

### 用于连接字符集配置的 SQL 语句

建立连接后，客户端可以为当前会话更改字符集和排序规则系统变量。这些变量可以使用`SET`语句单独更改，但另外两个更方便的语句会作为一组影响与连接相关的字符集系统变量：

+   `SET NAMES '*`charset_name`*' [COLLATE '*`collation_name`*']`

    `SET NAMES`指示客户端用于向服务器发送 SQL 语句的字符集。因此，`SET NAMES 'cp1251'`告诉服务器，“来自此客户端的未来传入消息使用字符集`cp1251`。”它还指定服务器在将结果发送回客户端时应使用的字符集。（例如，如果使用产生结果集的`SELECT`语句，则指定用于列值的字符集。）

    一个`SET NAMES '*`charset_name`*'`语句等同于这三个语句：

    ```sql
    SET character_set_client = *charset_name*;
    SET character_set_results = *charset_name*;
    SET character_set_connection = *charset_name*;
    ```

    将`character_set_connection`设置为*`charset_name`*也会隐式地将`collation_connection`设置为*`charset_name`*的默认排序规则。不需要显式设置该排序规则。要指定用于`collation_connection`的特定排序规则，请添加`COLLATE`子句：

    ```sql
    SET NAMES '*charset_name*' COLLATE '*collation_name*'
    ```

+   `SET CHARACTER SET '*`charset_name`*'`

    `SET CHARACTER SET`类似于`SET NAMES`，但将`character_set_connection`和`collation_connection`设置为`character_set_database`和`collation_database`（如前所述，指示默认数据库的字符集和排序规则）。

    `SET CHARACTER SET *`charset_name`*`语句等同于这三个语句：

    ```sql
    SET character_set_client = *charset_name*;
    SET character_set_results = *charset_name*;
    SET collation_connection = @@collation_database;
    ```

    设置`collation_connection`也会隐式地将`character_set_connection`设置为与排序规则相关联的字符集（等同于执行`SET character_set_connection = @@character_set_database`）。不需要显式设置`character_set_connection`。

注意

有些字符集不能作为客户端字符集使用。尝试在`SET NAMES`或`SET CHARACTER SET`中使用它们会产生错误。请参阅不允许的客户端字符集。

例如：假设`column1`定义为`CHAR(5) CHARACTER SET latin2`。如果不在发出`SELECT column1 FROM t`之前说`SET NAMES`或`SET CHARACTER SET`，那么服务器会使用客户端连接时指定的字符集发送`column1`的所有值。另一方面，如果在发出`SELECT`语句之前说`SET NAMES 'latin1'`或`SET CHARACTER SET 'latin1'`，服务器会在发送结果之前将`latin2`值转换为`latin1`。对于不在两个字符集中的字符，转换可能会有损失。

### 连接字符集错误处理

尝试使用不合适的连接字符集或排序规则可能会产生错误，或导致服务器回退到给定连接的默认字符集和排序规则。本节描述了在配置连接字符集时可能出现的问题。这些问题可能在建立连接时或在已建立连接中更改字符集时发生。

+   连接时错误处理

+   运行时错误处理

#### 连接时错误处理

一些字符集不能用作客户端字符集；参见不允许的客户端字符集。如果指定了一个有效但不允许作为客户端字符集的字符集，服务器会返回一个错误：

```sql
$> mysql --default-character-set=ucs2
ERROR 1231 (42000): Variable 'character_set_client' can't be set to
the value of 'ucs2'
```

如果指定了客户端不认识的字符集，会产生错误：

```sql
$> mysql --default-character-set=bogus
mysql: Character set 'bogus' is not a compiled character set and is
not specified in the '/usr/local/mysql/share/charsets/Index.xml' file
ERROR 2019 (HY000): Can't initialize character set bogus
(path: /usr/local/mysql/share/charsets/)
```

如果指定了客户端识别但服务器不识别的字符集，服务器会回退到其默认字符集和排序规则。假设服务器配置为使用`latin1`和`latin1_swedish_ci`作为默认值，并且不认识`gb18030`作为有效字符集。指定`--default-character-set=gb18030`的客户端可以连接到服务器，但结果的字符集不是客户端想要的：

```sql
mysql> SHOW SESSION VARIABLES LIKE 'character\_set\_%';
+--------------------------+--------+
| Variable_name            | Value  |
+--------------------------+--------+
| character_set_client     | latin1 |
| character_set_connection | latin1 |
...
| character_set_results    | latin1 |
...
+--------------------------+--------+
mysql> SHOW SESSION VARIABLES LIKE 'collation_connection';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | latin1_swedish_ci |
+----------------------+-------------------+
```

您可以看到连接系统变量已设置为反映`latin1`和`latin1_swedish_ci`的字符集和排序规则。这是因为服务器无法满足客户端字符集请求并回退到其默认值。

在这种情况下，客户端无法使用它想要的字符集，因为服务器不支持。客户端必须要么愿意使用不同的字符集，要么连接到支持所需字符集的不同服务器。

在更微妙的情况下也会出现相同的问题：当客户端告诉服务器使用服务器识别的字符集，但客户端端默认排序规则在服务器端是未知的。例如，当一个 MySQL 8.0 客户端想要使用`utf8mb4`作为客户端字符集连接到 MySQL 5.7 服务器时，就会出现这种情况。指定`--default-character-set=utf8mb4`的客户端可以连接到服务器。然而，与前面的例子一样，服务器会回退到其默认字符集和排序规则，而不是客户端请求的内容：

```sql
mysql> SHOW SESSION VARIABLES LIKE 'character\_set\_%';
+--------------------------+--------+
| Variable_name            | Value  |
+--------------------------+--------+
| character_set_client     | latin1 |
| character_set_connection | latin1 |
...
| character_set_results    | latin1 |
...
+--------------------------+--------+
mysql> SHOW SESSION VARIABLES LIKE 'collation_connection';
+----------------------+-------------------+
| Variable_name        | Value             |
+----------------------+-------------------+
| collation_connection | latin1_swedish_ci |
+----------------------+-------------------+
```

为什么会发生这种情况？毕竟，`utf8mb4` 是 8.0 客户端和 5.7 服务器都知道的，所以它们都识别它。要理解这种行为，有必要了解当客户端告诉服务器它想要使用哪种字符集时，实际上是告诉服务器该字符集的默认排序规则。因此，上述行为是由多种因素的组合造成的：

+   `utf8mb4` 的默认排序规则在 MySQL 5.7 和 8.0 之间不同（5.7 为 `utf8mb4_general_ci`，8.0 为 `utf8mb4_0900_ai_ci`）。

+   当 8.0 客户端请求一个字符集为 `utf8mb4` 时，发送给服务器的是默认的 8.0 `utf8mb4` 排序规则；即 `utf8mb4_0900_ai_ci`。

+   `utf8mb4_0900_ai_ci` 仅在 MySQL 8.0 中实现，因此 5.7 服务器不识别它。

+   由于 5.7 服务器不识别 `utf8mb4_0900_ai_ci`，无法满足客户端字符集请求，因此回退到其默认字符集和排序规则（`latin1` 和 `latin1_swedish_ci`）。

在这种情况下，客户端仍然可以在连接后发出 `SET NAMES 'utf8mb4'` 语句来使用 `utf8mb4`。结果的排序规则是 5.7 默认的 `utf8mb4` 排序规则；即 `utf8mb4_general_ci`。如果客户端还想要 `utf8mb4_0900_ai_ci` 的排序规则，由于服务器不识别该排序规则，无法实现。客户端必须要么愿意使用不同的 `utf8mb4` 排序规则，要么连接到 MySQL 8.0 或更高版本的服务器。

#### 运行时错误处理

在已建立的连接中，客户端可以通过 `SET NAMES` 或 `SET CHARACTER SET` 请求更改连接字符集和排序规则。

一些字符集不能用作客户端字符集；请参阅不允许的客户端字符集。如果指定了一个有效但不允许作为客户端字符集的字符集，服务器会返回一个错误：

```sql
mysql> SET NAMES 'ucs2';
ERROR 1231 (42000): Variable 'character_set_client' can't be set to
the value of 'ucs2'
```

如果服务器不识别字符集（或排序规则），则会产生错误：

```sql
mysql> SET NAMES 'bogus';
ERROR 1115 (42000): Unknown character set: 'bogus' 
mysql> SET NAMES 'utf8mb4' COLLATE 'bogus';
ERROR 1273 (HY000): Unknown collation: 'bogus'
```

提示

一个想要验证服务器是否接受其请求的字符集的客户端可以在连接后执行以下语句，并检查结果是否是预期的字符集：

```sql
SELECT @@character_set_client;
```
