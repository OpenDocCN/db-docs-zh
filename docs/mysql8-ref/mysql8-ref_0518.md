> 原文：[`dev.mysql.com/doc/refman/8.0/en/engine-condition-pushdown-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/engine-condition-pushdown-optimization.html)

#### 10.2.1.5 引擎条件下推优化

这种优化提高了非索引列与常量之间直接比较的效率。在这种情况下，条件被“下推”到存储引擎进行评估。这种优化只能由`NDB`存储引擎使用。

对于 NDB Cluster，这种优化可以消除在集群的数据节点和发出查询的 MySQL 服务器之间发送不匹配行的需要，并且可以加快使用的查询速度，速度提高了 5 到 10 倍，超过了可以但未使用条件下推的情况。

假设一个 NDB Cluster 表定义如下：

```sql
CREATE TABLE t1 (
    a INT,
    b INT,
    KEY(a)
) ENGINE=NDB;
```

引擎条件下推可以与如下所示的查询一起使用，其中包括非索引列与常量之间的比较：

```sql
SELECT a, b FROM t1 WHERE b = 10;
```

引擎条件下推的使用可以在`EXPLAIN`的输出中看到：

```sql
mysql> EXPLAIN SELECT a, b FROM t1 WHERE b = 10\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 10
        Extra: Using where with pushed condition
```

然而，引擎条件下推 *不能* 与以下查询一起使用：

```sql
SELECT a,b FROM t1 WHERE a = 10;
```

引擎条件下推在这里不适用，因为列 `a` 上存在索引。（索引访问方法更有效，因此会选择索引访问方法而不是条件下推。）

当索引列使用 `>` 或 `<` 运算符与常量进行比较时，也可以使用引擎条件下推：

```sql
mysql> EXPLAIN SELECT a, b FROM t1 WHERE a < 2\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: t1
         type: range
possible_keys: a
          key: a
      key_len: 5
          ref: NULL
         rows: 2
        Extra: Using where with pushed condition
```

引擎条件下推的其他支持比较包括以下内容：

+   `*`column`* [NOT] LIKE *`pattern`*`

    *`pattern`* 必须是包含要匹配的模式的字符串文字；有关语法，请参见第 14.8.1 节，“字符串比较函数和运算符”。

+   `*`column`* IS [NOT] NULL`

+   `*`column`* IN (*`value_list`*)`

    *`value_list`* 中的每个项目必须是常量、文字值。

+   `*`column`* BETWEEN *`constant1`* AND *`constant2`*`

    *`constant1`* 和 *`constant2`* 必须是常量、文字值。

在前述列表中的所有情况中，条件可以转换为一个或多个列与常量之间的直接比较形式。

引擎条件下推默认启用。要在服务器启动时禁用它，请将`optimizer_switch`系统变量的`engine_condition_pushdown`标志设置为 `off`。例如，在 `my.cnf` 文件中，使用以下行：

```sql
[mysqld]
optimizer_switch=engine_condition_pushdown=off
```

在运行时，像这样禁用条件下推：

```sql
SET optimizer_switch='engine_condition_pushdown=off';
```

**限制。** 引擎条件下推受以下限制：

+   引擎条件下推仅受支持`NDB`存储引擎。

+   在 NDB 8.0.18 之前，列只能与常量或计算为常量值的表达式进行比较。在 NDB 8.0.18 及更高版本中，只要它们具有完全相同的类型，包括相同的符号、长度、字符集、精度和比例，这些都适用，列就可以相互比较。

+   用于比较的列不能是任何`BLOB`或`TEXT`类型。这个排除范围还包括`JSON`、`BIT`和`ENUM`列。

+   与列进行比较的字符串值必须使用与列相同的排序规则。

+   不支持直接连接；尽可能将涉及多个表的条件分开推送。使用扩展的`EXPLAIN`输出来确定实际被推送的条件。参见 Section 10.8.3, “Extended EXPLAIN Output Format”。

以前，引擎条件推送仅限于引用来自正在被推送条件的相同表的列值的术语。从 NDB 8.0.16 开始，还可以从查询计划中较早的表中引用推送条件的列值。这减少了 SQL 节点在连接处理期间必须处理的行数。过滤也可以在 LDM 线程中并行执行，而不是在单个**mysqld**进程中。这有可能显着提高查询的性能。

从 NDB 8.0.20 开始，如果在相同连接嵌套中使用的任何表上没有无法推送的条件，或者在其上依赖的连接嵌套中没有无法推送的条件，则可以推送使用扫描的外连接。对于半连接也是如此，前提是所采用的优化策略是`firstMatch`（参见 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”）。

在以下两种情况下，连接算法不能与引用先前表的列结合使用：

1.  当任何被引用的先前表在连接缓冲区中时。在这种情况下，从扫描过滤表中检索的每一行都与缓冲区中的每一行匹配。这意味着在生成扫描过滤时无法从单个特定行中获取列值。

1.  当列来自推送连接中的子操作时。这是因为在生成扫描过滤时，尚未检索到连接中祖先操作引用的行。

从 NDB 8.0.27 开始，可以将连接中祖先表的列下推，前提是它们符合先前列出的要求。以下是一个使用先前创建的表 `t1` 的查询示例：

```sql
mysql> EXPLAIN 
 ->   SELECT * FROM t1 AS x 
 ->   LEFT JOIN t1 AS y 
 ->   ON x.a=0 AND y.b>=3\G
*************************** 1\. row ***************************
           id: 1
  select_type: SIMPLE
        table: x
   partitions: p0,p1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: NULL
*************************** 2\. row ***************************
           id: 1
  select_type: SIMPLE
        table: y
   partitions: p0,p1
         type: ALL
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using where; Using pushed condition (`test`.`y`.`b` >= 3); Using join buffer (hash join) 2 rows in set, 2 warnings (0.00 sec)
```
