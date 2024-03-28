# 14.17.6 JSON 表函数

> 译文：[`dev.mysql.com/doc/refman/8.0/en/json-table-functions.html`](https://dev.mysql.com/doc/refman/8.0/en/json-table-functions.html)

本节包含将 JSON 数据转换为表格数据的 JSON 函数的信息。MySQL 8.0 支持一种名为`JSON_TABLE()`的函数。

[`JSON_TABLE(*`expr`*, *`path`* COLUMNS (*`column_list`*) [AS] *`alias`*)`](json-table-functions.html#function_json-table)

从 JSON 文档中提取数据，并将其作为具有指定列的关系表返回。此函数的完整语法如下所示：

```sql
JSON_TABLE(
    *expr*,
    *path* COLUMNS (*column_list*)
)   [AS] *alias*

*column_list*:
    *column*[, *column*][, ...]

*column*:
    *name* FOR ORDINALITY
    |  *name* *type* PATH *string path* [*on_empty*] [*on_error*]
    |  *name* *type* EXISTS PATH *string path*
    |  NESTED [PATH] *path* COLUMNS (*column_list*)

*on_empty*:
    {NULL | DEFAULT *json_string* | ERROR} ON EMPTY

*on_error*:
    {NULL | DEFAULT *json_string* | ERROR} ON ERROR
```

*`expr`*：这是返回 JSON 数据的表达式。这可以是一个常量（`'{"a":1}'`），一个列（`t1.json_data`，在`FROM`子句中在`JSON_TABLE()`之前指定了表`t1`），或一个函数调用（`JSON_EXTRACT(t1.json_data,'$.post.comments')`）。

*`path`*：一个应用于数据源的 JSON 路径表达式。我们将匹配路径的 JSON 值称为*行源*；这用于生成关系数据的一行。`COLUMNS`子句评估行源，在行源中找到特定的 JSON 值，并将这些 JSON 值作为关系数据行的各个列中的 SQL 值返回。

*`alias`*是必需的。适用于表别名的通常规则（参见第 11.2 节，“模式对象名称”）。

从 MySQL 8.0.27 开始，此函数以不区分大小写的方式比较列名。

`JSON_TABLE()`支持四种列类型，描述如下：

1.  `*`name`* FOR ORDINALITY`：此类型在`COLUMNS`子句中枚举行；名为*`name`*的列是一个计数器，其类型为`UNSIGNED INT`，初始值为 1。这相当于在`CREATE TABLE`语句中指定列为`AUTO_INCREMENT`，并可用于区分由`NESTED [PATH]`子句生成的多行中具有相同值的父行。

1.  `*`name`* *`type`* PATH *`string_path`* [*`on_empty`*] [*`on_error`*]`：此类型的列用于提取由*`string_path`*指定的值。*`type`*是 MySQL 标量数据类型（即，不能是对象或数组）。`JSON_TABLE()`将数据提取为 JSON，然后将其强制转换为列类型，使用 MySQL 中适用于 JSON 数据的常规自动类型转换。缺少值会触发*`on_empty`*子句。保存对象或数组会触发可选的*`on_error`*子句；当在将保存为 JSON 的值从 JSON 转换为表列时发生错误时，例如尝试将字符串`'asd'`保存到整数列时，也会发生这种情况。

1.  `*`name`* *`type`* EXISTS PATH *`path`*`：如果指定的 *`path`* 位置存在任何数据，则此列返回 1，否则返回 0。*`type`* 可以是任何有效的 MySQL 数据类型，但通常应指定为某种类型的 `INT` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT")。

1.  `NESTED [PATH] *`path`* COLUMNS (*`column_list`*)`：这将 JSON 数据中嵌套的对象或数组展开为单行，并包括来自父对象或数组的 JSON 值。使用多个 `PATH` 选项允许将多个嵌套级别的 JSON 值投影到单行中。

    *`path`* 相对于 `JSON_TABLE()` 的父路径行路径，或者在嵌套路径的情况下，相对于父 `NESTED [PATH]` 子句的路径。

*`on empty`*，如果指定，确定 `JSON_TABLE()` 在数据缺失时（取决于类型）的操作。当 `NESTED PATH` 子句中的列没有匹配项并且为其生成了一个 `NULL` 补充行时，此子句也会触发。*`on empty`* 可以采用以下值：

+   `NULL ON EMPTY`：列被设置为 `NULL`；这是默认行为。

+   `DEFAULT *`json_string`* ON EMPTY`：提供的 *`json_string`* 被解析为 JSON，只要它是有效的，并且存储在缺失值的位置。列类型规则也适用于默认值。

+   `ERROR ON EMPTY`：抛出错误。

如果使用 *`on_error`*，则可以采用以下值，并显示相应的结果如下：

+   `NULL ON ERROR`：列被设置为 `NULL`；这是默认行为。

+   `DEFAULT *`json string`* ON ERROR`：*`json_string`* 被解析为 JSON（前提是它是有效的），并存储在对象或数组的位置。 

+   `ERROR ON ERROR`：抛出错误。

在 MySQL 8.0.20 之前，如果发生类型转换错误，并且指定或暗示了 `NULL ON ERROR` 或 `DEFAULT ... ON ERROR`，则会发出警告。在 MySQL 8.0.20 及更高版本中，不再会出现这种情况。（Bug #30628330）

以前，可以以任何顺序指定 `ON EMPTY` 和 `ON ERROR` 子句。这与 SQL 标准相悖，后者规定，如果指定了 `ON EMPTY`，则必须在任何 `ON ERROR` 子句之前。因此，从 MySQL 8.0.20 开始，指定 `ON ERROR` 在 `ON EMPTY` 之前已被弃用；尝试这样做会导致服务器发出警告。预计在未来的 MySQL 版本中将删除对非标准语法的支持。

当将一个值保存到列中时被截断，例如将 3.14159 保存在 `DECIMAL(10,1)` - DECIMAL, NUMERIC") 列中，将发出警告，与任何 `ON ERROR` 选项无关。当在单个语句中截断多个值时，只会发出一次警告。

在 MySQL 8.0.21 之前，当传递给此函数的表达式和路径解析为 JSON null 时，`JSON_TABLE()`会引发错误。在 MySQL 8.0.21 及更高版本中，在这种情况下返回 SQL `NULL`，符合 SQL 标准，如下所示（Bug #31345503，Bug #99557）：

```sql
mysql> SELECT *
 ->   FROM
 ->     JSON_TABLE(
 ->       '[ {"c1": null} ]',
 ->       '$[*]' COLUMNS( c1 INT PATH '$.c1' ERROR ON ERROR )
 ->     ) as jt;
+------+
| c1   |
+------+
| NULL |
+------+
1 row in set (0.00 sec)
```

以下查询演示了`ON EMPTY`和`ON ERROR`的使用。对应于`{"b":1}`的行在路径`"$.a"`上为空，并尝试将`[1,2]`保存为标量会产生错误；这些行在输出中被突出显示。

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[{"a":"3"},{"a":2},{"b":1},{"a":0},{"a":[1,2]}]',
 ->     "$[*]"
 ->     COLUMNS(
 ->       rowid FOR ORDINALITY,
 ->       ac VARCHAR(100) PATH "$.a" DEFAULT '111' ON EMPTY DEFAULT '999' ON ERROR,
 ->       aj JSON PATH "$.a" DEFAULT '{"x": 333}' ON EMPTY,
 ->       bx INT EXISTS PATH "$.b"
 ->     )
 ->   ) AS tt;

+-------+------+------------+------+
| rowid | ac   | aj         | bx   |
+-------+------+------------+------+
|     1 | 3    | "3"        |    0 |
|     2 | 2    | 2          |    0 |
*|     3 | 111  | {"x": 333} |    1 |*
|     4 | 0    | 0          |    0 |
*|     5 | 999  | [1, 2]     |    0 |*
+-------+------+------------+------+
5 rows in set (0.00 sec)
```

列名受表列名规则和限制的约束。请参见第 11.2 节，“模式对象名称”。

所有 JSON 和 JSON 路径表达式都会被检查其有效性；任何一种类型的无效表达式都会导致错误。

在`COLUMNS`关键字之前的*`path`*的每个匹配项映射到结果表中的一个单独行。例如，以下查询给出了这里显示的结果：

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[{"x":2,"y":"8"},{"x":"3","y":"7"},{"x":"4","y":6}]',
 ->     "$[*]" COLUMNS(
 ->       xval VARCHAR(100) PATH "$.x",
 ->       yval VARCHAR(100) PATH "$.y"
 ->     )
 ->   ) AS  jt1;

+------+------+
| xval | yval |
+------+------+
| 2    | 8    |
| 3    | 7    |
| 4    | 6    |
+------+------+
```

表达式`"$[*]"`匹配数组的每个元素。您可以通过修改路径来过滤结果中的行。例如，使用`"$[1]"`将提取限制为用作源的 JSON 数组的第二个元素，如下所示：

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[{"x":2,"y":"8"},{"x":"3","y":"7"},{"x":"4","y":6}]',
 ->     "$[1]" COLUMNS(
 ->       xval VARCHAR(100) PATH "$.x",
 ->       yval VARCHAR(100) PATH "$.y"
 ->     )
 ->   ) AS  jt1;

