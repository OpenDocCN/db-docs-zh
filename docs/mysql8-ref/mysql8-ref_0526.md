> 原文：[`dev.mysql.com/doc/refman/8.0/en/condition-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/condition-filtering.html)

#### 10.2.1.13 条件过滤

在连接处理中，前缀行是从一个连接中传递到下一个连接的行。 一般来说，优化器尝试尽早将前缀计数较低的表放在连接顺序中，以避免行组合数量迅速增加。 在优化器可以使用有关从一个表中选择并传递到下一个表的行的条件的信息的程度上，它可以更准确地计算行估计并选择最佳执行计划。

没有条件过滤，表的前缀行数是基于优化器根据选择的访问方法估计的`WHERE`子句选择的行数。 条件过滤使优化器能够使用`WHERE`子句中未被访问方法考虑的其他相关条件，并因此改进其前缀行数估计。 例如，即使可能存在可用于从当前表中选择行的基于索引的访问方法，也可能存在`WHERE`子句中对表的其他条件进行过滤（进一步限制）的情况，以过滤传递到下一个表的符合条件的行的估计。

仅当条件对过滤估计有贡献时才会计入：

+   它指的是当前表。

+   它取决于连接序列中较早表的常量值或值。

+   它在访问方法中尚未考虑。

在`EXPLAIN`输出中，`rows`列指示所选访问方法的行估计，而`filtered`列反映条件过滤的效果。 `filtered`值以百分比表示。 最大值为 100，表示未发生任何行过滤。 从 100 减少的值表示过滤量增加。

前缀行数（从当前连接中传递到下一个连接的估计行数）是`rows`和`filtered`值的乘积。 也就是说，前缀行数是估计的行数，减去估计的过滤效果。 例如，如果`rows`为 1000，`filtered`为 20％，则条件过滤将 1000 的估计行数减少为 1000 × 20％= 1000 × .2 = 200。

考虑以下查询：

```sql
SELECT *
  FROM employee JOIN department ON employee.dept_no = department.dept_no
  WHERE employee.first_name = 'John'
  AND employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01';
```

假设数据集具有以下特征：

+   `employee`表有 1024 行。

+   `department`表有 12 行。

+   两个表在`dept_no`上都有索引。

+   `employee`表在`first_name`上有一个索引。

+   8 行满足`employee.first_name`上的这个条件：

    ```sql
    employee.first_name = 'John'
    ```

+   150 行满足`employee.hire_date`上的这个条件：

    ```sql
    employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01'
    ```

+   1 行同时满足这两个条件：

    ```sql
    employee.first_name = 'John'
    AND employee.hire_date BETWEEN '2018-01-01' AND '2018-06-01'
    ```

没有条件过滤，`EXPLAIN`会产生如下输出：

```sql
+----+------------+--------+------------------+---------+---------+------+----------+
| id | table      | type   | possible_keys    | key     | ref     | rows | filtered |
+----+------------+--------+------------------+---------+---------+------+----------+
| 1  | employee   | ref    | name,h_date,dept | name    | const   | 8    | 100.00   |
| 1  | department | eq_ref | PRIMARY          | PRIMARY | dept_no | 1    | 100.00   |
+----+------------+--------+------------------+---------+---------+------+----------+
```

对于`employee`，在`name`索引上的访问方法选择匹配名称为`'John'`的 8 行。不进行过滤（`filtered`为 100%），因此所有行都是下一个表的前缀行：前缀行数为`rows` × `filtered` = 8 × 100% = 8。

使用条件过滤，优化器还考虑`WHERE`子句中未考虑的条件。在这种情况下，优化器使用启发式方法估计`employee.hire_date`上`BETWEEN`条件的过滤效果为 16.31%。因此，`EXPLAIN`生成类似以下输出：

```sql
+----+------------+--------+------------------+---------+---------+------+----------+
| id | table      | type   | possible_keys    | key     | ref     | rows | filtered |
+----+------------+--------+------------------+---------+---------+------+----------+
| 1  | employee   | ref    | name,h_date,dept | name    | const   | 8    | 16.31    |
| 1  | department | eq_ref | PRIMARY          | PRIMARY | dept_no | 1    | 100.00   |
+----+------------+--------+------------------+---------+---------+------+----------+
```

现在前缀行数为`rows` × `filtered` = 8 × 16.31% = 1.3，更接近实际数据集。

通常，优化器不会计算最后一个连接表的条件过滤效果（前缀行数减少），因为没有下一个表可以传递行。一个例外情况是`EXPLAIN`：为了提供更多信息，过滤效果将计算所有连接表，包括最后一个。

要控制优化器是否考虑额外的过滤条件，请使用`optimizer_switch`系统变量的`condition_fanout_filter`标志（参见第 10.9.2 节，“可切换的优化”）。该标志默认启用，但可以禁用以抑制条件过滤（例如，如果发现特定查询在没有条件过滤的情况下性能更好）。

如果优化器高估了条件过滤的效果，性能可能会比不使用条件过滤更差。在这种情况下，可以采用以下技巧：

+   如果某列没有索引，请为其创建索引，以便优化器了解列值的分布并改进其行估计。

+   类似地，如果没有列直方图信息可用，请生成直方图（参见第 10.9.6 节，“优化器统计”）。

+   更改连接顺序。实现这一点的方法包括连接顺序优化器提示（参见第 10.9.3 节，“优化器提示”）、`SELECT`后紧跟`STRAIGHT_JOIN`，以及`STRAIGHT_JOIN`连接操作符。

+   禁用会话的条件过滤：

    ```sql
    SET optimizer_switch = 'condition_fanout_filter=off';
    ```

    或者，对于给定的查询，使用优化器提示：

    ```sql
    SELECT /*+ SET_VAR(optimizer_switch = 'condition_fanout_filter=off') */ ...
    ```
