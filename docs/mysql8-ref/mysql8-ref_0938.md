# 15.2.2 DELETE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/delete.html`](https://dev.mysql.com/doc/refman/8.0/en/delete.html)

`DELETE`是一条从表中删除行的 DML 语句。

`DELETE`语句可以以`WITH`")子句开头，以定义在`DELETE`内可访问的公共表达式。请参阅第 15.2.20 节，“WITH (Common Table Expressions)”")。

#### 单表语法

```sql
DELETE [LOW_PRIORITY] [QUICK] [IGNORE] FROM *tbl_name* [[AS] *tbl_alias*]
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [WHERE *where_condition*]
    [ORDER BY ...]
    [LIMIT *row_count*]
```

`DELETE`语句从*`tbl_name`*中删除行并返回已删除行的数量。要检查已删除行的数量，请调用第 14.15 节，“信息函数”中描述的`ROW_COUNT()`函数。

#### 主要子句

可选的`WHERE`子句中的条件标识要删除的行。如果没有`WHERE`子句，则删除所有行。

*`where_condition`*是一个表达式，对于要删除的每一行都计算为 true。它的指定方式如第 15.2.13 节，“SELECT 语句”中描述的那样。

如果指定了`ORDER BY`子句，则按指定的顺序删除行。`LIMIT`子句限制可以删除的行数。这些子句适用于单表删除，但不适用于多表删除。

#### 多表语法

```sql
DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
    *tbl_name*[.*] [, *tbl_name*[.*]] ...
    FROM *table_references*
    [WHERE *where_condition*]

DELETE [LOW_PRIORITY] [QUICK] [IGNORE]
    FROM *tbl_name*[.*] [, *tbl_name*[.*]] ...
    USING *table_references*
    [WHERE *where_condition*]
```

#### 权限

您需要在表上具有`DELETE`权限才能从中删除行。对于仅读取的列（例如`WHERE`子句中命名的列），您只需要具有`SELECT`权限。

#### 性能

当您不需要知道已删除行的数量时，`TRUNCATE TABLE`语句是比没有`WHERE`子句的`DELETE`语句更快地清空表的方法。与`DELETE`不同，`TRUNCATE TABLE`不能在事务内使用，也不能在表上加锁。请参阅第 15.1.37 节，“TRUNCATE TABLE 语句”和第 15.3.6 节，“LOCK TABLES 和 UNLOCK TABLES 语句”。

删除操作的速度也可能受到第 10.2.5.3 节，“优化 DELETE 语句”中讨论的因素的影响。

为确保给定的`DELETE`语句不会花费太多时间，MySQL 特定的`LIMIT *`row_count`*`子句用于`DELETE`指定要删除的最大行数。如果要删除的行数大于限制，则重复`DELETE`语句，直到受影响的行数小于`LIMIT`值。

#### 子查询

你不能在子查询中从一个表中删除并从同一个表中选择。

#### 分区表支持

`DELETE`支持使用`PARTITION`子句进行显式分区选择，该子句接受一个逗号分隔的一个或多个分区或子分区（或两者）的名称列表，从中选择要删除的行。未包含在列表中的分区将被忽略。给定一个具有名为`p0`的分区的分区表`t`，执行语句`DELETE FROM t PARTITION (p0)`对表具有与执行`ALTER TABLE t TRUNCATE PARTITION (p0)`相同的效果；在这两种情况下，分区`p0`中的所有行都将被删除。

`PARTITION`可以与`WHERE`条件一起使用，在这种情况下，条件仅在列出的分区中的行上进行测试。例如，`DELETE FROM t PARTITION (p0) WHERE c < 5`仅删除条件`c < 5`为真的分区`p0`中的行；任何其他分区中的行都不会被检查，因此不受`DELETE`影响。

`PARTITION`子句也可以在多表`DELETE`语句中使用。您可以在`FROM`选项中命名的每个表中使用最多一个此类选项。

有关更多信息和示例，请参阅 Section 26.5, “Partition Selection”。

