> 原文：[`dev.mysql.com/doc/refman/8.0/en/range-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/range-optimization.html)

#### 10.2.1.2 范围优化

`range` 访问方法使用单个索引检索包含在一个或多个索引值区间内的表行子集。它可用于单部分或多部分索引。以下部分描述了优化器在何种条件下使用范围访问。

+   单部分索引的范围访问方法

+   多部分索引的范围访问方法

+   多值比较的等值范围优化

+   跳过扫描范围访问方法

+   行构造表达式的范围优化

+   限制范围优化的内存使用

##### 单部分索引的范围访问方法

对于单部分索引，索引值区间可以通过`WHERE`子句中对应的条件方便地表示，表示为范围条件而不是“区间”。

单部分索引的范围条件定义如下：

+   对于`BTREE`和`HASH`索引，当使用`=`、`<=>`、`IN()`、`IS NULL`或`IS NOT NULL`运算符进行键部分与常量值的比较时，为范围条件。

+   此外，对于`BTREE`索引，当使用`>`、`<`、`>=`、`<=`、`BETWEEN`、`!=`或`<>`运算符进行键部分与常量值的比较，或者对于`LIKE`比较，如果`LIKE`的参数是不以通配符字符开头的常量字符串，则为范围条件。

+   对于所有索引类型，多个范围条件与`OR`或`AND`组合形成一个范围条件。

在前述描述中，“常量值”指以下之一：

+   来自查询字符串的一个常量

+   来自相同连接的`const`或`system`表的列

+   一个无关子查询的结果

+   由前述类型的子表达式完全组成的任何表达式

以下是一些在`WHERE`子句中具有范围条件的查询示例：

```sql
SELECT * FROM t1
  WHERE *key_col* > 1
  AND *key_col* < 10;

SELECT * FROM t1
  WHERE *key_col* = 1
  OR *key_col* IN (15,18,20);

SELECT * FROM t1
  WHERE *key_col* LIKE 'ab%'
  OR *key_col* BETWEEN 'bar' AND 'foo';
```

优化器常量传播阶段可能会将一些非常量值转换为常量。

MySQL 尝试从`WHERE`子句中提取每个可能索引的范围条件。在提取过程中，无法用于构建范围条件的条件被丢弃，产生重叠范围的条件被合并，产生空范围的条件被移除。

考虑以下语句，其中`key1`是一个索引列，而`nonkey`不是索引列：

```sql
SELECT * FROM t1 WHERE
  (key1 < 'abc' AND (key1 LIKE 'abcde%' OR key1 LIKE '%b')) OR
  (key1 < 'bar' AND nonkey = 4) OR
  (key1 < 'uux' AND key1 > 'z');
```

对`key1`的提取过程如下：

1.  从原始`WHERE`子句开始：

    ```sql
    (key1 < 'abc' AND (key1 LIKE 'abcde%' OR key1 LIKE '%b')) OR
    (key1 < 'bar' AND nonkey = 4) OR
    (key1 < 'uux' AND key1 > 'z')
    ```

1.  移除`nonkey = 4`和`key1 LIKE '%b'`，因为它们不能用于范围扫描。正确的做法是用`TRUE`替换它们，这样在进行范围扫描时不会错过任何匹配的行。将它们替换为`TRUE`得到：

    ```sql
    (key1 < 'abc' AND (key1 LIKE 'abcde%' OR TRUE)) OR
    (key1 < 'bar' AND TRUE) OR
    (key1 < 'uux' AND key1 > 'z')
    ```

1.  合并始终为真或假的条件：

    +   `(key1 LIKE 'abcde%' OR TRUE)`始终为真

    +   `(key1 < 'uux' AND key1 > 'z')`始终为假

    将这些条件替换为常量得到：

    ```sql
    (key1 < 'abc' AND TRUE) OR (key1 < 'bar' AND TRUE) OR (FALSE)
    ```

    移除不必要的`TRUE`和`FALSE`常量得到：

    ```sql
    (key1 < 'abc') OR (key1 < 'bar')
    ```

1.  将重叠的区间合并为一个得到用于范围扫描的最终条件：

    ```sql
    (key1 < 'bar')
    ```

一般来说（并且如前面的示例所示），用于范围扫描的条件比`WHERE`子句要宽松。MySQL 执行额外检查以过滤出满足范围条件但不满足完整`WHERE`子句的行。

范围条件提取算法可以处理任意深度的嵌套`AND`/`OR`结构，并且其输出不依赖于条件在`WHERE`子句中出现的顺序。

MySQL 不支持合并多个范围用于空间索引的`range`访问方法。为了解决这个限制，可以使用具有相同`SELECT`语句的`UNION`，只是将每个空间谓词放在不同的`SELECT`中。

##### 用于多部分索引的范围访问方法

多部分索引上的范围条件是单部分索引的范围条件的扩展。多部分索引上的范围条件将索引行限制在一个或多个键元组区间内。键元组区间是在索引上使用的键元组集合上定义的，使用索引的排序。

例如，考虑一个定义为`key1(*`key_part1`*, *`key_part2`*, *`key_part3`*)`的多部分索引，以及按关键顺序列出的以下一组关键元组：

```sql
*key_part1*  *key_part2*  *key_part3*
  NULL       1          'abc'
  NULL       1          'xyz'
  NULL       2          'foo'
   1         1          'abc'
   1         1          'xyz'
   1         2          'abc'
   2         1          'aaa'
```

条件`*`key_part1`* = 1`定义了这个区间：

```sql
(1,-inf,-inf) <= (*key_part1*,*key_part2*,*key_part3*) < (1,+inf,+inf)
```

该区间涵盖了前述数据集中的第 4、5 和 6 个元组，并可被范围访问方法使用。

相比之下，条件`*`key_part3`* = 'abc'`并未定义单个区间，因此无法被范围访问方法使用。

以下描述更详细地说明了多部分索引的范围条件如何工作。

+   对于`HASH`索引，每个包含相同值的区间都可以使用。这意味着区间只能针对以下形式的条件生成：

    ```sql
     *key_part1* *cmp* *const1*
    AND *key_part2* *cmp* *const2*
    AND ...
    AND *key_partN* *cmp* *constN*;
    ```

    在这里，*`const1`*、*`const2`*、…是常量，*`cmp`*是`=`、`<=>`或`IS NULL`比较运算符之一，条件涵盖了所有索引部分。（也就是说，有*`N`*个条件，每个条件对应一个*`N`*部分索引的部分。）例如，以下是三部分`HASH`索引的范围条件：

    ```sql
    *key_part1* = 1 AND *key_part2* IS NULL AND *key_part3* = 'foo'
    ```

    有关什么被视为常量的定义，请参阅单部分索引的范围访问方法。

+   对于`BTREE`索引，一个区间可能适用于与`AND`组合的条件，其中每个条件将一个关键部分与常量值使用`=`、`<=>`、`IS NULL`、`>`、`<`、`>=`、`<=`、`!=`、`<>`、`BETWEEN`或`LIKE '*`pattern`*'`进行比较（其中`'*`pattern`*'`不以通配符开头）。只要能够确定包含所有符合条件的行的单个关键元组（或者如果使用`<>`或`!=`则为两个区间），就可以使用区间。

    优化器尝试使用额外的键部分来确定区间，只要比较运算符是`=`、`<=>`或`IS NULL`。如果运算符是`>`、`<`、`>=`、`<=`、`!=`、`<>`、`BETWEEN`或`LIKE`，优化器会使用它，但不考虑更多的键部分。对于以下表达式，优化器从第一个比较中使用`=`。它还从第二个比较中使用`>=`，但不考虑更多的键部分，并且不使用第三个比较来构建区间：

    ```sql
    *key_part1* = 'foo' AND *key_part2* >= 10 AND *key_part3* > 10
    ```

    单个区间是：

    ```sql
    ('foo',10,-inf) < (*key_part1*,*key_part2*,*key_part3*) < ('foo',+inf,+inf)
    ```

    创建的区间可能包含比初始条件更多的行。例如，前面的区间包括值`('foo', 11, 0)`，这不符合原始条件。

+   如果涵盖区间内的行集的条件与`OR`组合，它们形成一个涵盖其区间并集内的行集的条件。如果条件与`AND`组合，它们形成一个涵盖其区间交集内的行集的条件。例如，对于两部分索引上的此条件：

    ```sql
    (*key_part1* = 1 AND *key_part2* < 2) OR (*key_part1* > 5)
    ```

    区间是：

    ```sql
    (1,-inf) < (*key_part1*,*key_part2*) < (1,2)
    (5,-inf) < (*key_part1*,*key_part2*)
    ```

    在此示例中，第一行的区间使用一个键部分作为左边界，两个键部分作为右边界。第二行的区间仅使用一个键部分。`EXPLAIN`输出中的`key_len`列指示使用的键前缀的最大长度。

    在某些情况下，`key_len`可能表明使用了一个键部分，但这可能不是您期望的。假设*`key_part1`*和*`key_part2`*可以是`NULL`。然后，`key_len`列显示以下条件的两个键部分长度：

    ```sql
    *key_part1* >= 1 AND *key_part2* < 2
    ```

    但实际上，条件被转换为这样：

    ```sql
    *key_part1* >= 1 AND *key_part2* IS NOT NULL
    ```

有关如何对单部分索引上的范围条件执行优化以组合或消除区间的描述，请参见单部分索引的范围访问方法。类似的步骤也适用于多部分索引上的范围条件。

##### 多值比较的等值范围优化

考虑以下表达式，其中*`col_name`*是一个索引列：

```sql
*col_name* IN(*val1*, ..., *valN*)
*col_name* = *val1* OR ... OR *col_name* = *valN*
```

如果 *`col_name`* 等于多个值中的任何一个，则每个表达式为真。这些比较是相等范围比较（其中“范围”是单个值）。优化器估计读取符合相等范围比较条件的行的成本如下：

+   如果 *`col_name`* 上有唯一索引，则每个范围的行估计值为 1，因为最多只有一行可以具有给定值。

+   否则，*`col_name`* 上的任何索引都是非唯一的，优化器可以通过对索引或索引统计数据的深入来估计每个范围的行数。

使用索引深入，优化器在每个范围的两端进行深入，并将范围内的行数作为估计值。例如，表达式 `*`col_name`* IN (10, 20, 30)` 有三个相等范围，优化器每个范围进行两次深入以生成行估计值。每对深入提供给定值的行数的估计值。

索引深入提供准确的行估计值，但随着表达式中比较值的增加，优化器生成行估计值的时间也会增加。使用索引统计数据比索引深入的准确性低，但允许更快地为大型值列表进行行估计。

`eq_range_index_dive_limit` 系统变量允许您配置优化器在何时从一种行估计策略切换到另一种。要允许使用索引深入进行多达 *`N`* 个相等范围的比较，请将 `eq_range_index_dive_limit` 设置为 *`N`* + 1\. 要禁用统计数据的使用并始终使用索引深入，无论 *`N`* 如何，请将 `eq_range_index_dive_limit` 设置为 0。

要更新表索引统计数据以获得最佳估计值，请使用 `ANALYZE TABLE`。

在 MySQL 8.0 之前，除了使用 `eq_range_index_dive_limit` 系统变量外，没有跳过使用索引深入来估计索引有用性的方法。在 MySQL 8.0 中，对于满足以下所有条件的查询，可以跳过索引深入估计：

+   查询针对单个表，而不是多个表的连接。

+   存在单索引 `FORCE INDEX` 索引提示。这样做的想法是，如果强制使用索引，则执行对索引的深入所带来的额外开销没有任何好处。

+   索引是非唯一的，不是 `FULLTEXT` 索引。

+   没有子查询。

+   没有 `DISTINCT`、`GROUP BY` 或 `ORDER BY` 子句。

对于 `EXPLAIN FOR CONNECTION`，如果跳过索引深入，则输出如下更改：

+   对于传统输出，`rows` 和 `filtered` 值为 `NULL`。

+   对于 JSON 输出，`rows_examined_per_scan`和`rows_produced_per_join`不会出现，`skip_index_dive_due_to_force`为`true`，成本计算不准确。

没���`FOR CONNECTION`，当索引潜水被跳过时，`EXPLAIN`输出不会改变。

执行一个查询，其中索引潜水被跳过后，信息模式`OPTIMIZER_TRACE`表中的相应行包含一个`index_dives_for_range_access`值为`skipped_due_to_force_index`。

##### 跳过扫描范围访问方法

考虑以下情景：

```sql
CREATE TABLE t1 (f1 INT NOT NULL, f2 INT NOT NULL, PRIMARY KEY(f1, f2));
INSERT INTO t1 VALUES
  (1,1), (1,2), (1,3), (1,4), (1,5),
  (2,1), (2,2), (2,3), (2,4), (2,5);
INSERT INTO t1 SELECT f1, f2 + 5 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 10 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 20 FROM t1;
INSERT INTO t1 SELECT f1, f2 + 40 FROM t1;
ANALYZE TABLE t1;

EXPLAIN SELECT f1, f2 FROM t1 WHERE f2 > 40;
```

要执行这个查询，MySQL 可以选择一个索引扫描来获取所有行（索引包括所有要选择的列），然后应用`WHERE`子句中的`f2 > 40`条件来生成最终结果集。

范围扫描比完整索引扫描更有效，但在这种情况下无法使用，因为对第一个索引列`f1`没有条件。然而，从 MySQL 8.0.13 开始，优化器可以执行多个范围扫描，每个值`f1`一个，使用一种称为 Skip Scan 的方法，类似于 Loose Index Scan（参见 Section 10.2.1.17, “GROUP BY Optimization”）：

1.  在第一个索引部分`f1`（索引前缀）的不同值之间跳过。

1.  对剩余索引部分上的`f2 > 40`条件的每个不同前缀值执行子范围扫描。

对于之前显示的数据集，算法的操作如下：

1.  获取第一个关键部分的第一个不同值（`f1 = 1`）。

1.  基于第一个和第二个关键部分构建范围（`f1 = 1 AND f2 > 40`）。

1.  执行一个范围扫描。

1.  获取第一个关键部分的下一个不同值（`f1 = 2`）。

1.  基于第一个和第二个关键部分构建范围（`f1 = 2 AND f2 > 40`）。

1.  执行一个范围扫描。

使用这种策略可以减少访问的行数，因为 MySQL 跳过了不符合每个构建范围的行。这种 Skip Scan 访问方法适用于以下条件：

+   表 T 至少有一个复合索引，其关键部分形式为([A_1, ..., A_*`k`*,] B_1, ..., B_*`m`*, C [, D_1, ..., D_*`n`*])。关键部分 A 和 D 可能为空，但 B 和 C 必须非空。

+   查询仅引用一个表。

+   查询不使用`GROUP BY`或`DISTINCT`。

+   查询仅引用索引中的列。

+   A_1, ..., A_*`k`*上的谓词必须是等式谓词，且它们必须是常量。这包括`IN()`运算符。

+   查询必须是一个连接查询；也就是说，是`OR`条件的`AND`：`(*`cond1`*(*`key_part1`*) OR *`cond2`*(*`key_part1`*)) AND (*`cond1`*(*`key_part2`*) OR ...) AND ...`

+   必须对 C 列有一个范围条件。

+   允许对 D 列的条件。D 列上的条件必须与 C 列上的范围条件一起。

在`EXPLAIN`输出中指示使用跳过扫描如下：

+   在`Extra`列中使用`Using index for skip scan`表示使用了宽松索引跳过扫描访问方法。

+   如果索引可以用于跳过扫描，索引应该在`possible_keys`列中可见。

在优化器跟踪输出中，使用`"skip scan"`元素指示使用跳过扫描：

```sql
"skip_scan_range": {
  "type": "skip_scan",
  "index": *index_used_for_skip_scan*,
  "key_parts_used_for_access": [*key_parts_used_for_access*],
  "range": [*range*]
}
```

您还可能会看到一个`"best_skip_scan_summary"`元素。如果跳过扫描被选择为最佳范围访问变体，则会写入`"chosen_range_access_summary"`。如果跳过扫描被选择为整体最佳访问方法，则会出现`"best_access_path"`元素。

使用跳过扫描取决于`optimizer_switch`系统变量的`skip_scan`标志的值。请参见 Section 10.9.2, “Switchable Optimizations”。默认情况下，此标志为`on`。要禁用它，请将`skip_scan`设置为`off`。

除了使用`optimizer_switch`系统变量来控制会话范围内优化器使用跳过扫描外，MySQL 还支持优化器提示以影响每个语句的优化器。请参见 Section 10.9.3, “Optimizer Hints”。

##### 行构造器表达式的范围优化

优化器能够将范围扫描访问方法应用于此类查询：

```sql
SELECT ... FROM t1 WHERE ( col_1, col_2 ) IN (( 'a', 'b' ), ( 'c', 'd' ));
```

以前，为了使用范围扫描，必须将查询编写为：

```sql
SELECT ... FROM t1 WHERE ( col_1 = 'a' AND col_2 = 'b' )
OR ( col_1 = 'c' AND col_2 = 'd' );
```

为了使优化器使用范围扫描，查询必须满足以下条件：

+   仅使用`IN()`谓词，不使用`NOT IN()`。

+   在`IN()`谓词的左侧，行构造器仅包含列引用。

+   在`IN()`谓词的右侧，行构造器仅包含运行时常量，这些常量可以是文字或在执行期间绑定为常量的本地列引用。

+   在`IN()`谓词的右侧，存在多个行构造器。

有关优化器和行构造器的更多信息，请参见 Section 10.2.1.22, “Row Constructor Expression Optimization”

##### 限制范围优化的内存使用

要控制范围优化器可用的内存，请使用`range_optimizer_max_mem_size`系统变量：

+   值为 0 表示“没有限制”。

+   当值大于 0 时，优化器会跟踪考虑范围访问方法时消耗的内存。如果即将超过指定限制，范围访问方法将被放弃，而考虑其他方法，包括全表扫描。这可能不太理想。如果发生这种情况，将出现以下警告（其中*`N`*是当前`range_optimizer_max_mem_size`值）：

    ```sql
    Warning    3170    Memory capacity of *N* bytes for
                       'range_optimizer_max_mem_size' exceeded. Range
                       optimization was not done for this query.
    ```

+   对于`UPDATE`和`DELETE`语句，如果优化器回退到全表扫描，并且启用了`sql_safe_updates`系统变量，则会发生错误而不是警告，因为实际上没有使用键来确定要修改哪些行。有关更多信息，请参见使用安全更新模式（--safe-updates）")。

对于超出可用范围优化内存并且优化器回退到不太理想计划的单个查询，增加`range_optimizer_max_mem_size`值可能会提高性能。

要估算处理范围表达式所需的内存量，请使用以下准则：

+   对于像下面这样的简单查询，其中有一个候选键用于范围访问方法，每个与`OR`结合的谓词大约使用 230 字节：

    ```sql
    SELECT COUNT(*) FROM t
    WHERE a=1 OR a=2 OR a=3 OR .. . a=*N*;
    ```

+   类似地，对于像下面这样的查询，每个与`AND`结合的谓词大约使用 125 字节：

    ```sql
    SELECT COUNT(*) FROM t
    WHERE a=1 AND b=1 AND c=1 ... *N*;
    ```

+   对于带有`IN()`谓词的查询：

    ```sql
    SELECT COUNT(*) FROM t
    WHERE a IN (1,2, ..., *M*) AND b IN (1,2, ..., *N*);
    ```

    在`IN()`列表中的每个文字值都算作与`OR`结合的谓词。如果有两个`IN()`列表，则与`OR`结合的谓词数量是每个列表中文字值数量的乘积。因此，在前述情况下，与`OR`结合的谓词数量为*`M`* × *`N`*。
