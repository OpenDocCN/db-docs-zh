# 27.8 存储程序的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/stored-program-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/stored-program-restrictions.html)

+   存储过程中不允许的 SQL 语句

+   存储函数的限制

+   触发器的限制

+   存储过程中的名称冲突

+   复制注意事项

+   调试注意事项

+   不支持的 SQL:2003 标准语法

+   存储过程并发性注意事项

+   事件调度程序的限制

+   NDB Cluster 中的存储过程和触发器

这些限制适用于第二十七章，*存储对象*中描述的功能。

这里提到的一些限制适用于所有存储过程；即，既适用于存储过程也适用于存储函数。还有一些特定于存储函数的限制但不适用于存储过程。

存储函数的限制也适用于触发器。还有一些特定于触发器的限制。

存储过程的限制也适用于事件调度程序事件定义的`DO`子句。还有一些特定于事件的限制。

### 存储过程中不允许的 SQL 语句

存储过程不能包含任意的 SQL 语句。以下语句不允许：

+   锁定语句`LOCK TABLES`和`UNLOCK TABLES`。

+   `ALTER VIEW`。

+   `LOAD DATA` 和 `LOAD XML`。

+   SQL 准备语句（`PREPARE`，`EXECUTE`，`DEALLOCATE PREPARE`。例外是 `SIGNAL`，`RESIGNAL` 和 `GET DIAGNOSTICS`，它们不允许作为准备语句，但允许在存储程序中。

+   由于局部变量仅在存储程序执行期间处于作用域内，因此在存储程序内创建的准备语句中不允许引用它们。准备语句的作用域是当前会话，而不是存储程序，因此该语句可能在程序结束后执行，此时变量将不再处于作用域内。例如，`SELECT ... INTO *`local_var`*` 不能作为准备语句使用。此限制也适用于存储过程和函数参数。请参见 Section 15.5.1, “PREPARE Statement”。

+   在所有存储程序（存储过程和函数，触发器和事件）中，解析器将 [`BEGIN [WORK]`](commit.html "15.3.1 START TRANSACTION, COMMIT, and ROLLBACK Statements") 视为 `BEGIN ... END` 块的开始。在此上下文中开始事务，请改用 `START TRANSACTION`。

### 存储函数的限制

在存储函数中不允许以下附加语句或操作。它们在存储过程中允许，除了从存储函数或触发器内调用的存储过程。例如，如果您在存储过程中使用 `FLUSH`，则该存储过程不能从存储函数或触发器中调用。

+   执行显式或隐式提交或回滚的语句。SQL 标准不要求支持这些语句，它规定每个 DBMS 供应商可以决定是否允许它们。

+   返回结果集的语句。这包括没有`INTO *`var_list`*`子句的`SELECT`语句以及其他语句，如`SHOW`、`EXPLAIN`和`CHECK TABLE`。函数可以使用`SELECT ... INTO *`var_list`*`或使用游标和`FETCH`语句处理结果集。请参见第 15.2.13.1 节，“SELECT ... INTO Statement”和第 15.6.6 节，“游标”。

+   `FLUSH`语句。

+   存储函数不能递归使用。

+   存储函数或触发器不能修改已被调用函数或触发器的语句（用于读取或写入）中已经使用的表。

+   如果在存储函数中使用不同别名多次引用临时表，则会出现`Can't reopen table: '*`tbl_name`*`'`错误，即使引用出现在函数内的不同语句中也会发生。

+   `HANDLER ... READ`语句调用存储函数可能导致复制错误，因此不允许。

`### 触发器的限制

对于触发器，以下附加限制适用：

+   触发器不会被外键操作激活。

+   在使用基于行的复制时，副本上的触发器不会被源上发起的语句激活。在使用基于语句的复制时，副本上的触发器会被激活。有关更多信息，请参见第 19.5.1.36 节，“复制和触发器”。

+   `RETURN`语句不允许在触发器中使用，因为触发器不能返回值。要立即退出触发器，请使用`LEAVE`语句。

+   不允许在`mysql`数据库中的表上使用触发器。也不允许在`INFORMATION_SCHEMA`或`performance_schema`表上使用触发器。这些表实际上是视图，视图上不允许使用触发器。

+   触发器缓存无法检测基础对象的元数据是否发生了变化。如果触发器使用表，而表自加载触发器以来发生了变化，则触发器将使用过时的元数据运行。

### 存储例程内的名称冲突

相同的标识符可能用于例程参数、本地变量和表列。此外，相同的本地变量名称可以在嵌套块中使用。例如：

```sql
CREATE PROCEDURE p (i INT)
BEGIN
  DECLARE i INT DEFAULT 0;
  SELECT i FROM t;
  BEGIN
    DECLARE i INT DEFAULT 1;
    SELECT i FROM t;
  END;