#### 自增列

如果删除包含`AUTO_INCREMENT`列的最大值的行，则对于`MyISAM`或`InnoDB`表，该值不会被重用。如果在`autocommit`模式下使用`DELETE FROM *`tbl_name`*`（没有`WHERE`子句）从表中删除所有行，则除了`InnoDB`和`MyISAM`之外的所有存储引擎都会重新开始序列。关于`InnoDB`表的此行为有一些例外情况，如 Section 17.6.1.6, “AUTO_INCREMENT Handling in InnoDB”中所讨论的。

对于`MyISAM`表，您可以在多列键中指定一个`AUTO_INCREMENT`辅助列。在这种情况下，即使对于`MyISAM`表，从序列顶部删除的值也会被重用。请参阅 Section 5.6.9, “Using AUTO_INCREMENT”获取更多信息和示例。

#### 修饰符

`DELETE`语句支持以下修饰符：

+   如果指定了`LOW_PRIORITY`修饰符，服务器会延迟执行`DELETE`，直到没有其他客户端从表中读取数据。这仅影响只使用表级锁定的存储引擎（如`MyISAM`，`MEMORY`和`MERGE`）。

+   对于`MyISAM`表，如果使用`QUICK`修饰符，存储引擎在删除期间不会合并索引叶子，这可能会加快某些类型的删除操作。

+   `IGNORE`修饰符导致 MySQL 在删除行的过程中忽略可忽略的错误。（在解析阶段遇到的错误会按照通常的方式处理。）由于使用`IGNORE`而被忽略的错误会作为警告返回。有关更多信息，请参阅 IGNORE 对语句执行的影响。

#### 删除顺序

如果`DELETE`语句包括`ORDER BY`子句，则按照子句指定的顺序删除行。这主要与`LIMIT`结合使用。例如，以下语句查找与`WHERE`子句匹配的行，按`timestamp_column`排序，并删除第一行（最旧的行）：

```sql
DELETE FROM somelog WHERE user = 'jcole'
ORDER BY timestamp_column LIMIT 1;
```

`ORDER BY`还有助于按照所需顺序删除行，以避免引用完整性违规。

#### InnoDB 表

如果要从大表中删除许多行，可能会超出`InnoDB`表的锁表大小。为避免此问题，或者仅仅为了最小化表保持锁定的时间，以下策略（完全不使用`DELETE`）可能会有所帮助：

1.  将*不*要删除的行选择到一个与原始表具有相同结构的空表中：

    ```sql
    INSERT INTO t_copy SELECT * FROM t WHERE ... ;
    ```

1.  使用`RENAME TABLE`原子性地将原始表移开并将副本重命名为原始名称：

    ```sql
    RENAME TABLE t TO t_old, t_copy TO t;
    ```

1.  删除原始表：

    ```sql
    DROP TABLE t_old;
    ```

在执行`RENAME TABLE`期间，没有其他会话可以访问涉及的表，因此重命名操作不会受到并发问题的影响。请参阅 Section 15.1.36, “RENAME TABLE Statement”。

#### MyISAM 表

在`MyISAM`表中，删除的行会保留在一个链表中，随后的`INSERT`操作会重用旧的行位置。为了回收未使用的空间并减小文件大小，使用`OPTIMIZE TABLE`语句或**myisamchk**实用程序重新组织表。`OPTIMIZE TABLE`更容易使用，但**myisamchk**更快。请参阅 Section 15.7.3.4, “OPTIMIZE TABLE Statement”和 Section 6.6.4, “myisamchk — MyISAM Table-Maintenance Utility”。

`QUICK`修饰符影响删除操作时是否合并索引叶子。`DELETE QUICK`对于删除的行的索引值被后续插入的类似索引值替换的应用程序最有用。在这种情况下，被删除值留下的空洞会被重用。

当删除的值导致跨越新插入发生的索引值范围的索引块不足时，`DELETE QUICK`是无用的。在这种情况下，使用`QUICK`可能会导致索引中的浪费空间无法回收。以下是这种情况的一个示例：

