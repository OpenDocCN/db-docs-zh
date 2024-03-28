> 原文：[`dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html`](https://dev.mysql.com/doc/refman/8.0/en/insert-on-duplicate.html)

#### 15.2.7.2 INSERT ... ON DUPLICATE KEY UPDATE Statement

如果您指定了一个`ON DUPLICATE KEY UPDATE`子句，并且要插入的行会导致在`UNIQUE`索引或`PRIMARY KEY`中出现重复值，则会更新旧行。例如，如果列`a`被声明为`UNIQUE`并包含值`1`，则以下两个语句具有类似的效果：

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3)
  ON DUPLICATE KEY UPDATE c=c+1;

UPDATE t1 SET c=c+1 WHERE a=1;
```

效果并不完全相同：对于`InnoDB`表，其中`a`是自增列，`INSERT`语句会增加自增值，但`UPDATE`不会。

如果列`b`也是唯一的，那么`INSERT`等同于这个`UPDATE`语句：

```sql
UPDATE t1 SET c=c+1 WHERE a=1 OR b=2 LIMIT 1;
```

如果`a=1 OR b=2`匹配多行，只会更新*一行*。一般来说，应尽量避免在具有多个唯一索引的表上使用`ON DUPLICATE KEY UPDATE`子句。

使用`ON DUPLICATE KEY UPDATE`，每行的受影响行数为 1，如果该行被插入为新行，则为 2，如果更新现有行，则为 0。如果在连接到**mysqld**时，通过在`mysql_real_connect()` C API 函数中指定`CLIENT_FOUND_ROWS`标志，受影响行数为 1（而不是 0），如果将现有行设置为其当前值。

如果表中包含一个`AUTO_INCREMENT`列，并且`INSERT ... ON DUPLICATE KEY UPDATE`插入或更新一行，则`LAST_INSERT_ID()`函数将返回`AUTO_INCREMENT`值。

`ON DUPLICATE KEY UPDATE`子句可以包含多个列赋值，用逗号分隔。

在`ON DUPLICATE KEY UPDATE`子句中的赋值表达式中，您可以使用`VALUES(*`col_name`*)`函数引用`INSERT`部分的列值。换句话说，在`ON DUPLICATE KEY UPDATE`子句中，`VALUES(*`col_name`*)`指的是如果没有发生重复键冲突，将要插入的*`col_name`*的值。这个函数在多行插入中特别有用。`VALUES()`函数只在`ON DUPLICATE KEY UPDATE`子句或`INSERT`语句中有意义，否则返回`NULL`。例如：

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3),(4,5,6)
  ON DUPLICATE KEY UPDATE c=VALUES(a)+VALUES(b);
```

该语句与以下两个语句相同：

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3)
  ON DUPLICATE KEY UPDATE c=3;
INSERT INTO t1 (a,b,c) VALUES (4,5,6)
  ON DUPLICATE KEY UPDATE c=9;
```

注意

从 MySQL 8.0.20 开始，使用`VALUES()`来引用新行和列已经不推荐，并且可能在未来的 MySQL 版本中被移除。相反，使用行和列别名，如本节接下来几段所述。

从 MySQL 8.0.19 开始，可以使用行别名，可选地，在`VALUES`或`SET`子句后面插入一个或多个列，并在`AS`关键字之前。使用行别名`new`，之前使用`VALUES()`访问新列值的语句可以以这种形式编写：

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3),(4,5,6) AS new
  ON DUPLICATE KEY UPDATE c = new.a+new.b;
```

如果此外，你使用列别名`m`、`n`和`p`，你可以在赋值子句中省略行别名，并像这样编写相同的语句：

```sql
INSERT INTO t1 (a,b,c) VALUES (1,2,3),(4,5,6) AS new(m,n,p)
  ON DUPLICATE KEY UPDATE c = m+n;
```

在这种情况下使用列别名时，即使在赋值子句中没有直接使用它，你仍然必须在`VALUES`子句后面使用行别名。

从 MySQL 8.0.20 开始，使用`UPDATE`子句中的`VALUES()`的`INSERT ... SELECT ... ON DUPLICATE KEY UPDATE`语句会发出警告：

```sql
INSERT INTO t1
  SELECT c, c+d FROM t2
  ON DUPLICATE KEY UPDATE b = VALUES(b);
```

你可以通过使用子查询来消除此类警告，如下所示：

```sql
INSERT INTO t1
  SELECT * FROM (SELECT c, c+d AS e FROM t2) AS dt
  ON DUPLICATE KEY UPDATE b = e;
```

你也可以在`SET`子句中使用行和列别名，如前面提到的。刚刚显示的两个`INSERT ... ON DUPLICATE KEY UPDATE`语句中使用`SET`而不是`VALUES`可以像这样完成：

```sql
INSERT INTO t1 SET a=1,b=2,c=3 AS new
  ON DUPLICATE KEY UPDATE c = new.a+new.b;

INSERT INTO t1 SET a=1,b=2,c=3 AS new(m,n,p)
  ON DUPLICATE KEY UPDATE c = m+n;
```

行别名不能与表名相同。如果不使用列别名，或者如果它们与列名相同，则必须在`ON DUPLICATE KEY UPDATE`子句中使用行别名来区分它们。列别名必须与它们应用的行别名唯一（即，不得有指向同一行列的相同列别名）。

对于`INSERT ... SELECT`语句，在`ON DUPLICATE KEY UPDATE`子句中可以引用的`SELECT`查询表达式的可接受形式如下：

+   引用单个表查询中的列，该表可能是一个派生表。

+   引用跨多个表连接的查询中的列。

+   引用`DISTINCT`查询中的列。

+   引用其他表中的列，只要`SELECT`没有使用`GROUP BY`。一个副作用是你必须限定对非唯一列名的引用。

不支持从`UNION`中引用列。为了解决这个限制，将`UNION`重写为派生表，以便其行可以被视为单表结果集。例如，这个语句会产生错误：

```sql
INSERT INTO t1 (a, b)
  SELECT c, d FROM t2
  UNION
  SELECT e, f FROM t3
ON DUPLICATE KEY UPDATE b = b + c;
```

相反，使用等效语句将`UNION`重写为派生表：

```sql
INSERT INTO t1 (a, b)
SELECT * FROM
  (SELECT c, d FROM t2
   UNION
   SELECT e, f FROM t3) AS dt
ON DUPLICATE KEY UPDATE b = b + c;
```

将查询重写为派生表的技术还可以使`GROUP BY`查询中的列引用成为可能。

因为`INSERT ... SELECT`语句的结果取决于`SELECT`中行的排序，而这种顺序并不总是能够保证，所以在记录源和副本的`INSERT ... SELECT ON DUPLICATE KEY UPDATE`语句时可能会出现分歧。因此，当使用基于语句的复制时，`INSERT ... SELECT ON DUPLICATE KEY UPDATE`语句被标记为不安全。这种语句在使用基于语句的模式时会在错误日志中产生警告，并在使用`MIXED`模式时以基于行的格式写入二进制日志。对于针对具有多个唯一或主键的表的`INSERT ... ON DUPLICATE KEY UPDATE`语句也被标记为不安全。（Bug #11765650, Bug #58637）

参见 Section 19.2.1.1, “基于语句和基于行的复制的优缺点”。
