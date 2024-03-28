# A.11 MySQL 8.0 FAQ：MySQL 中文、日文和韩文字符集

> 原文：[`dev.mysql.com/doc/refman/8.0/en/faqs-cjk.html`](https://dev.mysql.com/doc/refman/8.0/en/faqs-cjk.html)

这组常见问题来源于 MySQL 支持和开发团队处理许多关于 CJK（中文-日文-韩文）问题的经验。

A.11.1\. MySQL 中有哪些 CJK 字符集可用？

A.11.2\. 我已经将 CJK 字符插入到我的表中。为什么 SELECT 显示它们为“?”字符？

A.11.3\. 在使用 Big5 中文字符集时应注意哪些问题？

A.11.4\. 为什么日文字符集转换失败？

A.11.5\. 如果我想将 SJIS 81CA 转换为 cp932，我该怎么办？

A.11.6\. MySQL 如何表示日元（¥）符号？

A.11.7\. 在 MySQL 中使用韩文字符集时应注意哪些问题？

A.11.8\. 为什么会出现“Incorrect string value”错误消息？

A.11.9\. 为什么我的 GUI 前端或浏览器在使用 Access、PHP 或其他 API 的应用程序中不正确显示 CJK 字符？

A.11.10\. 我已升级到 MySQL 8.0。如何恢复到 MySQL 4.0 中关于字符集的行为？

A.11.11\. 为什么一些带有 CJK 字符的 LIKE 和 FULLTEXT 搜索失败？

A.11.12\. 如何知道字符 X 是否在所有字符集中都可用？

A.11.13\. 为什么在 Unicode 中 CJK 字符串排序不正确？（I）

A.11.14\. 为什么在 Unicode 中 CJK 字符串排序不正确？（II）

A.11.15\. 为什么我的补充字符被 MySQL 拒绝？

A.11.16\. “CJK” 应该是“CJKV”吗？

A.11.17\. MySQL 允许在数据库和表名中使用 CJK 字符吗？

A.11.18\. 在哪里可以找到 MySQL 手册的中文、日文和韩文翻译？

A.11.19\. 在 MySQL 中如何获取有关 CJK 和相关问题的帮助？

| **A.11.1.** | MySQL 中有哪些 CJK 字符集可用？ |
| --- | --- |

|  | CJK 字符集的列表可能会根据您的 MySQL 版本而有所不同。例如，`gb18030`字符集在 MySQL 5.7.4 之前不受支持。然而，由于适用语言的名称出现在`INFORMATION_SCHEMA.CHARACTER_SETS`表中的`DESCRIPTION`列中，您可以使用此查询获取所有非 Unicode CJK 字符集的当前列表：

```sql
mysql> SELECT CHARACTER_SET_NAME, DESCRIPTION
       FROM INFORMATION_SCHEMA.CHARACTER_SETS
       WHERE DESCRIPTION LIKE '%Chin%'
       OR DESCRIPTION LIKE '%Japanese%'
       OR DESCRIPTION LIKE '%Korean%'
       ORDER BY CHARACTER_SET_NAME;
+--------------------+---------------------------------+
&#124; CHARACTER_SET_NAME &#124; DESCRIPTION                     &#124;
+--------------------+---------------------------------+
&#124; big5               &#124; Big5 Traditional Chinese        &#124;
&#124; cp932              &#124; SJIS for Windows Japanese       &#124;
&#124; eucjpms            &#124; UJIS for Windows Japanese       &#124;
&#124; euckr              &#124; EUC-KR Korean                   &#124;
&#124; gb18030            &#124; China National Standard GB18030 &#124;
&#124; gb2312             &#124; GB2312 Simplified Chinese       &#124;
&#124; gbk                &#124; GBK Simplified Chinese          &#124;
&#124; sjis               &#124; Shift-JIS Japanese              &#124;
&#124; ujis               &#124; EUC-JP Japanese                 &#124;
+--------------------+---------------------------------+
```

（更多信息，请参阅第 28.3.4 节，“INFORMATION_SCHEMA CHARACTER_SETS 表”。）MySQL 支持三种 GB（*国家标准*或*简体中文*）字符集的变体，这在中华人民共和国是官方的：`gb2312`、`gbk`和（自 MySQL 5.7.4 起）`gb18030`。有时人们尝试将`gbk`字符插入`gb2312`，大多数情况下都可以成功，因为`gbk`是`gb2312`的超集。但最终他们尝试插入一个更罕见的中文字符，却无法成功。（例如，请参阅 Bug #16072）。在这里，我们试图明确`gb2312`或`gbk`中哪些字符是合法的，参考官方文件。请在报告`gb2312`或`gbk`错误之前检查这些参考文献：

+   MySQL 的`gbk`字符集实际上是“Microsoft 代码页 936”。这与官方的`gbk`有所不同，对于字符`A1A4`（中点）、`A1AA`（破折号）、`A6E0-A6F5`和`A8BB-A8C0`。

+   要查看`gbk`/Unicode 映射列表，请参阅[`www.unicode.org/Public/MAPPINGS/VENDORS/MICSFT/WINDOWS/CP936.TXT`](http://www.unicode.org/Public/MAPPINGS/VENDORS/MICSFT/WINDOWS/CP936.TXT)。

也可以在 Unicode 字符集中存储 CJK 字符，尽管可用的排序规则可能不会按您的预期对字符进行排序：

+   `utf8`和`ucs2`字符集支持 Unicode 基本多语言平面（BMP）中的字符。这些字符的代码点值介于`U+0000`和`U+FFFF`之间。

+   `utf8mb4`、`utf16`、`utf16le`和`utf32`字符集支持 BMP 字符，以及位于 BMP 之外的补充字符。补充字符的代码点值介于`U+10000`和`U+10FFFF`之间。

用于 Unicode 字符集的排序规则决定了对集合中的字符进行排序（即区分）的能力：

+   基于 Unicode Collation Algorithm（UCA）4.0.0 的排序规则只区分 BMP 字符。

+   基于 UCA 5.2.0 或 9.0.0 的排序规则区分 BMP 和补充字符。

+   非 UCA 排序规则可能无法区分所有 Unicode 字符。例如，MySQL 的`utf8mb4`默认排序规则是`utf8mb4_general_ci`，只区分 BMP 字符。

此外，区分字符并不等同于按照给定 CJK 语言的约定对其进行排序。目前，MySQL 只有一个 CJK 特定的 UCA 排序规则，即`gb18030_unicode_520_ci`（需要使用非 Unicode `gb18030`字符集）。有关 Unicode 排序规则及其区分属性的信息，包括辅助字符的排序属性，请参见第 12.10.1 节，“Unicode 字符集”。

| **A.11.2.** | 我已经将 CJK 字符插入到我的表中。为什么`SELECT`显示它们为“?”字符？ |
| --- | --- |

|  | 这个问题通常是由于 MySQL 中的设置与应用程序或操作系统的设置不匹配。以下是纠正这些问题的一些常见步骤：

+   *确保您正在使用的 MySQL 版本*。

    使用语句`SELECT VERSION();`来确定这一点。

+   *确保数据库实际上正在使用所需的字符集*。

    人们经常认为客户端字符集总是与服务器字符集或用于显示目的的字符集相同。然而，这两者都是错误的假设。您可以通过检查`SHOW CREATE TABLE *`tablename`*`的结果或更好地使用以下语句来确保：

    ```sql
    SELECT character_set_name, collation_name
        FROM information_schema.columns
        WHERE table_schema = your_database_name
            AND table_name = your_table_name
            AND column_name = your_column_name;
    ```

+   *确定未正确显示的字符或字符的十六进制值*。

    您可以使用以下查询在表*`table_name`*中获取列*`column_name`*的此信息：

    ```sql
    SELECT HEX(*column_name*)
    FROM *table_name*;
    ```

    `3F`是`?`字符的编码；这意味着`?`实际上是存储在列中的字符。这通常是由于将特定字符从客户端字符集转换为目标字符集时出现问题而导致的。

+   *确保往返旅程是可能的。当您选择*`literal`*（或*`_introducer hexadecimal-value`*）时，您是否获得*`literal`*作为结果*？

    例如，日文片假名字符*Pe*（`ペ'`）存在于所有 CJK 字符集中，并且具有十六进制编码的代码点值`0x30da`。要测试此字符的往返旅程，请使用以下查询：

    ```sql
    SELECT 'ペ' AS `ペ`;         /* or SELECT _ucs2 0x30da; */
    ```

    如果结果不是`ペ`，则往返旅程失败。

    对于关于此类失败的错误报告，我们可能会要求您跟进`SELECT HEX('ペ');`。然后我们可以确定客户端编码是否正确。

+   *确保问题不是由浏览器或其他应用程序引起的，而不是由 MySQL 引起的*。

    使用**mysql**客户端程序来完成此任务。如果**mysql**正确显示字符，但您的应用程序没有显示正确，那么您的问题可能是由于系统设置引起的。

    要确定您的设置，请使用`SHOW VARIABLES`语句，其输出应类似于所示内容：

    ```sql
    mysql> SHOW VARIABLES LIKE 'char%';
    +--------------------------+----------------------------------------+
    &#124; Variable_name            &#124; Value                                  &#124;
    +--------------------------+----------------------------------------+
    &#124; character_set_client     &#124; utf8                                   &#124;
    &#124; character_set_connection &#124; utf8                                   &#124;
    &#124; character_set_database   &#124; latin1                                 &#124;
    &#124; character_set_filesystem &#124; binary                                 &#124;
    &#124; character_set_results    &#124; utf8                                   &#124;
    &#124; character_set_server     &#124; latin1                                 &#124;
    &#124; character_set_system     &#124; utf8                                   &#124;
    &#124; character_sets_dir       &#124; /usr/local/mysql/share/mysql/charsets/ &#124;
    +--------------------------+----------------------------------------+
    ```

    这些是面向国际客户的典型字符集设置（请注意使用`utf8` Unicode）连接到西方服务器（`latin1`是西欧字符集）。

    尽管 Unicode（通常在 Unix 上是`utf8`变体，在 Windows 上是`ucs2`变体）优于 Latin，但通常不是您的操作系统实用程序最好支持的。许多 Windows 用户发现，例如对于日本 Windows，使用 Microsoft 字符集，如`cp932`，是合适的。

    如果您无法控制服务器设置，并且不知道底层计算机使用的设置，请尝试切换到您所在国家的常见字符集（`euckr` = 韩国；`gb18030`、`gb2312`或`gbk` = 中华人民共和国；`big5` = 台湾；`sjis`、`ujis`、`cp932`或`eucjpms` = 日本；`ucs2`或`utf8` = 任何地方）。通常只需要更改客户端、连接和结果设置。`SET NAMES`语句可以同时更改这三个设置。例如：

    ```sql
    SET NAMES 'big5';
    ```

    一旦设置正确，您可以通过编辑`my.cnf`或`my.ini`使其永久化。例如，您可以添加类似以下内容的行：

    ```sql
    [mysqld]
    character-set-server=big5
    [client]
    default-character-set=big5
    ```

    可能还存在与应用程序中使用的 API 配置设置有关的问题；有关更多信息，请参阅*为什么我的 GUI 前端或浏览器无法正确显示 CJK 字符...？*。

|

| **A.11.3.** | 在使用 Big5 中文字符集时应该注意哪些问题？ |
| --- | --- |
|  | MySQL 支持在香港和台湾（中华民国）常见的 Big5 字符集。MySQL 的`big5`字符集实际上是 Microsoft 的代码页 950，与原始的`big5`字符集非常相似。已经提出了添加`HKSCS`扩展的功能请求。需要此扩展的人可能会对 Bug＃13577 的建议补丁感兴趣。 |
| **A.11.4.** | 为什么日文字符集转换会失败？ |

|  | MySQL 支持`sjis`、`ujis`、`cp932`和`eucjpms`字符集，以及 Unicode。常见的需求是在字符集之间进行转换。例如，可能有一个 Unix 服务器（通常使用`sjis`或`ujis`）和一个 Windows 客户端（通常使用`cp932`）。在下表中，`ucs2`列代表源，而`sjis`、`cp932`、`ujis`和`eucjpms`列代表目的地；也就是说，最后 4 列提供了当我们使用`CONVERT(ucs2)`或将包含该值的`ucs2`列分配给`sjis`、`cp932`、`ujis`或`eucjpms`列时的十六进制结果。

&#124; 字符名称 &#124; ucs2 &#124; sjis &#124; cp932 &#124; ujis &#124; eucjpms &#124;

&#124; 破折号 &#124; 00A6 &#124; 3F &#124; 3F &#124; 8FA2C3 &#124; 3F &#124;

&#124; 全角破折号 &#124; FFE4 &#124; 3F &#124; FA55 &#124; 3F &#124; 8FA2 &#124;

&#124; 日元符号 &#124; 00A5 &#124; 3F &#124; 3F &#124; 20 &#124; 3F &#124;

&#124; 全角日元符号 &#124; FFE5 &#124; 818F &#124; 818F &#124; A1EF &#124; 3F &#124;

&#124; 波浪线 &#124; 007E &#124; 7E &#124; 7E &#124; 7E &#124; 7E &#124;

&#124; 上划线 &#124; 203E &#124; 3F &#124; 3F &#124; 20 &#124; 3F &#124;

&#124; 横线 &#124; 2015 &#124; 815C &#124; 815C &#124; A1BD &#124; A1BD &#124;

&#124; 破折号 &#124; 2014 &#124; 3F &#124; 3F &#124; 3F &#124; 3F &#124;

&#124; 反斜线 &#124; 005C &#124; 815F &#124; 5C &#124; 5C &#124; 5C &#124;

&#124; 全角反斜线 &#124; FF3C &#124; 3F &#124; 815F &#124; 3F &#124; A1C0 &#124;

&#124; 波浪破折号 &#124; 301C &#124; 8160 &#124; 3F &#124; A1C1 &#124; 3F &#124;

&#124; 全角波浪线 &#124; FF5E &#124; 3F &#124; 8160 &#124; 3F &#124; A1C1 &#124;

&#124; 双竖线 &#124; 2016 &#124; 8161 &#124; 3F &#124; A1C2 &#124; 3F &#124;

&#124; 平行于 &#124; 2225 &#124; 3F &#124; 8161 &#124; 3F &#124; A1C2 &#124;

&#124; 减号符号 &#124; 2212 &#124; 817C &#124; 3F &#124; A1DD &#124; 3F &#124;

&#124; 全角连字符 &#124; FF0D &#124; 3F &#124; 817C &#124; 3F &#124; A1DD &#124;

&#124; 分币符号 &#124; 00A2 &#124; 8191 &#124; 3F &#124; A1F1 &#124; 3F &#124;

&#124; 全角分币符号 &#124; FFE0 &#124; 3F &#124; 8191 &#124; 3F &#124; A1F1 &#124;

&#124; 英镑符号 &#124; 00A3 &#124; 8192 &#124; 3F &#124; A1F2 &#124; 3F &#124;

&#124; 全角英镑符号 &#124; FFE1 &#124; 3F &#124; 8192 &#124; 3F &#124; A1F2 &#124;

&#124; 非逻辑非符 &#124; 00AC &#124; 81CA &#124; 3F &#124; A2CC &#124; 3F &#124;

&#124; 全角非逻辑非符 &#124; FFE2 &#124; 3F &#124; 81CA &#124; 3F &#124; A2CC &#124;

&#124; 字符名称 &#124; ucs2 &#124; sjis &#124; cp932 &#124; ujis &#124; eucjpms &#124;

现在考虑表格的以下部分。

&#124;  &#124; ucs2 &#124; sjis &#124; cp932 &#124;

&#124; 非逻辑非符 &#124; 00AC &#124; 81CA &#124; 3F &#124;

&#124; 全角非逻辑非符 &#124; FFE2 &#124; 3F &#124; 81CA &#124;

这意味着 MySQL 将`非逻辑非符`（Unicode `U+00AC`）转换为`sjis`代码点`0x81CA`，转换为`cp932`代码点`3F`。（`3F`是问号（“?”）。这是在无法执行转换时始终使用的。） |

| **A.11.5.** | 如果我想将 SJIS `81CA` 转换为 `cp932`，我应该怎么做？ |
| --- | --- |
|  | 我们的答案是：“?”。这样做有缺点，许多人更喜欢“宽松”转换，这样`sjis`中的`81CA（非逻辑非符）`就会变成`cp932`中的`81CA（全角非逻辑非符）`。 |
| **A.11.6.** | MySQL 如何表示日元（¥）符号？ |
|  | 一个问题出现在于一些版本的日文字符集（`sjis`和`euc`）将`5C`视为反斜杠（`\`，也称为反斜线），而其他版本将其视为日元符号（¥）。MySQL 遵循 JIS（日本工业标准）标准描述的一个版本。在 MySQL 中，*`5C`始终是反斜杠（\）*。 |
| **A.11.7.** | 在 MySQL 中使用韩文字符集时应注意哪些问题？ |

| 理论上，虽然有几个版本的`euckr`（扩展 Unix 代码韩国）字符集，但只有一个问题被指出。我们使用“ASCII”变体的 EUC-KR，其中代码点`0x5c`是反斜杠，即`\`，而不是“KS-Roman”变体的 EUC-KR，其中代码点`0x5c`是`WON SIGN`（₩）。这意味着你无法将 Unicode `U+20A9`转换为`euckr`：

```sql
mysql> SELECT
           CONVERT('₩' USING euckr) AS euckr,
           HEX(CONVERT('₩' USING euckr)) AS hexeuckr;
+-------+----------+
&#124; euckr &#124; hexeuckr &#124;
+-------+----------+
&#124; ?     &#124; 3F       &#124;
+-------+----------+
```

|

| **A.11.8.** | 为什么我会收到不正确的字符串值错误消息？ |
| --- | --- |

|  | 要查看问题，创建一个具有一个 Unicode（`ucs2`）列和一个中文（`gb2312`）列的表。

```sql
mysql> CREATE TABLE ch
       (ucs2 CHAR(3) CHARACTER SET ucs2,
       gb2312 CHAR(3) CHARACTER SET gb2312);
```

在非严格 SQL 模式下，尝试在两列中都放置罕见字符`汌`。

```sql
mysql> SET sql_mode = '';
mysql> INSERT INTO ch VALUES ('A 汌 B','A 汌 B');
Query OK, 1 row affected, 1 warning (0.00 sec)
```

`INSERT`产生一个警告。使用以下语句查看是什么：

```sql
mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1366
Message: Incorrect string value: '\xE6\xB1\x8CB' for column 'gb2312' at row 1
```

所以这只是关于`gb2312`列的警告。

```sql
mysql> SELECT ucs2,HEX(ucs2),gb2312,HEX(gb2312) FROM ch;
+-------+--------------+--------+-------------+
&#124; ucs2  &#124; HEX(ucs2)    &#124; gb2312 &#124; HEX(gb2312) &#124;
+-------+--------------+--------+-------------+
&#124; A 汌 B &#124; 00416C4C0042 &#124; A?B    &#124; 413F42      &#124;
+-------+--------------+--------+-------------+
```

这里有几件事情需要解释：

1.  如前所述，`汌`字符不在`gb2312`字符集中。

1.  如果你正在使用旧版本的 MySQL，你可能会看到不同的消息。

1.  出现警告而不是错误，是因为 MySQL 没有设置为使用严格的 SQL 模式。在非严格模式下，MySQL 会尽力而为，以获得最佳匹配，而不是放弃。在严格的 SQL 模式下，不正确的字符串值消息会作为错误而不是警告出现，`INSERT`将失败。

|

| **A.11.9.** | 为什么我的 GUI 前端或浏览器在使用 Access、PHP 或其他 API 的应用程序中不正确显示 CJK 字符？ |
| --- | --- |

|  | 使用**mysql**客户端直接连接到服务器，并在那里尝试相同的查询。如果**mysql**正确响应，问题可能是你的应用程序接口需要初始化。使用**mysql**告诉你它使用的字符集或字符集的语句`SHOW VARIABLES LIKE 'char%';`。如果你正在使用 Access，你很可能是通过 Connector/ODBC 连接的。在这种情况下，你应该检查配置 Connector/ODBC。例如，如果你使用`big5`，你会输入`SET NAMES 'big5'`。（在这种情况下，不需要`；`字符。）如果你正在使用 ASP，你可能需要在代码中添加`SET NAMES`。以下是过去有效的示例：

```sql
<%
Session.CodePage=0
Dim strConnection
Dim Conn
strConnection="driver={MySQL ODBC 3.51 Driver};server=*server*;uid=*username*;" \
               & "pwd=*password*;database=*database*;stmt=SET NAMES 'big5';"
Set Conn = Server.CreateObject("ADODB.Connection")
Conn.Open strConnection
%>
```

同样，如果你在 Connector/NET 中使用除`latin1`之外的任何字符集，你必须在连接字符串中指定字符集。有关更多信息，请参阅 Connector/NET Connections。如果���正在使用 PHP，尝试这样做：

```sql
<?php
  $link = new mysqli($host, $usr, $pwd, $db);

  if( mysqli_connect_errno() )
  {
    printf("Connect failed: %s\n", mysqli_connect_error());
    exit();
  }

  $link->query("SET NAMES 'utf8'");
?>
```

在这种情况下，我们使用 `SET NAMES` 来更改 `character_set_client`、`character_set_connection` 和 `character_set_results`。PHP 应用程序经常遇到的另一个问题与浏览器的假设有关。有时添加或更改 `<meta>` 标签就足以解决问题：例如，为了确保用户代理将页面内容解释为 `UTF-8`，在 HTML 页面的 `<head>` 部分包含 `<meta http-equiv="Content-Type" content="text/html; charset=utf-8">`。如果您正在使用 Connector/J，请参阅 使用字符集和 Unicode。

| **A.11.10.** | 我已升级到 MySQL 8.0。如何恢复到 MySQL 4.0 中关于字符集的行为？ |
| --- | --- |

|  | 在 MySQL 版本 4.0 中，服务器和客户端共用一个“全局”字符集，由服务器管理员决定使用哪种字符集。从 MySQL 版本 4.1 开始，情况发生了变化。现在是一个“握手”过程，如 第 12.4 节，“连接字符集和校对规则” 中所描述的：

> 当客户端连接时，它会向服务器发送要使用的字符集的名称。服务器使用该名称设置 `character_set_client`、`character_set_results` 和 `character_set_connection` 系统变量。实际上，服务器使用字符集名称执行 `SET NAMES` 操作。

这样做的效果是，您无法通过使用`--character-set-server=utf8`启动**mysqld**来控制客户端字符集。然而，一些亚洲客户更喜欢 MySQL 4.0 的行为。为了保留这种行为，我们添加了一个**mysqld**开关，`--character-set-client-handshake`，可以通过`--skip-character-set-client-handshake`关闭。如果您使用`--skip-character-set-client-handshake`启动**mysqld**，那么当客户端连接时，它会向服务器发送它想要使用的字符集的名称。然而，*服务器会忽略客户端的这个请求*。举例来说，假设您最喜欢的服务器字符集是`latin1`。进一步假设客户端使用`utf8`，因为这是客户端操作系统支持的。以`latin1`作为默认字符集启动服务器：

```sql
mysqld --character-set-server=latin1
```

然后使用默认字符集`utf8`启动客户端：

```sql
mysql --default-character-set=utf8
```

可以通过查看`SHOW VARIABLES`的输出来查看结果设置：

```sql
mysql> SHOW VARIABLES LIKE 'char%';
+--------------------------+----------------------------------------+
&#124; Variable_name            &#124; Value                                  &#124;
+--------------------------+----------------------------------------+
&#124; character_set_client     &#124; utf8                                   &#124;
&#124; character_set_connection &#124; utf8                                   &#124;
&#124; character_set_database   &#124; latin1                                 &#124;
&#124; character_set_filesystem &#124; binary                                 &#124;
&#124; character_set_results    &#124; utf8                                   &#124;
&#124; character_set_server     &#124; latin1                                 &#124;
&#124; character_set_system     &#124; utf8                                   &#124;
&#124; character_sets_dir       &#124; /usr/local/mysql/share/mysql/charsets/ &#124;
+--------------------------+----------------------------------------+
```

现在停止客户端，并使用**mysqladmin**停止服务器。然后再次启动服务器，但这次告诉它跳过握手过程，如下所示：

```sql
mysqld --character-set-server=utf8 --skip-character-set-client-handshake
```

再次将客户端以`utf8`作为默认字符集启动，然后显示结果设置：

```sql
mysql> SHOW VARIABLES LIKE 'char%';
+--------------------------+----------------------------------------+
&#124; Variable_name            &#124; Value                                  &#124;
+--------------------------+----------------------------------------+
&#124; character_set_client     &#124; latin1                                 &#124;
&#124; character_set_connection &#124; latin1                                 &#124;
&#124; character_set_database   &#124; latin1                                 &#124;
&#124; character_set_filesystem &#124; binary                                 &#124;
&#124; character_set_results    &#124; latin1                                 &#124;
&#124; character_set_server     &#124; latin1                                 &#124;
&#124; character_set_system     &#124; utf8                                   &#124;
&#124; character_sets_dir       &#124; /usr/local/mysql/share/mysql/charsets/ &#124;
+--------------------------+----------------------------------------+
```

通过比较`SHOW VARIABLES`的不同结果，可以看到如果使用`--skip-character-set-client-handshake`选项，服务器会忽略客户端的初始设置。|

| **A.11.11.** | 为什么一些带有 CJK 字符的`LIKE`和`FULLTEXT`搜索会失败？ |
| --- | --- |

|  | 对于`LIKE`搜索，像`BINARY`和`BLOB`这样的二进制字符串列类型存在一个非常简单的问题：我们必须知道字符的结束位置。在多字节字符集中，不同的字符可能具有不同的八位字节长度。例如，在`utf8`中，`A`需要一个字节，但`ペ`需要三个字节，如下所示：

```sql
+-------------------------+---------------------------+
&#124; OCTET_LENGTH(_utf8 'A') &#124; OCTET_LENGTH(_utf8 'ペ') &#124;
+-------------------------+---------------------------+
&#124;                       1 &#124;                         3 &#124;
+-------------------------+---------------------------+
```

如果我们不知道字符串中第一个字符的结束位置，那么我们也不知道第二个字符从哪里开始，这种情况下，即使是非常简单的搜索，比如`LIKE '_A%'`也会失败。解决方法是使用定义为具有适当 CJK 字符集的非二进制字符串列类型。例如：`mycol TEXT CHARACTER SET sjis`。或者，在比较之前转换为 CJK 字符集。这也是 MySQL 不能允许不存在字符的编码的原因之一。如果不严格拒绝错误输入，MySQL 就无法知道字符的结束位置。对于`FULLTEXT`搜索，我们必须知道单词的起始和结束位置。对于西方语言，这很少是问题，因为大多数（如果不是全部）都使用易于识别的单词边界：空格字符。然而，对于亚洲文字通常不是这种情况。我们可以使用任意的中间措施，比如假设所有汉字代表单词，或者（对于日语）依赖于片假名到平假名的变化，因为语法结尾。然而，唯一确定的解决方案需要一个全面的单词列表，这意味着我们必须为每种支持的亚洲语言在服务器中包含一个字典。这根本不可行。

| **A.11.12.** | 我如何知道字符*`X`*在所有字符集中是否可用？ |
| --- | --- |

|  | 大多数简体中文和基本非半角宽日语假名字符出现在所有 CJK 字符集中。以下存储过程接受一个`UCS-2` Unicode 字符，将其转换为其他字符集，并以十六进制显示结果。

```sql
DELIMITER //

CREATE PROCEDURE p_convert(ucs2_char CHAR(1) CHARACTER SET ucs2)
BEGIN

CREATE TABLE tj
             (ucs2 CHAR(1) character set ucs2,
              utf8 CHAR(1) character set utf8,
              big5 CHAR(1) character set big5,
              cp932 CHAR(1) character set cp932,
              eucjpms CHAR(1) character set eucjpms,
              euckr CHAR(1) character set euckr,
              gb2312 CHAR(1) character set gb2312,
              gbk CHAR(1) character set gbk,
              sjis CHAR(1) character set sjis,
              ujis CHAR(1) character set ujis);

INSERT INTO tj (ucs2) VALUES (ucs2_char);

UPDATE tj SET utf8=ucs2,
              big5=ucs2,
              cp932=ucs2,
              eucjpms=ucs2,
              euckr=ucs2,
              gb2312=ucs2,
              gbk=ucs2,
              sjis=ucs2,
              ujis=ucs2;

/* If there are conversion problems, UPDATE produces warnings. */

SELECT hex(ucs2) AS ucs2,
       hex(utf8) AS utf8,
       hex(big5) AS big5,
       hex(cp932) AS cp932,
       hex(eucjpms) AS eucjpms,
       hex(euckr) AS euckr,
       hex(gb2312) AS gb2312,
       hex(gbk) AS gbk,
       hex(sjis) AS sjis,
       hex(ujis) AS ujis
FROM tj;

DROP TABLE tj;

END//

DELIMITER ;
```

输入可以是任何单个`ucs2`字符，或者可以是该字符的代码值（十六进制表示）。例如，从 Unicode 的`ucs2`编码和名称列表（[`www.unicode.org/Public/UNIDATA/UnicodeData.txt`](http://www.unicode.org/Public/UNIDATA/UnicodeData.txt)）中，我们知道片假名字符*Pe*出现在所有 CJK 字符集中，并且其代码值为`X'30DA'`。如果我们将这个值作为`p_convert()`的参数，结果如下所示：

```sql
mysql> CALL p_convert(X'30DA');
+------+--------+------+-------+---------+-------+--------+------+------+------+
&#124; ucs2 &#124; utf8   &#124; big5 &#124; cp932 &#124; eucjpms &#124; euckr &#124; gb2312 &#124; gbk  &#124; sjis &#124; ujis &#124;
+------+--------+------+-------+---------+-------+--------+------+------+------+
&#124; 30DA &#124; E3839A &#124; C772 &#124; 8379  &#124; A5DA    &#124; ABDA  &#124; A5DA   &#124; A5DA &#124; 8379 &#124; A5DA &#124;
+------+--------+------+-------+---------+-------+--------+------+------+------+
```

由于列值中没有`3F`（即问号字符，`?`），我们知道每次转换都成功了。

| **A.11.13.** | 为什么 CJK 字符串在 Unicode 中排序不正确？（I） |
| --- | --- |
|  | 在 MySQL 8.0 中，可以通过使用`utf8mb4`字符集和`utf8mb4_ja_0900_as_cs`校对规则来解决旧版本 MySQL 中出现的 CJK 排序问题。 |
| **A.11.14.** | 为什么 CJK 字符串在 Unicode 中排序不正确？（II） |
|  | 在 MySQL 8.0 中，可以通过使用`utf8mb4`字符集和`utf8mb4_ja_0900_as_cs`校对规则来解决旧版本 MySQL 中出现的 CJK 排序问题。 |
| **A.11.15.** | 为什么我的补充字符被 MySQL 拒绝？ |

|  | 补充字符位于 Unicode *基本多语言平面/平面 0*之外。BMP 字符的代码点值介于`U+0000`和`U+FFFF`之间。补充字符的代码点值介于`U+10000`和`U+10FFFF`之间。要存储补充字符，必须使用允许它们的字符集：

+   `utf8`和`ucs2`字符集仅支持 BMP 字符。

    `utf8`字符集仅允许占用最多三个字节的`UTF-8`字符。这导致了 Bug #12600 中的报告，我们将其拒绝为“不是一个 bug”。使用`utf8`时，MySQL 在遇到无法理解的字节时必须截断输入字符串。否则，无法确定坏的多字节字符有多长。

    一个可能的解决方法是使用`ucs2`代替`utf8`，在这种情况下，“坏”字符会被更改为问号。但是，不会发生截断。您还可以将数据类型更改为`BLOB`或`BINARY`，这些类型不执行有效性检查。

+   `utf8mb4`、`utf16`、`utf16le`和`utf32`字符集支持 BMP 字符，以及 BMP 之外的补充字符。

|

| **A.11.16.** | “CJK”应该改为“CJKV”吗？ |
| --- | --- |
|  | 不。术语“CJKV”（中文、日文、韩文、越南文）指的是包含汉字（最初是中文）字符的越南字符集。MySQL 支持带有西方字符的现代越南文脚本，但不支持使用汉字字符的旧越南文脚本。截至 MySQL 5.6，有适用于 Unicode 字符集的越南文排序规则，如第 12.10.1 节“Unicode 字符集”所述。 |
| **A.11.17.** | MySQL 允许在数据库和表名中使用 CJK 字符吗？ |
|  | 是的。 |
| **A.11.18.** | 我在哪里可以找到 MySQL 手册的中文、日文和韩文翻译？ |
|  | MySQL 5.6 手册的日文翻译可从`dev.mysql.com/doc/`下载。 |
| **A.11.19.** | 我在哪里可以获取有关 MySQL 中 CJK 和相关问题的帮助？ |

|  | 以下资源可供使用：

+   可在[`wikis.oracle.com/display/mysql/List+of+MySQL+User+Groups`](https://wikis.oracle.com/display/mysql/List+of+MySQL+User+Groups)找到 MySQL 用户组列表。

+   查看与字符集问题相关的功能请求，请访问[`tinyurl.com/y6xcuf`](http://tinyurl.com/y6xcuf)。

+   访问 MySQL [字符集、排序规则、Unicode 论坛](https://forums.mysql.com/list.php?103)。[`forums.mysql.com/`](http://forums.mysql.com/)也提供外语论坛。

|
