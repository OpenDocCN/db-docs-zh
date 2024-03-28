# 15.1.23 创建视图语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-view.html`](https://dev.mysql.com/doc/refman/8.0/en/create-view.html)

```sql
CREATE
    [OR REPLACE]
    [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}]
    [DEFINER = *user*]
    [SQL SECURITY { DEFINER | INVOKER }]
    VIEW *view_name* [(*column_list*)]
    AS *select_statement*
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```

`CREATE VIEW`语句创建一个新视图，如果给出`OR REPLACE`子句，则替换现有视图。如果视图不存在，`CREATE OR REPLACE VIEW`与`CREATE VIEW`相同。如果视图存在，`CREATE OR REPLACE VIEW`将其替换。

有关视图使用限制的信息，请参阅第 27.9 节，“视图限制”。

*`select_statement`*是一个`SELECT`语句，提供了视图的定义。（从视图中选择实际上是使用`SELECT`语句进行选择。）*`select_statement`*可以从基本表或其他视图中进行选择。从 MySQL 8.0.19 开始，`SELECT`语句可以使用`VALUES`语句作为其来源，或者可以被替换为`TABLE`语句，就像`CREATE TABLE ... SELECT`一样。

视图定义在创建时“冻结”，不受后续对基础表定义的更改的影响。例如，如果一个视图被定义为在表上`SELECT *`，那么稍后添加到表中的新列不会成为视图的一部分，而从表中删除的列在从视图中选择时会导致错误。

`ALGORITHM`子句影响 MySQL 处理视图的方式。`DEFINER`和`SQL SECURITY`子句指定在视图调用时检查访问权限时要使用的安全上下文。`WITH CHECK OPTION`子句可以用于限制对视图引用的表中的行的插入或更新。这些子句稍后在本节中描述。

`CREATE VIEW`语句需要视图的`CREATE VIEW`权限，并且对`SELECT`语句中选择的每个列都需要一些权限。对于在`SELECT`语句中的其他地方使用的列，您必须具有`SELECT`权限。如果存在`OR REPLACE`子句，则还必须具有视图的`DROP`权限。如果存在`DEFINER`子句，则所需的权限取决于*`user`*值，如第 27.6 节，“存储对象访问控制”中所讨论的那样。

当引用视图时，权限检查将按照本节后面描述的方式进行。

视图属于数据库。默认情况下，新视图将在默认数据库中创建。要在特定数据库中显式创建视图，请使用*`db_name.view_name`*语法，以数据库名称限定视图名称：

```sql
CREATE VIEW test.v AS SELECT * FROM t;
```

`SELECT`语句中的未限定表或视图名称也会根据默认数据库进行解释。视图可以通过使用适当的数据库名称限定表或视图名称来引用其他数据库中的表或视图。

在数据库中，基本表和视图共享相同的命名空间，因此基本表和视图不能具有相同的名称。

由`SELECT`语句检索的列可以是对表列的简单引用，也可以是使用函数、常量值、运算符等的表达式。

视图必须具有唯一的列名，不能有重复，就像基本表一样。默认情况下，由`SELECT`语句检索的列的名称用于视图列名。要为视图列定义显式名称，请指定可选的*`column_list`*子句作为逗号分隔的标识符列表。*`column_list`*中的名称数量必须与由`SELECT`语句检索的列的数量相同。

视图可以从许多种类的`SELECT`语句创建。它可以引用基本表或其他视图。它可以使用连接、`UNION`和子查询。`SELECT`甚至不需要引用任何表：

```sql
CREATE VIEW v_today (today) AS SELECT CURRENT_DATE;
```

以下示例定义了一个视图，从另一个表中选择了两列，以及从这些列计算出的表达式：

```sql
mysql> CREATE TABLE t (qty INT, price INT);
mysql> INSERT INTO t VALUES(3, 50);
mysql> CREATE VIEW v AS SELECT qty, price, qty*price AS value FROM t;
mysql> SELECT * FROM v;
+------+-------+-------+
| qty  | price | value |
+------+-------+-------+
|    3 |    50 |   150 |
+------+-------+-------+
```