1.  创建一个包含带有索引的`AUTO_INCREMENT`列的表。

1.  插入多行到表中。每次插入都会导致一个索引值被添加到索引的高端。

1.  使用`DELETE QUICK`删除列范围的低端的一块行。

在这种情况下，与被删除的索引值相关联的索引块变得不足，但由于使用了`QUICK`，它们不会与其他索引块合并。当新的插入发生时，它们仍然不足，因为新行没有在被删除范围内的索引值。此外，即使稍后使用`DELETE`而不使用`QUICK`，它们仍然保持不足，除非一些被删除的索引值恰好位于在不足块内或相邻块内的索引块中。在这些情况下，为了回收未使用的索引空间，请使用`OPTIMIZE TABLE`。

如果要从表中删除许多行，使用`DELETE QUICK`后跟`OPTIMIZE TABLE`可能更快。这将重建索引，而不是执行许多索引块合并操作。

#### 多表删除

您可以在`DELETE`语句中指定多个表，根据`WHERE`子句中的条件从一个或多个表中删除行。在多表`DELETE`语句中不能使用`ORDER BY`或`LIMIT`。*`table_references`*子句列出了参与连接的表，如第 15.2.13.2 节“JOIN 子句”中所述。

对于第一个多表语法，只删除在`FROM`子句之前列出的表中匹配的行。对于第二个多表语法，只删除在`FROM`子句（在`USING`子句之前）中列出的表中匹配的行。其效果是您可以同时从多个表中删除行，并且还可以使用仅用于搜索的其他表：

```sql
DELETE t1, t2 FROM t1 INNER JOIN t2 INNER JOIN t3
WHERE t1.id=t2.id AND t2.id=t3.id;
```

或：

```sql
DELETE FROM t1, t2 USING t1 INNER JOIN t2 INNER JOIN t3
WHERE t1.id=t2.id AND t2.id=t3.id;
```

这些语句在搜索要删除的行时使用了所有三个表，但只从表`t1`和`t2`中删除匹配的行。

前面的示例使用了`INNER JOIN`，但多表`DELETE`语句可以使用在`SELECT`语句中允许的其他类型的连接，例如`LEFT JOIN`。例如，要删除在`t1`中存在但在`t2`中没有匹配的行，请使用`LEFT JOIN`：

```sql
DELETE t1 FROM t1 LEFT JOIN t2 ON t1.id=t2.id WHERE t2.id IS NULL;
```

该语法允许在每个*`tbl_name`*后面使用`.*`以与**Access**兼容。

如果您使用涉及具有外键约束的`InnoDB`表的多表`DELETE`语句，MySQL 优化器可能以与其父/子关系不同的顺序处理表。在这种情况下，语句将失败并回滚。相反，您应该从单个表中删除，并依赖`InnoDB`提供的`ON DELETE`功能来相应地修改其他表。

注意

如果为表声明了别名，则在引用该表时必须使用该别名：

```sql
DELETE t1 FROM test AS t1, test2 WHERE ...
```

多表`DELETE`语句中的表别名应仅在语句的*`table_references`*部分中声明。在其他地方，允许别名引用，但不允许别名声明。

正确：

```sql
DELETE a1, a2 FROM t1 AS a1 INNER JOIN t2 AS a2
WHERE a1.id=a2.id;

DELETE FROM a1, a2 USING t1 AS a1 INNER JOIN t2 AS a2
WHERE a1.id=a2.id;
```

错误：

```sql
DELETE t1 AS a1, t2 AS a2 FROM t1 INNER JOIN t2
WHERE a1.id=a2.id;

DELETE FROM t1 AS a1, t2 AS a2 USING t1 INNER JOIN t2
WHERE a1.id=a2.id;
```

从 MySQL 8.0.16 开始，还支持单表`DELETE`语句的表别名。 (Bug #89410,Bug #27455809)
