> 原文：[`dev.mysql.com/doc/refman/8.0/en/hash-joins.html`](https://dev.mysql.com/doc/refman/8.0/en/hash-joins.html)

#### 10.2.1.4 哈希连接优化

默认情况下，MySQL（8.0.18 及更高版本）尽可能使用哈希连接。可以使用`BNL`和`NO_BNL`优化器提示之一，或者将`block_nested_loop=on`或`block_nested_loop=off`作为 optimizer_switch 服务器系统变量设置的一部分来控制是否使用哈希连接。

注意

MySQL 8.0.18 支持在`optimizer_switch`中设置`hash_join`标志，以及优化器提示`HASH_JOIN`和`NO_HASH_JOIN`。在 MySQL 8.0.19 及更高版本中，这些都不再起作用。

从 MySQL 8.0.18 开始，MySQL 为每个连接都具有等值连接条件的查询使用哈希连接，并且没有可以应用于任何连接条件的索引，就像这个例子一样：

```sql
SELECT *
    FROM t1
    JOIN t2
        ON t1.c1=t2.c1;
```

当存在一个或多个可以用于单表谓词的索引时，也可以使用哈希连接。

哈希连接通常比块嵌套循环算法更快，并且在这种情况下应该使用哈希连接，而不是在以前版本的 MySQL 中使用的块嵌套循环算法（参见块嵌套循环连接算法）。从 MySQL 8.0.20 开始，不再支持块嵌套循环，服务器在以前可能使用块嵌套循环的地方使用哈希连接。

在刚刚显示的示例和本节中的其余示例中，我们假设使用以下语句创建了三个表`t1`、`t2`和`t3`：

```sql
CREATE TABLE t1 (c1 INT, c2 INT);
CREATE TABLE t2 (c1 INT, c2 INT);
CREATE TABLE t3 (c1 INT, c2 INT);
```

你可以通过使用`EXPLAIN`来查看哈希连接的使用情况，就像这样：

```sql
mysql> EXPLAIN
 -> SELECT * FROM t1
 ->     JOIN t2 ON t1.c1=t2.c1\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t2
   partitions: NULL
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using where; Using join buffer (hash join)
```

(在 MySQL 8.0.20 之前，有必要包含`FORMAT=TREE`选项，以查看给定连接是否使用了哈希连接。)

`EXPLAIN ANALYZE`还显示了使用的哈希连接的信息。

哈希连接也用于涉及多个连接的查询，只要每对表的至少一个连接条件是等值连接，就像这里显示的查询一样：

```sql
SELECT * FROM t1
    JOIN t2 ON (t1.c1 = t2.c1 AND t1.c2 < t2.c2)
    JOIN t3 ON (t2.c1 = t3.c1);
```

在像刚刚显示的这种情况下，使用内连接时，任何不是等值连接的额外条件在连接执行后作为过滤器应用。（对于外连接，如左连接、半连接和反连接，它们作为连接的一部分打印。）这可以在`EXPLAIN`的输出中看到：

```sql
mysql> EXPLAIN FORMAT=TREE
 -> SELECT *
 ->     FROM t1
 ->     JOIN t2
 ->         ON (t1.c1 = t2.c1 AND t1.c2 < t2.c2)
 ->     JOIN t3
 ->         ON (t2.c1 = t3.c1)\G
*************************** 1\. row ***************************
EXPLAIN: -> Inner hash join (t3.c1 = t1.c1)  (cost=1.05 rows=1)
 -> Table scan on t3  (cost=0.35 rows=1)
 -> Hash
 -> Filter: (t1.c2 < t2.c2)  (cost=0.70 rows=1)
 -> Inner hash join (t2.c1 = t1.c1)  (cost=0.70 rows=1)
 -> Table scan on t2  (cost=0.35 rows=1)
 -> Hash
 -> Table scan on t1  (cost=0.35 rows=1)
```

正如刚刚展示的输出所示，多个哈希连接可以用于具有多个等值连接条件的连接。

在 MySQL 8.0.20 之前，如果任何一对连接的表没有至少一个等值连接条件，则无法使用哈希连接，并且会使用较慢的块嵌套循环算法。在 MySQL 8.0.20 及更高版本中，在这种情况下使用哈希连接，如下所示：

```sql
mysql> EXPLAIN FORMAT=TREE
 -> SELECT * FROM t1
 ->     JOIN t2 ON (t1.c1 = t2.c1)
 ->     JOIN t3 ON (t2.c1 < t3.c1)\G
*************************** 1\. row ***************************
EXPLAIN: -> Filter: (t1.c1 < t3.c1)  (cost=1.05 rows=1)
 -> Inner hash join (no condition)  (cost=1.05 rows=1)
 -> Table scan on t3  (cost=0.35 rows=1)
 -> Hash
 -> Inner hash join (t2.c1 = t1.c1)  (cost=0.70 rows=1)
 -> Table scan on t2  (cost=0.35 rows=1)
 -> Hash
 -> Table scan on t1  (cost=0.35 rows=1)
```

(本节稍后提供更多示例。)

当未指定连接条件时，哈希连接也适用于笛卡尔积，如下所示：

```sql
mysql> EXPLAIN FORMAT=TREE
 -> SELECT *
 ->     FROM t1
 ->     JOIN t2
 ->     WHERE t1.c2 > 50\G
*************************** 1\. row ***************************
EXPLAIN: -> Inner hash join  (cost=0.70 rows=1)
 -> Table scan on t2  (cost=0.35 rows=1)
 -> Hash
 -> Filter: (t1.c2 > 50)  (cost=0.35 rows=1)
 -> Table scan on t1  (cost=0.35 rows=1)
```

在 MySQL 8.0.20 及更高版本中，连接不再需要至少包含一个等值连接条件才能使用哈希连接。这意味着可以使用哈希连接优化的查询类���包括以下列表中的查询（附有示例）：

+   *内部非等值连接*：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM t1 JOIN t2 ON t1.c1 < t2.c1\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Filter: (t1.c1 < t2.c1)  (cost=4.70 rows=12)
     -> Inner hash join (no condition)  (cost=4.70 rows=12)
     -> Table scan on t2  (cost=0.08 rows=6)
     -> Hash
     -> Table scan on t1  (cost=0.85 rows=6)
    ```

+   *半连接*：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM t1 
     ->     WHERE t1.c1 IN (SELECT t2.c2 FROM t2)\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Hash semijoin (t2.c2 = t1.c1)  (cost=0.70 rows=1)
     -> Table scan on t1  (cost=0.35 rows=1)
     -> Hash
     -> Table scan on t2  (cost=0.35 rows=1)
    ```

+   *反连接*：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM t2 
     ->     WHERE NOT EXISTS (SELECT * FROM t1 WHERE t1.c1 = t2.c1)\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Hash antijoin (t1.c1 = t2.c1)  (cost=0.70 rows=1)
     -> Table scan on t2  (cost=0.35 rows=1)
     -> Hash
     -> Table scan on t1  (cost=0.35 rows=1)

    1 row in set, 1 warning (0.00 sec)

    mysql> SHOW WARNINGS\G
    *************************** 1\. row ***************************
      Level: Note
       Code: 1276
    Message: Field or reference 't3.t2.c1' of SELECT #2 was resolved in SELECT #1
    ```

+   *左外连接*：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM t1 LEFT JOIN t2 ON t1.c1 = t2.c1\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Left hash join (t2.c1 = t1.c1)  (cost=0.70 rows=1)
     -> Table scan on t1  (cost=0.35 rows=1)
     -> Hash
     -> Table scan on t2  (cost=0.35 rows=1)
    ```

+   *右外连接*（请注意，MySQL 将所有右外连接重写为左外连接）：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM t1 RIGHT JOIN t2 ON t1.c1 = t2.c1\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Left hash join (t1.c1 = t2.c1)  (cost=0.70 rows=1)
     -> Table scan on t2  (cost=0.35 rows=1)
     -> Hash
     -> Table scan on t1  (cost=0.35 rows=1)
    ```

默认情况下，MySQL 8.0.18 及更高版本在可能的情况下始终使用哈希连接。可以使用`BNL`和`NO_BNL`优化器提示之一来控制是否使用哈希连接。

（MySQL 8.0.18 支持`hash_join=on`或`hash_join=off`作为`optimizer_switch`服务器系统变量设置的一部分，以及优化器提示`HASH_JOIN`或`NO_HASH_JOIN`。在 MySQL 8.0.19 及更高版本中，这些不再起作用。）

可以使用`join_buffer_size`系统变量来控制哈希连接的内存使用情况；哈希连接不能使用超过此数量的内存。当哈希连接所需的内存超过可用量时，MySQL 会通过在磁盘上使用文件来处理此问题。如果发生这种情况，您应该意识到，如果哈希连接无法适应内存并且创建的文件多于为`open_files_limit`设置的数量，则连接可能不会成功。为避免此类问题，请进行以下更改之一：

+   增加`join_buffer_size`，以使哈希连接不会溢出到磁盘。

+   增加`open_files_limit`。

从 MySQL 8.0.18 开始，为哈希连接分配增量式连接缓冲区；因此，您可以将`join_buffer_size`设置得更高，而不会使小查询分配大量 RAM，但外连接会分配整个缓冲区。在 MySQL 8.0.20 及更高版本中，哈希连接也用于外连接（包括反连接和半连接），因此这不再是一个问题。
