# 27.5.3 可更新和可插入视图

> 原文：[`dev.mysql.com/doc/refman/8.0/en/view-updatability.html`](https://dev.mysql.com/doc/refman/8.0/en/view-updatability.html)

一些视图是可更新的，并且对它们的引用可以用于指定在数据更改语句中要更新的表。也就是说，您可以在`UPDATE`、`DELETE`或`INSERT`等语句中使用它们来更新基础表的内容。派生表和公共表达式也可以在多表`UPDATE`和`DELETE`语句中指定，但只能用于读取数据以指定要更新或删除的行。通常，视图引用必须是可更新的，这意味着它们可以合并而不是实体化。复合视图有更复杂的规则。

要使视图可更新，视图中的行与基础表中的行之间必须是一对一的关系。还有一些其他构造使视图不可更新。更具体地说，如果视图包含以下任何内容，则视图不可更新：

+   聚合函数或窗口函数（`SUM()`、`MIN()`、`MAX()`、`COUNT()`等）

+   `DISTINCT`

+   `GROUP BY`

+   `HAVING`

+   `UNION`或`UNION ALL`

+   在选择列表中的子查询

    在选择列表中，非依赖子查询对于`INSERT`失败，但对于`UPDATE`和`DELETE`可行。对于选择列表中的依赖子查询，不允许数据更改语句。

+   某些连接（请参阅本节后面的其他连接讨论）

+   在`FROM`子句中引用不可更新的视图

+   在`FROM`子句中引用`WHERE`子句中的子查询

+   仅引用文字值（在这种情况下，没有要更新的基础表）

+   `ALGORITHM = TEMPTABLE`（始终使用临时表会使视图不可更新）

+   对基表的任何列的多次引用（对于`INSERT`失败，对于`UPDATE`和`DELETE`可行）

视图中的生成列被视为可更新，因为可以对其进行赋值。但是，如果显式更新此类列，则唯一允许的值是`DEFAULT`。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”。

有时，多表视图可能是可更新的，假设它可以使用`MERGE`算法处理。为使其工作，视图必须使用内连接（而不是外连接或`UNION`）。此外，视图定义中只能更新一个表，因此`SET`子句必须仅命名视图中一个表的列。即使理论上可更新，也不允许使用`UNION ALL`的视图。

就插入性（使用`INSERT`语句可更新）而言，如果可更新视图还满足视图列的以下附加要求，则可插入：

+   视图列名称不能重复。

+   视图必须包含基表中没有默认值的所有列。

+   视图列必须是简单的列引用。不能是表达式，比如这些：

    ```sql
    3.14159
    col1 + 3
    UPPER(col2)
    col3 / col4
    (*subquery*)
    ```

MySQL 在`CREATE VIEW`时设置一个称为视图可更新性标志的标志。如果视图对`UPDATE`和`DELETE`（以及类似操作）是合法的，则将该标志设置为`YES`（true）。否则，将该标志设置为`NO`（false）。信息模式`VIEWS`表中的`IS_UPDATABLE`列显示此标志的状态。这意味着服务器始终知道视图是否可更新。

如果视图不可更新，则`UPDATE`、`DELETE`和`INSERT`等语句是非法的并将被拒绝。（即使视图是可更新的，也可能无法插入，如本节其他地方所述。）

视图的可更新性可能会受到`updatable_views_with_limit`系统变量值的影响。请参阅第 7.1.8 节，“服务器系统变量”。

对于以下讨论，假设存在这些表和视图：

```sql
CREATE TABLE t1 (x INTEGER);
CREATE TABLE t2 (c INTEGER);
CREATE VIEW vmat AS SELECT SUM(x) AS s FROM t1;
CREATE VIEW vup AS SELECT * FROM t2;
CREATE VIEW vjoin AS SELECT * FROM vmat JOIN vup ON vmat.s=vup.c;
```

允许`INSERT`、`UPDATE`和`DELETE`语句如下：

+   `INSERT`：`INSERT`语句的插入表可以是一个合并的视图引用。如果视图是一个连接视图，则视图的所有组件必须是可更新的（非物化的）。对于多表可更新视图，如果插入到单个表，则`INSERT`可以工作。

    该语句无效，因为连接视图的一个组件是不可更新的：

    ```sql
    INSERT INTO vjoin (c) VALUES (1);
    ```

    这个语句是有效的；视图不包含实体组件：

    ```sql
    INSERT INTO vup (c) VALUES (1);
    ```

+   `UPDATE`：在`UPDATE`语句中要更新的表或表可以是合并的视图引用。如果一个视图是连接视图，视图的至少一个组件必须是可更新的（这与`INSERT`不同）。

    在多表`UPDATE`语句中，语句的更新表引用必须是基本表或可更新的视图引用。未更新的表引用可以是物化视图或派生表。

    这个语句是有效的；列`c`来自连接视图的可更新部分：

    ```sql
    UPDATE vjoin SET c=c+1;
    ```

    这个语句是无效的；列`x`来自不可更新的部分：

    ```sql
    UPDATE vjoin SET x=x+1;
    ```

    这个语句是有效的；多表`UPDATE`的更新表引用是一个可更新的视图（`vup`）：

    ```sql
    UPDATE vup JOIN (SELECT SUM(x) AS s FROM t1) AS dt ON ...
    SET c=c+1;
    ```

    这个语句是无效的；它试图更新一个实体派生表：

    ```sql
    UPDATE vup JOIN (SELECT SUM(x) AS s FROM t1) AS dt ON ...
    SET s=s+1;
    ```

+   `DELETE`：在`DELETE`语句中要从中删除的表或表必须是合并视图。不允许连接视图（这与`INSERT`和`UPDATE`不同）。

    这个语句是无效的，因为视图是一个连接视图：

    ```sql
    DELETE vjoin WHERE ...;
    ```

    这个语句是有效的，因为视图是一个合并的（可更新的）视图：

    ```sql
    DELETE vup WHERE ...;
    ```

    这个语句是有效的，因为它从一个合并的（可更新的）视图中删除：

    ```sql
    DELETE vup FROM vup JOIN (SELECT SUM(x) AS s FROM t1) AS dt ON ...;
    ```

附加讨论和示例如下。

本节早期讨论指出，如果视图不是所有列都是简单列引用（例如，如果包含表达式或复合表达式的列），则视图是不可插入的。尽管这样的视图不可插入，但如果只更新非表达式列，则可以更新。考虑这个视图：

```sql
CREATE VIEW v AS SELECT col1, 1 AS col2 FROM t;
```

这个视图不可插入，因为`col2`是一个表达式。但如果更新不尝试更新`col2`，则可以更新。这个更新是允许的：

```sql
UPDATE v SET col1 = 0;
```

此更新不允许，因为它试图更新一个表达式列：

```sql
UPDATE v SET col2 = 0;
```

如果表包含一个`AUTO_INCREMENT`列，在对不包括`AUTO_INCREMENT`列的表上插入可插入视图时，不会改变`LAST_INSERT_ID()`的值，因为插入默认值到视图中不是可见的列的副作用不应该可见。