+------+------+
| xval | yval |
+------+------+
| 3    | 7    |
+------+------+
```

在列定义中，`"$"`将整个匹配项传递给列；`"$.x"`和`"$.y"`分别仅传递与该匹配项中的键`x`和`y`对应的值。有关更多信息，请参见 JSON 路径语法。

`NESTED PATH`（或简称`NESTED`；`PATH`是可选的）为`COLUMNS`子句中的每个匹配项生成一组记录。如果没有匹配项，则嵌套路径的所有列都设置为`NULL`。这实现了顶层子句和`NESTED [PATH]`之间的外连接。可以通过在`WHERE`子句中应用适当条件来模拟内连接，如下所示：

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[ {"a": 1, "b": [11,111]}, {"a": 2, "b": [22,222]}, {"a":3}]',
 ->     '$[*]' COLUMNS(
 ->             a INT PATH '$.a',
 ->             NESTED PATH '$.b[*]' COLUMNS (b INT PATH '$')
 ->            )
 ->    ) AS jt
 -> WHERE b IS NOT NULL;

+------+------+
| a    | b    |
+------+------+
|    1 |   11 |
|    1 |  111 |
|    2 |   22 |
|    2 |  222 |
+------+------+
```

兄弟嵌套路径——即在同一`COLUMNS`子句中的两个或多个`NESTED [PATH]`实例——依次处理，一次处理一个。当一个嵌套路径生成记录时，任何兄弟嵌套路径表达式的列都设置为`NULL`。这意味着在单个包含`COLUMNS`子句中的单个匹配项的总记录数是由`NESTED [PATH]`修饰符生成的所有记录的总和，而不是乘积，如下所示：

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[{"a": 1, "b": [11,111]}, {"a": 2, "b": [22,222]}]',
 ->     '$[*]' COLUMNS(
 ->         a INT PATH '$.a',
 ->         NESTED PATH '$.b[*]' COLUMNS (b1 INT PATH '$'),
 ->         NESTED PATH '$.b[*]' COLUMNS (b2 INT PATH '$')
 ->     )
 -> ) AS jt;

