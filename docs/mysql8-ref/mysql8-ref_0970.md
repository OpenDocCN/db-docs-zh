# 15.2.17 UPDATE 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/update.html`](https://dev.mysql.com/doc/refman/8.0/en/update.html)

`UPDATE`是修改表中行的 DML 语句。

一个`UPDATE`语句可以以`WITH`")子句开头，以定义在`UPDATE`中可访问的常用表达式。请参阅第 15.2.20 节，“WITH (Common Table Expressions)”")。

单表语法：

```sql
UPDATE [LOW_PRIORITY] [IGNORE] *table_reference*
    SET *assignment_list*
    [WHERE *where_condition*]
    [ORDER BY ...]
    [LIMIT *row_count*]

*value*:
    {*expr* | DEFAULT}

*assignment*:
    *col_name* = *value*

*assignment_list*:
    *assignment* [, *assignment*] ...
```

多表语法：

```sql
UPDATE [LOW_PRIORITY] [IGNORE] *table_references*
    SET *assignment_list*
    [WHERE *where_condition*]
```

对于单表语法，`UPDATE`语句使用新值更新命名表中现有行的列。`SET`子句指示要修改的列以及它们应该被赋予的值。每个值可以作为表达式给出，或者使用关键字`DEFAULT`将列明确设置为其默认值。如果给出`WHERE`子句，则指定标识要更新的行的条件。如果没有`WHERE`子句，则更新所有行。如果指定了`ORDER BY`子句，则按指定的顺序更新行。`LIMIT`子句限制可以更新的行数。

对于多表语法，`UPDATE`更新满足条件的每个表中的行。每个匹配的行只更新一次，即使它多次匹配条件。对于多表语法，不能使用`ORDER BY`和`LIMIT`。

对于分区表，此语句的单表和多表形式都支持`PARTITION`子句作为表引用的一部分。此选项接受一个或多个分区或子分区（或两者）的列表。仅检查列出的分区（或子分区）是否匹配，并且不在任何这些分区或子分区中的行不会被更新，无论它是否满足*`where_condition`*。

注意

与在`INSERT`或`REPLACE`语句中使用`PARTITION`时不同，即使列出的分区（或子分区）中没有行与*`where_condition`*匹配，否则有效的`UPDATE ... PARTITION`语句也被认为是成功的。

有关更多信息和示例，请参见第 26.5 节，“分区选择”。

*`where_condition`*是一个对每个要更新的行求值为 true 的表达式。有关表达式语法，请参见第 11.5 节，“表达式”。

*`table_references`*和*`where_condition`*的指定如第 15.2.13 节，“SELECT 语句”所述。

只有在实际上被更新的`UPDATE`中引用的列才需要`UPDATE`权限。对于只读取但不修改的任何列，只需要`SELECT`权限。

`UPDATE` 语句支持以下修饰符：

+   使用`LOW_PRIORITY`修饰符，`UPDATE` 的执行会延迟，直到没有其他客户端从表中读取数据。这仅影响仅使用表级锁定的存储引擎（如`MyISAM`，`MEMORY`和`MERGE`）。

+   使用`IGNORE`修饰符，即使在更新过程中发生错误，更新语句也不会中止。对于唯一键值上发生重复键冲突的行不会被更新。将更新为会导致数据转换错误的值的行将被更新为最接近的有效值。更多信息，请参见 The Effect of IGNORE on Statement Execution。

`UPDATE IGNORE` 语句，包括具有`ORDER BY`子句的语句，被标记为不安全的用于基于语句的复制。（这是因为更新行的顺序决定了哪些行被忽略。）这样的语句在使用基于语句模式时会在错误日志中产生警告，并在使用`MIXED`模式时以基于行的格式写入二进制日志。（Bug #11758262, Bug #50439）更多信息请参见 Section 19.2.1.3, “Determination of Safe and Unsafe Statements in Binary Logging”。

如果在表达式中访问要更新的表中的列，`UPDATE` 会使用列的当前值。例如，以下语句将`col1`设置为比其当前值多一的值：

```sql
UPDATE t1 SET col1 = col1 + 1;
```

以下语句中的第二个赋值将`col2`设置为当前（更新后）的`col1`值，而不是原始的`col1`值。结果是`col1`和`col2`具有相同的值。这种行为与标准 SQL 不同。

```sql
UPDATE t1 SET col1 = col1 + 1, col2 = col1;
```

单表`UPDATE`赋值通常从左到右进行评估。对于多表更新，不能保证赋值按任何特定顺序执行。

如果将列设置为其当前值，MySQL 会注意到这一点并且不会对其进行更新。

如果您通过将已声明为`NOT NULL`的列设置为`NULL`来更新列，并且启用了严格的 SQL 模式，则会发生错误；否则，该列将设置为列数据类型的隐式默认值，并且警告计数会增加。对于数值类型，隐式默认值为`0`，对于字符串类型，为空字符串（`''`），对于日期和时间类型，为“零”值。请参见第 13.6 节，“数据类型默认值”。

