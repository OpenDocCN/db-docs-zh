# 14.17.8 JSON 实用函数

> 原文：[`dev.mysql.com/doc/refman/8.0/en/json-utility-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-utility-functions.html)

本节记录了作用于 JSON 值或可解析为 JSON 值的字符串的实用函数。`JSON_PRETTY()` 以易于阅读的格式打印出 JSON 值。`JSON_STORAGE_SIZE()` 和 `JSON_STORAGE_FREE()` 分别显示给定 JSON 值使用的存储空间量以及部分更新后 `JSON` 列中剩余空间量。

+   `JSON_PRETTY(*`json_val`*)`

    提供类似于 PHP 和其他语言和数据库系统中实现的 JSON 值的漂亮打印。提供的值必须是 JSON 值或 JSON 值的有效字符串表示。该值中存在的多余空格和换行符对输出没有影响。对于 `NULL` 值，该函数返回 `NULL`。如果值不是 JSON 文档，或者无法解析为 JSON 文档，则函数将出错。

    此函数的输出格式遵循以下规则：

    +   每个数组元素或对象成员显示在单独的一行上，相对于其父级缩进一个额外级别。

    +   每个缩进级别增加两个前导空格。

    +   在分隔两个元素或成员的换行符之前打印分隔单个数组元素或对象成员的逗号。

    +   对象成员的键和值由冒号后跟一个空格（'`:` '）分隔。

    +   空对象或数组打印在一行上。在开放和闭合大括号之间不打印空格。

    +   字符串标量和键名中的特殊字符使用与 `JSON_QUOTE()` 函数相同的规则进行转义。

    ```sql
    mysql> SELECT JSON_PRETTY('123'); # scalar
    +--------------------+
    | JSON_PRETTY('123') |
    +--------------------+
    | 123                |
    +--------------------+

    mysql> SELECT JSON_PRETTY("[1,3,5]"); # array
    +------------------------+
    | JSON_PRETTY("[1,3,5]") |
    +------------------------+
    | [
      1,
      3,
      5
    ]      |
    +------------------------+

    mysql> SELECT JSON_PRETTY('{"a":"10","b":"15","x":"25"}'); # object
    +---------------------------------------------+
    | JSON_PRETTY('{"a":"10","b":"15","x":"25"}') |
    +---------------------------------------------+
    | {
      "a": "10",
      "b": "15",
      "x": "25"
    }   |
    +---------------------------------------------+

    mysql> SELECT JSON_PRETTY('["a",1,{"key1":
        '>    "value1"},"5",     "77" ,
        '>       {"key2":["value3","valueX",
        '> "valueY"]},"j", "2"   ]')\G  # nested arrays and objects
    *************************** 1\. row ***************************
    JSON_PRETTY('["a",1,{"key1":
                 "value1"},"5",     "77" ,
                    {"key2":["value3","valuex",
              "valuey"]},"j", "2"   ]'): [
      "a",
      1,
      {
        "key1": "value1"
      },
      "5",
      "77",
      {
        "key2": [
          "value3",
          "valuex",
          "valuey"
        ]
      },
      "j",
      "2"
    ]
    ```

+   `JSON_STORAGE_FREE(*`json_val`*)`

    对于`JSON`列值，此函数显示了在使用`JSON_SET()`、`JSON_REPLACE()`或`JSON_REMOVE()`进行原地更新后，其二进制表示中释放了多少存储空间。参数也可以是有效的 JSON 文档或可以解析为 JSON 的字符串——无论是作为文字值还是作为用户变量的值——在这种情况下，函数返回 0。如果参数是已根据前述描述更新的`JSON`列值，使其二进制表示占用的空间少于更新之前，则返回一个正的非零值。对于已更新的`JSON`列，其二进制表示与之前相同或更大，或者更新无法利用部分更新的情况，返回 0；如果参数为`NULL`，则返回`NULL`。

    如果*`json_val`*不为`NULL`，且既不是有效的 JSON 文档，也无法成功解析为 JSON 文档，则会产生错误。

    在此示例中，我们创建一个包含`JSON`列的表，然后插入一个包含 JSON 对象的行：

    ```sql
    mysql> CREATE TABLE jtable (jcol JSON);
    Query OK, 0 rows affected (0.38 sec)

    mysql> INSERT INTO jtable VALUES
     ->     ('{"a": 10, "b": "wxyz", "c": "[true, false]"}');
    Query OK, 1 row affected (0.04 sec)

    mysql> SELECT * FROM jtable;
    +----------------------------------------------+
    | jcol                                         |
    +----------------------------------------------+
    | {"a": 10, "b": "wxyz", "c": "[true, false]"} |
    +----------------------------------------------+
    1 row in set (0.00 sec)
    ```

    现在我们使用`JSON_SET()`更新列值，以便执行部分更新；在这种情况下，我们用占用空间更少的值（整数`1`）替换了由`c`键指向的值（数组`[true, false]）：

    ```sql
    mysql> UPDATE jtable
     ->     SET jcol = JSON_SET(jcol, "$.a", 10, "$.b", "wxyz", "$.c", 1);
    Query OK, 1 row affected (0.03 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT * FROM jtable;
    +--------------------------------+
    | jcol                           |
    +--------------------------------+
    | {"a": 10, "b": "wxyz", "c": 1} |
    +--------------------------------+
    1 row in set (0.00 sec)

    mysql> SELECT JSON_STORAGE_FREE(jcol) FROM jtable;
    +-------------------------+
    | JSON_STORAGE_FREE(jcol) |
    +-------------------------+
    |                      14 |
    +-------------------------+
    1 row in set (0.00 sec)
    ```

    连续部分更新对此空闲空间的影响是累积的，如此示例中使用`JSON_SET()`减少具有键`b`的值占用的空间（并不进行其他更改）所示：

    ```sql
    mysql> UPDATE jtable
     ->     SET jcol = JSON_SET(jcol, "$.a", 10, "$.b", "wx", "$.c", 1);
    Query OK, 1 row affected (0.03 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT JSON_STORAGE_FREE(jcol) FROM jtable;
    +-------------------------+
    | JSON_STORAGE_FREE(jcol) |
    +-------------------------+
    |                      16 |
    +-------------------------+
    1 row in set (0.00 sec)
    ```

    在不使用`JSON_SET()`、`JSON_REPLACE()`或`JSON_REMOVE()`更新列的情况下，意味着优化器无法原地执行更新；在这种情况下，`JSON_STORAGE_FREE()`返回 0，如下所示：

    ```sql
    mysql> UPDATE jtable SET jcol = '{"a": 10, "b": 1}';
    Query OK, 1 row affected (0.05 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT JSON_STORAGE_FREE(jcol) FROM jtable;
    +-------------------------+
    | JSON_STORAGE_FREE(jcol) |
    +-------------------------+
    |                       0 |
    +-------------------------+
    1 row in set (0.00 sec)
    ```

    只能对列值执行 JSON 文档的部分更新。对于存储 JSON 值的用户变量，值总是完全替换，即使使用`JSON_SET()`执行更新时也是如此：

    ```sql
    mysql> SET @j = '{"a": 10, "b": "wxyz", "c": "[true, false]"}';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SET @j = JSON_SET(@j, '$.a', 10, '$.b', 'wxyz', '$.c', '1');
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @j, JSON_STORAGE_FREE(@j) AS Free;
    +----------------------------------+------+
    | @j                               | Free |
    +----------------------------------+------+
    | {"a": 10, "b": "wxyz", "c": "1"} |    0 |
    +----------------------------------+------+
    1 row in set (0.00 sec)
    ```

    对于 JSON 文本，此函数始终返回 0：

    ```sql
    mysql> SELECT JSON_STORAGE_FREE('{"a": 10, "b": "wxyz", "c": "1"}') AS Free;
    +------+
    | Free |
    +------+
    |    0 |
    +------+
    1 row in set (0.00 sec)
    ```

