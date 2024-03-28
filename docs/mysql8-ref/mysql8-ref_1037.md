> 原文：[`dev.mysql.com/doc/refman/8.0/en/diagnostics-area.html`](https://dev.mysql.com/doc/refman/8.0/en/diagnostics-area.html)

#### 15.6.7.7 MySQL 诊断区

SQL 语句生成填充诊断区的诊断信息。标准 SQL 具有诊断区堆栈，每个嵌套执行上下文都包含一个诊断区。标准 SQL 还支持`GET STACKED DIAGNOSTICS`语法，用于在条件处理程序执行期间引用第二个诊断区。

以下讨论描述了 MySQL 中诊断区的结构，MySQL 识别的信息项，语句如何清除和设置诊断区，以及诊断区如何推送到堆栈并从堆栈中弹出。

+   诊断区结构

+   诊断区信息项

+   诊断区如何清除和填充

+   诊断区堆栈的工作原理

+   与诊断区相关的系统变量

##### 诊断区结构

诊断区包含两种信息：

+   语句信息，例如发生的条件数量或受影响行数。

+   条件信息，例如错误代码和消息。如果语句引发多个条件，则诊断区的此部分为每个条件区域都有一个条件区域。如果语句未引发任何条件，则诊断区的此部分为空。

对于生成三个条件的语句，诊断区包含如下语句和条件信息：

```sql
Statement information:
  row count
  ... other statement information items ...
Condition area list:
  Condition area 1:
    error code for condition 1
    error message for condition 1
    ... other condition information items ...
  Condition area 2:
    error code for condition 2:
    error message for condition 2
    ... other condition information items ...
  Condition area 3:
    error code for condition 3
    error message for condition 3
    ... other condition information items ...
```

##### 诊断区信息项

诊断区包含语句和条件信息项。数值项为整数。字符项的字符集为 UTF-8。没有任何项可以是`NULL`。如果语句未设置填充诊断区的语句或条件项，则其值为 0 或空字符串，取决于项的数据类型。

诊断区的语句信息部分包含以下内容：

+   `NUMBER`: 一个整数，表示具有信息的条件区域数量。

+   `ROW_COUNT`: 一个整数，表示语句影响的行数。`ROW_COUNT`与`ROW_COUNT()`函数的值相同（参见第 14.15 节，“信息函数”）。

诊断区域的条件信息部分包含每个条件的条件区域。条件区域从 1 到`NUMBER`语句条件项的值编号。如果`NUMBER`为 0，则没有条件区域。

每个条件区域包含以下列表中的项目。所有项目都是标准 SQL，除了`MYSQL_ERRNO`，它是 MySQL 的扩展。这些定义适用于除信号（即由`SIGNAL`或`RESIGNAL`语句生成的条件之外的条件。对于非信号条件，MySQL 仅填充未描述为始终为空的那些条件项。信号对条件区域的影响稍后描述。

+   `CLASS_ORIGIN`：包含`RETURNED_SQLSTATE`值的类的字符串。如果`RETURNED_SQLSTATE`值以 SQL 标准文档 ISO 9075-2（第 24.1 节，SQLSTATE）中定义的类值开头，则`CLASS_ORIGIN`为`'ISO 9075'`。否则，`CLASS_ORIGIN`为`'MySQL'`。

+   `SUBCLASS_ORIGIN`：包含`RETURNED_SQLSTATE`值的子类的字符串。如果`CLASS_ORIGIN`为`'ISO 9075'`或`RETURNED_SQLSTATE`以`'000'`结尾，则`SUBCLASS_ORIGIN`为`'ISO 9075'`。否则，`SUBCLASS_ORIGIN`为`'MySQL'`。

+   `RETURNED_SQLSTATE`：指示条件的`SQLSTATE`值的字符串。

+   `MESSAGE_TEXT`：指示条件的错误消息的字符串。

+   `MYSQL_ERRNO`：指示条件的 MySQL 错误代码的整数。

+   `CONSTRAINT_CATALOG`、`CONSTRAINT_SCHEMA`、`CONSTRAINT_NAME`：指示违反约束的目录、模式和名称的字符串。它们始终为空。

+   `CATALOG_NAME`、`SCHEMA_NAME`、`TABLE_NAME`、`COLUMN_NAME`：指示与条件相关的目录、模式、表和列的字符串。它们始终为空。

+   `CURSOR_NAME`：指示游标名称的字符串。这始终为空。

有关特定错误的`RETURNED_SQLSTATE`、`MESSAGE_TEXT`和`MYSQL_ERRNO`值，请参阅服务器错误消息参考。

如果`SIGNAL`（或`RESIGNAL`）语句填充诊断区域，则其`SET`子句可以为除`RETURNED_SQLSTATE`之外的任何条件信息项分配合法的数据类型值。`SIGNAL`还设置`RETURNED_SQLSTATE`值，但不是直接在其`SET`子句中。该值来自`SIGNAL`语句的`SQLSTATE`参数。

`SIGNAL`还设置语句信息项。它将`NUMBER`设置为 1。对于错误，它将`ROW_COUNT`设置为−1，否则为 0。

##### 诊断区域如何清除和填充

非诊断性 SQL 语句会自动填充诊断区域，并且其内容可以通过`SIGNAL`和`RESIGNAL`语句明确设置。可以使用`GET DIAGNOSTICS`提取特定项来检查诊断区域，或者使用`SHOW WARNINGS`或`SHOW ERRORS`来查看条件或错误。

SQL 语句如下清除和设置诊断区域：

+   当服务器开始执行解析后的语句时，它会清除非诊断性语句的诊断区域。诊断性语句不会清除诊断区域。这些语句是诊断性的：

    +   `GET DIAGNOSTICS`

    +   `SHOW ERRORS`

    +   `SHOW WARNINGS`

+   如果一个语句引发条件，那么诊断区域将清除属于先前语句的条件。唯一的例外是由`GET DIAGNOSTICS`和`RESIGNAL`引发的条件会被添加到诊断区域而不清除它。

因此，即使一个语句在开始执行时通常不清除诊断区域，但如果该语句引发条件，则会清除它。

以下示例展示了各种语句对诊断区域的影响，使用`SHOW WARNINGS`显示存储在其中的条件信息。

这个`DROP TABLE`语句在条件发生时清除诊断区域并填充它：

```sql
mysql> DROP TABLE IF EXISTS test.no_such_table;
Query OK, 0 rows affected, 1 warning (0.01 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------------+
| Level | Code | Message                            |
+-------+------+------------------------------------+
| Note  | 1051 | Unknown table 'test.no_such_table' |
+-------+------+------------------------------------+
1 row in set (0.00 sec)
```

这个`SET`语句生成一个错误，因此它会清除并填充诊断区域：

```sql
mysql> SET @x = @@x;
ERROR 1193 (HY000): Unknown system variable 'x' 
mysql> SHOW WARNINGS;
+-------+------+-----------------------------+
| Level | Code | Message                     |
+-------+------+-----------------------------+
| Error | 1193 | Unknown system variable 'x' |
+-------+------+-----------------------------+
1 row in set (0.00 sec)
```

先前的`SET`语句产生了一个条件，因此在这一点上，1 是唯一有效的`GET DIAGNOSTICS`条件号。以下语句使用条件号为 2，这会产生一个警告，该警告被添加到诊断区域而不清除它：

```sql
mysql> GET DIAGNOSTICS CONDITION 2 @p = MESSAGE_TEXT;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> SHOW WARNINGS;
+-------+------+------------------------------+
| Level | Code | Message                      |
+-------+------+------------------------------+
| Error | 1193 | Unknown system variable 'xx' |
| Error | 1753 | Invalid condition number     |
+-------+------+------------------------------+
2 rows in set (0.00 sec)
```

现在诊断区域中有两个条件，因此相同的`GET DIAGNOSTICS`语句成功执行：

```sql
mysql> GET DIAGNOSTICS CONDITION 2 @p = MESSAGE_TEXT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @p;
+--------------------------+
| @p                       |
+--------------------------+
| Invalid condition number |
+--------------------------+
1 row in set (0.01 sec)
```

##### 诊断区域栈的工作原理

当诊断区域栈发生推送时，第一个（当前的）诊断区域变为第二个（堆叠的）诊断区域，并创建一个新的当前诊断区域作为其副本。诊断区域在以下情况下被推送到栈中并从栈中弹出：

+   执行存储程序

    程序执行前发生推送，执行后发生弹出。如果存储程序在处理程序执行时结束，则可能有多个要弹出的诊断区；这是由于没有适当处理程序的异常或处理程序中的`返回`引起的。

    弹出的诊断区中的任何警告或错误条件都会添加到当前诊断区中，但对于触发器，只会添加错误。当存储程序结束时，调用者会在其当前诊断区中看到这些条件。

+   在存储程序中执行条件处理程序

    当由于条件处理程序激活而发生推送时，堆栈诊断区是在推送之前存储程序中的当前区域。新的当前诊断区现在是处理程序的当前诊断区。[`获取[当前]诊断`](get-diagnostics.html "15.6.7.3 获取诊断语句")和`获取堆叠诊断`可以在处理程序中使用，以访问当前（处理程序）和堆叠（存储程序）诊断区的内容。最初，它们返回相同的结果，但在处理程序中执行的语句会修改当前诊断区，根据正常规则清除和设置其内容（参见诊断区如何清除和填充）。堆叠诊断区不能被处理程序中执行的语句修改，除非使用`重新发出`。

    如果处理程序成功执行，则当前（处理程序）诊断区将被弹出，堆叠（存储程序）诊断区再次成为当前诊断区。在处理程序执行期间添加到处理程序诊断区的条件将被添加到当前诊断区。

+   执行`重新发出`

    `重新发出`语句传递在存储程序内部复合语句中执行条件处理程序期间可用的错误条件信息。`重新发出`可能在传递之前更改一些或所有信息，根据第 15.6.7.4 节，“重新发出语句”中描述的方式修改诊断堆栈。

##### 与诊断区相关的系统变量

某些系统变量控制或与诊断区的某些方面相关：

+   `max_error_count` 控制诊断区域中条件区域的数量。如果发生的条件超过这个数量，MySQL 会悄悄地丢弃多余条件的信息。（通过 `RESIGNAL` 添加的条件始终会被添加，旧条件会根据需要被丢弃以腾出空间。）

+   `warning_count` 表示发生的条件数量。这包括错误、警告和注释。通常情况下，`NUMBER` 和 `warning_count` 是相同的。然而，当生成的条件数量超过 `max_error_count` 时，`warning_count` 的值会继续上升，而 `NUMBER` 保持在 `max_error_count` 上限，因为诊断区域中不会存储额外的条件。

+   `error_count` 表示发生的错误数量。这个值包括“未找到”和异常条件，但不包括警告和注释。与 `warning_count` 类似，它的值可以超过 `max_error_count`。

+   如果 `sql_notes` 系统变量设置为 0，则不会存储注释，也不会增加 `warning_count`。

例如：如果 `max_error_count` 是 10，诊断区域最多可以包含 10 个条件区域。假设一条语句引发了 20 个条件，其中有 12 个错误。在这种情况下，诊断区域包含前 10 个条件，`NUMBER` 是 10，`warning_count` 是 20，`error_count` 是 12。

对 `max_error_count` 的更改在下一次尝试修改诊断区域时才会生效。如果诊断区域包含 10 个条件区域，而 `max_error_count` 设置为 5，这对诊断区域的大小或内容没有立即影响。