如果显式更新生成列，则唯一允许的值是`DEFAULT`。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”。

`UPDATE`返回实际更改的行数。`mysql_info()` C API 函数返回匹配并更新的行数以及在`UPDATE`过程中发生的警告数。

您可以使用`LIMIT *`row_count`*`来限制`UPDATE`的范围。`LIMIT`子句是一个匹配行数的限制。一旦找到满足`WHERE`子句的*`row_count`*行，无论它们是否实际更改，语句都会停止。

如果`UPDATE`语句包含`ORDER BY`子句，则按照子句指定的顺序更新行。在某些情况下，这可能会避免错误。假设表`t`包含具有唯一索引的列`id`。以下语句可能会因为更新行的顺序不同而导致重复键错误：

```sql
UPDATE t SET id = id + 1;
```

例如，如果表中`id`列包含 1 和 2，并且在将 1 更新为 2 之前将 2 更新为 3，则会发生错误。为避免此问题，请添加`ORDER BY`子句，以使具有较大`id`值的行在具有较小值的行之前更新：

```sql
UPDATE t SET id = id + 1 ORDER BY id DESC;
```

您还可以执行涵盖多个表的`UPDATE`操作。但是，不能在多表`UPDATE`中使用`ORDER BY`或`LIMIT`。*`table_references`*子句列出了参与连接的表。其语法在第 15.2.13.2 节，“JOIN Clause”中描述。以下是一个示例：

```sql
UPDATE items,month SET items.price=month.price
WHERE items.id=month.id;
```

前面的示例显示了使用逗号运算符的内连接，但是多表`UPDATE`语句可以使用在`SELECT`语句中允许的任何类型的连接，例如`LEFT JOIN`。

如果您使用涉及`InnoDB`表的多表`UPDATE`语句，并且这些表存在外键约束，MySQL 优化器可能会以与它们的父/子关系不同的顺序处理表。在这种情况下，该语句将失败并回滚。相反，更新单个表，并依赖`InnoDB`提供的`ON UPDATE`功能，以使其他表相应地进行修改。请参阅第 15.1.20.5 节，“外键约束”。

您不能在子查询中直接更新表并从同一表中进行选择。您可以通过使用多表更新来解决此问题，其中一个表是从您实际希望更新的表派生的，并使用别名引用派生表。假设您希望更新一个名为`items`的表，该表是使用以下语句定义的：

```sql
CREATE TABLE items (
    id BIGINT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    wholesale DECIMAL(6,2) NOT NULL DEFAULT 0.00,
    retail DECIMAL(6,2) NOT NULL DEFAULT 0.00,
    quantity BIGINT NOT NULL DEFAULT 0
);
```

要减少任何标记幅度为 30%或更高且库存少于一百的任何商品的零售价格，您可以尝试使用类似以下的`UPDATE`语句，其中在`WHERE`子句中使用了子查询。如下所示，此语句不起作用：

```sql
mysql> UPDATE items
     > SET retail = retail * 0.9
     > WHERE id IN
     >     (SELECT id FROM items
     >         WHERE retail / wholesale >= 1.3 AND quantity > 100);
ERROR 1093 (HY000): You can't specify target table 'items' for update in FROM clause
```

相反，您可以使用多表更新，其中子查询移动到要更新的表列表中，并使用别名在最外层`WHERE`子句中引用它，就像这样：

```sql
UPDATE items,
       (SELECT id FROM items
        WHERE id IN
            (SELECT id FROM items
             WHERE retail / wholesale >= 1.3 AND quantity < 100))
        AS discounted
SET items.retail = items.retail * 0.9
WHERE items.id = discounted.id;
```

因为优化器默认尝试将派生表`discounted`合并到最外层查询块中，所以只有在强制实体化派生表时才起作用。您可以通过在运行更新之前将`optimizer_switch`系统变量的`derived_merge`标志设置为`off`，或者使用`NO_MERGE`优化提示来实现这一点，如下所示：

```sql
UPDATE /*+ NO_MERGE(discounted) */ items,
       (SELECT id FROM items
        WHERE retail / wholesale >= 1.3 AND quantity < 100)
        AS discounted
    SET items.retail = items.retail * 0.9
    WHERE items.id = discounted.id;
```

在这种情况下使用优化提示的优势在于它仅在使用它的查询块内部应用，因此在执行`UPDATE`后不需要再次更改`optimizer_switch`的值。

另一种可能性是重写子查询，使其不使用`IN`或`EXISTS`，就像这样：

```sql
UPDATE items,
       (SELECT id, retail / wholesale AS markup, quantity FROM items)
       AS discounted
    SET items.retail = items.retail * 0.9
    WHERE discounted.markup >= 1.3
    AND discounted.quantity < 100
    AND items.id = discounted.id;
```

在这种情况下，默认情况下子查询是实体化的，而不是合并的，因此不需要禁用派生表的合并。
