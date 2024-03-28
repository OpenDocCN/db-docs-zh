> 原文：[`dev.mysql.com/doc/refman/8.0/en/subquery-optimization-with-exists.html`](https://dev.mysql.com/doc/refman/8.0/en/subquery-optimization-with-exists.html)

#### 10.2.2.3 优化 EXISTS 策略的子查询

某些优化适用于使用 `IN`（或 `=ANY`）运算符测试子查询结果的比较。本节讨论了这些优化，特别是关于 `NULL` 值带来的挑战。讨论的最后部分建议您如何帮助优化器。

考虑以下子查询比较：

```sql
*outer_expr* IN (SELECT *inner_expr* FROM ... WHERE *subquery_where*)
```

MySQL 从“外到内”评估查询。也就是说，它首先获取外部表达式 *`outer_expr`* 的值，然后运行子查询并捕获其生成的行。

一个非常有用的优化是“通知”子查询感兴趣的行仅为内部表达式 *`inner_expr`* 等于 *`outer_expr`* 的行。这是通过将适当的相等性推送到子查询的 `WHERE` 子句中以使其更加严格来完成的。转换后的比较如下所示：

```sql
EXISTS (SELECT 1 FROM ... WHERE *subquery_where* AND *outer_expr*=*inner_expr*)
```

转换后，MySQL 可以使用推送的相等性来限制必须检查的行数以评估子查询。

更一般地，将 *`N`* 个值与返回 *`N`* 个值行的子查询进行比较会受到相同的转换影响。如果 *`oe_i`* 和 *`ie_i`* 表示相应的外部和内部表达式值，则此子查询比较：

```sql
(*oe_1*, ..., *oe_N*) IN
  (SELECT *ie_1*, ..., *ie_N* FROM ... WHERE *subquery_where*)
```

变为：

```sql
EXISTS (SELECT 1 FROM ... WHERE *subquery_where*
                          AND *oe_1* = *ie_1*
                          AND ...
                          AND *oe_N* = *ie_N*)
```

为简单起见，以下讨论假定有一对外部和内部表达式值。

刚刚描述的“推送”策略在以下条件之一为真时有效：

+   *`outer_expr`* 和 *`inner_expr`* 不能为 `NULL`。

+   您无需区分 `NULL` 和 `FALSE` 子查询结果。如果子查询是 `WHERE` 子句中的 `OR` 或 `AND` 表达式的一部分，MySQL 假定您不关心。另一个优化器注意到 `NULL` 和 `FALSE` 子查询结果无需区分的情况是这种结构：

    ```sql
    ... WHERE *outer_expr* IN (*subquery*)
    ```

    在这种情况下，`WHERE` 子句拒绝行，无论 `IN (*`subquery`*)` 返回 `NULL` 还是 `FALSE`。

假设 *`outer_expr`* 已知是非 `NULL` 值，但子查询没有生成这样的行，使得 *`outer_expr`* = *`inner_expr`*。那么 `*`outer_expr`* IN (SELECT ...)` 的评估如下：

+   `NULL`，如果 `SELECT` 生成任何行，其中 *`inner_expr`* 为 `NULL`

+   `FALSE`，如果 `SELECT` 仅生成非 `NULL` 值或不生成任何内容

在这种情况下，寻找 `*`outer_expr`* = *`inner_expr`*` 的行的方法不再有效。需要寻找这样的行，但如果找不到，则还需要寻找 *`inner_expr`* 为 `NULL` 的行。粗略地说，子查询可以转换为类似于以下内容：

```sql
EXISTS (SELECT 1 FROM ... WHERE *subquery_where* AND
        (*outer_expr*=*inner_expr* OR *inner_expr* IS NULL))
```

需要评估额外的`IS NULL`条件是 MySQL 拥有`ref_or_null`访问方法的原因：

```sql
mysql> EXPLAIN
       SELECT *outer_expr* IN (SELECT t2.maybe_null_key
                             FROM t2, t3 WHERE ...)
       FROM t1;
*************************** 1\. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
...
*************************** 2\. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t2
         type: ref_or_null
possible_keys: maybe_null_key
          key: maybe_null_key
      key_len: 5
          ref: func
         rows: 2
        Extra: Using where; Using index
...
```

`unique_subquery`和`index_subquery`子查询特定的访问方法也有“or `NULL`”变体。

额外的`OR ... IS NULL`条件使得查询执行稍微复杂一些（子查询内的一些优化变得不适用），但通常是可以容忍的。

当*`outer_expr`*可以为`NULL`时情况会更糟。根据 SQL 将`NULL`解释为“未知值”，`NULL IN (SELECT *`inner_expr`* ...)`应该评估为：

+   如果`SELECT`产生任何行，则为`NULL`

+   如果`SELECT`不产生任何行，则为`FALSE`

为了正确评估，有必要检查`SELECT`是否产生了任何行，因此`*`outer_expr`* = *`inner_expr`*`不能被推送到子查询中。这是一个问题，因为许多现实世界的子查询会变得非常缓慢，除非相等性可以被推送下去。

本质上，必须有不同的方式来执行子查询，取决于*`outer_expr`*的值。

优化器选择 SQL 的兼容性而非速度，因此它考虑到*`outer_expr`*可能是`NULL`的可能性：

+   如果*`outer_expr`*为`NULL`，为了评估以下表达式，需要执行`SELECT`以确定是否产生任何行：

    ```sql
    NULL IN (SELECT *inner_expr* FROM ... WHERE *subquery_where*)
    ```

    在这里需要执行原始的`SELECT`，而不带有之前提到的任何推送下来的相等性。

+   另一方面，当*`outer_expr`*不为`NULL`时，绝对必要进行以下比较：

    ```sql
    *outer_expr* IN (SELECT *inner_expr* FROM ... WHERE *subquery_where*)
    ```

    被转换为使用推送下来条件的这个表达式：

    ```sql
    EXISTS (SELECT 1 FROM ... WHERE *subquery_where* AND *outer_expr*=*inner_expr*)
    ```

    没有这种转换，子查询会很慢。

为了解决是否将条件推送到子查询中的困境，条件被包装在“触发器”函数中。因此，以下形式的表达式：

```sql
*outer_expr* IN (SELECT *inner_expr* FROM ... WHERE *subquery_where*)
```

被转换为：

```sql
EXISTS (SELECT 1 FROM ... WHERE *subquery_where*
                          AND trigcond(*outer_expr*=*inner_expr*))
```

更一般地，如果子查询比较基于几对外部和内部表达式，转换将采用这种比较：

```sql
(*oe_1*, ..., *oe_N*) IN (SELECT *ie_1*, ..., *ie_N* FROM ... WHERE *subquery_where*)
```

并将其转换为以下表达式：

```sql
EXISTS (SELECT 1 FROM ... WHERE *subquery_where*
                          AND trigcond(*oe_1*=*ie_1*)
                          AND ...
                          AND trigcond(*oe_N*=*ie_N*)
       )
```

每个`trigcond(*`X`*)`是一个特殊函数，其计算结果如下：

+   当“链接”外部表达式*`oe_i`*不为`NULL`时为*`X`*

+   当“链接”外部表达式*`oe_i`*为`NULL`时为`TRUE`

注意

触发器函数不是您使用`CREATE TRIGGER`创建的那种触发器。

被包裹在`trigcond()`函数内的相等性不是查询优化器的一流谓词。大多数优化无法处理可能在查询执行时打开和关闭的谓词，因此它们假定任何`trigcond(*`X`*)`都是一个未知函数并将其忽略。触发的相等性可以被这些优化使用：

+   参考优化：`trigcond(*`X`*=*`Y`* [OR *`Y`* IS NULL])`可以用来构建`ref`、`eq_ref`或`ref_or_null`表访问。

+   基于索引查找的子查询执行引擎：`trigcond(*`X`*=*`Y`*)`可以用来构建`unique_subquery`或`index_subquery`访问。

+   表条件生成器：如果子查询是几个表的连接，触发的条件将尽快被检查。

当优化器使用触发的条件来创建某种基于索引查找的访问（如前述列表的前两项），它必须有一个针对条件关闭的备用策略。这个备用策略总是相同的：做一个完整的表扫描。在`EXPLAIN`输出中，备用策略显示为`Extra`列中的`Full scan on NULL key`：

```sql
mysql> EXPLAIN SELECT t1.col1,
       t1.col1 IN (SELECT t2.key1 FROM t2 WHERE t2.col2=t1.col2) FROM t1\G
*************************** 1\. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
        ...
*************************** 2\. row ***************************
           id: 2
  select_type: DEPENDENT SUBQUERY
        table: t2
         type: index_subquery
possible_keys: key1
          key: key1
      key_len: 5
          ref: func
         rows: 2
        Extra: Using where; Full scan on NULL key
```

如果你运行`EXPLAIN`，然后跟着运行`SHOW WARNINGS`，你可以看到触发的条件：

```sql
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: select `test`.`t1`.`col1` AS `col1`,
         <in_optimizer>(`test`.`t1`.`col1`,
         <exists>(<index_lookup>(<cache>(`test`.`t1`.`col1`) in t2
         on key1 checking NULL
         where (`test`.`t2`.`col2` = `test`.`t1`.`col2`) having
         trigcond(<is_not_null_test>(`test`.`t2`.`key1`))))) AS
         `t1.col1 IN (select t2.key1 from t2 where t2.col2=t1.col2)`
         from `test`.`t1`
```

使用触发条件会带来一些性能影响。现在，一个`NULL IN (SELECT ...)`表达式可能会导致一个全表扫描（这是慢的），而以前则不会。这是为了获得正确结果而付出的代价（触发条件策略的目标是提高合规性，而不是速度）。

对于多表子查询，执行`NULL IN (SELECT ...)`特别慢，因为连接优化器不会为外部表达式为`NULL`的情况进行优化。它假设左侧带有`NULL`的子查询评估非常罕见，即使有统计数据表明相反。另一方面，如果外部表达式可能是`NULL`但实际上从未是，那么就不会有性能惩罚。

为了帮助查询优化器更好地执行您的查询，请使用以下建议：

+   如果一个列确实是`NOT NULL`，就声明它为`NOT NULL`。这也通过简化列的条件测试来帮助优化器的其他方面。

+   如果你不需要区分`NULL`和`FALSE`子查询结果，你可以轻松避免慢执行路径。替换一个看起来像这样的比较：

    ```sql
    *outer_expr* [NOT] IN (SELECT *inner_expr* FROM ...)
    ```

    使用这个表达式：

    ```sql
    (*outer_expr* IS NOT NULL) AND (*outer_expr* [NOT] IN (SELECT *inner_expr* FROM ...))
    ```

    然后 `NULL IN (SELECT ...)` 从未被评估，因为一旦表达式结果明确，MySQL 就会停止评估`AND`部分。

    另一种可能的重写：

    ```sql
    [NOT] EXISTS (SELECT *inner_expr* FROM ...
            WHERE *inner_expr*=*outer_expr*)
    ```

`optimizer_switch`系统变量的`subquery_materialization_cost_based`标志允许控制在子查询材料化和`IN`到`EXISTS`子查询转换之间的选择。参见第 10.9.2 节，“可切换优化”。