+------+------+------+
| a    | b1   | b2   |
+------+------+------+
|    1 |   11 | NULL |
|    1 |  111 | NULL |
|    1 | NULL |   11 |
|    1 | NULL |  111 |
|    2 |   22 | NULL |
|    2 |  222 | NULL |
|    2 | NULL |   22 |
|    2 | NULL |  222 |
+------+------+------+
```

`FOR ORDINALITY`列枚举由`COLUMNS`子句生成的记录，并可用于区分嵌套路径的父记录，特别是如果父记录中的值相同，则可以看到：

```sql
mysql> SELECT *
 -> FROM
 ->   JSON_TABLE(
 ->     '[{"a": "a_val",
    '>       "b": [{"c": "c_val", "l": [1,2]}]},
    '>     {"a": "a_val",
    '>       "b": [{"c": "c_val","l": [11]}, {"c": "c_val", "l": [22]}]}]',
    ->     '$[*]' COLUMNS(
    ->       top_ord FOR ORDINALITY,
    ->       apath VARCHAR(10) PATH '$.a',
    ->       NESTED PATH '$.b[*]' COLUMNS (
    ->         bpath VARCHAR(10) PATH '$.c',
    ->         ord FOR ORDINALITY,
    ->         NESTED PATH '$.l[*]' COLUMNS (lpath varchar(10) PATH '$')
 ->         )
 ->     )
 -> ) as jt;

