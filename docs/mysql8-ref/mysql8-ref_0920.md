# 15.1.22 CREATE TRIGGER Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-trigger.html`](https://dev.mysql.com/doc/refman/8.0/en/create-trigger.html)

```sql
CREATE
    [DEFINER = *user*]
    TRIGGER [IF NOT EXISTS] *trigger_name*
    *trigger_time* *trigger_event*
    ON *tbl_name* FOR EACH ROW
    [*trigger_order*]
    *trigger_body*

*trigger_time*: { BEFORE | AFTER }

*trigger_event*: { INSERT | UPDATE | DELETE }

*trigger_order*: { FOLLOWS | PRECEDES } *other_trigger_name*
```

此语句创建一个新触发器。触发器是与表关联的命名数据库对象，当表发生特定事件时激活。触发器与名为 *`tbl_name`* 的表关联，该表必须引用永久表。您不能将触发器与 `TEMPORARY` 表或视图关联。

触发器名称存在于模式命名空间中，这意味着所有触发器在模式内必须具有唯一名称。不同模式中的触发器可以具有相同的名称。

`IF NOT EXISTS` 可以防止在同一模式中存在具有相同名称、在同一表上的触发器时发生错误。此选项从 MySQL 8.0.29 开始支持。

本节描述了 `CREATE TRIGGER` 语法。有关更多讨论，请参见 第 27.3.1 节“触发器语法和示例”。

`CREATE TRIGGER` 需要与触发器关联的表的 `TRIGGER` 权限。如果存在 `DEFINER` 子句，则所需的权限取决于 *`user`* 值，如 第 27.6 节“存储对象访问控制” 中所讨论的。如果启用了二进制日志记录，则 `CREATE TRIGGER` 可能需要 `SUPER` 权限，如 第 27.7 节“存储程序二进制日志记录” 中所述。

`DEFINER` 子句确定在触发器激活时用于检查访问权限的安全上下文，如本节后面所述。

*`trigger_time`* 是触发器动作时间。它可以是 `BEFORE` 或 `AFTER`，表示触发器在修改每行之前或之后激活。

在触发器激活之前会进行基本列值检查，因此您不能使用 `BEFORE` 触发器将不适合列类型的值转换为有效值。

*`trigger_event`* 表示激活触发器的操作类型。这些 *`trigger_event`* 值是允许的：

+   `INSERT`: 当向表中插入新行时触发器会激活（例如，通过 `INSERT`、`LOAD DATA` 和 `REPLACE` 语句）。

+   `UPDATE`: 当行被修改时触发器会激活（例如，通过 `UPDATE` 语句）。

+   `DELETE`：该触发器在从表中删除行时激活（例如，通过`DELETE`和`REPLACE`语句）。对表进行`DROP TABLE`和`TRUNCATE TABLE`操作不会激活该触发器，因为它们不使用`DELETE`。删除分区也不会激活`DELETE`触发器。

*`trigger_event`* 不代表激活触发器的 SQL 语句类型，而更多地代表一种表操作类型。例如，一个`INSERT`触发器不仅激活`INSERT`语句，还激活`LOAD DATA`语句，因为这两个语句都向表中插入行。

这种情况可能会令人困惑的一个例子是`INSERT INTO ... ON DUPLICATE KEY UPDATE ...`语法：一个`BEFORE INSERT`触发器为每一行激活，接着是一个`AFTER INSERT`触发器或者`BEFORE UPDATE`和`AFTER UPDATE`触发器，具体取决于该行是否存在重复键。

注意

级联外键操作不会触发触发器。

可以为给定表定义多个具有相同触发事件和动作时间的触发器。例如，可以为表定义两个`BEFORE UPDATE`触发器。默认情况下，具有相同触发事件和动作时间的触发器按照它们创建的顺序激活。要影响触发器顺序，请指定一个*`trigger_order`*子句，指示`FOLLOWS`或`PRECEDES`以及一个同样具有相同触发事件和动作时间的现有触发器的名称。使用`FOLLOWS`，新触发器在现有触发器之后激活。使用`PRECEDES`，新触发器在现有触发器之前激活。

*`trigger_body`* 是触发器激活时要执行的语句。要执行多个语句，请使用`BEGIN ... END`复合语句结构。这还使您能够使用存储过程中允许的相同语句。请参阅 Section 15.6.1, “BEGIN ... END Compound Statement”。某些语句在触发器中不允许使用；请参阅 Section 27.8, “Restrictions on Stored Programs”。

在触发器主体中，您可以通过使用别名`OLD`和`NEW`引用主题表（与触发器关联的表）中的列。`OLD.*`col_name`*`指的是更新或删除之前现有行的列。`NEW.*`col_name`*`指的是要插入的新行或更新后的现有行的列。

触发器不能使用`NEW.*`col_name`*`或使用`OLD.*`col_name`*`来引用生成列。有关生成列的信息，请参见第 15.1.20.8 节，“CREATE TABLE and Generated Columns”。

MySQL 在创建触发器时存储`sql_mode`系统变量设置，并始终以此设置执行触发器主体，*无论触发器开始执行时当前服务器 SQL 模式如何*。

`DEFINER`子句指定在触发器激活时检查访问权限时要使用的 MySQL 帐户。如果存在`DEFINER`子句，则*`user`*值应为指定为`'*`user_name`*'@'*`host_name`*'`、`CURRENT_USER`或`CURRENT_USER()`的 MySQL 帐户。允许的*`user`*值取决于您拥有的权限，如第 27.6 节“存储对象访问控制”中所讨论的。还请参阅该部分以获取有关触发器安全性的其他信息。

如果省略`DEFINER`子句，则默认定义者是执行`CREATE TRIGGER`语句的用户。这与明确指定`DEFINER = CURRENT_USER`相同。

MySQL 在检查触发器权限时考虑`DEFINER`用户如下：

+   在`CREATE TRIGGER`时，发出该语句的用户必须具有`TRIGGER`权限。

+   在触发器激活时，权限将针对`DEFINER`用户进行检查。此用户必须具有以下权限：

    +   主题表的`TRIGGER`权限。

    +   如果触发器主体中使用`OLD.*`col_name`*`或`NEW.*`col_name`*`引用表列，则需要对主题表具有`SELECT`权限。

    +   如果表列是触发器主体中`SET NEW.*`col_name`* = *`value`*`的目标，则需要对主题表具有`UPDATE`权限。

    +   触发器执行的语句通常需要的其他权限。

在触发器体内，`CURRENT_USER`函数返回在触发器激活时用于检查权限的账户。这是`DEFINER`用户，而不是触发器激活的用户。有关触发器内用户审计的信息，请参阅 Section 8.2.23, “SQL-Based Account Activity Auditing”。

如果你使用`LOCK TABLES`来锁定一个带有触发器的表，那么触发器中使用的表也会被锁定，就像在 LOCK TABLES and Triggers 中描述的那样。

欲了解更多关于触发器使用的讨论，请参阅 Section 27.3.1, “Trigger Syntax and Examples”。
