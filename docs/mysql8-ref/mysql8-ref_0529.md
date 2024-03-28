> 原文：[`dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/order-by-optimization.html)

#### 10.2.1.16 ORDER BY 优化

本节描述了 MySQL 何时可以使用索引来满足`ORDER BY`子句，当无法使用索引时使用的`filesort`操作，以及优化器提供的关于`ORDER BY`的执行计划信息。

带有和不带有`LIMIT`的`ORDER BY`可能以不同的顺序返回行，如第 10.2.1.19 节，“LIMIT 查询优化”中讨论的那样。

+   使用索引满足 ORDER BY

+   使用 filesort 满足 ORDER BY

+   影响 ORDER BY 优化

+   ORDER BY 执行计划信息可用

##### 使用索引满足 ORDER BY

在某些情况下，MySQL 可能使用索引来满足`ORDER BY`子句，并避免执行`filesort`操作所涉及的额外排序。

只要`ORDER BY`与索引不完全匹配，只要索引的所有未使用部分和所有额外的`ORDER BY`列在`WHERE`子句中都是常量，索引也可以被使用。如果索引不包含查询访问的所有列，那么只有在索引访问比其他访问方法更便宜时才会使用索引。

假设在`(*`key_part1`*, *`key_part2`*)`上有一个索引，以下查询可能使用索引来解决`ORDER BY`部分。优化器是否实际这样做取决于如果索引不包含的列也必须被读取，那么读取索引是否比表扫描更有效。

+   在这个查询中，对`(*`key_part1`*, *`key_part2`*)`的索引使得优化器可以避免排序：

    ```sql
    SELECT * FROM t1
      ORDER BY *key_part1*, *key_part2*;
    ```

    然而，该查询使用了`SELECT *`，可能选择的列比*`key_part1`*和*`key_part2`*多。在这种情况下，扫描整个索引并查找表行以找到不在索引中的列可能比扫描表并对结果进行排序更昂贵。如果是这样，优化器可能不使用索引。如果`SELECT *`只选择索引列，那么索引会被使用，避免排序。

    如果`t1`是一个`InnoDB`表，表主键隐式地是索引的一部分，索引可以用于解决此查询的`ORDER BY`：

    ```sql
    SELECT *pk*, *key_part1*, *key_part2* FROM t1
      ORDER BY *key_part1*, *key_part2*;
    ```

+   在这个查询中，*`key_part1`*是常量，因此通过索引访问的所有行都按*`key_part2`*顺序排列，而且在`WHERE`子句足够选择性的情况下，`(*`key_part1`*, *`key_part2`*)`上的索引避免排序比表扫描更便宜：

    ```sql
    SELECT * FROM t1
      WHERE *key_part1* = *constant*
      ORDER BY *key_part2*;
    ```

+   在接下来的两个查询中，是否使用索引与之前显示的没有`DESC`的相同查询类似：

    ```sql
    SELECT * FROM t1
      ORDER BY *key_part1* DESC, *key_part2* DESC;

    SELECT * FROM t1
      WHERE *key_part1* = *constant*
      ORDER BY *key_part2* DESC;
    ```

+   `ORDER BY`中的两列可以按相同方向排序（都是`ASC`或都是`DESC`），也可以按相反方向排序（一个`ASC`，一个`DESC`）。索引使用的条件是索引必须具有相同的同质性，但实际方向不必相同。

    如果查询混合使用`ASC`和`DESC`，优化器可以在索引上使用这些列，如果索引也使用相应的混合升序和降序列：

    ```sql
    SELECT * FROM t1
      ORDER BY *key_part1* DESC, *key_part2* ASC;
    ```

    如果`key_part1`是降序的，而`key_part2`是升序的，优化器可以在(*`key_part1`*, *`key_part2`*)上使用索引。如果`key_part1`是升序的，而`key_part2`是降序的，它也可以使用这些列的索引（进行反向扫描）。参见 Section 10.3.13, “Descending Indexes”。

+   在接下来的两个查询中，*`key_part1`*与一个常量进行比较。如果`WHERE`子句足够选择性，使得索引范围扫描比表扫描更便宜，则会使用索引：

    ```sql
    SELECT * FROM t1
      WHERE *key_part1* > *constant*
      ORDER BY *key_part1* ASC;

    SELECT * FROM t1
      WHERE *key_part1* < *constant*
      ORDER BY *key_part1* DESC;
    ```

+   在下一个查询中，`ORDER BY`没有命名*`key_part1`*，但所选的所有行都具有常量*`key_part1`*值，因此仍然可以使用索引：

    ```sql
    SELECT * FROM t1
      WHERE *key_part1* = *constant1* AND *key_part2* > *constant2*
      ORDER BY *key_part2*;
    ```

在某些情况下，MySQL *无法* 使用索引解析`ORDER BY`，尽管仍然可以使用索引找到与`WHERE`子句匹配的行。例如：

+   查询在不同的索引上使用`ORDER BY`：

    ```sql
    SELECT * FROM t1 ORDER BY *key1*, *key2*;
    ```

+   查询在索引的非连续部分上使用`ORDER BY`：

    ```sql
    SELECT * FROM t1 WHERE *key2*=*constant* ORDER BY *key1_part1*, *key1_part3*;
    ```

+   用于获取行的索引与`ORDER BY`中使用的索引不同：

    ```sql
    SELECT * FROM t1 WHERE *key2*=*constant* ORDER BY *key1*;
    ```

+   查询使用包含除索引列名之外的项的表达式进行`ORDER BY`：

    ```sql
    SELECT * FROM t1 ORDER BY ABS(*key*);
    SELECT * FROM t1 ORDER BY -*key*;
    ```

+   查询涉及多个表，并且`ORDER BY`中的列并非全部来自用于检索行的第一个非常量表（这是`EXPLAIN`输出中第一个没有`const`连接类型的表）。

+   查询具有不同的`ORDER BY`和`GROUP BY`表达式。

+   在`ORDER BY`子句中只有列名的前缀上有索引。在这种情况下，索引无法完全解析排序顺序。例如，如果只对`CHAR(20)`列的前 10 个字节建立索引，则索引无法区分第 10 个字节之后的值，需要使用`filesort`。

+   索引不按顺序存储行。例如，在`MEMORY`表中的`HASH`索引是如此。

排序的索引可用性可能受到列别名的影响。假设列`t1.a`已建立索引。在此语句中，选择列表中的列名为`a`。它指的是`t1.a`，与`ORDER BY`中对`a`的引用一样，因此可以使用`t1.a`上的索引：

```sql
SELECT a FROM t1 ORDER BY a;
```

在此语句中，选择列表中的列名也是`a`，但它是别名。它指的是`ABS(a)`，与`ORDER BY`中对`a`的引用一样，因此无法使用`t1.a`上的索引：

```sql
SELECT ABS(a) AS a FROM t1 ORDER BY a;
```

在以下语句中，`ORDER BY` 指的是选择列表中不是列名的名称。但是在 `t1` 中有一个名为 `a` 的列，因此 `ORDER BY` 指的是 `t1.a`，并且可以使用在 `t1.a` 上的索引。（当然，生成的排序顺序可能与 `ABS(a)` 的顺序完全不同。）

```sql
SELECT ABS(a) AS b FROM t1 ORDER BY a;
```

以前（MySQL 5.7 及更低版本），在某些条件下，`GROUP BY` 会隐式排序。在 MySQL 8.0 中，不再发生这种情况，因此不再需要在末尾指定 `ORDER BY NULL` 来抑制隐式排序（如以前所做）。但是，查询结果可能与以前的 MySQL 版本不同。为了产生给定的排序顺序，请提供一个 `ORDER BY` 子句。

##### 使用 filesort 满足 ORDER BY

如果无法使用索引满足 `ORDER BY` 子句，MySQL 执行一个 `filesort` 操作，读取表行并对其进行排序。`filesort` 在查询执行中构成额外的排序阶段。

从 MySQL 8.0.12 开始，为了获取 `filesort` 操作的内存，优化器根据需要逐步分配内存缓冲区，直到达到 `sort_buffer_size` 系统变量指示的大小，而不是像在 MySQL 8.0.12 之前那样一次性分配固定数量的 `sort_buffer_size` 字节。这使用户可以将 `sort_buffer_size` 设置为较大的值，以加快较大的排序，而不必担心小排序的过度内存使用。（在 Windows 上，对于多个并发排序，这种好处可能不会发生，因为 Windows 具有弱多线程 `malloc`。）

`filesort` 操作根据需要使用临时磁盘文件，如果结果集太大而无法放入内存中。某些类型的查询特别适合完全在内存中进行 `filesort` 操作。例如，优化器可以使用 `filesort` 来有效地处理内存中的 `ORDER BY` 操作，而无需临时文件，对于以下形式的查询（和子查询）：

```sql
SELECT ... FROM *single_table* ... ORDER BY *non_index_column* [DESC] LIMIT [*M*,]*N*;
```

这些查询在仅显示较大结果集中的几行的 Web 应用程序中很常见。例如：

```sql
SELECT col1, ... FROM t1 ... ORDER BY name LIMIT 10;
SELECT col1, ... FROM t1 ... ORDER BY RAND() LIMIT 15;
```

##### 影响 ORDER BY 优化

对于慢的 `ORDER BY` 查询，如果不使用 `filesort`，请尝试降低 `max_length_for_sort_data` 系统变量的值，以触发 `filesort`。 （设置此变量值过高的症状是高磁盘活动和低 CPU 活动的组合。）这种技术仅适用于 MySQL 8.0.20 之前。从 8.0.20 开始，由于优化器的更改使其过时且无效，`max_length_for_sort_data` 已被弃用。

为了提高 `ORDER BY` 的速度，请检查是否可以让 MySQL 使用索引而不是额外的排序阶段。如果不可能，请尝试以下策略：

+   增加`sort_buffer_size`变量值。理想情况下，该值应足够大，以使整个结果集适合排序缓冲区（以避免写入磁盘和合并操作）。

    请注意，存储在排序缓冲区中的列值的大小受`max_sort_length`系统变量值的影响。例如，如果元组存储长字符串列的值，并增加`max_sort_length`的值，则排序缓冲区元组的大小也会增加，可能需要您增加`sort_buffer_size`。

    要监视合并临时文件的合并次数，请检查`Sort_merge_passes`状态变量。

+   增加`read_rnd_buffer_size`变量值，以便一次读取更多行。

+   将`tmpdir`系统变量更改为指向具有大量可用空间的专用文件系统。变量值可以列出几个路径，以轮询方式使用；您可以使用此功能将负载分散到几个目录中。在 Unix 上用冒号字符（`:`）分隔路径，在 Windows 上用分号字符（`;`）分隔。这些路径应命名位于不同*物理*磁盘上的文件系统中的目录，而不是同一磁盘上的不同分区。

##### ORDER BY 执行计划信息可用

使用`EXPLAIN`（参见 Section 10.8.1，“使用 EXPLAIN 优化查询”输出的`Extra`列不包含`Using filesort`，则使用索引，不执行`filesort`。

+   如果`EXPLAIN`输出的`Extra`列包含`Using filesort`，则未使用索引并执行`filesort`。

此外，如果执行了`filesort`，优化器跟踪输出将包括一个`filesort_summary`块。例如：

```sql
"filesort_summary": {
  "rows": 100,
  "examined_rows": 100,
  "number_of_tmp_files": 0,
  "peak_memory_used": 25192,
  "sort_mode": "<sort_key, packed_additional_fields>"
}
```

`peak_memory_used` 表示在排序过程中任意时间点使用的最大内存。这个值可能与 `sort_buffer_size` 系统变量的值一样大，但不一定。在 MySQL 8.0.12 之前，输出显示的是 `sort_buffer_size`，表示 `sort_buffer_size` 的值。（在 MySQL 8.0.12 之前，优化器总是为排序缓冲区分配 `sort_buffer_size` 字节。从 8.0.12 开始，优化器逐渐分配排序缓冲区内存，从一个小量开始，根据需要逐渐增加，直到 `sort_buffer_size` 字节。）

`sort_mode` 值提供有关排序缓冲区中元组内容的信息：

+   `<sort_key, rowid>`：表示排序缓冲区元组是包含排序键值和原始表行的行 ID 的对。元组按排序键值排序，行 ID 用于从表中读取行。

+   `<sort_key, additional_fields>`：表示排序缓冲区元组包含排序键值和查询引用的列。元组按排序键值排序，列值直接从元组中读取。

+   `<sort_key, packed_additional_fields>`：与前一种变体类似，但附加列紧密打包在一起，而不是使用固定长度编码。

`EXPLAIN` 无法区分优化器是否在内存中执行 `filesort`。内存中的 `filesort` 使用可以在优化器跟踪输出中看到。查找 `filesort_priority_queue_optimization`。有关优化器跟踪的信息，请参阅 MySQL Internals: Tracing the Optimizer。
