# 27.3.1 触发器语法和示例

> 原文：[`dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/trigger-syntax.html)

要创建触发器或删除触发器，请使用`CREATE TRIGGER`或`DROP TRIGGER`语句，描述在第 15.1.22 节，“CREATE TRIGGER Statement”和第 15.1.34 节，“DROP TRIGGER Statement”中。

这里有一个简单的示例，将触发器与表关联起来，以激活`INSERT`操作。 触发器充当累加器，对表的某一列插入的值进行求和。

```sql
mysql> CREATE TABLE account (acct_num INT, amount DECIMAL(10,2));
Query OK, 0 rows affected (0.03 sec)

mysql> CREATE TRIGGER ins_sum BEFORE INSERT ON account
       FOR EACH ROW SET @sum = @sum + NEW.amount;
Query OK, 0 rows affected (0.01 sec)
```

`CREATE TRIGGER`语句创建一个名为`ins_sum`的触发器，与`account`表关联。 它还包括指定触发器动作时间、触发事件以及触发器激活时要执行的操作的子句：

+   关键字`BEFORE`表示触发器动作时间。 在这种情况下，触发器在每行插入到表之前激活。 这里允许的其他关键字是`AFTER`。

+   关键字`INSERT`表示触发事件；也就是说，激活触发器的操作类型。 在示例中，`INSERT`操作会导致触发器激活。 您还可以为`DELETE`和`UPDATE`操作创建触发器。

+   `FOR EACH ROW`后面的语句定义了触发器体；也就是说，每次触发器激活时执行的语句，这发生在触发事件影响的每一行。 在示例中，触发器体是一个简单的`SET`，将插入到`amount`列中的值累积到用户变量中。 该语句将列称为`NEW.amount`，意思是“要插入新行的`amount`列的值”。

要使用触发器，请将累加器变量设置为零，执行一个`INSERT`语句，然后查看变量之后的值：

```sql
mysql> SET @sum = 0;
mysql> INSERT INTO account VALUES(137,14.98),(141,1937.50),(97,-100.00);
mysql> SELECT @sum AS 'Total amount inserted';
+-----------------------+
| Total amount inserted |
+-----------------------+
|               1852.48 |
+-----------------------+
```

在这种情况下，`INSERT`语句执行后`@sum`的值为`14.98 + 1937.50 - 100`，即`1852.48`。

要销毁触发器，请使用`DROP TRIGGER`语句。 如果触发器不在默认模式中，则必须指定模式名称：

```sql
mysql> DROP TRIGGER test.ins_sum;
```

如果删除表，则表的任何触发器也将被删除。

触发器名称存在于模式命名空间中，这意味着所有触发器在模式内必须具有唯一名称。 不同模式中的触发器可以具有相同的名称。

可以为给定表定义多个具有相同触发事件和操作时间的触发器。例如，您可以为表定义两个`BEFORE UPDATE`触发器。默认情况下，具有相同触发事件和操作时间的触发器按照它们创建的顺序激活。要影响触发器顺序，请在`FOR EACH ROW`之后指定一个子句，指示`FOLLOWS`或`PRECEDES`以及一个具有相同触发事件和操作时间的现有触发器的名称。使用`FOLLOWS`，新触发器在现有触发器之后激活。使用`PRECEDES`，新触发器在现有触发器之前激活。

例如，以下触发器定义为`account`表定义了另一个`BEFORE INSERT`触发器：

```sql
mysql> CREATE TRIGGER ins_transaction BEFORE INSERT ON account
       FOR EACH ROW PRECEDES ins_sum
       SET
       @deposits = @deposits + IF(NEW.amount>0,NEW.amount,0),
       @withdrawals = @withdrawals + IF(NEW.amount<0,-NEW.amount,0);
Query OK, 0 rows affected (0.01 sec)
```

此触发器`ins_transaction`类似于`ins_sum`，但分别累积存款和取款。它具有一个`PRECEDES`子句，使其在`ins_sum`之前激活；如果没有该子句，它将在`ins_sum`之后激活，因为它是在`ins_sum`之后创建的。

在触发器主体内，`OLD`和`NEW`关键字使您能够访问触发器影响的行中的列。`OLD`和`NEW`是 MySQL 触发器的扩展；它们不区分大小写。

在`INSERT`触发器中，只能使用`NEW.*`col_name`*`；没有旧行。在`DELETE`触发器中，只能使用`OLD.*`col_name`*`；没有新行。在`UPDATE`触发器中，您可以使用`OLD.*`col_name`*`来引用更新前行的列，使用`NEW.*`col_name`*`来引用更新后行的列。

以`OLD`命名的列是只读的。您可以引用它（如果您具有`SELECT`权限），但不能修改它。如果您对其具有`SELECT`权限，则可以引用以`NEW`命名的列。在`BEFORE`触发器中，如果您对其具有`UPDATE`权限，则还可以使用`SET NEW.*`col_name`* = *`value`*`来更改其值。这意味着您可以使用触发器修改要插入新行或用于更新行的值。（在`AFTER`触发器中，这样的`SET`语句没有效果，因为行更改已经发生。）

在`BEFORE`触发器中，`AUTO_INCREMENT`列的`NEW`值为 0，而不是在实际插入新行时自动生成的序列号。

通过使用`BEGIN ... END`结构，可以定义执行多个语句的触发器。在`BEGIN`块内，还可以使用其他在存储过程中允许的语法，如条件和循环。然而，就像对于存储过程一样，如果使用**mysql**程序定义执行多个语句的触发器，则需要重新定义**mysql**语句定界符，以便在触发器定义内使用`;`语句定界符。以下示例说明了这些要点。它定义了一个`UPDATE`触发器，检查要用于更新每行的新值，并修改值使其在 0 到 100 的范围内。这必须是`BEFORE`触发器，因为必须在使用该值更新行之前检查该值：

```sql
mysql> delimiter //
mysql> CREATE TRIGGER upd_check BEFORE UPDATE ON account
       FOR EACH ROW
       BEGIN
           IF NEW.amount < 0 THEN
               SET NEW.amount = 0;
           ELSEIF NEW.amount > 100 THEN
               SET NEW.amount = 100;
           END IF;
       END;//
mysql> delimiter ;
```

将存储过程单独定义，然后使用简单的`CALL`语句从触发器中调用它可能更容易。如果要从多个触发器内执行相同的代码，这也是有利的。

触发器执行时，触发器执行的语句中出现的内容有限制：

+   触发器不能使用`CALL`语句调用返回数据给客户端或使用动态 SQL 的存储过程。（存储过程可以通过`OUT`或`INOUT`参数将数据返回给触发器。）

+   触发器不能使用明确或隐式开始或结束事务的语句，例如`START TRANSACTION`，`COMMIT`或`ROLLBACK`。(`ROLLBACK to SAVEPOINT`是允许的，因为它不会结束事务。)。

另请参阅 Section 27.8, “存储程序的限制”。

MySQL 在触发器执行期间处理错误如下：

+   如果`BEFORE`触发器失败，则不会执行对应行的操作。

+   `BEFORE`触发器在*尝试*插入或修改行时被激活，无论尝试是否成功。

+   仅当任何`BEFORE`触发器和行操作成功执行时，才会执行`AFTER`触发器。

+   `BEFORE`或`AFTER`触发器中的错误导致触发器调用的整个语句失败。

+   对于事务性表，语句失败应导致语句执行的所有更改回滚。触发器失败会导致语句失败，因此触发器失败也会导致回滚。对于非事务性表，无法进行此类回滚，因此尽管语句失败，但在错误点之前执行的任何更改仍然有效。

触发器可以通过名称直接引用表，例如此示例中显示的名为`testref`的触发器：

```sql
CREATE TABLE test1(a1 INT);
CREATE TABLE test2(a2 INT);
CREATE TABLE test3(a3 INT NOT NULL AUTO_INCREMENT PRIMARY KEY);
CREATE TABLE test4(
  a4 INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
  b4 INT DEFAULT 0
);

delimiter |

CREATE TRIGGER testref BEFORE INSERT ON test1
  FOR EACH ROW
  BEGIN
    INSERT INTO test2 SET a2 = NEW.a1;
    DELETE FROM test3 WHERE a3 = NEW.a1;
    UPDATE test4 SET b4 = b4 + 1 WHERE a4 = NEW.a1;
  END;
|

delimiter ;

INSERT INTO test3 (a3) VALUES
  (NULL), (NULL), (NULL), (NULL), (NULL),
  (NULL), (NULL), (NULL), (NULL), (NULL);

INSERT INTO test4 (a4) VALUES
  (0), (0), (0), (0), (0), (0), (0), (0), (0), (0);
```

假设您将以下数值插入到表`test1`中，如下所示：

```sql
mysql> INSERT INTO test1 VALUES 
       (1), (3), (1), (7), (1), (8), (4), (4);
Query OK, 8 rows affected (0.01 sec)
Records: 8  Duplicates: 0  Warnings: 0
```

结果，四个表包含以下数据：

```sql
mysql> SELECT * FROM test1;
+------+
| a1   |
+------+
|    1 |
|    3 |
|    1 |
|    7 |
|    1 |
|    8 |
|    4 |
|    4 |
+------+
8 rows in set (0.00 sec)

mysql> SELECT * FROM test2;
+------+
| a2   |
+------+
|    1 |
|    3 |
|    1 |
|    7 |
|    1 |
|    8 |
|    4 |
|    4 |
+------+
8 rows in set (0.00 sec)

mysql> SELECT * FROM test3;
+----+
| a3 |
+----+
|  2 |
|  5 |
|  6 |
|  9 |
| 10 |
+----+
5 rows in set (0.00 sec)

mysql> SELECT * FROM test4;
+----+------+
| a4 | b4   |
+----+------+
|  1 |    3 |
|  2 |    0 |
|  3 |    1 |
|  4 |    2 |
|  5 |    0 |
|  6 |    0 |
|  7 |    1 |
|  8 |    1 |
|  9 |    0 |
| 10 |    0 |
+----+------+
10 rows in set (0.00 sec)
```
