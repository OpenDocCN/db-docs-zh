> 译文：[`dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/subquery-restrictions.html)

#### 15.2.15.12 子查询的限制

+   一般来说，你不能修改一个表并在子查询中从同一个表中选择。例如，这个限制适用于以下形式的语句：

    ```sql
    DELETE FROM t WHERE ... (SELECT ... FROM t ...);
    UPDATE t ... WHERE col = (SELECT ... FROM t ...);
    {INSERT|REPLACE} INTO t (SELECT ... FROM t ...);
    ```

    例外：前述禁令不适用于如果对修改的表使用了派生表，并且该派生表是实现而不是合并到外部查询中。 (参见 Section 10.2.2.4, “使用合并或实现优化派生表、视图引用和公共表达式”.) 例如：

    ```sql
    UPDATE t ... WHERE col = (SELECT * FROM (SELECT ... FROM t...) AS dt ...);
    ```

    这里派生表的结果被实现为一个临时表，因此在对`t`进行更新时，相关行已经被选择。

    一般来说，你可以通过添加`NO_MERGE`优化器提示来影响优化器实现一个派生表。参见 Section 10.9.3, “优化器提示”。

+   行比较操作只部分支持：

    +   对于`*`expr`* [NOT] IN *`subquery`*`, *`expr`*可以是一个*`n`*元组（使用行构造器语法指定），子查询可以返回*`n`*元组行。因此，允许的语法更具体地表达为`*`row_constructor`* [NOT] IN *`table_subquery`*`

    +   对于`*`expr`* *`op`* {ALL|ANY|SOME} *`subquery`*`, *`expr`*必须是一个标量值，子查询必须是一个列子查询；它不能返回多列行。

    换句话说，对于返回*`n`*元组行的子查询，这是支持的：

    ```sql
    (*expr_1*, ..., *expr_n*) [NOT] IN *table_subquery*
    ```

    但是这是不支持的：

    ```sql
    (*expr_1*, ..., *expr_n*) *op* {ALL|ANY|SOME} *subquery*
    ```

    之所以支持`IN`的行比较而不支持其他操作的原因是，`IN`是通过将其重写为一系列`=`比较和`AND`操作来实现的。这种方法不能用于`ALL`、`ANY`或`SOME`。

+   在 MySQL 8.0.14 之前，`FROM`子句中的子查询不能是相关子查询。它们在查询执行期间被整体实现（评估以产生结果集），因此不能针对外部查询的每一行进行评估。优化器延迟实现直到结果需要，这可能允许避免实现。参见 Section 10.2.2.4, “使用合并或实现优化派生表、视图引用和公共表达式”。

+   MySQL 不支持在某些子查询操作符中的子查询中使用`LIMIT`：

    ```sql
    mysql> SELECT * FROM t1
           WHERE s1 IN (SELECT s2 FROM t2 ORDER BY s1 LIMIT 1);
    ERROR 1235 (42000): This version of MySQL doesn't yet support
     'LIMIT & IN/ALL/ANY/SOME subquery'
    ```

    参见第 15.2.15.10 节，“子查询错误”。

+   MySQL 允许子查询引用具有插入行等数据修改副作用的存储函数。例如，如果`f()`插入行，则以下查询可以修改数据：

    ```sql
    SELECT ... WHERE x IN (SELECT f() ...);
    ```

    这种行为是对 SQL 标准的扩展。在 MySQL 中，它可能产生不确定的结果，因为`f()`可能在给定查询的不同执行中由优化器选择处理的方式而执行不同次数。

    对于基于语句或混合格式的复制，这种不确定性的一个影响是这样的查询可能在源数据库和其副本上产生不同的结果。