视图定义受以下限制：

+   `SELECT`语句不能引用系统变量或用户定义的变量。

+   在存储程序中，`SELECT`语句不能引用程序参数或局部变量。

+   `SELECT`语句不能引用准备好的语句参数。

+   定义中引用的任何表或视图必须存在。如果在创建视图之后，定义引用的表或视图被删除，则使用该视图会导致错误。要检查此类问题的视图定义，请使用`CHECK TABLE`语句。

+   定义不能引用`TEMPORARY`表，也不能创建`TEMPORARY`视图。

+   不能将触发器与视图关联。

+   在`SELECT`语句中，列名的别名会被检查，其最大长度为 64 个字符（而不是最大别名长度为 256 个字符）。

视图定义中允许使用`ORDER BY`，但如果使用具有自己`ORDER BY`的语句从视图中进行选择，则会被忽略。

对于定义中的其他选项或子句，它们会被添加到引用视图的语句的选项或子句中，但效果是未定义的。例如，如果视图定义包括`LIMIT`子句，并且您使用具有自己`LIMIT`子句的语句从视图中进行选择，则未定义哪个限制适用。这个原则也适用于跟随`SELECT`关键字的`ALL`、`DISTINCT`或`SQL_SMALL_RESULT`等选项，以及诸如`INTO`、`FOR UPDATE`、`FOR SHARE`、`LOCK IN SHARE MODE`和`PROCEDURE`等子句。

如果更改查询处理环境，可能会影响从视图中获取的结果：

```sql
mysql> CREATE VIEW v (mycol) AS SELECT 'abc';
Query OK, 0 rows affected (0.01 sec)

mysql> SET sql_mode = '';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT "mycol" FROM v;
+-------+
| mycol |
+-------+
| mycol |
+-------+
1 row in set (0.01 sec)

mysql> SET sql_mode = 'ANSI_QUOTES';
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT "mycol" FROM v;
+-------+
| mycol |
+-------+
| abc   |
+-------+
1 row in set (0.00 sec)
```

`DEFINER`和`SQL SECURITY`子句确定在执行引用视图的语句时使用哪个 MySQL 账户来检查访问权限。有效的`SQL SECURITY`特性值为`DEFINER`（默认）和`INVOKER`。这表示所需的权限必须由定义或调用视图的用户持有。

如果存在`DEFINER`子句，则*`user`*值应为 MySQL 账户，指定为`'*`user_name`*'@'*`host_name`*`、`CURRENT_USER`或`CURRENT_USER()`。允许的*`user`*值取决于您拥有的权限，如第 27.6 节“存储对象访问控制”中所讨论的。还请参阅该部分以获取有关视图安全性的其他信息。

如果省略了`DEFINER`子句，则默认的定义者是执行`CREATE VIEW`语句的用户。这与明确指定`DEFINER = CURRENT_USER`相同。

在视图定义中，`CURRENT_USER`函数默认返回视图的`DEFINER`值。对于使用`SQL SECURITY INVOKER`特性定义的视图，`CURRENT_USER`返回视图调用者的账户。有关视图内用户审计的信息，请参阅第 8.2.23 节“基于 SQL 的账户活动审计”。

在使用`SQL SECURITY DEFINER`特性定义的存储过程中，`CURRENT_USER`返回该存储过程的`DEFINER`值。如果视图定义中包含`CURRENT_USER`的`DEFINER`值，这也会影响到在此类存储过程中定义的视图。

MySQL 检查视图权限的方式如下：

+   在视图定义时，视图创建者必须具有使用视图访问的顶层对象所需的权限。例如，如果视图定义引用表列，则创建者必须对定义中的每个列具有某些权限，并且对定义中其他地方使用的每个列都需要`SELECT`权限。如果定义引用了一个存储函数，则只能检查调用函数所需的权限。在函数调用时需要的权限只能在执行时检查：对于不同的调用，函数内的不同执行路径可能被采取。

