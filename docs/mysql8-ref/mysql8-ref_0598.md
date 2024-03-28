# 10.8.3 扩展 EXPLAIN 输出格式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/explain-extended.html`](https://dev.mysql.com/doc/refman/8.0/en/explain-extended.html)

`EXPLAIN`语句生成额外（“扩展”）信息，这些信息不是`EXPLAIN`输出的一部分，但可以通过在`EXPLAIN`后发出`SHOW WARNINGS`语句来查看。从 MySQL 8.0.12 开始，扩展信息适用于`SELECT`、`DELETE`、`INSERT`、`REPLACE`和`UPDATE`语句。在 8.0.12 之前，扩展信息仅适用于`SELECT`语句。

`SHOW WARNINGS`输出中的`Message`值显示了优化器在`SELECT`语句中对表和列名进行限定的方式，应用重写和优化规则后的`SELECT`的样子，以及可能关于优化过程的其他说明。

在`EXPLAIN`后跟随`SHOW WARNINGS`语句的扩展信息仅适用于`SELECT`语句。对于其他可解释的语句（`DELETE`、`INSERT`、`REPLACE`和`UPDATE`），`SHOW WARNINGS`显示空结果。

这里是扩展`EXPLAIN`输出的示例：

```sql
mysql> EXPLAIN
       SELECT t1.a, t1.a IN (SELECT t2.a FROM t2) FROM t1\G
*************************** 1\. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using index
*************************** 2\. row ***************************
           id: 2
  select_type: SUBQUERY
        table: t2
         type: index
possible_keys: a
          key: a
      key_len: 5
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using index 2 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a` AS `a`,
         <in_optimizer>(`test`.`t1`.`a`,`test`.`t1`.`a` in
         ( <materialize> (/* select#2 */ select `test`.`t2`.`a`
         from `test`.`t2` where 1 having 1 ),
         <primary_index_lookup>(`test`.`t1`.`a` in
         <temporary table> on <auto_key>
         where ((`test`.`t1`.`a` = `materialized-subquery`.`a`))))) AS `t1.a
         IN (SELECT t2.a FROM t2)` from `test`.`t1` 1 row in set (0.00 sec)
```

因为`SHOW WARNINGS`显示的语句可能包含特殊标记，提供有关查询重写或优化器操作的信息，所以该语句不一定是有效的 SQL，也不打算执行。输出还可能包含具有提供有关优化器执行的其他非 SQL 解释说明的`Message`值的行。

下面的列表描述了可以出现在`SHOW WARNINGS`显示的扩展输出中的特殊标记：

+   `<auto_key>`

    临时表的自动生成键。

+   `<cache>(*`expr`*)`

    表达式（如标量子查询）只执行一次，并且结果值保存在内存中以供后续使用。对于由多个值组成的结果，可能会创建一个临时表，显示为`<temporary table>`。

+   `<exists>(*`query fragment`*)`

    子查询谓词被转换为`EXISTS`谓词，并且子查询被转换以便与`EXISTS`谓词一起使用。

+   `<in_optimizer>(*`query fragment`*)`

    这是一个内部优化器对象，对用户没有意义。

+   `<index_lookup>(*`query fragment`*)`

    使用索引查找来处理查询片段以找到符合条件的行。

+   `<if>(*`condition`*, *`expr1`*, *`expr2`*)`

    如果条件为真，则评估为*`expr1`*，否则为*`expr2`*。

+   `<is_not_null_test>(*`expr`*)`

    用于验证表达式不会评估为`NULL`的测试。

+   `<materialize>(*`query fragment`*)`

    使用子查询实现物化。

+   ``materialized-subquery`.*`col_name`*`

    对内部临时表中的*`col_name`*列的引用，该临时表用于保存子查询的结果。

+   `<primary_index_lookup>(*`query fragment`*)`

    使用主键查找来处理查询片段以找到符合条件的行。

+   `<ref_null_helper>(*`expr`*)`

    这是一个内部优化器对象，对用户没有意义。

+   `/* select#*`N`* */ *`select_stmt`*`

    `SELECT`与非扩展的`EXPLAIN`输出中`id`值为*`N`*的行相关联。

+   `*`outer_tables`* semi join (*`inner_tables`*)`

    半连接操作。*`inner_tables`*显示未被提取的表。参见 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”。

+   `<temporary table>`

    这代表一个内部临时表，用于缓存中间结果。

当某些表是`const`或`system`类型时，涉及这些表列的表达式会被优化器提前评估，并且不会显示在语句中。然而，使用`FORMAT=JSON`时，一些`const`表访问会显示为使用 const 值的`ref`访问。
