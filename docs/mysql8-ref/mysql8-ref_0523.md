> 原文：[`dev.mysql.com/doc/refman/8.0/en/outer-join-simplification.html`](https://dev.mysql.com/doc/refman/8.0/en/outer-join-simplification.html)

#### 10.2.1.10 外连接简化

查询的`FROM`子句中的表达式在许多情况下被简化。

在解析器阶段，具有右外连接操作的查询被转换为仅包含左连接操作的等效查询。在一般情况下，转换是这样执行的，即这个右连接：

```sql
(T1, ...) RIGHT JOIN (T2, ...) ON P(T1, ..., T2, ...)
```

变成这个等效的左连接：

```sql
(T2, ...) LEFT JOIN (T1, ...) ON P(T1, ..., T2, ...)
```

所有形式为`T1 INNER JOIN T2 ON P(T1,T2)`的内连接表达式都被替换为列表`T1,T2`，`P(T1,T2)`作为`WHERE`条件的一个连接被连接（或者连接的连接条件，如果有的话）。

当优化器评估外连接操作的计划时，它只考虑那些在每个这样的操作中，外部表在内部表之前被访问的计划。优化器的选择是有限的，因为只有这样的计划才能使用嵌套循环算法执行外连接。

考虑这种形式的查询，其中`R(T2)`大大缩小了与表`T2`匹配行的数量：

```sql
SELECT * T1 FROM T1
  LEFT JOIN T2 ON P1(T1,T2)
  WHERE P(T1,T2) AND R(T2)
```

如果按照原样执行查询，优化器别无选择，只能在访问受限制较少的表`T1`之前访问受限制较多的表`T2`，这可能会产生一个非常低效的执行计划。

相反，如果`WHERE`条件被外连接操作拒绝，MySQL 会将查询转换为不包含外连接操作的查询。（也就是说，它将外连接转换为内连接。）如果条件对于为操作生成的任何`NULL`-补充行求值为`FALSE`或`UNKNOWN`，则称条件对于外连接操作被拒绝。

因此，对于这个外连接：

```sql
T1 LEFT JOIN T2 ON T1.A=T2.A
```

诸如这些条件之类的条件被拒绝，因为它们对于任何`NULL`-补充行（`T2`列设置为`NULL`）都不可能为真：

```sql
T2.B IS NOT NULL
T2.B > 3
T2.C <= T1.C
T2.B < 2 OR T2.C > 1
```

诸如这些条件之类的条件不被拒绝，因为它们可能对于一个`NULL`-补充行为真：

```sql
T2.B IS NULL
T1.B < 3 OR T2.B IS NOT NULL
T1.B < 3 OR T2.B > 3
```

检查条件是否被外连接操作拒绝的一般规则很简单：

+   它的形式是`A IS NOT NULL`，其中`A`是任何内部表的属性

+   它是一个包含对内部表的引用的谓词，当其参数之一为`NULL`时求值为`UNKNOWN`

+   它是一个包含被拒绝条件的合取作为一个连接

+   它是一个被拒绝的条件的析取

一个条件可以被一个查询中的一个外连接操作拒绝，但对另一个查询中的另一个外连接操作不被拒绝。在这个查询中，`WHERE`条件对于第二个外连接操作被拒绝，但对于第一个外连接操作不被拒绝：

```sql
SELECT * FROM T1 LEFT JOIN T2 ON T2.A=T1.A
                 LEFT JOIN T3 ON T3.B=T1.B
  WHERE T3.C > 0
```

如果`WHERE`条件在查询中被外连接操作拒绝，外连接操作将被替换为内连接操作。

例如，在前面的查询中，第二个外连接被空拒绝，可以被内连接替换：

```sql
SELECT * FROM T1 LEFT JOIN T2 ON T2.A=T1.A
                 INNER JOIN T3 ON T3.B=T1.B
  WHERE T3.C > 0
```

对于原始查询，优化器仅评估与单表访问顺序`T1，T2，T3`兼容的计划。对于重写后的查询，它还考虑了访问顺序`T3，T1，T2`。

一个外连接操作的转换可能会触发另一个的转换。因此，查询：

```sql
SELECT * FROM T1 LEFT JOIN T2 ON T2.A=T1.A
                 LEFT JOIN T3 ON T3.B=T2.B
  WHERE T3.C > 0
```

首先被转换为查询：

```sql
SELECT * FROM T1 LEFT JOIN T2 ON T2.A=T1.A
                 INNER JOIN T3 ON T3.B=T2.B
  WHERE T3.C > 0
```

相当于查询：

```sql
SELECT * FROM (T1 LEFT JOIN T2 ON T2.A=T1.A), T3
  WHERE T3.C > 0 AND T3.B=T2.B
```

剩余的外连接操作也可以被内连接替换，因为条件`T3.B=T2.B`被拒绝为空。这导致一个没有任何外连接的查询：

```sql
SELECT * FROM (T1 INNER JOIN T2 ON T2.A=T1.A), T3
  WHERE T3.C > 0 AND T3.B=T2.B
```

有时优化器成功地替换了嵌套的外连接操作，但无法转换嵌套外连接。以下查询：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2 LEFT JOIN T3 ON T3.B=T2.B)
              ON T2.A=T1.A
  WHERE T3.C > 0
```

被转换为：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2 INNER JOIN T3 ON T3.B=T2.B)
              ON T2.A=T1.A
  WHERE T3.C > 0
```

只能被重写为仍然包含嵌套外连接操作的形式：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2,T3)
              ON (T2.A=T1.A AND T3.B=T2.B)
  WHERE T3.C > 0
```

任何尝试将查询中的嵌套外连接操作转换必须考虑嵌套外连接的连接条件以及`WHERE`条件。在这个查询中，`WHERE`条件对于嵌套外连接不被拒绝为空，但是嵌套外连接的连接条件`T2.A=T1.A AND T3.C=T1.C`被拒绝为空：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2 LEFT JOIN T3 ON T3.B=T2.B)
              ON T2.A=T1.A AND T3.C=T1.C
  WHERE T3.D > 0 OR T1.D > 0
```

因此，查询可以转换为：

```sql
SELECT * FROM T1 LEFT JOIN
              (T2, T3)
              ON T2.A=T1.A AND T3.C=T1.C AND T3.B=T2.B
  WHERE T3.D > 0 OR T1.D > 0
```