END;
```

在这种情况下，标识符是模棱两可的，以下优先规则适用：

+   本地变量优先于例程参数或表列。

+   例程参数优先于表列。

+   内部块中的局部变量优先于外部块中的局部变量。

变量优先于表列的行为是非标准的。

### 复制考虑事项

使用存储例程可能会导致复制问题。此问题在第 27.7 节，“存储程序二进制日志记录”中进一步讨论。

`--replicate-wild-do-table=*`db_name.tbl_name`*` 选项适用于表、视图和触发器。不适用于存储过程和函数，或事件。要过滤操作后者对象的语句，请使用一个或多个 `--replicate-*-db` 选项。

### 调试考虑事项

没有存储例程调试设施。

### 来自 SQL:2003 标准的不支持语法

MySQL 存储例程语法基于 SQL:2003 标准。该标准中的以下项目目前不受支持：

+   `UNDO` 处理程序

+   `FOR` 循环

### 存储例程并发考虑事项

为了防止会话之间的交互问题，当客户端发出语句时，服务器使用可执行该语句的例程和触发器的快照。也就是说，服务器计算在执行语句期间可能使用的过程、函数和触发器列表，加载它们，然后继续执行语句。在语句执行时，它不会看到其他会话执行的例程的更改。

为了最大并发性，存储函数应最小化其副作用；特别是，在存储函数中更新表可能会减少对该表的并发操作。存储函数在执行之前获取表锁，以避免由于语句执行顺序不匹配和在日志中出现时导致二进制日志不一致。当使用基于语句的二进制日志记录时，调用函数的语句会被记录，而不是在函数内执行的语句。因此，更新相同基础表的存储函数不会并行执行。相反，存储过程不会获取表级锁。在存储过程中执行的所有语句都会写入二进制日志，即使是基于语句的二进制日志记录。参见第 27.7 节，“存储程序二进制日志记录”。

### 事件调度程序限制

以下限制特定于事件调度程序：

+   事件名称以不区分大小写的方式处理。例如，不能在同一数据库中使用名称为 `anEvent` 和 `AnEvent` 的两个事件。

+   不能在存储过程内创建事件。如果事件名称是通过变量指定的，则不能在存储过程内更改或删除事件。事件也不能创建、更改或删除存储例程或触发器。

+   在执行`LOCK TABLES`语句时，禁止对事件进行 DDL 语句。

+   使用`YEAR`、`QUARTER`、`MONTH`和`YEAR_MONTH`间隔的事件时间以月为单位解析；使用其他任何间隔的事件时间以秒为单位解析。无法使安排在同一秒执行的事件按照给定顺序执行。此外，由于四舍五入、多线程应用程序的性质以及创建事件和信号其执行所需的非零时间，事件可能会延迟至多 1 或 2 秒。然而，在信息模式`EVENTS`表的`LAST_EXECUTED`列中显示的时间始终准确到实际事件执行时间的一秒内。（另请参见 Bug #16522。）

+   事件体中包含的语句的每次执行都在一个新连接中进行；因此，这些语句对服务器的语句计数（如`Com_select`和`Com_insert`）在给定用户会话中没有影响，这些计数是通过`SHOW STATUS`显示的。然而，这些计数在全局范围内是更新的。（Bug #16422）

+   事件不支持晚于 Unix 纪元结束的时间；这大约是 2038 年初。这些日期明确不被事件调度程序允许。（Bug #16396）

+   在`CREATE EVENT`和`ALTER EVENT`语句的`ON SCHEDULE`子句中引用存储函数、可加载函数和表格是不被支持的。这类引用是不允许的。（更多信息请参见 Bug #22830。）

### NDB 集群中的存储过程和触发器

虽然`NDB`存储引擎支持表格使用存储过程、存储函数、触发器和定时事件，但你必须记住这些在充当集群 SQL 节点的 MySQL 服务器之间*不会*自动传播。这是因为存储过程和触发器定义存储在`InnoDB`表格中的`mysql`系统数据库中，这些表格在集群节点之间不会被复制。

与 MySQL Cluster 表交互的任何存储过程或触发器都必须通过在参与使用存储过程或触发器的每个 MySQL 服务器上运行适当的`CREATE PROCEDURE`、`CREATE FUNCTION`或`CREATE TRIGGER`语句来重新创建。同样，对现有存储过程或触发器的任何更改都必须在所有 Cluster SQL 节点上显式执行，使用适当的`ALTER`或`DROP`语句在访问集群的每个 MySQL 服务器上执行。

警告

*不要*尝试通过将任何`mysql`数据库表转换为使用`NDB`存储引擎来解决刚才描述的问题。*修改`mysql`数据库中的系统表不受支持*，很可能会产生不良结果。
