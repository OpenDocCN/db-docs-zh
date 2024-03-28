# 10.3.11 优化器对生成列索引的使用

> 原文：[`dev.mysql.com/doc/refman/8.0/en/generated-column-index-optimizations.html`](https://dev.mysql.com/doc/refman/8.0/en/generated-column-index-optimizations.html)

MySQL 支持对生成列创建索引。例如：

```sql
CREATE TABLE t1 (f1 INT, gc INT AS (f1 + 1) STORED, INDEX (gc));
```

生成列`gc`被定义为表达式`f1 + 1`。该列也被索引，优化器在执行计划构建过程中可以考虑该索引。在以下查询中，`WHERE`子句引用了`gc`，优化器会考虑该列上的索引是否产生更有效的计划：

```sql
SELECT * FROM t1 WHERE gc > 9;
```

优化器可以使用生成列上的索引生成执行计划，即使查询中没有直接按名称引用这些列。如果`WHERE`、`ORDER BY`或`GROUP BY`子句引用与某个索引生成列的定义匹配的表达式，则会发生这种情况。以下查询并不直接引用`gc`，但确实使用与`gc`定义匹配的表达式：

```sql
SELECT * FROM t1 WHERE f1 + 1 > 9;
```

优化器识别到表达式`f1 + 1`与`gc`的定义匹配，并且`gc`被索引，因此在执行计划构建过程中考虑了该索引。您可以使用`EXPLAIN`查看这一点：

```sql
mysql> EXPLAIN SELECT * FROM t1 WHERE f1 + 1 > 9\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: range
possible_keys: gc
          key: gc
      key_len: 5
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using index condition
```

实际上，优化器已经用与表达式匹配的生成列名称替换了表达式`f1 + 1`。这也可以在由`SHOW WARNINGS`显示的扩展`EXPLAIN`信息中重新编写的查询中看到：

```sql
mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`f1` AS `f1`,`test`.`t1`.`gc`
         AS `gc` from `test`.`t1` where (`test`.`t1`.`gc` > 9)
```

优化器对生成列索引的使用受到以下限制和条件的约束：

+   要使查询表达式与生成列定义匹配，表达式必须完全相同，并且必须具有相同的结果类型。例如，如果生成列表达式是`f1 + 1`，则如果查询使用`1 + f1`，或者如果将`f1 + 1`（整数表达式）与字符串进行比较，则优化器不会识别匹配。

+   优化适用于这些运算符：`=`, `<`, `<=`, `>`, `>=`, `BETWEEN`和`IN()`。

    对于除了`BETWEEN`和`IN()`之外的运算符，任一操作数都可以被匹配的生成列替换。对于`BETWEEN`和`IN()`，只有第一个参数可以被匹配的生成列替换，其他参数必须具有相同的结果类型。目前不支持涉及 JSON 值的比较的`BETWEEN`和`IN()`。

+   生成列必须定义为包含至少一个函数调用或前一项提到的运算符之一的表达式。表达式不能仅由对另一列的简单引用组成。例如，`gc INT AS (f1) STORED` 仅由列引用组成，因此对 `gc` 的索引不会被考虑。

+   对于将字符串与从返回带引号的字符串的 JSON 函数计算值的索引生成列进行比较，需要在列定义中使用`JSON_UNQUOTE()`来去除函数值中的额外引号。（对于直接将字符串与函数结果进行比较，JSON 比较器会处理引号的移除，但这不适用于索引查找。）例如，不要像这样编写列定义：

    ```sql
    doc_name TEXT AS (JSON_EXTRACT(jdoc, '$.name')) STORED
    ```

    要像这样编写：

    ```sql
    doc_name TEXT AS (JSON_UNQUOTE(JSON_EXTRACT(jdoc, '$.name'))) STORED
    ```

    使用后一种定义，优化器可以检测到这两个比较的匹配：

    ```sql
    ... WHERE JSON_EXTRACT(jdoc, '$.name') = '*some_string*' ...
    ... WHERE JSON_UNQUOTE(JSON_EXTRACT(jdoc, '$.name')) = '*some_string*' ...
    ```

    在列定义中没有`JSON_UNQUOTE()`，优化器只会检测到这些比较中的第一个匹配。

+   如果优化器选择了错误的索引，可以使用索引提示来禁用它，并强制优化器做出不同的选择。
