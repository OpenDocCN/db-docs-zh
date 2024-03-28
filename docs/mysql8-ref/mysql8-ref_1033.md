> 原文：[`dev.mysql.com/doc/refman/8.0/en/get-diagnostics.html`](https://dev.mysql.com/doc/refman/8.0/en/get-diagnostics.html)

#### 15.6.7.3 获取诊断信息语句

```sql
GET [CURRENT | STACKED] DIAGNOSTICS {
    *statement_information_item*
    [, *statement_information_item*] ...
  | CONDITION *condition_number*
    *condition_information_item*
    [, *condition_information_item*] ...
}

*statement_information_item*:
    *target* = *statement_information_item_name*

*condition_information_item*:
    *target* = *condition_information_item_name*

*statement_information_item_name*: {
    NUMBER
  | ROW_COUNT
}

*condition_information_item_name*: {
    CLASS_ORIGIN
  | SUBCLASS_ORIGIN
  | RETURNED_SQLSTATE
  | MESSAGE_TEXT
  | MYSQL_ERRNO
  | CONSTRAINT_CATALOG
  | CONSTRAINT_SCHEMA
  | CONSTRAINT_NAME
  | CATALOG_NAME
  | SCHEMA_NAME
  | TABLE_NAME
  | COLUMN_NAME
  | CURSOR_NAME
}

*condition_number*, *target*:
    (see following discussion)
```

SQL 语句生成填充诊断区域的诊断信息。`获取诊断信息`语句使应用程序能够检查这些信息。（您还可以使用`显示警告`或`显示错误`来查看条件或错误。）

执行`获取诊断信息`不需要特殊权限。

关键字`CURRENT`表示从当前诊断区域检索信息。关键字`STACKED`表示从第二诊断区域检索信息，仅当当前上下文为条件处理程序时才可用。如果未给出任何关键字，则默认使用当前诊断区域。

`获取诊断信息`语句通常在存储程序内的处理程序中使用。这是 MySQL 的一个扩展，允许在处理程序上下文之外使用[`获取[当前]诊断信息`](get-diagnostics.html "15.6.7.3 获取诊断信息语句")来检查任何 SQL 语句的执行。例如，如果调用**mysql**客户端程序，则可以在提示符下输入这些语句：

```sql
mysql> DROP TABLE test.no_such_table;
ERROR 1051 (42S02): Unknown table 'test.no_such_table'
mysql> GET DIAGNOSTICS CONDITION 1
         @p1 = RETURNED_SQLSTATE, @p2 = MESSAGE_TEXT;
mysql> SELECT @p1, @p2;
+-------+------------------------------------+
| @p1   | @p2                                |
+-------+------------------------------------+
| 42S02 | Unknown table 'test.no_such_table' |
+-------+------------------------------------+
```

此扩展仅适用于当前诊断区域。它不适用于第二诊断区域，因为只有在当前上下文为条件处理程序时才允许使用`获取堆叠诊断信息`。如果不是这种情况，则会发生`处理程序未激活时获取堆叠诊断信息`错误。

有关诊断区域的描述，请参阅第 15.6.7.7 节，“MySQL 诊断区域”。简而言之，它包含两种信息：

+   语句信息，如发生的条件数或受影响行数。

+   条件信息，如错误代码和消息。如果语句引发多个条件，则诊断区域的此部分为每个条件区域。如果语句未引发任何条件，则诊断区域的此部分为空。

对于产生三个条件的语句，诊断区域包含如下语句和条件信息：

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

`获取诊断信息`可以获取语句或条件信息，但不能在同一语句中同时获取两者：

+   要获取语句信息，请将所需的语句项检索到目标变量中。此`获取诊断信息`实例将可用条件数和受影响行数分配给用户变量`@p1`和`@p2`：

    ```sql
    GET DIAGNOSTICS @p1 = NUMBER, @p2 = ROW_COUNT;
    ```

+   要获取条件信息，请指定条件编号，并将所需的条件项检索到目标变量中。`GET DIAGNOSTICS`的这个实例将 SQLSTATE 值和错误消息分配给用户变量`@p3`和`@p4`：

    ```sql
    GET DIAGNOSTICS CONDITION 1
      @p3 = RETURNED_SQLSTATE, @p4 = MESSAGE_TEXT;
    ```

检索列表指定一个或多个`*`target`* = *`item_name`*`赋值，用逗号分隔。每个赋值命名一个目标变量，要么是*`statement_information_item_name`*，要么是*`condition_information_item_name`*标识符，取决于语句检索语句还是条件信息。

用于存储项目信息的有效*`target`*标识符可以是存储过程或函数参数，使用`DECLARE`声明的存储程序本地变量，或用户定义变量。

有效的*`condition_number`*标识符可以是存储过程或函数参数，使用`DECLARE`声明的存储程序本地变量，用户定义变量，系统变量或文字。如果条件编号不在具有信息的条件区域数量范围内，则会发出警告。在这种情况下，警告将添加到诊断区域而不清除它。

当发生条件时，MySQL 不会填充`GET DIAGNOSTICS`识别的所有条件项。例如：

```sql
mysql> GET DIAGNOSTICS CONDITION 1
         @p5 = SCHEMA_NAME, @p6 = TABLE_NAME;
mysql> SELECT @p5, @p6;
+------+------+
| @p5  | @p6  |
+------+------+
|      |      |
+------+------+
```

在标准 SQL 中，如果存在多个条件，则第一个条件与前一个 SQL 语句返回的`SQLSTATE`值相关。在 MySQL 中，这并不保证。要获取主要错误，不能这样做：

```sql
GET DIAGNOSTICS CONDITION 1 @errno = MYSQL_ERRNO;
```

相反，首先检索条件计数，然后使用它指定要检查的条件编号：

```sql
GET DIAGNOSTICS @cno = NUMBER;
GET DIAGNOSTICS CONDITION @cno @errno = MYSQL_ERRNO;
```

有关允许的语句和条件信息项以及在发生条件时哪些信息项被填充的信息，请参阅诊断区域信息项。

这是一个在存储过程上下文中使用`GET DIAGNOSTICS`和异常处理程序来评估插入操作结果的示例。如果插入成功，该过程使用`GET DIAGNOSTICS`获取受影响行数。这表明只要当前诊断区域未被清除，您可以多次使用`GET DIAGNOSTICS`来检索有关语句的信息。

```sql
CREATE PROCEDURE do_insert(value INT)
BEGIN
  -- Declare variables to hold diagnostics area information
  DECLARE code CHAR(5) DEFAULT '00000';
  DECLARE msg TEXT;
  DECLARE nrows INT;
  DECLARE result TEXT;
  -- Declare exception handler for failed insert
  DECLARE CONTINUE HANDLER FOR SQLEXCEPTION
    BEGIN
      GET DIAGNOSTICS CONDITION 1
        code = RETURNED_SQLSTATE, msg = MESSAGE_TEXT;
    END;

  -- Perform the insert
  INSERT INTO t1 (int_col) VALUES(value);
  -- Check whether the insert was successful
  IF code = '00000' THEN
    GET DIAGNOSTICS nrows = ROW_COUNT;
    SET result = CONCAT('insert succeeded, row count = ',nrows);
  ELSE
    SET result = CONCAT('insert failed, error = ',code,', message = ',msg);
  END IF;
  -- Say what happened
  SELECT result;
END;
```

假设`t1.int_col`是声明为`NOT NULL`的整数列。当调用该过程以插入非`NULL`和`NULL`值时，该过程产生以下结果：

```sql
mysql> CALL do_insert(1);
+---------------------------------+
| result                          |
+---------------------------------+
| insert succeeded, row count = 1 |
+---------------------------------+

mysql> CALL do_insert(NULL);
+-------------------------------------------------------------------------+
| result                                                                  |
+-------------------------------------------------------------------------+
| insert failed, error = 23000, message = Column 'int_col' cannot be null |
+-------------------------------------------------------------------------+
```

当条件处理程序激活时，会发生对诊断区域堆栈的推送：

+   第一个（当前）诊断区域变为第二个（堆叠）诊断区域，并创建一个新的当前诊断区域作为其副本。

+   [`GET [CURRENT] DIAGNOSTICS`](get-diagnostics.html "15.6.7.3 GET DIAGNOSTICS Statement")和`GET STACKED DIAGNOSTICS`可以在处理程序内部使用，以访问当前诊断区域和堆叠诊断区域的内容。

+   最初，两个诊断区域返回相同的结果，因此可以从当前诊断区域获取有关激活处理程序的条件的信息，*只要*在处理程序内不执行更改其当前诊断区域的语句。

+   然而，在处理程序内执行的语句可以修改当前诊断区域，根据正常规则清除和设置其内容（参见诊断区域如何清除和填充）。

    获取有关激活处理程序条件的更可靠方法是使用堆叠的诊断区域，除了`RESIGNAL`之外，处理程序内执行的语句无法修改它。有关当前诊断区域何时设置和清除的信息，请参见第 15.6.7.7 节，“MySQL 诊断区域”。

下一个示例展示了如何在处理程序内部使用`GET STACKED DIAGNOSTICS`来获取有关已处理异常的信息，即使当前诊断区域已被处理程序语句修改。

在存储过程`p()`中，我们尝试向包含`TEXT NOT NULL`列的表中插入两个值。第一个值是非`NULL`字符串，第二个是`NULL`。该列禁止`NULL`值，因此第一个插入成功，但第二个导致异常。该过程包括一个异常处理程序，将尝试插入`NULL`映射为插入空字符串：

```sql
DROP TABLE IF EXISTS t1;
CREATE TABLE t1 (c1 TEXT NOT NULL);
DROP PROCEDURE IF EXISTS p;
delimiter //
CREATE PROCEDURE p ()
BEGIN
  -- Declare variables to hold diagnostics area information
  DECLARE errcount INT;
  DECLARE errno INT;
  DECLARE msg TEXT;
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    -- Here the current DA is nonempty because no prior statements
    -- executing within the handler have cleared it
    GET CURRENT DIAGNOSTICS CONDITION 1
      errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
    SELECT 'current DA before mapped insert' AS op, errno, msg;
    GET STACKED DIAGNOSTICS CONDITION 1
      errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
    SELECT 'stacked DA before mapped insert' AS op, errno, msg;

    -- Map attempted NULL insert to empty string insert
    INSERT INTO t1 (c1) VALUES('');

    -- Here the current DA should be empty (if the INSERT succeeded),
    -- so check whether there are conditions before attempting to
    -- obtain condition information
    GET CURRENT DIAGNOSTICS errcount = NUMBER;
    IF errcount = 0
    THEN
      SELECT 'mapped insert succeeded, current DA is empty' AS op;
    ELSE
      GET CURRENT DIAGNOSTICS CONDITION 1
        errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
      SELECT 'current DA after mapped insert' AS op, errno, msg;
    END IF ;
    GET STACKED DIAGNOSTICS CONDITION 1
      errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
    SELECT 'stacked DA after mapped insert' AS op, errno, msg;
  END;
  INSERT INTO t1 (c1) VALUES('string 1');
  INSERT INTO t1 (c1) VALUES(NULL);
END;
//
delimiter ;
CALL p();
SELECT * FROM t1;
```

当处理程序激活时，当前诊断区域的副本被推送到诊断区域堆栈。处理程序首先显示当前诊断区域和堆叠诊断区域的内容，最初两者都相同：

```sql
+---------------------------------+-------+----------------------------+
| op                              | errno | msg                        |
+---------------------------------+-------+----------------------------+
| current DA before mapped insert |  1048 | Column 'c1' cannot be null |
+---------------------------------+-------+----------------------------+

+---------------------------------+-------+----------------------------+
| op                              | errno | msg                        |
+---------------------------------+-------+----------------------------+
| stacked DA before mapped insert |  1048 | Column 'c1' cannot be null |
+---------------------------------+-------+----------------------------+
```

`GET DIAGNOSTICS`语句之后执行的语句可能会重置当前诊断区域。例如，处理程序将`NULL`插入映射为空字符串插入并显示结果。新插入成功并清除当前诊断区域，但堆叠的诊断区域保持不变，仍然包含激活处理程序的条件信息：

```sql
+----------------------------------------------+
| op                                           |
+----------------------------------------------+
| mapped insert succeeded, current DA is empty |
+----------------------------------------------+

+--------------------------------+-------+----------------------------+
| op                             | errno | msg                        |
+--------------------------------+-------+----------------------------+
| stacked DA after mapped insert |  1048 | Column 'c1' cannot be null |
+--------------------------------+-------+----------------------------+
```

当条件处理程序结束时，其当前诊断区域从堆栈中弹出，堆叠的诊断区域成为存储过程中的当前诊断区域。

执行完该过程后，表中包含两行。空行是由于尝试插入`NULL`而映射到空字符串插入导致的：

```sql
+----------+
| c1       |
+----------+
| string 1 |
|          |
+----------+
```

在上面的示例中，在条件处理程序中的前两个`GET DIAGNOSTICS`语句中，从当前和堆叠的诊断区域检索信息的返回值相同。如果在处理程序内部较早执行重置当前诊断区域的语句，则情况就不同了。假设`p()`被重写为将`DECLARE`语句放在处理程序定义内部而不是在其之前：

```sql
CREATE PROCEDURE p ()
BEGIN
  DECLARE EXIT HANDLER FOR SQLEXCEPTION
  BEGIN
    -- Declare variables to hold diagnostics area information
    DECLARE errcount INT;
    DECLARE errno INT;
    DECLARE msg TEXT;
    GET CURRENT DIAGNOSTICS CONDITION 1
      errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
    SELECT 'current DA before mapped insert' AS op, errno, msg;
    GET STACKED DIAGNOSTICS CONDITION 1
      errno = MYSQL_ERRNO, msg = MESSAGE_TEXT;
    SELECT 'stacked DA before mapped insert' AS op, errno, msg;
...
```

在这种情况下，结果取决于版本：

+   在 MySQL 5.7.2 之前，`DECLARE`不会更改当前诊断区域，因此前两个`GET DIAGNOSTICS`语句返回相同的结果，就像在`p()`的原始版本中一样。

    在 MySQL 5.7.2 中，已经做了工作以确保所有非诊断语句填充诊断区域，符合 SQL 标准。`DECLARE`是其中之一，因此在 5.7.2 及更高版本中，执行处理程序开头的`DECLARE`语句会清除当前诊断区域，并且`GET DIAGNOSTICS`语句会产生不同的结果：

    ```sql
    +---------------------------------+-------+------+
    | op                              | errno | msg  |
    +---------------------------------+-------+------+
    | current DA before mapped insert |  NULL | NULL |
    +---------------------------------+-------+------+

    +---------------------------------+-------+----------------------------+
    | op                              | errno | msg                        |
    +---------------------------------+-------+----------------------------+
    | stacked DA before mapped insert |  1048 | Column 'c1' cannot be null |
    +---------------------------------+-------+----------------------------+
    ```

在条件处理程序中避免此问题时，当试图获取激活处理程序的条件的信息时，请确保访问堆叠的诊断区域，而不是当前诊断区域。
