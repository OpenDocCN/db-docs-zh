# 14.11 XML 函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/xml-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/xml-functions.html)

**表 14.16 XML 函数**

| 名称 | 描述 |
| --- | --- |
| `ExtractValue()` | 使用 XPath 表示法从 XML 字符串中提取值 |
| `UpdateXML()` | 返回替换的 XML 片段 |

本节讨论了 MySQL 中的 XML 及相关功能。

注意

可以通过在 `--xml` 选项下调用 **mysql** 和 **mysqldump** 客户端来从 MySQL 中获取 XML 格式化输出。请参阅 Section 6.5.1, “mysql — The MySQL Command-Line Client”，以及 Section 6.5.4, “mysqldump — A Database Backup Program”。

提供基本 XPath 1.0（XML Path Language，版本 1.0）功能的两个函数可用。关于 XPath 语法和用法的一些基本信息稍后在本节中提供；但是，关于这些主题的深入讨论超出了本手册的范围，您应参考 [XML Path Language (XPath) 1.0 标准](http://www.w3.org/TR/xpath) 获取确切信息。对于对 XPath 新手或希望复习基础知识的人来说，[Zvon.org XPath 教程](http://www.zvon.org/xxl/XPathTutorial/) 是一个有用的资源，提供多种语言版本。

注意

这些函数仍在开发中。我们将继续改进 MySQL 8.0 及以后版本中的 XML 和 XPath 功能的其他方面。您可以在 [MySQL XML 用户论坛](https://forums.mysql.com/list.php?44) 中讨论这些问题，提出问题，并从其他用户那里获得帮助。

与这些函数一起使用的 XPath 表达式支持用户变量和存储程序局部变量。用户变量进行弱类型检查；存储程序局部变量进行强类型检查（另请参见 Bug #26518）：

+   **用户变量（弱类型检查）。** 使用 `$@*`variable_name`*`（即用户变量）语法的变量不受检查。如果变量类型错误或之前未分配值，服务器不会发出警告或错误。这也意味着用户完全负责任何打字错误，因为如果（例如）使用 `$@myvairable` 而意图使用 `$@myvariable`，服务器不会发出警告。 

    示例：

    ```sql
    mysql> SET @xml = '<a><b>X</b><b>Y</b></a>';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SET @i =1, @j = 2;
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @i, ExtractValue(@xml, '//b[$@i]');
    +------+--------------------------------+
    | @i   | ExtractValue(@xml, '//b[$@i]') |
    +------+--------------------------------+
    |    1 | X                              |
    +------+--------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT @j, ExtractValue(@xml, '//b[$@j]');
    +------+--------------------------------+
    | @j   | ExtractValue(@xml, '//b[$@j]') |
    +------+--------------------------------+
    |    2 | Y                              |
    +------+--------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT @k, ExtractValue(@xml, '//b[$@k]');
    +------+--------------------------------+
    | @k   | ExtractValue(@xml, '//b[$@k]') |
    +------+--------------------------------+
    | NULL |                                |
    +------+--------------------------------+
    1 row in set (0.00 sec)
    ```

+   **存储程序中的变量（强类型检查）。** 使用 `$*`variable_name`*` 语法声明和使用这些函数时，当它们在存储程序内部调用时，这些变量是局部于定义它们的存储程序，并且对类型和值进行了强类型检查。

    示例：

    ```sql
    mysql> DELIMITER |

    mysql> CREATE PROCEDURE myproc ()
     -> BEGIN
     ->   DECLARE i INT DEFAULT 1;
     ->   DECLARE xml VARCHAR(25) DEFAULT '<a>X</a><a>Y</a><a>Z</a>';
     ->
     ->   WHILE i < 4 DO
     ->     SELECT xml, i, ExtractValue(xml, '//a[$i]');
     ->     SET i = i+1;
     ->   END WHILE;
     -> END |
    Query OK, 0 rows affected (0.01 sec)

    mysql> DELIMITER ;

    mysql> CALL myproc();
    +--------------------------+---+------------------------------+
    | xml                      | i | ExtractValue(xml, '//a[$i]') |
    +--------------------------+---+------------------------------+
    | <a>X</a><a>Y</a><a>Z</a> | 1 | X                            |
    +--------------------------+---+------------------------------+
    1 row in set (0.00 sec)

    +--------------------------+---+------------------------------+
    | xml                      | i | ExtractValue(xml, '//a[$i]') |
    +--------------------------+---+------------------------------+
    | <a>X</a><a>Y</a><a>Z</a> | 2 | Y                            |
    +--------------------------+---+------------------------------+
    1 row in set (0.01 sec)

    +--------------------------+---+------------------------------+
    | xml                      | i | ExtractValue(xml, '//a[$i]') |
    +--------------------------+---+------------------------------+
    | <a>X</a><a>Y</a><a>Z</a> | 3 | Z                            |
    +--------------------------+---+------------------------------+
    1 row in set (0.01 sec)
    ```

    **参数。** 在存储过程内部 XPath 表达式中使用的变量作为参数传递时也要进行严格检查。

包含用户变量或存储程序本地变量的表达式必须（除了符号）符合 XPath 1.0 规范中包含变量的 XPath 表达式的规则。

注意

用于存储 XPath 表达式的用户变量被视为空字符串。因此，不可能将 XPath 表达式存储为用户变量。（Bug #32911）

+   `ExtractValue(*`xml_frag`*, *`xpath_expr`*)`

    `ExtractValue()`接受两个字符串参数，一个是 XML 标记片段*`xml_frag`*，另一个是 XPath 表达式*`xpath_expr`*（也称为定位器）；它返回与 XPath 表达式匹配的元素的第一个文本节点的文本（`CDATA`）。

    使用此函数等同于在追加`/text()`后使用*`xpath_expr`*进行匹配。换句话说，`ExtractValue('<a><b>Sakila</b></a>', '/a/b')`和`ExtractValue('<a><b>Sakila</b></a>', '/a/b/text()')`产生相同的结果。如果*`xml_frag`*或*`xpath_expr`*为`NULL`，函数将返回`NULL`。

    如果找到多个匹配项，则返回每个匹配元素的第一个子文本节点的内容（按匹配顺序）作为一个单独的、以空格分隔的字符串。

    如果找不到与表达式匹配的文本节点（包括隐式的`/text()`）——无论出于何种原因，只要*`xpath_expr`*有效，*`xml_frag`*由正确嵌套和关闭的元素组成——则返回空字符串。不区分匹配空元素和根本没有匹配。这是设计上的考虑。

    如果您需要确定*`xml_frag`*中是否找不到匹配元素或找到这样的元素但不包含子文本节点，您应该测试使用 XPath `count()`函数的表达式的结果。例如，这两个语句都返回空字符串，如下所示：

    ```sql
    mysql> SELECT ExtractValue('<a><b/></a>', '/a/b');
    +-------------------------------------+
    | ExtractValue('<a><b/></a>', '/a/b') |
    +-------------------------------------+
    |                                     |
    +-------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue('<a><c/></a>', '/a/b');
    +-------------------------------------+
    | ExtractValue('<a><c/></a>', '/a/b') |
    +-------------------------------------+
    |                                     |
    +-------------------------------------+
    1 row in set (0.00 sec)
    ```

    然而，您可以通过以下方式确定是否实际上存在匹配的元素：

    ```sql
    mysql> SELECT ExtractValue('<a><b/></a>', 'count(/a/b)');
    +-------------------------------------+
    | ExtractValue('<a><b/></a>', 'count(/a/b)') |
    +-------------------------------------+
    | 1                                   |
    +-------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue('<a><c/></a>', 'count(/a/b)');
    +-------------------------------------+
    | ExtractValue('<a><c/></a>', 'count(/a/b)') |
    +-------------------------------------+
    | 0                                   |
    +-------------------------------------+
    1 row in set (0.01 sec)
    ```

    重要提示

    `ExtractValue()`仅返回`CDATA`，不返回匹配标记内可能包含的任何标记，也不返回它们的内容（请参见以下示例中作为`val1`返回的结果）。

    ```sql
    mysql> SELECT
     ->   ExtractValue('<a>ccc<b>ddd</b></a>', '/a') AS val1,
     ->   ExtractValue('<a>ccc<b>ddd</b></a>', '/a/b') AS val2,
     ->   ExtractValue('<a>ccc<b>ddd</b></a>', '//b') AS val3,
     ->   ExtractValue('<a>ccc<b>ddd</b></a>', '/b') AS val4,
     ->   ExtractValue('<a>ccc<b>ddd</b><b>eee</b></a>', '//b') AS val5;

    +------+------+------+------+---------+
    | val1 | val2 | val3 | val4 | val5    |
    +------+------+------+------+---------+
    | ccc  | ddd  | ddd  |      | ddd eee |
    +------+------+------+------+---------+
    ```

    此函数使用当前的 SQL 校对规则来与`contains()`进行比较，执行与其他字符串函数（如`CONCAT()`）相同的校对聚合，考虑到它们的参数的校对可强制性；请参阅第 12.8.4 节，“表达式中的校对可强制性”，了解规定此行为的规则的解释。

    （以前，总是使用二进制—即，区分大小写—比较。）

    如果*`xml_frag`*包含未正确嵌套或关闭的元素，则返回`NULL`，并生成警告，如下例所示：

    ```sql
    mysql> SELECT ExtractValue('<a>c</a><b', '//a');
    +-----------------------------------+
    | ExtractValue('<a>c</a><b', '//a') |
    +-----------------------------------+
    | NULL                              |
    +-----------------------------------+
    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Warning
       Code: 1525
    Message: Incorrect XML value: 'parse error at line 1 pos 11:
             END-OF-INPUT unexpected ('>' wanted)' 1 row in set (0.00 sec)

    mysql> SELECT ExtractValue('<a>c</a><b/>', '//a');
    +-------------------------------------+
    | ExtractValue('<a>c</a><b/>', '//a') |
    +-------------------------------------+
    | c                                   |
    +-------------------------------------+
    1 row in set (0.00 sec)
    ```

+   `UpdateXML(*`xml_target`*, *`xpath_expr`*, *`new_xml`*)`

    此函数用新的 XML 片段*`new_xml`*替换给定 XML 标记片段*`xml_target`*的单个部分，然后返回更改后的 XML。被替换的*`xml_target`*部分与用户提供的 XPath 表达式*`xpath_expr`*匹配。

    如果找不到与*`xpath_expr`*匹配的表达式，或者找到多个匹配项，则该函数返回原始的*`xml_target`* XML 片段。所有三个参数都应为字符串。如果`UpdateXML()`的任何参数为`NULL`，则函数返回`NULL`。

    ```sql
    mysql> SELECT
     ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/a', '<e>fff</e>') AS val1,
     ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/b', '<e>fff</e>') AS val2,
     ->   UpdateXML('<a><b>ccc</b><d></d></a>', '//b', '<e>fff</e>') AS val3,
     ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/a/d', '<e>fff</e>') AS val4,
     ->   UpdateXML('<a><d></d><b>ccc</b><d></d></a>', '/a/d', '<e>fff</e>') AS val5
     -> \G

    *************************** 1\. row ***************************
    val1: <e>fff</e>
    val2: <a><b>ccc</b><d></d></a>
    val3: <a><e>fff</e><d></d></a>
    val4: <a><b>ccc</b><e>fff</e></a>
    val5: <a><d></d><b>ccc</b><d></d></a>
    ```

注意

对 XPath 语法和用法的深入讨论超出了本手册的范围。请参阅[XML Path Language (XPath) 1.0 规范](http://www.w3.org/TR/xpath)以获取确切信息。对于那些对 XPath 新手或希望在基础知识方面进行复习的人来说，[Zvon.org XPath 教程](http://www.zvon.org/xxl/XPathTutorial/)是一个有用的资源，提供多种语言版本。

以下是一些基本 XPath 表达式的描述和示例：

+   `/*`tag`*`

    仅当`<*`tag`*/>`是根元素时，才匹配`<*`tag`*/>`。

    示例：`/a`在`<a><b/></a>`中有匹配项，因为它匹配最外层（根）标记。它不匹配`<b><a/></b>`中的内部*`a`*元素，因为在这种情况下，它是另一个元素的子元素。

+   `/*`tag1`*/*`tag2`*`

    仅当`<*`tag2`*/>`是`<*`tag1`*/>`的子元素，并且`<*`tag1`*/>`是根元素时，才匹配`<*`tag2`*/>`。

    示例：`/a/b`匹配 XML 片段`<a><b/></a>`中的*`b`*元素，因为它是根元素*`a`*的子元素。它在`<b><a/></b>`中没有匹配项，因为在这种情况下，*`b`*是根元素（因此不是其他元素的子元素）。XPath 表达式在`<a><c><b/></c></a>`中也没有匹配项；这里，*`b`*是*`a`*的后代，但实际上不是*`a`*的子元素。

    此结构可扩展到三个或更多元素。例如，XPath 表达式`/a/b/c`匹配片段`<a><b><c/></b></a>`中的*`c`*元素。

+   `//*`tag`*`

    匹配任何`<*`tag`*>`的实例。

    示例：`//a` 匹配以下任意一个元素中的*`a`*元素：`<a><b><c/></b></a>`；`<c><a><b/></a></b>`；`<c><b><a/></b></c>`。

    `//`可以与`/`结合使用。例如，`//a/b`匹配片段`<a><b/></a>`或`<c><a><b/></a></c>`中的`b`元素。

    注意

    `//*`tag`*`等同于`/descendant-or-self::*/*`tag`*`。一个常见错误是将其与`/descendant-or-self::*`tag`*`混淆，尽管后者的表达实际上可能导致非常不同的结果，如下所示：

    ```sql
    mysql> SET @xml = '<a><b><c>w</c><b>x</b><d>y</d>z</b></a>';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @xml;
    +-----------------------------------------+
    | @xml                                    |
    +-----------------------------------------+
    | <a><b><c>w</c><b>x</b><d>y</d>z</b></a> |
    +-----------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue(@xml, '//b[1]');
    +------------------------------+
    | ExtractValue(@xml, '//b[1]') |
    +------------------------------+
    | x z                          |
    +------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue(@xml, '//b[2]');
    +------------------------------+
    | ExtractValue(@xml, '//b[2]') |
    +------------------------------+
    |                              |
    +------------------------------+
    1 row in set (0.01 sec)

    mysql> SELECT ExtractValue(@xml, '/descendant-or-self::*/b[1]');
    +---------------------------------------------------+
    | ExtractValue(@xml, '/descendant-or-self::*/b[1]') |
    +---------------------------------------------------+
    | x z                                               |
    +---------------------------------------------------+
    1 row in set (0.06 sec)

    mysql> SELECT ExtractValue(@xml, '/descendant-or-self::*/b[2]');
    +---------------------------------------------------+
    | ExtractValue(@xml, '/descendant-or-self::*/b[2]') |
    +---------------------------------------------------+
    |                                                   |
    +---------------------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue(@xml, '/descendant-or-self::b[1]');
    +-------------------------------------------------+
    | ExtractValue(@xml, '/descendant-or-self::b[1]') |
    +-------------------------------------------------+
    | z                                               |
    +-------------------------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT ExtractValue(@xml, '/descendant-or-self::b[2]');
    +-------------------------------------------------+
    | ExtractValue(@xml, '/descendant-or-self::b[2]') |
    +-------------------------------------------------+
    | x                                               |
    +-------------------------------------------------+
    1 row in set (0.00 sec)
    ```

+   `*`运算符充当“通配符”，匹配任何元素。例如，表达式`/*/b`匹配 XML 片段`<a><b/></a>`或`<c><b/></c>`中的`b`元素。然而，在片段`<b><a/></b>`中，该表达式不会产生匹配，因为`b`必须是其他元素的子元素。通配符可以在任何位置使用：表达式`/*/b/*`匹配`b`元素的任何子元素，该子元素本身不是根元素。

+   您可以使用`|`（`UNION`）运算符匹配多个定位器中的任何一个。例如，表达式`//b|//c`匹配 XML 目标中的所有`b`和`c`元素。

+   还可以根据一个或多个属性的值匹配元素。使用语法`*`tag`*[@*`attribute`*="*`value`*"]`。例如，表达式`//b[@id="idB"]`匹配片段`<a><b id="idA"/><c/><b id="idB"/></a>`中的第二个`b`元素。要匹配具有`*`attribute`*="*`value`*"`的*任何*元素，请使用 XPath 表达式`//*[*`attribute`*="*`value`*"]`。

    要过滤多个属性值，只需连续使用多个属性比较子句。例如，表达式`//b[@c="x"][@d="y"]`匹配出现在给定 XML 片段中的任何位置的元素`<b c="x" d="y"/>`。

    要查找具有多个值中的任何一个匹配的元素，可以使用由`|`运算符连接的多个定位器。例如，要匹配所有`b`元素，其`c`属性具有值 23 或 17 中的任何一个，请使用表达式`//b[@c="23"]|//b[@c="17"]`。您也可以使用逻辑`or`运算符来实现此目的：`//b[@c="23" or @c="17"]`。

    注意

    `or`和`|`之间的区别在于`or`连接条件，而`|`连接结果集。

**XPath 限制。** 这些函数支持的 XPath 语法目前受到以下限制：

+   不支持节点集到节点集的比较（例如`'/a/b[@c=@d]'`）。

+   所有标准的 XPath 比较运算符都受支持。（Bug #22823）

+   相对定位器表达式在根节点的上下文中解析。例如，考虑以下查询和结果：

    ```sql
    mysql> SELECT ExtractValue(
     ->   '<a><b c="1">X</b><b c="2">Y</b></a>',
     ->    'a/b'
     -> ) AS result;
    +--------+
    | result |
    +--------+
    | X Y    |
    +--------+
    1 row in set (0.03 sec)
    ```

    在这种情况下，定位器`a/b`解析为`/a/b`。

    在谓词内也支持相对定位器。在以下示例中，`d[../@c="1"]`解析为`/a/b[@c="1"]/d`：

    ```sql
    mysql> SELECT ExtractValue(
     ->      '<a>
        ->        <b c="1"><d>X</d></b>
        ->        <b c="2"><d>X</d></b>
        ->      </a>',
     ->      'a/b/d[../@c="1"]')
     -> AS result;
    +--------+
    | result |
    +--------+
    | X      |
    +--------+
    1 row in set (0.00 sec)
    ```

+   不允许使用作为标量值评估的表达式前缀定位器，包括变量引用、文字、数字和标量函数调用，其使用会导致错误。

+   `::` 运算符与以下节点类型的组合不受支持：

    +   `*`axis`*::comment()`

    +   `*`axis`*::text()`

    +   `*`axis`*::processing-instructions()`

    +   `*`axis`*::node()`

    然而，名称测试（如`*`axis`*::*`name`*`和`*`axis`*::*`）是受支持的，如下例所示：

    ```sql
    mysql> SELECT ExtractValue('<a><b>x</b><c>y</c></a>','/a/child::b');
    +-------------------------------------------------------+
    | ExtractValue('<a><b>x</b><c>y</c></a>','/a/child::b') |
    +-------------------------------------------------------+
    | x                                                     |
    +-------------------------------------------------------+
    1 row in set (0.02 sec)

    mysql> SELECT ExtractValue('<a><b>x</b><c>y</c></a>','/a/child::*');
    +-------------------------------------------------------+
    | ExtractValue('<a><b>x</b><c>y</c></a>','/a/child::*') |
    +-------------------------------------------------------+
    | x y                                                   |
    +-------------------------------------------------------+
    1 row in set (0.01 sec)
    ```

+   在路径会导致“超出”根元素的情况下，不支持“上下”导航。也就是说，您不能使用在给定元素的祖先的后代上匹配的表达式，其中当前元素的一个或多个祖先也是根元素的祖先（请参见 Bug #16321）。

+   不支持以下 XPath 函数，或者存在已知问题，如下所示：

    +   `id()`

    +   `lang()`

    +   `local-name()`

    +   `name()`

    +   `namespace-uri()`

    +   `normalize-space()`

    +   `starts-with()`

    +   `string()`

    +   `substring-after()`

    +   `substring-before()`

    +   `translate()`

+   不支持以下轴：

    +   `following-sibling`

    +   `following`

    +   `preceding-sibling`

    +   `preceding`

作为参数传递给 `ExtractValue()` 和 `UpdateXML()` 的 XPath 表达式可能包含元素选择器中的冒号字符（`:`），这使它们可以与使用 XML 命名空间表示法的标记一起使用。例如：

```sql
mysql> SET @xml = '<a>111<b:c>222<d>333</d><e:f>444</e:f></b:c></a>';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT ExtractValue(@xml, '//e:f');
+-----------------------------+
| ExtractValue(@xml, '//e:f') |
+-----------------------------+
| 444                         |
+-----------------------------+
1 row in set (0.00 sec)

mysql> SELECT UpdateXML(@xml, '//b:c', '<g:h>555</g:h>');
+--------------------------------------------+
| UpdateXML(@xml, '//b:c', '<g:h>555</g:h>') |
+--------------------------------------------+
| <a>111<g:h>555</g:h></a>                   |
+--------------------------------------------+
1 row in set (0.00 sec)
```

这在某些方面类似于 [Apache Xalan](http://xalan.apache.org/) 和其他一些解析器允许的内容，比起要求命名空间声明或使用 `namespace-uri()` 和 `local-name()` 函数要简单得多。

**错误处理。** 对于 `ExtractValue()` 和 `UpdateXML()`，使用的 XPath 定位器必须有效，并且要搜索的 XML 必须由正确嵌套和关闭的元素组成。如果定位器无效，则会生成错误：

```sql
mysql> SELECT ExtractValue('<a>c</a><b/>', '/&a');
ERROR 1105 (HY000): XPATH syntax error: '&a'
```

如果 *`xml_frag`* 不包含正确嵌套和关闭的元素，则返回 `NULL` 并生成警告，如下例所示：

```sql
mysql> SELECT ExtractValue('<a>c</a><b', '//a');
+-----------------------------------+
| ExtractValue('<a>c</a><b', '//a') |
+-----------------------------------+
| NULL                              |
+-----------------------------------+
1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 1525
Message: Incorrect XML value: 'parse error at line 1 pos 11:
         END-OF-INPUT unexpected ('>' wanted)' 1 row in set (0.00 sec)

mysql> SELECT ExtractValue('<a>c</a><b/>', '//a');
+-------------------------------------+
| ExtractValue('<a>c</a><b/>', '//a') |
+-------------------------------------+
| c                                   |
+-------------------------------------+
1 row in set (0.00 sec)
```

重要提示

作为传递给 `UpdateXML()` 的第三个参数的替换 XML *不会* 被检查，以确定它是否仅由正确嵌套和关闭的元素组成。

**XPath 注入。** 代码注入发生在恶意代码被引入系统以获取未经授权的权限和数据时。它基于开发人员对用户输入数据类型和内容所做的假设。XPath 也不例外。

一个常见的情况是应用程序处理授权的情况，通过将登录名和密码的组合与 XML 文件中找到的内容进行匹配，使用类似于以下的 XPath 表达式：

```sql
//user[login/text()='neapolitan' and password/text()='1c3cr34m']/attribute::id
```

这是类似于 SQL 语句的 XPath 等效语句：

```sql
SELECT id FROM users WHERE login='neapolitan' AND password='1c3cr34m';
```

使用 XPath 的 PHP 应用程序可能会处理登录过程如下：

```sql
<?php

  $file     =   "users.xml";

  $login    =   $POST["login"];
  $password =   $POST["password"];

  $xpath = "//user[login/text()=$login and password/text()=$password]/attribute::id";

  if( file_exists($file) )
  {
    $xml = simplexml_load_file($file);

    if($result = $xml->xpath($xpath))
      echo "You are now logged in as user $result[0].";
    else
      echo "Invalid login name or password.";
  }
  else
    exit("Failed to open $file.");

?>
```

输入上没有进行检查。这意味着恶意用户可以通过在登录名和密码中输入`' or 1=1`来“绕过”测试，导致`$xpath`被评估如下所示：

```sql
//user[login/text()='' or 1=1 and password/text()='' or 1=1]/attribute::id
```

由于方括号内的表达式始终评估为`true`，它实际上与这个表达式相同，该表达式匹配 XML 文档中每个`user`元素的`id`属性：

```sql
//user/attribute::id
```

可以简单地通过在`$xpath`的定义中引用要插入的变量名来规避这种特定攻击，强制从 Web 表单传递的值转换为字符串：

```sql
$xpath = "//user[login/text()='$login' and password/text()='$password']/attribute::id";
```

这与通常用于防止 SQL 注入攻击的策略相同。一般来说，用于防止 XPath 注入攻击的实践应该与防止 SQL 注入攻击的实践相同：

+   永远不要在应用程序中接受未经测试的用户数据。

+   检查所有用户提交的数据的类型；拒绝或转换错误类型的数据。

+   测试数值数据是否超出范围；截断、四舍五入或拒绝超出范围的值。测试字符串是否包含非法字符，要么将其剥离，要么拒绝包含这些字符的输入。

+   不要输出可能向未经授权的用户提供线索以危害系统的明确错误消息；而是将这些记录到文件或数据库表中。

就像 SQL 注入攻击可以用于获取有关数据库模式的信息一样，XPath 注入也可以用于遍历 XML 文件以揭示其结构，如 Amit Klein 的论文[Blind XPath Injection](http://www.packetstormsecurity.org/papers/bypass/Blind_XPath_Injection_20040518.pdf)（PDF 文件，46KB）中所讨论的。

重要的是检查发送回客户端的输出。考虑当我们使用 MySQL `ExtractValue()` 函数时可能发生的情况：

```sql
mysql> SELECT ExtractValue(
 ->     LOAD_FILE('users.xml'),
 ->     '//user[login/text()="" or 1=1 and password/text()="" or 1=1]/attribute::id'
 -> ) AS id;
+-------------------------------+
| id                            |
+-------------------------------+
| 00327 13579 02403 42354 28570 |
+-------------------------------+
1 row in set (0.01 sec)
```

因为 `ExtractValue()` 将多个匹配项作为一个以空格分隔的字符串返回，这种注入攻击将 `users.xml` 中的每个有效 ID 作为单行输出提供给用户。作为额外的保障，你还应该在返回给用户之前测试输出。这里是一个简单的例子：

```sql
mysql> SELECT @id = ExtractValue(
 ->     LOAD_FILE('users.xml'),
 ->     '//user[login/text()="" or 1=1 and password/text()="" or 1=1]/attribute::id'
 -> );
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT IF(
 ->     INSTR(@id, ' ') = 0,
 ->     @id,
 ->     'Unable to retrieve user ID')
 -> AS singleID;
+----------------------------+
| singleID                   |
+----------------------------+
| Unable to retrieve user ID |
+----------------------------+
1 row in set (0.00 sec)
```

一般来说，将数据安全返回给用户的指导原则与接受用户输入的原则相同。这可以总结为：

+   始终测试传出数据的类型和允许的值。

+   永远不要允许未经授权的用户查看可能提供有关应用程序信息的错误消息，这些信息可能被用于利用它。