+   `JSON_STORAGE_SIZE(*`json_val`*)`

    此函数返回用于存储 JSON 文档的二进制表示所使用的字节数。当参数为`JSON`列时，这是用于存储 JSON 文档的空间，即插入到列中之前的空间，之后可能对其执行的任何部分更新。*`json_val`*必须是有效的 JSON 文档或可以成功解析为 JSON 的字符串。如果它是字符串，则函数返回通过将字符串解析为 JSON 并将其转换为二进制而创建的 JSON 二进制表示中的存储空间量。如果参数为`NULL`，则返回`NULL`。

    当*`json_val`*不是`NULL`，并且不是或无法成功解析为 JSON 文档时，会产生错误。

    为了说明当使用`JSON`列作为参数时，此函数的行为，我们创建一个名为`jtable`的表，其中包含一个`JSON`列`jcol`，将一个 JSON 值插入表中，然后使用`JSON_STORAGE_SIZE()`获取此列使用的存储空间，如下所示：

    ```sql
    mysql> CREATE TABLE jtable (jcol JSON);
    Query OK, 0 rows affected (0.42 sec)

    mysql> INSERT INTO jtable VALUES
     ->     ('{"a": 1000, "b": "wxyz", "c": "[1, 3, 5, 7]"}');
    Query OK, 1 row affected (0.04 sec)

    mysql> SELECT
     ->     jcol,
     ->     JSON_STORAGE_SIZE(jcol) AS Size,
     ->     JSON_STORAGE_FREE(jcol) AS Free
     -> FROM jtable;
    +-----------------------------------------------+------+------+
    | jcol                                          | Size | Free |
    +-----------------------------------------------+------+------+
    | {"a": 1000, "b": "wxyz", "c": "[1, 3, 5, 7]"} |   47 |    0 |
    +-----------------------------------------------+------+------+
    1 row in set (0.00 sec)
    ```

    根据`JSON_STORAGE_SIZE()`的输出，插入到列中的 JSON 文档占用了 47 字节的空间。我们还使用`JSON_STORAGE_FREE()`检查了通过任何先前的列部分更新释放的空间量；由于尚未执行任何更新，因此这个值是 0，符合预期。

    接下来我们对表执行一个`UPDATE`，这应该会导致存储在`jcol`中的文档的部分更新，然后测试结果如下：

    ```sql
    mysql> UPDATE jtable SET jcol = 
     ->     JSON_SET(jcol, "$.b", "a");
    Query OK, 1 row affected (0.04 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT
     ->     jcol,
     ->     JSON_STORAGE_SIZE(jcol) AS Size,
     ->     JSON_STORAGE_FREE(jcol) AS Free
     -> FROM jtable;
    +--------------------------------------------+------+------+
    | jcol                                       | Size | Free |
    +--------------------------------------------+------+------+
    | {"a": 1000, "b": "a", "c": "[1, 3, 5, 7]"} |   47 |    3 |
    +--------------------------------------------+------+------+
    1 row in set (0.00 sec)
    ```

    前一个查询中`JSON_STORAGE_FREE()`返回的值表明对 JSON 文档进行了部分更新，并且这释放了 3 字节的用于存储它的空间。`JSON_STORAGE_SIZE()`返回的结果不受部分更新的影响。

    使用`JSON_SET()`、`JSON_REPLACE()`或`JSON_REMOVE()`进行更新支持部分更新。不能对`JSON`列直接赋值进行部分更新；在此类更新之后，`JSON_STORAGE_SIZE()`始终显示新设置值的存储使用情况：

    ```sql
    mysql> UPDATE jtable
    mysql>     SET jcol = '{"a": 4.55, "b": "wxyz", "c": "[true, false]"}';
    Query OK, 1 row affected (0.04 sec)
    Rows matched: 1  Changed: 1  Warnings: 0

    mysql> SELECT
     ->     jcol,
     ->     JSON_STORAGE_SIZE(jcol) AS Size,
     ->     JSON_STORAGE_FREE(jcol) AS Free
     -> FROM jtable;
    +------------------------------------------------+------+------+
    | jcol                                           | Size | Free |
    +------------------------------------------------+------+------+
    | {"a": 4.55, "b": "wxyz", "c": "[true, false]"} |   56 |    0 |
    +------------------------------------------------+------+------+
    1 row in set (0.00 sec)
    ```

    无法对 JSON 用户变量进行部分更新。这意味着此函数始终显示当前用于存储用户变量中 JSON 文档的空间：

    ```sql
    mysql> SET @j = '[100, "sakila", [1, 3, 5], 425.05]';
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @j, JSON_STORAGE_SIZE(@j) AS Size;
    +------------------------------------+------+
    | @j                                 | Size |
    +------------------------------------+------+
    | [100, "sakila", [1, 3, 5], 425.05] |   45 |
    +------------------------------------+------+
    1 row in set (0.00 sec)

    mysql> SET @j = JSON_SET(@j, '$[1]', "json");
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @j, JSON_STORAGE_SIZE(@j) AS Size;
    +----------------------------------+------+
    | @j                               | Size |
    +----------------------------------+------+
    | [100, "json", [1, 3, 5], 425.05] |   43 |
    +----------------------------------+------+
    1 row in set (0.00 sec)

    mysql> SET @j = JSON_SET(@j, '$[2][0]', JSON_ARRAY(10, 20, 30));
    Query OK, 0 rows affected (0.00 sec)

    mysql> SELECT @j, JSON_STORAGE_SIZE(@j) AS Size;
    +---------------------------------------------+------+
    | @j                                          | Size |
    +---------------------------------------------+------+
    | [100, "json", [[10, 20, 30], 3, 5], 425.05] |   56 |
    +---------------------------------------------+------+
    1 row in set (0.00 sec)
    ```

    对于 JSON 文字，此函数始终返回当前使用的存储空间：

    ```sql
    mysql> SELECT
     ->     JSON_STORAGE_SIZE('[100, "sakila", [1, 3, 5], 425.05]') AS A,
     ->     JSON_STORAGE_SIZE('{"a": 1000, "b": "a", "c": "[1, 3, 5, 7]"}') AS B,
     ->     JSON_STORAGE_SIZE('{"a": 1000, "b": "wxyz", "c": "[1, 3, 5, 7]"}') AS C,
     ->     JSON_STORAGE_SIZE('[100, "json", [[10, 20, 30], 3, 5], 425.05]') AS D;
    +----+----+----+----+
    | A  | B  | C  | D  |
    +----+----+----+----+
    | 45 | 44 | 47 | 56 |
    +----+----+----+----+
    1 row in set (0.00 sec)
    ```