+   引用视图的用户必须具有适当的权限来访问它（`SELECT`用于从中选择，`INSERT`用于插入等）。

+   当引用了一个视图时，会根据视图`DEFINER`账户或调用者持有的权限进行对象访问权限检查，具体取决于`SQL SECURITY`特性是`DEFINER`还是`INVOKER`。

+   如果引用视图导致执行存储函数，则在函数内执行的语句的权限检查取决于函数的`SQL SECURITY`特性是`DEFINER`还是`INVOKER`。如果安全特性是`DEFINER`，则函数以`DEFINER`账户的权限运行。如果特性是`INVOKER`，则函数以视图的`SQL SECURITY`特性确定的权限运行。

例如：一个视图可能依赖于一个存储函数，而该函数可能调用其他存储过程。例如，以下视图调用了一个存储函数`f()`：

```sql
CREATE VIEW v AS SELECT * FROM t WHERE t.id = f(t.name);
```

假设`f()`包含如下语句：

```sql
IF name IS NULL then
  CALL p1();
ELSE
  CALL p2();
END IF;
```

在执行`f()`时，需要检查执行语句所需的权限。这可能意味着在`f()`内部执行时需要`p1()`或`p2()`的权限，具体取决于`f()`内的执行路径。这些权限必须在运行时检查，而需要拥有这些权限的用户由视图`v`和函数`f()`的`SQL SECURITY`值确定。

视图的`DEFINER`和`SQL SECURITY`子句是标准 SQL 的扩展。在标准 SQL 中，视图使用`SQL SECURITY DEFINER`规则处理。标准规定视图的定义者，即视图模式的所有者，获得视图的适用权限（例如，`SELECT`）并可以授予它们。MySQL 没有“模式所有者”的概念，因此 MySQL 添加了一个子句来标识定义者。`DEFINER`子句是一个扩展，其目的是拥有标准的内容；也就是说，永久记录谁定义了视图。这就是为什么默认的`DEFINER`值是视图创建者的帐户。

可选的`ALGORITHM`子句是 MySQL 对标准 SQL 的扩展。它影响 MySQL 处理视图的方式。`ALGORITHM`有三个值：`MERGE`、`TEMPTABLE`或`UNDEFINED`。有关更多信息，请参见 Section 27.5.2, “View Processing Algorithms”，以及 Section 10.2.2.4, “Optimizing Derived Tables, View References, and Common Table Expressions with Merging or Materialization”。

一些视图是可更新的。也就是说，你可以在`UPDATE`、`DELETE`或`INSERT`等语句中使用它们来更新底层表的内容。要使视图可更新，视图中的行与底层表中的行之间必须是一对一的关系。还有一些其他构造使视图不可更新。

视图中的生成列被认为是可更新的，因为可以对其进行赋值。但是，如果显式更新这样的列，唯一允许的值是`DEFAULT`。有关生成列的信息，请参见 Section 15.1.20.8, “CREATE TABLE and Generated Columns”。

可以为可更新视图提供`WITH CHECK OPTION`子句，以防止插入或更新行，除非`select_statement`中的`WHERE`子句为真。

在可更新视图的`WITH CHECK OPTION`子句中，`LOCAL`和`CASCADED`关键字确定了在视图以另一个视图的形式定义时进行检查测试的范围。`LOCAL`关键字将`CHECK OPTION`限制在正在定义的视图中。`CASCADED`会导致对底层视图的检查也被评估。当没有给出关键字时，默认值为`CASCADED`。

关于可更新视图和`WITH CHECK OPTION`子句的更多信息，请参见第 27.5.3 节，“可更新和可插入视图”，以及第 27.5.4 节，“带有 CHECK OPTION 子句的视图”。
