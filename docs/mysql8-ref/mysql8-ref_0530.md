> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-optimization.html)

#### 10.2.1.17 GROUP BY 优化

最一般的满足`GROUP BY`子句的方法是扫描整个表，并创建一个新的临时表，其中每个组的所有行都是连续的，然后使用这个临时表来发现组并应用聚合函数（如果有的话）。在某些情况下，MySQL 能够比这做得更好，通过使用索引访问来避免创建临时表。

使用索引进行`GROUP BY`的最重要的前提条件是所有`GROUP BY`列引用来自同一索引的属性，并且索引按顺序存储其键（例如`BTREE`索引是这样的，但`HASH`索引不是）。临时表的使用是否可以被索引访问替代还取决于查询中使用了索引的哪些部分，为这些部分指定的条件以及选择的聚合函数。

有两种通过索引访问执行`GROUP BY`查询的方法，如下节所述。第一种方法将分组操作与所有范围谓词（如果有的话）一起应用。第二种方法首先执行范围扫描，然后对结果元组进行分组。

+   松散索引扫描

+   紧凑索引扫描

在某些条件下，即使没有`GROUP BY`，也可以使用松散索引扫描。请参阅跳过扫描范围访问方法。

##### 松散索引扫描

处理`GROUP BY`最有效的方法是直接使用索引检索分组列。通过这种访问方法，MySQL 使用某些索引类型的属性，即键是有序的（例如`BTREE`）。这种属性使得可以在索引中查找组，而无需考虑满足所有`WHERE`条件的所有键。这种访问方法只考虑索引中的一部分键，因此被称为松散索引扫描。当没有`WHERE`子句时，松散索引扫描读取与组数相同的键，这可能比所有键的数量要小得多。如果`WHERE`子句包含范围谓词（请参阅第 10.8.1 节“使用 EXPLAIN 优化查询”中对`range`连接类型的讨论），松散索引扫描查找满足范围条件的每个组的第一个键，然后再次读取最小可能数量的键。这在以下条件下是可能的：

+   查询只涉及单个表。

+   `GROUP BY`仅命名了构成索引最左边前缀的列，没有其他列。（如果查询中有`DISTINCT`子句而不是`GROUP BY`，则所有不同的属性都指向构成索引最左边前缀的列。）例如，如果表`t1`在`(c1,c2,c3)`上有索引，如果查询中有`GROUP BY c1, c2`，则可以应用松散索引扫描。如果查询中有`GROUP BY c2, c3`（列不是最左边前缀）或`GROUP BY c1, c2, c4`（`c4`不在索引中），则不适用。

+   选择列表中使用的唯一聚合函数（如果有）是`MIN()`和`MAX()`，并且它们都引用相同的列。该列必须在索引中，并且必须紧随`GROUP BY`中的列之后。

+   查询中除了`GROUP BY`引用的索引部分外，其他索引部分必须是常量（即，它们必须与常量相等），除了`MIN()`或`MAX()`函数的参数。

+   对于索引中的列，必须对完整列值建立索引，而不仅仅是前缀。例如，对于`c1 VARCHAR(20), INDEX (c1(10))`，索引仅使用`c1`值的前缀，不能用于松散索引扫描。

如果查询适用于松散索引扫描，则`EXPLAIN`输出中的`Extra`列显示`Using index for group-by`。

假设在表`t1(c1,c2,c3,c4)`上有索引`idx(c1,c2,c3)`。可以对以下查询使用松散索引扫描访问方法：

```sql
SELECT c1, c2 FROM t1 GROUP BY c1, c2;
SELECT DISTINCT c1, c2 FROM t1;
SELECT c1, MIN(c2) FROM t1 GROUP BY c1;
SELECT c1, c2 FROM t1 WHERE c1 < *const* GROUP BY c1, c2;
SELECT MAX(c3), MIN(c3), c1, c2 FROM t1 WHERE c2 > *const* GROUP BY c1, c2;
SELECT c2 FROM t1 WHERE c1 < *const* GROUP BY c1, c2;
SELECT c1, c2 FROM t1 WHERE c3 = *const* GROUP BY c1, c2;
```

以下查询不能使用这种快速选择方法执行，原因如下：

+   存在除了`MIN()`或`MAX()`之外的其他聚合函数：

    ```sql
    SELECT c1, SUM(c2) FROM t1 GROUP BY c1;
    ```

+   `GROUP BY`子句中的列不构成索引的最左边前缀：

    ```sql
    SELECT c1, c2 FROM t1 GROUP BY c2, c3;
    ```

+   查询引用了`GROUP BY`部分之后的键的一部分，且没有与常量相等的情况：

    ```sql
    SELECT c1, c3 FROM t1 GROUP BY c1, c2;
    ```

    如果查询包括`WHERE c3 = *`const`*`，则可以使用松散索引扫描。

松散索引扫描访问方法可以应用于选择列表中的其他形式的聚合函数引用，除了已经支持的`MIN()`和`MAX()`引用：

+   支持`AVG(DISTINCT)`、`SUM(DISTINCT)`和`COUNT(DISTINCT)`。`AVG(DISTINCT)`和`SUM(DISTINCT)`接受单个参数。`COUNT(DISTINCT)`可以有多个列参数。

+   查询中不能包含`GROUP BY`或`DISTINCT`子句。

+   仍然适用前面描述的松散索引扫描限制。

假设在表`t1(c1,c2,c3,c4)`上有一个索引`idx(c1,c2,c3)`。松散索引扫描访问方法可用于以下查询：

```sql
SELECT COUNT(DISTINCT c1), SUM(DISTINCT c1) FROM t1;

SELECT COUNT(DISTINCT c1, c2), COUNT(DISTINCT c2, c1) FROM t1;
```

##### 紧凑索引扫描

紧凑索引扫描可以是完整索引扫描或范围索引扫描，具体取决于查询条件。

当不满足松散索引扫描的条件时，仍然可能避免为`GROUP BY`查询创建临时表。如果`WHERE`子句中存在范围条件，则此方法仅读取满足这些条件的键。否则，它执行索引扫描。因为此方法在`WHERE`子句定义的每个范围中读取所有键，或者如果没有范围条件则扫描整个索引，所以称为紧凑索引扫描。使用紧凑索引扫描，只有在找到满足范围条件的所有键之后才执行分组操作。

要使这种方法起作用，查询中涉及到`GROUP BY`键的所有列都需要有一个常量相等条件，该条件位于`GROUP BY`键的前部或中间。相等条件中的常量填补了搜索键中的任何“间隙”，从而可以形成索引的完整前缀。然后可以使用这些索引前缀进行索引查找。如果`GROUP BY`结果需要排序，并且可以形成索引的前缀搜索键，MySQL 还会避免额外的排序操作，因为在有序索引中使用前缀搜索已经按顺序检索了所有键。

假设在表`t1(c1,c2,c3,c4)`上有一个索引`idx(c1,c2,c3)`。前面描述的松散索引扫描访问方法无法处理以下查询，但仍可使用紧凑索引扫描访问方法。

+   `GROUP BY`中存在一个间隙，但被条件`c2 = 'a'`覆盖：

    ```sql
    SELECT c1, c2, c3 FROM t1 WHERE c2 = 'a' GROUP BY c1, c3;
    ```

+   `GROUP BY`不是以键的第一部分开头，但有一个条件为该部分提供了一个常量：

    ```sql
    SELECT c1, c2, c3 FROM t1 WHERE c1 = 'a' GROUP BY c2, c3;
    ```
