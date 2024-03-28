> 原文：[`dev.mysql.com/doc/refman/8.0/en/where-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/where-optimization.html)

#### 10.2.1.1 WHERE 子句优化

本节讨论了用于处理`WHERE`子句的优化。示例使用`SELECT`语句，但相同的优化也适用于`DELETE`和`UPDATE`语句中的`WHERE`子句。

注意

由于 MySQL 优化器的工作正在进行中，MySQL 执行的并非所有优化都在此处记录。

您可能会尝试重写查询以加快算术运算速度，但会牺牲可读性。由于 MySQL 自动执行类似的优化，您通常可以避免这项工作，并将查询保留在更易理解和可维护的形式中。MySQL 执行的一些优化如下：

+   删除不必要的括号：

    ```sql
     ((a AND b) AND c OR (((a AND b) AND (c AND d))))
    -> (a AND b AND c) OR (a AND b AND c AND d)
    ```

+   常量折叠：

    ```sql
     (a<b AND b=c) AND a=5
    -> b>5 AND b=c AND a=5
    ```

+   常量条件移除：

    ```sql
     (b>=5 AND b=5) OR (b=6 AND 5=5) OR (b=7 AND 5=6)
    -> b=5 OR b=6
    ```

    在 MySQL 8.0.14 及更高版本中，这是在准备阶段而不是在优化阶段进行的，这有助于简化连接。有关更多信息和示例，请参见 Section 10.2.1.9, “Outer Join Optimization”。

+   用于索引的常量表达式仅评估一次。

+   从 MySQL 8.0.16 开始，将检查和折叠或删除数值类型列与常量值的比较，以处理无效或超出范围的值：

    ```sql
    # CREATE TABLE t (c TINYINT UNSIGNED NOT NULL);
      SELECT * FROM t WHERE c ≪ 256;
    -≫ SELECT * FROM t WHERE 1;
    ```

    更多信息请参见 Section 10.2.1.14, “Constant-Folding Optimization”。

+   在没有`WHERE`的单个表上的`COUNT(*)`直接从`MyISAM`和`MEMORY`表的表信息中检索。当与仅一个表一起使用时，对于任何`NOT NULL`表达式也会执行此操作。

+   早期检测无效常量表达式。MySQL 快速检测到一些`SELECT`语句是不可能的，并且不返回任何行。

+   如果您不使用`GROUP BY`或聚合函数（`COUNT()`, `MIN()`等），`HAVING`将与`WHERE`合并。

+   对于连接中的每个表，构建一个更简单的`WHERE`以获得快速的表`WHERE`评估，并尽快跳过行。

+   在查询中的任何其他表之前首先读取所有常量表。常量表包括以下内容：

    +   空表或具有一行的表。

    +   在`PRIMARY KEY`或`UNIQUE`索引上使用`WHERE`子句的表，其中所有索引部分与常量表达式进行比较并定义为`NOT NULL`。

    所有以下表都被用作常量表：

    ```sql
    SELECT * FROM t WHERE *primary_key*=1;
    SELECT * FROM t1,t2
      WHERE t1.*primary_key*=1 AND t2.*primary_key*=t1.id;
    ```

+   通过尝试所有可能性来找到连接表的最佳连接组合。如果`ORDER BY`和`GROUP BY`子句中的所有列来自同一张表，那么在连接时首选该表。

+   如果存在`ORDER BY`子句和不同的`GROUP BY`子句，或者如果`ORDER BY`或`GROUP BY`包含来自连接队列中第一张表以外的表的列，那么会创建一个临时表。

+   如果使用`SQL_SMALL_RESULT`修饰符，MySQL 会使用内存临时表。

+   每个表索引都会被查询，除非优化器认为使用表扫描更有效，否则会使用最佳索引。曾经，基于最佳索引是否跨越表的 30%以上而使用扫描，但现在不再根据固定百分比来决定使用索引还是扫描。优化器现在更加复杂，根据诸如表大小、行数和 I/O 块大小等额外因素来估计。

+   在某些情况下，MySQL 可以从索引中读取行，甚至无需查阅数据文件。如果从索引中使用的所有列都是数字型的，则仅使用索引树来解析查询。

+   在输出每一行之前，不符合`HAVING`子句的行会被跳过。

一些查询的示例非常快速：

```sql
SELECT COUNT(*) FROM *tbl_name*;

SELECT MIN(*key_part1*),MAX(*key_part1*) FROM *tbl_name*;

SELECT MAX(*key_part2*) FROM *tbl_name*
  WHERE *key_part1*=*constant*;

SELECT ... FROM *tbl_name*
  ORDER BY *key_part1*,*key_part2*,... LIMIT 10;

SELECT ... FROM *tbl_name*
  ORDER BY *key_part1* DESC, *key_part2* DESC, ... LIMIT 10;
```

MySQL 仅使用索引树解析以下查询，假设索引列是数字型的：

```sql
SELECT *key_part1*,*key_part2* FROM *tbl_name* WHERE *key_part1*=*val*;

SELECT COUNT(*) FROM *tbl_name*
  WHERE *key_part1*=*val1* AND *key_part2*=*val2*;

SELECT MAX(*key_part2*) FROM *tbl_name* GROUP BY *key_part1*;
```

以下查询使用索引检索按排序顺序排列的行，而无需单独的排序过程：

```sql
SELECT ... FROM *tbl_name*
  ORDER BY *key_part1*,*key_part2*,... ;

SELECT ... FROM *tbl_name*
  ORDER BY *key_part1* DESC, *key_part2* DESC, ... ;
```