+---------+---------+---------+------+-------+
| top_ord | apath   | bpath   | ord  | lpath |
+---------+---------+---------+------+-------+
|       1 |  a_val  |  c_val  |    1 | 1     |
|       1 |  a_val  |  c_val  |    1 | 2     |
|       2 |  a_val  |  c_val  |    1 | 11    |
|       2 |  a_val  |  c_val  |    2 | 22    |
+---------+---------+---------+------+-------+
```

源文档包含一个包含两个元素的数组；每个元素产生两行。 `apath` 和 `bpath` 的值在整个结果集中保持不变；这意味着它们不能用来确定 `lpath` 值是来自相同还是不同的父级。 `ord` 列的值与具有 `top_ord` 等于 1 的记录集保持一致，因此这两个值来自单个对象。 剩下的两个值来自不同的对象，因为它们在 `ord` 列中具有不同的值。

通常情况下，您不能在相同的 `FROM` 子句中连接依赖于前面表的列的派生表。 MySQL 根据 SQL 标准对表函数做了一个例外；即使在尚未支持 `LATERAL` 关键字的 MySQL 版本中（8.0.13 及更早版本），这些被视为横向派生表。 在支持 `LATERAL` 的版本中（8.0.14 及更高版本），它是隐式的，并且因此在 `JSON_TABLE()` 之前不允许使用。

假设您已经创建并使用以下语句填充了一个名为 `t1` 的表：

```sql
CREATE TABLE t1 (c1 INT, c2 CHAR(1), c3 JSON);

INSERT INTO t1 () VALUES
	ROW(1, 'z', JSON_OBJECT('a', 23, 'b', 27, 'c', 1)),
	ROW(1, 'y', JSON_OBJECT('a', 44, 'b', 22, 'c', 11)),
	ROW(2, 'x', JSON_OBJECT('b', 1, 'c', 15)),
	ROW(3, 'w', JSON_OBJECT('a', 5, 'b', 6, 'c', 7)),
	ROW(5, 'v', JSON_OBJECT('a', 123, 'c', 1111))
;
```

然后，您可以执行诸如这样的连接，其中 `JSON_TABLE()` 充当派生表，同时引用先前引用表中的列：

```sql
SELECT c1, c2, JSON_EXTRACT(c3, '$.*') 
FROM t1 AS m 
JOIN 
JSON_TABLE(
  m.c3, 
  '$.*' 
  COLUMNS(
    at VARCHAR(10) PATH '$.a' DEFAULT '1' ON EMPTY, 
    bt VARCHAR(10) PATH '$.b' DEFAULT '2' ON EMPTY, 
    ct VARCHAR(10) PATH '$.c' DEFAULT '3' ON EMPTY
  )
) AS tt
ON m.c1 > tt.at;
```

尝试在此查询中使用 `LATERAL` 关键字会引发 `ER_PARSE_ERROR`。
