# 11.2.2 标识符限定符

> 原文：[`dev.mysql.com/doc/refman/8.0/en/identifier-qualifiers.html`](https://dev.mysql.com/doc/refman/8.0/en/identifier-qualifiers.html)

对象名称可以是未限定的或限定的。在名称解释明确的情况下，可以使用未限定名称。限定名称包括至少一个限定符，以通过覆盖默认上下文或提供缺失上下文来澄清解释上下文。

例如，此语句使用未限定名称`t1`创建表：

```sql
CREATE TABLE t1 (i INT);
```

因为`t1`不包含指定数据库的限定符，该语句在默认数据库中创建表。如果没有默认数据库，则会出现错误。

此语句使用限定名`db1.t1`创建表：

```sql
CREATE TABLE db1.t1 (i INT);
```

因为`db1.t1`包含数据库限定符`db1`，该语句在名为`db1`的数据库中创建`t1`，而不管默认数据库如何。如果没有默认数据库，则必须指定限定符。如果有默认数据库，则可以指定限定符，以指定与默认数据库不同的数据库，或者如果默认数据库与指定的数据库相同，则明确指定数据库。

限定符具有以下特点：

+   未限定名称由单个标识符组成。限定名称由多个标识符组成。

+   多部分名称的组件必须用句点（`.`）字符分隔。多部分名称的初始部分充当限定符，影响解释最终标识符的上下文。

+   限定符字符是一个单独的标记，不需要与相关标识符连续。例如，*`tbl_name.col_name`*和*`tbl_name . col_name`*是等效的。

+   如果多部分名称的任何组件需要引号，请单独引用它们，而不是整体引用名称。例如，写成``my-table`.`my-column``，而不是``my-table.my-column``。

+   在限定名中跟在句点后面的保留字必须是标识符，因此在该上下文中不需要引号。

对象名称的允许限定符取决于对象类型：

+   数据库名称是完全限定的，不需要限定符：

    ```sql
    CREATE DATABASE db1;
    ```

+   表、视图或存储程序名称可以给定数据库名称限定符。在`CREATE`语句中的未限定和限定名称示例：

    ```sql
    CREATE TABLE mytable ...;
    CREATE VIEW myview ...;
    CREATE PROCEDURE myproc ...;
    CREATE FUNCTION myfunc ...;
    CREATE EVENT myevent ...;

    CREATE TABLE mydb.mytable ...;
    CREATE VIEW mydb.myview ...;
    CREATE PROCEDURE mydb.myproc ...;
    CREATE FUNCTION mydb.myfunc ...;
    CREATE EVENT mydb.myevent ...;
    ```

+   触发器与表相关联，因此任何限定符都适用于表名：

    ```sql
    CREATE TRIGGER mytrigger ... ON mytable ...;

    CREATE TRIGGER mytrigger ... ON mydb.mytable ...;
    ```

+   列名可以给定多个限定符，以指示在引用它的语句中的上下文，如下表所示。

    | 列引用 | 含义 |
    | --- | --- |
    | *`col_name`* | 语句中使用的任何表中包含具有该名称的列的列*`col_name`* |
    | *`tbl_name.col_name`* | 来自默认数据库表*`tbl_name`*的列*`col_name`* |
    | *`db_name.tbl_name.col_name`* | 来自数据库*`db_name`*的表*`tbl_name`*的列*`col_name`* |

    换句话说，列名可以被赋予表名限定符，而表名本身可以被赋予数据库名限定符。在`SELECT`语句中的未限定和限定列引用的示例：

    ```sql
    SELECT c1 FROM mytable
    WHERE c2 > 100;

    SELECT mytable.c1 FROM mytable
    WHERE mytable.c2 > 100;

    SELECT mydb.mytable.c1 FROM mydb.mytable
    WHERE mydb.mytable.c2 > 100;
    ```

除非未限定引用是模棱两可的，否则在语句中不需要为对象引用指定限定符。假设列`c1`仅出现在表`t1`中，`c2`仅在`t2`中，`c`在`t1`和`t2`中都有。在引用这两个表的语句中，对`c`的任何未限定引用都是模棱两可的，必须限定为`t1.c`或`t2.c`以指示你指的是哪个表：

```sql
SELECT c1, c2, t1.c FROM t1 INNER JOIN t2
WHERE t2.c > 100;
```

同样地，在同一语句中从数据库`db1`的表`t`和数据库`db2`的表`t`中检索数据时，必须对表引用进行限定：对于这些表中的列名的引用，只有在两个表中都出现的列名需要限定。假设列`c1`仅出现在表`db1.t`中，`c2`仅在`db2.t`中，`c`在`db1.t`和`db2.t`中都有。在这种情况下，`c`是模棱两可的，必须进行限定，但`c1`和`c2`则不需要：

```sql
SELECT c1, c2, db1.t.c FROM db1.t INNER JOIN db2.t
WHERE db2.t.c > 100;
```

表别名使得可以更简单地编写限定列引用：

```sql
SELECT c1, c2, t1.c FROM db1.t AS t1 INNER JOIN db2.t AS t2
WHERE t2.c > 100;
```
